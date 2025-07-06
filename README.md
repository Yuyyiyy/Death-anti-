-- ⚙️ SERVICES
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

-- 💪 LOCKED POSE: Arms hang straight down
local LOCKED_CFRAMES = {
    R6 = {
        Left = {C0 = CFrame.new(-1.5, 0.5, 0), C1 = CFrame.new(0.5, 0.5, 0)},
        Right = {C0 = CFrame.new(1.5, 0.5, 0), C1 = CFrame.new(-0.5, 0.5, 0)},
    },
    R15 = {
        LeftShoulder = {C0 = CFrame.new(-1, 0.5, 0), C1 = CFrame.new(0.5, 0.5, 0)},
        RightShoulder = {C0 = CFrame.new(1, 0.5, 0), C1 = CFrame.new(-0.5, 0.5, 0)},
        LeftElbow = {C0 = CFrame.new(0, -1, 0), C1 = CFrame.new(0, 1, 0)},
        RightElbow = {C0 = CFrame.new(0, -1, 0), C1 = CFrame.new(0, 1, 0)}
    }
}

-- 🔒 Store original joint properties for maximum lock
local jointConnections = {}

-- 🧊 Force-lock arms in place, override everything
local function freezeArms()
    if character:FindFirstChild("Torso") then
        -- R6 Rig
        local ls = character.Torso:FindFirstChild("Left Shoulder")
        local rs = character.Torso:FindFirstChild("Right Shoulder")
        
        if ls then 
            ls.C0 = LOCKED_CFRAMES.R6.Left.C0 
            ls.C1 = LOCKED_CFRAMES.R6.Left.C1 
            -- Lock additional properties to prevent any movement
            ls.MaxVelocity = 0
            ls.DesiredAngle = 0
        end
        if rs then 
            rs.C0 = LOCKED_CFRAMES.R6.Right.C0 
            rs.C1 = LOCKED_CFRAMES.R6.Right.C1 
            rs.MaxVelocity = 0
            rs.DesiredAngle = 0
        end
    elseif character:FindFirstChild("UpperTorso") then
        -- R15 Rig
        local us = character.UpperTorso
        local ls = us:FindFirstChild("LeftShoulder")
        local rs = us:FindFirstChild("RightShoulder")
        local le = character:FindFirstChild("LeftUpperArm") and character.LeftUpperArm:FindFirstChild("LeftElbow")
        local re = character:FindFirstChild("RightUpperArm") and character.RightUpperArm:FindFirstChild("RightElbow")

        if ls then 
            ls.C0 = LOCKED_CFRAMES.R15.LeftShoulder.C0 
            ls.C1 = LOCKED_CFRAMES.R15.LeftShoulder.C1 
        end
        if rs then 
            rs.C0 = LOCKED_CFRAMES.R15.RightShoulder.C0 
            rs.C1 = LOCKED_CFRAMES.R15.RightShoulder.C1 
        end
        if le then 
            le.C0 = LOCKED_CFRAMES.R15.LeftElbow.C0 
            le.C1 = LOCKED_CFRAMES.R15.LeftElbow.C1 
        end
        if re then 
            re.C0 = LOCKED_CFRAMES.R15.RightElbow.C0 
            re.C1 = LOCKED_CFRAMES.R15.RightElbow.C1 
        end
    end
end

-- ❌ Cancel all animations and prevent new ones
local function disableArmAnimations()
    local animator = humanoid:FindFirstChildOfClass("Animator")
    if animator then
        -- Stop all currently playing animations
        for _, track in pairs(animator:GetPlayingAnimationTracks()) do
            if track.Name ~= "Idle" then -- Keep idle for legs/torso
                track:Stop()
            end
        end
        
        -- Block new animation tracks from affecting arms
        for _, track in pairs(animator:GetPlayingAnimationTracks()) do
            track.Priority = Enum.AnimationPriority.Idle
        end
    end
    
    -- Disable humanoid states that might move arms
    humanoid:ChangeState(Enum.HumanoidStateType.Physics)
    humanoid.PlatformStand = true
    wait()
    humanoid.PlatformStand = false
end

-- 🛡️ Create change detection to instantly reset any modifications
local function setupJointProtection()
    -- Clear existing connections
    for _, connection in pairs(jointConnections) do
        connection:Disconnect()
    end
    jointConnections = {}
    
    local function protectJoint(joint, lockedC0, lockedC1)
        if joint then
            local connection = joint:GetPropertyChangedSignal("C0"):Connect(function()
                if joint.C0 ~= lockedC0 then
                    joint.C0 = lockedC0
                end
            end)
            table.insert(jointConnections, connection)
            
            local connection2 = joint:GetPropertyChangedSignal("C1"):Connect(function()
                if joint.C1 ~= lockedC1 then
                    joint.C1 = lockedC1
                end
            end)
            table.insert(jointConnections, connection2)
        end
    end
    
    -- Setup protection for all arm joints
    if character:FindFirstChild("Torso") then
        -- R6
        local ls = character.Torso:FindFirstChild("Left Shoulder")
        local rs = character.Torso:FindFirstChild("Right Shoulder")
        protectJoint(ls, LOCKED_CFRAMES.R6.Left.C0, LOCKED_CFRAMES.R6.Left.C1)
        protectJoint(rs, LOCKED_CFRAMES.R6.Right.C0, LOCKED_CFRAMES.R6.Right.C1)
    elseif character:FindFirstChild("UpperTorso") then
        -- R15
        local us = character.UpperTorso
        local ls = us:FindFirstChild("LeftShoulder")
        local rs = us:FindFirstChild("RightShoulder")
        local le = character:FindFirstChild("LeftUpperArm") and character.LeftUpperArm:FindFirstChild("LeftElbow")
        local re = character:FindFirstChild("RightUpperArm") and character.RightUpperArm:FindFirstChild("RightElbow")
        
        protectJoint(ls, LOCKED_CFRAMES.R15.LeftShoulder.C0, LOCKED_CFRAMES.R15.LeftShoulder.C1)
        protectJoint(rs, LOCKED_CFRAMES.R15.RightShoulder.C0, LOCKED_CFRAMES.R15.RightShoulder.C1)
        protectJoint(le, LOCKED_CFRAMES.R15.LeftElbow.C0, LOCKED_CFRAMES.R15.LeftElbow.C1)
        protectJoint(re, LOCKED_CFRAMES.R15.RightElbow.C0, LOCKED_CFRAMES.R15.RightElbow.C1)
    end
end

-- 🔄 Initialize the lock system
local function initializeLock()
    wait(0.5) -- Small delay to ensure character is fully loaded
    setupJointProtection()
    freezeArms()
    disableArmAnimations()
end

-- 🔁 Update constantly to fight back against anything trying to move arms
RunService.RenderStepped:Connect(function()
    freezeArms()
end)

RunService.Heartbeat:Connect(function()
    freezeArms()
    disableArmAnimations()
end)

RunService.Stepped:Connect(freezeArms)

-- ♻️ Reapply when character respawns
player.CharacterAdded:Connect(function(char)
    character = char
    humanoid = character:WaitForChild("Humanoid")
    initializeLock()
end)

-- 🚀 Initialize for current character
initializeLock()

print("✅ ULTRA-LOCKED Arms: Now completely frozen even while moving!")
print("🔒 Real-time protection active - arms will NEVER move until you leave.")
