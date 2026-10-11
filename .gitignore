
--[[
Banana Cat Hub - Delta compatibility and diagnostics loader
Version: 1.2 (2026-10-11) — runtime guards, settings recovery and diagnostics

This loader downloads the original script supplied by the user, applies a
limited set of compatibility fixes, saves the complete patched source when
Delta supports file writing, and then launches it.

Keeps the original menu; changes quest, farm and Trial runtime handling.
Other original features and external dependencies are not live-verified.
It is NOT a replacement menu or a verified bypass for game-side restrictions.
This code has NOT been tested in a live Blox Fruits/Delta session.

Never run untrusted scripts without inspecting the original source.
]]

local SOURCE_URL = "https://raw.githubusercontent.com/ptlhub2026/ptlhub/refs/heads/main/3nn_main.lua.txt"
local SAVE_FOLDER = "Banana Cat Hub"
local SAVE_FILE = SAVE_FOLDER .. "/BananaCatHub_Delta_Fixed_Full.lua"
local LOG_FILE = SAVE_FOLDER .. "/Delta_Fix_Errors.txt"
local PREFIX = "[BananaCat Delta Fix] "

local function info(message)
    print(PREFIX .. tostring(message))
end

local function problem(message)
    warn(PREFIX .. tostring(message))
end

if type(getgenv) ~= "function" then
    problem("Delta/getgenv() is unavailable; cannot start.")
    return
end

local env = getgenv()
if env.__BC_DELTA_PATCH_RUNNING then
    problem("The Delta patch loader is already running. Do not execute it twice.")
    return
end

-- The original UI and workers cannot be safely recreated in the same session.
if env.__BF_LOADED or env.LoadScript then
    problem("A Banana Cat Hub session is already loaded. Leave and rejoin Blox Fruits, then use ONLY this file once.")
    return
end

env.__BC_DELTA_PATCH_RUNNING = true
local function stop(message)
    env.__BC_DELTA_PATCH_RUNNING = false
    problem(message)
end

if type(loadstring) ~= "function" then
    stop("Delta does not expose loadstring().")
    return
end

if not game:IsLoaded() then
    game.Loaded:Wait()
end

local players = game:GetService("Players")
local deadline = os.clock() + 25
while not players.LocalPlayer and os.clock() < deadline do
    task.wait(0.1)
end
if not players.LocalPlayer then
    stop("LocalPlayer is not ready.")
    return
end

-- Download as TEXT ONLY. We patch it before compilation/execution.
local downloaded, original = pcall(function()
    local ok, body = pcall(function() return game:HttpGet(SOURCE_URL, true) end)
    if ok and type(body)=="string" then return body end
    local req = (type(request)=="function" and request) or (type(http_request)=="function" and http_request)
      or (type(syn)=="table" and syn.request) or (type(http)=="table" and http.request)
    if type(req)~="function" then error(tostring(body)) end
    local response = req({Url=SOURCE_URL,Method="GET"})
    if type(response)~="table" or tonumber(response.StatusCode)~=200 or type(response.Body)~="string" then
     error("HTTP download failed")
    end
    return response.Body
end)
if not downloaded or type(original) ~= "string" then
    stop("Failed to download original source: " .. tostring(original))
    return
end
if #original < 100000
    or not original:find("function toTarget(", 1, true)
    or not original:find("function AttackAOE(", 1, true)
    or not original:find("Settings = ReadSetting()", 1, true)
then
    stop("The GitHub source no longer matches the inspected Banana Cat Hub file. Stopping rather than patching unrelated code.")
    return
end

local source = original:gsub("\r\n", "\n")
local counts = {}
local function escapePattern(value)
    return (value:gsub("([^%w])", "%%%1"))
end
local function replaceLiteral(name, oldText, newText)
    local amount
    source, amount = source:gsub(escapePattern(oldText), function()
        return newText
    end)
    counts[name] = amount
    info(name .. " -> " .. tostring(amount) .. " replacement(s)")
    return amount
end

if replaceLiteral("Settings IO recovery", [====[function SaveSettings(b, t, A)
	if A ~= nil then
		Settings[b] = Settings[b] or {}
		Settings[b][t] = A
	elseif b ~= nil then
		Settings[b] = t
	end
	if not isfolder(FolderName) then
		makefolder(FolderName)
	end
	writefile(FolderName .. "/" .. SaveFileName, HttpService:JSONEncode(Settings))
end
if getgenv().Config then
	Settings = getgenv().Config
	SaveSettings()
end
function ReadSetting()
	local b, t = pcall(function()
		if not isfolder(FolderName) then
			makefolder(FolderName)
		end
		return HttpService:JSONDecode(readfile(FolderName .. "/" .. SaveFileName))
	end)
	if b then
		return t
	else
		SaveSettings()
		return ReadSetting()
	end
end
]====], [====[function SaveSettings(key, value, member)
 if type(Settings) ~= "table" then Settings = {} end
 if member ~= nil then
  if type(Settings[key]) ~= "table" then Settings[key] = {} end
  Settings[key][value] = member
 elseif key ~= nil then Settings[key] = value end
 if type(isfolder)~="function" or type(makefolder)~="function" or type(writefile)~="function" then return end
 local ok, err = pcall(function()
  if not isfolder(FolderName) then makefolder(FolderName) end
  writefile(FolderName .. "/" .. SaveFileName, HttpService:JSONEncode(Settings))
 end)
 if not ok and not getgenv().__BC_SETTINGS_WARNED then
  getgenv().__BC_SETTINGS_WARNED=true
  warn("[BananaCat] Settings kept in memory; save failed: " .. tostring(err))
 end
end
function ReadSetting()
 if type(getgenv().Config)=="table" then return getgenv().Config end
 if type(readfile)=="function" then
  local ok, decoded = pcall(function()
   return HttpService:JSONDecode(readfile(FolderName .. "/" .. SaveFileName))
  end)
  if ok and type(decoded)=="table" then return decoded end
 end
 return type(Settings)=="table" and Settings or {}
end
]====]) ~= 1 then
 stop("Source mismatch: Settings IO recovery") return
end

if replaceLiteral("Team selection readiness", [====[repeat
	wait()
until b:FindFirstChild("ChooseTeam")
repeat
	task.wait()
	pcall(function()
		if Settings["Select Team"] == "Pirate" then
			FireButton(
				game:GetService("Players").LocalPlayer.PlayerGui["Main (minimal)"].ChooseTeam.Container.Pirates.Frame.TextButton
			)
			wait(1)
		else
			FireButton(
				game:GetService("Players").LocalPlayer.PlayerGui["Main (minimal)"].ChooseTeam.Container.Marines.Frame.TextButton
			)
			wait(1)
		end
	end)
until game:GetService("Players").LocalPlayer.PlayerGui:FindFirstChild("Main (minimal)")
		and (game:GetService("Players").LocalPlayer.PlayerGui["Main (minimal)"]:FindFirstChild("ChooseTeam"))
		and not game:GetService("Players").LocalPlayer.PlayerGui["Main (minimal)"]:WaitForChild("ChooseTeam").Visible
	or game:GetService("Players").LocalPlayer.PlayerGui:FindFirstChild("Main")
		and (game:GetService("Players").LocalPlayer.PlayerGui.Main:FindFirstChild("ChooseTeam"))
		and not game:GetService("Players").LocalPlayer.PlayerGui.Main:WaitForChild("ChooseTeam").Visible
]====], [====[local teamDeadline = os.clock()+45
while game.Players.LocalPlayer.Team == nil and os.clock()<teamDeadline do
 local gui = game.Players.LocalPlayer:FindFirstChild("PlayerGui")
 for _, name in ipairs({"Main (minimal)","Main"}) do
  local main = gui and gui:FindFirstChild(name)
  local chooser = main and main:FindFirstChild("ChooseTeam")
  local container = chooser and chooser:FindFirstChild("Container")
  local team = container and container:FindFirstChild(Settings["Select Team"]=="Marine" and "Marines" or "Pirates")
  local frame = team and team:FindFirstChild("Frame")
  local button = frame and frame:FindFirstChild("TextButton")
  if chooser and chooser.Visible and button then FireButton(button) end
 end
 task.wait(0.5)
end
if game.Players.LocalPlayer.Team == nil then error("Choose a team manually, then rejoin and retry") end
]====]) ~= 1 then
 stop("Source mismatch: Team selection readiness") return
end

if replaceLiteral("Request API selection", [====[getgenv().ExploitReq = syn and syn.request
	or identifyexecutor() == "Fluxus" and request
	or http_request
	or http.request
	or requests
]====], [====[getgenv().ExploitReq = (type(request)=="function" and request)
 or (type(http_request)=="function" and http_request)
 or (type(syn)=="table" and type(syn.request)=="function" and syn.request)
 or (type(http)=="table" and type(http.request)=="function" and http.request)
 or (type(requests)=="function" and requests)
]====]) ~= 1 then
 stop("Source mismatch: Request API selection") return
end

-- 1. A plain-text source should not call an undefined code-protector macro.
replaceLiteral("Protection directive", "LPH_ATTRIBUTES(VM(NONE))", "do end -- BC_DELTA: ignored source protection directive")

-- 2. Fix 'S' (Players service) when calculating hit targets. This was an
--    undeclared alias in the inspected source section.
replaceLiteral("Player hit target lookup", "S:GetPlayerFromCharacter(", 'game:GetService("Players"):GetPlayerFromCharacter(')

-- 3. Roblox service/member names are case-sensitive in Lua.
replaceLiteral("LocalPlayer capitalization", "game.Players.localPlayer", "game.Players.LocalPlayer")

-- 4. Some executors don't define a global 'http'; avoid indexing nil.
replaceLiteral("HTTP request nil guard", "or http.request", "or (http and http.request)")

-- 5. Set missing defaults; preserve the user's saved toggles.
replaceLiteral(
    "Missing farm defaults",
    "Settings = ReadSetting()",
    [[Settings = ReadSetting()
if Settings["Select Method Farm"] == nil then
    SaveSettings("Select Method Farm", "Level Farm")
end
if Settings["Select Weapon"] == nil then
    SaveSettings("Select Weapon", "Melee")
end]]
)


-- Quest and Trial fixes matched against the supplied source.
replaceLiteral("Quest state helpers", [====[function DontQuest()
	return Z.Data and Z.Data.QuestData ~= nil
end
function GetNameDoubleQuest()
	if DontQuest() then
		for C, J in next, Z.Data.QuestData.Task, nil do
			return C
		end
	end
end
]====], [====[
function BCPath(object, ...)
 for _, name in ipairs({...}) do
  if not object then return nil end
  object = object:FindFirstChild(name)
 end
 return object
end
function BCLivingRoot()
 local char = t.Character
 local human = char and char:FindFirstChildOfClass("Humanoid")
 local root = char and char:FindFirstChild("HumanoidRootPart")
 if human and human.Health > 0 and root and root:IsA("BasePart") then return root end
end
function BCTransformed()
 local frame = BCPath(t, "PlayerGui", "TransformationHUD", "ImageLabel")
 return frame and frame:IsA("GuiObject") and frame.Visible or false
end
function BCFarmRunning()
 return Settings["Start Farm"] and StackFarm and not Settings["Auto Trial"] and BCLivingRoot() ~= nil
end
-- A visible quest and its task data can arrive on different frames.
function BCQuestVisible()
 local gui = t:FindFirstChild("PlayerGui")
 if not gui then return false end
 for _, name in ipairs({"Main", "Main (minimal)"}) do
  local main = gui:FindFirstChild(name)
  local frame = main and main:FindFirstChild("Quest")
  if frame and frame:IsA("GuiObject") and frame.Visible then return true end
 end
 return false
end
function DontQuest()
 local data = Z and Z.Data
 local quest = data and data.QuestData
 return type(quest) == "table" and type(quest.Task) == "table" and next(quest.Task) ~= nil
end
function GetNameDoubleQuest()
 if DontQuest() then
  for name, amount in pairs(Z.Data.QuestData.Task) do
   if type(name) == "string" and type(amount) == "number" and amount > 0 then return name end
  end
 end
end
function BCQuestStatus(message)
 local env = getgenv()
 if env.__BC_QUEST_STATUS ~= message or os.clock() - (env.__BC_QUEST_STATUS_AT or 0) > 15 then
  env.__BC_QUEST_STATUS, env.__BC_QUEST_STATUS_AT = message, os.clock()
  warn("[BananaCat Quest] " .. message)
 end
end
]====])
replaceLiteral("Quest point initialization", [====[function CFrameQuest()
	local B = {}
	for C, J in next, H, nil do
		if C ~= "MarineQuest" then
			for F, F in next, J, nil do
				B[F.LevelReq] = C
			end
		end
	end
	getgenv().questpoint = {}
	for C, J in next, Z.Data.NPCList, nil do
		for F, q in next, J.Levels, nil do
			F = B[q]
			if C.Parent.Name ~= "Marine Leader" and F and not getgenv().questpoint[F] then
				getgenv().questpoint[F] = CFrame.new(J.Position)
			end
		end
	end
	getgenv().questpoint.SkyExp1Quest = CFrame.new(-7857.28516, 5544.34033, -382.321503)
end
]====], [====[function CFrameQuest()
 getgenv().questpoint = getgenv().questpoint or {}
 local list = Z and Z.Data and Z.Data.NPCList
 if type(list) == "table" then
  for _, npc in pairs(list) do
   if type(npc) == "table" and type(npc.InternalQuestName) == "string" then
    local pos = npc.Position
    if typeof(pos) == "CFrame" or typeof(pos) == "Vector3" then
     getgenv().questpoint[npc.InternalQuestName] = typeof(pos) == "CFrame" and pos or CFrame.new(pos)
    end
   end
  end
 end
 getgenv().questpoint.SkyExp1Quest = CFrame.new(-7857.28516, 5544.34033, -382.321503)
end
]====])
replaceLiteral("Quest selection", [====[local function B(C)
	local J = Z.Data.QuestData
	local F, q, c = J and (next(J.Task)), 0, {}
	for D, r in pairs(Z.Data.NPCList) do
		if not table.find(V, r.InternalQuestName) then
			for V, n in pairs(r.Levels) do
				D = H[r.InternalQuestName][V]
				if D then
					local u, W = next(D.Task)
					if W and W > 1 and n <= C and n >= q then
						c = {
							Level = n,
							Name = r.NPCName,
							QuestName = r.InternalQuestName,
							Pos = r.Position,
							Id = V,
							Mob = u,
						}
						if F == u and V > 1 then
							J = H[r.InternalQuestName][V - 1]
							if J and J.Task then
								for H, C in pairs(J.Task) do
									if H ~= F and C > 1 then
										c.Mob = H
										c.Id = V - 1
										break
									end
								end
							end
						end
						q = n
					end
				end
			end
		end
	end
	return c
end
]====], [====[local function B(level)
 local list = Z and Z.Data and Z.Data.NPCList
 if type(list) ~= "table" then return nil end
 local best
 for _, npc in pairs(list) do
  if type(npc) == "table" and type(npc.Levels) == "table"
   and not table.find(V, npc.InternalQuestName)
   and (typeof(npc.Position) == "Vector3" or typeof(npc.Position) == "CFrame") then
   local definitions = type(H) == "table" and H[npc.InternalQuestName]
   for id, required in pairs(npc.Levels) do
    local questId = tonumber(id)
    local definition = definitions and (definitions[id] or definitions[questId])
    required = tonumber(required)
    if questId and required and required <= level and type(definition) == "table"
     and type(definition.Task) == "table" and (not best or required > best.Level) then
     for mob, amount in pairs(definition.Task) do
      if type(amount) == "number" and amount > 1 then
       best = {Level=required, QuestName=npc.InternalQuestName, Id=questId, Pos=npc.Position, Mob=mob}
       break
      end
     end
    end
   end
  end
 end
 return best
end
]====])
replaceLiteral("Quest acceptance", [====[TakeQuestLevel = function()
	local V = B(t.Data.Level.Value)
	if not V or not V.Pos then
		return
	end
	local H, B, C =
		typeof(V.Pos) == "CFrame" and V.Pos.Position or V.Pos,
		t.Character and (t.Character:FindFirstChild("HumanoidRootPart")),
		t.Character and (t.Character:FindFirstChild("Humanoid"))
	if not B or not C then
		return
	end
	if (H - B.Position).Magnitude <= 8 and C.Health > 0 then
		wait(2)
		CommF:InvokeServer("StartQuest", tostring(V.QuestName), V.Id)
	else
		toTarget(CFrame.new(H) * CFrame.new(0, 4, 2), true)
	end
end

]====], [====[TakeQuestLevel = function()
 if BCQuestVisible() or DontQuest() then return end
 getgenv().__BC_QUEST_MOB = nil
 local data = t:FindFirstChild("Data")
 local level = data and data:FindFirstChild("Level")
 local quest = level and B(level.Value)
 if not quest then BCQuestStatus("Waiting for quest/NPC data") return end
 local character = t.Character
 local root = character and character:FindFirstChild("HumanoidRootPart")
 local human = character and character:FindFirstChildOfClass("Humanoid")
 if not root or not human or human.Health <= 0 then return end
 local pos = typeof(quest.Pos) == "CFrame" and quest.Pos.Position or quest.Pos
 if (pos-root.Position).Magnitude > 8 then
  toTarget(CFrame.new(pos) * CFrame.new(0,4,2), true)
  return
 end
 local env = getgenv()
 if os.clock() - (env.__BC_QUEST_TRY or -10) < 2 then return end
 env.__BC_QUEST_TRY = os.clock()
 local result = CommF:InvokeServer("StartQuest", quest.QuestName, quest.Id)
 BCQuestStatus("Requested " .. quest.QuestName .. "; waiting for confirmation: " .. tostring(result))
end

]====])
replaceLiteral("Special quest throttle", [====[function QuestBoneAndkatakuri(V, H)
]====], [====[function QuestBoneAndkatakuri(V, H)
 if BCQuestVisible() or DontQuest() then return end
 if os.clock() - (getgenv().__BC_SPECIAL_QUEST_TRY or -10) < 2 then return end
 getgenv().__BC_SPECIAL_QUEST_TRY = os.clock()
]====])
replaceLiteral("Trial entry and lifecycle", [====[function AutoTrialV4()
	if Settings["Auto Finish Train Quest"] and Settings["Stack Train With Trial Race"] and (CheckGoTrain()) then
		return
	end
	local g = game.Lighting.ClockTime
	if
		(CheckMoon() == "Full Moon" and not (g > 5 and g < 12) or CheckMoon() == "Next Night")
		and Settings["Hop Server [Trial Or Pull Lever]"]
	then
		if getgenv().TurnOffHOPSVPullAndTrial then
			getgenv().TurnOffHOPSVPullAndTrial:SetStage(false)
		end
		task.wait(3)
	elseif Settings["Hop Server [Trial Or Pull Lever]"] then
		HopServer()
		return
	end
	g = GetTempleOfTime()
	if not g and not VerifyNearbyTrial() then
		toTarget(CFrame.new(28282.5703125, 14896.8505859375, 105.1042709350586))
		return
	end
	if
		g and (g.FFABorder:FindFirstChild("Forcefield")) and g.FFABorder.Forcefield.Transparency == 1
		or (VerifyNearbyTrial())
	then
		if game:GetService("Players").LocalPlayer.PlayerGui.Main.TopHUDList.RaidTimer.Visible then
			if VerifyNearbyTrial() and not getgenv().VerifyTrial then
				getgenv().VerifyTrial = true
			end
			repeat
				wait()
			until VerifyNearbyTrial()
				or not game:GetService("Players").LocalPlayer.PlayerGui.Main.TopHUDList.RaidTimer.Visible
			local R = game.Players.LocalPlayer.Data.Race.Value
			if R == "Human" then
				repeat
					task.wait()
					local m = TrialHuman()
					if m then
						repeat
							task.wait()
							sizepart(m)
							if Settings["Select Weapon"] == "Blox Fruit" then
								toTarget(m.HumanoidRootPart.CFrame * CFrame.new(-7, 20, 0))
							else
								toTarget(m.HumanoidRootPart.CFrame * CFrame.new(7, 20, 0))
							end
							ClickM1(m)
							UsedualFlock()
						until not IsMobAlive(m)
							or not game:GetService("Players").LocalPlayer.PlayerGui.Main.TopHUDList.RaidTimer.Visible
							or (game.Players.LocalPlayer.Character.HumanoidRootPart.Position - game:GetService(
									"Workspace"
								)._WorldOrigin.Locations["Trial of Strength"].Position).Magnitude
								> 1000
					end
				until not game:GetService("Players").LocalPlayer.PlayerGui.Main.TopHUDList.RaidTimer.Visible
					or (
							game.Players.LocalPlayer.Character.HumanoidRootPart.Position
							- game:GetService("Workspace")._WorldOrigin.Locations["Trial of Strength"].Position
						).Magnitude
						> 1000
			elseif R == "Skypiea" then
				repeat
					task.wait()
					if
						game:GetService("Workspace")._WorldOrigin.Locations["Trial of the King"]
						and (game.Players.LocalPlayer.Character.HumanoidRootPart.Position - game:GetService(
								"Workspace"
							)._WorldOrigin.Locations["Trial of the King"].CFrame.Position).Magnitude
							<= 1000
					then
						toTarget(game:GetService("Workspace").Map.SkyTrial.Model.FinishPart.CFrame)
						task.wait(3)
					end
				until not game:GetService("Players").LocalPlayer.PlayerGui.Main.TopHUDList.RaidTimer.Visible
					or workspace.Map:FindFirstChild("Temple of Time") and workspace.Map["Temple of Time"].FFABorder.Forcefield.Transparency == 0
					or t:DistanceFromCharacter(game:GetService("Workspace").Map.SkyTrial.Model.FinishPart.Position)
						> 1000
			elseif R == "Fishman" then
				local m = if game:GetService("Workspace")._WorldOrigin.Locations:FindFirstChild("Trial of Water")
					then (game:GetService("Workspace")._WorldOrigin.Locations:FindFirstChild("Trial of Water"))
					else nil
				if m and (m.Position - t.Character.HumanoidRootPart.Position).Magnitude < 1500 then
					local m = GetSeaBeastTrial()
					repeat
						task.wait()
						if m then
							local l = m:FindFirstChild("HumanoidRootPart")
							getgenv().AimPos = CFrame.new(l.Position.X, 40, l.Position.Z)
							TeleportSeabeast2(m)
							if t:DistanceFromCharacter(l.Position) < 400 then
								AutoAllSkill()
							end
						end
					until not m
						or not m.Parent
						or m.Health.Value == 0
						or not game:GetService("Players").LocalPlayer.PlayerGui.Main.TopHUDList.RaidTimer.Visible
						or workspace.Map:FindFirstChild("Temple of Time") and workspace.Map["Temple of Time"].FFABorder.Forcefield.Transparency == 0
						or t:DistanceFromCharacter(
								game:GetService("Workspace")._WorldOrigin.Locations
									:FindFirstChild("Trial of Water").Position
							)
							> 1000
				end
			elseif R == "Mink" then
				repeat
					task.wait()
					if
						(
							game.Players.LocalPlayer.Character.HumanoidRootPart.Position
							- game:GetService("Workspace")._WorldOrigin.Locations["Trial of Speed"].Position
						).Magnitude <= 1000
					then
						toTarget(game:GetService("Workspace").StartPoint.CFrame * CFrame.new(0, 2, 0))
					end
				until not game:GetService("Players").LocalPlayer.PlayerGui.Main.TopHUDList.RaidTimer.Visible
					or t:DistanceFromCharacter(
							game:GetService("Workspace")._WorldOrigin.Locations
								:FindFirstChild("Trial of Speed").Position
						)
						> 1000
			elseif R == "Ghoul" then
				repeat
					task.wait()
					local m = TrialGhoul()
					if m then
						repeat
							task.wait()
							sizepart(m)
							if Settings["Select Weapon"] == "Blox Fruit" then
								toTarget(m.HumanoidRootPart.CFrame * CFrame.new(-7, 20, 0))
							else
								toTarget(m.HumanoidRootPart.CFrame * CFrame.new(7, 20, 0))
							end
							UsedualFlock()
							ClickM1(m)
						until not IsMobAlive(m)
							or not game:GetService("Players").LocalPlayer.PlayerGui.Main.TopHUDList.RaidTimer.Visible
							or t:DistanceFromCharacter(
									game:GetService("Workspace")._WorldOrigin.Locations
										:FindFirstChild("Trial of Carnage").Position
								)
								> 1000
					end
				until not game:GetService("Players").LocalPlayer.PlayerGui.Main.TopHUDList.RaidTimer.Visible
					or t:DistanceFromCharacter(
							game:GetService("Workspace")._WorldOrigin.Locations
								:FindFirstChild("Trial of Carnage").Position
						)
						> 1000
			elseif R == "Cyborg" then
				repeat
					task.wait()
					toTarget(CFrame.new(28282.5703125, 14896.8505859375, 105.1042709350586))
				until not game:GetService("Players").LocalPlayer.PlayerGui.Main.TopHUDList.RaidTimer.Visible
			end
		else
			if not g then
				return
			end
			local R = g[t.Data.Race.Value .. "Corridor"].Door.Door.RightDoor.Union
			if t:DistanceFromCharacter(R.Position) > 8 then
				toTarget(R.CFrame)
			end
			if Settings["Multi Trial"] and (CheckMultiTeleDoor()) and t:DistanceFromCharacter(R.Position) <= 8 then
				game:service("VirtualInputManager"):SendKeyEvent(true, "T", false, game)
				task.wait()
				game:service("VirtualInputManager"):SendKeyEvent(false, "T", false, game)
				return
			end
			if Settings["Auto Turn On V3 Near Door"] and (CheckMultiPlayerNearDoor()) then
				game:service("VirtualInputManager"):SendKeyEvent(true, "T", false, game)
				task.wait()
				game:service("VirtualInputManager"):SendKeyEvent(false, "T", false, game)
			end
		end
	elseif getgenv().VerifyTrial then
		if not Settings["Multi Trial"] and not Settings["Auto Reset Character"] then
			Settings["Auto Trial"] = false
			ToggleAutoTrial:SetStage(false)
		end
		getgenv().VerifyTrial = false
	end
end
]====], [====[function BCTrialMessage(message)
 local env = getgenv()
 if env.__BC_TRIAL_STATUS ~= message or os.clock()-(env.__BC_TRIAL_STATUS_AT or 0)>15 then
  env.__BC_TRIAL_STATUS, env.__BC_TRIAL_STATUS_AT = message, os.clock()
  warn("[BananaCat Trial] " .. message)
 end
end
function BCTrialTimer()
 if not Settings["Auto Trial"] or not BCLivingRoot() then return false end
 for _, name in ipairs({"Main", "Main (minimal)"}) do
  local timer = BCPath(t, "PlayerGui", name, "TopHUDList", "RaidTimer")
  if timer and timer:IsA("GuiObject") and timer.Visible then return true end
 end
 return false
end
function BCTrialLocation(name, radius)
 local root = BCLivingRoot()
 local location = BCPath(workspace, "_WorldOrigin", "Locations", name)
 if root and location and location:IsA("BasePart") then
  return (root.Position-location.Position).Magnitude <= (radius or 1000)
 end
 return false
end
function BCNearbyTrial()
 for _, name in ipairs({"Trial of the Machine","Trial of Speed","Trial of Strength","Trial of Water","Trial of the King","Trial of Carnage","Trial of Flames"}) do
  if BCTrialLocation(name, 1500) then return true end
 end
 return false
end
function BCEnterTemple()
 local root = BCLivingRoot()
 if not root then return end
 local env = getgenv()
 if os.clock()-(env.__BC_TRIAL_POLL or -10)>=2 then
  env.__BC_TRIAL_POLL = os.clock()
  env.__BC_TRIAL_PROGRESS = CommF:InvokeServer("RaceV4Progress", "Check")
 end
 if not Settings["Auto Trial"] or not BCLivingRoot() then return end
 local stage = env.__BC_TRIAL_PROGRESS
 local tree = CFrame.new(3032.780029296875,2280.85107421875,-7325.47802734375)
 local temple = Vector3.new(28282.5703125,14896.8505859375,105.1042709350586)
 if stage == 1 then
  if os.clock()-(env.__BC_TRIAL_ACTION or -10)>=3 then
   env.__BC_TRIAL_ACTION=os.clock()
   BCTrialMessage("Starting NPC progress")
   CommF:InvokeServer("RaceV4Progress", "Begin")
  end
  return
 end
 root = BCLivingRoot()
 if not root then return end
 if (root.Position-temple).Magnitude>1500 then
  BCTrialMessage("Moving to the Great Tree NPC")
  if (root.Position-tree.Position).Magnitude>8 then
   toTarget(tree, true)
  elseif os.clock()-(env.__BC_TRIAL_ACTION or -10)>=3 then
   env.__BC_TRIAL_ACTION=os.clock()
   CommF:InvokeServer("RaceV4Progress", "Teleport")
  end
  return
 end
 if stage==3 and os.clock()-(env.__BC_TRIAL_ACTION or -10)>=3 then
  env.__BC_TRIAL_ACTION=os.clock()
  CommF:InvokeServer("RaceV4Progress", "Continue")
 end
end
function AutoTrialV4()
 if not Settings["Auto Trial"] or not BCLivingRoot() then return end
 if Settings["Auto Finish Train Quest"] and Settings["Stack Train With Trial Race"] and CheckGoTrain() then return end
 local nearby = BCNearbyTrial()
 local active = BCTrialTimer() and nearby
 local env = getgenv()
 if active then
  env.VerifyTrial = true
  local race = BCPath(t,"Data","Race")
  race = race and race.Value
  if race == "Human" or race == "Ghoul" then
   local location = race == "Human" and "Trial of Strength" or "Trial of Carnage"
   if not BCTrialLocation(location) then return end
   local mob = race == "Human" and TrialHuman() or nil
   if race == "Ghoul" then mob = TrialGhoul() end
   if not IsMobAlive(mob) or not BCTrialTimer() then return end
   sizepart(mob)
   local part = mob:FindFirstChild("HumanoidRootPart")
   if not part then return end
   toTarget(part.CFrame * CFrame.new(Settings["Select Weapon"] == "Blox Fruit" and -7 or 7,20,0))
   if not BCTrialTimer() or not IsMobAlive(mob) then return end
   ClickM1(mob)
   UsedualFlock()
  elseif race == "Skypiea" then
   local finish = BCPath(workspace,"Map","SkyTrial","Model","FinishPart")
   if BCTrialLocation("Trial of the King") and finish and finish:IsA("BasePart") then toTarget(finish.CFrame) end
  elseif race == "Fishman" then
   if not BCTrialLocation("Trial of Water",1500) then return end
   local mob = GetSeaBeastTrial()
   local part = mob and mob.Parent and mob:FindFirstChild("HumanoidRootPart")
   local health = mob and mob:FindFirstChild("Health")
   if not part or not part:IsA("BasePart") or not health or not (health:IsA("NumberValue") or health:IsA("IntValue")) or health.Value<=0 then return end
   env.AimPos = CFrame.new(part.Position.X,40,part.Position.Z)
   TeleportSeabeast2(mob)
   local root = BCLivingRoot()
   if root and BCTrialTimer() and part.Parent and (root.Position-part.Position).Magnitude<400 then AutoAllSkill() end
  elseif race == "Mink" then
   local start = workspace:FindFirstChild("StartPoint")
   if BCTrialLocation("Trial of Speed") and start and start:IsA("BasePart") then toTarget(start.CFrame*CFrame.new(0,2,0)) end
  elseif race == "Cyborg" then
   if BCTrialLocation("Trial of the Machine",1500) then toTarget(CFrame.new(28282.5703125,14896.8505859375,105.1042709350586)) end
  else
   BCTrialMessage("Race is not handled by this Trial routine: " .. tostring(race))
  end
  return
 end
 if env.VerifyTrial then
  env.VerifyTrial = false
  if not Settings["Multi Trial"] and not Settings["Auto Reset Character"] then
   SaveSettings("Auto Trial",false)
   if ToggleAutoTrial and type(ToggleAutoTrial.SetStage)=="function" then ToggleAutoTrial:SetStage(false) end
   return
  end
 end
 -- Do not server-hop while a trial timer is active or a trial map is nearby.
 if not nearby and not BCTrialTimer() and Settings["Hop Server [Trial Or Pull Lever]"] then
  local hour, moon = game.Lighting.ClockTime, CheckMoon()
  if (moon=="Full Moon" and not (hour>5 and hour<12)) or moon=="Next Night" then
   if env.TurnOffHOPSVPullAndTrial then env.TurnOffHOPSVPullAndTrial:SetStage(false) end
  else HopServer() return end
 end
 if nearby then BCTrialMessage("Waiting for trial timer") return end
 BCEnterTemple()
 local root = BCLivingRoot()
 if not root or not Settings["Auto Trial"] then return end
 if (root.Position-Vector3.new(28282.5703125,14896.8505859375,105.1042709350586)).Magnitude>1500 then return end
 local temple = BCPath(workspace,"Map","Temple of Time")
 if not temple or temple:GetAttribute("ClientBorrowed") then return end
 local field = BCPath(temple,"FFABorder","Forcefield")
 if not field or not field:IsA("BasePart") or field.Transparency~=1 then
  BCTrialMessage("Waiting for temple access/lever conditions") return
 end
 local race = BCPath(t,"Data","Race")
 local door = race and BCPath(temple,tostring(race.Value).."Corridor","Door","Door","RightDoor","Union")
 if not door or not door:IsA("BasePart") then BCTrialMessage("Waiting for race door to load") return end
 if (root.Position-door.Position).Magnitude>8 then toTarget(door.CFrame) return end
 BCTrialMessage("Waiting at race door for trial conditions/other players")
 if os.clock()-(env.__BC_TRIAL_KEY_AT or -10)<2 then return end
 if (Settings["Multi Trial"] and CheckMultiTeleDoor()) or (Settings["Auto Turn On V3 Near Door"] and CheckMultiPlayerNearDoor()) then
  env.__BC_TRIAL_KEY_AT=os.clock()
  local input = game:GetService("VirtualInputManager")
  input:SendKeyEvent(true,Enum.KeyCode.T,false,game)
  task.wait()
  input:SendKeyEvent(false,Enum.KeyCode.T,false,game)
 end
end
]====])
replaceLiteral("Farm state and movement ownership", [====[function FarmMethod()
	local f, V, H = Settings["Select Method Farm"]
	local C, J = 9999, 2
	if f == "Farm Katakuri" then
		C, V, H = 2275, y, "CakeQuest2"
	elseif f == "Farm Bones" then
		C, V, H = 2050, e, "HauntedQuest2"
	elseif f == "Farm Tyrant of the Skies" then
		C, V, H = 2575, Y, "TikiQuest3"
	else
		V = if f == "Aura Farm" and (DetectMobAura()) then { DetectMobAura() } else V
	end
	if Settings["Farm Material"] then
		V = NameMaterials[Settings["Select Material"]]
		if not NameWorldMaterials[Settings["Select Material"]][game.PlaceId] then
			f = NameWorldMaterials[Settings["Select Material"]][getgenv().CheckPlaceId2]
				or NameWorldMaterials[Settings["Select Material"]][getgenv().CheckPlaceId3]
				or NameWorldMaterials[Settings["Select Material"]][getgenv().CheckPlaceId]
			game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(f)
			return
		end
	end
	f = V or (GetNameDoubleQuest()) or ""
	if not t.PlayerGui.Main:FindFirstChild("Quest").Visible and typeof(f) == "string" then
		TakeQuestLevel()
	else
		if
			Settings["Auto Quest [Katakuri/Bone/Tyrant]"]
			and t.Data.Level.Value >= C
			and not t.PlayerGui.Main:FindFirstChild("Quest").Visible
		then
			QuestBoneAndkatakuri(H, J)
			return
		end
		if not Settings["Farm Material"] and Settings["Select Method Farm"] == "Farm Tyrant of the Skies" then
			if CheckNameBoss("Tyrant of the Skies") then
				V = CheckNameBoss("Tyrant of the Skies")
				repeat
					task.wait()
					sizepart(V)
					if
						game:GetService("Players").LocalPlayer.PlayerGui.TransformationHUD.ImageLabel.Visible
						and (Settings["Auto Finish Train Quest"] or Settings["Auto Finish Train Draco Quest"])
					then
						toTarget(V.HumanoidRootPart.CFrame * CFrame.new(7, 20, 0))
					elseif Settings["Select Weapon"] == "Blox Fruit" then
						toTarget(V.HumanoidRootPart.CFrame * CFrame.new(-7, 20, 0))
					else
						toTarget(V.HumanoidRootPart.CFrame * CFrame.new(7, 20, 0))
					end
					UsedualFlock()
					ClickM1(V)
				until not IsMobAlive(V) or not Settings["Start Farm"] or not StackFarm
				return
			else
				local V = workspace:FindFirstChild("Map", true)
					and (workspace.Map:FindFirstChild("TikiOutpost", true))
					and (workspace.Map.TikiOutpost:FindFirstChild("IslandModel", true))
				if V then
					local Y, H, C, J =
						V:FindFirstChild("Eye1", true),
						V:FindFirstChild("Eye2", true),
						V:FindFirstChild("Eye3", true),
						V:FindFirstChild("Eye4", true)
					if
						Y
						and H
						and C
						and J
						and Y.Transparency == 0
						and H.Transparency == 0
						and C.Transparency == 0
						and J.Transparency == 0
					then
						local V = DetectModelDestroyTyrant()
						if V then
							if t:DistanceFromCharacter(V.WorldPivot.Position) > 10 then
								toTarget(V.WorldPivot)
							elseif CheckItemInventory("Skull Guitar") then
								if not NameWeapon("Gun") or NameWeapon("Gun") ~= "Skull Guitar" then
									game:GetService("ReplicatedStorage").Remotes.CommF_
										:InvokeServer(unpack({ [1] = "LoadItem", [2] = "Skull Guitar" }))
								else
									equiptool(NameWeapon("Gun"))
									getgenv().SpamGunSkullGuitar(V.WorldPivot)
								end
							else
								getgenv().AimPos = V.WorldPivot
								AutoAllSkill()
							end
						end
						return
					end
				end
			end
		end
		if
			not Settings["Farm Material"]
			and Settings["Select Method Farm"] == "Farm Katakuri"
			and not Settings["Ignore Attack Katakuri"]
		then
			if CheckNameBoss("Cake Prince") then
				local V = CheckNameBoss("Cake Prince")
				repeat
					task.wait()
					sizepart(V)
					if
						game:GetService("Players").LocalPlayer.PlayerGui.TransformationHUD.ImageLabel.Visible
						and (Settings["Auto Finish Train Quest"] or Settings["Auto Finish Train Draco Quest"])
					then
						toTarget(V.HumanoidRootPart.CFrame * CFrame.new(7, 20, 0))
					elseif Settings["Select Weapon"] == "Blox Fruit" then
						toTarget(V.HumanoidRootPart.CFrame * CFrame.new(-7, 20, 0))
					else
						toTarget(V.HumanoidRootPart.CFrame * CFrame.new(7, 20, 0))
					end
					UsedualFlock()
					ClickM1(V)
				until not IsMobAlive(V) or not Settings["Start Farm"] or not StackFarm
				return
			else
				spawn(function()
					if Settings["Hop Find Katakuri"] then
						SpecialHop("Cake Prince")
					end
				end)
			end
		end
		local V = DetectMob(f)
		if not V then
			if typeof(f) == "table" then
				if #N >= #f then
					N = {}
					return
				end
				local Y = DetectNameTablePart(f)
				local H = DetectPartSpawnMob(Y)
				if H then
					table.insert(N, Y)
					repeat
						task.wait()
						toTarget(H.CFrame * CFrame.new(0, 60, 0))
					until (H.Position - t.Character.HumanoidRootPart.Position).Magnitude <= 100
						or (DetectMob(f))
						or not Settings["Start Farm"]
						or not StackFarm
					wait(1)
				end
			else
				local Y = DetectPartSpawnMob(f, true)
				if Y then
					Instance.new("IntValue", Y).Name = "Ignored"
					repeat
						task.wait()
						toTarget(Y.CFrame * CFrame.new(0, 60, 0))
					until (Y.Position - t.Character.HumanoidRootPart.Position).Magnitude <= 100
						or (DetectMob(f))
						or not Settings["Start Farm"]
						or not StackFarm
					wait(1)
				else
					DeleteIgnoredMobSpawn()
				end
			end
		else
			repeat
				task.wait()
				sizepart(V)
				BringMob(V)
				FarmMastery(V)
				ClickM1(V)
				if
					game:GetService("Players").LocalPlayer.PlayerGui.TransformationHUD.ImageLabel.Visible
					and (Settings["Auto Finish Train Quest"] or Settings["Auto Finish Train Draco Quest"])
				then
					toTarget(V.HumanoidRootPart.CFrame * CFrame.new(7, 20, 0))
				elseif Settings["Select Weapon"] == "Blox Fruit" then
					toTarget(V.HumanoidRootPart.CFrame * CFrame.new(-7, getgenv().YPosFruit, 0))
				else
					toTarget(V.HumanoidRootPart.CFrame * CFrame.new(7, 20, 0))
				end
			until not IsMobAlive(V) or not Settings["Start Farm"] or not StackFarm
			if getgenv().QuestTrainer and getgenv().QuestTrainer.CountKillMob then
				getgenv().QuestTrainer.CountKillMob = getgenv().QuestTrainer.CountKillMob + 1
			end
		end
	end
end
spawn(function()
	while task.wait() do
		local f, f = pcall(function()
			if Settings["Start Farm"] and StackFarm then
				FarmMethod()
			end
		end)
		if f then
			print(f)
		end
	end
end)
]====], [====[function FarmMethod()
 if not BCFarmRunning() then return end
	local f, V, H = Settings["Select Method Farm"]
	local C, J = 9999, 2
	if f == "Farm Katakuri" then
		C, V, H = 2275, y, "CakeQuest2"
	elseif f == "Farm Bones" then
		C, V, H = 2050, e, "HauntedQuest2"
	elseif f == "Farm Tyrant of the Skies" then
		C, V, H = 2575, Y, "TikiQuest3"
	else
		V = if f == "Aura Farm" and (DetectMobAura()) then { DetectMobAura() } else V
	end
	if Settings["Farm Material"] then
		local material = Settings["Select Material"]
        local worlds = NameWorldMaterials and NameWorldMaterials[material]
        V = NameMaterials and NameMaterials[material]
        if type(worlds) ~= "table" or not V then
         BCQuestStatus("Select a valid material before farming") return
        end
		if not NameWorldMaterials[Settings["Select Material"]][game.PlaceId] then
			f = NameWorldMaterials[Settings["Select Material"]][getgenv().CheckPlaceId2]
				or NameWorldMaterials[Settings["Select Material"]][getgenv().CheckPlaceId3]
				or NameWorldMaterials[Settings["Select Material"]][getgenv().CheckPlaceId]
			if f then game:GetService("ReplicatedStorage").Remotes.CommF_:InvokeServer(f) end
			return
		end
	end
 local questVisible = BCQuestVisible() or DontQuest()
 local activeMob = GetNameDoubleQuest()
 getgenv().__BC_QUEST_MOB = nil
 f = V or activeMob or ""
 if questVisible and type(V) == "table" and activeMob and table.find(V, activeMob) then
  f = activeMob
 end
 if questVisible and f == "" then
  BCQuestStatus("Quest visible but Task data is missing; waiting for GuideModule")
  return
 end
 if not questVisible and typeof(f) == "string" then
		TakeQuestLevel()
	else
		if
			Settings["Auto Quest [Katakuri/Bone/Tyrant]"]
			and t.Data.Level.Value >= C
			and not questVisible
		then
			QuestBoneAndkatakuri(H, J)
			return
		end
		if not Settings["Farm Material"] and Settings["Select Method Farm"] == "Farm Tyrant of the Skies" then
			if CheckNameBoss("Tyrant of the Skies") then
				V = CheckNameBoss("Tyrant of the Skies")
				repeat
					task.wait()
					if not BCFarmRunning() or not IsMobAlive(V) then return end
					sizepart(V)
					if
						BCTransformed()
						and (Settings["Auto Finish Train Quest"] or Settings["Auto Finish Train Draco Quest"])
					then
						toTarget(V.HumanoidRootPart.CFrame * CFrame.new(7, 20, 0))
					elseif Settings["Select Weapon"] == "Blox Fruit" then
						toTarget(V.HumanoidRootPart.CFrame * CFrame.new(-7, 20, 0))
					else
						toTarget(V.HumanoidRootPart.CFrame * CFrame.new(7, 20, 0))
					end
					UsedualFlock()
					ClickM1(V)
				until not IsMobAlive(V) or not Settings["Start Farm"] or not StackFarm or Settings["Auto Trial"]
				return
			else
				local V = workspace:FindFirstChild("Map", true)
					and (workspace.Map:FindFirstChild("TikiOutpost", true))
					and (workspace.Map.TikiOutpost:FindFirstChild("IslandModel", true))
				if V then
					local Y, H, C, J =
						V:FindFirstChild("Eye1", true),
						V:FindFirstChild("Eye2", true),
						V:FindFirstChild("Eye3", true),
						V:FindFirstChild("Eye4", true)
					if
						Y
						and H
						and C
						and J
						and Y.Transparency == 0
						and H.Transparency == 0
						and C.Transparency == 0
						and J.Transparency == 0
					then
						local V = DetectModelDestroyTyrant()
						if V then
							if t:DistanceFromCharacter(V.WorldPivot.Position) > 10 then
								toTarget(V.WorldPivot)
							elseif CheckItemInventory("Skull Guitar") then
								if not NameWeapon("Gun") or NameWeapon("Gun") ~= "Skull Guitar" then
									game:GetService("ReplicatedStorage").Remotes.CommF_
										:InvokeServer(unpack({ [1] = "LoadItem", [2] = "Skull Guitar" }))
								else
									equiptool(NameWeapon("Gun"))
									getgenv().SpamGunSkullGuitar(V.WorldPivot)
								end
							else
								getgenv().AimPos = V.WorldPivot
								AutoAllSkill()
							end
						end
						return
					end
				end
			end
		end
		if
			not Settings["Farm Material"]
			and Settings["Select Method Farm"] == "Farm Katakuri"
			and not Settings["Ignore Attack Katakuri"]
		then
			if CheckNameBoss("Cake Prince") then
				local V = CheckNameBoss("Cake Prince")
				repeat
					task.wait()
					if not BCFarmRunning() or not IsMobAlive(V) then return end
					sizepart(V)
					if
						BCTransformed()
						and (Settings["Auto Finish Train Quest"] or Settings["Auto Finish Train Draco Quest"])
					then
						toTarget(V.HumanoidRootPart.CFrame * CFrame.new(7, 20, 0))
					elseif Settings["Select Weapon"] == "Blox Fruit" then
						toTarget(V.HumanoidRootPart.CFrame * CFrame.new(-7, 20, 0))
					else
						toTarget(V.HumanoidRootPart.CFrame * CFrame.new(7, 20, 0))
					end
					UsedualFlock()
					ClickM1(V)
				until not IsMobAlive(V) or not Settings["Start Farm"] or not StackFarm or Settings["Auto Trial"]
				return
			else
				spawn(function()
					if Settings["Hop Find Katakuri"] then
						SpecialHop("Cake Prince")
					end
				end)
			end
		end
		local V = DetectMob(f)
		if not V then
			if typeof(f) == "table" then
				if #N >= #f then
					N = {}
					return
				end
				local Y = DetectNameTablePart(f)
				local H = Y and DetectPartSpawnMob(Y)
				if H and Y then
					table.insert(N, Y)
					repeat
						task.wait()
						if not BCFarmRunning() or not H.Parent then return end
						toTarget(H.CFrame * CFrame.new(0, 60, 0))
					until (H.Position - t.Character.HumanoidRootPart.Position).Magnitude <= 100
						or (DetectMob(f))
						or not Settings["Start Farm"]
						or not StackFarm or Settings["Auto Trial"]
					wait(1)
				end
			else
				local Y = DetectPartSpawnMob(f, true)
				if Y then
					Instance.new("IntValue", Y).Name = "Ignored"
					repeat
						task.wait()
						if not BCFarmRunning() or not Y.Parent then return end
						toTarget(Y.CFrame * CFrame.new(0, 60, 0))
					until (Y.Position - t.Character.HumanoidRootPart.Position).Magnitude <= 100
						or (DetectMob(f))
						or not Settings["Start Farm"]
						or not StackFarm or Settings["Auto Trial"]
					wait(1)
				else
					DeleteIgnoredMobSpawn()
				end
			end
		else
			repeat
				task.wait()
				if not BCFarmRunning() or not IsMobAlive(V) then return end
				sizepart(V)
				BringMob(V)
				FarmMastery(V)
				ClickM1(V)
				if
					BCTransformed()
					and (Settings["Auto Finish Train Quest"] or Settings["Auto Finish Train Draco Quest"])
				then
					toTarget(V.HumanoidRootPart.CFrame * CFrame.new(7, 20, 0))
				elseif Settings["Select Weapon"] == "Blox Fruit" then
					toTarget(V.HumanoidRootPart.CFrame * CFrame.new(-7, getgenv().YPosFruit, 0))
				else
					toTarget(V.HumanoidRootPart.CFrame * CFrame.new(7, 20, 0))
				end
			until not IsMobAlive(V) or not Settings["Start Farm"] or not StackFarm or Settings["Auto Trial"]
			if getgenv().QuestTrainer and getgenv().QuestTrainer.CountKillMob then
				getgenv().QuestTrainer.CountKillMob = getgenv().QuestTrainer.CountKillMob + 1
			end
		end
	end
end
spawn(function()
	while task.wait() do
		local ok, err = pcall(function()
			if Settings["Start Farm"] and StackFarm then
				FarmMethod()
			end
		end)
		if not ok then
            BCQuestStatus("Farm error: " .. tostring(err))
		end
	end
end)
]====])
for _, name in ipairs({"Quest state helpers","Quest point initialization","Quest selection","Quest acceptance","Special quest throttle","Trial entry and lifecycle","Farm state and movement ownership"}) do
 if counts[name] ~= 1 then
  stop("Source version mismatch for " .. name .. "; expected one match")
  return
 end
end

-- 6. Many loops hide errors inside pcall. The scoped wrapper preserves the
--    original return values while reporting the FIRST errors in the console.
--    Limit the rate and the total count so RenderStepped does not spam Delta.
local instrument = [==[
-- BEGIN Banana Cat Hub Delta diagnostics
local Workspace = game:GetService("Workspace")
local __BC_NATIVE_PCALL = pcall
local __BC_SEEN_ERRORS = {}
local __BC_ERROR_COUNT = 0
local function pcall(...)
    local results = table.pack(__BC_NATIVE_PCALL(...))
    if results[1] == false then
        local message = tostring(results[2])
        local now = os.clock()
        local previous = __BC_SEEN_ERRORS[message]
        if __BC_ERROR_COUNT < 30 and (not previous or now - previous > 20) then
            __BC_SEEN_ERRORS[message] = now
            __BC_ERROR_COUNT = __BC_ERROR_COUNT + 1
            warn("[BananaCat Delta Fix][caught error] " .. message)
            local shared = getgenv()
            shared.__BC_DELTA_ERRORS = shared.__BC_DELTA_ERRORS or {}
            table.insert(shared.__BC_DELTA_ERRORS, message)
            if type(appendfile) == "function" then
                __BC_NATIVE_PCALL(function()
                    appendfile("Banana Cat Hub/Delta_Fix_Errors.txt", message .. "\n")
                end)
            end
        end
    end
    return table.unpack(results, 1, results.n)
end
-- END Banana Cat Hub Delta diagnostics
]==]
source = instrument .. "\n" .. source

-- Structural verification: at least these two critical areas must still be present.
if (counts["Protection directive"] or 0) == 0
    or (counts["Player hit target lookup"] or 0) == 0
    or (counts["Missing farm defaults"] or 0) ~= 1
then
    stop("Critical patch markers not found; refusing to execute a mismatched version.")
    return
end

local compiled, compiler, compileError = pcall(loadstring, source)
if not compiled then
 stop("Compiler raised an error: " .. tostring(compiler)) return
end
if not compiler then
    stop("Luau compile failed: " .. tostring(compileError))
    return
end

-- Save the COMPLETE modified (~19k lines) code to Delta's local script folder.
local folderAvailable = false
if type(isfolder) == "function" and type(makefolder) == "function" then
    local ok, message = pcall(function()
        if not isfolder(SAVE_FOLDER) then
            makefolder(SAVE_FOLDER)
        end
    end)
    folderAvailable = ok
    if not ok then
        problem("Couldn't create output directory: " .. tostring(message))
    end
end
if folderAvailable and type(writefile) == "function" then
    local saved, why = pcall(function()
        writefile(SAVE_FILE, source)
        writefile(LOG_FILE, "Banana Cat Hub Delta Fix log (2026-10-11)\n")
    end)
    if saved then
        info("The full fixed Lua source was saved to: " .. SAVE_FILE)
    else
        problem("The patched script is in memory; could not write it: " .. tostring(why))
    end
else
    problem("File APIs are not available. Running patched source from memory.")
end

if type(debug) ~= "table"
    or type(debug.getupvalues) ~= "function"
    or type(debug.getupvalue) ~= "function"
then
    problem("Some debug.getupvalue(s) APIs are missing in this Delta build. Some features may still fail.")
end

if not (workspace:FindFirstChild("Enemies") and game:GetService("ReplicatedStorage"):FindFirstChild("Remotes")) then
    problem("Enemies/Remotes was not found at startup; the game may still be loading or may have changed.")
end

info("Launching original Banana Cat Hub with source-matched patches (live testing required).")
local success, executionError = pcall(compiler)
-- Keep session lock: spawned workers may survive both success and failure.
if not success then
    problem("Main script stopped: " .. tostring(executionError))
    problem("If the menu already appeared, rejoin before another attempt.")
    return
end

info("Loader returned. Open Delta Console and check for '[caught error]' if a feature fails.")
info("Full patched script: " .. SAVE_FILE)
