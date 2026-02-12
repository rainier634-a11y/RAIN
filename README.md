--=========================================================
-- RAIN EXTREME OPTIMIZER V2
-- Focus: Reduce entity load, disable heavy systems,
-- stabilize frame time, reduce spikes
--=========================================================

local ped = nil
local playerId = PlayerId()
local density = 0.0
local performanceMode = "NORMAL"
local lastCoords = vector3(0,0,0)

--=========================================================
-- PLAYER CACHE SYSTEM
--=========================================================

CreateThread(function()
    while true do
        ped = PlayerPedId()
        Wait(1000)
    end
end)

--=========================================================
-- DISABLE DISPATCH / RANDOM EVENTS
--=========================================================

CreateThread(function()
    for i=1,15 do
        EnableDispatchService(i,false)
    end

    SetCreateRandomCops(false)
    SetCreateRandomCopsOnScenarios(false)
    SetCreateRandomCopsNotOnScenarios(false)
    SetGarbageTrucks(false)
    SetRandomBoats(false)
end)

--=========================================================
-- ULTRA DENSITY CONTROL (Every Frame)
--=========================================================

CreateThread(function()
    while true do
        Wait(0)

        SetVehicleDensityMultiplierThisFrame(density)
        SetPedDensityMultiplierThisFrame(density)
        SetRandomVehicleDensityMultiplierThisFrame(density)
        SetParkedVehicleDensityMultiplierThisFrame(density)
        SetScenarioPedDensityMultiplierThisFrame(0.0, 0.0)
    end
end)

--=========================================================
-- AREA CLEAN SYSTEM
--=========================================================

CreateThread(function()
    while true do
        Wait(4000)

        local coords = GetEntityCoords(ped)

        ClearAreaOfVehicles(coords.x, coords.y, coords.z, 30.0, false, false, false, false, false)
        ClearAreaOfPeds(coords.x, coords.y, coords.z, 30.0, false)
        ClearAreaOfObjects(coords.x, coords.y, coords.z, 20.0, 0)
    end
end)

--=========================================================
-- HARD VEHICLE LIMITER
--=========================================================

CreateThread(function()
    while true do
        Wait(7000)

        local vehicles = GetGamePool('CVehicle')
        local playerCoords = GetEntityCoords(ped)

        for _,veh in pairs(vehicles) do
            if not IsPedAPlayer(GetPedInVehicleSeat(veh,-1)) then
                if #(GetEntityCoords(veh) - playerCoords) < 50.0 then
                    DeleteEntity(veh)
                end
            end
        end
    end
end)

--=========================================================
-- FRAME TIME MONITOR (Dynamic Performance Mode)
--=========================================================

CreateThread(function()
    while true do
        Wait(5000)

        local frameTime = GetFrameTime()

        if frameTime > 0.024 then
            performanceMode = "LOW"
            density = 0.0
        elseif frameTime > 0.018 then
            performanceMode = "BALANCED"
            density = 0.1
        else
            performanceMode = "HIGH"
            density = 0.3
        end
    end
end)

--=========================================================
-- AFK / MOVEMENT CHECK
--=========================================================

CreateThread(function()
    while true do
        Wait(3000)

        local coords = GetEntityCoords(ped)

        if #(coords - lastCoords) < 2.0 then
            density = 0.0
        end

        lastCoords = coords
    end
end)

--=========================================================
-- EFFECT DISABLER
--=========================================================

CreateThread(function()
    while true do
        Wait(1000)

        InvalidateIdleCam()
        InvalidateVehicleIdleCam()

        DisableVehicleDistantlights(true)
    end
end)

--=========================================================
-- AUDIO OPTIMIZATION
--=========================================================

CreateThread(function()
    StartAudioScene("CHARACTER_CHANGE_IN_SKY_SCENE")
    SetAudioFlag("PoliceScannerDisabled", true)
    SetAudioFlag("DisableFlightMusic", true)
end)

--=========================================================
-- REMOVE SCENARIOS
--=========================================================

CreateThread(function()
    local scenarios = {
        "WORLD_VEHICLE_POLICE_CAR",
        "WORLD_VEHICLE_POLICE_BIKE",
        "WORLD_VEHICLE_FIRE_TRUCK",
        "WORLD_VEHICLE_AMBULANCE",
        "WORLD_VEHICLE_TAXI"
    }

    for i=1,#scenarios do
        SetScenarioTypeEnabled(scenarios[i], false)
    end
end)

--=========================================================
-- POPULATION BUDGET LIMITER
--=========================================================

CreateThread(function()
    while true do
        Wait(2000)
        SetPedPopulationBudget(0)
        SetVehiclePopulationBudget(0)
    end
end)

--=========================================================
-- MEMORY CLEANER
--=========================================================

CreateThread(function()
    while true do
        Wait(10000)
        CollectGarbage()
    end
end)

--=========================================================
-- STAMINA REWARD DISABLE (reduce background calc)
--=========================================================

CreateThread(function()
    while true do
        Wait(2000)
        DisablePlayerVehicleRewards(playerId)
    end
end)

--=========================================================
-- FINAL ENTITY PURGE SYSTEM
--=========================================================

CreateThread(function()
    while true do
        Wait(15000)

        local objects = GetGamePool('CObject')

        for _,obj in pairs(objects) do
            if not DoesEntityBelongToThisScript(obj, false) then
                DeleteEntity(obj)
            end
        end
    end
end)
