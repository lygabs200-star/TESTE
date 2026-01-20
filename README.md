--////////////////////////////////////////////////////////////--
--  LOADSCRIPT UNIFICADO V32.0 (INTACTO - ONLY GUI FIX)       --
--////////////////////////////////////////////////////////////--

if not game:IsLoaded() then game.Loaded:Wait() end

local Players = game:GetService("Players")
local LP = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")

task.spawn(function()
    local character, humanoid, animator, hrp, hitboxPart
    local R = ReplicatedStorage:WaitForChild("LightsaberRemotes")
    local MouseDown, Attack, Swing, MouseUp = R:WaitForChild("MouseDown"), R:WaitForChild("Attack"), R:WaitForChild("Swing"), R:WaitForChild("MouseUp")
    local Block, Unblock, ResetDir, FinishSwing, OnHit = R:WaitForChild("Block"), R:WaitForChild("Unblock"), R:WaitForChild("ResetSwingDirection"), R:WaitForChild("FinishSwing"), R:WaitForChild("OnHit")

    local enabled = false
    local flickEnabled = true 
    local bodyFlickEnabled = true 
    local guarding = false
    local currentMode = "Saber" 
    local flickDuration, flickSmooth = 0.06, 8
    local isResetting = false 

    -- [SISTEMA CAMERA 360]
    local function doCamera360Flick()
        if not flickEnabled then return end
        local cam = workspace.CurrentCamera
        local startCF = cam.CFrame
        local startPos, pitch, yaw = startCF.Position, startCF:ToEulerAnglesYXZ()
        coroutine.wrap(function()
            for i=1, flickSmooth do
                cam.CFrame = CFrame.new(startPos) * CFrame.fromEulerAnglesYXZ(pitch, yaw + math.rad(360)*(i/flickSmooth), 0)
                task.wait(flickDuration / flickSmooth)
            end
        end)()
    end

    -- [SISTEMA BODY FLICK]
    local b_yaw, b_flickActive = 0, false
    local function executeBodyFlick()
        if not bodyFlickEnabled or b_flickActive then return end
        b_flickActive = true
        task.spawn(doCamera360Flick)
        if humanoid then for _, t in ipairs(humanoid:GetPlayingAnimationTracks()) do t:AdjustSpeed(3) end end
        task.delay(0.12, function() b_flickActive = false end)
    end

    local oldNameCall; oldNameCall = hookmetamethod(game, "__namecall", function(self, ...)
        if self == Attack and getnamecallmethod() == "FireServer" then task.spawn(executeBodyFlick) end
        return oldNameCall(self, ...)
    end)

    RunService.RenderStepped:Connect(function()
        if not hrp or not bodyFlickEnabled then if humanoid then humanoid.AutoRotate = true end return end
        if b_flickActive then
            humanoid.AutoRotate = false
            b_yaw = (b_yaw + 180) % 360 
            hrp.CFrame = CFrame.new(hrp.Position) * CFrame.Angles(0, math.rad(b_yaw), 0)
        else
            humanoid.AutoRotate = true
            local _, y, _ = hrp.CFrame:ToEulerAnglesYXZ()
            b_yaw = math.degrees(y)
        end
    end)

    local function watchAnimator(anim)
        if not anim or anim:FindFirstChild("__HitBlocker") then return end
        Instance.new("BoolValue", anim).Name = "__HitBlocker"
        anim.AnimationPlayed:Connect(function(track)
            local id = track.Animation and track.Animation.AnimationId
            if id and (id == "rbxassetid://12625862519" or id == "rbxassetid://12625868684") then track:Stop() end
        end)
    end

    local function refreshCharacter()
        character = LP.Character or LP.CharacterAdded:Wait()
        humanoid  = character:WaitForChild("Humanoid")
        animator  = humanoid:WaitForChild("Animator")
        hrp       = character:WaitForChild("HumanoidRootPart")
        watchAnimator(animator)
    end
    LP.CharacterAdded:Connect(refreshCharacter); refreshCharacter()

    -- LISTAS DE ANIMAÇÕES
    local saberIds = {[1]="rbxassetid://12625839385",[2]="rbxassetid://12625848489",[4]="rbxassetid://12625853257",[5]="rbxassetid://12625843823",[6]="rbxassetid://12625846167",[8]="rbxassetid://12625851115"}
    local staffIds = {[1]="rbxassetid://12718503706",[2]="rbxassetid://12718486016",[3]="rbxassetid://12718504431",[4]="rbxassetid://12718500875"}
    local formaIIds = {[1]="rbxassetid://12625848489",[2]="rbxassetid://12625853257",[3]="rbxassetid://12625851115",[4]="rbxassetid://12625846167"}
    local wizardIds = {[1]="rbxassetid://12625841878",[2]="rbxassetid://12625841878",[3]="rbxassetid://12625853257",[4]="rbxassetid://12625841878",[5]="rbxassetid://12625848489",[6]="rbxassetid://12625846167",[7]="rbxassetid://12625848489",[8]="rbxassetid://12625853257",[9]="rbxassetid://12625841878",[10]="rbxassetid://12625848489",[11]="rbxassetid://12625846167",[12]="rbxassetid://12625843823",[13]="rbxassetid://12625846167",[14]="rbxassetid://12625841878"}
    local comboZuIds = {"rbxassetid://12718501806","rbxassetid://12718501806","rbxassetid://12718501806","rbxassetid://12718501806","rbxassetid://12718483984","rbxassetid://12718483984","rbxassetid://12718483984","rbxassetid://12718483984","rbxassetid://12718486016","rbxassetid://12718486016","rbxassetid://12718486016","rbxassetid://12718486016","rbxassetid://12718504431","rbxassetid://12718504431","rbxassetid://12718504431","rbxassetid://12718504431"}
    local godComboIds = {"rbxassetid://12625853257", "rbxassetid://12625843823", "rbxassetid://12625846167", "rbxassetid://12625846167", "rbxassetid://12625846167", "rbxassetid://12625846167"}
    local baguasIds = {"rbxassetid://12625846167", "rbxassetid://12625848489", "rbxassetid://12625853257"}
    local cyanIds = {"rbxassetid://12625846167", "rbxassetid://12625843823", "rbxassetid://12625846167", "rbxassetid://12625848489", "rbxassetid://12625846167", "rbxassetid://12625848489", "rbxassetid://12625843823", "rbxassetid://12625848489", "rbxassetid://12625841878", "rbxassetid://12625848489"}
    local tripleStIds = {"rbxassetid://17372039079", "rbxassetid://17566657634", "rbxassetid://17372038496", "rbxassetid://17372041039", "rbxassetid://17372037456", "rbxassetid://17372036678", "rbxassetid://17566667400"}
    local foiceClanIds = {"rbxassetid://15563346564", "rbxassetid://15564066873", "rbxassetid://15563343338", "rbxassetid://15563342470", "rbxassetid://15563346027", "rbxassetid://15563344914", "rbxassetid://15563343338"}
    local turtleIds = {"rbxassetid://13783293417", "rbxassetid://13781667793", "rbxassetid://13781621647", "rbxassetid://13783202348", "rbxassetid://13781663786", "rbxassetid://13783395464", "rbxassetid://13783497920"}
    local tonfaIds = {"rbxassetid://13304786458","rbxassetid://13304774028","rbxassetid://13306520673","rbxassetid://13304781510"}

    local function findHitbox()
        hitboxPart = nil
        local w = character:FindFirstChild(LP.Name.."Saber1", true) or character:FindFirstChild("Staff", true) or character:FindFirstChild("Double", true)
        if w then hitboxPart = w:FindFirstChild("Hitbox", true) end
    end

    local seq, ptr, touchConn = {}, 1, nil
    local function newSeq()
        seq, ptr = {}, 1
        if currentMode == "TONFA" then
            for i=1, 4 do table.insert(seq, i) end
            for i=1, 4 do table.insert(seq, math.random(#tonfaIds)) end
        elseif currentMode == "Wizard" then for i=1,#wizardIds do table.insert(seq,i) end
        elseif currentMode == "Combo Zu" then for i=1,#comboZuIds do table.insert(seq,i) end
        elseif currentMode == "GOD Combo" then for i=1,#godComboIds do table.insert(seq,i) end
        elseif currentMode == "BAGUAS AURA" then for i=1,#baguasIds do table.insert(seq,i) end
        elseif currentMode == "CYAN MODE" then for i=1,#cyanIds do table.insert(seq,i) end
        elseif currentMode == "TRIPLE ST" or currentMode == "FOICE CLAN" or currentMode == "TURTLE NINJA" then
            local targetTable = (currentMode == "TRIPLE ST" and tripleStIds) or (currentMode == "FOICE CLAN" and foiceClanIds) or turtleIds
            for i=1, 4 do table.insert(seq, math.random(#targetTable)) end
        else
            local pool = (currentMode=="Saber" and {1,2,4,5,6,8}) or (currentMode=="Staff" and {1,2,3,4}) or (currentMode=="Forma I" and {1,2,3,4}) or {1,2,3,4}
            for i=1,#pool do table.insert(seq, table.remove(pool, math.random(#pool))) end
        end
    end

    local function guardOn() if not guarding then Block:FireServer() guarding=true end end
    local function guardOff() if guarding then Unblock:FireServer() guarding=false end end

    local function swingNext()
        if not enabled or isResetting then return end
        local isSpecialMode = (currentMode == "TRIPLE ST" or currentMode == "FOICE CLAN" or currentMode == "TURTLE NINJA" or currentMode == "TONFA")
        local maxPtr = isSpecialMode and 4 or #seq
        if ptr > maxPtr then 
            isResetting = true; guardOn(); task.wait(0.2); isResetting = false; newSeq(); ptr = 1
        end
        local dir = seq[ptr]
        local speed = (currentMode=="TONFA" and 1.35) or (currentMode=="TURTLE NINJA" and 1.5) or (currentMode=="FOICE CLAN" and 1.4) or (currentMode=="TRIPLE ST" and 1.3) or (currentMode=="CYAN MODE" and 1.15) or (currentMode=="BAGUAS AURA" and 1.2) or (currentMode=="GOD Combo" and 1.2) or (currentMode=="Combo Zu" and 2.8) or (currentMode=="Wizard" and 1.95) or (currentMode=="Staff" and 1.8) or 1.25
        findHitbox()
        local function getAnim(id) local a=Instance.new("Animation") a.AnimationId=id return a end
        local currentIds = (currentMode=="Saber" and saberIds) or (currentMode=="Staff" and staffIds) or (currentMode=="Forma I" and formaIIds) or (currentMode=="Wizard" and wizardIds) or (currentMode=="Combo Zu" and comboZuIds) or (currentMode=="GOD Combo" and godComboIds) or (currentMode=="BAGUAS AURA" and baguasIds) or (currentMode=="CYAN MODE" and cyanIds) or (currentMode=="TRIPLE ST" and tripleStIds) or (currentMode=="FOICE CLAN" and foiceClanIds) or (currentMode=="TURTLE NINJA" and turtleIds) or (currentMode=="TONFA" and tonfaIds) or saberIds
        local track = animator:LoadAnimation(getAnim(currentIds[dir] or currentIds[1]))
        track:Play(0.05, 1, speed)
        task.wait(0.02)
        local gotHit = false
        task.delay(0.04, doCamera360Flick)
        track.KeyframeReached:Connect(function(f)
            if f ~= "Start" or touchConn or not hitboxPart or isResetting then return end
            touchConn = hitboxPart.Touched:Connect(function(p)
                if gotHit or isResetting then return end
                local tgt = p:FindFirstAncestorWhichIsA("Model")
                if tgt and tgt ~= character and tgt:FindFirstChild("Humanoid") then
                    gotHit=true; guardOff(); MouseDown:FireServer(); Attack:FireServer(dir,1,false,false)
                    Swing:FireServer(); OnHit:FireServer(tgt); track:AdjustSpeed(-speed*1.1); guardOn()
                end
            end)
        end)
        track.Stopped:Connect(function()
            if touchConn then touchConn:Disconnect() touchConn=nil end
            ResetDir:FireServer(); FinishSwing:FireServer(); MouseUp:FireServer()
            ptr = ptr + 1; guardOn()
            local rTime = (currentMode=="TONFA" and 0.02) or (currentMode=="TURTLE NINJA" and 0.05) or (currentMode=="FOICE CLAN" and 0.07) or (currentMode=="TRIPLE ST" and 0.08) or 0.06
            if enabled then task.delay(rTime, swingNext) end
        end)
    end

    -- [GUI SETUP]
    local gui = Instance.new("ScreenGui", LP.PlayerGui); gui.Name = "SaberUniversal"; gui.ResetOnSpawn = false
    
    local toggleBtn = Instance.new("TextButton", gui)
    toggleBtn.Size, toggleBtn.Position = UDim2.new(0, 70, 0, 30), UDim2.fromOffset(50, 120)
    toggleBtn.Text = "MENU"; toggleBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 80); toggleBtn.TextColor3 = Color3.new(1,1,1); toggleBtn.Draggable, toggleBtn.Active = true, true; Instance.new("UICorner", toggleBtn)

    local autoBtn = Instance.new("TextButton", gui)
    autoBtn.Size, autoBtn.Position = UDim2.new(0, 150, 0, 40), UDim2.fromOffset(130, 115)
    autoBtn.Text = "AUTO: OFF (Y)"; autoBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 65); autoBtn.TextColor3 = Color3.new(1,1,1); autoBtn.Draggable, autoBtn.Active = true, true; Instance.new("UICorner", autoBtn)

    local win = Instance.new("Frame", gui)
    win.Size, win.Position = UDim2.new(0, 200, 0, 200), UDim2.fromOffset(50, 160)
    win.BackgroundColor3 = Color3.fromRGB(35, 35, 40); win.Active, win.Draggable = true, true; win.Visible = true; Instance.new("UICorner", win)

    -- [BOTÕES DO MENU]
    local modeBtn = Instance.new("TextButton", win)
    modeBtn.Size, modeBtn.Position = UDim2.new(1, -20, 0, 40), UDim2.new(0, 10, 0, 10)
    modeBtn.Text = "MODE: SABER (M)"; modeBtn.BackgroundColor3 = Color3.fromRGB(0, 100, 200); modeBtn.TextColor3 = Color3.new(1,1,1); Instance.new("UICorner", modeBtn)

    local flickBtn = Instance.new("TextButton", win) 
    flickBtn.Size, flickBtn.Position = UDim2.new(1, -20, 0, 40), UDim2.new(0, 10, 0, 55)
    flickBtn.Text = "360 CAM: ON (F8)"; flickBtn.BackgroundColor3 = Color3.fromRGB(0, 150, 0); flickBtn.TextColor3 = Color3.new(1,1,1); Instance.new("UICorner", flickBtn)

    local bodyBtn = Instance.new("TextButton", win)
    bodyBtn.Size, bodyBtn.Position = UDim2.new(1, -20, 0, 40), UDim2.new(0, 10, 0, 100)
    bodyBtn.Text = "BODY GIRO: ON (J)"; bodyBtn.BackgroundColor3 = Color3.fromRGB(0, 180, 0); bodyBtn.TextColor3 = Color3.new(1,1,1); Instance.new("UICorner", bodyBtn)

    local gameOpacityBtn = Instance.new("TextButton", win)
    gameOpacityBtn.Size, gameOpacityBtn.Position = UDim2.new(1, -20, 0, 40), UDim2.new(0, 10, 0, 145)
    gameOpacityBtn.Text = "BOTÕES JOGO: 0%"; gameOpacityBtn.BackgroundColor3 = Color3.fromRGB(120, 120, 120); gameOpacityBtn.TextColor3 = Color3.new(1,1,1); Instance.new("UICorner", gameOpacityBtn)

    -- [LÓGICA DE TOGGLE AUTO]
    local function handleAutoToggle()
        enabled = not enabled
        autoBtn.Text = "AUTO: " .. (enabled and "ON (Y)" or "OFF (Y)")
        autoBtn.BackgroundColor3 = enabled and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(60, 60, 65)
        if enabled then isResetting = false; newSeq(); swingNext() end
    end

    -- [LÓGICA DE ESCONDER TUDO (FIXED)]
    local isMenuOpen = true
    toggleBtn.MouseButton1Click:Connect(function()
        isMenuOpen = not isMenuOpen
        win.Visible = isMenuOpen
        if isMenuOpen then
            toggleBtn.BackgroundTransparency = 0
            toggleBtn.TextTransparency = 0
        else
            -- Fica invisível mas clicável para você voltar quando quiser
            toggleBtn.BackgroundTransparency = 1
            toggleBtn.TextTransparency = 1
        end
    end)

    local function showMenu()
        isMenuOpen = true
        win.Visible = true
        toggleBtn.BackgroundTransparency = 0
        toggleBtn.TextTransparency = 0
    end

    local isInvis = false
    gameOpacityBtn.MouseButton1Click:Connect(function()
        isInvis = not isInvis
        local trans = isInvis and 1 or 0
        gameOpacityBtn.Text = "BOTÕES JOGO: " .. (isInvis and "100%" or "0%")
        gameOpacityBtn.BackgroundColor3 = isInvis and Color3.fromRGB(255, 85, 0) or Color3.fromRGB(120, 120, 120)
        for _, obj in pairs(LP.PlayerGui:GetDescendants()) do
            if obj:IsDescendantOf(gui) then continue end
            if obj:IsA("TextButton") or obj:IsA("ImageButton") then
                pcall(function()
                    obj.BackgroundTransparency = trans
                    if obj:IsA("TextButton") then obj.TextTransparency = trans else obj.ImageTransparency = trans end
                end)
            end
        end
    end)

    local function doToggleMode()
        if currentMode == "Saber" then currentMode = "Staff"
        elseif currentMode == "Staff" then currentMode = "Forma I"
        elseif currentMode == "Forma I" then currentMode = "Wizard"
        elseif currentMode == "Wizard" then currentMode = "Combo Zu"
        elseif currentMode == "Combo Zu" then currentMode = "GOD Combo"
        elseif currentMode == "GOD Combo" then currentMode = "BAGUAS AURA"
        elseif currentMode == "BAGUAS AURA" then currentMode = "CYAN MODE"
        elseif currentMode == "CYAN MODE" then currentMode = "TRIPLE ST"
        elseif currentMode == "TRIPLE ST" then currentMode = "FOICE CLAN"
        elseif currentMode == "FOICE CLAN" then currentMode = "TURTLE NINJA"
        elseif currentMode == "TURTLE NINJA" then currentMode = "TONFA"
        else currentMode = "Saber" end
        modeBtn.Text = "MODE: " .. currentMode:upper() .. " (M)"
        modeBtn.BackgroundColor3 = (currentMode=="TONFA" and Color3.fromRGB(200, 200, 200)) or (currentMode=="TURTLE NINJA" and Color3.fromRGB(50, 200, 50)) or (currentMode=="FOICE CLAN" and Color3.fromRGB(255, 100, 0)) or (currentMode=="TRIPLE ST" and Color3.fromRGB(255, 50, 255)) or (currentMode=="CYAN MODE" and Color3.fromRGB(0, 255, 255)) or (currentMode=="BAGUAS AURA" and Color3.fromRGB(255, 120, 0)) or (currentMode=="GOD Combo" and Color3.fromRGB(255, 215, 0)) or (currentMode=="Combo Zu" and Color3.fromRGB(255, 0, 50)) or (currentMode=="Forma I" and Color3.fromRGB(150, 0, 150)) or Color3.fromRGB(0, 100, 200)
        newSeq()
    end

    autoBtn.MouseButton1Click:Connect(handleAutoToggle)
    modeBtn.MouseButton1Click:Connect(doToggleMode)
    flickBtn.MouseButton1Click:Connect(function()
        flickEnabled = not flickEnabled
        flickBtn.Text = "360 CAM: " .. (flickEnabled and "ON (F8)" or "OFF (F8)")
        flickBtn.BackgroundColor3 = flickEnabled and Color3.fromRGB(0, 150, 0) or Color3.fromRGB(150, 0, 0)
    end)
    bodyBtn.MouseButton1Click:Connect(function()
        bodyFlickEnabled = not bodyFlickEnabled
        bodyBtn.Text = "BODY GIRO: " .. (bodyFlickEnabled and "ON (J)" or "OFF (J)")
        bodyBtn.BackgroundColor3 = bodyFlickEnabled and Color3.fromRGB(0, 180, 0) or Color3.fromRGB(180, 0, 0)
    end)

    UserInputService.InputBegan:Connect(function(input, p)
        if p then return end
        if input.KeyCode == Enum.KeyCode.Y then 
            handleAutoToggle()
            if not isMenuOpen then showMenu() end 
        elseif input.KeyCode == Enum.KeyCode.M then doToggleMode()
        elseif input.KeyCode == Enum.KeyCode.F8 then flickBtn:Click()
        elseif input.KeyCode == Enum.KeyCode.J then bodyBtn:Click() end
    end)
end)
