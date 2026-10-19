-- Ultra Battlegrounds: Авто-Whiplash + Кастомные анимации + Авто-Инвулн + ОЧЕНЬ сильный удар (как в старом)
local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local LP = Players.LocalPlayer

local Unloaded = false
local gui = nil
local AUTO_WHIPLASH = true -- Вкл/выкл авто-финишер

-- ============================================================
-- ===== НАСТРОЙКА ID АНИМАЦИЙ =====
-- ============================================================
local CUSTOM_ANIMS = {
    SPAWN = "rbxassetid://107150053573004",
    HIT_1 = "rbxassetid://122122931121226",
    HIT_2 = "rbxassetid://74084641430913",
    HIT_3 = "rbxassetid://97472554025018",
    HIT_4 = "rbxassetid://115392387511790",
    DOWN_SLAM = "rbxassetid://136152119620299",
    ULT = "rbxassetid://80058342966317",
    STRONG_UPPERCUT = "rbxassetid://137327036857058",
    VERY_STRONG = "rbxassetid://74023776559916", -- Старая анимация сильного удара
}

-- ============================================================
-- ===== СИСТЕМА ОТСЛЕЖИВАНИЯ УБИЙСТВ (AUTO-WHIPLASH) =====
-- ============================================================
local recentHits = {}

local function trackHit(char)
    if not char then return end
    recentHits[char] = tick()
end

local function triggerWhiplashFinisher(deadChar)
    if not AUTO_WHIPLASH then return end
    local c = LP.Character
    if not c then return end
    
    local backpack = LP:FindFirstChild("Backpack")
    local whiplash = (backpack and backpack:FindFirstChild("Whiplash")) or c:FindFirstChild("Whiplash")
    if not whiplash then return end
    
    local rs = whiplash:FindFirstChild("ReplicateScript")
    local ev = rs and rs:FindFirstChild("ReplicateEvent")
    if not ev then return end
    
    ev:FireServer(unpack({
        c,
        "V1",
        {
            shootHook = false,
            doFinisher = true,
            playersHit = { deadChar }
        }
    }))
    print("Авто-Whiplash сработал на " .. (deadChar.Name or "Игроке"))
end

local function setupDeathListener(char)
    local hum = char:WaitForChild("Humanoid", 5)
    if not hum then return end
    hum.Died:Connect(function()
        if recentHits[char] and tick() - recentHits[char] <= 5 then
            triggerWhiplashFinisher(char)
        end
        recentHits[char] = nil
    end)
end

for _, plr in ipairs(Players:GetPlayers()) do
    if plr ~= LP then
        if plr.Character then setupDeathListener(plr.Character) end
        plr.CharacterAdded:Connect(setupDeathListener)
    end
end
Players.PlayerAdded:Connect(function(plr)
    if plr ~= LP then
        plr.CharacterAdded:Connect(setupDeathListener)
        if plr.Character then setupDeathListener(plr.Character) end
    end
end)

task.spawn(function()
    while task.wait(5) do
        local now = tick()
        for char, time in pairs(recentHits) do
            if now - time > 5 then recentHits[char] = nil end
        end
    end
end)

-- ============================================================
-- ===== АВТО-ИНВУЛН =====
-- ============================================================
local function triggerInvuln()
    local c = LP.Character
    if not c then return end
    local backpack = LP:FindFirstChild("Backpack")
    if not backpack then return end
    local parry = backpack:FindFirstChild("Parry")
    if not parry then return end
    local rs = parry:FindFirstChild("ReplicateScript")
    if not rs then return end
    local ev = rs:FindFirstChild("ReplicateEvent")
    if not ev then return end
    ev:FireServer(unpack({ c, "V1", "invuln" }))
end

-- ============================================================
-- ===== ФУНКЦИЯ АНИМАЦИЙ =====
-- ============================================================
local function playCustomAnim(animId, priority, looped)
    local c = LP.Character
    if not c then return end
    local hum = c:FindFirstChildOfClass("Humanoid")
    if not hum then return end
    local animator = hum:FindFirstChildOfClass("Animator")
    if not animator then
        animator = Instance.new("Animator")
        animator.Parent = hum
    end
    local anim = Instance.new("Animation")
    anim.AnimationId = animId
    local ok, track = pcall(function() return animator:LoadAnimation(anim) end)
    if ok and track then
        track.Priority = priority or Enum.AnimationPriority.Action
        track.Looped = looped or false
        track:Play()
        return track
    end
end

-- ============================================================
-- ===== АНИМАЦИЯ СПАВНА =====
-- ============================================================
local function playSpawnAnimation(char)
    local hum = char:WaitForChild("Humanoid", 10)
    if not hum then return end
    local animator = hum:WaitForChild("Animator", 10)
    if not animator then return end
    task.wait(0.5)
    local anim = Instance.new("Animation")
    anim.AnimationId = CUSTOM_ANIMS.SPAWN
    local ok, track = pcall(function() return animator:LoadAnimation(anim) end)
    if ok and track then
        track.Priority = Enum.AnimationPriority.Action
        track.Looped = false
        track:Play()
    end
end

LP.CharacterAdded:Connect(function(char)
    task.spawn(function() playSpawnAnimation(char) end)
end)
if LP.Character then
    task.spawn(function() playSpawnAnimation(LP.Character) end)
end

-- ============================================================
-- ===== ИНТЕРФЕЙС =====
-- ============================================================
local function createInterface()
    if gui then return end

    gui = Instance.new("ScreenGui")
    gui.Name = "CombatGui_CustomAnims"
    gui.ResetOnSpawn = false
    gui.DisplayOrder = 2147483647
    gui.IgnoreGuiInset = true
    gui.Parent = LP:WaitForChild("PlayerGui")

    local function createButton(image, x, y, color)
        local SIZE = 65
        local btn = Instance.new("ImageButton", gui)
        btn.Size = UDim2.new(0, SIZE, 0, SIZE)
        btn.Position = UDim2.new(0, x, 1, y) 
        btn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
        btn.BackgroundTransparency = 0.5
        btn.Image = image
        btn.ImageColor3 = color or Color3.fromRGB(255, 255, 255)
        btn.ImageTransparency = 0.3
        btn.BorderSizePixel = 0
        btn.AutoButtonColor = false
        btn.ZIndex = 1000
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
        local stroke = Instance.new("UIStroke", btn)
        stroke.Color = Color3.fromRGB(255, 255, 255)
        stroke.Thickness = 2
        stroke.Transparency = 0.6
        return btn
    end

    local btnHit = createButton("rbxassetid://100890221092639", 200, -120)
    local btnKick = createButton("rbxassetid://100890221092639", 275, -120, Color3.fromRGB(255, 150, 150))
    local btnUlt = createButton("rbxassetid://100890221092639", 352, -120) 
    local btnUppercut = createButton("rbxassetid://100890221092639", 426, -120, Color3.fromRGB(255, 200, 100))
    local btnStrong = createButton("rbxassetid://100890221092639", 500, -120, Color3.fromRGB(255, 80, 80))
    
    -- НОВАЯ КНОПКА: ОЧЕНЬ СИЛЬНЫЙ УДАР (Над обычным сильным)
    local btnVeryStrong = createButton("rbxassetid://100890221092639", 500, -200, Color3.fromRGB(255, 0, 0)) 
    
    local btnBlock = createButton("rbxassetid://136884788250361", 627, -280, Color3.fromRGB(150, 255, 150)) 
    local btnMine = createButton("rbxassetid://100890221092639", 627, -200, Color3.fromRGB(255, 100, 255))

    btnUlt.ImageColor3 = Color3.fromRGB(100, 100, 100) 

    -- ============================================================
    -- ===== КОМБО УДАРОВ =====
    -- ============================================================
    local HIT_COMBO = { CUSTOM_ANIMS.HIT_1, CUSTOM_ANIMS.HIT_2, CUSTOM_ANIMS.HIT_3, CUSTOM_ANIMS.HIT_4 }
    local currentHit = 1
    local hitCooldown = false

    btnHit.MouseButton1Click:Connect(function()
        if hitCooldown then return end
        local c = LP.Character
        if not c then return end
        local hum = c:FindFirstChildOfClass("Humanoid")
        local hrp = c:FindFirstChild("HumanoidRootPart")
        if not hum or not hrp then return end

        triggerInvuln()
        hitCooldown = true
        btnHit.ImageTransparency = 0.8
        local isFinisher = (currentHit == 4)
        playCustomAnim(HIT_COMBO[currentHit], Enum.AnimationPriority.Action, false)

        task.wait(0.1)
        local ctrl = c:FindFirstChild("PlayerControllerScript")
        if ctrl then
            local remote = ctrl:FindFirstChild("PlayerInputEvent")
            if remote then
                local myPos = hrp.Position
                local lookDir = hrp.CFrame.LookVector
                local targets = {}
                for _, plr in ipairs(Players:GetPlayers()) do
                    if plr ~= LP and plr.Character then
                        local tHrp = plr.Character:FindFirstChild("HumanoidRootPart")
                        local tHum = plr.Character:FindFirstChildOfClass("Humanoid")
                        if tHrp and tHum and tHum.Health > 0 then
                            local diff = tHrp.Position - (myPos + lookDir * 3)
                            if math.abs(diff.X) <= 2.5 and math.abs(diff.Y) <= 3 and math.abs(diff.Z) <= 2.5 then
                                table.insert(targets, plr.Character)
                            end
                        end
                    end
                end
                if #targets > 0 then
                    remote:FireServer("dealDamage", true, {
                        hitstuns = not isFinisher, blockable = false, ultGain = 1, parryable = true,
                        damage = isFinisher and 35 or 20, knockbackForce = isFinisher and 150 or 20,
                        hitsRagdolled = isFinisher, destruction = false, 
                        axesForce = isFinisher and { Y = 1/0, X = 1/0, Z = 1/0 } or { Y = 0, X = 0, Z = 0 },
                        hitboxSize = vector.create(5, 6, 5), ragdolls = isFinisher,
                        playersHit = targets, knockbackDirection = { lookDir }
                    })
                    for _, t in ipairs(targets) do trackHit(t) end
                end
            end
        end
        currentHit = currentHit + 1
        if currentHit > 4 then currentHit = 1 end
        task.delay(0.5, function() hitCooldown = false; btnHit.ImageTransparency = 0.3 end)
    end)

    -- ============================================================
    -- ===== 2. УДАР НОГОЙ =====
    -- ============================================================
    local kickCooldown = false

    btnKick.MouseButton1Click:Connect(function()
        if kickCooldown then return end
        local c = LP.Character
        if not c then return end
        local hum = c:FindFirstChildOfClass("Humanoid")
        local hrp = c:FindFirstChild("HumanoidRootPart")
        if not hum or not hrp then return end

        triggerInvuln()
        kickCooldown = true
        btnKick.ImageTransparency = 0.8
        playCustomAnim(CUSTOM_ANIMS.DOWN_SLAM, Enum.AnimationPriority.Action, false)

        task.wait(0.2)
        local ctrl = c:FindFirstChild("PlayerControllerScript")
        if ctrl then
            local remote = ctrl:FindFirstChild("PlayerInputEvent")
            if remote then
                local myPos = hrp.Position
                remote:FireServer("doVFX", true, {
                    maxTilt = 10, radius = 6, hasCollision = false, startPos = hrp.CFrame,
                    maxRocks = 15, minRocks = 8, vfxType = "crater",
                    sizeTable = { maxY = 4, minY = 1, minX = 1, maxX = 3, minZ = 1, maxZ = 3 }
                })
                local hitData = {}
                for _, plr in ipairs(Players:GetPlayers()) do
                    if plr ~= LP and plr.Character then
                        local tHrp = plr.Character:FindFirstChild("HumanoidRootPart")
                        local tHum = plr.Character:FindFirstChildOfClass("Humanoid")
                        if tHrp and tHum and tHum.Health > 0 and (tHrp.Position - myPos).Magnitude <= 6 then
                            local dir = (tHrp.Position - myPos).Unit
                            table.insert(hitData, {char = plr.Character, dir = dir})
                        end
                    end
                end
                if #hitData > 0 then
                    for _, data in ipairs(hitData) do
                        remote:FireServer("dealDamage", true, {
                            hitstuns = true, blockable = false, ultGain = 1, parryable = true,
                            damage = 30, knockbackForce = 100, hitsRagdolled = true,
                            destruction = false, axesForce = { Y = 1/0, X = 1/0, Z = 1/0 },
                            hitboxSize = vector.create(12, 12, 12), ragdolls = true,
                            playersHit = { data.char }, knockbackDirection = { data.dir }
                        })
                        trackHit(data.char)
                    end
                end
            end
        end
        task.delay(3, function() kickCooldown = false; btnKick.ImageTransparency = 0.3 end)
    end)

    -- ============================================================
    -- ===== 3. УЛЬТА =====
    -- ============================================================
    local ultCooldown = false

    btnUlt.MouseButton1Click:Connect(function()
        if ultCooldown then return end
        local c = LP.Character
        if not c then return end
        local hum = c:FindFirstChildOfClass("Humanoid")
        local hrp = c:FindFirstChild("HumanoidRootPart")
        if not hum or not hrp then return end

        triggerInvuln()
        ultCooldown = true
        btnUlt.ImageColor3 = Color3.fromRGB(100, 100, 100)
        playCustomAnim(CUSTOM_ANIMS.ULT, Enum.AnimationPriority.Action, false)

        task.wait(0.3)
        local ctrl = c:FindFirstChild("PlayerControllerScript")
        if ctrl then
            local remote = ctrl:FindFirstChild("PlayerInputEvent")
            if remote then
                local myPos = hrp.Position
                remote:FireServer("doVFX", true, {
                    maxTilt = 10, radius = 20, hasCollision = false, startPos = hrp.CFrame,
                    maxRocks = 40, minRocks = 20, vfxType = "crater",
                    sizeTable = { maxY = 10, minY = 2, minX = 2, maxX = 5, minZ = 2, maxZ = 5 }
                })
                local hitData = {}
                for _, plr in ipairs(Players:GetPlayers()) do
                    if plr ~= LP and plr.Character then
                        local tHrp = plr.Character:FindFirstChild("HumanoidRootPart")
                        local tHum = plr.Character:FindFirstChildOfClass("Humanoid")
                        if tHrp and tHum and tHum.Health > 0 and (tHrp.Position - myPos).Magnitude <= 20 then
                            local dir = (tHrp.Position - myPos).Unit
                            table.insert(hitData, {char = plr.Character, dir = dir})
                        end
                    end
                end
                for _, data in ipairs(hitData) do
                    remote:FireServer("dealDamage", true, {
                        hitstuns = true, blockable = false, ultGain = -50, parryable = false,
                        damage = 100, knockbackForce = 200, hitsRagdolled = true,
                        destruction = false, axesForce = { Y = 1/0, X = 1/0, Z = 1/0 },
                        hitboxSize = vector.create(20, 20, 20), ragdolls = true,
                        playersHit = { data.char }, knockbackDirection = { data.dir }
                    })
                    trackHit(data.char)
                end
            end
        end
        task.delay(15, function()
            ultCooldown = false
            if btnUlt and btnUlt.Parent then btnUlt.ImageColor3 = Color3.fromRGB(255, 255, 255) end
        end)
    end)

    -- ============================================================
    -- ===== 4. АППЕРКОТ =====
    -- ============================================================
    local uppercutCooldown = false

    btnUppercut.MouseButton1Click:Connect(function()
        if uppercutCooldown then return end
        local c = LP.Character
        if not c then return end
        local hum = c:FindFirstChildOfClass("Humanoid")
        local hrp = c:FindFirstChild("HumanoidRootPart")
        if not hum or not hrp then return end

        triggerInvuln()
        uppercutCooldown = true
        btnUppercut.ImageTransparency = 0.8
        playCustomAnim(CUSTOM_ANIMS.STRONG_UPPERCUT, Enum.AnimationPriority.Action, false)

        task.wait(0.15)
        local ctrl = c:FindFirstChild("PlayerControllerScript")
        if ctrl then
            local remote = ctrl:FindFirstChild("PlayerInputEvent")
            if remote then
                local myPos = hrp.Position
                local lookDir = hrp.CFrame.LookVector
                local targets = {}
                for _, plr in ipairs(Players:GetPlayers()) do
                    if plr ~= LP and plr.Character then
                        local tHrp = plr.Character:FindFirstChild("HumanoidRootPart")
                        local tHum = plr.Character:FindFirstChildOfClass("Humanoid")
                        if tHrp and tHum and tHum.Health > 0 then
                            local diff = tHrp.Position - (myPos + lookDir * 3)
                            if math.abs(diff.X) <= 3 and math.abs(diff.Y) <= 5 and math.abs(diff.Z) <= 3 then
                                table.insert(targets, plr.Character)
                            end
                        end
                    end
                end
                if #targets > 0 then
                    remote:FireServer("dealDamage", true, {
                        hitstuns = true, blockable = false, ultGain = 1, parryable = true,
                        damage = 40, knockbackForce = 120, hitsRagdolled = true,
                        destruction = false, axesForce = { Y = 1/0, X = 0, Z = 0 },
                        hitboxSize = vector.create(6, 8, 6), ragdolls = true,
                        playersHit = targets, knockbackDirection = { Vector3.new(0, 1, 0) }
                    })
                    for _, t in ipairs(targets) do trackHit(t) end
                end
            end
        end
        task.delay(4, function() uppercutCooldown = false; btnUppercut.ImageTransparency = 0.3 end)
    end)

    -- ============================================================
    -- ===== 5. СИЛЬНЫЙ УДАР =====
    -- ============================================================
    local strongCooldown = false

    btnStrong.MouseButton1Click:Connect(function()
        if strongCooldown then return end
        local c = LP.Character
        if not c then return end
        local hum = c:FindFirstChildOfClass("Humanoid")
        local hrp = c:FindFirstChild("HumanoidRootPart")
        if not hum or not hrp then return end

        triggerInvuln()
        strongCooldown = true
        btnStrong.ImageTransparency = 0.8
        playCustomAnim(CUSTOM_ANIMS.HIT_4, Enum.AnimationPriority.Action, false)

        task.wait(0.15)
        local ctrl = c:FindFirstChild("PlayerControllerScript")
        if ctrl then
            local remote = ctrl:FindFirstChild("PlayerInputEvent")
            if remote then
                local myPos = hrp.Position
                local lookDir = hrp.CFrame.LookVector
                local targets = {}
                for _, plr in ipairs(Players:GetPlayers()) do
                    if plr ~= LP and plr.Character then
                        local tHrp = plr.Character:FindFirstChild("HumanoidRootPart")
                        local tHum = plr.Character:FindFirstChildOfClass("Humanoid")
                        if tHrp and tHum and tHum.Health > 0 then
                            local diff = tHrp.Position - (myPos + lookDir * 3)
                            if math.abs(diff.X) <= 3 and math.abs(diff.Y) <= 3 and math.abs(diff.Z) <= 3 then
                                table.insert(targets, plr.Character)
                            end
                        end
                    end
                end
                if #targets > 0 then
                    remote:FireServer("dealDamage", true, {
                        hitstuns = true, blockable = false, ultGain = 1, parryable = true,
                        damage = 50, knockbackForce = 200, hitsRagdolled = true,
                        destruction = false, axesForce = { Y = 1/0, X = 1/0, Z = 1/0 },
                        hitboxSize = vector.create(6, 6, 6), ragdolls = true,
                        playersHit = targets, knockbackDirection = { lookDir }
                    })
                    for _, t in ipairs(targets) do trackHit(t) end
                end
            end
        end
        task.delay(5, function() strongCooldown = false; btnStrong.ImageTransparency = 0.3 end)
    end)

    -- ============================================================
    -- ===== 5.5. ОЧЕНЬ СИЛЬНЫЙ УДАР (КАК В СТАРОМ) =====
    -- ============================================================
    local veryStrongCooldown = false

    btnVeryStrong.MouseButton1Click:Connect(function()
        if veryStrongCooldown then return end
        local c = LP.Character
        if not c then return end
        local hum = c:FindFirstChildOfClass("Humanoid")
        local hrp = c:FindFirstChild("HumanoidRootPart")
        if not hum or not hrp then return end

        triggerInvuln()
        veryStrongCooldown = true
        btnVeryStrong.ImageTransparency = 0.8
        
        -- Используем старую анимацию
        playCustomAnim(CUSTOM_ANIMS.VERY_STRONG, Enum.AnimationPriority.Action, false)

        task.wait(0.15)
        local ctrl = c:FindFirstChild("PlayerControllerScript")
        if ctrl then
            local remote = ctrl:FindFirstChild("PlayerInputEvent")
            if remote then
                local myPos = hrp.Position
                local lookDir = hrp.CFrame.LookVector
                local targets = {}
                for _, plr in ipairs(Players:GetPlayers()) do
                    if plr ~= LP and plr.Character then
                        local tHrp = plr.Character:FindFirstChild("HumanoidRootPart")
                        local tHum = plr.Character:FindFirstChildOfClass("Humanoid")
                        if tHrp and tHum and tHum.Health > 0 then
                            local diff = tHrp.Position - (myPos + lookDir * 3)
                            if math.abs(diff.X) <= 3 and math.abs(diff.Y) <= 3 and math.abs(diff.Z) <= 3 then
                                table.insert(targets, plr.Character)
                            end
                        end
                    end
                end
                if #targets > 0 then
                    remote:FireServer("dealDamage", true, {
                        hitstuns = true, blockable = false, ultGain = 1, parryable = true,
                        damage = 200, -- ОГРОМНЫЙ УРОН
                        knockbackForce = 200, -- Как в старом
                        hitsRagdolled = true,
                        destruction = false, 
                        axesForce = { Y = 1/0, X = 1/0, Z = 1/0 }, -- Как в старом
                        hitboxSize = vector.create(6, 6, 6), -- Как в старом
                        ragdolls = true,
                        playersHit = targets, 
                        knockbackDirection = { lookDir }
                    })
                    for _, t in ipairs(targets) do trackHit(t) end
                end
            end
        end
        task.delay(5, function() veryStrongCooldown = false; btnVeryStrong.ImageTransparency = 0.3 end)
    end)

    -- ============================================================
    -- ===== 6. БЛОК (ХИЛКА) =====
    -- ============================================================
    local healCooldown = false

    btnBlock.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
            if healCooldown then return end
            local c = LP.Character
            if c then
                triggerInvuln()
                local ctrl = c:FindFirstChild("PlayerControllerScript")
                if ctrl then
                    local remote = ctrl:FindFirstChild("PlayerInputEvent")
                    if remote then
                        healCooldown = true
                        btnBlock.ImageTransparency = 0.8
                        btnBlock.ImageColor3 = Color3.fromRGB(100, 100, 100) 
                        remote:FireServer("dealDamage", true, {
                            hitstuns = false, blockable = false, ultGain = 0, parryable = false,
                            damage = -50, knockbackForce = 0, hitsRagdolled = false, destruction = false, 
                            axesForce = { Y = 0, X = 0, Z = 0 },
                            hitboxSize = vector.create(5, 5, 5), ragdolls = false,
                            playersHit = { c }, knockbackDirection = { vector.zero }
                        })
                        btnBlock.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
                        task.delay(0.2, function()
                            if btnBlock and btnBlock.Parent then
                                btnBlock.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
                            end
                        end)
                        task.delay(10, function()
                            healCooldown = false
                            if btnBlock and btnBlock.Parent then
                                btnBlock.ImageTransparency = 0.3
                                btnBlock.ImageColor3 = Color3.fromRGB(150, 255, 150)
                            end
                        end)
                    end
                end
            end
        end
    end)

    -- ============================================================
    -- ===== 7. МИНА (VFX) =====
    -- ============================================================
    local mineCooldown = false

    btnMine.MouseButton1Click:Connect(function()
        if mineCooldown then return end
        local c = LP.Character
        if not c then return end
        local hrp = c:FindFirstChild("HumanoidRootPart")
        if not hrp then return end

        triggerInvuln()
        mineCooldown = true
        btnMine.ImageTransparency = 0.8
        local minePosition = hrp.Position - Vector3.new(0, 2.5, 0)
        local mineCFrame = CFrame.new(minePosition)
        local ctrl = c:FindFirstChild("PlayerControllerScript")
        if ctrl then
            local remote = ctrl:FindFirstChild("PlayerInputEvent")
            if remote then
                remote:FireServer("doVFX", true, {
                    maxTilt = 0, radius = 0, hasCollision = false, startPos = mineCFrame,
                    maxRocks = 5, minRocks = 2, vfxType = "crater",
                    sizeTable = { maxY = 1, minY = 0.5, minX = 1, maxX = 2, minZ = 1, maxZ = 2 }
                })
            end
        end
        task.delay(3, function()
            if c and c.Parent and c:FindFirstChild("Humanoid") and c.Humanoid.Health > 0 then
                local ctrl2 = c:FindFirstChild("PlayerControllerScript")
                if ctrl2 then
                    local remote2 = ctrl2:FindFirstChild("PlayerInputEvent")
                    if remote2 then
                        remote2:FireServer("doVFX", true, {
                            maxTilt = 10, radius = 15, hasCollision = false, startPos = mineCFrame,
                            maxRocks = 30, minRocks = 15, vfxType = "crater",
                            sizeTable = { maxY = 6, minY = 2, minX = 2, maxX = 4, minZ = 2, maxZ = 4 }
                        })
                        local hitData = {}
                        for _, plr in ipairs(Players:GetPlayers()) do
                            if plr ~= LP and plr.Character then
                                local tHrp = plr.Character:FindFirstChild("HumanoidRootPart")
                                local tHum = plr.Character:FindFirstChildOfClass("Humanoid")
                                if tHrp and tHum and tHum.Health > 0 then
                                    local dist = (tHrp.Position - minePosition).Magnitude
                                    if dist <= 15 then
                                        local dir = (tHrp.Position - minePosition).Unit
                                        table.insert(hitData, {char = plr.Character, dir = dir})
                                    end
                                end
                            end
                        end
                        if #hitData > 0 then
                            for _, data in ipairs(hitData) do
                                remote2:FireServer("dealDamage", true, {
                                    hitstuns = true, blockable = false, ultGain = 1, parryable = false,
                                    damage = 60, knockbackForce = 150, hitsRagdolled = true,
                                    destruction = false, axesForce = { Y = 1/0, X = 1/0, Z = 1/0 },
                                    hitboxSize = vector.create(15, 15, 15), ragdolls = true,
                                    playersHit = { data.char }, knockbackDirection = { data.dir }
                                })
                                trackHit(data.char)
                            end
                        end
                    end
                end
            end
        end)
        task.delay(15, function()
            mineCooldown = false
            if btnMine and btnMine.Parent then btnMine.ImageTransparency = 0.3 end
        end)
    end)
end

-- ============================================================
-- ===== ПРОВЕРКА PARRY =====
-- ============================================================
local function checkParry()
    local backpack = LP:FindFirstChild("Backpack")
    local character = LP.Character
    local hasParry = false
    if backpack and backpack:FindFirstChild("Parry") then hasParry = true end
    if character and character:FindFirstChild("Parry") then hasParry = true end

    if hasParry then
        createInterface()
    else
        if gui then
            gui:Destroy()
            gui = nil
        end
    end
end

checkParry()

if LP:FindFirstChild("Backpack") then
    LP.Backpack.ChildAdded:Connect(checkParry)
    LP.Backpack.ChildRemoved:Connect(checkParry)
else
    LP.ChildAdded:Connect(function(child)
        if child.Name == "Backpack" then
            child.ChildAdded:Connect(checkParry)
            child.ChildRemoved:Connect(checkParry)
            checkParry()
        end
    end)
end

LP.CharacterAdded:Connect(function(char)
    task.wait(1)
    char.ChildAdded:Connect(function(child) if child.Name == "Parry" then checkParry() end end)
    char.ChildRemoved:Connect(function(child) if child.Name == "Parry" then checkParry() end end)
    checkParry()
end)

LP.CharacterAdded:Connect(function()
    task.wait(2)
    checkParry()
end)

-- ============================================================
-- ===== UNLOAD =====
-- ============================================================
UIS.InputBegan:Connect(function(i, gp)
    if gp then return end
    if i.KeyCode == Enum.KeyCode.RightShift then
        Unloaded = true
        if gui then gui:Destroy() end
    end
end)

print("Скрипт загружен: ОЧЕНЬ сильный удар (как в старом) + Авто-Whiplash + Авто-Инвулн!")
