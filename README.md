local Players = game:GetService("Players")
local Camera = workspace.CurrentCamera
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

local espEnabled = false
local dragging = false
local offset = Vector2.new(0, 0)

local espBoxes = {}

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(0, 200, 0, 50)
toggleButton.Position = UDim2.new(0, 10, 0, 10)
toggleButton.Text = "Ativar ESP"
toggleButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
toggleButton.Parent = screenGui

function drawESP(player)
    local character = player.Character
    if not character or not character:FindFirstChild("Head") then return end

    local head = character.Head
    local screenPos = Camera:WorldToViewportPoint(head.Position)
    local x, y = screenPos.X, screenPos.Y

    local isEnemy = player.Team ~= LocalPlayer.Team
    local color = isEnemy and Color3.fromRGB(255, 0, 0) or Color3.fromRGB(255, 255, 255)

    local size = 50
    local espBox = Instance.new("Frame")
    espBox.Size = UDim2.new(0, size, 0, size)
    espBox.Position = UDim2.new(0, x - size / 2, 0, y - size / 2)
    espBox.BackgroundColor3 = color
    espBox.BackgroundTransparency = 0.5
    espBox.Parent = screenGui

    espBoxes[player] = espBox
end

function removeESP(player)
    if espBoxes[player] then
        espBoxes[player]:Destroy()
        espBoxes[player] = nil
    end
end

toggleButton.MouseButton1Click:Connect(function()
    espEnabled = not espEnabled
    if espEnabled then
        toggleButton.Text = "Desativar ESP"
        toggleButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer then
                drawESP(player)
            end
        end
    else
        toggleButton.Text = "Ativar ESP"
        toggleButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        for player, espBox in pairs(espBoxes) do
            removeESP(player)
        end
    end
end)

toggleButton.MouseButton1Down:Connect(function(input)
    dragging = true
    offset = input.Position - toggleButton.Position
end)

toggleButton.MouseButton1Up:Connect(function()
    dragging = false
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging then
        toggleButton.Position = UDim2.new(0, input.Position.X - offset.X, 0, input.Position.Y - offset.Y)
    end
end)

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        if espEnabled then
            drawESP(player)
        end
    end)
end)

Players.PlayerRemoving:Connect(function(player)
    removeESP(player)
end)

for _, player in pairs(Players:GetPlayers()) do
    if player ~= LocalPlayer and player.Character then
        drawESP(player)
    end
end
