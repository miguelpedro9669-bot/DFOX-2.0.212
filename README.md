--========================================================--
-- 🏴☣️ BSH — TUDO VERDE ✅
-- Botões VERDES | Hitbox +/- VERDE CLARO ✅
-- Fly v4 + Emote Externo | Delta + Celular ✅
-- Chave: BSH2026
--========================================================--

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local LP = Players.LocalPlayer
local GuiParent = LP:WaitForChild("PlayerGui")

pcall(function()
    for _, n in ipairs({"BSH_PANEL", "BSH_KeySystem"}) do
        local old = GuiParent:FindFirstChild(n)
        if old then old:Destroy() end
    end
end)

-- === CORES — TUDO VERDE ✅ ===
local C = {
    Fundo = Color3.fromRGB(0,0,0),
    Painel = Color3.fromRGB(8,12,8),
    Cartao = Color3.fromRGB(15,25,15),
    Botao = Color3.fromRGB(20,60,30),
    Destaque = Color3.fromRGB(0,100,40),
    Branco = Color3.fromRGB(220,255,220),
    -- ✅ Tudo verde — sem vermelho
    VerdeON = Color3.fromRGB(0,255,100),
    VerdeOFF = Color3.fromRGB(0,120,50),
    VerdeNeon = Color3.fromRGB(0,255,80),
    VerdeClaro = Color3.fromRGB(80,255,140), -- ✅ Verde claro para +/-
    VerdeClaro2 = Color3.fromRGB(100,255,160),
    VerdeTxt = Color3.fromRGB(120,255,180),
    RGB = {
        Color3.fromRGB(0,255,80), Color3.fromRGB(0,220,100),
        Color3.fromRGB(0,255,130), Color3.fromRGB(50,255,120),
        Color3.fromRGB(0,200,90), Color3.fromRGB(80,255,150)
    }
}

-- === AUXILIARES ===
local function Raiz(P) return P and P.Character and P.Character:FindFirstChild("HumanoidRootPart") end
local function Hum() return LP.Character and LP.Character:FindFirstChildOfClass("Humanoid") end
local function Cantar(o, r) local c=Instance.new("UICorner") c.CornerRadius=UDim.new(0,r) c.Parent=o end
local function Borda(o, esp) local s=Instance.new("UIStroke") s.Color=C.VerdeNeon s.Thickness=esp s.ApplyStrokeMode=Enum.ApplyStrokeMode.Border s.Parent=o; return s end

local RGBAlvo = {}
local function SincRGB(...) for _,v in ipairs({...}) do RGBAlvo[#RGBAlvo+1]=v end end
task.spawn(function()
    while task.wait(0.05) do
        for i=#RGBAlvo,1,-1 do if not RGBAlvo[i] or not RGBAlvo[i].Parent then table.remove(RGBAlvo,i) end end
        local t = (os.clock()*0.6)%#C.RGB
        local c = C.RGB[math.floor(t)+1]:Lerp(C.RGB[(math.floor(t)+1)%#C.RGB+1], t%1)
        for _,v in ipairs(RGBAlvo) do
            if v and v.Parent then
                if v:IsA("UIStroke") then v.Color=c
                elseif v:IsA("TextButton") or v:IsA("TextLabel") then v.TextColor3=c end
            end
        end
    end
end)

local function Arrastar(Botao, Pos)
    Botao.Position = Pos
    local arr, iniP, iniT = false, nil, nil
    Botao.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            arr, iniP, iniT = true, Botao.Position, i.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(i)
        if not arr then return end
        if i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch then
            local d = i.Position - iniT
            Botao.Position = UDim2.new(iniP.X.Scale, iniP.X.Offset + d.X, 0, math.max(0, iniP.Y.Offset + d.Y))
        end
    end)
    Botao.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then arr = false end
    end)
end

local function ArrastarPainel(Painel, Cabeca)
    local arr, iniP, iniT = false, nil, nil
    Cabeca.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            arr, iniP, iniT = true, Painel.Position, i.Position
        end
    end)
    Cabeca.InputChanged:Connect(function(i)
        if not arr then return end
        if i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch then
            local d = i.Position - iniT
            Painel.Position = UDim2.new(iniP.X.Scale, iniP.X.Offset + d.X, iniP.Y.Scale, iniP.Y.Offset + d.Y)
        end
    end)
    Cabeca.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then arr = false end
    end)
end

-- === VARIÁVEIS ===
local AlvoOrb, OrbLigado, OrbConn = nil, false, nil
local GodOn, GodConn = false, nil
local FlutVisiveis = false
local NoClip, AntiVoid, FugaAuto = false, false, false
local FlyAtivo = false
local OrbitModo8, Orbit8Conn = false, nil
local HBAtivo, TamHB = false, 100
local TAM_MIN, TAM_MAX = 10, 5000
local VELOCIDADE_BASE, VelAtual = 30, 30
local RAIO_GOD = 10000
local ALTURA_GOD = 600
local UltimoHP = 100
local ESPAtivo, ESPDist, ESPVida, ESPAlvo = false, false, false, false
local ESPObjetos = {}
local BtnFlutOrb, BtnFlutGod

-- === PARAR FUNÇÕES ===
local function PararOrb()
    OrbLigado = false
    if OrbConn then OrbConn:Disconnect(); OrbConn = nil end
    if Hum() then Hum().PlatformStand = false end
end
local function PararGod(silencioso)
    GodOn = false
    if GodConn then GodConn:Disconnect(); GodConn = nil end
    if Hum() then Hum().PlatformStand = false end
    VelAtual = VELOCIDADE_BASE
    if BtnFlutGod and not silencioso then
        BtnFlutGod.Text = "👑 GOD • OFF"
        BtnFlutGod.BackgroundColor3 = C.Cartao
    end
end
local function PararOrbit8()
    OrbitModo8 = false
    if Orbit8Conn then Orbit8Conn:Disconnect(); Orbit8Conn = nil end
    if Hum() then Hum().PlatformStand = false end
end

-- === ORBIT JOGADOR ===
local function LigarOrb(Alvo)
    if not Alvo or not Alvo.Character then return end
    PararOrb()
    AlvoOrb, OrbLigado = Alvo, true
    if Hum() then Hum().PlatformStand = true end
    local ang = 0
    OrbConn = RunService.Heartbeat:Connect(function(dt)
        if not OrbLigado or not AlvoOrb or not AlvoOrb.Character then PararOrb(); return end
        local eu, av = Raiz(LP), Raiz(AlvoOrb)
        if not eu or not av then return end
        ang = ang + 3 * dt
        eu.CFrame = CFrame.new(av.Position + Vector3.new(math.cos(ang)*9, 2.5, math.sin(ang)*9), av.Position)
    end)
end

-- === GOD ORBIT ===
local function LigarGod()
    PararGod(true)
    GodOn = true
    if Hum() then Hum().PlatformStand = true end
    if BtnFlutGod then
        BtnFlutGod.Text = "👑 GOD ×" .. VelAtual
        BtnFlutGod.BackgroundColor3 = C.VerdeON
    end
    GodConn = RunService.RenderStepped:Connect(function()
        if not GodOn then return end
        local eu = Raiz(LP); if not eu then return end
        local t = os.clock() * VelAtual
        eu.CFrame = CFrame.new(math.cos(t)*RAIO_GOD, ALTURA_GOD, math.sin(t)*RAIO_GOD)
    end)
end

-- === ORBIT 8 DIREÇÕES ===
local function IniciarOrbit8()
    PararOrbit8(); PararOrb(); PararGod(true)
    OrbitModo8 = true
    if Hum() then Hum().PlatformStand = true end
    local t0 = 0
    Orbit8Conn = RunService.RenderStepped:Connect(function(dt)
        if not OrbitModo8 then return end
        local eu = Raiz(LP); if not eu then return end
        t0 = t0 + dt
        local t = t0 * VelAtual * 0.4
        eu.CFrame = CFrame.new(math.sin(t)*RAIO_GOD, ALTURA_GOD + math.sin(t*1.8)*ALTURA_GOD*0.3, math.sin(t*2)*RAIO_GOD*0.5)
    end)
end

-- === FUGA ===
local function Fugir()
    local eu = Raiz(LP)
    if eu then eu.CFrame = CFrame.new(math.random(-800,800), 2500, math.random(-800,800)) end
end

-- === HITBOX +/- ===
local function Mais()
    TamHB = TamHB < 100 and math.min(TamHB+10, 100) or math.min(TamHB*2, TAM_MAX)
end
local function Menos()
    TamHB = TamHB <= 100 and math.max(TamHB-10, TAM_MIN) or math.max(math.floor(TamHB/2), TAM_MIN)
end

-- === ESP ===
local function CriarESP(P)
    if ESPObjetos[P] then return end
    local char = P.Character; if not char then return end
    local hl = Instance.new("Highlight")
    hl.FillColor = C.VerdeOFF; hl.OutlineColor = C.VerdeNeon
    hl.FillTransparency = 0.6; hl.OutlineTransparency = 0
    hl.Adornee = char; hl.Parent = char
    local bb = Instance.new("BillboardGui")
    bb.Size = UDim2.fromOffset(220,50); bb.AlwaysOnTop = true; bb.Adornee = char
    local txt = Instance.new("TextLabel")
    txt.Size = UDim2.new(1,0,1,0); txt.BackgroundTransparency = 1
    txt.TextColor3 = C.VerdeNeon; txt.TextStrokeTransparency = 0
    txt.TextStrokeColor3 = Color3.new(0,0,0)
    txt.Font = Enum.Font.Code; txt.TextSize = 13; txt.Text = P.DisplayName; txt.Parent = bb
    bb.Parent = char
    ESPObjetos[P] = {Highlight=hl, Billboard=bb, Texto=txt}
end
local function RemoverESP(P)
    local d = ESPObjetos[P]; if not d then return end
    if d.Highlight then d.Highlight:Destroy() end
    if d.Billboard then d.Billboard:Destroy() end
    ESPObjetos[P] = nil
end
local function RemoverTodoESP() for P in pairs(ESPObjetos) do RemoverESP(P) end end

task.spawn(function()
    while task.wait(0.2) do
        if not ESPAtivo then if next(ESPObjetos) then RemoverTodoESP() end
        else
            for _,P in ipairs(Players:GetPlayers()) do
                if P~=LP then
                    local isAlvo = AlvoOrb == P
                    if (not ESPAlvo or isAlvo) and P.Character then
                        CriarESP(P)
                        local d = ESPObjetos[P]
                        if d then
                            if d.Highlight then d.Highlight.FillColor = isAlvo and C.VerdeON or C.VerdeOFF end
                            local eu, av = Raiz(LP), Raiz(P)
                            local t = {P.DisplayName}
                            if ESPDist and eu and av then table.insert(t, "📏"..math.floor((eu.Position-av.Position).Magnitude)) end
                            if ESPVida then
                                local h = P.Character:FindFirstChildOfClass("Humanoid")
                                if h then table.insert(t, "💚"..math.floor(h.Health)) end
                            end
                            if d.Texto then d.Texto.Text = table.concat(t, " | ") end
                        end
                    else RemoverESP(P) end
                end
            end
        end
    end
end)
Players.PlayerRemoving:Connect(RemoverESP)

-- === SISTEMA DE CHAVE ===
local KeyGui = Instance.new("ScreenGui")
KeyGui.Name = "BSH_KeySystem"
KeyGui.ResetOnSpawn = false
KeyGui.Parent = GuiParent

local FundoKey = Instance.new("Frame")
FundoKey.Size = UDim2.fromOffset(360,200)
FundoKey.Position = UDim2.new(0.5,-180,0.5,-100)
FundoKey.BackgroundColor3 = C.Fundo
FundoKey.Parent = KeyGui
Cantar(FundoKey,18)
SincRGB(Borda(FundoKey,3))

local TitKey = Instance.new("TextLabel")
TitKey.Position = UDim2.new(0,0,0,20)
TitKey.Size = UDim2.new(1,0,0,30)
TitKey.BackgroundTransparency = 1
TitKey.Text = "🔐 DIGITE A CHAVE"
TitKey.TextColor3 = C.VerdeNeon
TitKey.Font = Enum.Font.GothamBold; TitKey.TextSize = 16
TitKey.Parent = FundoKey
SincRGB(TitKey)

local InputKey = Instance.new("TextBox")
InputKey.Position = UDim2.new(0.5,-140,0,70)
InputKey.Size = UDim2.fromOffset(280,45)
InputKey.BackgroundColor3 = C.Cartao
InputKey.Text = ""; InputKey.PlaceholderText = "Digite: BSH2026"
InputKey.TextColor3 = C.Branco; InputKey.Font = Enum.Font.GothamBold
InputKey.TextSize = 14; InputKey.ClearTextOnFocus = false
InputKey.Parent = FundoKey
Cantar(InputKey,10)

local MsgKey = Instance.new("TextLabel")
MsgKey.Position = UDim2.new(0,0,0,125)
MsgKey.Size = UDim2.new(1,0,0,20)
MsgKey.BackgroundTransparency = 1
MsgKey.Text = ""; MsgKey.Font = Enum.Font.GothamBold
MsgKey.TextSize = 11; MsgKey.Parent = FundoKey

local BtnEntrar = Instance.new("TextButton")
BtnEntrar.Position = UDim2.new(0.5,-100,0,155)
BtnEntrar.Size = UDim2.fromOffset(200,35)
BtnEntrar.BackgroundColor3 = C.Destaque
BtnEntrar.Text = "✅ ENTRAR"
BtnEntrar.Font = Enum.Font.GothamBold
BtnEntrar.AutoButtonColor = false
BtnEntrar.Parent = FundoKey
Cantar(BtnEntrar,10)
SincRGB(BtnEntrar)

-- === PAINEL PRINCIPAL ===
local function AbrirPainel()
    KeyGui:Destroy()
    local Gui = Instance.new("ScreenGui")
    Gui.Name = "BSH_PANEL"
    Gui.ResetOnSpawn = false
    Gui.Parent = GuiParent

    -- Botão Menu
    local BtnMenu = Instance.new("TextButton")
    BtnMenu.Size = UDim2.fromOffset(55,55)
    BtnMenu.BackgroundColor3 = C.Cartao
    BtnMenu.Text = "☣️"
    BtnMenu.Font = Enum.Font.GothamBold; BtnMenu.TextSize = 26
    BtnMenu.AutoButtonColor = false
    BtnMenu.Parent = Gui
    Cantar(BtnMenu,14)
    SincRGB(Borda(BtnMenu,2.5))
    Arrastar(BtnMenu, UDim2.new(0.008,0,0.02,0))

    -- Flutuantes
    BtnFlutOrb = Instance.new("TextButton")
    BtnFlutOrb.Size = UDim2.fromOffset(170,52)
    BtnFlutOrb.BackgroundColor3 = C.Cartao
    BtnFlutOrb.Text = "🌀 ORBIT • SELECIONE"
    BtnFlutOrb.Font = Enum.Font.GothamBold; BtnFlutOrb.TextSize = 11
    BtnFlutOrb.Visible = false; BtnFlutOrb.AutoButtonColor = false
    BtnFlutOrb.Parent = Gui
    Cantar(BtnFlutOrb,14)
    SincRGB(Borda(BtnFlutOrb,2))
    Arrastar(BtnFlutOrb, UDim2.new(0.008,0,0.09,0))

    BtnFlutGod = Instance.new("TextButton")
    BtnFlutGod.Size = UDim2.fromOffset(170,52)
    BtnFlutGod.BackgroundColor3 = C.Cartao
    BtnFlutGod.Text = "👑 GOD • OFF"
    BtnFlutGod.Font = Enum.Font.GothamBold; BtnFlutGod.TextSize = 11
    BtnFlutGod.Visible = false; BtnFlutGod.AutoButtonColor = false
    BtnFlutGod.Parent = Gui
    Cantar(BtnFlutGod,14)
    SincRGB(Borda(BtnFlutGod,2))
    Arrastar(BtnFlutGod, UDim2.new(0.008,0,0.17,0))

    -- Painel
    local Painel = Instance.new("Frame")
    Painel.Size = UDim2.fromOffset(520,360)
    Painel.Position = UDim2.new(0.5,-260,0.5,-180)
    Painel.BackgroundColor3 = C.Fundo
    Painel.Visible = false; Painel.Parent = Gui
    Cantar(Painel,22)
    SincRGB(Borda(Painel,3.5))

    local Topo = Instance.new("TextButton")
    Topo.Size = UDim2.new(1,0,0,50)
    Topo.BackgroundColor3 = C.Painel
    Topo.Text = ""; Topo.AutoButtonColor = false
    Topo.Parent = Painel
    Cantar(Topo,22)
    ArrastarPainel(Painel, Topo)

    local Titulo = Instance.new("TextLabel")
    Titulo.Position = UDim2.fromOffset(18,12)
    Titulo.Size = UDim2.fromOffset(400,26)
    Titulo.BackgroundTransparency = 1
    Titulo.Text = "🏴☣️ BSH — TUDO VERDE ✅"
    Titulo.Font = Enum.Font.GothamBold; Titulo.TextSize = 14
    Titulo.TextXAlignment = Enum.TextXAlignment.Left
    Titulo.Parent = Topo
    SincRGB(Titulo)

    -- Abas
    local Lateral = Instance.new("Frame")
    Lateral.Position = UDim2.fromOffset(12,62)
    Lateral.Size = UDim2.fromOffset(105,286)
    Lateral.BackgroundColor3 = C.Painel
    Lateral.Parent = Painel
    Cantar(Lateral,16)

    local function Aba(Nome,Y,Ativa)
        local B = Instance.new("TextButton")
        B.Position = UDim2.fromOffset(8,Y)
        B.Size = UDim2.new(1,-16,0,34)
        B.BackgroundColor3 = Ativa and C.Destaque or C.Botao
        B.Text = Nome; B.TextColor3 = C.Branco
        B.Font = Enum.Font.GothamBold; B.TextSize = 9
        B.AutoButtonColor = false; B.Parent = Lateral
        Cantar(B,10); return B
    end
    local Aba_Ply = Aba("👤 PLAYER",10,true)
    local Aba_Orb = Aba("🌀 ORBIT",54,false)
    local Aba_HB = Aba("💠 HITBOX",98,false)
    local Aba_ESP = Aba("👁️ ESP",142,false)
    local Aba_Flut = Aba("🎯 FLUTUANTES",186,false)
    local Aba_Ext = Aba("🔗 EXTRA",230,false)

    local Conteudo = Instance.new("ScrollingFrame")
    Conteudo.Position = UDim2.fromOffset(129,62)
    Conteudo.Size = UDim2.new(1,-141,1,-74)
    Conteudo.BackgroundColor3 = C.Painel
    Conteudo.ScrollBarThickness = 6
    Conteudo.AutomaticCanvasSize = Enum.AutomaticSize.Y
    Conteudo.Parent = Painel
    Cantar(Conteudo,16)

    local function Pag()
        local P = Instance.new("Frame")
        P.Size = UDim2.new(1,-16,0,600)
        P.Position = UDim2.fromOffset(8,8)
        P.BackgroundTransparency = 1; P.Visible = false; P.Parent = Conteudo
        return P
    end
    local PagPly,PagOrb,PagHB,PagESP,PagFlut,PagExt = Pag(),Pag(),Pag(),Pag(),Pag(),Pag()
    PagPly.Visible = true

    local AbaAtual,PagAtual = Aba_Ply,PagPly
    local function Trocar(Botao,Pagina)
        AbaAtual.BackgroundColor3 = C.Botao
        Botao.BackgroundColor3 = C.Destaque
        AbaAtual = Botao; PagAtual.Visible = false; Pagina.Visible = true; PagAtual = Pagina
    end
    Aba_Ply.Activated:Connect(function() Trocar(Aba_Ply,PagPly) end)
    Aba_Orb.Activated:Connect(function() Trocar(Aba_Orb,PagOrb) end)
    Aba_HB.Activated:Connect(function() Trocar(Aba_HB,PagHB) end)
    Aba_ESP.Activated:Connect(function() Trocar(Aba_ESP,PagESP) end)
    Aba_Flut.Activated:Connect(function() Trocar(Aba_Flut,PagFlut) end)
    Aba_Ext.Activated:Connect(function() Trocar(Aba_Ext,PagExt) end)

    -- ✅ BOTÕES LIGA/DESLIGA — TODOS VERDES
    local function BotaoToggle(Pag,Nome,Y,Acao)
        local Ligado=false
        local B = Instance.new("TextButton")
        B.Position = UDim2.new(0,0,0,Y)
        B.Size = UDim2.new(1,0,0,46)
        B.BackgroundColor3 = C.VerdeOFF -- ✅ Verde quando desligado
        B.Text = Nome.." • OFF"
        B.TextColor3 = C.Branco
        B.Font = Enum.Font.GothamBold; B.TextSize = 11
        B.AutoButtonColor = false; B.Parent = Pag
        Cantar(B,12)
        B.Activated:Connect(function()
            Ligado = not Ligado
            B.Text = Nome.." • "..(Ligado and "ON ✅" or "OFF")
            B.BackgroundColor3 = Ligado and C.VerdeON or C.VerdeOFF -- ✅ Verde claro ligado
            Acao(Ligado)
        end)
    end

    -- ABA PLAYER
    BotaoToggle(PagPly,"🛡️ ANTI VOID",0,function(L) AntiVoid=L end)
    BotaoToggle(PagPly,"👻 NOCLIP",56,function(L) NoClip=L end)

    local BtnFly = Instance.new("TextButton")
    BtnFly.Position = UDim2.new(0,0,0,112)
    BtnFly.Size = UDim2.new(1,0,0,46)
    BtnFly.BackgroundColor3 = C.VerdeOFF
    BtnFly.Text = "✈️ FLY v4 • OFF"
    BtnFly.Font = Enum.Font.GothamBold; BtnFly.TextSize = 11
    BtnFly.AutoButtonColor = false; BtnFly.Parent = PagPly
    Cantar(BtnFly,12)
    BtnFly.Activated:Connect(function()
        FlyAtivo = not FlyAtivo
        BtnFly.Text = "✈️ FLY v4 • "..(FlyAtivo and "ON ✅" or "OFF")
        BtnFly.BackgroundColor3 = FlyAtivo and C.VerdeON or C.VerdeOFF
        if FlyAtivo then
            loadstring(game:HttpGet("https://rawscripts.net/raw/Universal-Script-Fly-v4-the-best-fly-227539"))()
        end
    end)

    local BtnFuga = Instance.new("TextButton")
    BtnFuga.Position = UDim2.new(0,0,0,172)
    BtnFuga.Size = UDim2.new(1,0,0,46)
    BtnFuga.BackgroundColor3 = C.VerdeClaro -- ✅ Verde claro
    BtnFuga.Text = "🚀 FUGA"
    BtnFuga.Font = Enum.Font.GothamBold
    BtnFuga.AutoButtonColor = false
    BtnFuga.Parent = PagPly
    Cantar(BtnFuga,12)
    BtnFuga.Activated:Connect(Fugir)

    local BtnAutoFuga = Instance.new("TextButton")
    BtnAutoFuga.Position = UDim2.new(0,0,0,228)
    BtnAutoFuga.Size = UDim2.new(1,0,0,46)
    BtnAutoFuga.BackgroundColor3 = C.VerdeOFF
    BtnAutoFuga.Text = "🔄 AUTO FUGA • OFF"
    BtnAutoFuga.Font = Enum.Font.GothamBold
    BtnAutoFuga.AutoButtonColor = false
    BtnAutoFuga.Parent = PagPly
    Cantar(BtnAutoFuga,12)
    BtnAutoFuga.Activated:Connect(function()
        FugaAuto = not FugaAuto
        BtnAutoFuga.Text = "🔄 AUTO FUGA • "..(FugaAuto and "ON ✅" or "OFF")
        BtnAutoFuga.BackgroundColor3 = FugaAuto and C.VerdeON or C.VerdeOFF
    end)

    -- ABA EXTRA — Emote Script
    local BtnEmote = Instance.new("TextButton")
    BtnEmote.Position = UDim2.new(0,0,0,20)
    BtnEmote.Size = UDim2.new(1,0,0,60)
    BtnEmote.BackgroundColor3 = C.VerdeClaro
    BtnEmote.Text = "🎭 ABRIR EMOTES ✅"
    BtnEmote.Font = Enum.Font.GothamBold; BtnEmote.TextSize = 14
    BtnEmote.AutoButtonColor = false; BtnEmote.Parent = PagExt
    Cantar(BtnEmote,12)
    SincRGB(BtnEmote)
    BtnEmote.Activated:Connect(function()
        loadstring(game:HttpGet("https://rawscripts.net/raw/Universal-Script-7yd7-I-Emote-Script-48024"))()
    end)

    local InfoExt = Instance.new(
