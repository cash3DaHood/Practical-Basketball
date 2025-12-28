-- Fridgehub Auto Green Module (Practical Basketball)
-- Set AutoGreenEnabled = true/false via UI toggle

local UIS = game:GetService("UserInputService")
local Stats = game:GetService("Stats")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local ShootRemote = ReplicatedStorage:WaitForChild("Aero"):WaitForChild("AeroRemoteServices"):WaitForChild("InputService"):WaitForChild("Shoot")

local AutoGreenEnabled = false  -- Control via UI
local HoldTime = 0.53
local CurrentHoldTask = nil

local function getPing()
    local success, ping = pcall(function()
        return Stats.Network.ServerStatsItem["Data Ping"]:GetValue()
    end)
    return success and math.max(ping or 0, 0) or 100
end

UIS.InputBegan:Connect(function(input, gpe)
    if gpe or not AutoGreenEnabled or input.KeyCode ~= Enum.KeyCode.E then return end
    
    local ping = getPing()
    local pingAdjust = ping / 1000
    local effective = HoldTime + pingAdjust
    
    if CurrentHoldTask then task.cancel(CurrentHoldTask) end
    
    print("[Fridgehub] Auto-release in " .. string.format("%.3f", effective) .. "s (ping: " .. math.floor(ping) .. "ms)")
    ShootRemote:FireServer({Shoot = true, Input = "E"})
    
    CurrentHoldTask = task.delay(effective, function()
        ShootRemote:FireServer({Shoot = false})
        print("[Fridgehub] PERFECT GREEN!")
        CurrentHoldTask = nil
    end)
end)

UIS.InputEnded:Connect(function(input, gpe)
    if gpe or input.KeyCode ~= Enum.KeyCode.E or not CurrentHoldTask then return end
    
    task.cancel(CurrentHoldTask)
    ShootRemote:FireServer({Shoot = false})
    print("[Fridgehub] Manual release")
    CurrentHoldTask = nil
end)

print("[Fridgehub] ✅ Module loaded! Toggle AutoGreenEnabled in UI.")