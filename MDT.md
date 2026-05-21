# MDT API Reference

## 1. `MDT:GetCurrentPreset()` — function

`MythicDungeonTools.lua:1924`

```lua
function MDT:GetCurrentPreset()
  return db.presets[db.currentDungeonIdx][db.currentPreset[db.currentDungeonIdx]]
end
```

### Output

A single **preset table** — the currently selected route for the currently selected dungeon. Structure:

| Field | Type | Description |
|---|---|---|
| `text` | string | Preset display name (e.g. `"Default"`). The sentinel `"<New Preset>"` entry has `value = 0`. |
| `value` | table | The route payload (see below). For the `<New Preset>` placeholder this is the number `0`. |
| `value.pulls` | array | List of pulls; each pull maps `enemyIdx` (numeric string key) → array of `cloneIdx` numbers. Pulls also carry meta keys like `color`. |
| `value.currentPull` | number | Index of the currently selected pull. |
| `value.currentSublevel` | number | Currently displayed sublevel/floor. |
| `value.currentDungeonIdx` | number | Dungeon index this preset belongs to. |
| `week` | number (1–10) | Affix week the preset is built for. |
| `difficulty` | number | Keystone level. |
| `uid` | string | Unique ID (set lazily via `SetUniqueID`). |
| `objects` | table | Free-draw annotations (lines, notes, arrows). |
| `colorPaletteInfo` | table | `{ autoColoring = bool, colorPaletteIdx = number }`. |
| `mdiEnabled`, `createdBy`, etc. | various | Optional metadata. |

It always returns a live reference (callers mutate it directly, e.g. `MDT:GetCurrentPreset().week = key`). It assumes the dungeon/preset indices are valid — no nil-guard.

---

## 2. `MDT.dungeonEnemies` — table

Declared empty at `MythicDungeonTools.lua:326` (`MDT.dungeonEnemies = {}`), then populated by each dungeon file (e.g. `Midnight/AlgetharAcademy.lua:90`).

### Structure

`MDT.dungeonEnemies[dungeonIndex][enemyIdx]` → an **enemy definition**. Indexed by dungeon index (outer) then sequential enemy index (inner). The authoritative field list is the `"enemies"` schema in `Developer/Schema.lua:205`.

| Field | Type | Description |
|---|---|---|
| `name` | string | Creature name. |
| `id` | number | NPC ID. |
| `count` | number | Enemy forces / count value contributed. |
| `health` | number | Base health. |
| `scale` | number | Display scale. |
| `displayId` | number | Model display ID. |
| `creatureType` | string | e.g. `"Elemental"`, `"Humanoid"`. |
| `level` | number | Creature level. |
| `isBoss` | boolean | Boss flag. |
| `encounterID` / `instanceID` | number | Journal encounter/instance IDs. |
| `ignoreFortified`, `neutral`, `stealth`, `stealthDetect` | boolean | Behavior flags. |
| `iconTexture` | number | Optional icon override. |
| `characteristics` | table | Map of crowd-control type → boolean (`Stun`, `Fear`, `Taunt`, …). |
| `spells` | table | Map of `spellId` → `{ interruptible, magic, poison, disease, curse, bleed, enrage }` (booleans). |
| `powers` | table | Per-role power info (`dps`, `healer`, `tank` booleans). |
| `clones` | array | Each entry is one placed instance of the enemy. |

**Clone entry** (`clones[cloneIdx]`): `x`, `y` (map coords), `g` (group/pull-group number), `sublevel` (number), `scale` (number), optional `note` (string), `patrol` (array of `{x,y}` waypoints), `constrained` (`{index, amount}`).

Used throughout for force counting, health sums, and clone-inclusion checks, e.g. `self.dungeonEnemies[db.currentDungeonIdx][enemyIdx].count`.

