# 7-days-winter-survival
7-days-winter-survival
# ❄️ 7 Days of Winter Survival (Roblox)

A 7-day survival game prototype built in Luau for Roblox Studio. Players must gather resources, build shelters, manage hunger, and stay warm as blizzards grow deadlier every single day.

## ✨ Features
- **7-Day Escalating Storm System:** Each day introduces harsher weather, thicker fog, and faster body heat drain—culminating in *The Final Freeze* on Day 7.
- **Day & Night Cycle:** Daytime allows safer resource gathering; night triggers intense *Blizzard Surges*.
- **Warmth & Shelter Mechanics:** Campfires provide radial heat, and raycast roof detection rewards players who build enclosed structures.
- **Resource Gathering & Building:** Harvest wood, stone, and berries to craft walls (`B`) and roofs (`R`).

## 🛠️ Roblox Studio Setup

### 1. Workspace
- Create a **Folder** named `Resources` containing Models named `Tree`, `Rock`, and `BerryBush` (set a `PrimaryPart` on each).
- Create an empty **Folder** named `Shelters`.
- Create a **Part** named `Campfire` with a `Fire` instance inside (`Enabled = true`).

### 2. ReplicatedStorage
- Add a **RemoteEvent** named `BuildEvent`.
- Add a **RemoteEvent** named `GatherEvent`.

### 3. StarterGui
- Create a **ScreenGui** named `SurvivalHUD` containing:
  - `TextLabel`s: `DayLabel`, `WarmthLabel`, `HungerLabel`, `InventoryLabel`
  - `TextButton`: `BuildWallButton`

### 4. Scripts
- Place `src/server/SurvivalManager.server.luau` inside **ServerScriptService**.
- Place `src/client/SurvivalClient.client.luau` inside **StarterPlayer > StarterPlayerScripts**.

## 🎮 Controls
- **Left Click:** Harvest trees, rocks, or berry bushes
- **`B` Key:** Build a wooden wall (Costs 5 Wood)
- **`R` Key:** Build a wooden roof (Costs 8 Wood)
