---
# DinoSurvivor — Sprite Types
**ARK-Inspired 2D Sprite Design Document**

---
## Overview
This document defines the **sprite types** for DinoSurvivor, categorized into **decorative (non-interactive)** and **resource (interactive)** sprites. Sprites are designed to align with ARK’s aesthetic while being simple and performant for a 2D mobile game.

---

---
## Key Differences Between Sprite Types

| **Feature**          | **Decorative Sprites**               | **Resource Sprites**                     |
|----------------------|--------------------------------------|------------------------------------------|
| **Purpose**          | Visual immersion only.               | Gatherable (interactive).                 |
| **Color**            | Natural, blended.                    | Brighter or more saturated.               |
| **Shape**            | Simple, irregular.                   | More defined.                             |
| **Size**             | Smaller or varied.                   | Slightly larger.                          |
| **Outline/Effect**   | None.                                | Subtle glow or outline when MC is nearby. |
| **Density**          | Higher (e.g., 5% for small rocks).   | Lower (e.g., 2-3%).                       |
| **Collision**        | Some collidable, some not.           | **Not always collidable**.               |

---

---
## Sprite Types

---
### 1. Decorative Sprites (Non-Interactive)
**Purpose**: Enhance visual diversity and immersion.

| **Sprite Type**       | **Description**               | **Color**      | **Size**       | **Density** | **Collidable** | **Notes**                     |
|-----------------------|-------------------------------|----------------|----------------|--------------|-----------------|--------------------------------|
| Rock (Small)          | Small gray boulders.          | `#808080`      | 20x20px        | 5%           | No              | Scattered naturally.          |
| Rock (Large)          | Larger rock formations.       | `#696969`      | 40x40px        | 2%           | Yes             | Blocks movement.              |
| Grass Tufts          | Clumps of taller grass.       | `#7CFC00`      | 10x30px        | 10%          | No              | Overlays on grass chunks.     |
| Flowers              | Small colorful flowers.      | `#FF69B4`      | 5x5px          | 3%           | No              | Adds color variety.           |
| Bushes               | Dense green bushes.          | `#228B22`      | 30x30px        | 5%           | **No**          | Non-gatherable (decor only).   |
| Tree Stumps          | Remnants of trees.            | `#8B4513`      | 25x15px        | 2%           | Yes             | Non-gatherable (decor only).   |
| Water Plants         | Simple aquatic plants.       | `#006400`      | 15x30px        | 4%           | No              | Only on water chunks.         |

---
---
### 2. Resource Sprites (Interactive)
**Purpose**: Gatherable items tied to **Phase 3 (Resources & Crafting)**.

| **Sprite Type**       | **Description**               | **Color**      | **Size**       | **Density** | **Drops**       | **Biome**       | **Visual Cues**                     | **Collidable** |
|-----------------------|-------------------------------|----------------|----------------|--------------|------------------|-----------------|--------------------------------------|-----------------|
| Gatherable Rock       | Jagged, shiny rock.           | `#A9A9A9`      | 30x30px        | 2%           | Stone           | All             | White outline when nearby.           | **No**          |
| Tree                  | Tall with a brown trunk.      | `#228B22` (leaves), `#8B4513` (trunk) | 40x60px | 3% | Wood | Grass/Dirt | Thicker trunk + glow effect. | **Yes**         |
| Berry Bush            | Green bush with red berries.  | `#228B22` (bush), `#FF0000` (berries) | 25x25px | 3% | Berries | Grass/Dirt | Pulsing berries (animation). | **No**          |
| Metal Node            | Shiny gray rock with sparkles.| `#C0C0C0`      | 20x20px        | 1%           | Metal           | Dirt (rare)     | Sparkle animation.              | **Yes**         |

---
---
## Visual Cues for Interactivity
To make **Resource Sprites** stand out, use the following visual cues:

1. **Outline**:
   - Add a **white or yellow outline** (2px) when the MC is within **100px**.
   - Example:
     ```javascript
     // In renderMap():
     if (isResourceSprite(chunk) && distanceToMC < 100) {
       ctx.strokeStyle = '#FFFFFF';
       ctx.lineWidth = 2;
       ctx.strokeRect(sx, sy, chunk.sprite.size, chunk.sprite.size);
     }
     ```

2. **Glow Effect**:
   - Apply a **subtle glow** (e.g., `ctx.shadowBlur = 5; ctx.shadowColor = '#FFFF00'`) to resource sprites.

3. **Animation**:
   - **Berry Bushes**: Pulsing berries (scale up/down slightly).
   - **Metal Nodes**: Sparkle effect (random white dots appearing/disappearing).

---
---
## Implementation Notes

### Sprite Placement Logic
- **Decorative Sprites**: Place randomly on all terrain types (grass, dirt, water) based on their density.
- **Resource Sprites**: Place based on biome rules (e.g., trees on grass/dirt, metal nodes on dirt).

### Example Code Snippet
```javascript
// Define sprites:
const sprites = [
  // Decorative Sprites
  { type: 'decor_rock_small', color: '#808080', size: 20, density: 0.05, collidable: false, resource: false },
  { type: 'decor_rock_large', color: '#696969', size: 40, density: 0.02, collidable: true, resource: false },
  { type: 'decor_grass_tuft', color: '#7CFC00', size: 10, density: 0.10, collidable: false, resource: false },
  { type: 'decor_flower', color: '#FF69B4', size: 5, density: 0.03, collidable: false, resource: false },
  { type: 'decor_bush', color: '#228B22', size: 30, density: 0.05, collidable: false, resource: false },
  { type: 'decor_tree_stump', color: '#8B4513', size: 25, density: 0.02, collidable: true, resource: false },
  { type: 'decor_water_plant', color: '#006400', size: 15, density: 0.04, collidable: false, resource: false },

  // Resource Sprites
  { type: 'resource_rock', color: '#A9A9A9', size: 30, density: 0.02, collidable: false, resource: true },
  { type: 'resource_tree', color: '#228B22', size: 60, density: 0.03, collidable: true, resource: true },
  { type: 'resource_berry_bush', color: '#228B22', size: 25, density: 0.03, collidable: false, resource: true },
  { type: 'resource_metal_node', color: '#C0C0C0', size: 20, density: 0.01, collidable: true, resource: true }
];

// Place sprites randomly:
for (const sprite of sprites) {
  for (let i = 0; i < TOTAL_CHUNKS * sprite.density; i++) {
    const idx = Math.floor(Math.random() * TOTAL_CHUNKS);
    if (!map[idx].sprite) { // Avoid overwriting existing sprites
      map[idx].sprite = sprite;
    }
  }
}
```

---
---
## ARK-Inspired Biome Mapping
To mimic ARK’s biomes, assign sprite sets to terrain types:

| **Terrain** | **ARK Biome Equivalent** | **Sprite Types**                          |
|-------------|--------------------------|------------------------------------------|
| Grass       | Plains                   | Trees, bushes, flowers, Raptors, Rex     |
| Dirt        | Forest/Jungle            | Large rocks, tree stumps, Stegos, Triceratops |
| Water       | Ocean/Shore              | Water plants, small rocks, Plesiosaurs (future) |

---
---
## Next Steps
1. **Implement Decor Sprites First**:
   - Add **rocks (small/large), grass tufts, and flowers** to test rendering performance.

2. **Add Resource Sprites**:
   - Start with **gatherable rocks and trees**.

3. **Test Visual Cues**:
   - Verify that **outlines/glow effects** are visible and intuitive.

4. **Plan for Collision**:
   - Ensure **collidable sprites** block movement where applicable.
---