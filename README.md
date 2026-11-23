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
- local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Lighting = game:GetService("Lighting")

local BuildEvent = ReplicatedStorage:WaitForChild("BuildEvent")
local GatherEvent = ReplicatedStorage:WaitForChild("GatherEvent")

local SheltersFolder = workspace:WaitForChild("Shelters")
local Campfire = workspace:WaitForChild("Campfire")

-- Game Settings
local DAY_LENGTH = 90
local MAX_DAYS = 7
local currentDay = 1
local isStormActive = false

-- Storm Intensity per Day (1 to 7)
local StormData = {
	[1] = {Name = "Light Snowfall", ColdDrain = 0.4, FogEnd = 500},
	[2] = {Name = "Chilly Winds", ColdDrain = 0.7, FogEnd = 400},
	[3] = {Name = "Frostbite Gusts", ColdDrain = 1.1, FogEnd = 300},
	[4] = {Name = "Heavy Blizzard", ColdDrain = 1.6, FogEnd = 200},
	[5] = {Name = "Whiteout Gale", ColdDrain = 2.2, FogEnd = 120},
	[6] = {Name = "Sub-Zero Tempest", ColdDrain = 3.0, FogEnd = 75},
	[7] = {Name = "THE FINAL FREEZE", ColdDrain = 4.2, FogEnd = 40},
}

-- Setup Player Data
Players.PlayerAdded:Connect(function(player)
	local stats = Instance.new("Folder")
	stats.Name = "SurvivalStats"
	stats.Parent = player

	local warmth = Instance.new("NumberValue", stats)
	warmth.Name = "Warmth"
	warmth.Value = 100

	local hunger = Instance.new("NumberValue", stats)
	hunger.Name = "Hunger"
	hunger.Value = 100

	local wood = Instance.new("IntValue", stats)
	wood.Name = "Wood"
	wood.Value = 5

	local stone = Instance.new("IntValue", stats)
	stone.Name = "Stone"
	stone.Value = 0

	local berries = Instance.new("IntValue", stats)
	berries.Name = "Berries"
	berries.Value = 2
end)

-- Heat Bonus Calculation (Campfire proximity + Overhead roof check)
local function getHeatBonus(character)
	local root = character:FindFirstChild("HumanoidRootPart")
	if not root then return 0 end

	local bonus = 0
	if (root.Position - Campfire.Position).Magnitude <= 18 then
		bonus += 3.5
	end

	local rayParams = RaycastParams.new()
	rayParams.FilterDescendantsInstances = {SheltersFolder}
	rayParams.FilterType = Enum.RaycastFilterType.Include

	local roofHit = workspace:Raycast(root.Position, Vector3.new(0, 15, 0), rayParams)
	if roofHit then
		bonus += 2.0
	end

	return bonus
end

-- Gathering Resources
GatherEvent.OnServerEvent:Connect(function(player, targetModel)
	if not targetModel or not targetModel.Parent then return end
	local character = player.Character
	local root = character and character:FindFirstChild("HumanoidRootPart")
	if not root or not targetModel.PrimaryPart then return end

	if (root.Position - targetModel.PrimaryPart.Position).Magnitude > 15 then return end

	local stats = player:FindFirstChild("SurvivalStats")
	if not stats then return end

	if targetModel.Name == "Tree" then
		stats.Wood.Value += 3
	elseif targetModel.Name == "Rock" then
		stats.Stone.Value += 2
	elseif targetModel.Name == "BerryBush" then
		stats.Hunger.Value = math.min(100, stats.Hunger.Value + 20)
	end

	local parent = targetModel.Parent
	targetModel.Parent = nil
	task.delay(25, function()
		targetModel.Parent = parent
	end)
end)

-- Building Shelter Pieces
BuildEvent.OnServerEvent:Connect(function(player, buildType, buildCFrame)
	local stats = player:FindFirstChild("SurvivalStats")
	if not stats then return end

	if buildType == "Wall" and stats.Wood.Value >= 5 then
		stats.Wood.Value -= 5
		local wall = Instance.new("Part")
		wall.Name = "WoodenWall"
		wall.Size = Vector3.new(10, 8, 1)
		wall.CFrame = buildCFrame
		wall.Anchored = true
		wall.Material = Enum.Material.WoodPlanks
		wall.BrickColor = BrickColor.new("Reddish brown")
		wall.Parent = SheltersFolder
	elseif buildType == "Roof" and stats.Wood.Value >= 8 then
		stats.Wood.Value -= 8
		local roof = Instance.new("Part")
		roof.Name = "WoodenRoof"
		roof.Size = Vector3.new(12, 1, 12)
		roof.CFrame = buildCFrame
		roof.Anchored = true
		roof.Material = Enum.Material.Wood
		roof.BrickColor = BrickColor.new("Dark oak")
		roof.Parent = SheltersFolder
	end
end)

-- Main Survival Drain Loop
task.spawn(function()
	while true do
		task.wait(1)
		local currentStorm = StormData[currentDay] or StormData[7]
		local drainMultiplier = isStormActive and 1.8 or 1.0

		for _, player in ipairs(Players:GetPlayers()) do
			local stats = player:FindFirstChild("SurvivalStats")
			local char = player.Character
			local humanoid = char and char:FindFirstChild("Humanoid")

			if stats and humanoid and humanoid.Health > 0 then
				stats.Hunger.Value = math.max(0, stats.Hunger.Value - 0.5)
				if stats.Hunger.Value <= 0 then
					humanoid:TakeDamage(2)
				end

				local heatBonus = getHeatBonus(char)
				local netWarmthChange = heatBonus - (currentStorm.ColdDrain * drainMultiplier)
				stats.Warmth.Value = math.clamp(stats.Warmth.Value + netWarmthChange, 0, 100)

				if stats.Warmth.Value <= 0 then
					humanoid:TakeDamage(4)
				end
			end
		end
	end
end)

-- Day & Storm Progression Cycle
task.spawn(function()
	local gameStatus = Instance.new("StringValue", ReplicatedStorage)
	gameStatus.Name = "GameStatus"

	while currentDay <= MAX_DAYS do
		local storm = StormData[currentDay]

		isStormActive = false
		Lighting.ClockTime = 12
		Lighting.FogEnd = storm.FogEnd * 1.5
		gameStatus.Value = "Day " .. currentDay .. " - " .. storm.Name .. " (Gather Resources!)"
		task.wait(DAY_LENGTH * 0.5)

		isStormActive = true
		Lighting.ClockTime = 0
		Lighting.FogEnd = storm.FogEnd
		gameStatus.Value = "Night " .. currentDay .. " - BLIZZARD SURGE! GET TO SHELTER!"
		task.wait(DAY_LENGTH * 0.5)

		currentDay += 1
	end

	gameStatus.Value = "YOU SURVIVED ALL 7 DAYS OF WINTER!"
end)

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local mouse = player:GetMouse()

local BuildEvent = ReplicatedStorage:WaitForChild("BuildEvent")
local GatherEvent = ReplicatedStorage:WaitForChild("GatherEvent")
local GameStatus = ReplicatedStorage:WaitForChild("GameStatus")

local stats = player:WaitForChild("SurvivalStats")
local hud = player:WaitForChild("PlayerGui"):WaitForChild("SurvivalHUD")

-- Update UI Loop
task.spawn(function()
	while task.wait(0.2) do
		hud.DayLabel.Text = GameStatus.Value
		hud.WarmthLabel.Text = "Warmth: " .. math.floor(stats.Warmth.Value) .. "%"
		hud.HungerLabel.Text = "Hunger: " .. math.floor(stats.Hunger.Value) .. "%"
		hud.InventoryLabel.Text = "Wood: " .. stats.Wood.Value .. " | Stone: " .. stats.Stone.Value
	end
end)

-- Click to Gather Resources
mouse.Button1Down:Connect(function()
	local target = mouse.Target
	if target and target.Parent and target.Parent.Parent == workspace.Resources then
		GatherEvent:FireServer(target.Parent)
	end
end)

-- Keyboard Building Controls
UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then return end
	local char = player.Character
	local root = char and char:FindFirstChild("HumanoidRootPart")
	if not root then return end

	if input.KeyCode == Enum.KeyCode.B then
		BuildEvent:FireServer("Wall", root.CFrame * CFrame.new(0, 1, -7))
	elseif input.KeyCode == Enum.KeyCode.R then
		BuildEvent:FireServer("Roof", root.CFrame * CFrame.new(0, 6, 0))
	end
end)

-- UI Button Support
hud.BuildWallButton.MouseButton1Click:Connect(function()
	local char = player.Character
	local root = char and char:FindFirstChild("HumanoidRootPart")
	if root then
		BuildEvent:FireServer("Wall", root.CFrame * CFrame.new(0, 1, -7))
	end
end)


