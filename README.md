--[[ ROBLOX UNIVERSAL AIMBOT + ESP (por Loadstring)
 - Aimbot com auto-mira e auto-disparo
 - ESP com Nome, Distância e Vida (cor da equipa)
 - Compatível com loadstring (GitHub, Pastebin, etc)
]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local Mouse = LocalPlayer:GetMouse()

-- Configurações
local FOV_RADIUS = 150
local AUTO_AIM = true
local AUTO_SHOOT = true
local SHOOT_COOLDOWN = 0.2
local lastShot = 0
local ESPs = {}

-- Ajuste do FOV
UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.PageUp then
        FOV_RADIUS += 25
        print("FOV aumentado:", FOV_RADIUS)
    elseif input.KeyCode == Enum.KeyCode.PageDown then
        FOV_RADIUS = math.max(25, FOV_RADIUS - 25)
        print("FOV diminuído:", FOV_RADIUS)
    end
end)

-- Criar ESP para jogador
local function criarESP(player)
    if ESPs[player] then return end
    local esp = Drawing.new("Text")
    esp.Size = 13
    esp.Center = true
    esp.Outline = true
    esp.Font = 2
    esp.Visible = false
    ESPs[player] = esp
end

-- Remover ESP
local function removerESP(player)
    if ESPs[player] then
        ESPs[player]:Remove()
        ESPs[player] = nil
    end
end

-- Aplicar ESP a jogadores existentes
for _, p in pairs(Players:GetPlayers()) do
    if p ~= LocalPlayer then
        criarESP(p)
    end
end
Players.PlayerAdded:Connect(criarESP)
Players.PlayerRemoving:Connect(removerESP)

-- Encontra inimigo mais próximo do mouse
local function getClosestEnemy()
    local closest, shortest = nil, FOV_RADIUS
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Team ~= LocalPlayer.Team then
            local char = player.Character
            if char and char:FindFirstChild("HumanoidRootPart") then
                local pos = char.HumanoidRootPart.Position
                local screenPoint, onScreen = Camera:WorldToViewportPoint(pos)
                if onScreen then
                    local dist = (Vector2.new(screenPoint.X, screenPoint.Y) - Vector2.new(Mouse.X, Mouse.Y)).Magnitude
                    if dist < shortest then
                        shortest = dist
                        closest = player
                    end
                end
            end
        end
    end
    return closest
end

-- Loop do Aimbot + ESP
RunService.RenderStepped:Connect(function()
    -- ESP update
    for player, esp in pairs(ESPs) do
        local char = player.Character
        if char and char:FindFirstChild("HumanoidRootPart") and char:FindFirstChild("Humanoid") then
            local pos = char.HumanoidRootPart.Position
            local screenPoint, onScreen = Camera:WorldToViewportPoint(pos)
            if onScreen then
                local dist = math.floor((pos - Camera.CFrame.Position).Magnitude)
                local vida = math.floor(char.Humanoid.Health)
                esp.Text = string.format("[%s] %dm | %dhp", player.Name, dist, vida)
                esp.Position = Vector2.new(screenPoint.X, screenPoint.Y - 15)
                esp.Color = player.Team and player.Team.TeamColor.Color or Color3.new(1, 1, 1)
                esp.Visible = true
            else
                esp.Visible = false
            end
        else
            esp.Visible = false
        end
    end

    -- Aimbot + Auto-shoot
    if AUTO_AIM then
        local target = getClosestEnemy()
        if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
            local aimPart = target.Character:FindFirstChild("Head") or target.Character.HumanoidRootPart
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, aimPart.Position)

            if AUTO_SHOOT and tick() - lastShot >= SHOOT_COOLDOWN then
                VirtualInputManager:SendMouseButtonEvent(Mouse.X, Mouse.Y, 0, true, game, 0)
                VirtualInputManager:SendMouseButtonEvent(Mouse.X, Mouse.Y, 0, false, game, 0)
                lastShot = tick()
            end
        end
    end
end)
