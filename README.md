--// SERVICES
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer

--// ANTI DUPLICAÇÃO
if player.PlayerGui:FindFirstChild("LaranjaScript") then
    player.PlayerGui.LaranjaScript:Destroy()
end

--// CORES E VARIÁVEIS
local TP_SPEED = 15
local THEME_COLOR = Color3.fromRGB(255,140,0)
local BG_COLOR = Color3.fromRGB(0,0,0)
local DARK_GRAY = Color3.fromRGB(40,40,40)
local rgbTheme = false

local saved = {}
local teleporting = false
local loopEnabled = false
local loopIndex = 1

local themedTexts = {}
local themedButtons = {}
local themedFrames = {}
local neonStrokes = {}

--// FUNÇÕES AUXILIARES
local function drag(frame)
    local dragging, dragStart, startPos
    frame.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = i.Position
            startPos = frame.Position
        end
    end)
    frame.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
    UIS.InputChanged:Connect(function(i)
        if dragging and i.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = i.Position - dragStart
            frame.Position = UDim2.new(
                startPos.X.Scale,
                startPos.X.Offset + delta.X,
                startPos.Y.Scale,
                startPos.Y.Offset + delta.Y
            )
        end
    end)
end

local function neon(ui)
    local s = Instance.new("UIStroke", ui)
    s.Color = THEME_COLOR
    s.Thickness = 2
    s.Transparency = 0.35
    table.insert(neonStrokes, s)
    return s
end

local function applyTheme(color)
    rgbTheme = false
    THEME_COLOR = color
    for _, t in ipairs(themedTexts) do t.TextColor3 = color end
    for _, b in ipairs(themedButtons) do b.TextColor3 = color end
    for _, f in ipairs(themedFrames) do f.BackgroundColor3 = BG_COLOR end
    for _, n in ipairs(neonStrokes) do n.Color = color end
end

RunService.RenderStepped:Connect(function()
    if rgbTheme then
        local c = Color3.fromHSV((tick()%6)/6,1,1)
        for _, t in ipairs(themedTexts) do t.TextColor3 = c end
        for _, b in ipairs(themedButtons) do b.TextColor3 = c end
        for _, n in ipairs(neonStrokes) do n.Color = c end
    end
end)

local function pull(cf)
    if teleporting then return end
    teleporting = true
    local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then teleporting=false return end
    local d = (hrp.Position - cf.Position).Magnitude
    local t = math.clamp(d/TP_SPEED, 0.25, 2)
    TweenService:Create(hrp, TweenInfo.new(t, Enum.EasingStyle.Sine), {CFrame = cf}):Play()
    task.wait(t)
    teleporting = false
end

--// GUI PRINCIPAL
local gui = Instance.new("ScreenGui", player.PlayerGui)
gui.Name = "LaranjaScript"
gui.ResetOnSpawn = false

--// BOTÃO ABRIR MENU
local openBtn = Instance.new("TextButton", gui)
openBtn.Size = UDim2.fromOffset(52,52)
openBtn.Position = UDim2.fromScale(0.03,0.45)
openBtn.Text = "🍊"
openBtn.Font = Enum.Font.GothamBlack
openBtn.TextSize = 24
openBtn.BackgroundColor3 = DARK_GRAY
openBtn.BackgroundTransparency = 0.3
Instance.new("UICorner", openBtn).CornerRadius = UDim.new(1,0)
neon(openBtn)
drag(openBtn)
table.insert(themedButtons, openBtn)

--// MENU PRINCIPAL
local main = Instance.new("Frame", gui)
main.Size = UDim2.fromScale(0.55,0.6)
main.Position = UDim2.fromScale(0.5,0.5)
main.AnchorPoint = Vector2.new(0.5,0.5)
main.BackgroundColor3 = BG_COLOR
main.BackgroundTransparency = 0.35
Instance.new("UICorner", main).CornerRadius = UDim.new(0,14)
neon(main)
drag(main)
table.insert(themedFrames, main)
main.Visible = false

--// TÍTULO PRINCIPAL E SUBTÍTULO
local title = Instance.new("TextLabel", main)
title.Size = UDim2.new(1,0,0,36)
title.Text = "LARANJA SCRIPT"
title.Font = Enum.Font.GothamBlack
title.TextSize = 26
title.BackgroundTransparency = 1
table.insert(themedTexts, title)

local subtitle = Instance.new("TextLabel", main)
subtitle.Size = UDim2.new(1,0,0,20)
subtitle.Position = UDim2.fromOffset(0,36)
subtitle.Text = "Bem-vindo ao menu"
subtitle.Font = Enum.Font.Gotham
subtitle.TextSize = 14
subtitle.TextColor3 = THEME_COLOR
subtitle.BackgroundTransparency = 1
subtitle.TextXAlignment = Enum.TextXAlignment.Center
table.insert(themedTexts, subtitle)

--// BOTÃO FECHAR
local closeBtn = Instance.new("TextButton", main)
closeBtn.Size = UDim2.fromOffset(34,34)
closeBtn.Position = UDim2.fromScale(0.97,0.03)
closeBtn.AnchorPoint = Vector2.new(1,0)
closeBtn.Text = "X"
closeBtn.Font = Enum.Font.GothamBlack
closeBtn.TextSize = 18
closeBtn.BackgroundColor3 = DARK_GRAY
closeBtn.BackgroundTransparency = 0.2
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0,8)
table.insert(themedButtons, closeBtn)

-- Função para abrir menu e reiniciar loop para evitar bugs
local function openMenu()
    openBtn.Visible = false
    main.Visible = true
    loopEnabled = false -- desligar loop ao abrir o menu
    loopIndex = 1      -- reiniciar do slot 1
end

openBtn.MouseButton1Click:Connect(openMenu)

closeBtn.MouseButton1Click:Connect(function()
    main.Visible = false
    openBtn.Visible = true
    loopEnabled = false -- também garante desligar loop ao fechar
    loopIndex = 1
end)

--// SIDEBAR
local side = Instance.new("Frame", main)
side.Size = UDim2.new(0.22,0,0.6,0)
side.Position = UDim2.fromScale(0,0.35)
side.BackgroundColor3 = DARK_GRAY
side.BackgroundTransparency = 0.4
Instance.new("UICorner", side).CornerRadius = UDim.new(0,10)
table.insert(themedFrames, side)

local sideTitle = Instance.new("TextLabel", side)
sideTitle.Size = UDim2.new(1,0,0,28)
sideTitle.Position = UDim2.fromOffset(0,0)
sideTitle.Text = "CATEGORIAS"
sideTitle.Font = Enum.Font.GothamBold
sideTitle.TextSize = 16
sideTitle.BackgroundTransparency = 1
sideTitle.TextColor3 = THEME_COLOR
table.insert(themedTexts, sideTitle)

local function sideBtn(txt, y)
    local b = Instance.new("TextButton", side)
    b.Size = UDim2.new(0.9,0,0.12,0)
    b.Position = UDim2.new(0.05,0, y,0)
    b.Text = txt
    b.Font = Enum.Font.GothamBold
    b.TextSize = 15
    b.BackgroundColor3 = DARK_GRAY
    b.BackgroundTransparency = 0.3
    b.BorderSizePixel = 0
    Instance.new("UICorner", b).CornerRadius = UDim.new(0,8)
    table.insert(themedButtons,b)
    table.insert(themedTexts,b)
    return b
end

-- Categorias mais para baixo (y = 0.25,0.45,0.65)
local cfgBtn = sideBtn("Config",0.25)
local discordBtn = sideBtn("Discord",0.45)
local tpBtn = sideBtn("TP",0.65)

--===== PAGE TP =====
local tpPage = Instance.new("Frame", main)
tpPage.Size = UDim2.new(0.75,0,0.58,0)
tpPage.Position = UDim2.fromScale(0.24,0.35)
tpPage.BackgroundTransparency = 1
tpPage.Visible = false

local loopBtn = Instance.new("TextButton", tpPage)
loopBtn.Size = UDim2.fromOffset(140,28)
loopBtn.Position = UDim2.fromScale(0.5,0.01)
loopBtn.AnchorPoint = Vector2.new(0.5,0)
loopBtn.Text = "LOOP: OFF"
loopBtn.Font = Enum.Font.GothamBold
loopBtn.TextSize = 14
loopBtn.BackgroundColor3 = DARK_GRAY
loopBtn.BackgroundTransparency = 0.3
Instance.new("UICorner", loopBtn).CornerRadius = UDim.new(0,8)
table.insert(themedButtons, loopBtn)
table.insert(themedTexts, loopBtn)

loopBtn.MouseButton1Click:Connect(function()
    loopEnabled = not loopEnabled
    loopBtn.Text = loopEnabled and "LOOP: ON" or "LOOP: OFF"
    if not loopEnabled then loopIndex = 1 end
end)

-- SCROLL DE LOCAIS
local scroll = Instance.new("ScrollingFrame", tpPage)
scroll.Size = UDim2.new(1,0,1,-60)
scroll.Position = UDim2.fromOffset(0,50)
scroll.CanvasSize = UDim2.new(0,0,0,9*70)
scroll.ScrollBarImageTransparency = 0.4
scroll.BackgroundTransparency = 1
scroll.Visible = true

for i=1,9 do
    local slot = Instance.new("Frame", scroll)
    slot.Size = UDim2.fromOffset(420,60)
    slot.Position = UDim2.fromOffset(10,10+(i-1)*70)
    slot.BackgroundColor3 = BG_COLOR
    slot.BackgroundTransparency = 0.35
    Instance.new("UICorner", slot).CornerRadius = UDim.new(0,10)
    neon(slot)

    local name = Instance.new("TextLabel", slot)  
    name.Text = "Local "..i  
    name.Position = UDim2.fromOffset(10,6)  
    name.Size = UDim2.fromOffset(120,18)  
    name.BackgroundTransparency = 1  
    name.Font = Enum.Font.GothamBold  
    name.TextSize = 14  
    table.insert(themedTexts, name)  

    local status = Instance.new("TextLabel", slot)  
    status.Position = UDim2.fromOffset(10,40)  
    status.Size = UDim2.fromOffset(200,16)  
    status.BackgroundTransparency = 1  
    status.Font = Enum.Font.Gotham  
    status.TextSize = 12  
    status.TextColor3 = Color3.fromRGB(150,150,150)  

    local function createBtn(txt,x)
        local b = Instance.new("TextButton", slot)
        b.Size = UDim2.fromOffset(60,26)
        b.Position = UDim2.fromOffset(x,18)
        b.Text = txt
        b.Font = Enum.Font.GothamBold
        b.TextSize = 13
        b.BackgroundColor3 = DARK_GRAY
        b.BackgroundTransparency = 0.3
        b.BorderSizePixel = 0
        Instance.new("UICorner", b).CornerRadius = UDim.new(0,6)
        table.insert(themedButtons, b)
        return b
    end

    createBtn("Salvar",150).MouseButton1Click:Connect(function()
        local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
        if hrp then
            saved[i] = hrp.CFrame
            status.Text = "Salvo"
            status.TextColor3 = Color3.fromRGB(0,255,120)
        end
    end)

    createBtn("TP",220).MouseButton1Click:Connect(function()
        if saved[i] then pull(saved[i]) end
    end)

    createBtn("Rem",290).MouseButton1Click:Connect(function()
        saved[i] = nil
        status.Text = "Removido"
        status.TextColor3 = Color3.fromRGB(255,80,80)
        task.delay(3, function() 
            status.Text = "Vazio"
            status.TextColor3 = Color3.fromRGB(150,150,150)
        end)
    end)
end

-- LOOP AUTOMÁTICO
task.spawn(function()
    while true do
        task.wait(0.4)
        if loopEnabled then
            if loopIndex > 9 then loopIndex = 1 end
            if saved[loopIndex] then pull(saved[loopIndex]) end
            loopIndex += 1
        end
    end
end)

--================= PAGE CONFIG =================
local cfgPage = Instance.new("Frame", main)
cfgPage.Size = UDim2.new(0.75,0,0.58,0)
cfgPage.Position = UDim2.fromScale(0.24,0.35)
cfgPage.BackgroundTransparency = 1
cfgPage.Visible = true -- abre por padrão

local cfgScroll = Instance.new("ScrollingFrame", cfgPage)
cfgScroll.Size = UDim2.fromScale(1,1)
cfgScroll.Position = UDim2.fromOffset(0,0)
cfgScroll.CanvasSize = UDim2.new(0,0,0,400)
cfgScroll.ScrollBarImageTransparency = 0.4
cfgScroll.BackgroundTransparency = 1

local themes = {
    {"Vermelho",Color3.fromRGB(255,0,0)},
    {"Laranja",Color3.fromRGB(255,140,0)},
    {"Amarelo",Color3.fromRGB(255,255,0)},
    {"Verde",Color3.fromRGB(0,255,120)},
    {"Azul",Color3.fromRGB(0,170,255)},
    {"Roxo",Color3.fromRGB(180,0,255)},
    {"Marrom",Color3.fromRGB(120,70,40)},
    {"Preto",Color3.fromRGB(0,0,0)},
    {"Branco",Color3.fromRGB(255,255,255)},
    {"RGB",nil}
}

for i,v in ipairs(themes) do
    local b = Instance.new("TextButton", cfgScroll)
    b.Size = UDim2.fromOffset(220,32)
    b.Position = UDim2.fromOffset(20,20+(i-1)*36)
    b.Text = v[1]
    b.Font = Enum.Font.GothamBold
    b.TextSize = 14
    b.BackgroundColor3 = DARK_GRAY
    b.BackgroundTransparency = 0.3
    Instance.new("UICorner", b).CornerRadius = UDim.new(0,8)
    table.insert(themedButtons,b)
    table.insert(themedTexts,b)
    if v[1] == "RGB" then
        b.MouseButton1Click:Connect(function() rgbTheme = true end)
    else
        b.MouseButton1Click:Connect(function() applyTheme(v[2]) end)
    end
end

--================= PAGE DISCORD =================
local discordPage = Instance.new("Frame", main)
discordPage.Size = UDim2.new(0.75,0,0.58,0)
discordPage.Position = UDim2.fromScale(0.24,0.35)
discordPage.BackgroundTransparency = 1
discordPage.Visible = false

local discordDesc = Instance.new("TextLabel", discordPage)
discordDesc.Size = UDim2.fromScale(0.9,0.8)
discordDesc.Position = UDim2.fromOffset(0,0)
discordDesc.Text = "Junte-se ao nosso Discord para suporte, novidades e interações com a comunidade!"
discordDesc.TextWrapped = true
discordDesc.Font = Enum.Font.GothamBold
discordDesc.TextSize = 16
discordDesc.TextColor3 = THEME_COLOR
discordDesc.BackgroundTransparency = 1
discordDesc.TextXAlignment = Enum.TextXAlignment.Left
table.insert(themedTexts, discordDesc)

local discordCopyBtn = Instance.new("TextButton", discordPage)
discordCopyBtn.Size = UDim2.fromOffset(220,34)
discordCopyBtn.Position = UDim2.fromScale(0.5,0.95)
discordCopyBtn.AnchorPoint = Vector2.new(0.5,1)
discordCopyBtn.Text = "COPIAR DISCORD"
discordCopyBtn.Font = Enum.Font.GothamBold
discordCopyBtn.TextSize = 14
discordCopyBtn.BackgroundColor3 = DARK_GRAY
discordCopyBtn.BackgroundTransparency = 0.3
Instance.new("UICorner", discordCopyBtn).CornerRadius = UDim.new(0,8)
table.insert(themedButtons, discordCopyBtn)
table.insert(themedTexts, discordCopyBtn)

discordCopyBtn.MouseButton1Click:Connect(function()
    setclipboard("https://discord.gg/SEU_LINK_AQUI")
end)

--// SWITCH PAGES
local currentPage = cfgPage -- abre Config por padrão
local function switchPage(p)
    if currentPage then currentPage.Visible = false end
    p.Visible = true
    currentPage = p
end

cfgBtn.MouseButton1Click:Connect(function() switchPage(cfgPage) end)
discordBtn.MouseButton1Click:Connect(function() switchPage(discordPage) end)
tpBtn.MouseButton1Click:Connect(function() switchPage(tpPage) end)
