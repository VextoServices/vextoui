local Vexto = {}

-- Services
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

-- Local player and PlayerGui
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Create ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "VextoGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

-- Constants
local ANIMATION_INFO = TweenInfo.new(0.3, Enum.EasingStyle.Quint, Enum.EasingDirection.Out)
local PRIMARY_COLOR = Color3.fromRGB(10, 10, 20)
local ACCENT_COLOR = Color3.fromRGB(0, 220, 255)
local SECONDARY_COLOR = Color3.fromRGB(20, 20, 30)
local GLOW_COLOR = Color3.fromRGB(0, 255, 255)

-- Utility function to create a frame
local function createFrame(parent, name, size, position, backgroundColor, cornerRadius, zIndex)
	local frame = Instance.new("Frame")
	frame.Name = name
	frame.Size = size or UDim2.new(0, 100, 0, 50)
	frame.Position = position or UDim2.new(0, 0, 0, 0)
	frame.BackgroundColor3 = backgroundColor or PRIMARY_COLOR
	frame.BorderSizePixel = 0
	frame.ZIndex = zIndex or 1
	frame.Parent = parent

	-- Corner radius
	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, cornerRadius or 12)
	corner.Parent = frame

	-- Dynamic glow effect
	local stroke = Instance.new("UIStroke")
	stroke.Thickness = 2
	stroke.Color = GLOW_COLOR
	stroke.Transparency = 0.5
	local gradient = Instance.new("UIGradient")
	gradient.Color = ColorSequence.new({
		ColorSequenceKeypoint.new(0, GLOW_COLOR),
		ColorSequenceKeypoint.new(0.5, Color3.fromRGB(255, 0, 255)),
		ColorSequenceKeypoint.new(1, GLOW_COLOR)
	})
	gradient.Rotation = 45
	gradient.Parent = stroke
	stroke.Parent = frame

	return frame
end

-- Utility function to add gradient
local function addGradient(parent, colorSequence)
	local gradient = Instance.new("UIGradient")
	gradient.Color = colorSequence or ColorSequence.new({
		ColorSequenceKeypoint.new(0, Color3.fromRGB(20, 20, 40)),
		ColorSequenceKeypoint.new(1, Color3.fromRGB(5, 5, 15))
	})
	gradient.Rotation = 90
	gradient.Parent = parent
end

-- Utility function to create text label
local function createTextLabel(parent, name, text, size, position, textColor, fontWeight, zIndex)
	local label = Instance.new("TextLabel")
	label.Name = name
	label.Size = size or UDim2.new(1, 0, 0, 24)
	label.Position = position or UDim2.new(0, 0, 0, 0)
	label.BackgroundTransparency = 1
	label.Text = text
	label.TextColor3 = textColor or Color3.fromRGB(240, 240, 255)
	label.TextScaled = false
	label.TextSize = 20
	label.FontFace = Font.new("rbxasset://fonts/families/Gotham.json", fontWeight or Enum.FontWeight.Bold)
	label.ZIndex = zIndex or 1
	label.Parent = parent
	return label
end

-- Utility function to create text box
local function createTextBox(parent, name, placeholder, size, position, backgroundColor, textColor, zIndex)
	local textBoxFrame = createFrame(parent, name, size, position, backgroundColor or SECONDARY_COLOR, 10, zIndex)
	local textBox = Instance.new("TextBox")
	textBox.Name = "Input"
	textBox.Size = UDim2.new(1, -20, 1, -10)
	textBox.Position = UDim2.new(0, 10, 0, 5)
	textBox.BackgroundTransparency = 1
	textBox.Text = ""
	textBox.PlaceholderText = placeholder
	textBox.TextColor3 = textColor or Color3.fromRGB(240, 240, 255)
	textBox.TextScaled = false
	textBox.TextSize = 18
	textBox.FontFace = Font.new("rbxasset://fonts/families/Gotham.json", Enum.FontWeight.Medium)
	textBox.ZIndex = zIndex or 1
	textBox.Parent = textBoxFrame
	return textBoxFrame, textBox
end

-- Utility function to create button
local function createButton(parent, name, text, size, position, backgroundColor, textColor, onClick, zIndex)
	local buttonFrame = createFrame(parent, name, size, position, backgroundColor or ACCENT_COLOR, 10, zIndex)
	addGradient(buttonFrame)

	-- Button text
	local textLabel = createTextLabel(buttonFrame, "ButtonText", text, UDim2.new(1, -20, 1, 0), UDim2.new(0, 10, 0, 0), textColor, Enum.FontWeight.Medium, zIndex)

	-- Clickable area
	local textButton = Instance.new("TextButton")
	textButton.Name = "ClickArea"
	textButton.Size = UDim2.new(1, 0, 1, 0)
	textButton.BackgroundTransparency = 1
	textButton.Text = ""
	textButton.ZIndex = zIndex or 1
	textButton.Parent = buttonFrame

	-- Animations
	local originalSize = buttonFrame.Size
	local originalColor = buttonFrame.BackgroundColor3
	textButton.MouseEnter:Connect(function()
		TweenService:Create(buttonFrame, ANIMATION_INFO, {
			Size = UDim2.new(originalSize.X.Scale, originalSize.X.Offset + 5, originalSize.Y.Scale, originalSize.Y.Offset + 5),
			BackgroundColor3 = Color3.fromRGB(50, 255, 255)
		}):Play()
	end)
	textButton.MouseLeave:Connect(function()
		TweenService:Create(buttonFrame, ANIMATION_INFO, {
			Size = originalSize,
			BackgroundColor3 = originalColor
		}):Play()
	end)

	textButton.MouseButton1Down:Connect(function()
		TweenService:Create(buttonFrame, TweenInfo.new(0.1), {BackgroundColor3 = Color3.fromRGB(0, 180, 255)}):Play()
	end)
	textButton.MouseButton1Up:Connect(function()
		TweenService:Create(buttonFrame, TweenInfo.new(0.1), {BackgroundColor3 = originalColor}):Play()
		onClick()
	end)

	return buttonFrame
end

-- Utility function to create toggle
local function createToggle(parent, name, text, size, position, onToggle, zIndex)
	local toggleFrame = createFrame(parent, name, size or UDim2.new(0, 300, 0, 50), position, SECONDARY_COLOR, 10, zIndex)

	-- Toggle label
	local label = createTextLabel(toggleFrame, "ToggleText", text, UDim2.new(0.7, 0, 1, 0), UDim2.new(0, 15, 0, 0), Color3.fromRGB(240, 240, 255), Enum.FontWeight.Medium, zIndex)
	label.TextSize = 18

	-- Toggle switch
	local switch = createFrame(toggleFrame, "Switch", UDim2.new(0, 60, 0, 30), UDim2.new(0.85, -70, 0.5, -15), Color3.fromRGB(50, 50, 60), 15, zIndex)
	local switchKnob = createFrame(switch, "Knob", UDim2.new(0.5, -4, 1, -4), UDim2.new(0, 2, 0, 2), Color3.fromRGB(255, 255, 255), 15, zIndex + 1)

	local isToggled = false

	-- Clickable area
	local textButton = Instance.new("TextButton")
	textButton.Name = "ToggleArea"
	textButton.Size = UDim2.new(1, 0, 1, 0)
	textButton.BackgroundTransparency = 1
	textButton.Text = ""
	textButton.ZIndex = zIndex or 1
	textButton.Parent = toggleFrame

	-- Toggle logic
	local function updateToggle()
		local targetPos = isToggled and UDim2.new(0.5, 0, 0, 2) or UDim2.new(0, 2, 0, 2)
		local targetColor = isToggled and ACCENT_COLOR or Color3.fromRGB(50, 50, 60)
		TweenService:Create(switchKnob, ANIMATION_INFO, {Position = targetPos}):Play()
		TweenService:Create(switch, ANIMATION_INFO, {BackgroundColor3 = targetColor}):Play()
		onToggle(isToggled)
	end

	textButton.MouseButton1Click:Connect(function()
		isToggled = not isToggled
		updateToggle()
	end)

	updateToggle()
	return toggleFrame
end

-- Utility function to create dropdown
local function createDropdown(parent, name, items, size, position, onSelect, zIndex)
	local dropdownFrame = createFrame(parent, name, size or UDim2.new(0, 300, 0, 50), position, SECONDARY_COLOR, 10, zIndex)
	local isOpen = false

	-- Selected item
	local selectedLabel = createTextLabel(dropdownFrame, "Selected", items[1] or "Select...", UDim2.new(1, -50, 1, 0), UDim2.new(0, 15, 0, 0), Color3.fromRGB(240, 240, 255), Enum.FontWeight.Medium, zIndex)
	selectedLabel.TextSize = 18

	-- Dropdown arrow
	local arrow = createTextLabel(dropdownFrame, "Arrow", "▼", UDim2.new(0, 40, 1, 0), UDim2.new(1, -45, 0, 0), Color3.fromRGB(240, 240, 255), nil, zIndex)
	arrow.TextSize = 20

	-- Dropdown menu
	local menu = createFrame(dropdownFrame, "Menu", UDim2.new(1, 0, 0, #items * 50), UDim2.new(0, 0, 1, 5), SECONDARY_COLOR, 10, zIndex + 1)
	menu.Visible = false
	menu.ClipsDescendants = true

	-- Populate menu
	for i, item in ipairs(items) do
		createButton(menu, "Item" .. item, item, UDim2.new(1, 0, 0, 50), UDim2.new(0, 0, 0, (i-1)*50), ACCENT_COLOR, Color3.fromRGB(240, 240, 255), function()
			selectedLabel.Text = item
			isOpen = false
			menu.Visible = false
			TweenService:Create(arrow, ANIMATION_INFO, {Rotation = 0}):Play()
			onSelect(item)
		end, zIndex + 1)
	end

	-- Toggle menu
	local textButton = Instance.new("TextButton")
	textButton.Name = "DropdownArea"
	textButton.Size = UDim2.new(1, 0, 1, 0)
	textButton.BackgroundTransparency = 1
	textButton.Text = ""
	textButton.ZIndex = zIndex or 1
	textButton.Parent = dropdownFrame

	textButton.MouseButton1Click:Connect(function()
		isOpen = not isOpen
		menu.Visible = isOpen
		TweenService:Create(arrow, ANIMATION_INFO, {Rotation = isOpen and 180 or 0}):Play()
	end)

	return dropdownFrame
end

-- Create login panel
local function createLoginPanel(onLoginSuccess)
	local loginFrame = createFrame(screenGui, "LoginPanel", UDim2.new(0, 350, 0, 280), UDim2.new(0.5, -175, 0.5, -140), PRIMARY_COLOR, 12, 3)
	addGradient(loginFrame)

	-- Shadow
	local shadow = createFrame(screenGui, "LoginShadow", UDim2.new(0, 370, 0, 300), UDim2.new(0.5, -185, 0.5, -150), Color3.fromRGB(0, 0, 0), 12, 2)
	shadow.BackgroundTransparency = 0.5

	-- Title
	createTextLabel(loginFrame, "LoginTitle", "Vexto Login", UDim2.new(1, -20, 0, 50), UDim2.new(0, 10, 0, 10), Color3.fromRGB(255, 255, 255), Enum.FontWeight.Bold, 3)

	-- Username input
	local usernameFrame, usernameBox = createTextBox(loginFrame, "UsernameInput", "Username", UDim2.new(0, 310, 0, 45), UDim2.new(0, 20, 0, 70), SECONDARY_COLOR, nil, 3)

	-- Password input
	local passwordFrame, passwordBox = createTextBox(loginFrame, "PasswordInput", "Password", UDim2.new(0, 310, 0, 45), UDim2.new(0, 20, 0, 125), SECONDARY_COLOR, nil, 3)
	passwordBox.TextEditable = true
	passwordBox.ClearTextOnFocus = false

	-- Login button
	createButton(loginFrame, "LoginButton", "Login", UDim2.new(0, 310, 0, 50), UDim2.new(0, 20, 0, 190), ACCENT_COLOR, Color3.fromRGB(255, 255, 255), function()
		if usernameBox.Text ~= "" and passwordBox.Text ~= "" then
			TweenService:Create(loginFrame, ANIMATION_INFO, {Position = UDim2.new(0.5, -175, -1, -140), BackgroundTransparency = 1}):Play()
			TweenService:Create(shadow, ANIMATION_INFO, {BackgroundTransparency = 1}):Play()
			wait(0.3)
			loginFrame.Visible = false
			shadow.Visible = false
			onLoginSuccess()
		else
			TweenService:Create(loginFrame, TweenInfo.new(0.1, Enum.EasingStyle.Linear), {Position = UDim2.new(0.5, -165, 0.5, -140)}):Play()
			wait(0.05)
			TweenService:Create(loginFrame, TweenInfo.new(0.1, Enum.EasingStyle.Linear), {Position = UDim2.new(0.5, -175, 0.5, -140)}):Play()
		end
	end, 3)

	-- Fade-in animation
	loginFrame.BackgroundTransparency = 1
	shadow.BackgroundTransparency = 1
	TweenService:Create(loginFrame, TweenInfo.new(0.6, Enum.EasingStyle.Sine), {BackgroundTransparency = 0}):Play()
	TweenService:Create(shadow, TweenInfo.new(0.6, Enum.EasingStyle.Sine), {BackgroundTransparency = 0.5}):Play()

	return loginFrame, shadow
end

-- Create main UI window
function Vexto.CreateUI()
	-- Main window (initially hidden)
	local mainWindow = createFrame(screenGui, "VextoWindow", UDim2.new(0, 550, 0, 450), UDim2.new(0.5, -275, 0.5, -225), PRIMARY_COLOR, 12, 2)
	mainWindow.BackgroundTransparency = 1
	mainWindow.Visible = false
	addGradient(mainWindow)

	-- Shadow (initially hidden)
	local shadow = createFrame(screenGui, "Shadow", UDim2.new(0, 570, 0, 470), UDim2.new(0.5, -285, 0.5, -235), Color3.fromRGB(0, 0, 0), 12, 1)
	shadow.BackgroundTransparency = 1
	shadow.Visible = false

	-- Shadow animation
	local function pulseShadow()
		TweenService:Create(shadow, TweenInfo.new(2, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true), {
			BackgroundTransparency = 0.7
		}):Play()
	end

	-- Title bar
	local titleBar = createFrame(mainWindow, "TitleBar", UDim2.new(1, 0, 0, 60), UDim2.new(0, 0, 0, 0), Color3.fromRGB(5, 5, 15), 0, 2)
	createTextLabel(titleBar, "Title", "Vexto - Tha Bronx 3", UDim2.new(1, -20, 1, 0), UDim2.new(0, 15, 0, 0), Color3.fromRGB(0, 220, 255), Enum.FontWeight.Bold, 2)

	-- Tab container (horizontal at top)
	local tabContainer = createFrame(mainWindow, "TabContainer", UDim2.new(1, -30, 0, 50), UDim2.new(0, 15, 0, 70), Color3.fromRGB(5, 5, 15), 8, 2)

	-- Content area
	local contentArea = createFrame(mainWindow, "ContentArea", UDim2.new(1, -30, 1, -130), UDim2.new(0, 15, 0, 130), PRIMARY_COLOR, 8, 2)
	contentArea.ClipsDescendants = true

	-- Tab system
	local tabs = {}
	local currentTab = nil

	local function createTab(name, contentFrame)
		local tabButton = createButton(tabContainer, "Tab" .. name, name, UDim2.new(0, 120, 0, 40), UDim2.new(0, (#tabs * 130) + 10, 0, 5), Color3.fromRGB(5, 5, 15), Color3.fromRGB(240, 240, 255), function()
			if currentTab then
				TweenService:Create(currentTab, ANIMATION_INFO, {BackgroundTransparency = 1}):Play()
				wait(0.15)
				currentTab.Visible = false
			end
			contentFrame.BackgroundTransparency = 1
			contentFrame.Visible = true
			TweenService:Create(contentFrame, ANIMATION_INFO, {BackgroundTransparency = 0}):Play()
			currentTab = contentFrame
		end, 2)
		table.insert(tabs, contentFrame)
	end

	-- Main tab
	local mainTab = createFrame(contentArea, "MainTab", UDim2.new(1, 0, 1, 0), UDim2.new(0, 0, 0, 0), PRIMARY_COLOR, 8, 2)
	mainTab.Visible = false

	-- Auto-farm toggle
	createToggle(mainTab, "AutoFarmToggle", "Auto Farm", UDim2.new(0, 300, 0, 50), UDim2.new(0.5, -150, 0, 20), function(state)
		print("Auto Farm: " .. (state and "Enabled" or "Disabled"))
		-- Add Tha Bronx 3 auto-farm logic here
	end, 2)

	-- Infinite money button
	createButton(mainTab, "MoneyButton", "Get Infinite Money", UDim2.new(0, 300, 0, 50), UDim2.new(0.5, -150, 0, 90), ACCENT_COLOR, Color3.fromRGB(255, 255, 255), function()
		print("Infinite Money activated!")
		-- Add Tha Bronx 3 money script logic here
	end, 2)

	-- Teleports tab
	local teleportsTab = createFrame(contentArea, "TeleportsTab", UDim2.new(1, 0, 1, 0), UDim2.new(0, 0, 0, 0), PRIMARY_COLOR, 8, 2)
	teleportsTab.Visible = false

	-- Teleport dropdown
	local teleportLocations = {"Bank", "Store", "Safe Zone", "Police Station", "Club"}
	createDropdown(teleportsTab, "TeleportDropdown", teleportLocations, UDim2.new(0, 300, 0, 50), UDim2.new(0.5, -150, 0, 20), function(location)
		print("Teleporting to " .. location)
		-- Add Tha Bronx 3 teleport logic here
	end, 2)

	-- Create tabs
	createTab("Main", mainTab)
	createTab("Teleports", teleportsTab)

	-- Set default tab
	mainTab.Visible = true
	currentTab = mainTab

	-- Draggable window (for both mouse and touch)
	local dragging = false
	local dragInput = nil
	local dragStart = nil
	local startPos = nil

	local function updateInput(input)
		local delta = input.Position - dragStart
		mainWindow.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
		shadow.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X - 10, startPos.Y.Scale, startPos.Y.Offset + delta.Y - 10)
	end

	-- Mouse dragging
	titleBar.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = true
			dragStart = input.Position
			startPos = mainWindow.Position
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragging = false
				end
			end)
		end
	end)

	titleBar.InputChanged:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseMovement then
			dragInput = input
		end
	end)

	-- Touch dragging
	titleBar.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.Touch then
			dragging = true
			dragStart = input.Position
			startPos = mainWindow.Position
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragging = false
				end
			end)
		end
	end)

	titleBar.InputChanged:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.Touch then
			dragInput = input
		end
	end)

	UserInputService.InputChanged:Connect(function(input)
		if (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) and dragging then
			updateInput(input)
		end
	end)

	-- Create login panel
	local loginFrame, loginShadow = createLoginPanel(function()
		-- Show main UI after login
		mainWindow.Visible = true
		shadow.Visible = true
		TweenService:Create(mainWindow, TweenInfo.new(0.6, Enum.EasingStyle.Sine), {BackgroundTransparency = 0}):Play()
		TweenService:Create(shadow, TweenInfo.new(0.6, Enum.EasingStyle.Sine), {BackgroundTransparency = 0.5}):Play()
		coroutine.wrap(pulseShadow)()
	end)

	return mainWindow
end

-- Initialize UI
local mainUI = Vexto.CreateUI()

-- Toggle UI visibility (H key)
local isVisible = true
UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if not gameProcessed and input.KeyCode == Enum.KeyCode.H then
		isVisible = not isVisible
		TweenService:Create(mainUI, ANIMATION_INFO, {Position = UDim2.new(0.5, -275, isVisible and 0.5 or -1, isVisible and -225 or -500)}):Play()
	end
end)

return Vexto
