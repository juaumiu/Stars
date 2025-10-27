--[[ Five Nights Hunted - HUB Script
Desenvolvido para fins educacionais e uso em projetos próprios.
Inclui:
- HUB com botão para abrir/fechar (tecla H)
- Wallhack (visualizar jogadores atrás de paredes)
- Noclip (atravessar paredes)
- Aumentar/diminuir velocidade
]]

local UIS = game:GetService("UserInputService")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local Camera = workspace.CurrentCamera

-- HUB GUI
local ScreenGui = Instance.new("ScreenGui", LocalPlayer.PlayerGui)
ScreenGui.Name = "FiveNightsHuntedHub"

local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 200, 0, 200)
Frame.Position = UDim2.new(0.5, -100, 0.5, -100)
Frame.BackgroundTransparency = 0.2
Frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Frame.Visible = false

local function makeButton(text, y, callback)
    local btn = Instance.new("TextButton", Frame)
    btn.Size = UDim2.new(1, -20, 0, 40)
    btn.Position = UDim2.new(0, 10, 0, y)
    btn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Text = text
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 18
    btn.MouseButton1Click:Connect(callback)
    return btn
end

-- HUB State
local wallhackEnabled = false
local noclipEnabled = false
local speed = 16

makeButton("Wallhack: OFF", 10, function(btn)
    wallhackEnabled = not wallhackEnabled
    btn.Text = "Wallhack: " .. (wallhackEnabled and "ON" or "OFF")
end)

makeButton("Noclip: OFF", 60, function(btn)
    noclipEnabled = not noclipEnabled
    btn.Text = "Noclip: " .. (noclipEnabled and "ON" or "OFF")
end)

makeButton("Speed +", 110, function()
    speed = speed + 4
    Humanoid.WalkSpeed = speed
end)

makeButton("Speed -", 160, function()
    speed = math.max(4, speed - 4)
    Humanoid.WalkSpeed = speed
end)

-- Toggle HUB
UIS.InputBegan:Connect(function(input, processed)
    if not processed and input.KeyCode == Enum.KeyCode.H then
        Frame.Visible = not Frame.Visible
    end
end)

-- Wallhack (Highlight Players)
local highlights = {}
RunService.RenderStepped:Connect(function()
    if wallhackEnabled then
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                if not highlights[player] then
                    local highlight = Instance.new("Highlight")
                    highlight.Adornee = player.Character
                    highlight.FillColor = Color3.fromRGB(255, 0, 0)
                    highlight.OutlineColor = Color3.fromRGB(255, 255, 0)
                    highlight.Parent = player.Character
                    highlights[player] = highlight
                end
            end
        end
    else
        for player, highlight in pairs(highlights) do
            if highlight and highlight.Parent then highlight:Destroy() end
            highlights[player] = nil
        end
    end
end)

-- Noclip
RunService.Stepped:Connect(function()
    if noclipEnabled and Character then
        for _, part in pairs(Character:GetDescendants()) do
            if part:IsA("BasePart") and part.CanCollide then
                part.CanCollide = false
            end
        end
    elseif Character then
        for _, part in pairs(Character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = true
            end
        end
    end
end)

-- Atualização de personagem (reset, etc)
LocalPlayer.CharacterAdded:Connect(function(char)
    Character = char
    Humanoid = char:WaitForChild("Humanoid")
    Humanoid.WalkSpeed = speed
end)

-- Inicialização da velocidade
Humanoid.WalkSpeed = speed
