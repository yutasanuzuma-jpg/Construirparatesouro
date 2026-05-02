-- Script Build A Boat - PARADA NA CACHOEIRA + RESET (DISTÂNCIA SEGURA)
local ScreenGui = Instance.new("ScreenGui")
local MainFrame = Instance.new("Frame")
local Title = Instance.new("TextLabel")
local CloseButton = Instance.new("TextButton")
local StartButton = Instance.new("TextButton")
local LagFixButton = Instance.new("TextButton")
local StopButton = Instance.new("TextButton")

ScreenGui.Name = "BABFT_Cachoeira_Only"
ScreenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
ScreenGui.ResetOnSpawn = false

-- Interface Visual
MainFrame.Name = "MainFrame"
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
MainFrame.Position = UDim2.new(0.5, -100, 0.5, -100)
MainFrame.Size = UDim2.new(0, 220, 0, 230)
MainFrame.Active = true
Instance.new("UICorner", MainFrame)

Title.Parent = MainFrame
Title.Size = UDim2.new(1, 0, 0, 35)
Title.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Title.Text = "  FARM CACHOEIRA"
Title.TextColor3 = Color3.fromRGB(255, 255, 0)
Title.TextXAlignment = Enum.TextXAlignment.Left
Instance.new("UICorner", Title)

CloseButton.Parent = MainFrame
CloseButton.Position = UDim2.new(1, -30, 0, 5)
CloseButton.Size = UDim2.new(0, 25, 0, 25)
CloseButton.Text = "X"
CloseButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
CloseButton.TextColor3 = Color3.fromRGB(255, 255, 255)

StartButton.Parent = MainFrame
StartButton.Position = UDim2.new(0.1, 0, 0.25, 0)
StartButton.Size = UDim2.new(0.8, 0, 0, 35)
StartButton.Text = "INICIAR FARM"
StartButton.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
Instance.new("UICorner", StartButton)

LagFixButton.Parent = MainFrame
LagFixButton.Position = UDim2.new(0.1, 0, 0.45, 0)
LagFixButton.Size = UDim2.new(0.8, 0, 0, 35)
LagFixButton.Text = "ULTRA LAG FIX"
LagFixButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
Instance.new("UICorner", LagFixButton)

StopButton.Parent = MainFrame
StopButton.Position = UDim2.new(0.1, 0, 0.65, 0)
StopButton.Size = UDim2.new(0.8, 0, 0, 35)
StopButton.Text = "PARAR"
StopButton.BackgroundColor3 = Color3.fromRGB(200, 0, 0)
Instance.new("UICorner", StopButton)

-- [ AUTO CLAIM INSTANTÂNEO ] --
game:GetService("RunService").RenderStepped:Connect(function()
    if _G.AutoFarmAtivo then
        local pGui = game.Players.LocalPlayer:FindFirstChild("PlayerGui")
        if pGui then
            for _, v in pairs(pGui:GetDescendants()) do
                if v:IsA("TextButton") and v.Visible and (v.Text:find("Claim") or v.Text:find("Reivindicar") or v.Name == "ClaimButton") then
                    firesignal(v.MouseButton1Click)
                end
            end
        end
    end
end)

-- [ MOVIMENTO ] --
_G.AutoFarmAtivo = false
local currentTween = nil

game:GetService("RunService").Stepped:Connect(function()
    if _G.AutoFarmAtivo then
        local char = game.Players.LocalPlayer.Character
        if char then
            for _, part in pairs(char:GetDescendants()) do
                if part:IsA("BasePart") then part.CanCollide = false end
            end
        end
    end
end)

local function iniciarTrajeto()
    if not _G.AutoFarmAtivo then return end
    local player = game.Players.LocalPlayer
    local char = player.Character or player.CharacterAdded:Wait()
    local root = char:WaitForChild("HumanoidRootPart")
    local hum = char:WaitForChild("Humanoid")
    
    local eixoX = -55 
    local altura = 115
    local velocidade = 380

    root.CFrame = CFrame.new(eixoX, altura, root.Position.Z)
    wait(0.5)

    -- ROTA ATÉ A ENTRADA DA CACHOEIRA (SEM CHEGAR NO BAÚ)
    local stages = {
        {eixoX, altura, 2000}, {eixoX, altura, 4000}, {eixoX, altura, 6000}, 
        {eixoX, altura, 8000}, 
        {eixoX, altura, 9000} -- PARA AQUI: Na boca da cachoeira
    }

    local bv = Instance.new("BodyVelocity")
    bv.Velocity = Vector3.new(0,0,0)
    bv.MaxForce = Vector3.new(9e9, 9e9, 9e9)
    bv.Parent = root

    for _, pos in pairs(stages) do
        if not _G.AutoFarmAtivo then break end
        local target = CFrame.new(pos[1], pos[2], pos[3])
        currentTween = game:GetService("TweenService"):Create(root, TweenInfo.new((root.Position - target.Position).Magnitude / velocidade, Enum.EasingStyle.Linear), {CFrame = target})
        currentTween:Play()
        currentTween.Completed:Wait()
    end
    
    -- RESET NA CACHOEIRA (Longe do Baú)
    if _G.AutoFarmAtivo and root.Position.Z >= 8900 then
        wait(1.5) -- Garante que o último portão contou o ouro
        hum.Health = 0 
    end
end

-- Reinício automático seguro
game.Players.LocalPlayer.CharacterAdded:Connect(function()
    if _G.AutoFarmAtivo then
        wait(5) 
        iniciarTrajeto()
    end
end)

StartButton.MouseButton1Click:Connect(function()
    if _G.AutoFarmAtivo then return end
    _G.AutoFarmAtivo = true
    iniciarTrajeto()
end)

LagFixButton.MouseButton1Click:Connect(function()
    -- Redução de Lag Hardcore
    game:GetService("Lighting").GlobalShadows = false
    game:GetService("Lighting").Brightness = 0
    for _, v in pairs(game:GetDescendants()) do
        if v:IsA("BasePart") then v.Material = Enum.Material.Plastic v.Reflectance = 0
        elseif v:IsA("Decal") or v:IsA("Texture") then v:Destroy() end
    end
end)

StopButton.MouseButton1Click:Connect(function()
    _G.AutoFarmAtivo = false
    if currentTween then currentTween:Cancel() end
end)

CloseButton.MouseButton1Click:Connect(function()
    _G.AutoFarmAtivo = false
    ScreenGui:Destroy()
end)

-- Mobile Drag
local UIS = game:GetService("UserInputService")
local dragToggle, dragStart, startPos
MainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch then
        dragToggle = true; dragStart = input.Position; startPos = MainFrame.Position
    end
end)
UIS.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch and dragToggle then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
UIS.InputEnded:Connect(function(input) if input.UserInputType == Enum.UserInputType.Touch then dragToggle = false end end)
