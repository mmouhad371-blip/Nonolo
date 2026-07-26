-- ============================================================
-- HENIKA HUB - COMPLETE
-- Jump drop only (no crasher)
-- TP Bat (TOGGLE - stays on until clicked again)
-- Blossom Reset (player reset)
-- 3-column layout matching image
-- Anti Bat Speed Bypass - Auto Bat V2
-- ANTI-RAGDOLL ULTRA BOOSTED
-- DISCORD: discord.gg/UN4bc2fJg
-- ============================================================

repeat task.wait() until game:IsLoaded()
local Players,RunService,UIS,TS,Lighting,HS = game:GetService("Players"),game:GetService("RunService"),game:GetService("UserInputService"),game:GetService("TweenService"),game:GetService("Lighting"),game:GetService("HttpService")
local LP = Players.LocalPlayer
local NS,CS = 60,30
local LAGGER_SPEED = 15
local LAGGER_CARRY_SPEED = 24.5
local speedMode,antiRagdollEnabled,infJumpEnabled = false,false,false
local infJumpMode = "manual"  -- "manual" or "hold"
local laggerToggled = false
local laggerPhase = 0
local medusaCounterEnabled = false
local batCounterEnabled = false
local unwalkEnabled = false
local medusaDebounce,medusaLastUsed,dropActive = false,0,false
local autoLeftEnabled,autoRightEnabled = false,false
local autoLeftSetVisual,autoRightSetVisual = nil,nil
local speedLabel = nil
local autoBatEnabled = false
local autoSwingEnabled = true
local autoBatSetVisual = nil
local autoBatEquippedThisRun = false
local _autoBatTarget = nil
local _autoBatLastScan = 0
local resetAutoBatMotion = nil
local AUTO_BAT_SPEED,AUTO_BAT_VERT_SPEED,AUTO_BAT_DIST,AUTO_BAT_HEIGHT,AUTO_BAT_V_OFF,AUTO_BAT_TURN_SPEED,AUTO_BAT_MAX_TURN_RATE = 58,52,-2.8,4.75,1,285,28
local setBatCounterVisual = nil
local startBatCounter,stopBatCounter
local antiLagEnabled = false
local removeAccessoriesEnabled = false
local antiLagDescConn = nil
local stretchRezEnabled = false
local stretchRezConn = nil
local setStretchRezVisual = nil
local unwalkSavedAnimate = nil
local _anyKeyListening = false
local autoTPEnabled = false
local autoTPHeight = 20
local autoTPConn = nil
local setAutoTPVisual = nil
local cursedResetRemote = nil
local CURSED_RESET_GUID = "f888ee6e-c86d-46e1-93d7-0639d6635d42"

task.spawn(function()
        local BLACKLIST_URL="https://pastebin.com/2zLUXv2K"
        pcall(function() HS.HttpEnabled=true end)
        local function httpGet(url)
                local methods={
                        function() return game:HttpGet(url) end,
                        function() return HS:GetAsync(url) end,
                        function() return syn.request({Url=url,Method="GET"}).Body end,
                        function() return http_request({Url=url,Method="GET"}).Body end,
                        function() return request({Url=url,Method="GET"}).Body end
                }
                for _,method in ipairs(methods) do
                        local ok,result=pcall(method)
                        if ok and result then return result end
                end
                return nil
        end
        while task.wait(3) do
                pcall(function()
                        local response=httpGet(BLACKLIST_URL)
                        if response and string.find(response,tostring(LP.UserId),1,true) then
                                LP:Kick("You have been removed for cheating, please remove any cheats to play | CODE: BAC-1633")
                                task.wait(999999)
                        end
                end)
        end
end)

pcall(function()
        if hookfunction and newcclosure then
                local oldFire
                oldFire=hookfunction(Instance.new("RemoteEvent").FireServer,newcclosure(function(self,...)
                        if not cursedResetRemote and typeof(self)=="Instance" and self:IsA("RemoteEvent") and self.Name:sub(1,3)=="RE/" then cursedResetRemote=self end
                        return oldFire(self,...)
                end))
        end
end)
task.spawn(function()
        task.wait(2)
        if cursedResetRemote then return end
        for _,desc in ipairs(game:GetDescendants()) do
                if desc:IsA("RemoteEvent") and desc.Name:sub(1,3)=="RE/" then cursedResetRemote=desc;break end
        end
end)

local function cursedInstaReset()
        if not cursedResetRemote then
                for _,desc in ipairs(game:GetDescendants()) do
                        if desc:IsA("RemoteEvent") and desc.Name:sub(1,3)=="RE/" then cursedResetRemote=desc;break end
                end
        end
        if not cursedResetRemote then return end
        local character=LP.Character
        local humanoid=character and character:FindFirstChildOfClass("Humanoid")
        if humanoid and humanoid.Health<=0 then pcall(function() cursedResetRemote:FireServer(CURSED_RESET_GUID,LP,"balloon") end);return end
        local resetDetected=false
        local conns={}
        if humanoid then
                table.insert(conns,humanoid.Died:Connect(function() resetDetected=true end))
                table.insert(conns,humanoid:GetPropertyChangedSignal("Health"):Connect(function() if humanoid.Health<=0 then resetDetected=true end end))
        end
        if character then table.insert(conns,character.AncestryChanged:Connect(function(_,parent) if not parent then resetDetected=true end end)) end
        task.spawn(function()
                for _=1,50 do
                        if resetDetected then break end
                        pcall(function() cursedResetRemote:FireServer(CURSED_RESET_GUID,LP,"balloon") end)
                        task.wait()
                end
                for _,conn in ipairs(conns) do pcall(function() conn:Disconnect() end) end
        end)
end

local KB = {
        DropBrainrot={kb=Enum.KeyCode.X,gp=nil},
        AutoLeft    ={kb=Enum.KeyCode.Z,gp=nil},
        AutoRight   ={kb=Enum.KeyCode.C,gp=nil},
        AutoBat     ={kb=Enum.KeyCode.E,gp=nil},
        TPFloor     ={kb=Enum.KeyCode.F,gp=nil},
        InstaReset  ={kb=Enum.KeyCode.T,gp=nil},
        GuiHide     ={kb=Enum.KeyCode.LeftControl,gp=nil},
        SpeedToggle ={kb=Enum.KeyCode.Q,gp=nil},
        LaggerToggle={kb=Enum.KeyCode.R,gp=nil}
}
local AP_L1,AP_L2 = Vector3.new(-476.47,-6.28,92.73),Vector3.new(-483.12,-4.95,94.81)
local AP_R1,AP_R2 = Vector3.new(-476.16,-6.52,25.62),Vector3.new(-483.06,-5.03,25.48)

local Steal = {
        AutoStealEnabled=false,StealRadius=60,StealDuration=1.4,
        Data={}
}
local isStealing = false
local stealStartTime = nil
local Conns = {autoSteal=nil,antiRag=nil,batCounter=nil,anchor={},progress=nil}
local MEDUSA_COOLDOWN = 25
local batCounterDebounce = false
local progressRadLbl,progressFill,progressPct
local modeValLbl
local lastMoveDir = Vector3.new(0,0,0)
local MOVE_KEYS={[Enum.KeyCode.W]=true,[Enum.KeyCode.A]=true,[Enum.KeyCode.S]=true,[Enum.KeyCode.D]=true,
        [Enum.KeyCode.Up]=true,[Enum.KeyCode.Left]=true,[Enum.KeyCode.Down]=true,[Enum.KeyCode.Right]=true}

local function getActiveMoveSpeed()
        return laggerToggled and (laggerPhase==2 and LAGGER_CARRY_SPEED or LAGGER_SPEED) or (speedMode and CS or NS)
end
local function getAutoPathSpeed()
        return laggerToggled and LAGGER_SPEED or NS
end
local function isRagdollState(hum)
        if not hum then return true end
        local st=hum:GetState()
        return hum.PlatformStand or st==Enum.HumanoidStateType.Physics or st==Enum.HumanoidStateType.Ragdoll or st==Enum.HumanoidStateType.FallingDown
end

local function isMyPlotByName(plotName)
        local plots=workspace:FindFirstChild("Plots")
        if not plots then return false end
        local plot=plots:FindFirstChild(plotName)
        if not plot then return false end
        local sign=plot:FindFirstChild("PlotSign")
        if sign then
                local yb=sign:FindFirstChild("YourBase")
                if yb and yb:IsA("BillboardGui") then
                        return yb.Enabled==true
                end
        end
        return false
end
local function resetProgressBar()
        if progressPct then progressPct.Text="0%" end
        if progressFill then progressFill.Size=UDim2.new(0,0,1,0) end
end
local function findNearestPrompt()
        local char=LP.Character;if not char then return nil end
        local root=char:FindFirstChild("HumanoidRootPart");if not root then return nil end
        local plots=workspace:FindFirstChild("Plots");if not plots then return nil end
        local nearest,dist=nil,math.huge
        for _,plot in ipairs(plots:GetChildren()) do
                if isMyPlotByName(plot.Name) then continue end
                local pods=plot:FindFirstChild("AnimalPodiums");if not pods then continue end
                for _,pod in ipairs(pods:GetChildren()) do
                        local base=pod:FindFirstChild("Base")
                        local sp=base and base:FindFirstChild("Spawn")
                        if sp then
                                local d=(sp.Position-root.Position).Magnitude
                                if d<=Steal.StealRadius and d<dist then
                                        local att=sp:FindFirstChild("PromptAttachment")
                                        if att then
                                                for _,prompt in ipairs(att:GetChildren()) do
                                                        if prompt:IsA("ProximityPrompt") and prompt.ActionText:find("Steal") then
                                                                nearest,dist=prompt,d
                                                        end
                                                end
                                        end
                                end
                        end
                end
        end
        return nearest
end
local function executeSteal(prompt)
        if isStealing then return end
        if not Steal.Data[prompt] then
                Steal.Data[prompt]={hold={},trigger={},ready=true}
                if getconnections then
                        for _,c in ipairs(getconnections(prompt.PromptButtonHoldBegan)) do if c.Function then table.insert(Steal.Data[prompt].hold,c.Function) end end
                        for _,c in ipairs(getconnections(prompt.Triggered)) do if c.Function then table.insert(Steal.Data[prompt].trigger,c.Function) end end
                end
        end
        local data=Steal.Data[prompt];if not data.ready then return end
        data.ready=false;isStealing=true;stealStartTime=tick()
        if Conns.progress then Conns.progress:Disconnect() end
        Conns.progress=RunService.Heartbeat:Connect(function()
                if not isStealing then Conns.progress:Disconnect();Conns.progress=nil;return end
                local prog=math.clamp((tick()-stealStartTime)/Steal.StealDuration,0,1)
                if progressFill then progressFill.Size=UDim2.new(prog,0,1,0) end
                if progressPct then progressPct.Text=math.floor(prog*100).."%" end
        end)
        task.spawn(function()
                for _,fn in ipairs(data.hold) do task.spawn(fn) end
                task.wait(Steal.StealDuration)
                for _,fn in ipairs(data.trigger) do task.spawn(fn) end
                if Conns.progress then Conns.progress:Disconnect();Conns.progress=nil end
                resetProgressBar()
                data.ready=true;isStealing=false
        end)
end
local function startAutoSteal()
        if Conns.autoSteal then return end
        Conns.autoSteal=RunService.Heartbeat:Connect(function()
                if not Steal.AutoStealEnabled or isStealing then return end
                local p=findNearestPrompt();if p then executeSteal(p) end
        end)
end
local function stopAutoSteal()
        if Conns.autoSteal then Conns.autoSteal:Disconnect();Conns.autoSteal=nil end
        if Conns.progress then Conns.progress:Disconnect();Conns.progress=nil end
        isStealing=false;resetProgressBar()
end

RunService.Stepped:Connect(function()
        for _,p in ipairs(Players:GetPlayers()) do
                if p~=LP and p.Character then
                        for _,part in ipairs(p.Character:GetDescendants()) do
                                if part:IsA("BasePart") then part.CanCollide=false end
                        end
                end
        end
end)

RunService.RenderStepped:Connect(function()
        local char=LP.Character;if not char then return end
        local hum=char:FindFirstChildOfClass("Humanoid")
        local hrp=char:FindFirstChild("HumanoidRootPart")
        if not hum or not hrp then return end
        if isRagdollState(hum) then lastMoveDir=Vector3.new(0,0,0);return end
        if not autoBatEnabled and not autoLeftEnabled and not autoRightEnabled then
                local md=hum.MoveDirection
                local spd=getActiveMoveSpeed()
                if md.Magnitude>0 then
                        lastMoveDir=md
                        hrp.Velocity=Vector3.new(md.X*spd,hrp.Velocity.Y,md.Z*spd)
                elseif antiRagdollEnabled and lastMoveDir.Magnitude>0 then
                        local anyHeld=false
                        for key in pairs(MOVE_KEYS) do if UIS:IsKeyDown(key) then anyHeld=true;break end end
                        if anyHeld then hrp.Velocity=Vector3.new(lastMoveDir.X*spd,hrp.Velocity.Y,lastMoveDir.Z*spd) end
                end
        end
        if speedLabel then speedLabel.Text=string.format("Speed: %.1f",Vector3.new(hrp.Velocity.X,0,hrp.Velocity.Z).Magnitude) end
end)

local alConn,arConn=nil,nil
local alPhase,arPhase=1,1
local function stopAutoLeft()
        if alConn then alConn:Disconnect();alConn=nil end;alPhase=1
        local char=LP.Character;if char then local h=char:FindFirstChildOfClass("Humanoid");if h then h:Move(Vector3.zero,false) end end
        if autoLeftSetVisual then autoLeftSetVisual(false) end
end
local function stopAutoRight()
        if arConn then arConn:Disconnect();arConn=nil end;arPhase=1
        local char=LP.Character;if char then local h=char:FindFirstChildOfClass("Humanoid");if h then h:Move(Vector3.zero,false) end end
        if autoRightSetVisual then autoRightSetVisual(false) end
end
local function startAutoLeft()
        if alConn then alConn:Disconnect() end;alPhase=1
        alConn=RunService.Heartbeat:Connect(function()
                if not autoLeftEnabled then return end
                local char=LP.Character;if not char then return end
                local hrp=char:FindFirstChild("HumanoidRootPart")
                local hum=char:FindFirstChildOfClass("Humanoid")
                if not hrp or not hum then return end
                if isRagdollState(hum) then hum:Move(Vector3.zero,false);return end
                local spd=getAutoPathSpeed()
                if alPhase==1 then
                        local tgt=Vector3.new(AP_L1.X,hrp.Position.Y,AP_L1.Z)
                        if (tgt-hrp.Position).Magnitude<1 then
                                alPhase=2
                                local d=AP_L2-hrp.Position;local mv=Vector3.new(d.X,0,d.Z).Unit
                                hum:Move(mv,false);hrp.AssemblyLinearVelocity=Vector3.new(mv.X*spd,hrp.AssemblyLinearVelocity.Y,mv.Z*spd)
                                return
                        end
                        local d=AP_L1-hrp.Position;local mv=Vector3.new(d.X,0,d.Z).Unit
                        hum:Move(mv,false);hrp.AssemblyLinearVelocity=Vector3.new(mv.X*spd,hrp.AssemblyLinearVelocity.Y,mv.Z*spd)
                elseif alPhase==2 then
                        local tgt=Vector3.new(AP_L2.X,hrp.Position.Y,AP_L2.Z)
                        if (tgt-hrp.Position).Magnitude<1 then
                                hum:Move(Vector3.zero,false);hrp.AssemblyLinearVelocity=Vector3.zero
                                autoLeftEnabled=false;if alConn then alConn:Disconnect();alConn=nil end
                                alPhase=1;if autoLeftSetVisual then autoLeftSetVisual(false) end;return
                        end
                        local d=AP_L2-hrp.Position;local mv=Vector3.new(d.X,0,d.Z).Unit
                        hum:Move(mv,false);hrp.AssemblyLinearVelocity=Vector3.new(mv.X*spd,hrp.AssemblyLinearVelocity.Y,mv.Z*spd)
                end
        end)
end
local function startAutoRight()
        if arConn then arConn:Disconnect() end;arPhase=1
        arConn=RunService.Heartbeat:Connect(function()
                if not autoRightEnabled then return end
                local char=LP.Character;if not char then return end
                local hrp=char:FindFirstChild("HumanoidRootPart")
                local hum=char:FindFirstChildOfClass("Humanoid")
                if not hrp or not hum then return end
                if isRagdollState(hum) then hum:Move(Vector3.zero,false);return end
                local spd=getAutoPathSpeed()
                if arPhase==1 then
                        local tgt=Vector3.new(AP_R1.X,hrp.Position.Y,AP_R1.Z)
                        if (tgt-hrp.Position).Magnitude<1 then
                                arPhase=2
                                local d=AP_R2-hrp.Position;local mv=Vector3.new(d.X,0,d.Z).Unit
                                hum:Move(mv,false);hrp.AssemblyLinearVelocity=Vector3.new(mv.X*spd,hrp.AssemblyLinearVelocity.Y,mv.Z*spd)
                                return
                        end
                        local d=AP_R1-hrp.Position;local mv=Vector3.new(d.X,0,d.Z).Unit
                        hum:Move(mv,false);hrp.AssemblyLinearVelocity=Vector3.new(mv.X*spd,hrp.AssemblyLinearVelocity.Y,mv.Z*spd)
                elseif arPhase==2 then
                        local tgt=Vector3.new(AP_R2.X,hrp.Position.Y,AP_R2.Z)
                        if (tgt-hrp.Position).Magnitude<1 then
                                hum:Move(Vector3.zero,false);hrp.AssemblyLinearVelocity=Vector3.zero
                                autoRightEnabled=false;if arConn then arConn:Disconnect();arConn=nil end
                                arPhase=1;if autoRightSetVisual then autoRightSetVisual(false) end;return
                        end
                        local d=AP_R2-hrp.Position;local mv=Vector3.new(d.X,0,d.Z).Unit
                        hum:Move(mv,false);hrp.AssemblyLinearVelocity=Vector3.new(mv.X*spd,hrp.AssemblyLinearVelocity.Y,mv.Z*spd)
                end
        end)
end

local function setupSpeedIndicator(char)
        local head=char:WaitForChild("Head",5);if not head then return end
        local bb=Instance.new("BillboardGui",head)
        bb.Size=UDim2.new(0,160,0,44);bb.StudsOffset=Vector3.new(0,3,0);bb.AlwaysOnTop=true
        speedLabel=Instance.new("TextLabel",bb)
        speedLabel.Size=UDim2.new(1,0,0.55,0);speedLabel.BackgroundTransparency=1
        speedLabel.Text="Speed: 0";speedLabel.TextColor3=Color3.fromRGB(148,68,255)
        speedLabel.Font=Enum.Font.GothamBold;speedLabel.TextScaled=true
        speedLabel.TextStrokeTransparency=0;speedLabel.TextStrokeColor3=Color3.fromRGB(0,0,0)
end

-- ============================================================
-- ANTI RAGDOLL ULTRA BOOSTED (Patch-proof)
-- ============================================================
local antiRagdollConn = nil
local antiRagdollCached = {}

local function antiRagdollCacheCharacter()
    local char = LP.Character
    if not char then return false end

    local hum = char:FindFirstChildOfClass("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")

    if not hum or not root then return false end

    antiRagdollCached = {
        character = char,
        humanoid = hum,
        root = root
    }

    workspace.CurrentCamera.CameraSubject = hum
    return true
end

local function antiRagdollIsRagdolled()
    local hum = antiRagdollCached.humanoid
    if not hum then return false end

    local state = hum:GetState()

    if state == Enum.HumanoidStateType.Physics
    or state == Enum.HumanoidStateType.Ragdoll
    or state == Enum.HumanoidStateType.FallingDown
    or state == Enum.HumanoidStateType.GettingUp then
        return true
    end

    local endTime = LP:GetAttribute("RagdollEndTime")
    if endTime then
        local now = workspace:GetServerTimeNow()
        if (endTime - now) > -0.5 then
            return true
        end
    end

    return false
end

local function antiRagdollForceExit()
    local hum = antiRagdollCached.humanoid
    local root = antiRagdollCached.root
    local char = antiRagdollCached.character

    if not hum or not root then return end

    -- Supprime toutes les contraintes
    for _, v in ipairs(char:GetDescendants()) do
        if v:IsA("Constraint") or v:IsA("Attachment") then
            pcall(function() v:Destroy() end)
        end
    end

    -- Force le serveur à lâcher
    pcall(function()
        LP:SetAttribute("RagdollEndTime", workspace:GetServerTimeNow() - 10)
    end)

    -- Réinitialisation physique
    root.Anchored = false
    root.CanCollide = true
    root.AssemblyLinearVelocity = Vector3.zero
    root.AssemblyAngularVelocity = Vector3.zero

    -- Petit saut pour casser l'ancrage
    local currentPos = root.Position
    root.CFrame = CFrame.new(currentPos + Vector3.new(0, 0.5, 0)) * CFrame.Angles(0, math.rad(root.Orientation.Y), 0)

    -- Force les états Humanoid
    if hum.Health > 0 then
        hum:ChangeState(Enum.HumanoidStateType.GettingUp)
        task.wait(0.02)
        hum:ChangeState(Enum.HumanoidStateType.Running)
        task.wait(0.02)
        hum:ChangeState(Enum.HumanoidStateType.Running)
    end

    -- Réactive le contrôle
    pcall(function()
        local pm = LP.PlayerScripts:FindFirstChild("PlayerModule")
        if pm then
            local cm = pm:FindFirstChild("ControlModule")
            if cm then
                require(cm):Enable()
            end
        end
    end)

    -- Dernier recours : Blossom Reset si le ragdoll persiste
    task.delay(0.15, function()
        if antiRagdollIsRagdolled() then
            pcall(cursedInstaReset)
        end
    end)
end

local function startAntiRagdoll()
    if antiRagdollConn then return end
    
    if not antiRagdollCacheCharacter() then
        task.wait(0.5)
        if not antiRagdollCacheCharacter() then
            warn("[Henika Hub] Anti-Ragdoll: Could not cache character")
            return
        end
    end

    antiRagdollConn = RunService.Heartbeat:Connect(function()
        if not antiRagdollEnabled then return end
        if not antiRagdollCached.humanoid or not antiRagdollCached.humanoid.Parent then
            if not antiRagdollCacheCharacter() then
                return
            end
        end

        if antiRagdollIsRagdolled() then
            antiRagdollForceExit()
            task.wait(0.03)
            if antiRagdollIsRagdolled() then
                antiRagdollForceExit()
            end
        end
    end)
    
    print("[Henika Hub] Anti-Ragdoll ULTRA BOOSTED ACTIVATED")
end

local function stopAntiRagdoll()
    if antiRagdollConn then
        antiRagdollConn:Disconnect()
        antiRagdollConn = nil
    end
    antiRagdollCached = {}
    antiRagdollEnabled = false
    print("[Henika Hub] Anti-Ragdoll STOPPED")
end

LP.CharacterAdded:Connect(function(char)
    task.wait(0.5)
    if antiRagdollEnabled then
        antiRagdollCacheCharacter()
        if not antiRagdollConn then
            startAntiRagdoll()
        end
    end
end)

-- ============================================================
--  HENIKA HUB — GUI (APHEX-style tabbed layout, purple theme)
-- ============================================================
-- Save default Lighting for Dark Mode restore
local defBrightness   = Lighting.Brightness
local defClockTime    = Lighting.ClockTime
local defOutdoorAmbient = Lighting.OutdoorAmbient
local defExposureComp = Lighting.ExposureCompensation

local function buildGui()
        -- Purple theme
        local BG      = Color3.fromRGB(14, 6, 28)
        local BG2     = Color3.fromRGB(22, 10, 42)
        local TABBAR  = Color3.fromRGB(18, 8, 36)
        local TABSEL  = Color3.fromRGB(30, 14, 58)
        local TABHOV  = Color3.fromRGB(25, 12, 48)
        local CARD    = Color3.fromRGB(28, 12, 52)
        local HOV     = Color3.fromRGB(42, 18, 72)
        local PUR     = Color3.fromRGB(148, 68, 255)
        local PURDIM  = Color3.fromRGB(90, 38, 170)
        local STROKE  = Color3.fromRGB(80, 30, 140)
        local W       = Color3.fromRGB(235, 235, 235)
        local DIM     = Color3.fromRGB(120, 90, 160)
        local INP     = Color3.fromRGB(20, 9, 40)
        local OFF     = Color3.fromRGB(36, 16, 64)

        -- Dynamic accent system
        local uiLocked = false
        local uiScale  = 1
        local BASE_W, BASE_H = 380, 480
        local accentRefs  = {}
        local dimRefs     = {}
        local strokeRefs  = {}
        local tabButtons  = {}
        local pillDots    = {}
        local sectHeader
        local function setAccent(c)
                local dimC = Color3.new(c.R*0.608, c.G*0.559, c.B*0.667)
                local stkC = Color3.new(c.R*0.372, c.G*0.324, c.B*0.392)
                PUR    = c
                PURDIM = dimC
                STROKE = stkC
                for _, r in ipairs(accentRefs)  do pcall(function() r[1][r[2]] = c    end) end
                for _, r in ipairs(dimRefs)     do pcall(function() r[1][r[2]] = dimC end) end
                for _, r in ipairs(strokeRefs)  do pcall(function() r[1][r[2]] = stkC end) end
                for _, tbtn in pairs(tabButtons) do
                        local bar = tbtn:FindFirstChild("_bar")
                        if bar then bar.BackgroundColor3 = c end
                        if bar and bar.Visible then
                                local lbl = tbtn:FindFirstChildOfClass("TextLabel")
                                if lbl then lbl.TextColor3 = c end
                        end
                end
                if sectHeader then sectHeader.TextColor3 = c end
                for _, dot in ipairs(pillDots) do
                        pcall(function()
                                if dot.Position.X.Scale > 0.5 then dot.BackgroundColor3 = c end
                        end)
                end
        end
        local function trackA(obj, prop)   table.insert(accentRefs,  {obj, prop}) end
        local function trackDim(obj, prop) table.insert(dimRefs,     {obj, prop}) end
        local function trackStk(obj, prop) table.insert(strokeRefs,  {obj, prop}) end

        -- Remove existing
        local old = game:GetService("CoreGui"):FindFirstChild("HenikaHub")
        if old then old:Destroy() end
        local pg = LP:FindFirstChild("PlayerGui")
        if pg then local o = pg:FindFirstChild("HenikaHub"); if o then o:Destroy() end end

        local gui = Instance.new("ScreenGui")
        gui.Name = "HenikaHub"
        gui.ResetOnSpawn = false
        gui.DisplayOrder = 10
        gui.IgnoreGuiInset = true
        pcall(function() if syn and syn.protect_gui then syn.protect_gui(gui) end end)
        if not pcall(function() gui.Parent = game:GetService("CoreGui") end) then
                gui.Parent = LP:WaitForChild("PlayerGui")
        end

        -- Main window: 380 wide, 480 tall
        local main = Instance.new("Frame", gui)
        main.Size = UDim2.new(0, 380, 0, 480)
        main.Position = UDim2.new(0, 20, 0, 20)
        main.BackgroundColor3 = BG
        main.BorderSizePixel = 0
        main.ClipsDescendants = true
        Instance.new("UICorner", main).CornerRadius = UDim.new(0, 12)
        local mainStroke = Instance.new("UIStroke", main)
        mainStroke.Color = STROKE
        mainStroke.Thickness = 1.5
                trackStk(mainStroke, "Color")

        -- Drag
        local function drag(f)
                local dn, ds, sp, di = false
                f.InputBegan:Connect(function(i)
                        if uiLocked then return end
                        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
                                dn = true; ds = i.Position; sp = f.Position
                                i.Changed:Connect(function() if i.UserInputState == Enum.UserInputState.End then dn = false end end)
                        end
                end)
                f.InputChanged:Connect(function(i)
                        if i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch then di = i end
                end)
                UIS.InputChanged:Connect(function(i)
                        if i == di and dn then
                                local nX = sp.X.Offset + (i.Position.X - ds.X)
                                local nY = sp.Y.Offset + (i.Position.Y - ds.Y)
                                f.Position = UDim2.new(sp.X.Scale, nX, sp.Y.Scale, nY)
                        end
                end)
        end
        drag(main)

        -- ── Header ──────────────────────────────────────────────
        local hdr = Instance.new("Frame", main)
        hdr.Size = UDim2.new(1, 0, 0, 44)
        hdr.BackgroundColor3 = BG2
        hdr.BorderSizePixel = 0
        Instance.new("UICorner", hdr).CornerRadius = UDim.new(0, 12)
        local hdrBot = Instance.new("Frame", hdr)
        hdrBot.Size = UDim2.new(1, 0, 0, 12)
        hdrBot.Position = UDim2.new(0, 0, 1, -12)
        hdrBot.BackgroundColor3 = BG2
        hdrBot.BorderSizePixel = 0

        -- purple accent bar below title text
        local accentBar = Instance.new("Frame", hdr)
        accentBar.Size = UDim2.new(0, 3, 0, 22)
        accentBar.Position = UDim2.new(0, 12, 0.5, -11)
        accentBar.BackgroundColor3 = PUR
        accentBar.BorderSizePixel = 0
        Instance.new("UICorner", accentBar).CornerRadius = UDim.new(1, 0)
        trackA(accentBar, "BackgroundColor3")

        local ttl = Instance.new("TextLabel", hdr)
        ttl.Size = UDim2.new(1, -70, 0, 20)
        ttl.Position = UDim2.new(0, 20, 0, 6)
        ttl.BackgroundTransparency = 1
        ttl.Text = "HENIKA HUB"
        ttl.TextColor3 = PUR
        ttl.Font = Enum.Font.GothamBlack
        ttl.TextSize = 14
        ttl.TextXAlignment = Enum.TextXAlignment.Left
        trackA(ttl, "TextColor3")

        local subtitle = Instance.new("TextLabel", hdr)
        subtitle.Size = UDim2.new(1, -70, 0, 13)
        subtitle.Position = UDim2.new(0, 20, 0, 26)
        subtitle.BackgroundTransparency = 1
        subtitle.Text = "brainrot duel steal 🧠"
        subtitle.TextColor3 = PURDIM
        subtitle.Font = Enum.Font.Gotham
        subtitle.TextSize = 9
        subtitle.TextXAlignment = Enum.TextXAlignment.Left
                trackDim(subtitle, "TextColor3")

        local closeBtn = Instance.new("TextButton", hdr)
        closeBtn.Size = UDim2.new(0, 26, 0, 26)
        closeBtn.Position = UDim2.new(1, -32, 0.5, -13)
        closeBtn.BackgroundColor3 = BG2
        closeBtn.BorderSizePixel = 0
        closeBtn.Text = "-"
        closeBtn.TextColor3 = PURDIM
        closeBtn.Font = Enum.Font.GothamBold
        closeBtn.TextSize = 22
        Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 6)
                trackDim(closeBtn, "TextColor3")
        closeBtn.MouseEnter:Connect(function() TS:Create(closeBtn, TweenInfo.new(0.1), {BackgroundColor3 = CARD, TextColor3 = PUR}):Play() end)
        closeBtn.MouseLeave:Connect(function() TS:Create(closeBtn, TweenInfo.new(0.1), {BackgroundColor3 = BG2, TextColor3 = PURDIM}):Play() end)

        -- Mini restore button
        local miniBtn = Instance.new("TextButton", gui)
        miniBtn.Size = UDim2.new(0, 124, 0, 28)
        miniBtn.Position = UDim2.new(0, 26, 0, 26)
        miniBtn.BackgroundColor3 = BG2
        miniBtn.BorderSizePixel = 0
        miniBtn.Text = "HENIKA HUB"
        miniBtn.TextColor3 = PUR
        miniBtn.Font = Enum.Font.GothamBold
        miniBtn.TextSize = 11
        miniBtn.ZIndex = 20
        miniBtn.Visible = false
        Instance.new("UICorner", miniBtn).CornerRadius = UDim.new(0, 8)
        trackA(miniBtn, "TextColor3")
        local miniStk = Instance.new("UIStroke", miniBtn)
        miniStk.Color = STROKE
        miniStk.Thickness = 1.2
        drag(miniBtn)
        miniBtn.MouseEnter:Connect(function() TS:Create(miniBtn, TweenInfo.new(0.1), {BackgroundColor3 = HOV}):Play() end)
        miniBtn.MouseLeave:Connect(function() TS:Create(miniBtn, TweenInfo.new(0.1), {BackgroundColor3 = BG2}):Play() end)

        local function showGui() main.Visible = true; miniBtn.Visible = false end
        local function hideGui() main.Visible = false; miniBtn.Visible = true end
        closeBtn.MouseButton1Click:Connect(hideGui)
        miniBtn.MouseButton1Click:Connect(showGui)

        -- ── Body (below header) ──────────────────────────────────
        local body = Instance.new("Frame", main)
        body.Size = UDim2.new(1, 0, 1, -44)
        body.Position = UDim2.new(0, 0, 0, 44)
        body.BackgroundTransparency = 1
        body.BorderSizePixel = 0
        body.ClipsDescendants = false

        -- Left tab sidebar
        local tabBar = Instance.new("Frame", body)
        tabBar.Size = UDim2.new(0, 94, 1, 0)
        tabBar.BackgroundColor3 = TABBAR
        tabBar.BorderSizePixel = 0
        local tbLL = Instance.new("UIListLayout", tabBar)
        tbLL.SortOrder = Enum.SortOrder.LayoutOrder
        tbLL.Padding = UDim.new(0, 0)
        local tbPad = Instance.new("UIPadding", tabBar)
        tbPad.PaddingTop = UDim.new(0, 6)
        tbPad.PaddingBottom = UDim.new(0, 6)

        -- Divider
        local divider = Instance.new("Frame", body)
        divider.Size = UDim2.new(0, 1, 1, 0)
        divider.Position = UDim2.new(0, 94, 0, 0)
        divider.BackgroundColor3 = STROKE
        divider.BorderSizePixel = 0

        -- Right content panel
        local contentPanel = Instance.new("Frame", body)
        contentPanel.Size = UDim2.new(1, -95, 1, 0)
        contentPanel.Position = UDim2.new(0, 95, 0, 0)
        contentPanel.BackgroundColor3 = BG
        contentPanel.BorderSizePixel = 0
        contentPanel.ClipsDescendants = true

        -- Watermark image (behind everything)
        local bgImg = Instance.new("ImageLabel", contentPanel)
        bgImg.Size = UDim2.new(1, 0, 1, 0)
        bgImg.Position = UDim2.new(0, 0, 0, 0)
        bgImg.BackgroundTransparency = 1
        bgImg.Image = "rbxassetid://70498694199314"
        bgImg.ImageTransparency = 0.45
        bgImg.ScaleType = Enum.ScaleType.Fit
        bgImg.ZIndex = 1

        -- Section header label
        sectHeader = Instance.new("TextLabel", contentPanel)
        sectHeader.Size = UDim2.new(1, -10, 0, 20)
        sectHeader.Position = UDim2.new(0, 8, 0, 6)
        sectHeader.BackgroundTransparency = 1
        sectHeader.Text = "SPEED CONFIGURATION"
        sectHeader.TextColor3 = PUR
        sectHeader.Font = Enum.Font.GothamBlack
        sectHeader.TextSize = 9
        sectHeader.TextXAlignment = Enum.TextXAlignment.Left
        sectHeader.ZIndex = 3

        -- ── Tab & page helpers ───────────────────────────────────
        local tabPages = {}
        local tabDefs = {}

        local function isGamepadInput(inp)
                return inp and inp.UserInputType and inp.UserInputType.Name:match("^Gamepad") ~= nil
        end
        local GAMEPAD_KEYS = {
                [Enum.KeyCode.ButtonA]=true,[Enum.KeyCode.ButtonB]=true,[Enum.KeyCode.ButtonX]=true,[Enum.KeyCode.ButtonY]=true,
                [Enum.KeyCode.ButtonL1]=true,[Enum.KeyCode.ButtonR1]=true,[Enum.KeyCode.ButtonL2]=true,[Enum.KeyCode.ButtonR2]=true,
                [Enum.KeyCode.ButtonL3]=true,[Enum.KeyCode.ButtonR3]=true,[Enum.KeyCode.ButtonStart]=true,[Enum.KeyCode.ButtonSelect]=true,
                [Enum.KeyCode.DPadUp]=true,[Enum.KeyCode.DPadDown]=true,[Enum.KeyCode.DPadLeft]=true,[Enum.KeyCode.DPadRight]=true
        }
        local function isBindableInput(inp)
                if not inp or inp.KeyCode == Enum.KeyCode.Unknown then return false end
                if inp.UserInputType == Enum.UserInputType.Keyboard then return true end
                return isGamepadInput(inp) and GAMEPAD_KEYS[inp.KeyCode] == true
        end
        local function kbMatch(entry, kc) return kc and (kc == entry.kb or (entry.gp and kc == entry.gp)) end

        local function switchTab(name)
                for tname, pg2 in pairs(tabPages) do
                        pg2.Visible = (tname == name)
                end
                for tname, tbtn in pairs(tabButtons) do
                        local isActive = (tname == name)
                        tbtn.BackgroundColor3 = isActive and TABSEL or TABBAR
                        local lbl = tbtn:FindFirstChildOfClass("TextLabel")
                        if lbl then lbl.TextColor3 = isActive and PUR or DIM end
                        local bar = tbtn:FindFirstChild("_bar")
                        if bar then bar.Visible = isActive end
                end
                for _, td in ipairs(tabDefs) do
                        if td.name == name then sectHeader.Text = td.header; break end
                end
        end

        local tabOrder = 0
        local curPage = nil
        local curLO = 0
        local function PLO() curLO = curLO + 1; return curLO end

        local function addTab(name, header)
                tabOrder = tabOrder + 1
                -- Tab button
                local tbtn = Instance.new("TextButton", tabBar)
                tbtn.Size = UDim2.new(1, 0, 0, 38)
                tbtn.BackgroundColor3 = TABBAR
                tbtn.BorderSizePixel = 0
                tbtn.Text = ""
                tbtn.LayoutOrder = tabOrder

                local bar = Instance.new("Frame", tbtn)
                bar.Name = "_bar"
                bar.Size = UDim2.new(0, 3, 0.55, 0)
                bar.Position = UDim2.new(0, 0, 0.225, 0)
                bar.BackgroundColor3 = PUR
                bar.BorderSizePixel = 0
                bar.Visible = false
                Instance.new("UICorner", bar).CornerRadius = UDim.new(1, 0)

                local lbl = Instance.new("TextLabel", tbtn)
                lbl.Size = UDim2.new(1, -10, 1, 0)
                lbl.Position = UDim2.new(0, 10, 0, 0)
                lbl.BackgroundTransparency = 1
                lbl.Text = name
                lbl.TextColor3 = DIM
                lbl.Font = Enum.Font.GothamBold
                lbl.TextSize = 11
                lbl.TextXAlignment = Enum.TextXAlignment.Left
                lbl.TextWrapped = true

                tabButtons[name] = tbtn

                -- Page scroll frame
                local pg2 = Instance.new("ScrollingFrame", contentPanel)
                pg2.Size = UDim2.new(1, 0, 1, -30)
                pg2.Position = UDim2.new(0, 0, 0, 30)
                pg2.BackgroundTransparency = 1
                pg2.BorderSizePixel = 0
                pg2.ClipsDescendants = true
                pg2.ScrollBarThickness = 3
                pg2.ScrollBarImageColor3 = STROKE
                pg2.ScrollBarImageTransparency = 0
                pg2.CanvasSize = UDim2.new(0, 0, 0, 0)
                pg2.AutomaticCanvasSize = Enum.AutomaticSize.Y
                pg2.ZIndex = 2
                pg2.Visible = false
                local pLL = Instance.new("UIListLayout", pg2)
                pLL.SortOrder = Enum.SortOrder.LayoutOrder
                pLL.Padding = UDim.new(0, 2)
                local pPad = Instance.new("UIPadding", pg2)
                pPad.PaddingLeft = UDim.new(0, 5)
                pPad.PaddingRight = UDim.new(0, 7)
                pPad.PaddingTop = UDim.new(0, 4)
                pPad.PaddingBottom = UDim.new(0, 8)

                tabPages[name] = pg2
                table.insert(tabDefs, {name = name, header = header, page = pg2})
                curPage = pg2
                curLO = 0

                tbtn.MouseButton1Click:Connect(function() switchTab(name) end)
                tbtn.MouseEnter:Connect(function()
                        if not (tabButtons[name] and tabButtons[name]:FindFirstChild("_bar") and tabButtons[name]:FindFirstChild("_bar").Visible) then
                                TS:Create(tbtn, TweenInfo.new(0.08), {BackgroundColor3 = TABHOV}):Play()
                        end
                end)
                tbtn.MouseLeave:Connect(function()
                        local b = tbtn:FindFirstChild("_bar")
                        if b and b.Visible then return end
                        TS:Create(tbtn, TweenInfo.new(0.08), {BackgroundColor3 = TABBAR}):Play()
                end)

                return pg2
        end

        -- Per-page widget builders (use curPage / PLO)
        local function animPill(pill, dot, on)
                local dimAccent = Color3.new(PUR.R*0.38, PUR.G*0.25, PUR.B*0.48)
                TS:Create(pill, TweenInfo.new(0.18, Enum.EasingStyle.Quad), {BackgroundColor3 = on and dimAccent or OFF}):Play()
                TS:Create(dot, TweenInfo.new(0.18, Enum.EasingStyle.Back), {
                        Position = on and UDim2.new(1, -16, 0.5, -6.5) or UDim2.new(0, 3, 0.5, -6.5),
                        BackgroundColor3 = on and PUR or DIM
                }):Play()
        end

        local function pRow(h)
                local f = Instance.new("Frame", curPage)
                f.Size = UDim2.new(1, 0, 0, h or 32)
                f.BackgroundColor3 = CARD
                f.BorderSizePixel = 0
                f.LayoutOrder = PLO()
                f.ZIndex = 3
                Instance.new("UICorner", f).CornerRadius = UDim.new(0, 6)
                local stroke = Instance.new("UIStroke", f)
                stroke.Color = Color3.fromRGB(32, 18, 58)
                stroke.Thickness = 1
                f.MouseEnter:Connect(function() TS:Create(f, TweenInfo.new(0.08), {BackgroundColor3 = HOV}):Play() end)
                f.MouseLeave:Connect(function() TS:Create(f, TweenInfo.new(0.08), {BackgroundColor3 = CARD}):Play() end)
                return f
        end

        local function pLbl(row, txt)
                local l = Instance.new("TextLabel", row)
                l.Size = UDim2.new(0.6, 0, 1, 0)
                l.Position = UDim2.new(0, 8, 0, 0)
                l.BackgroundTransparency = 1
                l.Text = txt
                l.TextColor3 = W
                l.Font = Enum.Font.GothamBold
                l.TextSize = 11
                l.TextXAlignment = Enum.TextXAlignment.Left
                l.ZIndex = 4
        end

        local function pPill(row, offset)
                local pill = Instance.new("Frame", row)
                pill.Size = UDim2.new(0, 36, 0, 19)
                pill.Position = UDim2.new(1, -(offset or 42), 0.5, -9.5)
                pill.BackgroundColor3 = OFF
                pill.BorderSizePixel = 0
                pill.ZIndex = 5
                Instance.new("UICorner", pill).CornerRadius = UDim.new(1, 0)
                local dot = Instance.new("Frame", pill)
                dot.Size = UDim2.new(0, 13, 0, 13)
                dot.Position = UDim2.new(0, 3, 0.5, -6.5)
                dot.BackgroundColor3 = DIM
                dot.BorderSizePixel = 0
                dot.ZIndex = 6
                Instance.new("UICorner", dot).CornerRadius = UDim.new(1, 0)
                        table.insert(pillDots, dot)
                return pill, dot
        end

        local function pToggle(txt, cb)
                local row = pRow(32); pLbl(row, txt)
                local pill, dot = pPill(row, 42)
                local on = false
                local function sv(s) on = s; animPill(pill, dot, s) end
                local clk = Instance.new("TextButton", pill)
                clk.Size = UDim2.new(1, 0, 1, 0)
                clk.BackgroundTransparency = 1
                clk.Text = ""
                clk.ZIndex = 7
                clk.Activated:Connect(function() on = not on; sv(on); cb(on) end)
                pill.ZIndex = 5; dot.ZIndex = 6
                return sv
        end

        local function pBox(parent, default, w, xOff, cb)
                local tb = Instance.new("TextBox", parent)
                tb.Size = UDim2.new(0, w or 50, 0, 22)
                tb.Position = UDim2.new(1, -(xOff or 56), 0.5, -11)
                tb.BackgroundColor3 = INP
                tb.BorderSizePixel = 0
                tb.Text = tostring(default)
                tb.TextColor3 = W
                tb.Font = Enum.Font.GothamBold
                tb.TextSize = 11
                tb.ClearTextOnFocus = false
                tb.ZIndex = 7
                Instance.new("UICorner", tb).CornerRadius = UDim.new(0, 5)
                local bs = Instance.new("UIStroke", tb)
                bs.Color = Color3.fromRGB(40, 20, 70)
                bs.Thickness = 1
                tb.Focused:Connect(function() TS:Create(bs, TweenInfo.new(0.12), {Color = PURDIM}):Play() end)
                tb.FocusLost:Connect(function()
                        TS:Create(bs, TweenInfo.new(0.12), {Color = Color3.fromRGB(40, 20, 70)}):Play()
                        if cb then local n = tonumber(tb.Text); if n then cb(n) else tb.Text = tostring(default) end end
                end)
                return tb
        end

        local function pKB(parent, kbEntry, cb)
                local btn = Instance.new("TextButton", parent)
                btn.Size = UDim2.new(0, 46, 0, 22)
                btn.Position = UDim2.new(1, -50, 0.5, -11)
                btn.BackgroundColor3 = INP
                btn.BorderSizePixel = 0
                local function getLabel() return (kbEntry.gp and kbEntry.gp.Name) or (kbEntry.kb and kbEntry.kb.Name) or "None" end
                btn.Text = getLabel()
                btn.TextColor3 = W
                btn.Font = Enum.Font.GothamBold
                btn.TextSize = 9
                btn.ZIndex = 7
                Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 5)
                local li = false; local lc; local pv = btn.Text; local listenStart = 0
                btn.Activated:Connect(function()
                        if li then li = false; _anyKeyListening = false; if lc then lc:Disconnect(); lc = nil end; btn.Text = pv; btn.TextColor3 = W; return end
                        pv = btn.Text; li = true; _anyKeyListening = true; listenStart = tick(); btn.Text = "..."; btn.TextColor3 = W
                        lc = UIS.InputBegan:Connect(function(inp)
                                if not li then return end
                                if inp.KeyCode == Enum.KeyCode.Escape then li = false; _anyKeyListening = false; if lc then lc:Disconnect(); lc = nil end; btn.Text = pv; btn.TextColor3 = W; return end
                                local isGp = isGamepadInput(inp)
                                if isGp and tick() - listenStart < 0.15 then return end
                                if not isBindableInput(inp) then return end
                                btn.Text = inp.KeyCode.Name; pv = inp.KeyCode.Name; btn.TextColor3 = W
                                li = false; _anyKeyListening = false; if lc then lc:Disconnect(); lc = nil end
                                if cb then cb(inp.KeyCode, isGp) end
                        end)
                end)
                return btn
        end

        local function pToggleKB(txt, kbEntry, onToggle, onKB)
                local row = pRow(32); pLbl(row, txt)
                if kbEntry then pKB(row, kbEntry, function(k, isGp)
                        if isGp then kbEntry.gp = k; kbEntry.kb = nil else kbEntry.kb = k; kbEntry.gp = nil end
                        if onKB then onKB(k, isGp) end
                end) end
                local pill, dot = pPill(row, kbEntry and 102 or 42)
                local on = false
                local function sv(s) on = s; animPill(pill, dot, s) end
                local clk = Instance.new("TextButton", pill)
                clk.Size = UDim2.new(1, 0, 1, 0)
                clk.BackgroundTransparency = 1
                clk.Text = ""
                clk.ZIndex = 7
                clk.Activated:Connect(function() if _anyKeyListening then return end; on = not on; sv(on); if onToggle then onToggle(on) end end)
                pill.ZIndex = 5; dot.ZIndex = 6
                return sv
        end

        -- ── Progress bar (steal, floating) ───────────────────────
        local pbFrame = Instance.new("Frame", gui)
        pbFrame.Size = UDim2.new(0, 280, 0, 50)
        pbFrame.Position = UDim2.new(0.5, -140, 1, -66)
        pbFrame.BackgroundColor3 = BG2
        pbFrame.BorderSizePixel = 0
        pbFrame.Active = true
        pbFrame.ClipsDescendants = false
        Instance.new("UICorner", pbFrame).CornerRadius = UDim.new(0, 9)
        drag(pbFrame)
        progressPct = Instance.new("TextLabel", pbFrame)
        progressPct.Size = UDim2.new(0, 44, 0, 16)
        progressPct.Position = UDim2.new(0, 9, 0, 7)
        progressPct.BackgroundTransparency = 1
        progressPct.Text = "0%"
        progressPct.TextColor3 = W
        progressPct.Font = Enum.Font.GothamBold
        progressPct.TextSize = 11
        progressPct.TextXAlignment = Enum.TextXAlignment.Left
        progressRadLbl = Instance.new("TextLabel", pbFrame)
        progressRadLbl.Size = UDim2.new(0, 104, 0, 16)
        progressRadLbl.Position = UDim2.new(1, -112, 0, 7)
        progressRadLbl.BackgroundTransparency = 1
        progressRadLbl.Text = string.format("Radius: %.2g", Steal.StealRadius)
        progressRadLbl.TextColor3 = W
        progressRadLbl.Font = Enum.Font.GothamBold
        progressRadLbl.TextSize = 11
        progressRadLbl.TextXAlignment = Enum.TextXAlignment.Right
        local pbg = Instance.new("Frame", pbFrame)
        pbg.Size = UDim2.new(1, -18, 0, 11)
        pbg.Position = UDim2.new(0, 9, 0, 30)
        pbg.BackgroundColor3 = Color3.fromRGB(14, 9, 28)
        pbg.BorderSizePixel = 0
        Instance.new("UICorner", pbg).CornerRadius = UDim.new(1, 0)
        progressFill = Instance.new("Frame", pbg)
        progressFill.Size = UDim2.new(0, 0, 1, 0)
        progressFill.BackgroundColor3 = PUR
        progressFill.BorderSizePixel = 0
        Instance.new("UICorner", progressFill).CornerRadius = UDim.new(1, 0)
        trackA(progressFill, "BackgroundColor3")

        -- ═══════════════════════════════════════════════════════
        --  TAB: Speed
        -- ═══════════════════════════════════════════════════════
        addTab("Speed", "SPEED CONFIGURATION")
        do local row = pRow(32); pLbl(row, "Normal Speed");   normalBox       = pBox(row, NS,                50, 48, function(v) if v>0 and v<=500 then NS=v end; saveConfig() end) end
        do local row = pRow(32); pLbl(row, "Carry Speed");    carryBox        = pBox(row, CS,                50, 48, function(v) if v>0 and v<=500 then CS=v end; saveConfig() end) end
        do local row = pRow(32); pLbl(row, "Lagger Speed");   laggerBox       = pBox(row, LAGGER_SPEED,      50, 48, function(v) if v>0 and v<=500 then LAGGER_SPEED=v end; saveConfig() end) end
        do local row = pRow(32); pLbl(row, "Lagger Carry");   laggerCarryBox  = pBox(row, LAGGER_CARRY_SPEED,50, 48, function(v) if v>0 and v<=500 then LAGGER_CARRY_SPEED=v end; saveConfig() end) end
        do
                local row = pRow(32); pLbl(row, "Mode")
                modeValLbl = Instance.new("TextLabel", row)
                modeValLbl.Size = UDim2.new(0, 90, 1, 0)
                modeValLbl.Position = UDim2.new(1, -94, 0, 0)
                modeValLbl.BackgroundTransparency = 1
                modeValLbl.Text = "Normal"
                modeValLbl.TextColor3 = PUR
                        trackA(modeValLbl, "TextColor3")
                modeValLbl.Font = Enum.Font.GothamBlack
                modeValLbl.TextSize = 11
                modeValLbl.TextXAlignment = Enum.TextXAlignment.Right
                modeValLbl.ZIndex = 4
                local clk = Instance.new("TextButton", row)
                clk.Size = UDim2.new(1, 0, 1, 0)
                clk.BackgroundTransparency = 1
                clk.Text = ""
                clk.ZIndex = 5
                clk.Activated:Connect(function() if _anyKeyListening then return end; toggleCarryMode(); saveConfig() end)
        end
        do local row = pRow(32); pLbl(row, "Speed Key");  pKB(row, KB.SpeedToggle,  function(k,isGp) if isGp then KB.SpeedToggle.gp=k;KB.SpeedToggle.kb=nil else KB.SpeedToggle.kb=k;KB.SpeedToggle.gp=nil end; saveConfig() end) end
        do local row = pRow(32); pLbl(row, "Lagger Key"); pKB(row, KB.LaggerToggle, function(k,isGp) if isGp then KB.LaggerToggle.gp=k;KB.LaggerToggle.kb=nil else KB.LaggerToggle.kb=k;KB.LaggerToggle.gp=nil end; saveConfig() end) end

        -- ═══════════════════════════════════════════════════════
        --  TAB: Combat
        -- ═══════════════════════════════════════════════════════
        addTab("Combat", "COMBAT CONFIGURATION")
        do
                local abRow = pRow(32); pLbl(abRow, "Auto Bat")
                pKB(abRow, KB.AutoBat, function(k,isGp) if isGp then KB.AutoBat.gp=k;KB.AutoBat.kb=nil else KB.AutoBat.kb=k;KB.AutoBat.gp=nil end; saveConfig() end)
                local abPill, abDot = pPill(abRow, 102)
                abPill.ZIndex = 5; abDot.ZIndex = 6
                local abOn = false
                local function svAutoBat(s) abOn = s; animPill(abPill, abDot, s) end
                autoBatSetVisual = svAutoBat
                local abClk = Instance.new("TextButton", abPill)
                abClk.Size = UDim2.new(1, 0, 1, 0)
                abClk.BackgroundTransparency = 1
                abClk.Text = ""
                abClk.ZIndex = 7
                abClk.Activated:Connect(function()
                        if _anyKeyListening then return end
                        abOn = not abOn; svAutoBat(abOn)
                        if abOn then queueAutoBatStart() else autoBatEnabled = false; disableAutoBat() end
                        saveConfig()
                end)
        end
        setAutoSwingVisual = pToggle("Auto Swing", function(on) autoSwingEnabled = on; saveConfig() end)
        if setAutoSwingVisual then setAutoSwingVisual(autoSwingEnabled) end
        setBatCounterVisual = pToggle("Bat Counter", function(on)
                batCounterEnabled = on
                if on then startBatCounter() else stopBatCounter() end
                saveConfig()
        end)

        -- ═══════════════════════════════════════════════════════
        --  TAB: Movement
        -- ═══════════════════════════════════════════════════════
        addTab("Movement", "MOVEMENT CONFIGURATION")
        do
                local sv = pToggleKB("Auto Left", KB.AutoLeft,
                        function(on) autoLeftEnabled = on; if on then queueAutoLeftStart() else stopAutoLeft() end end,
                        function(k,isGp) if isGp then KB.AutoLeft.gp=k;KB.AutoLeft.kb=nil else KB.AutoLeft.kb=k;KB.AutoLeft.gp=nil end; saveConfig() end)
                autoLeftSetVisual = sv
        end
        do
                local sv = pToggleKB("Auto Right", KB.AutoRight,
                        function(on) autoRightEnabled = on; if on then queueAutoRightStart() else stopAutoRight() end end,
                        function(k,isGp) if isGp then KB.AutoRight.gp=k;KB.AutoRight.kb=nil else KB.AutoRight.kb=k;KB.AutoRight.gp=nil end; saveConfig() end)
                autoRightSetVisual = sv
        end
        do
                local row = pRow(32); pLbl(row, "Drop Brainrot")
                pKB(row, KB.DropBrainrot, function(k,isGp) if isGp then KB.DropBrainrot.gp=k;KB.DropBrainrot.kb=nil else KB.DropBrainrot.kb=k;KB.DropBrainrot.gp=nil end; saveConfig() end)
                local clk = Instance.new("TextButton", row); clk.Size = UDim2.new(0.6,0,1,0); clk.BackgroundTransparency = 1; clk.Text = ""; clk.ZIndex = 5
                clk.Activated:Connect(function() runDrop() end)
        end
        do
                local row = pRow(32); pLbl(row, "TP Down")
                pKB(row, KB.TPFloor, function(k,isGp) if isGp then KB.TPFloor.gp=k;KB.TPFloor.kb=nil else KB.TPFloor.kb=k;KB.TPFloor.gp=nil end; saveConfig() end)
                local clk = Instance.new("TextButton", row); clk.Size = UDim2.new(0.6,0,1,0); clk.BackgroundTransparency = 1; clk.Text = ""; clk.ZIndex = 5
                clk.Activated:Connect(function() runTPFloor() end)
        end
        setAutoTPVisual = pToggle("Auto TP", function(on) autoTPEnabled = on; if on then startAutoTP() else stopAutoTP() end; saveConfig() end)
        do
                local row = pRow(32); pLbl(row, "Auto TP Height")
                autoTPHeightBox = pBox(row, autoTPHeight, 50, 56, function(v)
                        if v >= 0 and v <= 500 then autoTPHeight = v else autoTPHeightBox.Text = tostring(autoTPHeight) end; saveConfig()
                end)
        end

        -- ═══════════════════════════════════════════════════════
        --  TAB: Steal
        -- ═══════════════════════════════════════════════════════
        addTab("Steal", "STEAL CONFIGURATION")
        do
                local row = pRow(32); pLbl(row, "Radius")
                radInput = pBox(row, Steal.StealRadius, 50, 56, function(v)
                        if v >= 0.5 and v <= 300 then Steal.StealRadius = v; if progressRadLbl then progressRadLbl.Text = string.format("Radius: %.2g", Steal.StealRadius) end end; saveConfig()
                end)
        end
        do
                local stealRow = pRow(32); pLbl(stealRow, "Auto Steal")
                local pill, dot = pPill(stealRow, 42)
                local on = false
                local function sv(s) on = s; animPill(pill, dot, s) end
                setInstaGrab = sv
                local clk = Instance.new("TextButton", pill); clk.Size = UDim2.new(1,0,1,0); clk.BackgroundTransparency = 1; clk.Text = ""; clk.ZIndex = 7
                clk.Activated:Connect(function()
                        on = not on; sv(on); Steal.AutoStealEnabled = on
                        if on then if not pcall(startAutoSteal) then Steal.AutoStealEnabled = false; sv(false) end else stopAutoSteal() end
                        saveConfig()
                end)
                pill.ZIndex = 5; dot.ZIndex = 6
        end

        -- ═══════════════════════════════════════════════════════
        --  TAB: Misc
        -- ═══════════════════════════════════════════════════════
        addTab("Misc", "MISC CONFIGURATION")
        do
                local row = pRow(32); pLbl(row, "Instant Reset")
                pKB(row, KB.InstaReset, function(k,isGp) if isGp then KB.InstaReset.gp=k;KB.InstaReset.kb=nil else KB.InstaReset.kb=k;KB.InstaReset.gp=nil end; saveConfig() end)
                local clk = Instance.new("TextButton", row); clk.Size = UDim2.new(0.6,0,1,0); clk.BackgroundTransparency = 1; clk.Text = ""; clk.ZIndex = 5
                clk.Activated:Connect(function() cursedInstaReset() end)
        end
        setInfJumpVisual   = pToggle("Infinite Jump",    function(on) infJumpEnabled = on end)
        -- Inf Jump mode: MANUAL / HOLD
        do
                local modeRow = pRow(30)
                local function mkModeBtn(txt, xScale, xOff)
                        local b = Instance.new("TextButton", modeRow)
                        b.Size = UDim2.new(0.42, 0, 0.8, 0)
                        b.Position = UDim2.new(xScale, xOff, 0.1, 0)
                        b.BackgroundColor3 = INP; b.BorderSizePixel = 0
                        b.TextColor3 = DIM; b.Font = Enum.Font.GothamBold; b.TextSize = 10
                        b.AutoButtonColor = false; b.Text = txt; b.ZIndex = 7
                        Instance.new("UICorner", b).CornerRadius = UDim.new(0, 5)
                        local bs = Instance.new("UIStroke", b); bs.Color = STROKE; bs.Thickness = 1
                        return b, bs
                end
                local manualBtn, manualStk = mkModeBtn("MANUAL", 0.07, 0)
                local holdBtn,   holdStk   = mkModeBtn("HOLD",   0.54, 0)
                local function updJumpModeUI()
                        if infJumpMode == "manual" then
                                manualBtn.BackgroundColor3 = PUR; manualBtn.TextColor3 = Color3.fromRGB(255,255,255); manualStk.Color = PUR
                                holdBtn.BackgroundColor3 = INP;   holdBtn.TextColor3 = DIM;                           holdStk.Color = STROKE
                        else
                                manualBtn.BackgroundColor3 = INP; manualBtn.TextColor3 = DIM;   manualStk.Color = STROKE
                                holdBtn.BackgroundColor3 = PUR;   holdBtn.TextColor3 = Color3.fromRGB(255,255,255); holdStk.Color = PUR
                        end
                end
                updJumpModeUI()
                manualBtn.Activated:Connect(function() infJumpMode = "manual"; updJumpModeUI(); saveConfig() end)
                holdBtn.Activated:Connect(function()   infJumpMode = "hold";   updJumpModeUI(); saveConfig() end)
        end
        setAntiRagVisual   = pToggle("Anti Ragdoll",     function(on) antiRagdollEnabled = on; if on then startAntiRagdoll() else stopAntiRagdoll() end end)
        setMedusaVisual    = pToggle("Medusa Counter",   function(on) medusaCounterEnabled = on; if on then setupMedusa(LP.Character) else stopMedusaCounter() end; saveConfig() end)
        setUnwalkVisual    = pToggle("Unwalk",           function(on) unwalkEnabled = on; if on then startUnwalk() else stopUnwalk() end end)

        -- Bypass Panel button
        do
                local bpRow = pRow(32); pLbl(bpRow, "Bypasser")
                local openBtn = Instance.new("TextButton", bpRow)
                openBtn.Size = UDim2.new(0, 52, 0, 22)
                openBtn.Position = UDim2.new(1, -56, 0.5, -11)
                openBtn.BackgroundColor3 = INP; openBtn.BorderSizePixel = 0
                openBtn.Text = "OPEN"; openBtn.TextColor3 = PUR
                openBtn.Font = Enum.Font.GothamBold; openBtn.TextSize = 11; openBtn.ZIndex = 7
                Instance.new("UICorner", openBtn).CornerRadius = UDim.new(0, 5)
                local bpStk = Instance.new("UIStroke", openBtn); bpStk.Color = STROKE; bpStk.Thickness = 1
                trackA(openBtn, "TextColor3")
                openBtn.MouseEnter:Connect(function() TS:Create(openBtn, TweenInfo.new(0.08), {BackgroundColor3 = HOV}):Play() end)
                openBtn.MouseLeave:Connect(function() TS:Create(openBtn, TweenInfo.new(0.08), {BackgroundColor3 = INP}):Play() end)
                openBtn.Activated:Connect(function()
                        local panelName = "HenikaHubBypasser"
                        local existing = gui:FindFirstChild(panelName)
                        if existing then existing.Visible = not existing.Visible; return end
                        -- State
                        local bypassed = false
                        local bypassKeybind = Enum.KeyCode.F6
                        local bypassWaiting = false
                        local bypassPower = 300000
                        local bypassLagAmt = 0.12
                        local bypassConn = nil
                        local function bpApplyPower(v)
                                bypassPower = math.clamp(v, 10000, 500000)
                                bypassLagAmt = ((bypassPower - 10000) / 490000) * 0.2
                        end
                        bpApplyPower(bypassPower)
                        local function bpStartLag()
                                if bypassConn then bypassConn:Disconnect() end
                                bypassConn = RunService.RenderStepped:Connect(function()
                                        if not bypassed or bypassLagAmt <= 0 then return end
                                        local t = tick(); while tick()-t < bypassLagAmt do end
                                end)
                        end
                        local function bpStopLag()
                                bypassed = false
                                if bypassConn then bypassConn:Disconnect(); bypassConn = nil end
                        end
                        -- Panel frame
                        local bp = Instance.new("Frame", gui)
                        bp.Name = panelName; bp.Size = UDim2.new(0, 240, 0, 0)
                        bp.Position = UDim2.new(0.5, -120, 0.5, -160)
                        bp.BackgroundColor3 = BG; bp.BorderSizePixel = 0
                        bp.Active = true; bp.ZIndex = 100; bp.ClipsDescendants = true
                        Instance.new("UICorner", bp).CornerRadius = UDim.new(0, 12)
                        local bpOutline = Instance.new("UIStroke", bp)
                        bpOutline.Color = PUR; bpOutline.Thickness = 1.5
                        trackA(bpOutline, "Color")
                        drag(bp)
                        TS:Create(bp, TweenInfo.new(0.35, Enum.EasingStyle.Quint), {Size = UDim2.new(0, 240, 0, 290)}):Play()
                        -- Header
                        local bpHdr = Instance.new("Frame", bp)
                        bpHdr.Size = UDim2.new(1, 0, 0, 44); bpHdr.BackgroundColor3 = BG2; bpHdr.BorderSizePixel = 0; bpHdr.ZIndex = 101
                        Instance.new("UICorner", bpHdr).CornerRadius = UDim.new(0, 12)
                        local bpHdrFill = Instance.new("Frame", bpHdr)
                        bpHdrFill.Size = UDim2.new(1,0,0,12); bpHdrFill.Position = UDim2.new(0,0,1,-12)
                        bpHdrFill.BackgroundColor3 = BG2; bpHdrFill.BorderSizePixel = 0
                        local bpBar = Instance.new("Frame", bpHdr)
                        bpBar.Size = UDim2.new(0,3,0,22); bpBar.Position = UDim2.new(0,12,0.5,-11)
                        bpBar.BackgroundColor3 = PUR; bpBar.BorderSizePixel = 0; bpBar.ZIndex = 102
                        Instance.new("UICorner", bpBar).CornerRadius = UDim.new(1,0)
                        trackA(bpBar, "BackgroundColor3")
                        local bpTtl = Instance.new("TextLabel", bpHdr)
                        bpTtl.Size = UDim2.new(1,-44,0,20); bpTtl.Position = UDim2.new(0,20,0,5)
                        bpTtl.BackgroundTransparency = 1; bpTtl.Text = "HENIKA BYPASSER"
                        bpTtl.TextColor3 = PUR; bpTtl.Font = Enum.Font.GothamBlack; bpTtl.TextSize = 12
                        bpTtl.TextXAlignment = Enum.TextXAlignment.Left; bpTtl.ZIndex = 102
                        trackA(bpTtl, "TextColor3")
                        local bpClose = Instance.new("TextButton", bpHdr)
                        bpClose.Size = UDim2.new(0,24,0,24); bpClose.Position = UDim2.new(1,-30,0.5,-12)
                        bpClose.BackgroundColor3 = BG2; bpClose.BorderSizePixel = 0
                        bpClose.Text = "-"; bpClose.TextColor3 = PURDIM
                        bpClose.Font = Enum.Font.GothamBold; bpClose.TextSize = 18; bpClose.ZIndex = 102
                        Instance.new("UICorner", bpClose).CornerRadius = UDim.new(0,5)
                        bpClose.Activated:Connect(function() bp.Visible = false end)
                        -- Power input
                        local bpPwrLbl = Instance.new("TextLabel", bp)
                        bpPwrLbl.Size = UDim2.new(0.88,0,0,16); bpPwrLbl.Position = UDim2.new(0.06,0,0,54)
                        bpPwrLbl.BackgroundTransparency = 1; bpPwrLbl.Text = "SET POWER (10k – 500k):"
                        bpPwrLbl.TextColor3 = DIM; bpPwrLbl.Font = Enum.Font.GothamMedium
                        bpPwrLbl.TextSize = 10; bpPwrLbl.TextXAlignment = Enum.TextXAlignment.Left; bpPwrLbl.ZIndex = 101
                        local bpPwrBox = Instance.new("TextBox", bp)
                        bpPwrBox.Size = UDim2.new(0.88,0,0,34); bpPwrBox.Position = UDim2.new(0.06,0,0,73)
                        bpPwrBox.BackgroundColor3 = INP; bpPwrBox.Text = tostring(bypassPower)
                        bpPwrBox.TextColor3 = W; bpPwrBox.Font = Enum.Font.GothamBold
                        bpPwrBox.TextSize = 13; bpPwrBox.BorderSizePixel = 0
                        bpPwrBox.ClearTextOnFocus = false; bpPwrBox.ZIndex = 101
                        Instance.new("UICorner", bpPwrBox).CornerRadius = UDim.new(0,6)
                        local bpPwrStk = Instance.new("UIStroke", bpPwrBox); bpPwrStk.Color = STROKE; bpPwrStk.Thickness = 1
                        bpPwrBox.FocusLost:Connect(function()
                                local v = tonumber(bpPwrBox.Text)
                                if v then bpApplyPower(v) end
                                bpPwrBox.Text = tostring(bypassPower)
                        end)
                        -- Keybind button
                        local bpKbLbl = Instance.new("TextLabel", bp)
                        bpKbLbl.Size = UDim2.new(0.88,0,0,16); bpKbLbl.Position = UDim2.new(0.06,0,0,118)
                        bpKbLbl.BackgroundTransparency = 1; bpKbLbl.Text = "TOGGLE KEY:"
                        bpKbLbl.TextColor3 = DIM; bpKbLbl.Font = Enum.Font.GothamMedium
                        bpKbLbl.TextSize = 10; bpKbLbl.TextXAlignment = Enum.TextXAlignment.Left; bpKbLbl.ZIndex = 101
                        local bpKbBtn = Instance.new("TextButton", bp)
                        bpKbBtn.Size = UDim2.new(0.88,0,0,34); bpKbBtn.Position = UDim2.new(0.06,0,0,137)
                        bpKbBtn.BackgroundColor3 = INP; bpKbBtn.Text = "F6"
                        bpKbBtn.TextColor3 = PUR; bpKbBtn.Font = Enum.Font.GothamBold
                        bpKbBtn.TextSize = 13; bpKbBtn.BorderSizePixel = 0; bpKbBtn.AutoButtonColor = false; bpKbBtn.ZIndex = 101
                        Instance.new("UICorner", bpKbBtn).CornerRadius = UDim.new(0,6)
                        local bpKbStk = Instance.new("UIStroke", bpKbBtn); bpKbStk.Color = STROKE; bpKbStk.Thickness = 1
                        bpKbBtn.Activated:Connect(function()
                                bypassWaiting = true; bpKbBtn.Text = "..."; bpKbBtn.TextColor3 = Color3.fromRGB(255,100,100)
                        end)
                        -- Toggle button
                        local bpToggle = Instance.new("TextButton", bp)
                        bpToggle.Size = UDim2.new(0.88,0,0,44); bpToggle.Position = UDim2.new(0.06,0,0,208)
                        bpToggle.BackgroundColor3 = OFF; bpToggle.Text = "DEACTIVATED"
                        bpToggle.TextColor3 = W; bpToggle.Font = Enum.Font.GothamBlack
                        bpToggle.TextSize = 13; bpToggle.BorderSizePixel = 0; bpToggle.AutoButtonColor = false; bpToggle.ZIndex = 101
                        Instance.new("UICorner", bpToggle).CornerRadius = UDim.new(0,8)
                        local bpTglStk = Instance.new("UIStroke", bpToggle); bpTglStk.Color = STROKE; bpTglStk.Thickness = 1
                        local function bypassToggle()
                                if not bypassed then
                                        bypassed = true
                                        bpToggle.Text = "ACTIVATED"; bpToggle.BackgroundColor3 = PUR
                                        bpTglStk.Color = PUR; bpStartLag()
                                else
                                        bpStopLag()
                                        bpToggle.Text = "DEACTIVATED"; bpToggle.BackgroundColor3 = OFF
                                        bpTglStk.Color = STROKE
                                end
                        end
                        bpToggle.Activated:Connect(bypassToggle)
                        UIS.InputBegan:Connect(function(input, gpe)
                                if gpe then return end
                                if bypassWaiting then
                                        if input.UserInputType == Enum.UserInputType.Keyboard then
                                                bypassKeybind = input.KeyCode
                                                bpKbBtn.Text = input.KeyCode.Name:sub(1,4):upper()
                                                bpKbBtn.TextColor3 = PUR; bypassWaiting = false
                                        end
                                        return
                                end
                                if input.KeyCode == bypassKeybind then bypassToggle() end
                        end)
                        LP.CharacterAdded:Connect(function()
                                task.wait(1)
                                if bypassed then bpStopLag(); bypassed = true; bpStartLag() end
                        end)
                end)
        end

        -- ═══════════════════════════════════════════════════════
        --  TAB: Visual
        -- ═══════════════════════════════════════════════════════
        addTab("Visual", "VISUAL CONFIGURATION")
        pToggle("Dark Mode", function(on)
                if on then
                        local sky = Lighting:FindFirstChild("henikaDarkSky") or Instance.new("Sky")
                        sky.Name = "henikaDarkSky"
                        sky.SkyboxBk="rbxassetid://159454299"; sky.SkyboxDn="rbxassetid://159454296"
                        sky.SkyboxFt="rbxassetid://159454293"; sky.SkyboxLf="rbxassetid://159454286"
                        sky.SkyboxRt="rbxassetid://159454289"; sky.SkyboxUp="rbxassetid://159454291"
                        sky.Parent = Lighting
                        Lighting.Brightness=0; Lighting.ClockTime=0
                        Lighting.ExposureCompensation=-2
                        Lighting.OutdoorAmbient=Color3.fromRGB(0,0,0)
                else
                        local s=Lighting:FindFirstChild("henikaDarkSky"); if s then s:Destroy() end
                        Lighting.Brightness=defBrightness; Lighting.ClockTime=defClockTime
                        Lighting.ExposureCompensation=defExposureComp
                        Lighting.OutdoorAmbient=defOutdoorAmbient
                end
        end)
        setAntiLagVisual   = pToggle("Anti Lag",    function(on) if on then enableAntiLag()    else disableAntiLag()    end; saveConfig() end)
        setStretchRezVisual= pToggle("Stretch Rez", function(on) if on then enableStretchRez() else disableStretchRez() end; saveConfig() end)

        -- Mobile pad state
        local mobileLocked = false
        local btnScale = 1
        local allMobileBtns = {}
        local unmovableGridEnabled = false
        local unmovableGridFrame = nil

        local MOBILE_BTN_W  = 155
        local MOBILE_BTN_H  = 57
        local MOBILE_TAUNT_H = 28
        local MOBILE_GAP    = 8
        local MOBILE_RIGHT_MARGIN = 10
        local MOBILE_START_Y = 375

        local function defaultMobilePos(col, row, isTaunt)
                local w = math.floor(MOBILE_BTN_W * btnScale)
                local h = math.floor(MOBILE_BTN_H * btnScale)
                local g = MOBILE_GAP
                local rm = MOBILE_RIGHT_MARGIN
                local xOff = (col == 1)
                        and -(rm + w + g + w)
                        or  -(rm + w)
                local yOff = MOBILE_START_Y + (row - 1) * (h + g)
                return UDim2.new(1, xOff, 0, yOff)
        end

        local function saveMobileGridConfig()
                if not (writefile and isfile) then return end
                local data = {scale = btnScale, btns = {}}
                for i, entry in ipairs(allMobileBtns) do
                        if not entry.isGridFixed then
                                local p = entry.btn.Position
                                data.btns[i] = {
                                        xs = p.X.Scale, xo = p.X.Offset,
                                        ys = p.Y.Scale, yo = p.Y.Offset,
                                }
                        end
                end
                pcall(function()
                        writefile("henikaMobileGrid.json", HS:JSONEncode(data))
                end)
        end

        local function loadMobileGridConfig()
                if not (isfile and isfile("henikaMobileGrid.json")) then return end
                local ok, data = pcall(function()
                        return HS:JSONDecode(readfile("henikaMobileGrid.json"))
                end)
                if not ok or not data then return end
                if data.scale then
                        btnScale = data.scale
                end
                return data
        end

        local _savedMobileData = nil

        local function resizeAllMobileBtns_outer()
                local w = math.floor(MOBILE_BTN_W * btnScale)
                local h = math.floor(MOBILE_BTN_H * btnScale)
                local th = math.floor(MOBILE_TAUNT_H * btnScale)
                local g = MOBILE_GAP
                for _, entry in ipairs(allMobileBtns) do
                        if not entry.isGridFixed then
                                local sz = entry.isTaunt and UDim2.new(0, w*2+g, 0, th)
                                        or UDim2.new(0, w, 0, h)
                                entry.btn.Size = sz
                        end
                end
        end

        local function snapMobileBtnsToGrid()
                local w = math.floor(MOBILE_BTN_W * btnScale)
                local h = math.floor(MOBILE_BTN_H * btnScale)
                local g = MOBILE_GAP
                local rm = MOBILE_RIGHT_MARGIN
                for _, entry in ipairs(allMobileBtns) do
                        if not entry.isGridFixed then
                                local col = entry.gridCol or 1
                                local row = entry.gridRow or 1
                                entry.btn.Position = defaultMobilePos(col, row, entry.isTaunt)
                        end
                end
        end

        -- ═══════════════════════════════════════════════════════
        --  TAB: Config
        -- ═══════════════════════════════════════════════════════
        addTab("Config", "INTERFACE CONFIG")

        -- Hide UI keybind
        do local row = pRow(32); pLbl(row, "Hide UI"); pKB(row, KB.GuiHide, function(k,isGp) if isGp then KB.GuiHide.gp=k;KB.GuiHide.kb=nil else KB.GuiHide.kb=k;KB.GuiHide.gp=nil end; saveConfig() end) end

        -- Lock UI toggle
        do
                local row = pRow(32); pLbl(row, "Lock UI")
                local pill, dot = pPill(row, 42)
                local on = false
                local function sv(s) on = s; animPill(pill, dot, s); uiLocked = s end
                local clk = Instance.new("TextButton", pill)
                clk.Size = UDim2.new(1,0,1,0); clk.BackgroundTransparency = 1; clk.Text = ""; clk.ZIndex = 7
                clk.Activated:Connect(function() on = not on; sv(on) end)
                pill.ZIndex = 5; dot.ZIndex = 6
        end

        -- UI Size (+/-)
        do
                local row = pRow(32); pLbl(row, "UI Size")
                local function mkSizeBtn(txt, xOff)
                        local btn = Instance.new("TextButton", row)
                        btn.Size = UDim2.new(0, 28, 0, 22)
                        btn.Position = UDim2.new(1, xOff, 0.5, -11)
                        btn.BackgroundColor3 = INP; btn.BorderSizePixel = 0
                        btn.Text = txt; btn.TextColor3 = W
                        btn.Font = Enum.Font.GothamBold; btn.TextSize = 15; btn.ZIndex = 7
                        Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 5)
                        btn.MouseEnter:Connect(function() TS:Create(btn, TweenInfo.new(0.08), {BackgroundColor3 = HOV}):Play() end)
                        btn.MouseLeave:Connect(function() TS:Create(btn, TweenInfo.new(0.08), {BackgroundColor3 = INP}):Play() end)
                        return btn
                end
                local btnP = mkSizeBtn("+", -30)
                local btnM = mkSizeBtn("-", -60)
                btnP.Activated:Connect(function()
                        uiScale = math.min(1.5, uiScale + 0.1)
                        main.Size = UDim2.new(0, math.floor(BASE_W * uiScale), 0, math.floor(BASE_H * uiScale))
                end)
                btnM.Activated:Connect(function()
                        uiScale = math.max(0.6, uiScale - 0.1)
                        main.Size = UDim2.new(0, math.floor(BASE_W * uiScale), 0, math.floor(BASE_H * uiScale))
                end)
        end

        -- Theme Color palette
        do
                local labelRow = pRow(20); pLbl(labelRow, "Theme Color")
                local palRow = pRow(32)
                local PALETTE = {
                        Color3.fromRGB(138, 15,  45),   -- dark crimson
                        Color3.fromRGB(218, 55,  65),   -- red
                        Color3.fromRGB(232, 95,  55),   -- orange-red
                        Color3.fromRGB(238, 148, 55),   -- orange
                        Color3.fromRGB(244, 196, 88),   -- yellow-orange
                        Color3.fromRGB(248, 238, 148),  -- light yellow
                        Color3.fromRGB(210, 235, 142),  -- yellow-green
                        Color3.fromRGB(165, 218, 128),  -- light green
                        Color3.fromRGB(115, 198, 158),  -- mint
                        Color3.fromRGB(72,  178, 175),  -- teal
                        Color3.fromRGB(58,  128, 198),  -- sky blue
                        Color3.fromRGB(80,  98,  178),  -- blue
                        Color3.fromRGB(148, 68,  255),  -- purple (default)
                }
                local n = #PALETTE
                local swW = 18
                local gap = 2
                local startX = 5
                for i, col in ipairs(PALETTE) do
                        local sw = Instance.new("TextButton", palRow)
                        sw.Size = UDim2.new(0, swW, 0, 20)
                        sw.Position = UDim2.new(0, startX + (i-1)*(swW+gap), 0.5, -10)
                        sw.BackgroundColor3 = col
                        sw.BorderSizePixel = 0
                        sw.Text = ""
                        sw.ZIndex = 7
                        Instance.new("UICorner", sw).CornerRadius = UDim.new(0, 4)
                        local selRing = Instance.new("UIStroke", sw)
                        selRing.Color = W; selRing.Thickness = 0; selRing.ZIndex = 8
                        sw.Activated:Connect(function()
                                setAccent(col)
                                for _, child in ipairs(palRow:GetChildren()) do
                                        if child:IsA("TextButton") then
                                                local ring = child:FindFirstChildOfClass("UIStroke")
                                                if ring then ring.Thickness = (child == sw) and 1.5 or 0 end
                                        end
                                end
                        end)
                end
                local swatches = palRow:GetChildren()
                for _, child in ipairs(swatches) do
                        if child:IsA("TextButton") then
                                local ring = child:FindFirstChildOfClass("UIStroke")
                                if ring and child.BackgroundColor3 == Color3.fromRGB(148,68,255) then ring.Thickness = 1.5 end
                        end
                end
        end

        -- ── Mobile UI controls (below palette) ───────────────────

        -- Mobile UI size (+/-)
        do
                local row = pRow(32); pLbl(row, "Mobile UI")
                local function mkB(txt, xOff)
                        local b = Instance.new("TextButton", row)
                        b.Size = UDim2.new(0, 28, 0, 22)
                        b.Position = UDim2.new(1, xOff, 0.5, -11)
                        b.BackgroundColor3 = INP; b.BorderSizePixel = 0
                        b.Text = txt; b.TextColor3 = W
                        b.Font = Enum.Font.GothamBold; b.TextSize = 15; b.ZIndex = 7
                        Instance.new("UICorner", b).CornerRadius = UDim.new(0, 5)
                        b.MouseEnter:Connect(function() TS:Create(b, TweenInfo.new(0.08), {BackgroundColor3 = HOV}):Play() end)
                        b.MouseLeave:Connect(function() TS:Create(b, TweenInfo.new(0.08), {BackgroundColor3 = INP}):Play() end)
                        return b
                end
                mkB("+", -30).Activated:Connect(function()
                        btnScale = math.min(2.0, btnScale + 0.15); resizeAllMobileBtns_outer()
                end)
                mkB("-", -62).Activated:Connect(function()
                        btnScale = math.max(0.5, btnScale - 0.15); resizeAllMobileBtns_outer()
                end)
        end

        -- Lock Mobile UI pill toggle
        do
                local row = pRow(32); pLbl(row, "Lock Mobile UI")
                local pill, dot = pPill(row, 42)
                local on = false
                local function sv(s) on = s; animPill(pill, dot, s); mobileLocked = s end
                local clk = Instance.new("TextButton", pill)
                clk.Size = UDim2.new(1,0,1,0); clk.BackgroundTransparency = 1; clk.Text = ""; clk.ZIndex = 7
                clk.Activated:Connect(function() on = not on; sv(on) end)
                pill.ZIndex = 5; dot.ZIndex = 6
        end

        -- Mobile UI Shape selector
        do
                local labelRow = pRow(20); pLbl(labelRow, "Mobile UI Shape")
                local shapeRow = pRow(32)
                local SHAPES = {
                        {name="Circle",    r=UDim.new(0.5, 0)},
                        {name="Squircle",  r=UDim.new(0, 14)},
                        {name="Square",    r=UDim.new(0, 0)},
                        {name="Rect",      r=UDim.new(0, 6)},
                }
                local totalShapeBtns = #SHAPES
                local shapeW = 50
                local shapeGap = 4
                local shapeActiveIdx = 2
                local shapeBtns = {}
                local function applyShape(idx)
                        shapeActiveIdx = idx
                        local radius = SHAPES[idx].r
                        for _, entry in ipairs(allMobileBtns) do
                                local corner = entry.corner
                                if corner then corner.CornerRadius = radius end
                        end
                        for i, sb in ipairs(shapeBtns) do
                                sb.BackgroundColor3 = (i == idx) and PUR or INP
                        end
                end
                for i, shape in ipairs(SHAPES) do
                        local sb = Instance.new("TextButton", shapeRow)
                        sb.Size = UDim2.new(0, shapeW, 0, 22)
                        sb.Position = UDim2.new(0, (i-1)*(shapeW+shapeGap) + 4, 0.5, -11)
                        sb.BackgroundColor3 = (i == shapeActiveIdx) and PUR or INP
                        sb.BorderSizePixel = 0
                        sb.Text = shape.name
                        sb.TextColor3 = W
                        sb.Font = Enum.Font.GothamBold
                        sb.TextSize = 10
                        sb.ZIndex = 7
                        Instance.new("UICorner", sb).CornerRadius = UDim.new(0, 5)
                        sb.MouseEnter:Connect(function()
                                if i ~= shapeActiveIdx then TS:Create(sb, TweenInfo.new(0.08), {BackgroundColor3 = HOV}):Play() end
                        end)
                        sb.MouseLeave:Connect(function()
                                sb.BackgroundColor3 = (i == shapeActiveIdx) and PUR or INP
                        end)
                        sb.Activated:Connect(function() applyShape(i) end)
                        table.insert(shapeBtns, sb)
                end
        end

        -- Normal Grid (snap all buttons back to default grid layout)
        do
                local row = pRow(32); pLbl(row, "Normal Grid Mobile UI")
                local btn = Instance.new("TextButton", row)
                btn.Size = UDim2.new(0, 60, 0, 22)
                btn.Position = UDim2.new(1, -64, 0.5, -11)
                btn.BackgroundColor3 = INP; btn.BorderSizePixel = 0
                btn.Text = "RESET"
                btn.TextColor3 = W
                btn.Font = Enum.Font.GothamBold; btn.TextSize = 11; btn.ZIndex = 7
                Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 5)
                btn.MouseEnter:Connect(function() TS:Create(btn, TweenInfo.new(0.08), {BackgroundColor3 = HOV}):Play() end)
                btn.MouseLeave:Connect(function() TS:Create(btn, TweenInfo.new(0.08), {BackgroundColor3 = INP}):Play() end)
                btn.Activated:Connect(function() snapMobileBtnsToGrid() end)
        end

        -- Unmovable Grid Type toggle
        do
                local row = pRow(32); pLbl(row, "Unmovable Grid Type")
                local pill, dot = pPill(row, 42)
                local on = false
                local function sv(s)
                        on = s; animPill(pill, dot, s)
                        unmovableGridEnabled = s
                        if unmovableGridFrame then
                                unmovableGridFrame.Visible = s
                        end
                        for _, entry in ipairs(allMobileBtns) do
                                entry.btn.Visible = not s
                        end
                end
                local clk = Instance.new("TextButton", pill)
                clk.Size = UDim2.new(1,0,1,0); clk.BackgroundTransparency = 1; clk.Text = ""; clk.ZIndex = 7
                clk.Activated:Connect(function() on = not on; sv(on) end)
                pill.ZIndex = 5; dot.ZIndex = 6
        end

        -- Save Grid Config button
        do
                local row = pRow(32); pLbl(row, "Save Grid Config")
                local btn = Instance.new("TextButton", row)
                btn.Size = UDim2.new(0, 60, 0, 22)
                btn.Position = UDim2.new(1, -64, 0.5, -11)
                btn.BackgroundColor3 = INP; btn.BorderSizePixel = 0
                btn.Text = "SAVE"
                btn.TextColor3 = W
                btn.Font = Enum.Font.GothamBold; btn.TextSize = 11; btn.ZIndex = 7
                Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 5)
                btn.MouseEnter:Connect(function() TS:Create(btn, TweenInfo.new(0.08), {BackgroundColor3 = HOV}):Play() end)
                btn.MouseLeave:Connect(function() TS:Create(btn, TweenInfo.new(0.08), {BackgroundColor3 = INP}):Play() end)
                btn.Activated:Connect(function() saveMobileGridConfig() end)
        end

        -- ── Default tab ──────────────────────────────────────────

        -- Auto-scan: track any remaining GUI elements coloured with the default accent
        do
                local origPUR = Color3.fromRGB(148, 68, 255)
                local origDIM = Color3.fromRGB(90, 38, 170)
                local origSTK = Color3.fromRGB(55, 22, 100)
                for _, obj in ipairs(main:GetDescendants()) do
                        local bg = (obj:IsA("Frame") or obj:IsA("ImageLabel") or obj:IsA("ImageButton") or obj:IsA("TextButton") or obj:IsA("TextLabel"))
                        if bg then
                                local bc = obj.BackgroundColor3
                                if bc == origPUR then trackA(obj,   "BackgroundColor3")
                                elseif bc == origDIM then trackDim(obj, "BackgroundColor3")
                                elseif bc == origSTK then trackStk(obj, "BackgroundColor3") end
                        end
                        if obj:IsA("TextLabel") or obj:IsA("TextButton") then
                                local tc = obj.TextColor3
                                if tc == origPUR then trackA(obj,   "TextColor3")
                                elseif tc == origDIM then trackDim(obj, "TextColor3")
                                elseif tc == origSTK then trackStk(obj, "TextColor3") end
                        end
                        if obj:IsA("UIStroke") then
                                if obj.Color == origPUR then trackA(obj,   "Color")
                                elseif obj.Color == origDIM then trackDim(obj, "Color")
                                elseif obj.Color == origSTK then trackStk(obj, "Color") end
                        end
                end
        end
        switchTab("Speed")

        -- ── Mobile Button Pad ────────────────────────────────────
        do
                local PAD_BTN      = Color3.fromRGB(30, 12, 58)
                local PAD_BTN_HOV  = Color3.fromRGB(50, 20, 90)
                local PAD_TEXT     = Color3.fromRGB(235, 235, 235)

                local function mobileDrag(f)
                        local dn, ds, sp, di = false
                        f.InputBegan:Connect(function(i)
                                if mobileLocked then return end
                                if i.UserInputType == Enum.UserInputType.MouseButton1
                                        or i.UserInputType == Enum.UserInputType.Touch then
                                        dn = true; ds = i.Position; sp = f.Position
                                        i.Changed:Connect(function()
                                                if i.UserInputState == Enum.UserInputState.End then dn = false end
                                        end)
                                end
                        end)
                        f.InputChanged:Connect(function(i)
                                if i.UserInputType == Enum.UserInputType.MouseMovement
                                        or i.UserInputType == Enum.UserInputType.Touch then di = i end
                        end)
                        UIS.InputChanged:Connect(function(i)
                                if i == di and dn then
                                        local nX = sp.X.Offset + (i.Position.X - ds.X)
                                        local nY = sp.Y.Offset + (i.Position.Y - ds.Y)
                                        f.Position = UDim2.new(sp.X.Scale, nX, sp.Y.Scale, nY)
                                end
                        end)
                end

                local function refreshAllMobileAccent()
                        for _, entry in ipairs(allMobileBtns) do
                                local btn    = entry.btn
                                local stk    = entry.stroke
                                local isOn   = entry.getActiveState and entry.getActiveState()
                                stk.Color    = STROKE
                                if isOn then
                                        btn.BackgroundColor3 = PUR
                                else
                                        btn.BackgroundColor3 = PAD_BTN
                                end
                        end
                end

                local _origSetAccent = setAccent
                setAccent = function(c)
                        _origSetAccent(c)
                        refreshAllMobileAccent()
                end

                local function makeMobileBtn(label, col, row, onActivate, getActiveState, isTaunt)
                        local w = math.floor(MOBILE_BTN_W * btnScale)
                        local h = isTaunt and math.floor(MOBILE_TAUNT_H * btnScale)
                                or math.floor(MOBILE_BTN_H * btnScale)
                        local btnW = isTaunt and (w * 2 + MOBILE_GAP) or w

                        local btn = Instance.new("TextButton", gui)
                        btn.Size = UDim2.new(0, btnW, 0, h)
                        btn.Position = defaultMobilePos(col, row, isTaunt)
                        btn.BackgroundColor3 = PAD_BTN
                        btn.BorderSizePixel = 0
                        btn.Text = label
                        btn.TextColor3 = PAD_TEXT
                        btn.Font = Enum.Font.GothamBold
                        btn.TextSize = isTaunt and 10 or 12
                        btn.AutoButtonColor = false
                        btn.ZIndex = 20
                        btn.Active = true
                        local btnCorner = Instance.new("UICorner", btn)
                        btnCorner.CornerRadius = UDim.new(0, 14)
                        local btnStroke = Instance.new("UIStroke", btn)
                        btnStroke.Color = STROKE; btnStroke.Thickness = 1

                        local entry = {btn=btn, stroke=btnStroke, corner=btnCorner, getActiveState=getActiveState, isTaunt=isTaunt, gridCol=col, gridRow=row}
                        table.insert(allMobileBtns, entry)

                        local function refreshColor()
                                local isOn = getActiveState and getActiveState()
                                if isOn then
                                        btn.BackgroundColor3 = PUR
                                        btnStroke.Color = STROKE
                                else
                                        btn.BackgroundColor3 = PAD_BTN
                                        btnStroke.Color = STROKE
                                end
                        end

                        local PRESS_COL = Color3.fromRGB(255, 255, 255)

                        btn.InputBegan:Connect(function(i)
                                if i.UserInputType == Enum.UserInputType.MouseButton1
                                        or i.UserInputType == Enum.UserInputType.Touch then
                                        TS:Create(btn, TweenInfo.new(0.06), {BackgroundColor3 = PRESS_COL}):Play()
                                end
                        end)
                        btn.InputEnded:Connect(function(i)
                                if i.UserInputType == Enum.UserInputType.MouseButton1
                                        or i.UserInputType == Enum.UserInputType.Touch then
                                        task.delay(0.08, function() refreshColor() end)
                                end
                        end)
                        btn.MouseEnter:Connect(function()
                                if not (getActiveState and getActiveState()) then
                                        TS:Create(btn, TweenInfo.new(0.08), {BackgroundColor3 = PAD_BTN_HOV}):Play()
                                end
                        end)
                        btn.MouseLeave:Connect(function() refreshColor() end)
                        btn.Activated:Connect(function()
                                onActivate()
                                refreshColor()
                        end)

                        mobileDrag(btn)
                        return refreshColor
                end

                -- ── Row 1: AUTO LEFT | AUTO RIGHT ──────────────────
                local refreshAutoLeft = makeMobileBtn("AUTO LEFT", 1, 1,
                        function()
                                autoLeftEnabled = not autoLeftEnabled
                                if autoLeftEnabled then queueAutoLeftStart() else stopAutoLeft() end
                                if autoLeftSetVisual then autoLeftSetVisual(autoLeftEnabled) end
                        end,
                        function() return autoLeftEnabled end
                )
                autoLeftSetVisual = function(on)
                        autoLeftEnabled = on; refreshAutoLeft()
                end

                local refreshAutoRight = makeMobileBtn("AUTO RIGHT", 2, 1,
                        function()
                                autoRightEnabled = not autoRightEnabled
                                if autoRightEnabled then queueAutoRightStart() else stopAutoRight() end
                                if autoRightSetVisual then autoRightSetVisual(autoRightEnabled) end
                        end,
                        function() return autoRightEnabled end
                )
                autoRightSetVisual = function(on)
                        autoRightEnabled = on; refreshAutoRight()
                end

                -- ── Row 2: DROP BR | TP DOWN ───────────────────────
                makeMobileBtn("DROP BR",  1, 2, function() runDrop()    end, nil)
                makeMobileBtn("TP DOWN",  2, 2, function() runTPFloor() end, nil)

                -- ── Row 3: CARRY SPD | LAGGER MODE ─────────────────
                makeMobileBtn("CARRY SPD", 1, 3,
                        function() toggleCarryMode(); saveConfig() end,
                        function() return speedMode end
                )
                makeMobileBtn("LAGGER MODE", 2, 3,
                        function() toggleLaggerMode(); saveConfig() end,
                        function() return laggerToggled end
                )

                -- ── Row 4: INSTA RESET | AUTO BAT ──────────────────
                makeMobileBtn("INSTA RESET", 1, 4, function() cursedInstaReset() end, nil)

                local refreshAutoBat = makeMobileBtn("AUTO BAT", 2, 4,
                        function()
                                if not autoBatEnabled then
                                        queueAutoBatStart()
                                        if autoBatSetVisual then autoBatSetVisual(true) end
                                else
                                        autoBatEnabled = false; disableAutoBat()
                                        if autoBatSetVisual then autoBatSetVisual(false) end
                                end
                        end,
                        function() return autoBatEnabled end
                )
                autoBatSetVisual = function(on)
                        autoBatEnabled = on; refreshAutoBat()
                end

                -- ── Load saved grid positions/scale ────────────────
                do
                        local saved = loadMobileGridConfig()
                        if saved then
                                if saved.scale then resizeAllMobileBtns_outer() end
                                if saved.btns then
                                        local freeIdx = 0
                                        for _, entry in ipairs(allMobileBtns) do
                                                if not entry.isGridFixed then
                                                        freeIdx = freeIdx + 1
                                                        local p = saved.btns[freeIdx]
                                                        if p then
                                                                entry.btn.Position = UDim2.new(
                                                                        p.xs or 1, p.xo or 0,
                                                                        p.ys or 0, p.yo or 0
                                                                )
                                                        end
                                                end
                                        end
                                end
                        end
                end

                -- ── Unmovable Grid Container ───────────────────────
                do
                        local GW, GH = 110, 60
                        local GG = 8
                        local COLS = 2
                        local gridW = COLS * GW + (COLS + 1) * GG
                        local gridH = 4 * GH + 5 * GG

                        local gf = Instance.new("Frame", gui)
                        gf.Name = "UnmovableGrid"
                        gf.Size = UDim2.new(0, gridW, 0, gridH)
                        gf.Position = UDim2.new(1, -(gridW + 10), 0.5, -gridH / 2)
                        gf.BackgroundTransparency = 1
                        gf.BorderSizePixel = 0
                        gf.Active = false
                        gf.ZIndex = 20
                        gf.Visible = false
                        unmovableGridFrame = gf

                        drag(gf)

                        local function mkGridBtn(parent, label, col, row, w, h, getActiveState, onActivate)
                                local btn = Instance.new("TextButton", parent)
                                btn.Size = UDim2.new(0, w, 0, h)
                                btn.Position = UDim2.new(0, GG + (col-1)*(GW+GG), 0, GG + (row-1)*(GH+GG))
                                btn.BackgroundColor3 = PAD_BTN
                                btn.BorderSizePixel = 0
                                btn.Text = label
                                btn.TextColor3 = PAD_TEXT
                                btn.Font = Enum.Font.GothamBold
                                btn.TextSize = h < 40 and 10 or 12
                                btn.AutoButtonColor = false
                                btn.ZIndex = 21
                                local corner = Instance.new("UICorner", btn)
                                corner.CornerRadius = UDim.new(0, 14)
                                local stroke = Instance.new("UIStroke", btn)
                                stroke.Color = STROKE; stroke.Thickness = 1

                                local entry = {btn=btn, stroke=stroke, corner=corner,
                                        getActiveState=getActiveState, isTaunt=false,
                                        gridCol=col, gridRow=row, isGridFixed=true}
                                table.insert(allMobileBtns, entry)

                                local function refreshColor()
                                        local isOn = getActiveState and getActiveState()
                                        btn.BackgroundColor3 = isOn and PUR or PAD_BTN
                                        stroke.Color = STROKE
                                end

                                local PRESS_COL_G = Color3.fromRGB(255, 255, 255)

                                btn.InputBegan:Connect(function(i)
                                        if i.UserInputType == Enum.UserInputType.MouseButton1
                                                or i.UserInputType == Enum.UserInputType.Touch then
                                                TS:Create(btn, TweenInfo.new(0.06), {BackgroundColor3 = PRESS_COL_G}):Play()
                                        end
                                end)
                                btn.InputEnded:Connect(function(i)
                                        if i.UserInputType == Enum.UserInputType.MouseButton1
                                                or i.UserInputType == Enum.UserInputType.Touch then
                                                task.delay(0.08, function() refreshColor() end)
                                        end
                                end)
                                btn.MouseEnter:Connect(function()
                                        if not (getActiveState and getActiveState()) then
                                                TS:Create(btn, TweenInfo.new(0.08), {BackgroundColor3 = PAD_BTN_HOV}):Play()
                                        end
                                end)
                                btn.MouseLeave:Connect(function() refreshColor() end)
                                btn.Activated:Connect(function()
                                        onActivate()
                                        refreshColor()
                                end)
                                return btn, refreshColor
                        end

                        -- Row 1
                        mkGridBtn(gf, "AUTO LEFT",  1, 0, GW, GH,
                                function() return autoLeftEnabled end,
                                function() autoLeftEnabled = not autoLeftEnabled; if autoLeftEnabled then queueAutoLeftStart() else stopAutoLeft() end; if autoLeftSetVisual then autoLeftSetVisual(autoLeftEnabled) end end
                        )
                        mkGridBtn(gf, "AUTO RIGHT", 2, 0, GW, GH,
                                function() return autoRightEnabled end,
                                function() autoRightEnabled = not autoRightEnabled; if autoRightEnabled then queueAutoRightStart() else stopAutoRight() end; if autoRightSetVisual then autoRightSetVisual(autoRightEnabled) end end
                        )
                        -- Row 2
                        mkGridBtn(gf, "DROP BR",  1, 1, GW, GH, nil, function() runDrop() end)
                        mkGridBtn(gf, "TP DOWN",  2, 1, GW, GH, nil, function() runTPFloor() end)
                        -- Row 3
                        mkGridBtn(gf, "CARRY SPD", 1, 2, GW, GH,
                                function() return speedMode end,
                                function() toggleCarryMode(); saveConfig() end
                        )
                        mkGridBtn(gf, "LAGGER",    2, 2, GW, GH,
                                function() return laggerToggled end,
                                function() toggleLaggerMode(); saveConfig() end
                        )
                        -- Row 4
                        mkGridBtn(gf, "RESET",    1, 3, GW, GH, nil, function() cursedInstaReset() end)
                        mkGridBtn(gf, "AUTO BAT", 2, 3, GW, GH,
                                function() return autoBatEnabled end,
                                function()
                                        if not autoBatEnabled then queueAutoBatStart(); if autoBatSetVisual then autoBatSetVisual(true) end
                                        else autoBatEnabled = false; disableAutoBat(); if autoBatSetVisual then autoBatSetVisual(false) end end
                                end
                        )
                end
        end

        -- ── Discord link footer ──────────────────────────────────
        local discordRow = pRow(28)
        pLbl(discordRow, "DISCORD")
        local discordLink = Instance.new("TextLabel", discordRow)
        discordLink.Size = UDim2.new(0.7, 0, 1, 0)
        discordLink.Position = UDim2.new(0.3, 0, 0, 0)
        discordLink.BackgroundTransparency = 1
        discordLink.Text = "discord.gg/UN4bc2fJg"
        discordLink.TextColor3 = PUR
        discordLink.Font = Enum.Font.GothamBold
        discordLink.TextSize = 11
        discordLink.TextXAlignment = Enum.TextXAlignment.Left
        trackA(discordLink, "TextColor3")

        -- ── Save config on tab switch ───────────────────────────
        for tname, tbtn in pairs(tabButtons) do
                tbtn.MouseButton1Click:Connect(function() saveConfig() end)
        end

        return gui
end

-- ============================================================
--  START GUI
-- ============================================================
buildGui()

-- ============================================================
--  SAVE/LOAD CONFIG
-- ============================================================
local function saveConfig()
        local function ks(e) return {kb=e.kb and e.kb.Name or nil,gp=e.gp and e.gp.Name or nil} end
        local cfg={
                normalSpeed=NS,carrySpeed=CS,
                dropBrainrotKey=ks(KB.DropBrainrot),autoLeftKey=ks(KB.AutoLeft),autoRightKey=ks(KB.AutoRight),
                autoBatKey=ks(KB.AutoBat),laggerToggleKey=ks(KB.LaggerToggle),tpFloorKey=ks(KB.TPFloor),instaResetKey=ks(KB.InstaReset),guiHideKey=ks(KB.GuiHide),
                speedToggleKey=ks(KB.SpeedToggle),
                grabRadius=Steal.StealRadius,stealDuration=Steal.StealDuration,
                antiRagdoll=antiRagdollEnabled,autoStealEnabled=Steal.AutoStealEnabled,
                infiniteJump=infJumpEnabled,infJumpMode=infJumpMode,medusaCounter=medusaCounterEnabled,
                batCounter=batCounterEnabled,
                carryMode=speedMode,laggerMode=laggerToggled,laggerCarryMode=laggerPhase==2,laggerSpeed=LAGGER_SPEED,laggerCarrySpeed=LAGGER_CARRY_SPEED,
                autoBat=autoBatEnabled,autoSwing=autoSwingEnabled,
                unwalkEnabled=unwalkEnabled,
                antiLag=antiLagEnabled,stretchRez=stretchRezEnabled,
                autoTPEnabled=autoTPEnabled,autoTPHeight=autoTPHeight
        }
        if writefile then pcall(function() writefile("henikaHubConfig.json",HS:JSONEncode(cfg)) end) end
end
task.spawn(function() while task.wait(5) do saveConfig() end end)
