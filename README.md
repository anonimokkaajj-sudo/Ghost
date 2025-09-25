-- LocalScript (StarterGui)
local player = game.Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")
local RunService = game:GetService("RunService")

-- Guardar transparência original (Xray)
local originalTransparency = {}

-- Ghost plataforma
local ghostPlatform
local ghostConnection
local ghostEnabled = false
local xrayEnabled = false

-- Função Ghost
local function enableGhost()
	ghostEnabled = true
	if ghostPlatform then ghostPlatform:Destroy() end

	local char = player.Character or player.CharacterAdded:Wait()
	local hrp = char:WaitForChild("HumanoidRootPart")

	ghostPlatform = Instance.new("Part")
	ghostPlatform.Size = Vector3.new(6, 1, 6)
	ghostPlatform.Anchored = true
	ghostPlatform.CanCollide = true
	-- NÃO muda cor nem nome
	ghostPlatform.Parent = workspace

	if ghostConnection then ghostConnection:Disconnect() end
	ghostConnection = RunService.Heartbeat:Connect(function()
		if hrp and hrp.Parent then
			ghostPlatform.CFrame = CFrame.new(hrp.Position.X, hrp.Position.Y - 3, hrp.Position.Z)
		end
	end)
end

local function disableGhost()
	ghostEnabled = false
	if ghostConnection then
		ghostConnection:Disconnect()
		ghostConnection = nil
	end
	if ghostPlatform then
		ghostPlatform:Destroy()
		ghostPlatform = nil
	end
end

-- Função Xray
local function enableXray()
	xrayEnabled = true
	for _, obj in pairs(workspace:GetDescendants()) do
		if obj:IsA("BasePart") then
			if obj.Name:lower():find("base") then
				if not originalTransparency[obj] then
					originalTransparency[obj] = obj.Transparency
				end
				obj.LocalTransparencyModifier = 0.7
			end
		end
	end
end

local function disableXray()
	xrayEnabled = false
	for obj, trans in pairs(originalTransparency) do
		if obj and obj.Parent then
			obj.LocalTransparencyModifier = 0
		end
	end
	originalTransparency = {}
end

-- Reaplica quando respawnar
player.CharacterAdded:Connect(function()
	local char = player.Character or player.CharacterAdded:Wait()
	char:WaitForChild("HumanoidRootPart")

	if ghostEnabled then
		enableGhost()
	end
	if xrayEnabled then
		enableXray()
	end
end)

-- Criar menu
local function createMenu(name, offsetY)
	local screenGui = Instance.new("ScreenGui")
	screenGui.Name = name .. "Gui"
	screenGui.Parent = playerGui
	screenGui.IgnoreGuiInset = true

	local mainFrame = Instance.new("Frame", screenGui)
	mainFrame.Size = UDim2.new(0, 85, 0, 55)
	mainFrame.Position = UDim2.new(0.5, -42, 0.5, offsetY) -- centralizado
	mainFrame.BackgroundColor3 = Color3.new(0,0,0)
	mainFrame.BackgroundTransparency = 0.4
	mainFrame.BorderSizePixel = 0
	Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 10)

	local label = Instance.new("TextLabel", mainFrame)
	label.Size = UDim2.new(1, 0, 0, 18)
	label.Position = UDim2.new(0, 0, 0, 3)
	label.BackgroundTransparency = 1
	label.Text = name
	label.AutoLocalize = false
	label.Font = Enum.Font.GothamBold
	label.TextSize = 13
	label.TextColor3 = Color3.new(1,1,1)

	local switchFrame = Instance.new("TextButton", mainFrame)
	switchFrame.Size = UDim2.new(0, 45, 0, 22)
	switchFrame.Position = UDim2.new(0.5, -22, 0, 28)
	switchFrame.BackgroundColor3 = Color3.fromRGB(100,100,100)
	switchFrame.Text = ""
	Instance.new("UICorner", switchFrame).CornerRadius = UDim.new(0,11)

	local ball = Instance.new("Frame", switchFrame)
	ball.Size = UDim2.new(0, 16, 0, 16)
	ball.Position = UDim2.new(0, 3, 0, 3)
	ball.BackgroundColor3 = Color3.new(1,1,1)
	Instance.new("UICorner", ball).CornerRadius = UDim.new(1,0)

	local isOn = false
	switchFrame.Activated:Connect(function()
		isOn = not isOn
		if isOn then
			switchFrame.BackgroundColor3 = Color3.fromRGB(128,0,128)
			ball:TweenPosition(UDim2.new(1,-19,0,3), "Out","Quad",0.25,true)

			if name == "Ghost" then
				enableGhost()
			elseif name == "Xray" then
				enableXray()
			end
		else
			switchFrame.BackgroundColor3 = Color3.fromRGB(100,100,100)
			ball:TweenPosition(UDim2.new(0,3,0,3), "Out","Quad",0.25,true)

			if name == "Ghost" then
				disableGhost()
			elseif name == "Xray" then
				disableXray()
			end
		end
	end)

	-- Arrastável
	local UserInputService = game:GetService("UserInputService")
	local dragging, dragInput, dragStart, startPos

	local function update(input)
		local delta = input.Position - dragStart
		mainFrame.Position = UDim2.new(
			startPos.X.Scale, startPos.X.Offset + delta.X,
			startPos.Y.Scale, startPos.Y.Offset + delta.Y
		)
	end

	mainFrame.InputBegan:Connect(function(input)
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

	mainFrame.InputChanged:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
			dragInput = input
		end
	end)

	UserInputService.InputChanged:Connect(function(input)
		if input == dragInput and dragging then
			update(input)
		end
	end)
end

-- Criar menus no centro
createMenu("Ghost", -70) -- em cima
createMenu("Xray", 0)   -- logo abaixo
