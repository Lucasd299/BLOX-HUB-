-- BLOX HUB COMPLETO v3.0

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- CONFIGURAÇÕES INICIAIS
local config = {
    autoFarm = true,
    targetNPCName = "Bandit",
    autoPickup = true,
    autoBuySkills = true,
    autoEquipWeapon = true,
    esp = true,
    antiKick = true,
}

local islands = {
    ["Starter Island"] = Vector3.new(-145, 7, 1439),
    ["Pirate Island"] = Vector3.new(1212, 7, 1058),
    ["Desert Island"] = Vector3.new(3492, 7, -1472),
    ["Sky Island"] = Vector3.new(789, 1017, -1725),
    ["Marineford"] = Vector3.new(-502, 7, 1033),
}

-- UTILITÁRIOS
local function teleportToTarget(target)
    if target and target:FindFirstChild("HumanoidRootPart") then
        LocalPlayer.Character.HumanoidRootPart.CFrame = target.HumanoidRootPart.CFrame * CFrame.new(0, 0, 3)
    elseif target and target:IsA("BasePart") then
        LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(target.Position)
    end
end

local function teleportToIsland(name)
    local pos = islands[name]
    if pos then
        LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(pos)
        print("Teleportado para: "..name)
    else
        warn("Ilha não encontrada: "..name)
    end
end

-- FUNÇÕES DO RLX HUB

-- AutoFarm
local autoFarmRunning = false
local function startAutoFarm()
    if autoFarmRunning then return end
    autoFarmRunning = true
    spawn(function()
        while config.autoFarm do
            local enemies = Workspace:FindFirstChild("Enemies")
            if enemies then
                for _, npc in pairs(enemies:GetChildren()) do
                    if npc.Name == config.targetNPCName and npc:FindFirstChild("Humanoid") and npc.Humanoid.Health > 0 then
                        teleportToTarget(npc)
                        wait(0.2)
                        pcall(function()
                            npc.Humanoid:TakeDamage(50)
                        end)
                        wait(0.5)
                    end
                end
            end
            wait(1)
        end
        autoFarmRunning = false
    end)
end

-- AutoPickup
local autoPickupRunning = false
local function autoPickupItems()
    if autoPickupRunning then return end
    autoPickupRunning = true
    spawn(function()
        while config.autoPickup do
            for _, item in pairs(Workspace:GetChildren()) do
                if (item.Name == "Chest" or item.Name == "Flower" or item.Name == "Devil Fruit" or item.Name == "Fruit") and item:IsA("BasePart") then
                    if (item.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude < 30 then
                        teleportToTarget(item)
                        wait(0.3)
                    end
                end
            end
            wait(2)
        end
        autoPickupRunning = false
    end)
end

-- AutoBuySkills
local autoBuyRunning = false
local function autoBuySkills()
    if autoBuyRunning then return end
    autoBuyRunning = true
    spawn(function()
        while config.autoBuySkills do
            pcall(function()
                local playerGui = LocalPlayer:WaitForChild("PlayerGui")
                local mainGui = playerGui:WaitForChild("Main")
                local skillsGui = mainGui:WaitForChild("Skills")
                local buyEvent = skillsGui:WaitForChild("Buy")
                for i = 1, 4 do
                    buyEvent:FireServer(i)
                end
            end)
            wait(5)
        end
        autoBuyRunning = false
    end)
end

-- AutoEquip
local autoEquipRunning = false
local function autoEquip()
    if autoEquipRunning then return end
    autoEquipRunning = true
    spawn(function()
        while config.autoEquipWeapon do
            local backpack = LocalPlayer:WaitForChild("Backpack")
            local character = LocalPlayer.Character
            if backpack and character then
                for _, item in pairs(backpack:GetChildren()) do
                    if item:IsA("Tool") then
                        pcall(function()
                            item.Parent = character
                        end)
                    end
                end
            end
            wait(10)
        end
        autoEquipRunning = false
    end)
end

-- ESP
local espObjects = {}

local function createESP(adornParent, color)
    if espObjects[adornParent] then return end
    local box = Instance.new("BoxHandleAdornment")
    box.Adornee = adornParent
    box.AlwaysOnTop = true
    box.ZIndex = 10
    box.Size = Vector3.new(4, 6, 4)
    box.Transparency = 0.5
    box.Color3 = color
    box.Parent = adornParent
    espObjects[adornParent] = box
end

local function enableESP()
    if not config.esp then return end
    -- Limpa ESP antigos
    for adornee, box in pairs(espObjects) do
        box:Destroy()
        espObjects[adornee] = nil
    end
    -- Jogadores
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            createESP(player.Character.HumanoidRootPart, Color3.new(0, 1, 0)) -- verde
        end
    end
    -- NPCs
    local enemies = Workspace:FindFirstChild("Enemies")
    if enemies then
        for _, npc in pairs(enemies:GetChildren()) do
            if npc:FindFirstChild("HumanoidRootPart") then
                createESP(npc.HumanoidRootPart, Color3.new(1, 0, 0)) -- vermelho
            end
        end
    end
    -- Baús e frutas
    for _, item in pairs(Workspace:GetChildren()) do
        if (item.Name == "Chest" or item.Name == "Flower" or item.Name == "Devil Fruit" or item.Name == "Fruit") and item:IsA("BasePart") then
            createESP(item, Color3.new(1, 1, 0)) -- amarelo
        end
    end
end

-- AntiKick
local function antiKick()
    local mt = getrawmetatable(game)
    setreadonly(mt, false)
    local oldNamecall = mt.__namecall
    mt.__namecall = newcclosure(function(self, ...)
        local method = getnamecallmethod()
        if method == "Kick" then
            print("Kick bloqueado!")
            return wait(9e9)
        end
        return oldNamecall(self, ...)
    end)
    setreadonly(mt, true)
end

-- Inicialização RLX HUB
if config.antiKick then
    antiKick()
end

if config.autoFarm then
    startAutoFarm()
end

if config.autoPickup then
    autoPickupItems()
end

if config.autoBuySkills then
    autoBuySkills()
end

if config.autoEquipWeapon then
    autoEquip()
end

if config.esp then
    enableESP()
end

print("RLX HUB 3.0 iniciado!")

-- =================
-- GUI AVANÇADA
-- =================

local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "RLXHubGUI"
screenGui.Parent = PlayerGui
screenGui.ResetOnSpawn = false

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 300, 0, 480)
frame.Position = UDim2.new(0, 20, 0, 20)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BorderSizePixel = 0
frame.Parent = screenGui

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 45)
title.BackgroundTransparency = 1
title.Text = "RLX HUB v3.0"
title.TextColor3 = Color3.new(1, 1, 1)
title.Font = Enum.Font.GothamBold
title.TextSize = 22
title.Parent = frame

-- Função para criar botão
local function createButton(text, posY)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, -30, 0, 40)
    btn.Position = UDim2.new(0, 15, 0, posY)
    btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.GothamSemibold
    btn.TextSize = 18
    btn.Text = text
    btn.AutoButtonColor = true
    btn.Parent = frame
    return btn
end

-- Função para criar Label
local function createLabel(text, posY)
    local lbl = Instance.new("TextLabel")
    lbl.Size = UDim2.new(1, -30, 0, 25)
    lbl.Position = UDim2.new(0, 15, 0, posY)
    lbl.BackgroundTransparency = 1
    lbl.TextColor3 = Color3.new(1, 1, 1)
    lbl.Font = Enum.Font.GothamSemibold
    lbl.TextSize = 16
    lbl.Text = text
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    lbl.Parent = frame
    return lbl
end

-- Função para criar dropdown (simples)
local function createDropdown(options, posY, default)
    local dropdown = Instance.new("TextButton")
    dropdown.Size = UDim2.new(1, -30, 0, 30)
    dropdown.Position = UDim2.new(0, 15, 0, posY)
    dropdown.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    dropdown.TextColor3 = Color3.new(1,1,1)
    dropdown.Font = Enum.Font.GothamSemibold
    dropdown.TextSize = 16
    dropdown.Text = default or options[1]
    dropdown.AutoButtonColor = true
    dropdown.Parent = frame

    local listFrame = Instance.new("Frame")
    listFrame.Size = UDim2.new(1, 0, 0, #options * 25)
    listFrame.Position = UDim2.new(0, 0, 1, 0)
    listFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    listFrame.Visible = false
    listFrame.Parent = dropdown

    for i, option in ipairs(options) do
        local optionBtn = Instance.new("TextButton")
        optionBtn.Size = UDim2.new(1, 0, 0, 25)
        optionBtn.Position = UDim2.new(0, 0, 0, (i-1)*25)
        optionBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        optionBtn.TextColor3 = Color3.new(1,1,1)
        optionBtn.Font = Enum.Font.GothamSemibold
        optionBtn.TextSize = 14
        optionBtn.Text = option
        optionBtn.Parent = listFrame

        optionBtn.MouseButton1Click:Connect(function()
            dropdown.Text = option
            listFrame.Visible = false
            if options == npcList then
                config.targetNPCName = option
            elseif options == islandNames then
                teleportToIsland(option)
            end
        end)
    end

    dropdown.MouseButton1Click:Connect(function()
        listFrame.Visible = not listFrame.Visible
    end)

    return dropdown
end

-- Listas para dropdown
local npcList = {"Bandit", "Arlong", "Marine", "Pirate", "Zombie"}

local islandNames = {}
for name,_ in pairs(islands) do
    table.insert(islandNames, name)
end

-- Criar botões e dropdowns
local y = 60

local autoFarmBtn = createButton("AutoFarm: ON", y); y = y + 45
local autoPickupBtn = createButton("AutoPickup: ON", y); y = y + 45
local autoBuyBtn = createButton("AutoBuySkills: ON", y); y = y + 45
local autoEquipBtn = createButton("AutoEquip: ON", y); y = y + 45
local espBtn = createButton("ESP: ON", y); y = y + 45

local npcDropdownLbl = createLabel("Escolha NPC para farm:", y); y = y + 25
local npcDropdown = createDropdown(npcList, y, config.targetNPCName); y = y + (25 * #npcList) + 10

local islandDropdownLbl = createLabel("Escolha ilha para teleporte:", y); y = y + 25
local islandDropdown = createDropdown(islandNames, y, islandNames[1]); y = y + (25 * #islandNames) + 10

-- Botões toggle
autoFarmBtn.MouseButton1Click:Connect(function()
    config.autoFarm =
