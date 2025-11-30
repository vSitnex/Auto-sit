local allowedGameId = 7267631004

if game.PlaceId ~= allowedGameId then
    warn("Este script solo se puede ejecutar en 'Carry People Simulator 3'.")
    return
end

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local AutoSitEnabled = false
local Busy = false
local DisimularEnabled = false
local HumanDelayEnabled = false

local function isAlive()
    local char = LocalPlayer.Character
    if not char then return false end
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if not humanoid then return false end
    if humanoid.Health <= 0 then return false end
    if humanoid:GetState() == Enum.HumanoidStateType.Dead then return false end
    return true
end

local function ForceSit()
    local character = LocalPlayer.Character
    if not character then return false end
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return false end
    humanoid.Sit = true
    return humanoid.Sit
end

local function ForceJump()
    local character = LocalPlayer.Character
    if not character then return false end
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return false end
    humanoid.Jump = true
    return true
end

local function ExecuteSitJumpImproved(isAuto)
    if not isAlive() then return end
    if Busy then return end
    Busy = true

    if HumanDelayEnabled and math.random(1,100) <= 30 then
        task.wait(math.random(40,140)/1000)
    end

    local sitSuccess = ForceSit()

    local humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
    if humanoid and sitSuccess then
        local startTime = os.clock()
        while not humanoid.Sit and os.clock() - startTime < 0.6 do
            task.wait()
            if not isAlive() then
                Busy = false
                return
            end
            if humanoid.Sit then break end
        end

        if humanoid.Sit then
            if isAuto then
                if HumanDelayEnabled then
                    task.wait(math.random(120,180)/1000)
                else
                    task.wait(0.02)
                end
            else
                if HumanDelayEnabled and math.random(1,100) <= 40 then
                    task.wait(math.random(30,120)/1000)
                end
            end

            if not isAlive() then
                Busy = false
                return
            end

            humanoid.Sit = false
            task.wait(0.05)
            task.wait(0.4)
            if isAlive() then
                ForceJump()
            end
        end
    end

    Busy = false
end

local function ExecuteManualSit()
    if not isAlive() then return end
    if Busy then return end
    Busy = true

    local sitSuccess = ForceSit()

    if sitSuccess then
        task.wait(0.15)
        local humanoid = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.Sit = false
        end
        task.wait(0.05)
        task.wait(0.4)
        if isAlive() then
            ForceJump()
        end
    end

    Busy = false
end

local function IsInAbnormalState()
    local character = LocalPlayer.Character
    if not character then return false end
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    if not humanoid or not rootPart then return false end
    if humanoid.PlatformStand then
        return true
    end
    local AngleX = math.abs(rootPart.Orientation.X)
    local AngleZ = math.abs(rootPart.Orientation.Z)
    if AngleX > 70 or AngleZ > 70 then
        return true
    end
    return false
end

local function AutoCheckImproved()
    if not AutoSitEnabled then return end
    if not isAlive() then return end
    if IsInAbnormalState() then
        if not Busy and isAlive() then
            ExecuteSitJumpImproved(true)
        end
    end
end

local function DragThingy(ui, dragui)
    if not dragui then dragui = ui end
    local UserInputService = game:GetService("UserInputService")
    local dragging
    local dragInput
    local dragStart
    local startPos

    local function update(input)
        local delta = input.Position - dragStart
        ui.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end

    dragui.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = ui.Position

            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    dragui.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            update(input)
        end
    end)
end

local ActionScreenGui = Instance.new("ScreenGui")
ActionScreenGui.Name = "AutoSitGUI"
ActionScreenGui.ResetOnSpawn = false
ActionScreenGui.Parent = game:GetService("CoreGui")

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 120, 0, 80)
MainFrame.Position = UDim2.new(0.5, -60, 0.3, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
MainFrame.BackgroundTransparency = 0.3
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ActionScreenGui

local UIStroke = Instance.new("UIStroke")
UIStroke.Color = Color3.fromRGB(80, 80, 80)
UIStroke.Thickness = 2
UIStroke.Parent = MainFrame

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 8)
UICorner.Parent = MainFrame

local AutoSitButton = Instance.new("TextButton")
AutoSitButton.Name = "AutoSitButton"
AutoSitButton.Size = UDim2.new(0.8, 0, 0.6, 0)
AutoSitButton.Position = UDim2.new(0.1, 0, 0.1, 0)
AutoSitButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
AutoSitButton.BackgroundTransparency = 0.4
AutoSitButton.Text = "AUTO SIT\nOFF"
AutoSitButton.TextColor3 = Color3.fromRGB(255, 100, 100)
AutoSitButton.Font = Enum.Font.GothamBold
AutoSitButton.TextSize = 12
AutoSitButton.TextWrapped = true
AutoSitButton.Parent = MainFrame

local ButtonCorner = Instance.new("UICorner")
ButtonCorner.CornerRadius = UDim.new(0, 6)
ButtonCorner.Parent = AutoSitButton

local ButtonStroke = Instance.new("UIStroke")
ButtonStroke.Color = Color3.fromRGB(100, 100, 100)
ButtonStroke.Thickness = 1
ButtonStroke.Parent = AutoSitButton

local function UpdateButtons()
    if AutoSitEnabled then
        AutoSitButton.Text = "AUTO SIT\nON"
        AutoSitButton.TextColor3 = Color3.fromRGB(100, 255, 100)
        AutoSitButton.BackgroundColor3 = Color3.fromRGB(40, 80, 40)
    else
        AutoSitButton.Text = "AUTO SIT\nOFF"
        AutoSitButton.TextColor3 = Color3.fromRGB(255, 100, 100)
        AutoSitButton.BackgroundColor3 = Color3.fromRGB(80, 40, 40)
    end
end

AutoSitButton.MouseEnter:Connect(function()
    TweenService:Create(AutoSitButton, TweenInfo.new(0.2), {
        BackgroundTransparency = 0.2
    }):Play()
end)

AutoSitButton.MouseLeave:Connect(function()
    TweenService:Create(AutoSitButton, TweenInfo.new(0.2), {
        BackgroundTransparency = 0.4
    }):Play()
end)

AutoSitButton.MouseButton1Click:Connect(function()
    AutoSitEnabled = not AutoSitEnabled
    UpdateButtons()
end)

DragThingy(MainFrame, MainFrame)

UpdateButtons()

RunService.Heartbeat:Connect(function()
    local success, err = pcall(AutoCheckImproved)
    if not success then
        warn("Error en AutoCheckImproved:", err)
    end
end)

LocalPlayer.CharacterAdded:Connect(function(char)
    local humanoid = char:FindFirstChildOfClass("Humanoid") or char:WaitForChild("Humanoid", 5)
    if humanoid then
        humanoid.Died:Connect(function()
            Busy = false
            UpdateButtons()
        end)
    end
end)

print("✅ AutoSit mejorado cargado - Diseño movible")
