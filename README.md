# Motivations of Gaia — Worldgen V2 Prefab Integration (Hytale)

This repository contains the **Motivations of Gaia** mod for Hytale.  
This README documents, in detail, how we got a custom Prefab (the **Orb Temple**) to naturally generate using **Hytale Worldgen V2**. The information below is based on real debugging in a live mod project and is meant to be a reliable, reproducible guide for other creators.

If you are new to Hytale’s worldgen assets, this will save you hours.

---

## Quick Summary (The Fix)

The prefab wasn’t spawning because the **Prefab path in the Assignment asset pointed to a folder-like path** instead of the actual prefab file.  
Hytale logged:

```
This prefab path contains no prefabs: MotivationsOfGaia/Structures/OrbTemple/OrbTemple
```

**Correct path** (note the `.prefab.json` extension):

```
MotivationsOfGaia/Structures/OrbTemple/OrbTemple.prefab.json
```

After rebuilding the mod JAR, the prefab spawned naturally during world generation.

---

## Project Structure (Relevant Files)

Key assets used for Worldgen V2 prefab spawning:

- Prefab asset  
  `src/main/resources/Server/Prefabs/MotivationsOfGaia/Structures/OrbTemple/OrbTemple.prefab.json`

- Assignment asset (spawner logic)  
  `src/main/resources/Server/HytaleGenerator/Assignments/OrbTemple_Assignments.json`

- Biome overrides (hook the assignment into Plains biomes)  
  `src/main/resources/Server/HytaleGenerator/Biomes/Plains1/Plains1_Oak.json`  
  `src/main/resources/Server/HytaleGenerator/Biomes/Plains1/Plains1_Shore.json`  
  `src/main/resources/Server/HytaleGenerator/Biomes/Plains1/Plains1_Mountains.json`  
  `src/main/resources/Server/HytaleGenerator/Biomes/Plains1/Plains1_Deeproot.json`  
  `src/main/resources/Server/HytaleGenerator/Biomes/Plains1/Plains1_Gorges.json`  
  `src/main/resources/Server/HytaleGenerator/Biomes/Plains1/Plains1_River.json`

---

## How We Implemented Natural Prefab Spawning

### 1. Create the Prefab

We created a prefab called **OrbTemple** and stored it here:

```
Server/Prefabs/MotivationsOfGaia/Structures/OrbTemple/OrbTemple.prefab.json
```

This is the actual file that must be referenced by the worldgen assignment.

---

### 2. Build a Worldgen Assignment Asset

We created a custom Assignment that points to our prefab:

**File:** `Server/HytaleGenerator/Assignments/OrbTemple_Assignments.json`

Inside it, the prefab is referenced via `WeightedPrefabPaths`:

```json
"WeightedPrefabPaths": [
  {
    "Path": "MotivationsOfGaia/Structures/OrbTemple/OrbTemple.prefab.json",
    "Weight": 10
  }
]
```

**Important:** The path must resolve to an actual prefab file.  
Do not omit the `.prefab.json` extension unless you are pointing at a folder of prefabs.

---

### 3. Attach the Assignment to Biomes

We injected the assignment into the Plains biome variants using the `Props` section.  
Example from `Plains1_Oak.json`:

```json
"Props": [
  {
    "Skip": false,
    "Runtime": 0,
    "Positions": {
      "Type": "Occurrence",
      "Seed": "OrbTemple",
      "FieldFunction": {
        "Type": "Constant",
        "Value": 0.01
      },
      "Positions": {
        "Type": "Mesh2D",
        "PointGenerator": {
          "Type": "Mesh",
          "ScaleX": 300,
          "ScaleY": 300,
          "ScaleZ": 300,
          "Jitter": 0.2
        }
      }
    },
    "Assignments": {
      "Type": "Imported",
      "Name": "OrbTemple_Assignments"
    }
  }
]
```

This is how the prefab gets **natural placement**, not forced / manually spawned.

---

## Debugging Process (What Actually Broke It)

### Symptom

The prefab never appeared, **no errors**, and worldgen looked normal.

### Log Clue

In the server log:

```
This prefab path contains no prefabs: MotivationsOfGaia/Structures/OrbTemple/OrbTemple
```

### Root Cause

The path in the assignment was interpreted as a folder, and the engine found no prefabs there.  
Hytale expects:

- A **folder path** ending in `/` that contains prefabs
  - Example: `Monuments/Unique/Mage_Towers/Sandstone/Tier_3/`
- Or a **full prefab file path**
  - Example: `MotivationsOfGaia/Structures/OrbTemple/OrbTemple.prefab.json`

Our original path didn’t match either case, so it silently failed.

---

## Verification (How We Confirmed the Fix)

After updating the path, we verified the mod JAR contained the correct assignment:

```text
Server/HytaleGenerator/Assignments/OrbTemple_Assignments.json
Path = MotivationsOfGaia/Structures/OrbTemple/OrbTemple.prefab.json
```

The prefab then spawned naturally in the world.

---

## Practical Tips for Reliable Spawning

If you’re not seeing your prefab yet, try these short-term debugging settings:

- Increase spawn chance:
  - `FieldFunction.Value` from `0.01` → `0.5`
- Increase mesh density:
  - `ScaleX/ScaleY/ScaleZ` from `300` → `75`

Once you confirm it works, scale those values back down.

---

## Build & Deploy

This project builds its JAR directly into the Hytale Mods folder.

Output directory (from `pom.xml`):

```
${user.home}/.var/app/com.hypixel.HytaleLauncher/data/Hytale/UserData/Mods
```

After every change:

1. Build the mod
2. Restart worldgen / recreate the test world
3. Check the latest log for prefab-path warnings

---

## Credits

This mod and worldgen work is by **Papercraft Games**.
If you build on this, please give credit and share improvements with the community.

