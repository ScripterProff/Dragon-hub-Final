local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

local Settings = {
    Aimbot = false,
    ESP = false,
    FOV = 100,
    AimPart = "Head",
    TeamCheck = true
}

-- GUI reduzido para mobile
local ScreenGui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
ScreenGui.Name = "DragonHub"
ScreenGui.ResetOnSpawn = false

local Main = Instance.new("Frame", ScreenGui)
Main.Size = UDim2.new(0, 220, 0, 260)
Main.Position = UDim2.new(0.5, -110, 0.5, -130)
Main.AnchorPoint = Vector2.new(0.5, 0.5)
Main.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Main.BorderSizePixel = 0
Main.Active = true
Main.Draggable = true

local UIScale = Instance.new("UIScale", Main)
UIScale.Scale = 1.0

local Title = Instance.new("TextLabel", Main)
Title.Size = UDim2.new(1, 0, 0, 30)
Title.Text = "Dragon Hub"
Title.BackgroundTransparency = 1
Title.TextColor3 = Color3.new(1, 1, 1)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 16

-- Botões compactos
local function createButton(text, yPos, callback)
	local button = Instance.new("TextButton", Main)
	button.Size = UDim2.new(0.9, 0, 0, 30)
	button.Position = UDim2.new(0.05, 0, 0, yPos)
	button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
	button.TextColor3 = Color3.new(1, 1, 1)
	button.Text = text
	button.Font = Enum.Font.Gotham
	button.TextSize = 14
	button.MouseButton1Click:Connect(callback)
end

createButton("Toggle Aimbot", 40, function()
	Settings.Aimbot = not Settings.Aimbot
end)

createButton("Toggle ESP", 80, function()
	Settings.ESP = not Settings.ESP
end)

createButton("Minimizar", 170, function()
	Main.Visible = false
end)

createButton("Fechar", 210, function()
	ScreenGui:Destroy()
end)

-- FOV Circle
local FOV = Drawing.new("Circle")
FOV.Color = Color3.fromRGB(255, 255, 0)
FOV.Thickness = 2
FOV.Filled = false
FOV.Radius = Settings.FOV
FOV.Visible = true

-- Aimbot
local function GetClosest()
	local closest, shortest = nil, Settings.FOV
	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild(Settings.AimPart) then
			if not Settings.TeamCheck or player.Team ~= LocalPlayer.Team then
				local pos, onScreen = Camera:WorldToViewportPoint(player.Character[Settings.AimPart].Position)
				if onScreen then
					local dist = (Vector2.new(pos.X, pos.Y) - Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)).Magnitude
					if dist < shortest then
						shortest = dist
						closest = player.Character[Settings.AimPart]
					end
				end
			end
		end
	end
	return closest
end

-- Toque para mirar
local aiming = false
UserInputService.TouchStarted:Connect(function()
	aiming = true
end)
UserInputService.TouchEnded:Connect(function()
	aiming = false
end)

RunService.RenderStepped:Connect(function()
	FOV.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

	if Settings.Aimbot and aiming then
		local target = GetClosest()
		if target then
			Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Position)
		end
	end
end)
