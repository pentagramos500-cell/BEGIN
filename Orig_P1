getgenv().PL_Hub = getgenv().PL_Hub or {}
local Env = getgenv().PL_Hub

Env.Version = "1.0.0"

Env.Services = {
    Players = game:GetService("Players"),
    RunService = game:GetService("RunService"),
    UserInputService = game:GetService("UserInputService"),
    ReplicatedStorage = game:GetService("ReplicatedStorage"),
    StarterGui = game:GetService("StarterGui"),
    TweenService = game:GetService("TweenService"),
    CoreGui = game:GetService("CoreGui"),
    GuiService = game:GetService("GuiService"),
    Teams = game:GetService("Teams"),
    HttpService = game:GetService("HttpService"),
    Debris = game:GetService("Debris"),
    SoundService = game:GetService("SoundService")
}

local Players = Env.Services.Players
local LocalPlayer = Players.LocalPlayer

Env.Teams = {
    Guards = Env.Services.Teams:FindFirstChild("Guards"),
    Inmates = Env.Services.Teams:FindFirstChild("Inmates"),
    Criminals = Env.Services.Teams:FindFirstChild("Criminals")
}

Env.cfg = {
    enabled = false,
    teamcheck = true,
    wallcheck = true,
    deathcheck = true,
    ffcheck = true,
    hostilecheck = true,
    trespasscheck = true,
    vehiclecheck = true,
    criminalsnoinnmates = true,
    inmatesnocriminals = true,
    shieldbreaker = false,
    shieldfrontangle = 0.3,
    shieldrandomhead = false,
    shieldheadchance = 30,
    taserbypasshostile = false,
    taserbypasstrespass = false,
    taseralwayshit = false,
    ifplayerstill = false,
    stillthreshold = 0.5,
    hitchance = 100,
    hitchanceAutoOnly = false,
    missspread = 5,
    shotgunnaturalspread = false,
    shotgungamehandled = false,
    prioritizeclosest = false,
    targetstickiness = false,
    targetstickinessduration = 0.6,
    targetstickinessrandom = false,
    targetstickinessmin = 0.3,
    targetstickinessmax = 0.7,
    fov = 120,
    showfov = true,
    showtargetline = false,
    togglekey = Enum.KeyCode.RightShift,
    aimpart = "Head",
    randomparts = true,
    partslist = {"Head", "Torso", "Left Arm", "Right Arm", "Left Leg", "Right Leg", "HumanoidRootPart"},
    esp = false,
    espteamcheck = true,
    espshowteam = false,
    esptargets = {guards = true, inmates = true, criminals = true},
    espmaxdist = 500,
    espshowdist = true,
    esptoggle = Enum.KeyCode.RightControl,
    espcolor = Color3.fromRGB(0, 170, 255),
    espguards = Color3.fromRGB(0, 170, 255),
    espinmates = Color3.fromRGB(255, 150, 50),
    espcriminals = Color3.fromRGB(255, 60, 60),
    espteam = Color3.fromRGB(60, 255, 60),
    espuseteamcolors = true,
    autoshoot = true,
    autoshootdelay = 0.12,
    autoshootstartdelay = 0.2,
    c4esp = false,
    c4esptoggle = Enum.KeyCode.B,
    c4espcolor = Color3.fromRGB(80, 255, 80),
    c4espmaxdist = 200,
    c4espshowdist = true,
    hitsound = {
        enabled = true,
        volume = 2,
        normal = 1129547534,
        headshot = 108290646184559
    },
    tracer = {
        enabled = true,
        color = Color3.fromRGB(255, 255, 255),
        thickness = 0.12,
        lifetime = 0.15,
        fade = 0.2
    },
    hitmarker = {
        enabled = true,
        thickness = 1.5,
        length = 6,
        gap = 3,
        expand = 8,
        headExpand = 14
    },
    crosshair = {
        customImage = ""
    }
}

Env.wallParams = RaycastParams.new()
Env.wallParams.FilterType = Enum.RaycastFilterType.Exclude
Env.wallParams.IgnoreWater = true
Env.wallParams.RespectCanCollide = false
Env.wallParams.CollisionGroup = "ClientBullet"

Env.vars = {
    currentGun = nil,
    rng = Random.new(),
    lastShotTime = 0,
    lastShotResult = false,
    currentTarget = nil,
    targetSwitchTime = 0,
    currentStickiness = 0
}

Env.partMap = {
    ["Torso"] = {"Torso", "UpperTorso", "LowerTorso"},
    ["LeftArm"] = {"Left Arm", "LeftUpperArm", "LeftLowerArm", "LeftHand"},
    ["RightArm"] = {"Right Arm", "RightUpperArm", "RightLowerArm", "RightHand"},
    ["LeftLeg"] = {"Left Leg", "LeftUpperLeg", "LeftLowerLeg", "LeftFoot"},
    ["RightLeg"] = {"Right Leg", "RightUpperLeg", "RightLowerLeg", "RightFoot"}
}

function Env.getPart(char, name)
    if not char then return nil end
    local p = char:FindFirstChild(name)
    if p then return p end
    local maps = Env.partMap[name]
    if maps then
        for _, n in ipairs(maps) do
            local part = char:FindFirstChild(n)
            if part then return part end
        end
    end
    return char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("Head")
end

function Env.isHeadshot(hitPart)
    if hitPart.Name == "Head" then return true end
    
    local char = hitPart:FindFirstAncestorOfClass("Model")
    if not char then return false end
    
    local head = char:FindFirstChild("Head")
    if not head then return false end
    
    if hitPart:IsA("Accessory") or hitPart:IsA("Hat") then
        local attachment = hitPart:FindFirstChildOfClass("Attachment")
        if attachment then
            local headAttachment = head:FindFirstChild(attachment.Name)
            if headAttachment then return true end
        end
    end
    
    if hitPart.Parent and hitPart.Parent:IsA("Accessory") then
        local accessory = hitPart.Parent
        local handle = accessory:FindFirstChild("Handle")
        if handle then
            local attachment = handle:FindFirstChildOfClass("Attachment")
            if attachment then
                local headAttachment = head:FindFirstChild(attachment.Name)
                if headAttachment then return true end
            end
        end
    end
    
    return false
end

function Env.getTargetPart(char)
    if not char then return nil end
    if Env.cfg.shieldbreaker then
        local shield = char:FindFirstChild("RiotShieldPart")
        if shield and shield:IsA("BasePart") then
            local hp = shield:GetAttribute("Health")
            if hp and hp > 0 then
                local myChar = LocalPlayer.Character
                local myHrp = myChar and myChar:FindFirstChild("HumanoidRootPart")
                local theirHrp = char:FindFirstChild("HumanoidRootPart")
                if myHrp and theirHrp then
                    local toMe = (myHrp.Position - theirHrp.Position).Unit
                    local theirLook = theirHrp.CFrame.LookVector
                    local dot = toMe:Dot(theirLook)
                    if dot > Env.cfg.shieldfrontangle then
                        if Env.cfg.shieldrandomhead and Env.vars.rng:NextInteger(1, 100) <= Env.cfg.shieldheadchance then
                            return Env.getPart(char, "Head")
                        end
                        return shield
                    end
                end
            end
        end
    end
    local partName
    if Env.cfg.randomparts then
        local list = Env.cfg.partslist
        partName = (list and #list > 0) and list[Env.vars.rng:NextInteger(1, #list)] or "Head"
    else
        partName = Env.cfg.aimpart
    end
    return Env.getPart(char, partName)
end

function Env.isDead(player)
    if not player or not player.Character then return true end
    local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
    return not humanoid or humanoid.Health <= 0
end

function Env.isStanding(player)
    if not player or not player.Character then return false end
    local hrp = player.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return false end
    local vel = hrp.AssemblyLinearVelocity
    return Vector2.new(vel.X, vel.Z).Magnitude <= Env.cfg.stillthreshold
end

function Env.hasForceField(player)
    if not player or not player.Character then return false end
    return player.Character:FindFirstChildOfClass("ForceField") ~= nil
end

function Env.isInVehicle(player)
    if not player or not player.Character then return false end
    local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return false end
    return humanoid.SeatPart ~= nil
end

function Env.wallBetween(startPos, endPos, targetChar)
    local myChar = LocalPlayer.Character
    if not myChar then return true end
    local filter = {myChar}
    if targetChar then table.insert(filter, targetChar) end
    Env.wallParams.FilterDescendantsInstances = filter
    local direction = endPos - startPos
    local distance = direction.Magnitude
    local unit = direction.Unit
    local currentStart = startPos
    local remaining = distance
    for _ = 1, 10 do
        local result = workspace:Raycast(currentStart, unit * remaining, Env.wallParams)
        if not result then return false end
        local hit = result.Instance
        if hit.Transparency < 0.8 and hit.CanCollide then return true end
        local hitDist = (result.Position - currentStart).Magnitude
        remaining = remaining - hitDist - 0.01
        if remaining <= 0 then return false end
        currentStart = result.Position + unit * 0.01
    end
    return false
end

function Env.quickCheck(player)
    if not player or player == LocalPlayer or not player.Character then return false end
    if not Env.getTargetPart(player.Character) then return false end
    if Env.cfg.deathcheck and Env.isDead(player) then return false end
    if Env.cfg.ffcheck and Env.hasForceField(player) then return false end
    if Env.cfg.vehiclecheck and Env.isInVehicle(player) then return false end
    if Env.cfg.teamcheck and player.Team == LocalPlayer.Team then return false end
    if Env.cfg.criminalsnoinnmates then
        if LocalPlayer.Team == Env.Teams.Criminals and player.Team == Env.Teams.Inmates then return false end
    end
    if Env.cfg.inmatesnocriminals then
        if LocalPlayer.Team == Env.Teams.Inmates and player.Team == Env.Teams.Criminals then return false end
    end
    if Env.cfg.hostilecheck or Env.cfg.trespasscheck then
        local isTaser = Env.vars.currentGun and Env.vars.currentGun:GetAttribute("Projectile") == "Taser"
        local bypassHostile = Env.cfg.taserbypasshostile and isTaser
        local bypassTrespass = Env.cfg.taserbypasstrespass and isTaser
        local targetChar = player.Character
        if LocalPlayer.Team == Env.Teams.Guards and player.Team == Env.Teams.Inmates then
            local hostile = targetChar:GetAttribute("Hostile")
            local trespass = targetChar:GetAttribute("Trespassing")
            if Env.cfg.hostilecheck and Env.cfg.trespasscheck then
                if not bypassHostile and not bypassTrespass then
                    if not hostile and not trespass then return false end
                end
            elseif Env.cfg.hostilecheck and not bypassHostile then
                if not hostile then return false end
            elseif Env.cfg.trespasscheck and not bypassTrespass then
                if not trespass then return false end
            end
        end
    end
    return true
end

function Env.fullCheck(player)
    if not Env.quickCheck(player) then return false end
    if Env.cfg.wallcheck then
        local myChar = LocalPlayer.Character
        local myHead = myChar and myChar:FindFirstChild("Head")
        local targetPart = Env.getTargetPart(player.Character)
        if myHead and targetPart then
            if Env.wallBetween(myHead.Position, targetPart.Position, player.Character) then
                return false
            end
        end
    end
    return true
end

function Env.rollHit()
    local now = os.clock()
    if now - Env.vars.lastShotTime > 0.15 then
        Env.vars.lastShotTime = now
        local chance = Env.cfg.hitchance
        if chance >= 100 then
            Env.vars.lastShotResult = true
        elseif chance <= 0 then
            Env.vars.lastShotResult = false
        else
            Env.vars.lastShotResult = Env.vars.rng:NextInteger(1, 100) <= chance
        end
    end
    return Env.vars.lastShotResult
end

function Env.getMissPos(targetPos)
    local spread = Env.cfg.missspread
    local angle = Env.vars.rng:NextNumber() * math.pi * 2
    local d = Env.vars.rng:NextNumber() * spread
    local yOffset = (Env.vars.rng:NextNumber() - 0.5) * spread
    return targetPos + Vector3.new(math.cos(angle) * d, yOffset, math.sin(angle) * d)
end

function Env.getClosest(fovRadius)
    fovRadius = fovRadius or Env.cfg.fov
    local camera = workspace.CurrentCamera
    if not camera then return nil, nil end

    local lastInput = Env.Services.UserInputService:GetLastInputType()
    local locked = (lastInput == Enum.UserInputType.Touch) or (Env.Services.UserInputService.MouseBehavior == Enum.MouseBehavior.LockCenter)

    local aimPos
    if locked then
        local viewportSize = camera.ViewportSize
        aimPos = Vector2.new(viewportSize.X / 2, viewportSize.Y / 2)
    else
        aimPos = Env.Services.UserInputService:GetMouseLocation()
    end

    local now = os.clock()

    if Env.cfg.targetstickiness and Env.vars.currentTarget and (now - Env.vars.targetSwitchTime) < Env.vars.currentStickiness then
        if Env.fullCheck(Env.vars.currentTarget) then
            local part = Env.getTargetPart(Env.vars.currentTarget.Character)
            if part then
                local screenPos, onScreen = camera:WorldToViewportPoint(part.Position)
                if onScreen and screenPos.Z > 0 then
                    local dist = (Vector2.new(screenPos.X, screenPos.Y) - aimPos).Magnitude
                    if dist < fovRadius then
                        return Env.vars.currentTarget, part.Position
                    end
                end
            end
        end
    end

    local candidates = {}
    for _, player in ipairs(Players:GetPlayers()) do
        if Env.quickCheck(player) then
            local part = Env.getTargetPart(player.Character)
            if part then
                local screenPos, onScreen = camera:WorldToViewportPoint(part.Position)
                if onScreen and screenPos.Z > 0 then
                    local dist = (Vector2.new(screenPos.X, screenPos.Y) - aimPos).Magnitude
                    if dist < fovRadius then
                        candidates[#candidates + 1] = {player = player, dist = dist, part = part}
                    end
                end
            end
        end
    end

    if Env.cfg.prioritizeclosest then
        table.sort(candidates, function(a, b) return a.dist < b.dist end)
    else
        for i = #candidates, 2, -1 do
            local j = Env.vars.rng:NextInteger(1, i)
            candidates[i], candidates[j] = candidates[j], candidates[i]
        end
    end

    for _, candidate in ipairs(candidates) do
        if Env.fullCheck(candidate.player) then
            if candidate.player ~= Env.vars.currentTarget then
                Env.vars.currentTarget = candidate.player
                Env.vars.targetSwitchTime = now
                if Env.cfg.targetstickinessrandom then
                    Env.vars.currentStickiness = Env.vars.rng:NextNumber(Env.cfg.targetstickinessmin, Env.cfg.targetstickinessmax)
                else
                    Env.vars.currentStickiness = Env.cfg.targetstickinessduration
                end
            end
            return candidate.player, candidate.part.Position
        end
    end

    Env.vars.currentTarget = nil
    return nil, nil
end

local Camera = workspace.CurrentCamera
local HitSound = Instance.new("Sound")
HitSound.Volume = Env.cfg.hitsound.volume
HitSound.Parent = Env.Services.SoundService

function Env.spawnHitmark(worldPos, isHead)
    if not Env.cfg.hitmarker.enabled then return end
    local screenPos, onScreen = Camera:WorldToViewportPoint(worldPos)
    if not onScreen then return end

    local gui = Instance.new("ScreenGui", LocalPlayer.PlayerGui)
    local container = Instance.new("Frame", gui)
    container.Position = UDim2.new(0, screenPos.X, 0, screenPos.Y)
    container.BackgroundTransparency = 1

    local conf = Env.cfg.hitmarker
    local color = isHead and Color3.new(1, 0, 0) or Color3.new(1, 1, 1)
    local dist = isHead and conf.headExpand or conf.expand

    for i = 1, 4 do
        local angle = ({45, 135, 225, 315})[i]
        local line = Instance.new("Frame", container)
        line.BackgroundColor3 = color
        line.BorderSizePixel = 0
        line.Size = UDim2.new(0, conf.thickness, 0, conf.length)
        line.AnchorPoint = Vector2.new(0.5, 0)
        line.Rotation = angle
        
        local rad = math.rad(angle)
        line.Position = UDim2.new(0, math.sin(rad) * conf.gap, 0, -math.cos(rad) * conf.gap)
        
        Env.Services.TweenService:Create(line, TweenInfo.new(0.2), {
            Position = UDim2.new(0, math.sin(rad) * (conf.gap + dist), 0, -math.cos(rad) * (conf.gap + dist)),
            BackgroundTransparency = 1
        }):Play()
    end
    Env.Services.Debris:AddItem(gui, 0.25)
end

function Env.updateCrosshair()
    task.spawn(function()
        pcall(function()
            local gui = LocalPlayer.PlayerGui:FindFirstChild("CrosshairGui")
            if gui then
                local frame = gui:FindFirstChild("CrosshairFrame")
                if frame then
                    local dot = frame:FindFirstChild("Dot")
                    if dot and Env.cfg.crosshair.customImage ~= "" then
                        dot.Image = "rbxassetid://" .. Env.cfg.crosshair.customImage
                    end
                end
            end
        end)
    end)
end

function Env.getEquippedTool()
    local char = LocalPlayer.Character
    if not char then return nil end
    for _, v in ipairs(char:GetChildren()) do
        if v:IsA("Tool") then return v end
    end
    return nil
end

function Env.getMuzzlePosition()
    local tool = Env.getEquippedTool()
    if not tool then return nil end
    local muzzle = tool:FindFirstChild("Muzzle", true) or tool:FindFirstChild("MuzzleAttachment", true) or tool:FindFirstChild("Barrel", true)
    if muzzle then
        return muzzle:IsA("Attachment") and muzzle.WorldPosition or muzzle.Position
    end
    local handle = tool:FindFirstChild("Handle")
    return handle and handle.Position or nil
end

function Env.createTracer(fromPos, toPos)
    if not Env.cfg.tracer.enabled then return end
    local p0 = Instance.new("Part", workspace)
    p0.Anchored = true
    p0.CanCollide = false
    p0.Transparency = 1
    p0.Size = Vector3.new(0.1, 0.1, 0.1)
    p0.CFrame = CFrame.new(fromPos)
    
    local p1 = p0:Clone()
    p1.CFrame = CFrame.new(toPos)
    p1.Parent = workspace
    
    local a0 = Instance.new("Attachment", p0)
    local a1 = Instance.new("Attachment", p1)
    local beam = Instance.new("Beam", p0)
    beam.Attachment0 = a0
    beam.Attachment1 = a1
    beam.Color = ColorSequence.new(Env.cfg.tracer.color)
    beam.Width0 = Env.cfg.tracer.thickness
    beam.Width1 = Env.cfg.tracer.thickness
    beam.FaceCamera = true
    beam.LightEmission = 1
    
    task.delay(Env.cfg.tracer.lifetime, function()
        Env.Services.TweenService:Create(beam, TweenInfo.new(Env.cfg.tracer.fade), {Transparency = NumberSequence.new(1)}):Play()
    end)
    
    Env.Services.Debris:AddItem(p0, Env.cfg.tracer.lifetime + Env.cfg.tracer.fade + 0.1)
    Env.Services.Debris:AddItem(p1, Env.cfg.tracer.lifetime + Env.cfg.tracer.fade + 0.1)
end

local lastHitTime = 0
function Env.playHitsound(hitPart)
    if not Env.cfg.hitsound.enabled or not hitPart then return end
    local char = hitPart:FindFirstAncestorOfClass("Model")
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    local plr = char and Players:GetPlayerFromCharacter(char)
    
    if hum and plr and plr ~= LocalPlayer and hum.Health > 0 then
        local now = tick()
        if now - lastHitTime < 0.05 then return end
        lastHitTime = now
        
        local isHead = Env.isHeadshot(hitPart)
        HitSound.SoundId = "rbxassetid://" .. (isHead and Env.cfg.hitsound.headshot or Env.cfg.hitsound.normal)
        HitSound:Play()
        
        Env.spawnHitmark(hitPart.Position, isHead)
    end
end

local mt = getrawmetatable(game)
local old = mt.__namecall
setreadonly(mt, false)

mt.__namecall = newcclosure(function(self, ...)
    local args = {...}
    local method = getnamecallmethod()
    
    if method == "FireServer" and self.Name == "ShootEvent" and Env.cfg.enabled then
        task.spawn(function()
            local shots = args[1]
            if type(shots) ~= "table" then return end
            local muzzlePos = Env.getMuzzlePosition()
            if not muzzlePos then return end
            
            for i, shot in ipairs(shots) do
                if type(shot) == "table" and #shot >= 2 then
                    Env.createTracer(muzzlePos, shot[2])
                    if i == 1 and shot[3] then
                        Env.playHitsound(shot[3])
                    end
                end
            end
        end)
    end
    return old(self, ...)
end)

setreadonly(mt, true)
