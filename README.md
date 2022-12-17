if Hwid[table.find(Hwid,game:GetService("RbxAnalyticsService"):GetClientId())] == game:GetService("RbxAnalyticsService"):GetClientId() then
--script
print("Script!!")
else
setclipboard(game:GetService("RbxAnalyticsService"):GetClientId())
game.Players.LocalPlayer:kick("หน้าหีจังเนอะ hwid ไม่ถูกเสือกรันสคริป")
end
print("Wait 6 seconds after this!!")
wait(6)
print("Start!!")
local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/xHeptc/Kavo-UI-Library/main/source.lua"))()
local Window = Library.CreateLib("Aimbot X KORN", "DarkTheme")
local Tab = Window:NewTab("Main")
local Section = Tab:NewSection("Aimbot!!")
Section:NewToggle("Aimbot key F", "", function(state)
    if state then
pcall(function()
	getgenv().Aimbot.Functions:Exit()
end)

--// Environment

getgenv().Aimbot = {}
local Environment = getgenv().Aimbot

--// Services

local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local HttpService = game:GetService("HttpService")
local TweenService = game:GetService("TweenService")
local StarterGui = game:GetService("StarterGui")
local Players = game:GetService("Players")
local Camera = game:GetService("Workspace").CurrentCamera

--// Variables

local LocalPlayer = Players.LocalPlayer
local Title = "KORN X HUB"
local FileNames = {"Aimbot", "Configuration.json", "Drawing.json"}
local RequiredDistance = math.huge
local Typing = false
local Running = false
local Animation = nil
local ServiceConnections = {RenderSteppedConnection = nil, InputBeganConnection = nil, InputEndedConnection = nil, TypingStartedConnection = nil, TypingEndedConnection = nil}

--// Support Functions

local mousemoverel = mousemoverel or (Input and Input.MouseMove)
local queueonteleport = syn.queue_on_teleport or queue_on_teleport

--// Script Settings

Environment.Settings = {
	SendNotifications = true,
    SaveSettings = true, -- Re-execute upon changing
    ReloadOnTeleport = true,
    Enabled = true,
    TeamCheck = false,
    AliveCheck = true,
    WallCheck = false, -- Laggy
    Sensitivity = 0, -- Animation length (in seconds) before fully locking onto target
    ThirdPerson = false, -- Uses mousemoverel instead of CFrame to support locking in third person (could be choppy)
    ThirdPersonSensitivity = 3, -- Boundary: 0.1 - 5
    TriggerKey = "F",
    Toggle = false,
    LockPart = "Head"   -- Body part to lock on
}

Environment.FOVSettings = {
	Enabled = true,
    Visible = true,
    Amount = 90,
    Color = "255, 255, 255",
    LockedColor = "255, 70, 70",
    Transparency = 0.5,
    Sides = 60,
    Thickness = 1,
    Filled = false
}

Environment.FOVCircle = Drawing.new("Circle")
Environment.Locked = nil

--// Core Functions

local function Encode(Table)
	if Table and type(Table) == "table" then
		local EncodedTable = HttpService:JSONEncode(Table)

		return EncodedTable
	end
end

local function Decode(String)
	if String and type(String) == "string" then
		local DecodedTable = HttpService:JSONDecode(String)

		return DecodedTable
	end
end

local function GetColor(Color)
	local R = tonumber(string.match(Color, "([%d]+)[%s]*,[%s]*[%d]+[%s]*,[%s]*[%d]+"))
	local G = tonumber(string.match(Color, "[%d]+[%s]*,[%s]*([%d]+)[%s]*,[%s]*[%d]+"))
	local B = tonumber(string.match(Color, "[%d]+[%s]*,[%s]*[%d]+[%s]*,[%s]*([%d]+)"))

	return Color3.fromRGB(R, G, B)
end

local function SendNotification(TitleArg, DescriptionArg, DurationArg)
	if Environment.Settings.SendNotifications then
		StarterGui:SetCore("SendNotification", {
			Title = TitleArg,
			Text = DescriptionArg,
			Duration = DurationArg
		})
	end
end

--// Functions

local function SaveSettings()
	if Environment.Settings.SaveSettings then
		if isfile(Title.."/"..FileNames[1].."/"..FileNames[2]) then
			writefile(Title.."/"..FileNames[1].."/"..FileNames[2], Encode(Environment.Settings))
		end

		if isfile(Title.."/"..FileNames[1].."/"..FileNames[3]) then
			writefile(Title.."/"..FileNames[1].."/"..FileNames[3], Encode(Environment.FOVSettings))
		end
	end
end

local function GetClosestPlayer()
	if Environment.Locked == nil then
		if Environment.FOVSettings.Enabled then
			RequiredDistance = Environment.FOVSettings.Amount
		else
			RequiredDistance = math.huge
		end

		for _, v in next, Players:GetPlayers() do
			if v ~= LocalPlayer then
				if v.Character and v.Character[Environment.Settings.LockPart] then
					if Environment.Settings.TeamCheck and v.Team == LocalPlayer.Team then continue end
					if Environment.Settings.AliveCheck and v.Character.Humanoid.Health <= 0 then continue end
					if Environment.Settings.WallCheck and #(Camera:GetPartsObscuringTarget({v.Character[Environment.Settings.LockPart].Position}, v.Character:GetDescendants())) > 0 then continue end

					local Vector, OnScreen = Camera:WorldToViewportPoint(v.Character[Environment.Settings.LockPart].Position)
					local Distance = (Vector2.new(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y) - Vector2.new(Vector.X, Vector.Y)).Magnitude

					if Distance < RequiredDistance and OnScreen then
						RequiredDistance = Distance
						Environment.Locked = v
					end
				end
			end
		end
	elseif (Vector2.new(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y) - Vector2.new(Camera:WorldToViewportPoint(Environment.Locked.Character[Environment.Settings.LockPart].Position).X, Camera:WorldToViewportPoint(Environment.Locked.Character[Environment.Settings.LockPart].Position).Y)).Magnitude > RequiredDistance then
		Environment.Locked = nil
		Animation:Cancel()
		Environment.FOVCircle.Color = GetColor(Environment.FOVSettings.Color)
	end
end

--// Typing Check

ServiceConnections.TypingStartedConnection = UserInputService.TextBoxFocused:Connect(function()
	Typing = true
end)

ServiceConnections.TypingEndedConnection = UserInputService.TextBoxFocusReleased:Connect(function()
	Typing = false
end)

--// Create, Save & Load Settings

if Environment.Settings.SaveSettings then
	if not isfolder(Title) then
		makefolder(Title)
	end

	if not isfolder(Title.."/"..FileNames[1]) then
		makefolder(Title.."/"..FileNames[1])
	end

	if not isfile(Title.."/"..FileNames[1].."/"..FileNames[2]) then
		writefile(Title.."/"..FileNames[1].."/"..FileNames[2], Encode(Environment.Settings))
	else
		Environment.Settings = Decode(readfile(Title.."/"..FileNames[1].."/"..FileNames[2]))
	end

	if not isfile(Title.."/"..FileNames[1].."/"..FileNames[3]) then
		writefile(Title.."/"..FileNames[1].."/"..FileNames[3], Encode(Environment.FOVSettings))
	else
		Environment.Visuals = Decode(readfile(Title.."/"..FileNames[1].."/"..FileNames[3]))
	end

	coroutine.wrap(function()
		while wait(10) do
			SaveSettings()
		end
	end)()
else
	if isfolder(Title) then
		delfolder(Title)
	end
end

local function Load()
	ServiceConnections.RenderSteppedConnection = RunService.RenderStepped:Connect(function()
		if Environment.FOVSettings.Enabled and Environment.Settings.Enabled then
			Environment.FOVCircle.Radius = Environment.FOVSettings.Amount
			Environment.FOVCircle.Thickness = Environment.FOVSettings.Thickness
			Environment.FOVCircle.Filled = Environment.FOVSettings.Filled
			Environment.FOVCircle.NumSides = Environment.FOVSettings.Sides
			Environment.FOVCircle.Color = GetColor(Environment.FOVSettings.Color)
			Environment.FOVCircle.Transparency = Environment.FOVSettings.Transparency
			Environment.FOVCircle.Visible = Environment.FOVSettings.Visible
			Environment.FOVCircle.Position = Vector2.new(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y)
		else
			Environment.FOVCircle.Visible = false
		end

		if Running and Environment.Settings.Enabled then
			GetClosestPlayer()

			if Environment.Settings.ThirdPerson then
				Environment.Settings.ThirdPersonSensitivity = math.clamp(Environment.Settings.ThirdPersonSensitivity, 0.1, 5)

				local Vector = Camera:WorldToViewportPoint(Environment.Locked.Character[Environment.Settings.LockPart].Position)
				mousemoverel((Vector.X - UserInputService:GetMouseLocation().X) * Environment.Settings.ThirdPersonSensitivity, (Vector.Y - UserInputService:GetMouseLocation().Y) * Environment.Settings.ThirdPersonSensitivity)
			else
				if Environment.Settings.Sensitivity > 0 then
					Animation = TweenService:Create(Camera, TweenInfo.new(Environment.Settings.Sensitivity, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {CFrame = CFrame.new(Camera.CFrame.Position, Environment.Locked.Character[Environment.Settings.LockPart].Position)})
					Animation:Play()
				else
					Camera.CFrame = CFrame.new(Camera.CFrame.Position, Environment.Locked.Character[Environment.Settings.LockPart].Position)
				end
			end

			Environment.FOVCircle.Color = GetColor(Environment.FOVSettings.LockedColor)
		end
	end)

	ServiceConnections.InputBeganConnection = UserInputService.InputBegan:Connect(function(Input)
		if not Typing then
			pcall(function()
				if Input.KeyCode == Enum.KeyCode[Environment.Settings.TriggerKey] then
					if Environment.Settings.Toggle then
						Running = not Running

						if not Running then
							Environment.Locked = nil
							Animation:Cancel()
							Environment.FOVCircle.Color = GetColor(Environment.FOVSettings.Color)
						end
					else
						Running = true
					end
				end
			end)

			pcall(function()
				if Input.UserInputType == Enum.UserInputType[Environment.Settings.TriggerKey] then
					if Environment.Settings.Toggle then
						Running = not Running

						if not Running then
							Environment.Locked = nil
							Animation:Cancel()
							Environment.FOVCircle.Color = GetColor(Environment.FOVSettings.Color)
						end
					else
						Running = true
					end
				end
			end)
		end
	end)

	ServiceConnections.InputEndedConnection = UserInputService.InputEnded:Connect(function(Input)
		if not Typing then
			pcall(function()
				if Input.KeyCode == Enum.KeyCode[Environment.Settings.TriggerKey] then
					if not Environment.Settings.Toggle then
						Running = false
						Environment.Locked = nil
						Animation:Cancel()
						Environment.FOVCircle.Color = GetColor(Environment.FOVSettings.Color)
					end
				end
			end)

			pcall(function()
				if Input.UserInputType == Enum.UserInputType[Environment.Settings.TriggerKey] then
					if not Environment.Settings.Toggle then
						Running = false
						Environment.Locked = nil
						Animation:Cancel()
						Environment.FOVCircle.Color = GetColor(Environment.FOVSettings.Color)
					end
				end
			end)
		end
	end)
end

--// Functions

Environment.Functions = {}

function Environment.Functions:Exit()
	SaveSettings()

	for _, v in next, ServiceConnections do
		v:Disconnect()
	end

	Environment.FOVCircle:Remove()

	getgenv().Aimbot.Functions = nil
	getgenv().Aimbot = nil
end

function Environment.Functions:Restart()
	SaveSettings()

	for _, v in next, ServiceConnections do
		v:Disconnect()
	end

	Environment.FOVCircle:Remove()

	Load()
end

function Environment.Functions:ResetSettings()
getgenv().Aimbot.Settings = {
    SendNotifications = true,
    SaveSettings = true, -- Re-execute upon changing
    ReloadOnTeleport = true,
    Enabled = true,
    TeamCheck = false,
    AliveCheck = true,
    WallCheck = false, -- Laggy
    Sensitivity = 0, -- Animation length (in seconds) before fully locking onto target
    ThirdPerson = false, -- Uses mousemoverel instead of CFrame to support locking in third person (could be choppy)
    ThirdPersonSensitivity = 3, -- Boundary: 0.1 - 5
    TriggerKey = "F",
    Toggle = false,
    LockPart = "Head" -- Body part to lock on (Character part's name)
}

Environment.FOVSettings = {
	Enabled = true,
    Visible = true,
    Amount = 90,
    Color = "255, 255, 255",
    LockedColor = "255, 70, 70",
    Transparency = 0.5,
    Sides = 60,
    Thickness = 1,
    Filled = false
}

	SaveSettings()

	for _, v in next, ServiceConnections do
		v:Disconnect()
	end

	Load()
end

--// Support Check

if not Drawing or not writefile or not makefolder then
	SendNotification(Title, "Your exploit does not support this script", 3); return
end

--// Reload On Teleport

if Environment.Settings.ReloadOnTeleport then
	if queueonteleport then
		queueonteleport(game:HttpGet("https://pastebin.com/raw/S5SRX4dT"))
	else
		
	end
end

--// Load

Load(); SendNotification(Title, "Eiei..", 5)

else
    
getgenv().Aimbot.Functions:Exit()
end
end)
Section:NewSlider("FOV", "", 1000, 0, function(s) -- 500 (MaxValue) | 0 (MinValue)
getgenv().Aimbot.Settings = {
	SendNotifications = true,
    SaveSettings = true, -- Re-execute upon changing
    ReloadOnTeleport = true,
    Enabled = true,
    TeamCheck = false,
    AliveCheck = true,
    WallCheck = false, -- Laggy
    Sensitivity = 0, -- Animation length (in seconds) before fully locking onto target
    ThirdPerson = false, -- Uses mousemoverel instead of CFrame to support locking in third person (could be choppy)
    ThirdPersonSensitivity = 3, -- Boundary: 0.1 - 5
    TriggerKey = "F",
    Toggle = false,
    LockPart = "Head"   -- Body part to lock on
}

getgenv().Aimbot.FOVSettings = {
	Enabled = true,
    Visible = true,
    Amount = s,
    Color = "4,0,255",
    LockedColor = "255, 0, 0",
    Transparency = 1,
    Sides = 60,
    Thickness = 1,
    Filled = false
}
end)
Section:NewButton("ButtonText", "ButtonInfo", function()
do
    local Plr = game:GetService("Players").LocalPlayer
    for i,v in pairs(Plr.Backpack:GetChildren()) do
        Plr.Character.Humanoid:EquipTool(v)
        task.wait()
        Plr.Character.Humanoid:UnequipTools(v)

    end
end

for i,v in pairs(getgc(true)) do
    if type(v) == 'table' and rawget(v, 'Ammo') then
        v.Ammo = 9e9
        v.StoredAmmo = 9e9
        v.MaxStoredAmmo = 9e9
        v.gunRecoil = {
            gunRecoilUp = {0,0},
            gunRecoilTilt = {0,0},
            gunRecoilLeft = {0,0},
            gunRecoilRight = {0,0},
        }
        v.camRecoil = {
            camRecoilUp = {0,0},
            camRecoilTilt = {0,0},
            camRecoilLeft = {0,0},
            camRecoilRight = {0,0},
        }
        v.FireModes = {
            ChangeFiremode = true,
            Semi = true,
            Burst = true,
            Auto = true }
        v.MinSpread = 0
        v.MaxSpread = 0
        task.wait()
    end
end
end)
local Section = Tab:NewSection("Other")
Section:NewButton("Can't open aimbot press here", "ButtonInfo", function()
    pcall(function()
        print("Successfully reset!!")
	if isfolder("Exunys Developer/Aimbot") then
		delfolder("Exunys Developer/Aimbot")
	end
end)
end)
Section:NewButton("Esp Gui", "", function()
    loadstring(game:HttpGet("https://pastebin.com/raw/Ty10AdDd"))()
end)
Section:NewToggle("Noclip", "", function(state)
    if state then
_G.NC = not _G.NC
 
local pos = game:GetService('Players').LocalPlayer.Character.LowerTorso.Position.Y
 
if _G.InitNC ~= true then
    _G.NCFunc = function(part)
        if _G.NC then
            if part.Position.Y > pos then
                part.CanCollide = false
            end
        end
    end
 
    _G.InitNC = true
end
 
if _G.NC and _G.NCFunc ~= nil then
    game:GetService('Players').LocalPlayer.Character.Humanoid.Touched:connect(_G.NCFunc)
end
    else
_G.NC = false
 
local pos = game:GetService('Players').LocalPlayer.Character.LowerTorso.Position.Y
 
if _G.InitNC ~= false then
    _G.NCFunc = function(part)
        if _G.NC then
            if part.Position.Y > pos then
                part.CanCollide = false
            end
        end
    end
 
    _G.InitNC = false
end
 
if _G.NC and _G.NCFunc ~= false then
    game:GetService('Players').LocalPlayer.Character.Humanoid.Touched:connect(_G.NCFunc)
end
    end
end)
Section:NewButton("Noclip[GUI]", "", function()
local Workspace = game:GetService("Workspace")
local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local Noclip = Instance.new("ScreenGui")
local BG = Instance.new("Frame")
local Title = Instance.new("TextLabel")
local Toggle = Instance.new("TextButton")
local StatusPF = Instance.new("TextLabel")
local Status = Instance.new("TextLabel")
local Credit = Instance.new("TextLabel")
local Plr = Players.LocalPlayer
local Clipon = false

Noclip.Name = "Noclip"
Noclip.Parent = game.CoreGui

BG.Name = "BG"
BG.Parent = Noclip
BG.BackgroundColor3 = Color3.new(0.0980392, 0.0980392, 0.0980392)
BG.BorderColor3 = Color3.new(0.0588235, 0.0588235, 0.0588235)
BG.BorderSizePixel = 2
BG.Position = UDim2.new(0.149479166, 0, 0.82087779, 0)
BG.Size = UDim2.new(0, 210, 0, 127)
BG.Active = true
BG.Draggable = true

Title.Name = "Title"
Title.Parent = BG
Title.BackgroundColor3 = Color3.new(0.266667, 0.00392157, 0.627451)
Title.BorderColor3 = Color3.new(0.180392, 0, 0.431373)
Title.BorderSizePixel = 2
Title.Size = UDim2.new(0, 210, 0, 33)
Title.Font = Enum.Font.Highway
Title.Text = "Noclip"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.FontSize = Enum.FontSize.Size32
Title.TextSize = 30
Title.TextStrokeColor3 = Color3.new(0.180392, 0, 0.431373)
Title.TextStrokeTransparency = 0

Toggle.Parent = BG
Toggle.BackgroundColor3 = Color3.new(0.266667, 0.00392157, 0.627451)
Toggle.BorderColor3 = Color3.new(0.180392, 0, 0.431373)
Toggle.BorderSizePixel = 2
Toggle.Position = UDim2.new(0.152380958, 0, 0.374192119, 0)
Toggle.Size = UDim2.new(0, 146, 0, 36)
Toggle.Font = Enum.Font.Highway
Toggle.FontSize = Enum.FontSize.Size28
Toggle.Text = "Toggle"
Toggle.TextColor3 = Color3.new(1, 1, 1)
Toggle.TextSize = 25
Toggle.TextStrokeColor3 = Color3.new(0.180392, 0, 0.431373)
Toggle.TextStrokeTransparency = 0

StatusPF.Name = "StatusPF"
StatusPF.Parent = BG
StatusPF.BackgroundColor3 = Color3.new(1, 1, 1)
StatusPF.BackgroundTransparency = 1
StatusPF.Position = UDim2.new(0.314285725, 0, 0.708661377, 0)
StatusPF.Size = UDim2.new(0, 56, 0, 20)
StatusPF.Font = Enum.Font.Highway
StatusPF.FontSize = Enum.FontSize.Size24
StatusPF.Text = "Status:"
StatusPF.TextColor3 = Color3.new(1, 1, 1)
StatusPF.TextSize = 20
StatusPF.TextStrokeColor3 = Color3.new(0.333333, 0.333333, 0.333333)
StatusPF.TextStrokeTransparency = 0
StatusPF.TextWrapped = true

Status.Name = "Status"
Status.Parent = BG
Status.BackgroundColor3 = Color3.new(1, 1, 1)
Status.BackgroundTransparency = 1
Status.Position = UDim2.new(0.580952346, 0, 0.708661377, 0)
Status.Size = UDim2.new(0, 56, 0, 20)
Status.Font = Enum.Font.Highway
Status.FontSize = Enum.FontSize.Size14
Status.Text = "off"
Status.TextColor3 = Color3.new(0.666667, 0, 0)
Status.TextScaled = true
Status.TextSize = 14
Status.TextStrokeColor3 = Color3.new(0.180392, 0, 0.431373)
Status.TextWrapped = true
Status.TextXAlignment = Enum.TextXAlignment.Left

Credit.Name = "Credit"
Credit.Parent = BG
Credit.BackgroundColor3 = Color3.new(1, 1, 1)
Credit.BackgroundTransparency = 1
Credit.Position = UDim2.new(0.195238099, 0, 0.866141737, 0)
Credit.Size = UDim2.new(0, 128, 0, 17)
Credit.Font = Enum.Font.SourceSans
Credit.FontSize = Enum.FontSize.Size18
Credit.Text = "Created by Korn"
Credit.TextColor3 = Color3.new(1, 1, 1)
Credit.TextSize = 16
Credit.TextStrokeColor3 = Color3.new(0.196078, 0.196078, 0.196078)
Credit.TextStrokeTransparency = 0
Credit.TextWrapped = true

Toggle.MouseButton1Click:connect(function()
	if Status.Text == "off" then
		Clipon = true
		Status.Text = "on"
		Status.TextColor3 = Color3.new(0,185,0)
		Stepped = game:GetService("RunService").Stepped:Connect(function()
			if not Clipon == false then
				for a, b in pairs(Workspace:GetChildren()) do
                if b.Name == Plr.Name then
                for i, v in pairs(Workspace[Plr.Name]:GetChildren()) do
                if v:IsA("BasePart") then
                v.CanCollide = false
                end end end end
			else
				Stepped:Disconnect()
			end
		end)
	elseif Status.Text == "on" then
		Clipon = false
		Status.Text = "off"
		Status.TextColor3 = Color3.new(170,0,0)
	end
end)
end)
Section:NewToggle("EIEI", "", function(state)
    if state then
_G.FF = true
while _G.FF do wait(0)
do
    local NM = game:GetService("Workspace"):FindFirstChild("Noclip")
    if NM then
        NM:Destroy()
    end
end
local Noclip = Instance.new("Part",workspace)
     Noclip.Name = "Noclip"
     Noclip.CanCollide = true
     Noclip.Anchored = true
     Noclip.Transparency = 0
     Noclip.Size = Vector3.new(30,-5,30)
     Noclip.CFrame = game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame * CFrame.new(0,-5,0)
end
     
    else
_G.FF = false
while _G.FF do wait(0)
do
    local NM = game:GetService("Workspace"):FindFirstChild("Noclip")
    if NM then
        NM:Destroy()
    end
end
local Noclip = Instance.new("Part",workspace)
     Noclip.Name = "Noclip"
     Noclip.CanCollide = true
     Noclip.Anchored = true
     Noclip.Transparency = 0
     Noclip.Size = Vector3.new(30,-5,30)
     Noclip.CFrame = game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame * CFrame.new(0,-5,0)
end
     
    end
end)
local Tab = Window:NewTab("UI")
local Section = Tab:NewSection("UI Key")
Section:NewKeybind("Key", "", Enum.KeyCode.RightControl, function()
	Library:ToggleUI()
end)


--479BA819-09FA-493C-BE57-EAF8CC265E73
