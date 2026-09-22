# payswitch-
--====================================================
-- 🌴 100 NIGHTS SURVIVAL CORE SERVER SYSTEM
-- ServerScriptService
--====================================================

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local DataStoreService = game:GetService("DataStoreService")
local Lighting = game:GetService("Lighting")

local SaveData = DataStoreService:GetDataStore("SURVIVAL_SAVE_V1")

----------------------------------------------------
-- REMOTES (AUTO CREATE)
----------------------------------------------------

local function GetRemote(name)
	local r = ReplicatedStorage:FindFirstChild(name)
	if not r then
		r = Instance.new("RemoteEvent")
		r.Name = name
		r.Parent = ReplicatedStorage
	end
	return r
end

local EquipClass = GetRemote("EquipClass")
local UpdateGUI = GetRemote("UpdateGUI")
local QuestEvent = GetRemote("QuestEvent")
local DailyReward = GetRemote("DailyReward")

----------------------------------------------------
-- GAME STATE (ENDLESS MODE)
----------------------------------------------------

local Night = 1
local EndlessMode = true
local Difficulty = 1

----------------------------------------------------
-- XP SYSTEM
----------------------------------------------------

local function addXP(player, amount)
	local stats = player:FindFirstChild("leaderstats")
	if not stats then return end

	local xp = stats:FindFirstChild("XP")
	local level = stats:FindFirstChild("Level")

	if xp and level then
		xp.Value += amount

		if xp.Value >= level.Value * 100 then
			xp.Value = 0
			level.Value += 1
			player:LoadCharacter()
		end
	end
end

----------------------------------------------------
-- DAILY REWARD (SIMPLE)
----------------------------------------------------

DailyReward.OnServerEvent:Connect(function(player)
	local stats = player:FindFirstChild("leaderstats")
	if stats then
		stats.Diamonds.Value += 50
	end
end)

----------------------------------------------------
-- QUEST SYSTEM (BASIC FRAMEWORK)
----------------------------------------------------

local Quests = {}

local function giveQuest(player, questName)
	Quests[player] = questName
	UpdateGUI:FireClient(player, "Quest", questName)
end

----------------------------------------------------
-- NIGHT SYSTEM (ENDLESS)
----------------------------------------------------

task.spawn(function()
	while true do
		task.wait(10)

		Night += 1
		Difficulty += 0.1

		Lighting.ClockTime = 18

		UpdateGUI:FireAllClients(Night, "Night")

		-- Boss every 10 nights
		if Night % 10 == 0 then
			print("BOSS SPAWN EVENT")
			UpdateGUI:FireAllClients("Boss", "Spawn")
		end

		-- Strong raid
		if Night % 3 == 0 then
			print("RAID EVENT")
		end
	end
end)

----------------------------------------------------
-- PLAYER SETUP
----------------------------------------------------

Players.PlayerAdded:Connect(function(player)

	local stats = Instance.new("Folder")
	stats.Name = "leaderstats"
	stats.Parent = player

	local Diamonds = Instance.new("IntValue")
	Diamonds.Name = "Diamonds"
	Diamonds.Value = 0
	Diamonds.Parent = stats

	local XP = Instance.new("IntValue")
	XP.Name = "XP"
	XP.Value = 0
	XP.Parent = stats

	local Level = Instance.new("IntValue")
	Level.Name = "Level"
	Level.Value = 1
	Level.Parent = stats

	-- load data
	local success, data = pcall(function()
		return SaveData:GetAsync(player.UserId)
	end)

	if success and data then
		Diamonds.Value = data.Diamonds or 0
		XP.Value = data.XP or 0
		Level.Value = data.Level or 1
	end

	giveQuest(player, "Survive Night 1")

end)

----------------------------------------------------
-- SAVE DATA
----------------------------------------------------

Players.PlayerRemoving:Connect(function(player)
	local stats = player:FindFirstChild("leaderstats")
	if not stats then return end

	pcall(function()
		SaveData:SetAsync(player.UserId, {
			Diamonds = stats.Diamonds.Value,
			XP = stats.XP.Value,
			Level = stats.Level.Value
		})
	end)
end)

----------------------------------------------------
-- CLASS SYSTEM HOOK
----------------------------------------------------

EquipClass.OnServerEvent:Connect(function(player, className)
	print(player.Name .. " selected class: " .. className)
	addXP(player, 25)
end)

print("🌴 SURVIVAL CORE SERVER LOADED")
