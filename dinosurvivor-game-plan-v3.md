# DinoSurvivor — Game Plan

**ARK-Inspired 2D Mobile Survival Game**

---

## Overview

A mobile-friendly 2D survival game inspired by **ARK: Survival Evolved**, simplified for performance and incremental development. The game focuses on:

- Dino recognition by silhouette at varying distances.
- Taming mechanics (knockout and passive).
- Survival systems (health, food, stamina).
- Chunk-based world with culling and terrain clustering (Number of Islands algorithm).

---

---

## Phase 1: Core World & Visual Identity

**Goal**: Establish the foundation for recognition, movement, and world interaction.  
**Dependencies**: None.

---

### Sub-Tasks Breakdown

---

#### 1. Generate the Map

**Goal**: Create a **5000x5000px fixed-size map** with chunk-based terrain, optimized rendering, and natural clustering.

---

##### 1.1 Chunk-Based World

- Implement a **5000x5000px map** divided into **200x200px chunks** (25x25 grid).
- Use **culling** to render only visible chunks (with a **200px buffer** to prevent popping).

---

##### 1.2 Terrain Types


| Terrain Type | Color     | Percentage | Distribution                                   |
| ------------ | --------- | ---------- | ---------------------------------------------- |
| Grass        | `#7CFC00` | 60%        | Default fill, clustered via cellular automata. |
| Dirt         | `#8B4513` | 30%        | Clustered via cellular automata.               |
| Water        | `#1E90FF` | 10%        | 70% in edge regions, 30% random.               |


**Steps**:

1. Initialize all chunks as `GRASS`.
2. Place **70% of water chunks in edge regions** (within 3 chunks of the border).
3. Place the remaining **30% of water randomly**.
4. Convert **30% of non-water chunks to DIRT** (randomly selected, then clustered via cellular automata).
5. Apply **3 passes of cellular automata** to grass and dirt only (water remains fixed):
  - For each non-water chunk, count the most common neighbor type (grass/dirt).
  - Set the chunk’s type to the majority neighbor type if it has **≥4 neighbors of the same type**.

---

##### 1.3 Fixed-Size Map

- The map is **static (5000x5000px)** with no dynamic loading/unloading.
- **Camera clamping** prevents the MC from moving outside the map bounds.

---

##### 1.4 Decorative Sprites (WIP)


| **Sprite Type** | **Description**         | **Color** | **Size** | **Density** | **Collidable** | **Notes**                    |
| --------------- | ----------------------- | --------- | -------- | ----------- | -------------- | ---------------------------- |
| Rock (Small)    | Small gray boulders.    | `#808080` | 20x20px  | 5%          | No             | Scattered naturally.         |
| Rock (Large)    | Larger rock formations. | `#696969` | 40x40px  | 2%          | Yes            | Blocks movement.             |
| Grass Tufts     | Clumps of taller grass. | `#7CFC00` | 10x30px  | 10%         | No             | Overlays on grass chunks.    |
| Flowers         | Small colorful flowers. | `#FF69B4` | 5x5px    | 3%          | No             | Adds color variety.          |
| Bushes          | Dense green bushes.     | `#228B22` | 30x30px  | 5%          | No             | Non-gatherable (decor only). |
| Tree Stumps     | Remnants of trees.      | `#8B4513` | 25x15px  | 2%          | Yes            | Non-gatherable (decor only). |
| Water Plants    | Simple aquatic plants.  | `#006400` | 15x30px  | 4%          | No             | Only on water chunks.        |


- Plan for additional sprite types.

NOTE: ref document - DinoSurvivor — Sprite Types

---

##### 1.5 Culling

- Calculate visible bounds: `cameraX ± canvas.width/2 + 200px buffer`.
- Render only chunks that **overlap with the visible area**.

---

#### 2. Dino Shapes

**Goal**: Ensure dinos are recognizable by silhouette at varying distances (10px, 20px, 40px).

---

##### 2.1 Shape Design

- Use **simple rectangles** for dinos (e.g., Rex = large rectangle, Raptor = smaller rectangle).
- Add **distinctive features** (e.g., spikes for Stego, long neck for Brachiosaurus) using `ctx.fillRect`.
- **Map sprite shapes and positions (WIP)**.

---

##### 2.2 Scaling

- Scale dino shapes based on distance from the MC:
  - **Far**: 10px (no details, just silhouette).
  - **Medium**: 20px (show name + level).
  - **Close**: 40px (show name + level + health bar).

---

#### 3. Distance-Based Recognition

**Goal**: Display contextual UI for dinos based on their distance from the MC.

---

##### 3.1 Distance Thresholds

- **Far**: >100px (no labels).
- **Medium**: 100–200px (show name + level).
- **Close**: <100px (show name + level + health bar).

---

##### 3.2 UI Rendering

- Draw text labels (name, level) above dinos at medium/close range using `ctx.fillText`.
- Draw a health bar (green rectangle) below the dino at close range using `ctx.fillRect`.
- *Simplification*: Skip the Spyglass for now; focus on scaling shapes for readability.

---

**Output**:  
Dinos display contextual UI based on their distance from the MC.

---

---

#### 4. Floating Joystick

**Goal**: Ensure the joystick spawns at touch origin and controls the MC smoothly.

---

##### 4.1 Joystick Spawning

- Spawn the joystick at the touch/mouse position when pressed.
- Hide it when released.

---

##### 4.2 MC Movement

- Move the MC proportionally to joystick input (no easing).
- Cap the max speed at 30.

---

##### 4.3 Boundary Checks

- Prevent the MC from moving outside the canvas (clamp to map bounds).

---

**Output**:  
The MC moves responsively with the floating joystick, with no drift or delay.

---

---

### Suggested Workflow for Phase 1

1. **Start with Task 1 (Map Generation)**:
  - Implement the chunk system and terrain types (grass, dirt, water).
  - Test scrolling and culling.
  - Apply cellular automata for terrain clustering.
  - Ensure water is concentrated in **edge regions (70%)** and randomly distributed (30%).
  - Add decorative sprites on **all terrain types**. Plan for additional sprite types.
2. **Move to Task 2 (Dino Shapes)**:
  - Add one dino at a time (e.g., start with Rex, then Raptor, etc.).
  - Test each dino’s recognizability at all scales (10px, 20px, 40px).
3. **Proceed to Task 3 (Distance-Based Recognition)**:
  - Implement distance thresholds and UI rendering.
  - Test with multiple dinos on screen.
4. **Finalize Task 4 (Floating Joystick)**:
  - Ensure the joystick spawns correctly and controls the MC smoothly.

---

**Note**: After Phase 1, we will discuss detailed sub-tasks for the remaining phases.

---

---

## Phase 2: Temperament AI & Behavior

**Goal**: Introduce dino behaviors to make the world feel alive.  
**Dependencies**: Phase 1 (world).

---

### Sub-Tasks Breakdown

---

#### 1. Dino Behaviors

- Implement **3 temperaments**:
  - **Passive**: Flee from MC.
  - **Neutral**: Ignore MC unless attacked.
  - **Aggressive**: Chase MC on sight.

---

#### 2. Movement Patterns

- **Passive**: Move away from MC (no pathfinding; simple vector away).
- **Neutral**: Wander randomly within a small radius.
- **Aggressive**: Move toward MC (simple vector toward).

---

#### 3. Attack System

- **Aggressive dinos**: Deal damage to MC on collision.
- **MC**: Deal damage to dinos on collision (melee attack).

---

---

## Phase 3: Resources & Crafting

**Goal**: Add progression through gathering and crafting.  
**Dependencies**: Phase 1 (world).

---

### Sub-Tasks Breakdown

---

#### 1. Resource Nodes

- Add **static resource nodes** (e.g., trees, rocks) to the map.
- Some are collidable, some are not.
- **Types**:
  - Trees: Drop wood.
  - Rocks: Drop stone.
  - Bushes: Drop berries.

---

#### 2. Gathering Mechanics

- MC can **collide with resource nodes** to gather resources.
- **Cooldown**: 1 second per gather to prevent spamming.

---

#### 3. Inventory System

- MC has an **inventory** (e.g., `{ wood: 0, stone: 0, berries: 0 }`).
- **UI**: Display inventory counts at the top of the screen.

---

#### 4. Crafting System

- **Recipes**:
  - Wood + Stone = Campfire.
  - Wood = Wooden Pickaxe (increases stone gather rate).
- **UI**: Crafting menu (e.g., press `C` to open).

---

---

## Phase 4: Taming & Riding

**Goal**: Implement taming and riding mechanics.  
**Dependencies**: Phase 2 (AI) + Phase 3 (resources).

---

### Sub-Tasks Breakdown

---

#### 1. Taming Mechanics

- **Knockout Taming**:
  - MC attacks a dino repeatedly to knock it out (health bar turns gray).
  - Feed the knocked-out dino berries to tame it.
- **Passive Taming**:
  - Feed a passive dino berries repeatedly to tame it.

---

#### 2. Riding Mechanics

- Tamed dinos can be ridden by the MC (press `E` to mount/dismount).
- **Movement**: Ridden dinos move with the MC (joystick controls the dino).

---

---

## Phase 5: Survival Systems

**Goal**: Add core survival mechanics.  
**Dependencies**: Phase 1 (world) + Phase 3 (resources).

---

### Sub-Tasks Breakdown

---

#### 1. Health System

- MC has **100 health**.
- **Damage Sources**:
  - Aggressive dinos: -10 health per hit.
  - Falling: -5 health per chunk height.
- **Healing**:
  - Consume berries: +5 health.
  - Sleep in a bed: +1 health per second.

---

#### 2. Food System

- MC has **100 food**.
- **Food Consumption**: -1 food per minute.
- **Starvation**: Below 20 food, health regenerates 50% slower.
- **Healing**: Consume berries: +10 food.

---

#### 3. Stamina System

- MC has **100 stamina**.
- **Stamina Consumption**:
  - Running: -1 stamina per second.
  - Attacking: -5 stamina per hit.
- **Regeneration**: +1 stamina per second when not running.

---

#### 4. Day/Night Cycle

- **Cycle Length**: 10 minutes (5 minutes day, 5 minutes night).
- **Effects**:
  - **Night**: Reduced visibility (darker screen overlay).
  - **Day**: Full visibility.
- **Time UI**: Display current time (e.g., "Day: 02:30").

---

#### 5. Respawn System

- If MC dies, respawn at the last placed bed (from Phase 3).
- If no bed exists, respawn at the world origin.

---

---

## Field Notes

- **Performance**: Prioritize **culling** and **chunk-based rendering** for smooth mobile performance.
- **Simplifications**:
  - Use **rectangles** for dinos and resources.
  - Skip **pathfinding**; use simple vector-based movement.
- **Testing**:
  - Test each phase independently before integrating.
  - Use `console.log` for debugging (e.g., MC position, dino states).