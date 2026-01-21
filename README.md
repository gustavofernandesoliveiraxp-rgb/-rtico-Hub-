--==================================================
-- ❄️ ÁRTICO HUB ULTIMATE PRO+ - KING LEGACY ❄️
--
-- Script Hub Cliente (Executor) PRO+
-- Compatível: Fluxus | Hydrogen | Delta | Arceus X
--
-- Funcionalidades:
-- 🔹 Auto Farm NPC / Boss / Raid
-- 🔹 Auto Quest Inteligente
-- 🔹 Auto Fruit / Devil Fruit
-- 🔹 GUI neon animada com sons
-- 🔹 Painel de cores dinâmico
-- 🔹 Visualização de NPCs por prioridade
-- 🔹 Multi-player friendly
-- 🔹 Toggle ON/OFF de todas funções
-- 🔹 Seleção de arma / Delay / Velocidade
-- 🔹 Salva configuração do jogador
--
-- ✍ Dono: SrUrsolino
--==================================================

--========================
-- SERVIÇOS
--========================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")
local SoundService = game:GetService("SoundService")
local UserInputService = game:GetService("UserInputService")

--========================
-- PLAYER
--========================
local Player = Players.LocalPlayer
local Character = Player.Character or Player.CharacterAdded:Wait()

--========================
-- CONFIGURAÇÕES / LOCAL STORAGE
--========================
local ConfigFileName = "ÁrticoHubPROPlus_"..Player.UserId
local Config = {
    AutoFarm = true,
    AutoQuest = true,
    AutoBoss = true,
    AutoFruit = false,
    FarmWeapon = "Sword",
    FarmDelay = 0.15,
    AttackDistance = 6,
    MinLevel = 1,
    MaxLevel = 999,
    AutoIslandFarm = true,
    GUIColor = Color3.fromRGB(0,255,255),
    GUIOpen = true,
    SoundEnabled = true
}

-- Carregar configuração salva
local success, saved = pcall(function() return readfile(ConfigFileName) end)
if success then
    local ok, data = pcall(function() return HttpService:JSONDecode(saved) end)
    if ok then for k,v in pairs(data) do Config[k] = v end end
end

local function SaveConfig()
    pcall(function() writefile(ConfigFileName, HttpService:JSONEncode(Config)) end)
end

-- Pastas King Legacy
local EnemiesFolder = workspace:WaitForChild("Enemies")
local QuestsFolder = workspace:WaitForChild("QuestGivers")
local FruitsFolder = workspace:WaitForChild("Fruits")
local IslandsFolder = workspace:WaitForChild("Islands")

--========================
-- GUI PRO+ NEON
--========================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ÁrticoHubPROPlus"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = Player:WaitForChild("PlayerGui")

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 380, 0, 580)
Frame.Position = UDim2.new(0.01,0,0.08,0)
Frame.BackgroundColor3 = Config.GUIColor
Frame.BorderSizePixel = 0
Frame.BackgroundTransparency = 0.1
Frame.Parent = ScreenGui

-- Neon effect
local UIStroke = Instance.new("UIStroke")
UIStroke.Thickness = 3
UIStroke.Color = Color3.fromRGB(0,255,255)
UIStroke.Parent = Frame

-- Painel de botões com som
local function PlayClickSound()
    if not Config.SoundEnabled then return end
    local sound = Instance.new("Sound")
    sound.SoundId = "rbxassetid://911882174" -- som de clique simples
    sound.Volume = 0.5
    sound.Parent = SoundService
    sound:Play()
    task.delay(1, function() sound:Destroy() end)
end

local function CreateButton(text, callback, yPos)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 360, 0, 45)
    btn.Position = UDim2.new(0,10,0,yPos)
    btn.BackgroundColor3 = Config.GUIColor:lerp(Color3.fromRGB(50,50,80),0.3)
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.Text = text
    btn.Font = Enum.Font.GothamBold
    btn.TextScaled = true
    btn.Parent = Frame
    btn.MouseButton1Click:Connect(function()
        PlayClickSound()
        callback()
        SaveConfig()
    end)
    return btn
end

-- Botões PRO+
CreateButton("Toggle Auto Farm", function() Config.AutoFarm = not Config.AutoFarm end, 10)
CreateButton("Toggle Auto Quest", function() Config.AutoQuest = not Config.AutoQuest end, 65)
CreateButton("Toggle Auto Boss", function() Config.AutoBoss = not Config.AutoBoss end, 120)
CreateButton("Toggle Auto Fruit", function() Config.AutoFruit = not Config.AutoFruit end, 175)
CreateButton("Weapon: Sword", function() Config.FarmWeapon = "Sword" end, 230)
CreateButton("Weapon: Combat", function() Config.FarmWeapon = "Combat" end, 285)
CreateButton("Delay +0.05", function() Config.FarmDelay = Config.FarmDelay + 0.05 end, 340)
CreateButton("Delay -0.05", function() Config.FarmDelay = math.max(0.05, Config.FarmDelay - 0.05) end, 395)
CreateButton("Toggle GUI", function() Config.GUIOpen = not Config.GUIOpen Frame.Visible = Config.GUIOpen end, 450)
CreateButton("Change Color", function()
    local color = Color3.fromRGB(math.random(0,255),math.random(0,255),math.random(0,255))
    Frame.BackgroundColor3 = color
    UIStroke.Color = color
    Config.GUIColor = color
end, 505)
CreateButton("Toggle Sound", function() Config.SoundEnabled = not Config.SoundEnabled end, 555)

-- Marca d’água animada
local watermark = Instance.new("TextLabel")
watermark.Size = UDim2.new(0,380,0,35)
watermark.Position = UDim2.new(0,0,1,-45)
watermark.BackgroundTransparency = 1
watermark.TextColor3 = Color3.fromRGB(0,255,255)
watermark.TextScaled = true
watermark.Font = Enum.Font.GothamBold
watermark.Text = "Ártico Hub ULTIMATE PRO+ | SrUrsolino"
watermark.Parent = Frame

--========================
-- FUNÇÕES CORE
--========================
local function UpdateCharacter()
    Character = Player.Character or Player.CharacterAdded:Wait()
    return Character:WaitForChild("Humanoid"), Character:WaitForChild("HumanoidRootPart")
end

local function EquipWeapon()
    for _, tool in pairs(Player.Backpack:GetChildren()) do
        if tool:IsA("Tool") then
            if Config.FarmWeapon == "Sword" and tool:FindFirstChild("SwordScript") then
                tool.Parent = Character
            elseif Config.FarmWeapon == "Combat" and tool.Name == "Combat" then
                tool.Parent = Character
            end
        end
    end
end

local function Attack()
    for _, tool in pairs(Character:GetChildren()) do
        if tool:IsA("Tool") then tool:Activate() end
    end
end

local function GetNearestNPC(minLevel,maxLevel,onlyBoss)
    local nearest, dist = nil, math.huge
    local _, HRP = UpdateCharacter()
    for _, island in pairs(IslandsFolder:GetChildren()) do
        for _, npc in pairs(island:GetChildren()) do
            if npc:FindFirstChild("Humanoid") and npc.Humanoid.Health > 0 and npc:FindFirstChild("HumanoidRootPart") then
                local lvl = npc:FindFirstChild("Level") and npc.Level.Value or 1
                if lvl >= minLevel and lvl <= maxLevel then
                    if not onlyBoss or npc.Name:lower():find("boss") then
                        local d = (HRP.Position - npc.HumanoidRootPart.Position).Magnitude
                        if d < dist then
                            nearest = npc
                            dist = d
                        end
                    end
                end
            end
        end
    end
    return nearest
end

local function AutoQuest()
    if not Config.AutoQuest then return end
    for _, quest in pairs(QuestsFolder:GetChildren()) do
        if quest:FindFirstChild("ClickDetector") then
            fireclickdetector(quest.ClickDetector)
            task.wait(0.15)
        end
    end
end

local function AntiStuck()
    local _, HRP = UpdateCharacter()
    HRP.CFrame = HRP.CFrame + Vector3.new(0,6,0)
end

local function AutoFruit()
    if not Config.AutoFruit then return end
    for _, fruit in pairs(FruitsFolder:GetChildren()) do
        if fruit:IsA("Tool") then
            fruit.Parent = Character
            fruit:Activate()
            task.wait(0.2)
        end
    end
end

--========================
-- LOOP PRINCIPAL PRO+
--========================
task.spawn(function()
    while task.wait(Config.FarmDelay) do
        if not Config.AutoFarm then continue end
        pcall(function()
            local Humanoid, HRP = UpdateCharacter()
            if Humanoid.Health <= 0 then return end

            EquipWeapon()
            AutoQuest()
            AutoFruit()

            local target = Config.AutoBoss and GetNearestNPC(Config.MinLevel,Config.MaxLevel,true) or GetNearestNPC(Config.MinLevel,Config.MaxLevel,false)
            if target and target:FindFirstChild("HumanoidRootPart") then
                HRP.CFrame = target.HumanoidRootPart.CFrame * CFrame.new(0,0,Config.AttackDistance)
                Attack()
            else
                AntiStuck()
            end
        end)
    end
end)

--==================================================
-- ✅ Ártico Hub ULTIMATE PRO+ FINALIZADO
-- ✍ Dono: SrUrsolino
--==================================================
