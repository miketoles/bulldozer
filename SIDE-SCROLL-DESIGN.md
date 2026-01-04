# Bulldozer Side-Scroll - Design Document

**Version:** 2.0
**Status:** Implemented & Playable
**File:** `bulldozer-side-scroll.html`

---

## Core Philosophy

**The bulldozer is an extension of your finger.**

The player should feel in complete control, as if the bulldozer is a puppet connected directly to their fingertip. Movement is responsive, immediate, and intuitive. The only resistance comes from the dirt itself - creating satisfying physics feedback when pushing through tough piles.

---

## Control Scheme

### The Golden Rule

> The bulldozer always tries to match the finger's position and movement as closely as possible.

- **No artificial lag or sluggishness**
- **Finger moves fast → bulldozer moves fast to catch up**
- **Finger moves slow → bulldozer moves slow, precise control**

### Horizontal Control (Direction & Speed)

| Finger Position | Bulldozer Behavior |
|-----------------|-------------------|
| Right of bulldozer | Drives right to catch up |
| Left of bulldozer | Drives left to catch up |
| Far from bulldozer | Moves faster to close the gap |
| Close to bulldozer | Moves slower, precise control |

**Speed Formula:**
```
speed = distance_to_finger × 0.15
```

### Vertical Control (Scoop Height) - RELATIVE

**Important:** Scoop control is RELATIVE to where you first touch, not absolute position.

| Finger Movement | Scoop State | Effect |
|-----------------|-------------|--------|
| Touch down | Scoop starts level | Neutral position |
| Move UP from start | Scoop raises | Climb over piles |
| Move DOWN from start | Scoop lowers | Dig and push dirt |

**Implementation:**
```javascript
fingerDeltaY = fingerY - startFingerY
targetAngle = fingerDeltaY * 1.2
clampedAngle = clamp(-70, +25, targetAngle)  // -70° up, +25° down
```

- Scoop won't go below ground level (max +25°)
- Can raise high to clear obstacles (up to -70°)
- Response factor: 0.8 (nearly instant)

### Turning Behavior

When finger crosses from one side to the other:
- Bulldozer immediately starts smooth rotation
- Uses scaleX animation: 1 → 0 → -1 (or reverse)
- Completes in ~7 frames
- Scoop always stays in front

---

## Dirt Physics

### Two Dirt Modes (Switchable)

**Mode: `'disappear'`** (Current Default)
- Dirt vanishes when scooped
- Wider scoop effect (5 columns)
- Satisfying for clearing terrain

**Mode: `'push'`** (Preserved)
- Dirt piles up ahead of scoop
- Realistic bulldozer behavior
- Angle of repose settling

To switch modes, change `dirtMode` property in code.

### Resistance System

Resistance scales with:
1. **Scoop depth** - Deeper scoop = more resistance
2. **Dirt ahead** - More dirt = more resistance
3. **Pushing uphill** - Significant extra resistance

```javascript
// No resistance when scoop raised (angle < -10)
// Base resistance from digging: up to 15% slowdown
// Extra from dirt piles: scales with height
// Max slowdown capped at 80%
```

### Terrain Model

```
Terrain = Array of columns, each with:
- height (0-100+ pixels)
- velocity (for settling physics)

Column width: 6 pixels
Ground level: 60% down screen
```

---

## Bulldozer Visual Design (Implemented)

### Component Structure

```
        ┌────────┐
        │ CABIN  │ ← Dark grey (#3A3A3A), two windows
        │ [w][W] │   Small back window, tall front window
    ┌───┴────────┴───────┐
    │      BODY          │ ← Yellow (#FFD700), orange stripes
    └────────────────────┘
   ╭────────────────────────╮
   │ ●    ●     ●     ●    │ ← Pill-shaped tracks, animated treads
   ╰────────────────────────╯   Drive wheel (back), idler (front), road wheels

        ══════════╗
                  ║ ← Hydraulic arm (yellow/gold)
              ┌───╨───┐
              │ BLADE │ ← Silver scoop, pivots with finger
              └───────┘
```

### Color Palette

| Component | Color | Hex |
|-----------|-------|-----|
| Body | Construction Yellow | #FFD700 |
| Body Stripes | Orange | #FFA500 |
| Cabin | Dark Grey | #3A3A3A |
| Windows | Sky Blue | #87CEEB |
| Tracks | Dark Slate | #2F4F4F |
| Wheels | Near Black | #1C1C1C |
| Scoop/Blade | Silver | #C0C0C0 |
| Arm | Goldenrod | #DAA520 |
| Exhaust | Grey | #696969 |

### Track Design (Based on Real Bulldozer Reference)

- **Shape:** Rounded rectangle (pill shape) using `roundRect()`
- **Wheels:** Large drive wheel (back), large idler (front), smaller road wheels (middle)
- **Treads:** Animated vertical lines, CLIPPED to track bounds
- **Position:** Extends past body at front and back

### Animations

- **Track treads:** Scroll with movement (trackOffset)
- **Exhaust smoke:** Random puffs when moving
- **Direction flip:** Smooth scaleX transition (1 → 0 → -1)

---

## Camera System

- Camera smoothly follows bulldozer (lerp factor: 0.1)
- Bulldozer stays roughly centered
- Infinite scrolling in both directions

---

## World Design

### Infinite Terrain

- Procedural generation using layered sine waves
- Extends automatically as player moves
- Column-based heightmap

### Terrain Generation

```javascript
height = 30
  + sin(x * 0.02) * 25      // Large hills
  + sin(x * 0.05) * 15      // Medium variations
  + sin(x * 0.1) * 8        // Small bumps
```

### Visual Layers

1. **Sky:** Gradient (#87CEEB → #E0F4FF)
2. **Background hills:** Parallax (muted blue/purple)
3. **Terrain:** Depth-based brown shading
4. **Grass line:** Green accent on surface
5. **Bedrock:** Dark brown below ground level

---

## UI Elements

- **Score panel:** "Dirt Moved" counter (top-left)
- **Reset button:** Generates fresh world
- **Finger indicator:** White circle when dragging

---

## Technical Implementation

### File Structure
Single HTML file: `bulldozer-side-scroll.html`
- All CSS inline in `<style>`
- All JavaScript in single `game` object
- No external dependencies

### Key Methods

| Method | Purpose |
|--------|---------|
| `init()` | Setup canvas, events, start game loop |
| `update()` | Handle input, physics, movement |
| `draw()` | Render all visual elements |
| `drawBulldozer()` | Render bulldozer with all components |
| `pushDirt()` | Handle dirt deformation (both modes) |
| `updateDirtPhysics()` | Angle of repose settling |
| `generateTerrain()` | Procedural terrain creation |
| `extendWorldIfNeeded()` | Infinite scroll expansion |

### Performance
- Target: 60 FPS
- Uses `requestAnimationFrame`
- Efficient column-based terrain

---

## MVP Checklist

- [x] Side-view bulldozer sprite
- [x] Finger-following movement (responsive, no lag)
- [x] Scoop follows finger Y position (RELATIVE control)
- [x] Smooth turning/rotation
- [x] Terrain heightmap
- [x] Dirt pushing/disappearing physics
- [x] Angle of repose settling
- [x] Camera scrolling
- [x] Infinite terrain generation
- [x] Depth-based dirt coloring
- [x] Reset button
- [ ] Particle effects (dust, dirt chunks)
- [ ] Sound effects

---

## Future Enhancements

### Particle Effects (Next Priority)

#### 1. Dirt Chunks When Digging
**Trigger:** When scoop is lowered and moving through dirt
**Behavior:**
- Small brown squares/circles spray up and out from the scoop
- Trajectory: arc upward, then fall with gravity
- Size: 3-6 pixels, varying
- Color: Match terrain depth colors (light to dark brown)
- Quantity: More particles = deeper scoop + more dirt

**Implementation approach:**
```javascript
particles = []  // Array of {x, y, vx, vy, size, color, life}
// Spawn particles in pushDirt() or removeDirt()
// Update positions with gravity in update()
// Draw in draw() loop, remove when life expires
```

#### 2. Dust Clouds Behind Tracks
**Trigger:** When bulldozer is moving
**Behavior:**
- Small tan/beige circles spawn behind the tracks
- Drift upward slightly, fade out over time
- More dust = faster movement
- Stays low to ground

#### 3. Enhanced Exhaust Smoke
**Current:** Random puffs when moving
**Enhanced:**
- Continuous stream of grey circles
- Rise upward with slight drift
- Fade from grey to transparent
- Size grows as they rise

#### 4. Impact Particles
**Trigger:** When scoop first hits a dirt pile after being raised
**Behavior:**
- Burst of particles in all directions
- Quick, satisfying "crunch" feeling
- Could add screen shake for extra impact

#### Implementation Order
1. **Dirt chunks** (most impactful, directly tied to gameplay)
2. **Dust clouds** (adds motion feel)
3. **Enhanced exhaust** (polish)
4. **Impact particles** (extra polish)

#### Performance Considerations
- Cap max particles (e.g., 100-200)
- Use object pooling to avoid garbage collection
- Simple physics (just gravity + velocity)
- Remove particles when off-screen or life expired

### Other Ideas
- Sound effects (engine, dirt, hydraulics)
- Different terrain types
- Day/night cycle
- Obstacles to push around

---

## Design Decisions Log

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Scoop control | Relative to touch start | More intuitive, starts level |
| Dirt mode | Disappear (default) | More satisfying for casual play |
| Turn animation | Smooth scaleX flip | Looks natural, no jarring snap |
| Track style | Pill-shaped with wheels | Based on real bulldozer reference |
| Cabin color | Dark grey | Contrasts with yellow body |
| Max scoop down | +25° | Prevents unrealistic underground dig |

---

*A satisfying, tactile, zen-like dirt-pushing experience.*
