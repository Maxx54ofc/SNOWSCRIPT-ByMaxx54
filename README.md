local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local mouse = player:GetMouse()

-- Configurações iniciais
local snowEnabled = false
local particleCount = 70
local particleLimit = 100
local spawnHeight = 30
local forwardDistance = 30
local fallSpeed = 5
local snowflakeSize = 0.5 -- Novo: tamanho padrão do floco

-- Criação da ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = player:WaitForChild("PlayerGui")
screenGui.Name = "SnowSystemGui"
screenGui.ResetOnSpawn = false -- Impede que o menu desapareça ao morrer

-- Função para criar botões com bordas arredondadas
local function createRoundedButton(parent, size, position, text)
	local button = Instance.new("TextButton")
	button.Size = size
	button.Position = position
	button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
	button.TextColor3 = Color3.fromRGB(255, 255, 255)
	button.Text = text
	button.BorderSizePixel = 0

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 10)
	corner.Parent = button

	button.Parent = parent
	return button
end

-- Função para criar campos de texto com bordas arredondadas
local function createTextBox(parent, size, position, placeholder)
	local textBox = Instance.new("TextBox")
	textBox.Size = size
	textBox.Position = position
	textBox.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
	textBox.TextColor3 = Color3.fromRGB(255, 255, 255)
	textBox.PlaceholderText = placeholder
	textBox.Text = ""
	textBox.BorderSizePixel = 0

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 10)
	corner.Parent = textBox

	textBox.Parent = parent
	return textBox
end

-- Função para tornar um elemento arrastável
local function makeDraggable(frame)
	local dragging = false
	local dragInput, dragStart, startPos

	frame.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = true
			dragStart = input.Position
			startPos = frame.Position
		end
	end)

	frame.InputChanged:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseMovement then
			dragInput = input
		end
	end)

	UserInputService.InputChanged:Connect(function(input)
		if input == dragInput and dragging then
			local delta = input.Position - dragStart
			frame.Position = UDim2.new(
				startPos.X.Scale,
				startPos.X.Offset + delta.X,
				startPos.Y.Scale,
				startPos.Y.Offset + delta.Y
			)
		end
	end)

	frame.InputEnded:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = false
		end
	end)
end

-- Botão principal
local mainButton = createRoundedButton(
	screenGui,
	UDim2.new(0, 45, 0, 45),
	UDim2.new(0, 10, 0, 10),
	"Snow"
)
makeDraggable(mainButton)

-- Frame do menu (inicialmente invisível)
local menuFrame = Instance.new("Frame")
menuFrame.Size = UDim2.new(0, 200, 0, 300)
menuFrame.Position = UDim2.new(0, 60, 0, 10)
menuFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
menuFrame.Visible = false
menuFrame.Parent = screenGui

local menuCorner = Instance.new("UICorner")
menuCorner.CornerRadius = UDim.new(0, 10)
menuCorner.Parent = menuFrame

makeDraggable(menuFrame)

-- ScrollingFrame dentro do menu
local scrollingFrame = Instance.new("ScrollingFrame")
scrollingFrame.Size = UDim2.new(1, 0, 1, 0)
scrollingFrame.Position = UDim2.new(0, 0, 0, 0)
scrollingFrame.BackgroundTransparency = 1
scrollingFrame.ScrollBarThickness = 5
scrollingFrame.CanvasSize = UDim2.new(0, 0, 0, 420) -- Aumentado para nova opção
scrollingFrame.Parent = menuFrame

-- Elementos do menu
local toggleButton = createRoundedButton(
	scrollingFrame,
	UDim2.new(0, 180, 0, 40),
	UDim2.new(0, 10, 0, 10),
	"Toggle Snow: OFF"
)

local particleLabel = Instance.new("TextLabel")
particleLabel.Size = UDim2.new(0, 180, 0, 20)
particleLabel.Position = UDim2.new(0, 10, 0, 60)
particleLabel.BackgroundTransparency = 1
particleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
particleLabel.Text = "Particle Count: " .. particleCount
particleLabel.Parent = scrollingFrame

local particleBox = createTextBox(
	scrollingFrame,
	UDim2.new(0, 180, 0, 30),
	UDim2.new(0, 10, 0, 80),
	"Enter count (max " .. particleLimit .. ")"
)

local limitLabel = Instance.new("TextLabel")
limitLabel.Size = UDim2.new(0, 180, 0, 20)
limitLabel.Position = UDim2.new(0, 10, 0, 120)
limitLabel.BackgroundTransparency = 1
limitLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
limitLabel.Text = "Particle Limit: " .. particleLimit
limitLabel.Parent = scrollingFrame

local limitBox = createTextBox(
	scrollingFrame,
	UDim2.new(0, 180, 0, 30),
	UDim2.new(0, 10, 0, 140),
	"Enter limit"
)

local heightLabel = Instance.new("TextLabel")
heightLabel.Size = UDim2.new(0, 180, 0, 20)
heightLabel.Position = UDim2.new(0, 10, 0, 180)
heightLabel.BackgroundTransparency = 1
heightLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
heightLabel.Text = "Spawn Height: " .. spawnHeight
heightLabel.Parent = scrollingFrame

local heightBox = createTextBox(
	scrollingFrame,
	UDim2.new(0, 180, 0, 30),
	UDim2.new(0, 10, 0, 200),
	"Enter height"
)

local distanceLabel = Instance.new("TextLabel")
distanceLabel.Size = UDim2.new(0, 180, 0, 20)
distanceLabel.Position = UDim2.new(0, 10, 0, 240)
distanceLabel.BackgroundTransparency = 1
distanceLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
distanceLabel.Text = "Distance: " .. forwardDistance
distanceLabel.Parent = scrollingFrame

local distanceBox = createTextBox(
	scrollingFrame,
	UDim2.new(0, 180, 0, 30),
	UDim2.new(0, 10, 0, 260),
	"Enter distance"
)

local speedLabel = Instance.new("TextLabel")
speedLabel.Size = UDim2.new(0, 180, 0, 20)
speedLabel.Position = UDim2.new(0, 10, 0, 300)
speedLabel.BackgroundTransparency = 1
speedLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
speedLabel.Text = "Fall Speed: " .. fallSpeed
speedLabel.Parent = scrollingFrame

local speedBox = createTextBox(
	scrollingFrame,
	UDim2.new(0, 180, 0, 30),
	UDim2.new(0, 10, 0, 320),
	"Enter speed"
)

-- Nova opção: Tamanho do floco
local sizeLabel = Instance.new("TextLabel")
sizeLabel.Size = UDim2.new(0, 180, 0, 20)
sizeLabel.Position = UDim2.new(0, 10, 0, 360)
sizeLabel.BackgroundTransparency = 1
sizeLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
sizeLabel.Text = "Snowflake Size: " .. snowflakeSize
sizeLabel.Parent = scrollingFrame

local sizeBox = createTextBox(
	scrollingFrame,
	UDim2.new(0, 180, 0, 30),
	UDim2.new(0, 10, 0, 380),
	"Enter size (0.1-5)"
)

-- Sistema de neve
local activeParticles = {} -- Tabela para rastrear partículas ativas

local function createSnowflake()
	local snowflake = Instance.new("Part")
	snowflake.Size = Vector3.new(snowflakeSize, snowflakeSize, snowflakeSize) -- Usa o tamanho configurável
	snowflake.Shape = Enum.PartType.Ball
	snowflake.Material = Enum.Material.SmoothPlastic
	snowflake.Color = Color3.fromRGB(255, 255, 255)
	snowflake.CanCollide = false
	snowflake.Anchored = false

	local char = player.Character or player.CharacterAdded:Wait()
	local root = char:WaitForChild("HumanoidRootPart")
	local spawnPos = root.Position + Vector3.new(
		math.random(-forwardDistance, forwardDistance),
		spawnHeight,
		math.random(-forwardDistance, forwardDistance)
	)

	snowflake.Position = spawnPos
	snowflake.Parent = workspace

	snowflake.Touched:Connect(function(hit)
		if hit ~= player.Character then
			snowflake:Destroy()
		end
	end)

	local bodyVelocity = Instance.new("BodyVelocity")
	bodyVelocity.Velocity = Vector3.new(0, -fallSpeed, 0)
	bodyVelocity.MaxForce = Vector3.new(0, math.huge, 0)
	bodyVelocity.Parent = snowflake

	-- Adiciona a partícula à tabela
	table.insert(activeParticles, snowflake)

	spawn(function()
		wait(10) -- Tempo de vida máximo
		if snowflake.Parent then
			snowflake:Destroy()
			-- Remove da tabela quando destruído
			for i, v in ipairs(activeParticles) do
				if v == snowflake then
					table.remove(activeParticles, i)
					break
				end
			end
		end
	end)

	-- Verifica o limite e remove o floco mais antigo se necessário
	if #activeParticles > particleLimit then
		local oldestSnowflake = table.remove(activeParticles, 1) -- Remove o primeiro (mais antigo)
		if oldestSnowflake and oldestSnowflake.Parent then
			oldestSnowflake:Destroy()
		end
	end
end

local snowLoopRunning = false

local function startSnowLoop()
	if snowLoopRunning then return end
	snowLoopRunning = true

	spawn(function()
		while snowEnabled do
			local spawnRate = particleCount / 10 -- Taxa de spawn por segundo
			local delayBetweenSpawns = 1 / spawnRate

			for i = 1, particleCount do
				if not snowEnabled then break end
				createSnowflake()
				wait(delayBetweenSpawns) -- Distribui o spawn ao longo do tempo
			end
		end
		snowLoopRunning = false
	end)
end

-- Conexões dos botões e campos de texto
mainButton.MouseButton1Click:Connect(function()
	menuFrame.Visible = not menuFrame.Visible
end)

toggleButton.MouseButton1Click:Connect(function()
	snowEnabled = not snowEnabled
	toggleButton.Text = "Toggle Snow: " .. (snowEnabled and "ON" or "OFF")
	if snowEnabled then
		startSnowLoop()
	end
end)

particleBox.FocusLost:Connect(function(enterPressed)
	if enterPressed then
		local value = tonumber(particleBox.Text)
		if value then
			particleCount = math.clamp(value, 1, particleLimit)
			particleLabel.Text = "Particle Count: " .. particleCount
			particleBox.Text = ""
		end
	end
end)

limitBox.FocusLost:Connect(function(enterPressed)
	if enterPressed then
		local value = tonumber(limitBox.Text)
		if value then
			particleLimit = math.max(value, 1)
			limitLabel.Text = "Particle Limit: " .. particleLimit
			particleBox.PlaceholderText = "Enter count (max " .. particleLimit .. ")"
			limitBox.Text = ""
			-- Remove partículas excedentes se o novo limite for menor
			while #activeParticles > particleLimit do
				local oldestSnowflake = table.remove(activeParticles, 1)
				if oldestSnowflake and oldestSnowflake.Parent then
					oldestSnowflake:Destroy()
				end
			end
		end
	end
end)

heightBox.FocusLost:Connect(function(enterPressed)
	if enterPressed then
		local value = tonumber(heightBox.Text)
		if value then
			spawnHeight = math.max(value, 1)
			heightLabel.Text = "Spawn Height: " .. spawnHeight
			heightBox.Text = ""
		end
	end
end)

distanceBox.FocusLost:Connect(function(enterPressed)
	if enterPressed then
		local value = tonumber(distanceBox.Text)
		if value then
			forwardDistance = math.max(value, 1)
			distanceLabel.Text = "Distance: " .. forwardDistance
			distanceBox.Text = ""
		end
	end
end)

speedBox.FocusLost:Connect(function(enterPressed)
	if enterPressed then
		local value = tonumber(speedBox.Text)
		if value then
			fallSpeed = math.max(value, 1)
			speedLabel.Text = "Fall Speed: " .. fallSpeed
			speedBox.Text = ""
		end
	end
end)

sizeBox.FocusLost:Connect(function(enterPressed)
	if enterPressed then
		local value = tonumber(sizeBox.Text)
		if value then
			snowflakeSize = math.clamp(value, 0.1, 5) -- Limite entre 0.1 e 5
			sizeLabel.Text = "Snowflake Size: " .. snowflakeSize
			sizeBox.Text = ""
		end
	end
end)
