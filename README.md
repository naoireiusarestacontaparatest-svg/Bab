--==================================================
--   AUTO GET SUPER TELESCOPE  ✨  [v6 - Completo]
--   LocalScript único - StarterPlayer > StarterPlayerScripts
--==================================================

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local player = Players.LocalPlayer

-- GUI (visual neon)
local gui = Instance.new("ScreenGui")
gui.Name = "AutoGetSuperTelescope"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local fundo = Instance.new("Frame")
fundo.Size = UDim2.new(0, 320, 0, 140)
fundo.Position = UDim2.new(0.5, -160, 0.5, -70)
fundo.BackgroundColor3 = Color3.fromRGB(15, 15, 35)
fundo.BorderSizePixel = 0
fundo.ClipsDescendants = true
fundo.Parent = gui
Instance.new("UICorner", fundo).CornerRadius = UDim.new(0, 16)

local gradiente = Instance.new("UIGradient")
gradiente.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(80, 0, 160)),
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(0, 120, 255)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(160, 0, 255))
})
gradiente.Parent = fundo

task.spawn(function()
    while gui.Parent do
        gradiente.Rotation = 0
        TweenService:Create(gradiente, TweenInfo.new(3, Enum.EasingStyle.Linear), {Rotation = 360}):Play()
        task.wait(3)
    end
end)

local borda = Instance.new("UIStroke")
borda.Thickness = 3
borda.Color = Color3.fromRGB(170, 100, 255)
borda.Parent = fundo

local titulo = Instance.new("TextLabel")
titulo.Size = UDim2.new(1, 0, 0, 45)
titulo.Position = UDim2.new(0, 0, 0, 12)
titulo.BackgroundTransparency = 1
titulo.Text = "🔭 AUTO GET SUPER TELESCOPE"
titulo.TextColor3 = Color3.new(1, 1, 1)
titulo.Font = Enum.Font.GothamBold
titulo.TextSize = 17
titulo.Parent = fundo

local sub = Instance.new("TextLabel")
sub.Size = UDim2.new(1, 0, 0, 22)
sub.Position = UDim2.new(0, 0, 0, 50)
sub.BackgroundTransparency = 1
sub.Text = "Iniciando..."
sub.TextColor3 = Color3.fromRGB(190, 170, 255)
sub.Font = Enum.Font.Gotham
sub.TextSize = 13
sub.Parent = fundo

local barraFundo = Instance.new("Frame")
barraFundo.Size = UDim2.new(0.85, 0, 0, 10)
barraFundo.Position = UDim2.new(0.075, 0, 0, 85)
barraFundo.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
barraFundo.BorderSizePixel = 0
barraFundo.Parent = fundo
Instance.new("UICorner", barraFundo).CornerRadius = UDim.new(1, 0)

local barra = Instance.new("Frame")
barra.Size = UDim2.new(0, 0, 1, 0)
barra.BackgroundColor3 = Color3.fromRGB(120, 255, 180)
barra.BorderSizePixel = 0
barra.Parent = barraFundo
Instance.new("UICorner", barra).CornerRadius = UDim.new(1, 0)

local function progresso(pct, txt)
    if txt then sub.Text = txt end
    if pct then
        TweenService:Create(barra, TweenInfo.new(0.5), {Size = UDim2.new(pct, 0, 1, 0)}):Play()
    end
end

--================= FUNÇÕES =================

local noclipAtivo = false

local function getChar()
    return player.Character or player.CharacterAdded:Wait()
end

local function ativarNoclip()
    noclipAtivo = true
    task.spawn(function()
        while noclipAtivo do
            local char = getChar()
            for _, parte in ipairs(char:GetDescendants()) do
                if parte:IsA("BasePart") then
                    parte.CanCollide = false
                end
            end
            task.wait(0.05)
        end
    end)
end

local function desativarNoclip()
    noclipAtivo = false
    local char = getChar()
    for _, parte in ipairs(char:GetDescendants()) do
        if parte:IsA("BasePart") then
            parte.CanCollide = true
        end
    end
end

local function teleportar(pos)
    local char = getChar()
    local hrp = char:WaitForChild("HumanoidRootPart")
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if humanoid and humanoid.Sit then
        humanoid.Sit = false
        task.wait(0.2)
    end
    hrp.CFrame = CFrame.new(pos)
    hrp.Velocity = Vector3.zero
end

local function moverAte(pos, velocidade)
    local char = getChar()
    local hrp = char:WaitForChild("HumanoidRootPart")
    local distancia = (hrp.Position - pos).Magnitude
    local tempo = distancia / (velocidade or 60)

    progresso(nil, "Movendo... " .. math.floor(distancia) .. " studs")

    local tween = TweenService:Create(hrp, TweenInfo.new(tempo, Enum.EasingStyle.Linear), {CFrame = CFrame.new(pos)})
    tween:Play()

    local conexao
    conexao = RunService.Heartbeat:Connect(function()
        local restante = (hrp.Position - pos).Magnitude
        local feito = 1 - (restante / distancia)
        barra.Size = UDim2.new(math.clamp(feito, 0, 1), 0, 1, 0)
    end)

    tween.Completed:Wait()
    conexao:Disconnect()
    hrp.Velocity = Vector3.zero
end

-- Personagem olha para BAIXO (câmera + corpo)
local neckC0Original = nil

local function olharParaBaixo()
    progresso(nil, "Olhando para baixo... ⬇️")
    local char = getChar()
    local camera = workspace.CurrentCamera

    -- Câmera olha para baixo
    local pos = camera.CFrame.Position
    local olhar = CFrame.new(pos, pos + Vector3.new(0, -50, 0.1))
    TweenService:Create(camera, TweenInfo.new(0.4, Enum.EasingStyle.Quad), {CFrame = olhar}):Play()

    -- Cabeça inclina para baixo
    local head = char:FindFirstChild("Head")
    local neck = head and head:FindFirstChild("Neck")
    if neck and neck:IsA("Motor6D") then
        pcall(function()
            neckC0Original = neck.C0
            neck.C0 = neck.C0 * CFrame.Angles(math.rad(60), 0, 0)
        end)
    end
    task.wait(0.5)
end

local function restaurarPescoco()
    local char = getChar()
    local head = char:FindFirstChild("Head")
    local neck = head and head:FindFirstChild("Neck")
    if neck and neckC0Original and neck:IsA("Motor6D") then
        pcall(function()
            neck.C0 = neckC0Original
        end)
    end
end

local function interagir()
    local char = getChar()
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    for _, prompt in ipairs(workspace:GetDescendants()) do
        if prompt:IsA("ProximityPrompt") then
            local parte = prompt.Parent
            if parte and parte:IsA("BasePart") then
                if (parte.Position - hrp.Position).Magnitude <= (prompt.MaxActivationDistance + 8) then
                    prompt:InputHoldBegin()
                    task.wait(prompt.HoldDuration + 0.1)
                    prompt:InputHoldEnd()
                end
            end
        end
    end

    for _, detector in ipairs(workspace:GetDescendants()) do
        if detector:IsA("ClickDetector") then
            local parte = detector.Parent
            if parte and parte:IsA("BasePart") then
                if (parte.Position - hrp.Position).Magnitude <= (detector.MaxActivationDistance + 8) then
                    pcall(function() detector.MouseClick:Fire(player) end)
                end
            end
        end
    end

    for _, obj in ipairs(player.PlayerGui:GetDescendants()) do
        if (obj:IsA("TextButton") or obj:IsA("ImageButton")) and obj.Visible then
            local abs = obj.AbsoluteSize
            if abs.X > 10 and abs.Y > 10 then
                pcall(function()
                    obj.MouseButton1Click:Fire()
                    obj.Activated:Fire()
                end)
            end
        end
    end
end

local function clicarTela()
    local vp = workspace.CurrentCamera.ViewportSize
    pcall(function()
        VirtualInputManager:SendMouseButtonEvent(vp.X / 2, vp.Y / 2, Enum.UserInputType.MouseButton1, true, game, 1)
        task.wait(0.05)
        VirtualInputManager:SendMouseButtonEvent(vp.X / 2, vp.Y / 2, Enum.UserInputType.MouseButton1, false, game, 1)
    end)
    interagir()
end

--================= EXECUÇÃO =================

task.wait(1)

-- ETAPA 1: Teleporta + interage (1ª vez)
progresso(0.1, "Etapa 1: Indo até X:4 Y:238 Z:11...")
teleportar(Vector3.new(4, 238, 11))
task.wait(0.5)
progresso(0.25, "Etapa 1: Interagindo (E)...")
interagir()

-- Espera 2 segundos
progresso(0.35, "Aguardando 2 segundos...")
task.wait(2)

-- ETAPA 2: Noclip + MOVIMENTO até X:21 Y:3 Z:-164
progresso(0.4, "Etapa 2: Noclip ativado ✨")
ativarNoclip()
task.wait(0.2)
moverAte(Vector3.new(21, 3, -164), 60)
progresso(0.6, "Etapa 2: Chegou! ✅")
task.wait(0.3)

-- 2ª interação: olha para BAIXO antes de interagir
progresso(0.65, "Preparando 2ª interação...")
olharParaBaixo()
progresso(0.7, "Etapa 2: Interagindo pela 2ª vez...")
interagir()
restaurarPescoco()
task.wait(0.3)

-- ETAPA 3: Espera 1s → equipa TODOS os itens → clica
progresso(0.75, "Etapa 3: Aguardando 1 segundo...")
task.wait(1)

progresso(0.85, "Etapa 3: Equipando todos os itens...")
local backpack = player:WaitForChild("Backpack")
for _, item in ipairs(backpack:GetChildren()) do
    if item:IsA("Tool") then
        item.Parent = getChar()
        task.wait(0.3)
        clicarTela()
        task.wait(0.3)
    end
end

desativarNoclip()

progresso(1, "✅ Concluído!")
titulo.Text = "🔭 SUPER TELESCOPE PEGO!"

task.wait(3)
TweenService:Create(fundo, TweenInfo.new(0.8), {Size = UDim2.new(0,0,0,0), Position = UDim2.new(0.5,0,0.5,0)}):Play()
task.wait(0.9)
gui:Destroy()
