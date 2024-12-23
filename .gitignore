local ESPSettings = {
    BoxFixed = false,       -- Whether the ESP box stays fixed or scales with distance
    aimbotEnabled = false,  -- Default to false
    fov = 100,              -- FOV radius for aimbot
    aimbotKey = Enum.UserInputType.MouseButton2,  -- Key for enabling aimbot
    boxColor = Color3.fromRGB(255, 0, 0),        -- ESP box color
    healthBarColor = Color3.fromRGB(0, 255, 0),  -- Health bar color
    nameColor = Color3.fromRGB(255, 255, 255),   -- Name text color
    healthTextColor = Color3.fromRGB(255, 255, 255), -- Health text color
}

local camera = game.Workspace.CurrentCamera
local players = game:GetService("Players")
local localPlayer = players.LocalPlayer
local userInputService = game:GetService("UserInputService")
local runService = game:GetService("RunService")

-- FOV Circle using Drawing API
local fovCircle = Drawing.new("Circle")
fovCircle.Radius = ESPSettings.fov
fovCircle.Color = Color3.fromRGB(255, 0, 0)
fovCircle.Thickness = 2
fovCircle.Filled = false
fovCircle.Transparency = 1
fovCircle.Visible = true

-- Create Hit/Kill Notification
local hitUI = Instance.new("ScreenGui", localPlayer.PlayerGui)
local hitText = Instance.new("TextLabel", hitUI)
hitText.Size = UDim2.new(0, 200, 0, 50)
hitText.Position = UDim2.new(0.5, -100, 0.9, -25)
hitText.BackgroundTransparency = 1
hitText.TextColor3 = Color3.fromRGB(255, 255, 255)  -- White text
hitText.Font = Enum.Font.GothamBold
hitText.TextSize = 20
hitText.Text = ""
hitText.TextStrokeTransparency = 0.5
hitText.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)

-- Function to log debug messages
local function debugLog(message)
    print("[DEBUG] " .. message)
end

-- Update FOV circle to follow the mouse
runService.RenderStepped:Connect(function()
    local mousePosition = userInputService:GetMouseLocation()
    fovCircle.Position = mousePosition
end)

-- Function to get the closest enemy's head within FOV
local function getClosestEnemyHead()
    local closestTarget = nil
    local closestDistance = math.huge
    local mousePosition = userInputService:GetMouseLocation()

    for _, player in pairs(players:GetPlayers()) do
        if player ~= localPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local head = player.Character.Head
            local screenPos, onScreen = camera:WorldToViewportPoint(head.Position)
            local targetPosition = Vector2.new(screenPos.X, screenPos.Y)
            local distance = (mousePosition - targetPosition).Magnitude
            if onScreen and distance <= fovCircle.Radius and distance < closestDistance then
                closestTarget = player
                closestDistance = distance
            end
        end
    end
    return closestTarget
end

-- Function to lock cursor on the target's head
local function lockCursorToHead(target)
    if target and target.Character and target.Character:FindFirstChild("Head") then
        local headPosition = camera:WorldToViewportPoint(target.Character.Head.Position)
        local mouse = userInputService:GetMouseLocation()
        local delta = Vector2.new(headPosition.X, headPosition.Y) - mouse
        mousemoverel(delta.X, delta.Y)  -- Adjust the cursor position
    else
        debugLog("Failed to lock cursor to target's head")
    end
end

-- Update hit/kill notifications
local function updateHitUI(name, damage)
    hitText.Text = name .. " hit for " .. damage
    wait(3)
    hitText.Text = ""
end

local function updateKillUI(name)
    hitText.Text = "You killed " .. name
    wait(3)
    hitText.Text = ""
end

-- Monitor health changes and log hits/kills
local function listenForDamage()
    for _, player in pairs(players:GetPlayers()) do
        if player ~= localPlayer and player.Character and player.Character:FindFirstChild("Humanoid") then
            local humanoid = player.Character.Humanoid
            humanoid.HealthChanged:Connect(function()
                if humanoid.Health < humanoid.MaxHealth and humanoid.Health > 0 then
                    local damage = humanoid.MaxHealth - humanoid.Health
                    updateHitUI(player.Name, damage)
                elseif humanoid.Health == 0 then
                    updateKillUI(player.Name)
                end
            end)
        end
    end
end

-- Toggle aimbot on key press
userInputService.InputBegan:Connect(function(input)
    if input.UserInputType == ESPSettings.aimbotKey then
        ESPSettings.aimbotEnabled = true
    end
end)

userInputService.InputEnded:Connect(function(input)
    if input.UserInputType == ESPSettings.aimbotKey then
        ESPSettings.aimbotEnabled = false
    end
end)

-- Aimbot logic
runService.RenderStepped:Connect(function()
    if ESPSettings.aimbotEnabled then
        local target = getClosestEnemyHead()
        if target then
            lockCursorToHead(target)
        end
    end
end)

-- Start listening for damage events
listenForDamage()

-- Function to clean up ESP elements
local function cleanupESP(elements)
    for _, element in ipairs(elements) do
        if element then
            element:Remove()
        end
    end
end

-- Function to create ESP for each player
local function createESP(player)
    local elements = {}  -- Local variable to store drawing elements

    -- Create drawing elements for the ESP
    local Rectangle = Drawing.new("Square")
    local HealthBarBackground = Drawing.new("Square")
    local HealthBar = Drawing.new("Square")
    local HealthText = Drawing.new("Text")
    local Name = Drawing.new("Text")

    -- Add elements to cleanup list
    table.insert(elements, Rectangle)
    table.insert(elements, HealthBarBackground)
    table.insert(elements, HealthBar)
    table.insert(elements, HealthText)
    table.insert(elements, Name)

    -- Local variable for connection (to disconnect later)
    local connection

    -- Render logic
    connection = runService.RenderStepped:Connect(function()
        -- Check if player has a valid character and humanoid
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character:FindFirstChild("Humanoid") then
            local rootPart = player.Character.HumanoidRootPart
            local humanoid = player.Character.Humanoid
            local vector, onScreen = camera:WorldToViewportPoint(rootPart.Position)

            -- Only render ESP if the player is on screen
            if onScreen then
                -- Update ESP box size and position
                Rectangle.Size = ESPSettings.BoxFixed and Vector2.new(100, 50) or Vector2.new((1500 / vector.Z) * 2, (2500 / vector.Z) * 2)
                Rectangle.Position = Vector2.new(vector.X - Rectangle.Size.X / 2, vector.Y - Rectangle.Size.Y / 2)
                Rectangle.Visible = true
                Rectangle.Color = ESPSettings.boxColor
                Rectangle.Thickness = 2
                Rectangle.Filled = false

                -- Health bar background
                HealthBarBackground.Size = Vector2.new(5, Rectangle.Size.Y)
                HealthBarBackground.Position = Vector2.new(Rectangle.Position.X - 5 - 5, Rectangle.Position.Y)
                HealthBarBackground.Color = Color3.new(0.2, 0.2, 0.2)
                HealthBarBackground.Filled = true
                HealthBarBackground.Visible = true

                -- Health bar
                local healthPercent = humanoid.Health / humanoid.MaxHealth
                HealthBar.Size = Vector2.new(5, Rectangle.Size.Y * healthPercent)
                HealthBar.Position = Vector2.new(HealthBarBackground.Position.X, HealthBarBackground.Position.Y + (Rectangle.Size.Y - HealthBar.Size.Y))
                HealthBar.Color = ESPSettings.healthBarColor
                HealthBar.Filled = true
                HealthBar.Visible = true

                -- Name above player
                Name.Text = "Name: " .. player.Name
                Name.Size = 20
                Name.Center = true
                Name.Outline = true
                Name.OutlineColor = Color3.new(0, 0, 0)
                Name.Position = Vector2.new(vector.X, vector.Y - Rectangle.Size.Y / 2 - 20)
                Name.Color = ESPSettings.nameColor
                Name.Visible = true

                -- Health text below the name
                HealthText.Text = "Health: " .. math.floor(humanoid.Health)  -- Use math.floor to display an integer value
                HealthText.Size = 16
                HealthText.Center = true
                HealthText.Outline = true
                HealthText.OutlineColor = Color3.new(0, 0, 0)
                HealthText.Position = Vector2.new(vector.X, vector.Y - Rectangle.Size.Y / 2 - 40)  -- Adjust position
                HealthText.Color = ESPSettings.healthTextColor
                HealthText.Visible = true
            else
                -- Hide ESP elements when off-screen
                Rectangle.Visible = false
                HealthBar.Visible = false
                HealthBarBackground.Visible = false
                Name.Visible = false
                HealthText.Visible = false
            end
        else
            -- If player is no longer in the game or has no character, clean up ESP
            cleanupESP(elements)
            connection:Disconnect()
        end
    end)

    -- Monitor player removal (when player leaves the game)
    player.AncestryChanged:Connect(function()
        if not player:IsDescendantOf(game) then
            -- Player left the game, clean up ESP
            cleanupESP(elements)
            connection:Disconnect()
        end
    end)
end

-- Monitor players that are already in the game when the script is executed
for _, player in pairs(players:GetPlayers()) do
    -- If the player already has a character, create the ESP
    if player.Character then
        createESP(player)
    end
    -- Handle player respawn by connecting to CharacterAdded
    player.CharacterAdded:Connect(function()
        -- Clean up ESP and create again after respawn
        cleanupESP(elements)
        createESP(player)
    end)
end

-- Monitor new players who join after the script has started
players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        createESP(player)
    end)
end)
