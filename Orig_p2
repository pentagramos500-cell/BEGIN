local Env = getgenv().PL_Hub

if not Env then
    return warn('Load Part 1 First!')
end

local Players = Env.Services.Players
local LocalPlayer = Players.LocalPlayer
local RunService = Env.Services.RunService
local Debris = Env.Services.Debris
local ReplicatedStorage = Env.Services.ReplicatedStorage
local fovCircle = Drawing.new('Circle')

fovCircle.Color = Color3.fromRGB(180, 120, 220)
fovCircle.Radius = Env.cfg.fov
fovCircle.Transparency = 0.5
fovCircle.Filled = false
fovCircle.NumSides = 64
fovCircle.Thickness = 1
fovCircle.Visible = Env.cfg.showfov and Env.cfg.enabled

local targetLine = Drawing.new('Line')

targetLine.Color = Color3.fromRGB(0, 255, 0)
targetLine.Thickness = 1
targetLine.Transparency = 0.5
targetLine.Visible = false

local visuals = {container = nil}
local espCache = {}

local function makeVisuals()
    local container

    if gethui then
        local screen = Instance.new('ScreenGui')

        screen.Name = 'SilentAimESP'
        screen.ResetOnSpawn = false
        screen.Parent = gethui()
        container = screen
    elseif syn and syn.protect_gui then
        local screen = Instance.new('ScreenGui')

        screen.Name = 'SilentAimESP'
        screen.ResetOnSpawn = false

        syn.protect_gui(screen)

        screen.Parent = Env.Services.CoreGui
        container = screen
    else
        local screen = Instance.new('ScreenGui')

        screen.Name = 'SilentAimESP'
        screen.ResetOnSpawn = false
        screen.Parent = Env.Services.CoreGui
        container = screen
    end

    visuals.container = container
end
local function makeEsp(player)
    if espCache[player] then
        return espCache[player]
    end

    local esp = Instance.new('BillboardGui')

    esp.Name = 'ESP_' .. player.Name
    esp.AlwaysOnTop = true
    esp.Size = UDim2.new(0, 20, 0, 20)
    esp.StudsOffset = Vector3.new(0, 3, 0)
    esp.LightInfluence = 0

    local diamond = Instance.new('Frame')

    diamond.Name = 'Diamond'
    diamond.BackgroundColor3 = Env.cfg.espcolor
    diamond.BorderSizePixel = 0
    diamond.Size = UDim2.new(0, 10, 0, 10)
    diamond.Position = UDim2.new(0.5, -5, 0.5, -5)
    diamond.Rotation = 45
    diamond.Parent = esp

    local stroke = Instance.new('UIStroke')

    stroke.Color = Color3.new(0, 0, 0)
    stroke.Thickness = 1.5
    stroke.Transparency = 0.3
    stroke.Parent = diamond

    local distLabel = Instance.new('TextLabel')

    distLabel.Name = 'DistanceLabel'
    distLabel.BackgroundTransparency = 1
    distLabel.Size = UDim2.new(0, 60, 0, 16)
    distLabel.Position = UDim2.new(0.5, -30, 1, 2)
    distLabel.Font = Enum.Font.GothamBold
    distLabel.TextSize = 11
    distLabel.TextColor3 = Color3.new(1, 1, 1)
    distLabel.TextStrokeTransparency = 0.5
    distLabel.TextStrokeColor3 = Color3.new(0, 0, 0)
    distLabel.Text = ''
    distLabel.Parent = esp

    local nameLabel = Instance.new('TextLabel')

    nameLabel.Name = 'NameLabel'
    nameLabel.BackgroundTransparency = 1
    nameLabel.Size = UDim2.new(0, 100, 0, 14)
    nameLabel.Position = UDim2.new(0.5, -50, 0, -16)
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.TextSize = 10
    nameLabel.TextColor3 = Color3.new(1, 1, 1)
    nameLabel.TextStrokeTransparency = 0.5
    nameLabel.TextStrokeColor3 = Color3.new(0, 0, 0)
    nameLabel.Text = player.Name
    nameLabel.Parent = esp
    espCache[player] = esp

    return esp
end
local function removeEsp(player)
    local e = espCache[player]

    if e then
        e:Destroy()

        espCache[player] = nil
    end
end
local function shouldShowEsp(player)
    if not player or player == LocalPlayer or not player.Character then
        return false
    end

    local humanoid = player.Character:FindFirstChildOfClass('Humanoid')

    if not humanoid or humanoid.Health <= 0 then
        return false
    end

    local hrp = player.Character:FindFirstChild('HumanoidRootPart')

    if not hrp then
        return false
    end

    local myChar = LocalPlayer.Character

    if not myChar then
        return false
    end

    local myHrp = myChar:FindFirstChild('HumanoidRootPart')

    if not myHrp then
        return false
    end

    local distance = (hrp.Position - myHrp.Position).Magnitude

    if distance > Env.cfg.espmaxdist then
        return false
    end

    local myTeam = LocalPlayer.Team
    local theirTeam = player.Team

    if theirTeam == myTeam then
        if not Env.cfg.espshowteam then
            return false
        end

        return true
    end
    if Env.cfg.espteamcheck then
        local imCrimOrInmate = (myTeam == Env.Teams.Criminals or myTeam == Env.Teams.Inmates)
        local theyCrimOrInmate = (theirTeam == Env.Teams.Criminals or theirTeam == Env.Teams.Inmates)

        if imCrimOrInmate and theyCrimOrInmate then
            return false
        end
    end
    if theirTeam == Env.Teams.Guards then
        return Env.cfg.esptargets.guards
    elseif theirTeam == Env.Teams.Inmates then
        return Env.cfg.esptargets.inmates
    elseif theirTeam == Env.Teams.Criminals then
        return Env.cfg.esptargets.criminals
    end

    return false
end

Env.updateEsp = function()
    if not Env.cfg.esp or not visuals.container then
        for _, e in pairs(espCache)do
            e.Parent = nil
        end

        return
    end

    local myChar = LocalPlayer.Character
    local myHrp = myChar and myChar:FindFirstChild('HumanoidRootPart')

    for _, player in ipairs(Players:GetPlayers())do
        local show = shouldShowEsp(player)

        if show then
            local char = player.Character
            local hrp = char and char:FindFirstChild('HumanoidRootPart')
            local head = char and char:FindFirstChild('Head')

            if hrp and head then
                local esp = makeEsp(player)

                esp.Adornee = head
                esp.Parent = visuals.container

                local d = esp:FindFirstChild('Diamond')

                if d and Env.cfg.espuseteamcolors then
                    local t = player.Team

                    if t == LocalPlayer.Team then
                        d.BackgroundColor3 = Env.cfg.espteam
                    elseif t == Env.Teams.Guards then
                        d.BackgroundColor3 = Env.cfg.espguards
                    elseif t == Env.Teams.Inmates then
                        d.BackgroundColor3 = Env.cfg.espinmates
                    elseif t == Env.Teams.Criminals then
                        d.BackgroundColor3 = Env.cfg.espcriminals
                    else
                        d.BackgroundColor3 = Env.cfg.espcolor
                    end
                end
                if Env.cfg.espshowdist and myHrp then
                    local label = esp:FindFirstChild('DistanceLabel')

                    if label then
                        label.Text = math.floor((hrp.Position - myHrp.Position).Magnitude) .. 'm'
                        label.Visible = true
                    end
                end
            end
        else
            local e = espCache[player]

            if e then
                e.Parent = nil
            end
        end
    end
end

local c4espCache = {}

local function makeC4Esp(c4Part)
    if c4espCache[c4Part] then
        return c4espCache[c4Part]
    end

    local esp = Instance.new('BillboardGui')

    esp.Name = 'C4ESP_' .. tostring(c4Part)
    esp.AlwaysOnTop = true
    esp.Size = UDim2.new(0, 24, 0, 24)
    esp.StudsOffset = Vector3.new(0, 1, 0)

    local icon = Instance.new('Frame')

    icon.BackgroundColor3 = Env.cfg.c4espcolor
    icon.Size = UDim2.new(0, 14, 0, 14)
    icon.Position = UDim2.new(0.5, -7, 0.5, -7)
    icon.Rotation = 45
    icon.Parent = esp

    local label = Instance.new('TextLabel')

    label.BackgroundTransparency = 1
    label.Size = UDim2.new(0, 60, 0, 14)
    label.Position = UDim2.new(0.5, -30, 1, 2)
    label.Font = Enum.Font.GothamBold
    label.TextSize = 11
    label.TextColor3 = Color3.new(1, 1, 1)
    label.Text = 'C4'
    label.Parent = esp

    local distLabel = Instance.new('TextLabel')

    distLabel.Name = 'DistLabel'
    distLabel.BackgroundTransparency = 1
    distLabel.Size = UDim2.new(0, 60, 0, 12)
    distLabel.Position = UDim2.new(0.5, -30, 1, 16)
    distLabel.Font = Enum.Font.GothamBold
    distLabel.TextSize = 10
    distLabel.TextColor3 = Env.cfg.c4espcolor
    distLabel.Text = ''
    distLabel.Parent = esp
    c4espCache[c4Part] = esp

    return esp
end

local trackedC4s = {}

local function isC4Part(part)
    if not part or not part:IsA('BasePart') then
        return false
    end

    local name = part.Name:lower()
    local parentName = part.Parent and part.Parent.Name:lower() or ''

    return name == 'explosive' or name == 'c4' or name == 'clientc4' or parentName:find('c4') or name:find('c4')
end
local function onDescendantAdded(desc)
    if isC4Part(desc) then
        trackedC4s[desc] = true
    end
end
local function onDescendantRemoving(desc)
    trackedC4s[desc] = nil

    if c4espCache[desc] then
        c4espCache[desc]:Destroy()

        c4espCache[desc] = nil
    end
end

for _, desc in ipairs(workspace:GetDescendants())do
    if isC4Part(desc) then
        trackedC4s[desc] = true
    end
end

workspace.DescendantAdded:Connect(onDescendantAdded)
workspace.DescendantRemoving:Connect(onDescendantRemoving)

Env.updateC4Esp = function()
    if not Env.cfg.c4esp or not visuals.container then
        for _, e in pairs(c4espCache)do
            e.Parent = nil
        end

        return
    end

    local myChar = LocalPlayer.Character
    local myHrp = myChar and myChar:FindFirstChild('HumanoidRootPart')

    for part in pairs(trackedC4s)do
        if part and part:IsDescendantOf(workspace) then
            local dist = myHrp and (part.Position - myHrp.Position).Magnitude or 0

            if dist <= Env.cfg.c4espmaxdist then
                local esp = makeC4Esp(part)

                esp.Adornee = part
                esp.Parent = visuals.container

                if Env.cfg.c4espshowdist and myHrp then
                    local distLabel = esp:FindFirstChild('DistLabel')

                    if distLabel then
                        distLabel.Text = math.floor(dist) .. 'm'
                    end
                end
            else
                if c4espCache[part] then
                    c4espCache[part].Parent = nil
                end
            end
        else
            trackedC4s[part] = nil

            if c4espCache[part] then
                c4espCache[part]:Destroy()

                c4espCache[part] = nil
            end
        end
    end
end

makeVisuals()

local ShootEvent = ReplicatedStorage:WaitForChild('GunRemotes'):WaitForChild('ShootEvent')
local lastAutoShoot = 0
local targetAcquiredTime = 0
local lastAutoTarget = nil
local cachedBulletsLabel = nil

local function createBulletTrail(startPos, endPos, isTaser)
    local distance = (endPos - startPos).Magnitude
    local trail = Instance.new('Part')

    trail.Name = 'BulletTrail'
    trail.Anchored = true
    trail.CanCollide = false
    trail.Material = Enum.Material.Neon
    trail.Size = Vector3.new(0.1, 0.1, distance)
    trail.CFrame = CFrame.new(startPos, endPos) * CFrame.new(0, 0, -distance / 2)
    trail.Transparency = 0.5

    if isTaser then
        trail.BrickColor = BrickColor.new('Cyan')
        trail.Size = Vector3.new(0.2, 0.2, distance)
    else
        trail.BrickColor = BrickColor.Yellow()
    end

    trail.Parent = workspace

    Debris:AddItem(trail, isTaser and 0.8 or 0.1)
end
local function autoShoot()
    if not Env.cfg.autoshoot or not Env.cfg.enabled or not Env.vars.currentGun then
        return
    end
    if Env.vars.currentGun:GetAttribute('Local_IsShooting') then
        return
    end
    if (Env.vars.currentGun:GetAttribute('Local_ReloadSession') or 0) > 0 then
        return
    end

    local now = os.clock()
    local fireRate = Env.vars.currentGun:GetAttribute('FireRate') or Env.cfg.autoshootdelay

    if now - lastAutoShoot < fireRate then
        return
    end

    local myChar = LocalPlayer.Character

    if not myChar then
        return
    end

    local myHead = myChar:FindFirstChild('Head')

    if not myHead then
        return
    end

    local muzzle = Env.vars.currentGun:FindFirstChild('Muzzle')
    local startPos = muzzle and muzzle.Position or myHead.Position
    local target, targetPos = Env.getClosest(Env.cfg.fov)

    if not target or not Env.fullCheck(target) then
        lastAutoTarget = nil

        return
    end
    if target ~= lastAutoTarget then
        targetAcquiredTime = now
        lastAutoTarget = target
    end
    if now - targetAcquiredTime < Env.cfg.autoshootstartdelay then
        return
    end

    local targetPart = Env.getTargetPart(target.Character)

    if not targetPart then
        return
    end

    local ammo = Env.vars.currentGun:GetAttribute('Local_CurrentAmmo') or Env.vars.currentGun:GetAttribute('CurrentAmmo') or 0

    if ammo <= 0 then
        return
    end

    lastAutoShoot = now

    local isTaser = Env.vars.currentGun:GetAttribute('Projectile') == 'Taser'
    local isShotgun = Env.vars.currentGun:GetAttribute('IsShotgun')
    local shouldHit = false

    if Env.cfg.taseralwayshit and isTaser then
        shouldHit = true
    elseif Env.cfg.ifplayerstill and Env.isStanding(target) then
        shouldHit = true
    elseif Env.cfg.hitchanceAutoOnly and isShotgun then
        shouldHit = true
    else
        shouldHit = Env.rollHit()
    end

    local projectileCount = Env.vars.currentGun:GetAttribute('ProjectileCount') or 1
    local shots = {}

    for i = 1, projectileCount do
        local finalPos

        if shouldHit then
            finalPos = targetPart.Position
        else
            if Env.cfg.missspread > 0 then
                finalPos = Env.getMissPos(targetPart.Position)
            else
                return
            end
        end

        shots[i] = {
            myHead.Position,
            finalPos,
            shouldHit and targetPart or nil,
        }

        createBulletTrail(startPos, finalPos, isTaser)
    end

    ShootEvent:FireServer(shots)

    local newAmmo = ammo - 1

    Env.vars.currentGun:SetAttribute('Local_CurrentAmmo', newAmmo)

    if not cachedBulletsLabel then
        local playerGui = LocalPlayer:FindFirstChild('PlayerGui')

        if playerGui then
            local home = playerGui:FindFirstChild('Home')

            if home and home:FindFirstChild('hud') then
                local br = home.hud:FindFirstChild('BottomRightFrame')

                if br and br:FindFirstChild('GunFrame') then
                    cachedBulletsLabel = br.GunFrame:FindFirstChild('BulletsLabel')
                end
            end
        end
    end
    if cachedBulletsLabel then
        cachedBulletsLabel.Text = newAmmo .. '/' .. (Env.vars.currentGun:GetAttribute('MaxAmmo') or 30)
    end

    local handle = Env.vars.currentGun:FindFirstChild('Handle')

    if handle and handle:FindFirstChild('ShootSound') then
        local sound = handle.ShootSound:Clone()

        sound.Parent = handle

        sound:Play()
        Debris:AddItem(sound, 2)
    end
end
local function getGun()
    local char = LocalPlayer.Character

    if not char then
        return nil
    end

    for _, tool in ipairs(char:GetChildren())do
        if tool:IsA('Tool') and tool:GetAttribute('ToolType') == 'Gun' then
            return tool
        end
    end

    return nil
end

local lastGun = nil

RunService.Heartbeat:Connect(function()
    Env.vars.currentGun = getGun()

    if Env.vars.currentGun ~= lastGun then
        lastAutoShoot = 0
        lastGun = Env.vars.currentGun
    end

    autoShoot()
end)
RunService.PreRender:Connect(function()
    local aimPos = Env.Services.UserInputService:GetMouseLocation()
    local camera = workspace.CurrentCamera

    if camera then
        local lastInput = Env.Services.UserInputService:GetLastInputType()
        local locked = (lastInput == Enum.UserInputType.Touch) or (Env.Services.UserInputService.MouseBehavior == Enum.MouseBehavior.LockCenter)

        if locked then
            local viewportSize = camera.ViewportSize

            aimPos = Vector2.new(viewportSize.X / 2, viewportSize.Y / 2)
        end
    end

    fovCircle.Position = aimPos
    fovCircle.Radius = Env.cfg.fov
    fovCircle.Visible = Env.cfg.showfov and Env.cfg.enabled

    if Env.cfg.showtargetline and Env.cfg.enabled then
        local target, targetPos = Env.getClosest()

        if target and targetPos and camera then
            local screenPos, onScreen = camera:WorldToViewportPoint(targetPos)

            if onScreen then
                targetLine.From = aimPos
                targetLine.To = Vector2.new(screenPos.X, screenPos.Y)
                targetLine.Visible = true
            else
                targetLine.Visible = false
            end
        else
            targetLine.Visible = false
        end
    else
        targetLine.Visible = false
    end

    Env.updateEsp()
    Env.updateC4Esp()
end)
Players.PlayerRemoving:Connect(removeEsp)
LocalPlayer:GetPropertyChangedSignal('Team'):Connect(function()
    for player, e in pairs(espCache)do
        if e then
            e:Destroy()
        end

        espCache[player] = nil
    end
end)

local origCastRay
local hooked = false

local function setupHook()
    local castRayFunc = filtergc('function', {
        Name = 'castRay',
    }, true)

    if not castRayFunc then
        return false
    end

    origCastRay = hookfunction(castRayFunc, function(startPos, targetPos, ...)
        if not Env.cfg.enabled then
            return origCastRay(startPos, targetPos, ...)
        end

        local closest, closestPos = Env.getClosest(Env.cfg.fov)

        if closest and closest.Character then
            local isTaser = Env.vars.currentGun and Env.vars.currentGun:GetAttribute('Projectile') == 'Taser'
            local isShotgun = Env.vars.currentGun and Env.vars.currentGun:GetAttribute('IsShotgun')
            local shouldHit = false

            if Env.cfg.hitchanceAutoOnly and isShotgun then
                return origCastRay(startPos, targetPos, ...)
            end
            if Env.cfg.shotgungamehandled and isShotgun then
                local targetPart = Env.getTargetPart(closest.Character)

                if targetPart then
                    return origCastRay(startPos, targetPart.Position, ...)
                end

                return origCastRay(startPos, targetPos, ...)
            end
            if Env.cfg.taseralwayshit and isTaser then
                shouldHit = true
            elseif Env.cfg.ifplayerstill and Env.isStanding(closest) then
                shouldHit = true
            else
                shouldHit = Env.rollHit()
            end
            if shouldHit then
                local targetPart = Env.getTargetPart(closest.Character)

                if targetPart then
                    if Env.cfg.shotgunnaturalspread and isShotgun then
                        return origCastRay(startPos, targetPart.Position, ...)
                    end

                    return targetPart, targetPart.Position
                end
            else
                if Env.cfg.missspread > 0 then
                    local targetPart = Env.getTargetPart(closest.Character)

                    if targetPart then
                        local missPos = Env.getMissPos(targetPart.Position)

                        return origCastRay(startPos, missPos, ...)
                    end
                end

                return origCastRay(startPos, targetPos, ...)
            end
        end

        return origCastRay(startPos, targetPos, ...)
    end)

    return true
end

if not setupHook() then
    task.spawn(function()
        while not hooked do
            task.wait(0.5)

            if setupHook() then
                hooked = true
            end
        end
    end)
else
    hooked = true
end
