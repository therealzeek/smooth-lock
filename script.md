-- Configuration
local Settings = {
    SmoothingFactor = 0.449,
    MaxLockDistance = 1000,
    PredictionMultiplier = 10,
    PredictionOffset = Vector3.new(0, 0.26, 0) --// (X, Y, Z)
}

-- Controls
local Controls = {
    ControllerKey = Enum.KeyCode.ButtonY,
    KeyboardKey = Enum.KeyCode.C,
}

--Services
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

-- Variables
local Camera = game.Workspace.CurrentCamera
local isTargetLocked = false
local targetPlayer = nil

-- Functions
local function FindClosestPlayer()
    local closestDistance = Settings.MaxLockDistance
    local closestPlayer = nil
    local myPosition = game.Players.LocalPlayer.Character.PrimaryPart.Position
    for _, player in pairs(game.Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local targetPosition = player.Character.PrimaryPart.Position
            local distance = (targetPosition - myPosition).magnitude
            if distance < closestDistance then
                closestDistance = distance
                closestPlayer = player
            end
        end
    end
    return closestPlayer
end

local function PredictTargetPosition(targetPosition, speed)
    local timeToReachHorizontal = math.abs((targetPosition.X - Camera.CFrame.Position.X) / speed)
    local timeToReachVertical = math.abs((targetPosition.Y - Camera.CFrame.Position.Y) / speed)
    local predictedX = targetPosition.X + targetPlayer.Character.HumanoidRootPart.Velocity.X * timeToReachHorizontal
    local predictedY = targetPosition.Y + targetPlayer.Character.HumanoidRootPart.Velocity.Y * timeToReachVertical
    local predictedZ = targetPosition.Z + targetPlayer.Character.HumanoidRootPart.Velocity.Z * timeToReachHorizontal
    local predictedPosition = Vector3.new(predictedX, predictedY, predictedZ) + Settings.PredictionOffset
    return predictedPosition
end

-- Additional Function
local function ShowNotification(playerName)
    game.StarterGui:SetCore("SendNotification", {
        Title = "nara.cc",
        Text = "Locked: " .. playerName,
        Duration = 5
    })
end

-- Input Handlers
UserInputService.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Gamepad1 and input.KeyCode == Controls.ControllerKey then
        isTargetLocked = not isTargetLocked
        if isTargetLocked then
            targetPlayer = FindClosestPlayer()
            if targetPlayer then
                ShowNotification(targetPlayer.DisplayName)
            end
        else
            targetPlayer = nil
        end
    elseif input.KeyCode == Controls.KeyboardKey then
        isTargetLocked = not isTargetLocked
        if isTargetLocked then
            targetPlayer = FindClosestPlayer()
            ShowNotification(targetPlayer.DisplayName)
        else
            targetPlayer = nil
        end
    end
end)

-- Camera Update
RunService.Heartbeat:Connect(function()
    if isTargetLocked and targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
        local targetPosition = targetPlayer.Character.PrimaryPart.Position
        local speed = targetPlayer.Character.Humanoid.WalkSpeed * Settings.PredictionMultiplier
        targetPosition = PredictTargetPosition(targetPosition, speed)
        local cameraPosition = Camera.CFrame.Position
        local offsetDirection = (targetPosition - cameraPosition).unit
        local newCameraPosition = targetPosition - offsetDirection
        Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(newCameraPosition, targetPosition), Settings.SmoothingFactor)
    end
end)
