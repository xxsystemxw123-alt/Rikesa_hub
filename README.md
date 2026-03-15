-- === RIKESAHUB (MEGA OPTIMIZADO) ===
local Players,UIS,RS,TS,SGui,WS,Repl=game:GetService("Players"),game:GetService("UserInputService"),game:GetService("RunService"),game:GetService("TweenService"),game:GetService("StarterGui"),game:GetService("Workspace"),game:GetService("ReplicatedStorage")local LP,Camera=Players.LocalPlayer,WS.CurrentCamera
SGui:SetCore("SendNotification",{Title="RIKESA HUB",Text="[ RIKESA HUB ] Loaded!",Duration=4})
-- [1] GLOBALS
_G.selectedPart=nil;_G.selectedHighlight=nil;_G.floatSpeed=10;_G.flySpeed=1.5;_G.isFlying=false;_G.hiddenfling=false;_G.antiFling=false;_G.followCamera=false;_G.followDistance=15;_G.moveDirection=Vector3.new(0,0,0);_G.initialRotation=CFrame.new();_G.flingSpeed=10000;_G.multiSelect={};_G.selectRadius=50;_G.selectCircle=nil;_G.ringPartsEnabled=false;_G.tornadoConfig={radius=50,height=100,rotationSpeed=7,attractionStrength=1000,};_G.tornadoParts={};_G.godModeActive=false;_G.godModeConnection=nil;_G.fallDamageConnection=nil;_G.fallVelocityConnection=nil;_G.godModeTimer=nil;_G.originalWalkSpeed=16;_G.originalJumpPower=50;_G.flyBG=nil;_G.flyBV=nil;_G.flyConnection=nil;_G.autoAnimActive=false;_G.autoAnimTrack=nil;_G.antiFlingConnection=nil;_G.flingUI=nil;_G.flingActive=false;_G.flingConnection=nil;_G.flingNoclipConnection=nil;_G.flingPlayerNoclipParts={};_G.flingOtherNoclipParts={};_G.flingMasslessParts={};_G.collisionProtection=nil;_G.velocityMonitor=nil;_G.healthForceTimer=nil;_G.velocityProtection = nil;_G.customGravity = workspace.Gravity;_G.gravityInputActive = false
_G.customGravity = workspace.Gravity
_G.gravityInputActive = false
_G.antiFlingActive = false
_G.antiFlingConnections = {}
_G.antiFlingOriginalCanCollide = {}
-- [2] TROLL VARS
_G.bangActive=false;_G.faceBangActive=false;_G.sitActive=false;_G.suckActive=false;_G.ragdollActive=false;_G.infiniteJumpActive=false;_G.noclipActive=false;_G.espEnabled=false;_G.jerkActive=false;_G.zeroGActive=false;_G.zeroGConnection=nil
-- [3] CONNECTIONS
_G.bangConnection,_G.faceBangConnection,_G.sitConnection,_G.suckConnection=nil,nil,nil,nil;_G.ragdollConnection,_G.infiniteJumpConnection,_G.noclipConnection=nil,nil,nil;_G.flingConnection,_G.jerkTrack,_G.closerhandsTrack,_G.jerkTool=nil,nil,nil,nil
-- [4] FLY CAR
_G.flyCarActive=false;_G.flyCarSpeed=50;_G.flyCarBV=nil;_G.flyCarBG=nil
-- [5] CRASH PROTECTION
pcall(function()getgenv().__crashProtection=true end)
-- [6] SAFE CALL
local function safeCall(func,...)local success,result=pcall(func,...)if not success then warn("[RIKESA HUB] Error en ejecución:",result)end return result end
-- [7] HELPERS
local function gR()local c=LP.Character return c and c:FindFirstChild("HumanoidRootPart")end
local function gH()local c=LP.Character return c and c:FindFirstChildOfClass("Humanoid")end
local function getCamDir(d)if d.Y~=0 then return d end local l,r=Camera.CFrame.LookVector,Camera.CFrame.RightVector local lf,rf=Vector3.new(l.X,0,l.Z).Unit,Vector3.new(r.X,0,r.Z).Unit if d.Z>0 then return lf elseif d.Z<0 then return -lf elseif d.X>0 then return rf elseif d.X<0 then return -rf end return d end
-- [8] MOVE PARTS
local function clearForces(p)if not p then return end for _,c in p:GetChildren()do if c:IsA("BodyVelocity")or c:IsA("BodyPosition")or c:IsA("BodyGyro")then c:Destroy()end end end
local function processPart(p)if not p then return end clearForces(p)local bv=Instance.new("BodyVelocity",p);bv.Name="CV";bv.MaxForce=Vector3.new(math.huge,math.huge,math.huge);bv.Velocity=_G.moveDirection*_G.floatSpeed;bv.P=1250;local bg=Instance.new("BodyGyro",p);bg.Name="CG";bg.P=9e4;bg.MaxTorque=Vector3.new(9e9,9e9,9e9);bg.CFrame=p.CFrame end
local function updateVel(p)local bv=p and p:FindFirstChild("CV")if bv then bv.Velocity=_G.moveDirection*_G.floatSpeed end end
-- [9] SELECTION CIRCLE
local function createSelectionCircle(r)if _G.selectCircle then _G.selectCircle:Destroy()end local c=Instance.new("Part");c.Name="SelectionCircle";c.Anchored=true;c.CanCollide=false;c.Transparency=0.7;c.BrickColor=BrickColor.new("Bright yellow");c.Material=Enum.Material.Neon;c.Size=Vector3.new(r*2,0.2,r*2);c.Shape=Enum.PartType.Cylinder;c.Parent=workspace;local m=Instance.new("CylinderMesh",c);m.Scale=Vector3.new(1,0.1,1);local h=gR();if h then c.Position=h.Position-Vector3.new(0,1,0)end;_G.selectCircle=c;return c end
local function updateCirclePosition()if _G.selectCircle and _G.selectCircle.Parent then local h=gR();if h then _G.selectCircle.Position=h.Position-Vector3.new(0,1,0)end end end
-- [10] SELECT ALL
local function selectAllUnanchored()for _,p in pairs(_G.multiSelect)do if p and p.Parent then for _,h in p:GetChildren()do if h.Name=="MultiHighlight"then h:Destroy()end end end end;_G.multiSelect={};local h=gR()if not h then return end local c=h.Position for _,p in pairs(workspace:GetDescendants())do if p:IsA("BasePart")and not p.Anchored and p~=h then local d=(p.Position-c).Magnitude if d<=_G.selectRadius then table.insert(_G.multiSelect,p)local h=Instance.new("SelectionBox",p);h.Name="MultiHighlight";h.Adornee=p;h.Color3=Color3.fromRGB(100,255,100);h.LineThickness=0.1 end end end if _G.selectedPart and _G.selectedPart.Parent then local f=false for _,p in pairs(_G.multiSelect)do if p==_G.selectedPart then f=true break end end if not f then table.insert(_G.multiSelect,_G.selectedPart)end end end
-- [11] PROCESS MULTI
local function processMultiPart()for _,p in pairs(_G.multiSelect)do if p and p.Parent then clearForces(p)local bv=Instance.new("BodyVelocity",p);bv.Name="CV";bv.MaxForce=Vector3.new(math.huge,math.huge,math.huge);bv.Velocity=_G.moveDirection*_G.floatSpeed;bv.P=1250;local bg=Instance.new("BodyGyro",p);bg.Name="CG";bg.P=9e4;bg.MaxTorque=Vector3.new(9e9,9e9,9e9);bg.CFrame=p.CFrame end end end
local function updateMultiVelocity()for _,p in pairs(_G.multiSelect)do if p and p.Parent then local bv=p:FindFirstChild("CV")if bv then bv.Velocity=_G.moveDirection*_G.floatSpeed end end end end
local function clearMultiForces()for _,p in pairs(_G.multiSelect)do if p and p.Parent then clearForces(p)end end end
-- [12] TORNADO FUNCS
local function RetainTornadoPart(p)if p:IsA("BasePart")and not p.Anchored and p:IsDescendantOf(workspace)then if p.Parent==LP.Character or p:IsDescendantOf(LP.Character)then return false end return true end return false end
local function addTornadoPart(p)if RetainTornadoPart(p)then if p.Parent~=LP.Character and not p:IsDescendantOf(LP.Character)then if not table.find(_G.tornadoParts,p)then table.insert(_G.tornadoParts,p)end end end end
local function removeTornadoPart(p)local i=table.find(_G.tornadoParts,p)if i then table.remove(_G.tornadoParts,i)end end
local function restoreTornadoParts()for _,p in pairs(_G.tornadoParts)do if p and p.Parent then p.CanCollide=true;p.Velocity=Vector3.new(0,0,0);p.RotVelocity=Vector3.new(0,0,0)end end end
for _,p in pairs(workspace:GetDescendants())do addTornadoPart(p)end
workspace.DescendantAdded:Connect(addTornadoPart)workspace.DescendantRemoving:Connect(removeTornadoPart)
if LP.Character then local r={}for i,p in ipairs(_G.tornadoParts)do if p.Parent==LP.Character or p:IsDescendantOf(LP.Character)then table.insert(r,i)end end for i=#r,1,-1 do table.remove(_G.tornadoParts,r[i])end end
-- [13] TOGGLE TORNADO
function toggleTornado(b)
    _G.ringPartsEnabled = not _G.ringPartsEnabled
    if b then
        b.Text = _G.ringPartsEnabled and "🌪️ RingParts: ON" or "🌪️ RingParts: OFF"
        b.BackgroundColor3 = _G.ringPartsEnabled and Color3.fromRGB(100,200,100) or Color3.fromRGB(210,160,130)
    end
    
    if _G.ringPartsEnabled then
        -- Activar noclip si no está activado
        if not _G.noclipActive then
            toggleNoclip()
        end
    else
        -- Desactivar noclip si estaba activado
        if _G.noclipActive then
            toggleNoclip()
        end
        restoreTornadoParts()
    end
end
-- [14] TOGGLE BANG (CORREGIDO - SIEMPRE DETRÁS)
function toggleBang(b)
    if _G.bangActive then
        _G.bangActive = false
        if _G.bangConnection then _G.bangConnection:Disconnect() _G.bangConnection = nil end
        local h = gR()
        if h then
            for _, c in h:GetChildren() do
                if c:IsA("BodyPosition") or c:IsA("BodyGyro") then c:Destroy() end
            end
        end
        local hum = gH()
        if hum then
            hum.Sit = false; hum.PlatformStand = false; hum.AutoRotate = true
            pcall(function() hum:ChangeState(Enum.HumanoidStateType.Running) end)
            if _G.originalWalkSpeed then pcall(function() hum.WalkSpeed = _G.originalWalkSpeed end) end
            pcall(function()
                local a = LP.Character and LP.Character:FindFirstChild("Animate")
                if a and a.walk and _G.originalWalkAnim then a.walk.WalkAnim.AnimationId = _G.originalWalkAnim end
            end)
        end
        if b then b.BackgroundColor3 = Color3.fromRGB(200,130,110) end
        return
    end
    
    _G.bangActive = true
    if b then b.BackgroundColor3 = Color3.fromRGB(100,200,100) end
    
    pcall(function()
        local h = gH()
        if h and not _G.originalWalkSpeed then _G.originalWalkSpeed = h.WalkSpeed end
        if h then h.WalkSpeed = (_G.originalWalkSpeed or 16) * 1.2 end
        local a = LP.Character and LP.Character:FindFirstChild("Animate")
        if a and a.walk and not _G.originalWalkAnim then _G.originalWalkAnim = a.walk.WalkAnim.AnimationId end
        if a and a.walk then a.walk.WalkAnim.AnimationId = "http://www.roblox.com/asset/?id=616168032" end
    end)
    
    local target, myRoot, nearest = nil, gR(), math.huge
    if myRoot then
        for _, p in Players:GetPlayers() do
            if p ~= LP and p.Character then
                local rt = p.Character:FindFirstChild("HumanoidRootPart")
                if rt then
                    local d = (myRoot.Position - rt.Position).Magnitude
                    if d < nearest then nearest = d; target = p end
                end
            end
        end
    end
    if not target then
        _G.bangActive = false
        if b then b.BackgroundColor3 = Color3.fromRGB(200,130,110) end
        return
    end
    
    local lastDirChange = tick()
    local direction = 1
    
    _G.bangConnection = RS.Heartbeat:Connect(function()
        if not _G.bangActive or not target.Character then
            _G.bangActive = false
            if _G.bangConnection then _G.bangConnection:Disconnect() _G.bangConnection = nil end
            return
        end
        
        local tr = target.Character:FindFirstChild("HumanoidRootPart")
        local myHrp = gR()
        if not tr or not myHrp then return end
        
        -- Cambiar dirección lateral cada 0.3 segundos
        if tick() - lastDirChange > 0.3 then
            direction = direction * -1
            lastDirChange = tick()
        end
        
        -- Calcular posición DETRÁS del jugador objetivo (basado en su dirección actual)
        local behindPos = tr.Position - tr.CFrame.LookVector * 1.5  -- 1.5 studs detrás
        
        -- Ajustar a la misma altura
        behindPos = Vector3.new(behindPos.X, tr.Position.Y, behindPos.Z)
        
        -- Añadir movimiento lateral (izquierda/derecha)
        local rightVector = tr.CFrame.RightVector
        local targetPos = behindPos + rightVector * direction * 1.2
        
        -- Crear o actualizar BodyPosition
        local bp = myHrp:FindFirstChild("BangBP")
        if not bp then
            bp = Instance.new("BodyPosition")
            bp.Name = "BangBP"
            bp.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
            bp.P = 10000
            bp.Parent = myHrp
        end
        bp.Position = targetPos
        
        -- Crear o actualizar BodyGyro para mirar al objetivo
        local bg = myHrp:FindFirstChild("BangBG")
        if not bg then
            bg = Instance.new("BodyGyro")
            bg.Name = "BangBG"
            bg.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
            bg.Parent = myHrp
        end
        bg.CFrame = CFrame.new(myHrp.Position, tr.Position)
    end)
end
function toggleFaceBang(b)
    if _G.faceBangActive then
        _G.faceBangActive = false
        if _G.faceBangConnection then _G.faceBangConnection:Disconnect() _G.faceBangConnection = nil end
        local h = gR()
        if h then
            for _, c in h:GetChildren() do
                if c:IsA("BodyPosition") or c:IsA("BodyGyro") then c:Destroy() end
            end
        end
        local hum = gH()
        if hum then
            hum.Sit = false; hum.PlatformStand = false; hum.AutoRotate = true
            hum:ChangeState(Enum.HumanoidStateType.Running)
        end
        if b then
            b.Text = "😳 FaceBang"
            b.BackgroundColor3 = Color3.fromRGB(150,170,200)
        end
        return
    end
    
    _G.faceBangActive = true
    if b then
        b.Text = "😳 FaceBang: ON"
        b.BackgroundColor3 = Color3.fromRGB(100,200,100)
    end
    
    -- Buscar el jugador más cercano
    local t, r, n = nil, gR(), math.huge
    if r then
        for _, p in Players:GetPlayers() do
            if p ~= LP and p.Character then
                local rt = p.Character:FindFirstChild("HumanoidRootPart")
                if rt then
                    local d = (r.Position - rt.Position).Magnitude
                    if d < n then n = d; t = p end
                end
            end
        end
    end
    if not t then
        _G.faceBangActive = false
        if b then b.BackgroundColor3 = Color3.fromRGB(150,170,200) end
        return
    end
    
    local hum = gH()
    if hum then hum.Sit = true end
    
    local l, d = tick(), 1
    _G.faceBangConnection = RS.Heartbeat:Connect(function()
        if not _G.faceBangActive or not t.Character then
            _G.faceBangActive = false
            if _G.faceBangConnection then _G.faceBangConnection:Disconnect() _G.faceBangConnection = nil end
            return
        end
        
        local th = t.Character:FindFirstChild("Head")
        local hrp, head = gR(), LP.Character and LP.Character:FindFirstChild("Head")
        if not th or not hrp or not head then return end
        
        if tick() - l > 0.2 then
            d = d * -1
            l = tick()
        end
        
        local bs = th.Position + th.CFrame.LookVector
        local tp = Vector3.new(bs.X, th.Position.Y - head.Size.Y/2 + 0.5, bs.Z) + th.CFrame.LookVector * d * 0.8
        
        local bp = hrp:FindFirstChild("FaceBangBP") or Instance.new("BodyPosition", hrp)
        bp.Name = "FaceBangBP"
        bp.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
        bp.P = 10000
        bp.Position = tp
        
        local bg = hrp:FindFirstChild("FaceBangBG") or Instance.new("BodyGyro", hrp)
        bg.Name = "FaceBangBG"
        bg.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
        bg.CFrame = CFrame.new(hrp.Position, th.Position)
    end)
end
-- [16] TOGGLE SIT
function toggleSit(b)if _G.sitActive then _G.sitActive=false if _G.sitConnection then _G.sitConnection:Disconnect()_G.sitConnection=nil end local h=gR()if h then for _,c in h:GetChildren()do if c:IsA("BodyPosition")or c:IsA("BodyGyro")then c:Destroy()end end end local hum=gH()if hum then hum.Sit=false;hum.PlatformStand=false;hum.AutoRotate=true;hum:ChangeState(Enum.HumanoidStateType.Running)end if b then b.BackgroundColor3=Color3.fromRGB(150,180,150)end return end _G.sitActive=true if b then b.BackgroundColor3=Color3.fromRGB(100,200,100)end local t,r,n=nil,gR(),math.huge if r then for _,p in Players:GetPlayers()do if p~=LP and p.Character then local rt=p.Character:FindFirstChild("HumanoidRootPart")if rt then local d=(r.Position-rt.Position).Magnitude if d<n then n=d t=p end end end end end if not t then _G.sitActive=false if b then b.BackgroundColor3=Color3.fromRGB(150,180,150)end return end local hum=gH()if hum then hum.Sit=true end _G.sitConnection=RS.Heartbeat:Connect(function()if not _G.sitActive or not t.Character then _G.sitActive=false if _G.sitConnection then _G.sitConnection:Disconnect()_G.sitConnection=nil end return end local th=t.Character:FindFirstChild("Head")local hrp=gR()if not th or not hrp then return end local bp=hrp:FindFirstChild("SitBP")or Instance.new("BodyPosition",hrp);bp.Name="SitBP";bp.MaxForce=Vector3.new(math.huge,math.huge,math.huge);bp.P=10000;bp.Position=th.Position+Vector3.new(0,0.8,0);local bg=hrp:FindFirstChild("SitBG")or Instance.new("BodyGyro",hrp);bg.Name="SitBG";bg.MaxTorque=Vector3.new(9e9,9e9,9e9);bg.CFrame=th.CFrame end)end
-- [17] TOGGLE SUCK
function toggleSuck(b)if _G.suckActive then _G.suckActive=false if _G.suckConnection then _G.suckConnection:Disconnect()_G.suckConnection=nil end local h=gR()if h then for _,c in h:GetChildren()do if c:IsA("BodyPosition")or c:IsA("BodyGyro")then c:Destroy()end end end local hum=gH()if hum then hum.Sit=false;hum.PlatformStand=false;hum.AutoRotate=true;pcall(function()hum:ChangeState(Enum.HumanoidStateType.Running)end)pcall(function()local a=LP.Character and LP.Character:FindFirstChild("Animate")if a and a.walk and _G.originalWalkAnim then a.walk.WalkAnim.AnimationId=_G.originalWalkAnim end end)end if b then b.BackgroundColor3=Color3.fromRGB(200,150,180)end return end _G.suckActive=true if b then b.BackgroundColor3=Color3.fromRGB(100,200,100)end pcall(function()local a=LP.Character and LP.Character:FindFirstChild("Animate")if a and a.walk and not _G.originalWalkAnim then _G.originalWalkAnim=a.walk.WalkAnim.AnimationId end if a and a.walk then a.walk.WalkAnim.AnimationId="http://www.roblox.com/asset/?id=95643163365384"end local h=gR()if h then h.CFrame=h.CFrame+Vector3.new(0,-0.5,0)end end)local t,r,n=nil,gR(),math.huge if r then for _,p in Players:GetPlayers()do if p~=LP and p.Character then local rt=p.Character:FindFirstChild("HumanoidRootPart")if rt then local d=(r.Position-rt.Position).Magnitude if d<n then n=d t=p end end end end end if not t then _G.suckActive=false if b then b.BackgroundColor3=Color3.fromRGB(200,150,180)end return end local l,d=tick(),1 _G.suckConnection=RS.Heartbeat:Connect(function()if not _G.suckActive or not t.Character then _G.suckActive=false if _G.suckConnection then _G.suckConnection:Disconnect()_G.suckConnection=nil end return end local tr=t.Character:FindFirstChild("HumanoidRootPart")or t.Character:FindFirstChild("Torso")local hrp,head=gR(),LP.Character and LP.Character:FindFirstChild("Head")if not tr or not hrp or not head then return end if tick()-l>0.25 then d=d*-1 l=tick()end local gr=tr.Position+Vector3.new(0,-2,0);local fr=gr+tr.CFrame.LookVector*1.2;local tp=Vector3.new(fr.X,gr.Y-head.Size.Y/2-0.2,fr.Z)+tr.CFrame.LookVector*d;local bp=hrp:FindFirstChild("SuckBP")or Instance.new("BodyPosition",hrp);bp.Name="SuckBP";bp.MaxForce=Vector3.new(math.huge,math.huge,math.huge);bp.P=10000;bp.Position=tp;bp.Parent=hrp;local bg=hrp:FindFirstChild("SuckBG")or Instance.new("BodyGyro",hrp);bg.Name="SuckBG";bg.MaxTorque=Vector3.new(9e9,9e9,9e9);bg.CFrame=CFrame.new(hrp.Position,tr.Position)end)end
-- [18] TOGGLE RAGDOLL
function toggleRagdoll(b)_G.ragdollActive=not _G.ragdollActive;local h=gH()if not h then return end if _G.ragdollActive then h.PlatformStand=true;h.AutoRotate=false;h:ChangeState(Enum.HumanoidStateType.Physics)local r=gR()if r then r.Velocity=Vector3.new(0,-10,0)end if b then b.BackgroundColor3=Color3.fromRGB(100,200,100)end else h.PlatformStand=false;h.AutoRotate=true;h:ChangeState(Enum.HumanoidStateType.Running)if b then b.BackgroundColor3=Color3.fromRGB(210,160,130)end end end
-- [19] TOGGLE INFINITE JUMP
function toggleInfiniteJump(b)_G.infiniteJumpActive=not _G.infiniteJumpActive if _G.infiniteJumpActive then _G.infiniteJumpConnection=UIS.JumpRequest:Connect(function()if not _G.infiniteJumpActive then if _G.infiniteJumpConnection then _G.infiniteJumpConnection:Disconnect()_G.infiniteJumpConnection=nil end return end local h=gH()if h then h:ChangeState(Enum.HumanoidStateType.Jumping)end end)if b then b.BackgroundColor3=Color3.fromRGB(100,200,100)end else if _G.infiniteJumpConnection then _G.infiniteJumpConnection:Disconnect()_G.infiniteJumpConnection=nil end if b then b.BackgroundColor3=Color3.fromRGB(150,170,200)end end end
-- [20] TOGGLE NOCLIP
function toggleNoclip(b)
    _G.noclipActive = not _G.noclipActive
    if b then
        b.BackgroundColor3 = _G.noclipActive and Color3.fromRGB(100,200,100) or Color3.fromRGB(150,170,200)
    end
    
    if _G.noclipActive then
        if not _G.noclipConnection then
            _G.noclipConnection = RS.Stepped:Connect(function()
                if not _G.noclipActive or not LP.Character then return end
                for _, p in LP.Character:GetDescendants() do
                    if p:IsA("BasePart") then p.CanCollide = false end
                end
            end)
        end
    else
        if _G.noclipConnection then
            _G.noclipConnection:Disconnect()
            _G.noclipConnection = nil
        end
        if LP.Character then
            for _, p in LP.Character:GetDescendants() do
                if p:IsA("BasePart") then p.CanCollide = true end
            end
        end
    end
end
-- [21] TOGGLE ESP
function toggleESP(b)_G.espEnabled=not _G.espEnabled if _G.espEnabled then for _,p in Players:GetPlayers()do if p~=LP and p.Character then local h=Instance.new("Highlight",p.Character);h.Adornee=p.Character;h.FillColor=Color3.fromRGB(255,255,0);h.FillTransparency=0.5 end end if b then b.BackgroundColor3=Color3.fromRGB(100,200,100)end else for _,p in Players:GetPlayers()do if p.Character then local o=p.Character:FindFirstChildOfClass("Highlight")if o then o:Destroy()end end end if b then b.BackgroundColor3=Color3.fromRGB(150,170,200)end end end
-- [22] RESET STATS
function resetStats(b)if _G.bangActive then toggleBang()end if _G.faceBangActive then toggleFaceBang()end if _G.sitActive then toggleSit()end if _G.suckActive then toggleSuck()end if _G.ragdollActive then toggleRagdoll()end if _G.infiniteJumpActive then toggleInfiniteJump()end if _G.noclipActive then toggleNoclip()end if _G.espEnabled then toggleESP()end if _G.jerkActive then toggleJerk()end if _G.hiddenfling then _G.hiddenfling=false if _G.flingConnection then _G.flingConnection:Disconnect()_G.flingConnection=nil end end if b then b.BackgroundColor3=Color3.fromRGB(180,140,100)end end
-- [23] TOGGLE GOD MODE (CONGELA LA SALUD - NUNCA BAJA)
function toggleGodMode(b)
    _G.godModeActive = not _G.godModeActive
    
    -- Desconectar conexiones anteriores
    if _G.godModeConnection then _G.godModeConnection:Disconnect() _G.godModeConnection = nil end
    if _G.healthFreeze then _G.healthFreeze:Disconnect() _G.healthFreeze = nil end
    
    local character = LP.Character
    if not character then
        if b then
            b.Text = _G.godModeActive and "🛡️ GOD MODE: ON" or "🛡️ GOD MODE: OFF"
            b.BackgroundColor3 = _G.godModeActive and Color3.fromRGB(0,170,0) or Color3.fromRGB(200,150,180)
        end
        return
    end
    
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    
    if _G.godModeActive then
        -- Guardar la salud actual como referencia
        local frozenHealth = humanoid.Health
        
        -- CONGELAR LA SALUD (nunca baja, nunca sube)
        _G.healthFreeze = RS.RenderStepped:Connect(function()
            if _G.godModeActive and humanoid and humanoid.Parent then
                -- Forzar la salud al valor congelado
                if humanoid.Health ~= frozenHealth then
                    humanoid.Health = frozenHealth
                end
            end
        end)
        
        -- También detectar cambios por si acaso
        _G.godModeConnection = humanoid:GetPropertyChangedSignal("Health"):Connect(function()
            if _G.godModeActive and humanoid.Health ~= frozenHealth then
                humanoid.Health = frozenHealth
            end
        end)
        
        if b then
            b.Text = "🛡️ GOD MODE: ON"
            b.BackgroundColor3 = Color3.fromRGB(0,170,0)
        end
        
    else
        -- DESACTIVAR GOD MODE
        if b then
            b.Text = "🛡️ GOD MODE: OFF"
            b.BackgroundColor3 = Color3.fromRGB(200,150,180)
        end
    end
end

-- MANEJAR RESPAWN
LP.CharacterAdded:Connect(function(newChar)
    task.wait(0.5)
    if _G.godModeActive then
        local humanoid = newChar:FindFirstChildOfClass("Humanoid")
        if humanoid then
            -- Al renacer, congelar la salud del nuevo personaje
            local frozenHealth = humanoid.Health
            
            if _G.healthFreeze then _G.healthFreeze:Disconnect() end
            if _G.godModeConnection then _G.godModeConnection:Disconnect() end
            
            _G.healthFreeze = RS.RenderStepped:Connect(function()
                if _G.godModeActive and humanoid and humanoid.Parent then
                    if humanoid.Health ~= frozenHealth then
                        humanoid.Health = frozenHealth
                    end
                end
            end)
            
            _G.godModeConnection = humanoid:GetPropertyChangedSignal("Health"):Connect(function()
                if _G.godModeActive and humanoid.Health ~= frozenHealth then
                    humanoid.Health = frozenHealth
                end
            end)
        end
    end
end)

-- [24] TOGGLE FLING ARKINEKU
function toggleFlingArkineku(b)
    if _G.flingActive then
        -- DESACTIVAR FLING
        _G.flingActive = false
        
        -- Detener conexiones
        if _G.flingConnection then _G.flingConnection:Disconnect() _G.flingConnection = nil end
        if _G.flingNoclipConnection then _G.flingNoclipConnection:Disconnect() _G.flingNoclipConnection = nil end
        
        -- Restaurar CanCollide de otros jugadores
        for part, state in pairs(_G.flingOtherNoclipParts) do
            if part and part.Parent then pcall(function() part.CanCollide = state end) end
        end
        _G.flingOtherNoclipParts = {}
        
        -- Restaurar CanCollide propio
        for part, state in pairs(_G.flingPlayerNoclipParts) do
            if part and part.Parent then pcall(function() part.CanCollide = state end) end
        end
        _G.flingPlayerNoclipParts = {}
        
        -- Restaurar Massless
        for part, state in pairs(_G.flingMasslessParts) do
            if part and part.Parent then pcall(function() part.Massless = state end) end
        end
        _G.flingMasslessParts = {}
        
        -- Restaurar velocidad angular
        local hrp = gR()
        if hrp then pcall(function() hrp.AssemblyAngularVelocity = Vector3.new(0,0,0) end) end
        
        -- Restaurar estados del humanoid
        local hum = gH()
        if hum then
            pcall(function() 
                hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, true)
                hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, true)
                hum:SetStateEnabled(Enum.HumanoidStateType.Physics, true)
            end)
        end
        
                -- Actualizar botón si existe
        if b then 
            b.Text = "⚡ FLING: OFF"
            b.BackgroundColor3 = Color3.fromRGB(200, 160, 120)  -- Beige oscuro
        end
        
        -- Destruir UI si existe
        if _G.flingUI and _G.flingUI.Parent then _G.flingUI:Destroy() end
        _G.flingUI = nil
        
    else
        -- ACTIVAR FLING
        _G.flingActive = true
        
        -- Función para iniciar fling
        local function startFlingLogic()
            local char = LP.Character
            if not char then return end
            local hum = char:FindFirstChildOfClass("Humanoid")
            local hrp = char:FindFirstChild("HumanoidRootPart")
            if not hrp or not hum then return end
            
            -- Intentar tomar ownership
            pcall(function() if hrp:CanSetNetworkOwnership() then hrp:SetNetworkOwner(LP) end end)
            
            -- Deshabilitar estados
            pcall(function()
                hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
                hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
                hum:SetStateEnabled(Enum.HumanoidStateType.Physics, false)
            end)
            
            -- Guardar y modificar Massless
            for _, part in pairs(char:GetDescendants()) do
                if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
                    _G.flingMasslessParts[part] = part.Massless
                    pcall(function() part.Massless = true end)
                end
            end
            
            -- Conexión de noclip
            if _G.flingNoclipConnection then _G.flingNoclipConnection:Disconnect() end
            _G.flingNoclipConnection = RS.Stepped:Connect(function()
                if not _G.flingActive then return end
                local c = LP.Character
                if not c then return end
                
                -- Noclip propio
                for _, part in pairs(c:GetDescendants()) do
                    if part:IsA("BasePart") then
                        if not _G.flingPlayerNoclipParts[part] then
                            _G.flingPlayerNoclipParts[part] = part.CanCollide
                        end
                        pcall(function() part.CanCollide = false end)
                    end
                end
                
                -- Noclip de otros jugadores
                for _, p in pairs(Players:GetPlayers()) do
                    if p ~= LP and p.Character then
                        for _, part in pairs(p.Character:GetDescendants()) do
                            if part:IsA("BasePart") then
                                if not _G.flingOtherNoclipParts[part] then
                                    _G.flingOtherNoclipParts[part] = part.CanCollide
                                end
                                pcall(function() part.CanCollide = false end)
                            end
                        end
                    end
                end
            end)
            
            -- Conexión de fling
            if _G.flingConnection then _G.flingConnection:Disconnect() end
            _G.flingConnection = RS.Heartbeat:Connect(function()
                if not _G.flingActive then return end
                local c = LP.Character
                if not c then return end
                local hrp2 = c:FindFirstChild("HumanoidRootPart")
                local hum2 = c:FindFirstChildOfClass("Humanoid")
                if not hrp2 or not hum2 or hum2.Health <= 0 then return end
                
                -- Rotación extrema
                local _, cy, _ = hrp2.CFrame:ToEulerAnglesYXZ()
                pcall(function()
                    hrp2.CFrame = CFrame.new(hrp2.Position) * CFrame.Angles(0, cy, 0)
                    hrp2.AssemblyAngularVelocity = Vector3.new(0, 99999999, 0)
                end)
            end)
        end
        
        -- Iniciar fling
        startFlingLogic()
        
        -- Actualizar botón principal
        if b then 
            b.Text = "⚡ FLING: ON"
            b.BackgroundColor3 = Color3.fromRGB(200, 160, 120)  -- Beige oscuro (mismo color)
        end
    end
end

-- Reiniciar fling al renacer
LP.CharacterAdded:Connect(function(newChar)
    task.wait(0.5)
    if _G.flingActive then
        -- Reiniciar el fling
        if _G.flingConnection then _G.flingConnection:Disconnect() _G.flingConnection = nil end
        if _G.flingNoclipConnection then _G.flingNoclipConnection:Disconnect() _G.flingNoclipConnection = nil end
        
        local function restartFling()
            local hum = newChar:FindFirstChildOfClass("Humanoid")
            local hrp = newChar:FindFirstChild("HumanoidRootPart")
            if not hrp or not hum then task.wait(0.1) return restartFling() end
            
            pcall(function() if hrp:CanSetNetworkOwnership() then hrp:SetNetworkOwner(LP) end end)
            
            pcall(function()
                hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
                hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
                hum:SetStateEnabled(Enum.HumanoidStateType.Physics, false)
            end)
            
            for _, part in pairs(newChar:GetDescendants()) do
                if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
                    _G.flingMasslessParts[part] = part.Massless
                    pcall(function() part.Massless = true end)
                end
            end
            
            _G.flingNoclipConnection = RS.Stepped:Connect(function()
                if not _G.flingActive then return end
                local c = LP.Character
                if not c then return end
                for _, part in pairs(c:GetDescendants()) do
                    if part:IsA("BasePart") then
                        if not _G.flingPlayerNoclipParts[part] then
                            _G.flingPlayerNoclipParts[part] = part.CanCollide
                        end
                        pcall(function() part.CanCollide = false end)
                    end
                end
                for _, p in pairs(Players:GetPlayers()) do
                    if p ~= LP and p.Character then
                        for _, part in pairs(p.Character:GetDescendants()) do
                            if part:IsA("BasePart") then
                                if not _G.flingOtherNoclipParts[part] then
                                    _G.flingOtherNoclipParts[part] = part.CanCollide
                                end
                                pcall(function() part.CanCollide = false end)
                            end
                        end
                    end
                end
            end)
            
            _G.flingConnection = RS.Heartbeat:Connect(function()
                if not _G.flingActive then return end
                local c = LP.Character
                if not c then return end
                local hrp2 = c:FindFirstChild("HumanoidRootPart")
                local hum2 = c:FindFirstChildOfClass("Humanoid")
                if not hrp2 or not hum2 or hum2.Health <= 0 then return end
                local _, cy, _ = hrp2.CFrame:ToEulerAnglesYXZ()
                pcall(function()
                    hrp2.CFrame = CFrame.new(hrp2.Position) * CFrame.Angles(0, cy, 0)
                    hrp2.AssemblyAngularVelocity = Vector3.new(0, 99999999, 0)
                end)
            end)
        end
        
        restartFling()
    end
end)

-- [25] TOGGLE AntiFling
function toggleAntiFling(b)
    _G.antiFlingActive = not _G.antiFlingActive
    
    local Players = game:GetService("Players")
    local localPlayer = Players.LocalPlayer
    
    -- Función para deshabilitar colisiones
    local function disableCollisions(character)
        if not character then return end
        
        -- Guardar estado original y deshabilitar
        for _, part in ipairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                if not _G.antiFlingOriginalCanCollide[part] then
                    _G.antiFlingOriginalCanCollide[part] = part.CanCollide
                end
                part.CanCollide = false
            end
        end
        
        -- Conectar para futuras partes
        local conn = character.DescendantAdded:Connect(function(part)
            if part:IsA("BasePart") then
                if not _G.antiFlingOriginalCanCollide[part] then
                    _G.antiFlingOriginalCanCollide[part] = part.CanCollide
                end
                part.CanCollide = false
            end
        end)
        table.insert(_G.antiFlingConnections, conn)
    end
    
    -- Función para restaurar colisiones
    local function restoreCollisions(character)
        if not character then return end
        
        for _, part in ipairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                if _G.antiFlingOriginalCanCollide[part] ~= nil then
                    part.CanCollide = _G.antiFlingOriginalCanCollide[part]
                end
            end
        end
    end
    
    -- Función para manejar jugadores
    local function onPlayerAdded(player)
        if player == localPlayer then return end
        
        if player.Character then
            if _G.antiFlingActive then
                disableCollisions(player.Character)
            end
        end
        
        local conn = player.CharacterAdded:Connect(function(character)
            if _G.antiFlingActive then
                disableCollisions(character)
            end
        end)
        table.insert(_G.antiFlingConnections, conn)
    end
    
    if _G.antiFlingActive then
        -- ACTIVAR ANTI FLING
        -- Desconectar conexiones anteriores si existen
        for _, conn in ipairs(_G.antiFlingConnections) do
            conn:Disconnect()
        end
        _G.antiFlingConnections = {}
        
        -- Aplicar a todos los jugadores actuales
        for _, player in ipairs(Players:GetPlayers()) do
            onPlayerAdded(player)
        end
        
        -- Conectar para jugadores nuevos
        local conn = Players.PlayerAdded:Connect(onPlayerAdded)
        table.insert(_G.antiFlingConnections, conn)
        
        if b then
            b.Text = "AntiFling: ON"
            b.BackgroundColor3 = Color3.fromRGB(200, 160, 120)  -- Beige oscuro (mismo que FLING)
        end
        
    else
        -- DESACTIVAR ANTI FLING
        -- Restaurar colisiones de todos los jugadores
        for _, player in ipairs(Players:GetPlayers()) do
            if player.Character then
                restoreCollisions(player.Character)
            end
        end
        
        -- Desconectar todas las conexiones
        for _, conn in ipairs(_G.antiFlingConnections) do
            conn:Disconnect()
        end
        _G.antiFlingConnections = {}
        
        -- Limpiar datos guardados
        _G.antiFlingOriginalCanCollide = {}
        
        if b then
            b.Text = "AntiFling: OFF"
            b.BackgroundColor3 = Color3.fromRGB(200, 160, 120)  -- MISMO beige oscuro
        end
    end
end

-- [26] TOGGLE JERK (CARGAR SCRIPT EXTERNO)
function toggleJerk(b)
    -- Solo cargar el script, sin lógica adicional
    loadstring(game:HttpGet("https://pastefy.app/YZoglOyJ/raw"))()
    
    -- Opcional: Feedback visual
    if b then
        b.Text = "✅ JERK CARGADO"
        b.BackgroundColor3 = Color3.fromRGB(100,200,100)
        task.wait(1)
        b.Text = "🍆 JERK"
        b.BackgroundColor3 = Color3.fromRGB(180,140,200)
    end
end
function toggleZeroG(b)
    _G.zeroGActive = not _G.zeroGActive
    local character = LP.Character
    if not character then return end
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if _G.zeroGActive then
        _G.savedWalkSpeed = humanoid and humanoid.WalkSpeed or 16
        _G.savedJumpPower = humanoid and humanoid.JumpPower or 50
        _G.savedGravity = workspace.Gravity
        
        local gravValue = _G.zeroGravityValue * _G.zeroGravityDirection
        
        if humanoid then
            humanoid.PlatformStand = true
            humanoid:ChangeState(Enum.HumanoidStateType.Physics)
        end
        
        if gravValue < 0 then
            workspace.Gravity = 0
            local rootPart = character:FindFirstChild("HumanoidRootPart") or character:FindFirstChild("Torso")
            if rootPart then
                local bodyForce = rootPart:FindFirstChild("ZeroGForce")
                if not bodyForce then
                    bodyForce = Instance.new("BodyForce")
                    bodyForce.Name = "ZeroGForce"
                    bodyForce.Parent = rootPart
                end
                bodyForce.Force = Vector3.new(0, math.abs(gravValue) * rootPart.AssemblyMass, 0)
            end
        else
            local bodyForce = character:FindFirstChild("HumanoidRootPart") and character:FindFirstChild("HumanoidRootPart"):FindFirstChild("ZeroGForce")
            if bodyForce then bodyForce:Destroy() end
            workspace.Gravity = gravValue
        end
        
        if b then
            b.Text = "🪐 ZeroGravity: ON"  -- 🔹 NOMBRE NUEVO (encendido)
            b.BackgroundColor3 = Color3.fromRGB(100,100,200)
        end
    else
        if humanoid then
            humanoid.PlatformStand = false
            humanoid:ChangeState(Enum.HumanoidStateType.Running)
            humanoid.WalkSpeed = _G.savedWalkSpeed or 16
            humanoid.JumpPower = _G.savedJumpPower or 50
        end
        
        local bodyForce = character and character:FindFirstChild("HumanoidRootPart") and character:FindFirstChild("HumanoidRootPart"):FindFirstChild("ZeroGForce")
        if bodyForce then bodyForce:Destroy() end
        
        if _G.savedGravity then
            workspace.Gravity = _G.savedGravity
        end
        
        if b then
            b.Text = "🪐 ZeroGravity: OFF"  -- 🔹 NOMBRE NUEVO (apagado)
            b.BackgroundColor3 = Color3.fromRGB(200,150,180)
        end
    end
end

-- [28] FUNCIONES DE MOVIMIENTO PARA FLY CAR
function flyCarForward() 
    if _G.flyCarActive and _G.flyCarBV then 
        _G.flyCarBV.Velocity = Camera.CFrame.LookVector * _G.flyCarSpeed 
    end 
end

function flyCarBackward() 
    if _G.flyCarActive and _G.flyCarBV then 
        _G.flyCarBV.Velocity = Camera.CFrame.LookVector * -_G.flyCarSpeed 
    end 
end

function flyCarStop() 
    if _G.flyCarBV then 
        _G.flyCarBV.Velocity = Vector3.new(0,0,0) 
    end 
end

-- [28.2] TOGGLE FLY CAR (CON TEXTO VERDE CUANDO ACTIVADO)
function toggleFlyCar(btn)
    if _G.flyCarActive then
        -- Desactivar Fly Car
        local h = gR()
        if h then 
            if _G.flyCarBV then 
                _G.flyCarBV:Destroy() 
                _G.flyCarBV = nil
            end 
            if _G.flyCarBG then 
                _G.flyCarBG:Destroy() 
                _G.flyCarBG = nil
            end 
        end
        _G.flyCarActive = false
        
        -- Ocultar botones grandes
        local gui = LP:FindFirstChild("PlayerGui"):FindFirstChild("FlyCarButtons")
        if gui then
            gui:Destroy()
        end
        
        if btn then 
            btn.Text = "OFF" 
            btn.BackgroundColor3 = Color3.fromRGB(200,150,120)  -- Color original
        end
    else
        -- Activar Fly Car
        local h = gR()
        if not h then 
            if btn then 
                btn.Text = "OFF" 
                btn.BackgroundColor3 = Color3.fromRGB(200,150,120)
            end
            return 
        end
        
        _G.flyCarBV = Instance.new("BodyVelocity", h)
        _G.flyCarBV.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
        _G.flyCarBV.Velocity = Vector3.new(0,0,0)
        
        _G.flyCarBG = Instance.new("BodyGyro", h)
        _G.flyCarBG.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
        _G.flyCarBG.CFrame = h.CFrame
        
        -- Mantener la orientación hacia la cámara
        spawn(function()
            while _G.flyCarActive and _G.flyCarBG and _G.flyCarBG.Parent do
                _G.flyCarBG.CFrame = Camera.CFrame
                RS.RenderStepped:Wait()
            end
        end)
        
        _G.flyCarActive = true
        
        -- Crear y mostrar botones grandes
        createFlyCarButtons()
        
        if btn then 
            btn.Text = "ON" 
            btn.BackgroundColor3 = Color3.fromRGB(100,200,100)  -- Verde cuando activado
        end
    end
end

-- [28.3] BOTONES GRANDES PARA FLY CAR (AMBOS AZULES, SIN TRANSPARENCIA)
function createFlyCarButtons()
    -- Eliminar GUI anterior si existe
    local oldGui = LP:FindFirstChild("PlayerGui"):FindFirstChild("FlyCarButtons")
    if oldGui then oldGui:Destroy() end
    
    -- Crear nuevo ScreenGui
    local gui = Instance.new("ScreenGui")
    gui.Name = "FlyCarButtons"
    gui.Parent = LP:FindFirstChild("PlayerGui")
    gui.ResetOnSpawn = false
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    gui.Enabled = true
    
    -- Determinar si es móvil
    local isMobile = UIS.TouchEnabled and not UIS.MouseEnabled
    local btnSize = isMobile and 90 or 70
    local bottomOffset = isMobile and 60 or 40
    local leftOffset = isMobile and 40 or 20  -- Posición izquierda
    
    -- Color azul para AMBAS flechas
    local azul = Color3.fromRGB(0, 120, 255)  -- Azul sólido
    
    -- Botón ADELANTE (azul)
    local fwdBtn = Instance.new("TextButton")
    fwdBtn.Name = "FlyCarForward"
    fwdBtn.Size = UDim2.new(0, btnSize, 0, btnSize)
    fwdBtn.Position = UDim2.new(0, leftOffset, 1, -(btnSize*2 + bottomOffset))
    fwdBtn.Text = "▲"
    fwdBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    fwdBtn.BackgroundColor3 = azul
    fwdBtn.BackgroundTransparency = 0  -- SIN TRANSPARENCIA
    fwdBtn.Font = Enum.Font.GothamBold
    fwdBtn.TextSize = 40
    fwdBtn.TextScaled = true
    fwdBtn.Parent = gui
    
    local fwdCorner = Instance.new("UICorner")
    fwdCorner.CornerRadius = UDim.new(0, 20)
    fwdCorner.Parent = fwdBtn
    
    -- Botón ATRÁS (azul también)
    local backBtn = Instance.new("TextButton")
    backBtn.Name = "FlyCarBack"
    backBtn.Size = UDim2.new(0, btnSize, 0, btnSize)
    backBtn.Position = UDim2.new(0, leftOffset, 1, -btnSize - bottomOffset)
    backBtn.Text = "▼"
    backBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    backBtn.BackgroundColor3 = azul
    backBtn.BackgroundTransparency = 0  -- SIN TRANSPARENCIA
    backBtn.Font = Enum.Font.GothamBold
    backBtn.TextSize = 40
    backBtn.TextScaled = true
    backBtn.Parent = gui
    
    local backCorner = Instance.new("UICorner")
    backCorner.CornerRadius = UDim.new(0, 20)
    backCorner.Parent = backBtn
    
    -- Eventos (feedback visual al presionar)
    fwdBtn.MouseButton1Down:Connect(function()
        flyCarForward()
        fwdBtn.BackgroundColor3 = Color3.fromRGB(0, 80, 200)  -- Azul más oscuro al presionar
    end)
    
    fwdBtn.MouseButton1Up:Connect(function()
        flyCarStop()
        fwdBtn.BackgroundColor3 = azul  -- Volver al azul original
    end)
    
    fwdBtn.MouseLeave:Connect(function()
        flyCarStop()
        fwdBtn.BackgroundColor3 = azul
    end)
    
    backBtn.MouseButton1Down:Connect(function()
        flyCarBackward()
        backBtn.BackgroundColor3 = Color3.fromRGB(0, 80, 200)  -- Azul más oscuro al presionar
    end)
    
    backBtn.MouseButton1Up:Connect(function()
        flyCarStop()
        backBtn.BackgroundColor3 = azul
    end)
    
    backBtn.MouseLeave:Connect(function()
        flyCarStop()
        backBtn.BackgroundColor3 = azul
    end)
    
    -- Eventos táctiles para móvil
    if isMobile then
        fwdBtn.TouchLongPress:Connect(function()
            flyCarForward()
        end)
        
        fwdBtn.TouchEnded:Connect(function()
            flyCarStop()
            fwdBtn.BackgroundColor3 = azul
        end)
        
        backBtn.TouchLongPress:Connect(function()
            flyCarBackward()
        end)
        
        backBtn.TouchEnded:Connect(function()
            flyCarStop()
            backBtn.BackgroundColor3 = azul
        end)
    end
    
    return gui
end

-- [29] TOGGLE FLY
function toggleFly(b)_G.isFlying=not _G.isFlying;local c=LP.Character if not c then return end local h,r=c:FindFirstChildOfClass("Humanoid"),c:FindFirstChild("HumanoidRootPart")or c:FindFirstChild("Torso")if not h or not r then return end if _G.isFlying then _G.originalWalkSpeed=h.WalkSpeed;_G.originalJumpPower=h.JumpPower for _,s in Enum.HumanoidStateType:GetEnumItems()do h:SetStateEnabled(s,false)end h:ChangeState(Enum.HumanoidStateType.Swimming)if _G.flyBG then _G.flyBG:Destroy()end if _G.flyBV then _G.flyBV:Destroy()end _G.flyBG=Instance.new("BodyGyro",r);_G.flyBG.MaxTorque=Vector3.new(9e9,9e9,9e9);_G.flyBV=Instance.new("BodyVelocity",r);_G.flyBV.MaxForce=Vector3.new(9e9,9e9,9e9);if _G.flyConnection then _G.flyConnection:Disconnect()end _G.flyConnection=RS.RenderStepped:Connect(function()if not _G.isFlying or not c or not c.Parent or not h or not h.Parent or not r or not r.Parent then if _G.flyConnection then _G.flyConnection:Disconnect()_G.flyConnection=nil end return end if _G.flyBG and _G.flyBG.Parent then _G.flyBG.CFrame=Camera.CoordinateFrame end if _G.flyBV and _G.flyBV.Parent then _G.flyBV.Velocity=h.MoveDirection.Magnitude>0 and h.MoveDirection*(45*_G.flySpeed)or Vector3.new(0,0,0)end end)if b then b.BackgroundColor3=Color3.fromRGB(150,180,150)end else if _G.flyConnection then _G.flyConnection:Disconnect()_G.flyConnection=nil end if _G.flyBG then pcall(function()_G.flyBG:Destroy()end)_G.flyBG=nil end if _G.flyBV then pcall(function()_G.flyBV:Destroy()end)_G.flyBV=nil end for _,s in Enum.HumanoidStateType:GetEnumItems()do pcall(function()h:SetStateEnabled(s,true)end)end pcall(function()h:ChangeState(Enum.HumanoidStateType.RunningNoPhysics)end)pcall(function()h.WalkSpeed=_G.originalWalkSpeed or 16;h.JumpPower=_G.originalJumpPower or 50 end)if r then pcall(function()r.Velocity=Vector3.new(0,0,0);r.RotVelocity=Vector3.new(0,0,0)end)end if b then b.BackgroundColor3=Color3.fromRGB(210,160,130)end end end
-- [30] ANIMATIONS
local Anims={Walk={Furry="102269417125238",Catwalk="109168724482748",Gojo="95643163365384",Geto="85811471336028",Adidas="122150855457006",Zombie="616168032",Ninja="656121766",Robot="10921250460",Stylish="10921276116",Sneaky="1132510133"},Run={Furry="102269417125238",Adidas="82598234841035",Heavy="3236836670",Cartoony="10921076136",Zombie="616163682",Ninja="656118852",Robot="10921250460",Sneaky="1132494274"},Idle={Furry={"102269417125238","102269417125238"},Astronaut={"891621366","891633237"},Bold={"16738333868","16738334710"},Zombie={"616158929","616160636"},Ninja={"656117400","656118341"},Stylish={"616136790","616138447"}},Jump={Furry="102269417125238",Adidas="75290611992385",Ninja="656117878",Robot="616090535",Stylish="616139451"},Fall={Furry="102269417125238",Adidas="98600215928904",Ninja="656115606",Robot="616087089"},Autos={[1]="76503595759461",[2]="115245341767944",[3]="127805235430271",[4]="138003068153218",[5]="116772752010894",[6]="116625361313832",[7]="81388785824317",[8]="108747312576405",[9]="113181071290859",[10]="134681712937413",[11]="115260380433565",[12]="72382226286301",[13]="507767082"}}
-- [31] SET ANIM
local function setAnim(t,n)local d=Anims[t]and Anims[t][n]if not d then return end local h=gH()if not h then return end local a=LP.Character and LP.Character:FindFirstChild("Animate")if not a then return end if _G.autoAnimTrack then _G.autoAnimTrack:Stop()_G.autoAnimTrack=nil end if t=="Autos"then _G.autoAnimActive=true local an=Instance.new("Animation");an.AnimationId="rbxassetid://"..d;local tr=h:LoadAnimation(an);tr.Priority=Enum.AnimationPriority.Action4;tr.Looped=true;tr:Play();_G.autoAnimTrack=tr;if _G.autoAnimConnection then _G.autoAnimConnection:Disconnect()end _G.autoAnimConnection=RS.RenderStepped:Connect(function()if _G.autoAnimActive and _G.autoAnimTrack and _G.autoAnimTrack.IsPlaying then elseif _G.autoAnimActive and _G.autoAnimTrack then _G.autoAnimTrack:Play()end end)if a then a.Disabled=true end else _G.autoAnimActive=false if _G.autoAnimConnection then _G.autoAnimConnection:Disconnect()_G.autoAnimConnection=nil end if a then a.Disabled=false end for _,tr in h:GetPlayingAnimationTracks()do tr:Stop()end pcall(function()if t=="Idle"and type(d)=="table"then if a.idle then a.idle.Animation1.AnimationId="http://www.roblox.com/asset/?id="..d[1];a.idle.Animation2.AnimationId="http://www.roblox.com/asset/?id="..d[2];_G.savedIdleAnim1=a.idle.Animation1.AnimationId;_G.savedIdleAnim2=a.idle.Animation2.AnimationId end elseif t=="Walk"and a.walk then a.walk.WalkAnim.AnimationId="http://www.roblox.com/asset/?id="..d elseif t=="Run"and a.run then a.run.RunAnim.AnimationId="http://www.roblox.com/asset/?id="..d elseif t=="Jump"and a.jump then a.jump.JumpAnim.AnimationId="http://www.roblox.com/asset/?id="..d elseif t=="Fall"and a.fall then a.fall.FallAnim.AnimationId="http://www.roblox.com/asset/?id="..d end end)end end
-- [32] SET IDLE ANIM
local function setIdleAnimation(n)local h=gH()if not h then return end local a=LP.Character and LP.Character:FindFirstChild("Animate")if not a then return end for _,t in pairs(h:GetPlayingAnimationTracks())do t:Stop()end if n=="Old School"then if a.idle then a.idle.Animation1.AnimationId="http://www.roblox.com/asset/?id=616136790";a.idle.Animation2.AnimationId="http://www.roblox.com/asset/?id=616138447"end elseif n=="Default"then if a.idle then a.idle.Animation1.AnimationId="http://www.roblox.com/asset/?id=616139622";a.idle.Animation2.AnimationId="http://www.roblox.com/asset/?id=616140386"end elseif n=="Furry"then if a.idle then a.idle.Animation1.AnimationId="http://www.roblox.com/asset/?id=102269417125238";a.idle.Animation2.AnimationId="http://www.roblox.com/asset/?id=102269417125238"end end end
-- [33] MAIN LOOP
RS.Heartbeat:Connect(function()safeCall(function()pcall(function()sethiddenproperty(LP,"SimulationRadius",math.huge);sethiddenproperty(LP,"MaxSimulationRadius",math.huge)end)if _G.ringPartsEnabled then local h=gR()if h then local c=h.Position for _,p in pairs(_G.tornadoParts)do if p and p.Parent and not p.Anchored then if p.Parent~=LP.Character and not p:IsDescendantOf(LP.Character)then p.CanCollide=false;local po=p.Position local d=(Vector3.new(po.X,c.Y,po.Z)-c).Magnitude local a=math.atan2(po.Z-c.Z,po.X-c.X)local na=a+math.rad(_G.tornadoConfig.rotationSpeed)local tp=Vector3.new(c.X+math.cos(na)*math.min(_G.tornadoConfig.radius,d),c.Y+(_G.tornadoConfig.height*(math.abs(math.sin((po.Y-c.Y)/_G.tornadoConfig.height)))),c.Z+math.sin(na)*math.min(_G.tornadoConfig.radius,d))local dir=(tp-po).unit p.Velocity=dir*_G.tornadoConfig.attractionStrength end end end end end updateCirclePosition()if not _G.ringPartsEnabled then if next(_G.multiSelect)and not _G.followCamera then updateMultiVelocity()elseif _G.selectedPart and _G.selectedPart.Parent and not _G.followCamera then updateVel(_G.selectedPart)end end if _G.followCamera and not _G.ringPartsEnabled then if next(_G.multiSelect)then for _,p in pairs(_G.multiSelect)do if p and p.Parent and LP.Character then clearForces(p)local r=LP.Character:FindFirstChild("HumanoidRootPart")or LP.Character:FindFirstChild("Torso")if r then local bp=Instance.new("BodyPosition",p);bp.Name="FollowBP";bp.MaxForce=Vector3.new(math.huge,math.huge,math.huge);bp.P=20000;bp.D=800;bp.Position=r.Position+(Camera.CFrame.LookVector*_G.followDistance);local bg=Instance.new("BodyGyro",p);bg.Name="FollowBG";bg.MaxTorque=Vector3.new(math.huge,math.huge,math.huge);bg.CFrame=p.CFrame end end end elseif _G.selectedPart and _G.selectedPart.Parent and LP.Character then clearForces(_G.selectedPart)local r=LP.Character:FindFirstChild("HumanoidRootPart")or LP.Character:FindFirstChild("Torso")if r then local bp=Instance.new("BodyPosition",_G.selectedPart);bp.Name="FollowBP";bp.MaxForce=Vector3.new(math.huge,math.huge,math.huge);bp.P=20000;bp.D=800;bp.Position=r.Position+(Camera.CFrame.LookVector*_G.followDistance);local bg=Instance.new("BodyGyro",_G.selectedPart);bg.Name="FollowBG";bg.MaxTorque=Vector3.new(math.huge,math.huge,math.huge);bg.CFrame=_G.initialRotation end end end end)end)
-- [34] CREATE HUB
local function CreateHub()
    local sg=Instance.new("ScreenGui",LP:WaitForChild("PlayerGui"));sg.Name="Whathedogdoin";sg.ResetOnSpawn=false
    local moon=Instance.new("TextButton",sg);moon.Name="MoonButton";moon.Size=UDim2.new(0,55,0,55);moon.Position=UDim2.new(0.02,0,0.5,-27.5);moon.BackgroundColor3=Color3.fromRGB(120,90,0);moon.Text="🧀\nRH";moon.TextColor3=Color3.fromRGB(255,255,0);moon.Font=Enum.Font.GothamBold;moon.TextSize=12;moon.Visible=false;moon.Draggable=true;Instance.new("UICorner",moon).CornerRadius=UDim.new(0,27.5)

local main=Instance.new("Frame",sg);main.Size=UDim2.new(0,300,0,540);main.Position=UDim2.new(0.5,-150,0.5,-270);main.BackgroundColor3=Color3.fromRGB(240,215,170);main.BorderSizePixel=0;main.Active=true;main.Draggable=true;main.Visible=false;main.Transparency=1;main.Rotation=-5;Instance.new("UICorner",main).CornerRadius=UDim.new(0,12)

-- BORDE GRUESO ALREDEDOR DEL MENÚ (CORREGIDO)
local borde = Instance.new("UIStroke", main)
borde.Thickness = 2  -- Aumenté el grosor para que se note más
borde.Color = Color3.fromRGB(170, 130, 90)
borde.Transparency = 0
borde.ApplyStrokeMode = Enum.ApplyStrokeMode.Border  -- Forzar que se aplique como borde

local scale=Instance.new("UIScale",main);scale.Scale=0.8

-- Botones de escala y cierre
local sd=createBtn(main,{0,28,0,28},{1,-65,0,50},"•",Color3.fromRGB(210,160,130))
local su=createBtn(main,{0,28,0,28},{1,-33,0,50},"×",Color3.fromRGB(210,160,130))
local close=createBtn(main,{0,28,0,28},{1,-98,0,50},"X",Color3.fromRGB(220,120,100))
    
    task.wait(0.1);main.Visible=true
    TS:Create(main,TweenInfo.new(0.5,Enum.EasingStyle.Back,Enum.EasingDirection.Out),{Transparency=0,Rotation=0}):Play()
    TS:Create(scale,TweenInfo.new(0.6,Enum.EasingStyle.Elastic,Enum.EasingDirection.Out),{Scale=1}):Play()
    
    local topBtn=Instance.new("TextButton",main);topBtn.Size=UDim2.new(0,160,0,30);topBtn.Position=UDim2.new(0.5,-80,0,8);topBtn.Text="▲ EXTRAS ▼";topBtn.TextColor3=Color3.fromRGB(255,255,255);topBtn.BackgroundColor3=Color3.fromRGB(180,130,90);topBtn.BorderSizePixel=0;topBtn.Font=Enum.Font.GothamBold;topBtn.TextSize=12;Instance.new("UICorner",topBtn).CornerRadius=UDim.new(0,8)
    
    local top=Instance.new("Frame",main);top.Size=UDim2.new(0,280,0,0);top.Position=UDim2.new(0.5,-140,0,-125);top.BackgroundColor3=Color3.fromRGB(220,190,150);top.BorderSizePixel=0;top.Visible=false;top.ClipsDescendants=true;Instance.new("UICorner",top).CornerRadius=UDim.new(0,10)

local topStroke = Instance.new("UIStroke", top)
topStroke.Thickness = 2
topStroke.Color = Color3.fromRGB(210, 160, 130)
topStroke.Transparency = 0
    
    local ty,th=10,0
    local rag=createBtn(top,{0.45,0,0,28},{0.05,0,0,ty},"😆 RAGDOLL",Color3.fromRGB(210,160,130))
    local inf=createBtn(top,{0.45,0,0,28},{0.5,0,0,ty},"🦘 INF JUMP",Color3.fromRGB(150,170,200))
    local noc=createBtn(top,{0.45,0,0,28},{0.05,0,0,ty+35},"🚪 NOCLIP",Color3.fromRGB(150,170,200))
    local esp=createBtn(top,{0.45,0,0,28},{0.5,0,0,ty+35},"👁️ ESP",Color3.fromRGB(150,170,200))
    local rst=createBtn(top,{0.45,0,0,28},{0.275,0,0,ty+70},"🔄 RESET",Color3.fromRGB(180,140,100))
    th=ty+105
    
   local title=Instance.new("TextLabel",main);title.Size=UDim2.new(0.75, -30, 0, 45);title.Position=UDim2.new(0.01, 0, 0, 45);title.Text="RIKESA HUB";title.TextColor3=Color3.fromRGB(255,255,255);title.BackgroundColor3=Color3.fromRGB(200,160,120);title.BorderSizePixel=0;title.Font=Enum.Font.GothamBold;title.TextSize=20;title.TextXAlignment=Enum.TextXAlignment.Left;Instance.new("UICorner",title).CornerRadius=UDim.new(0,10)

-- BORDE ALREDEDOR DEL TÍTULO
local titleBorder = Instance.new("UIStroke", title)
titleBorder.Thickness = 1  -- Grosor del borde
titleBorder.Color = Color3.fromRGB(150, 110, 70)  -- Color marrón oscuro
titleBorder.Transparency = 0

    local y,ws,jp=90,16,50
    
    local spdL=createLbl(main,{0.4,0,0,28},{0.1,0,0,y},"VEL: "..ws,Color3.fromRGB(200,120,80))
    local spdM=createBtn(main,{0,28,0,28},{0.55,0,0,y},"-",Color3.fromRGB(200,130,110))
    local spdP=createBtn(main,{0,28,0,28},{0.7,0,0,y},"+",Color3.fromRGB(150,180,150))
    spdM.MouseButton1Click:Connect(function()ws=math.max(10,ws-1);spdL.Text="VEL: "..ws;local h=gH()if h then h.WalkSpeed=ws end end)
    spdP.MouseButton1Click:Connect(function()ws=math.min(100,ws+1);spdL.Text="VEL: "..ws;local h=gH()if h then h.WalkSpeed=ws end end)
    
    y=y+33
    local jmpL=createLbl(main,{0.4,0,0,28},{0.1,0,0,y},"SALTO: "..jp,Color3.fromRGB(200,120,80))
    local jmpM=createBtn(main,{0,28,0,28},{0.55,0,0,y},"-",Color3.fromRGB(200,130,110))
    local jmpP=createBtn(main,{0,28,0,28},{0.7,0,0,y},"+",Color3.fromRGB(150,180,150))
    jmpM.MouseButton1Click:Connect(function()jp=math.max(20,jp-5);jmpL.Text="SALTO: "..jp;local h=gH()if h then h.JumpPower=jp end end)
    jmpP.MouseButton1Click:Connect(function()jp=math.min(250,jp+5);jmpL.Text="SALTO: "..jp;local h=gH()if h then h.JumpPower=jp end end)
    
    y=y+38
    local flySec=createFrame(main,{0.9,0,0,90},{0.05,0,0,y},Color3.fromRGB(220,190,150))
    createLbl(flySec,{1,0,0,22},{0,0,0,3},"✈️ VUELO",Color3.fromRGB(200,120,80),true)
    local flyB=createBtn(flySec,{0.3,0,0,32},{0.1,0,0,28},"FLY",Color3.fromRGB(210,160,130))
    local flySpd=createLbl(flySec,{0.15,0,0,32},{0.45,0,0,28},string.format("%.1f",_G.flySpeed),Color3.fromRGB(200,120,80))
    local flyM=createBtn(flySec,{0,24,0,24},{0.6,0,0,32},"-",Color3.fromRGB(200,130,110))
    local flyP=createBtn(flySec,{0,24,0,24},{0.7,0,0,32},"+",Color3.fromRGB(150,180,150))
    flyB.MouseButton1Click:Connect(function()toggleFly(flyB)end)
    flyM.MouseButton1Click:Connect(function()_G.flySpeed=math.max(0.1,_G.flySpeed-0.5);flySpd.Text=string.format("%.1f",_G.flySpeed)end)
    flyP.MouseButton1Click:Connect(function()_G.flySpeed=math.min(10,_G.flySpeed+0.5);flySpd.Text=string.format("%.1f",_G.flySpeed)end)
    
    y=y+100
    local flingSec=createFrame(main,{0.9,0,0,80},{0.05,0,0,y},Color3.fromRGB(220,190,150))
    createLbl(flingSec,{1,0,0,20},{0,0,0,3},"🌀 FLING",Color3.fromRGB(200,120,80),true)
   
    -- Botón FLING color beige oscuro (2px más largo)
local flingB = createBtn(flingSec, {0.6,0,0,27}, {0.2,0,0,26}, "⚡ FLING: OFF", Color3.fromRGB(200, 160, 120))
flingB.MouseButton1Click:Connect(function() toggleFlingArkineku(flingB) end)
    
    y=y+90
    local tpB=createBtn(main,{0.6,0,0,32},{0.2,0,0,y},"🚀 TELEPORT",Color3.fromRGB(150,170,200))
    tpB.MouseButton1Click:Connect(function()
        local closest, dist = nil, math.huge
        local myPos = gR() and gR().Position
        if not myPos then return end
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= LP and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                local pos = p.Character.HumanoidRootPart.Position
                local d = (myPos - pos).Magnitude
                if d < dist then
                    dist = d
                    closest = p
                end
            end
        end
        if closest and closest.Character then
            local r = closest.Character:FindFirstChild("HumanoidRootPart")
            local mr = gR()
            if r and mr then
                mr.CFrame = r.CFrame + Vector3.new(0,3,0)
            end
        end
    end)

-- Botón AntiFling (más bajo, 1px más abajo, mismo color que FLING)
local antiFlingBtn = Instance.new("TextButton")
antiFlingBtn.Size = UDim2.new(0.5, 0, 0, 20)  -- Más bajo (de 22 a 20)
antiFlingBtn.Position = UDim2.new(0.25, 0, 1, -25)  -- 1px más abajo (de -28 a -25)
antiFlingBtn.Text = "AntiFling: OFF"
antiFlingBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
antiFlingBtn.BackgroundColor3 = Color3.fromRGB(200, 160, 120)  -- MISMO COLOR QUE FLING
antiFlingBtn.BackgroundTransparency = 0
antiFlingBtn.Font = Enum.Font.GothamBold
antiFlingBtn.TextSize = 10
antiFlingBtn.Parent = flingSec

-- Borde redondeado
local antiCorner = Instance.new("UICorner")
antiCorner.CornerRadius = UDim.new(0, 6)
antiCorner.Parent = antiFlingBtn

-- Conectar funcionalidad
antiFlingBtn.MouseButton1Click:Connect(function() toggleAntiFling(antiFlingBtn) end)
    
    y=y+37
    local godBtn=createBtn(main,{0.6,0,0,32},{0.2,0,0,y},"🛡️ GOD MODE: OFF",Color3.fromRGB(200,150,180))
    godBtn.MouseButton1Click:Connect(function()toggleGodMode(godBtn)end)
    
    y=y+42
    local fcSec=createFrame(main,{0.9,0,0,70},{0.05,0,0,y},Color3.fromRGB(220,190,150))
    createLbl(fcSec,{1,0,0,22},{0,0,0,3},"🚗 FLY CAR",Color3.fromRGB(200,120,80),true)
    local fcB=createBtn(fcSec,{0.3,0,0,28},{0.1,0,0,28},"OFF",Color3.fromRGB(200,150,120))
    local fcSpd=createLbl(fcSec,{0.15,0,0,28},{0.45,0,0,28},tostring(_G.flyCarSpeed),Color3.fromRGB(200,120,80))
    local fcM=createBtn(fcSec,{0,24,0,24},{0.6,0,0,30},"-",Color3.fromRGB(200,130,110))
    local fcP=createBtn(fcSec,{0,24,0,24},{0.7,0,0,30},"+",Color3.fromRGB(150,180,150))

    fcB.MouseButton1Click:Connect(function()toggleFlyCar(fcB)end)
    fcM.MouseButton1Click:Connect(function()_G.flyCarSpeed=math.max(10,_G.flyCarSpeed-5);fcSpd.Text=tostring(_G.flyCarSpeed)end)
    fcP.MouseButton1Click:Connect(function()_G.flyCarSpeed=math.min(200,_G.flyCarSpeed+5);fcSpd.Text=tostring(_G.flyCarSpeed)end)

    y=y+75
    local trollBtn=createBtn(main,{0.9,0,0,30},{0.05,0,0,y},"👹 TROLL ▼",Color3.fromRGB(200,130,110))
    local troll=Instance.new("Frame",main);troll.Size=UDim2.new(0.9,0,0,0);troll.Position=UDim2.new(0.05,0,0,y+45);troll.BackgroundColor3=Color3.fromRGB(220,190,150);troll.BorderSizePixel=0;troll.Visible=false;troll.ClipsDescendants=true;Instance.new("UICorner",troll).CornerRadius=UDim.new(0,8)
    
    local ty2=5
    local bang=createBtn(troll,{0.45,0,0,28},{0.05,0,0,ty2},"💥 BANG",Color3.fromRGB(200,130,110))
    local face=createBtn(troll,{0.45,0,0,28},{0.5,0,0,ty2},"😳 FaceBang",Color3.fromRGB(150,170,200))
    local sit=createBtn(troll,{0.45,0,0,28},{0.05,0,0,ty2+30},"🪑 SIT",Color3.fromRGB(150,180,150))
    local suck=createBtn(troll,{0.45,0,0,28},{0.5,0,0,ty2+30},"👅 SUCK",Color3.fromRGB(200,150,180))
    local jerk=createBtn(troll,{0.45,0,0,28},{0.275,0,0,ty2+60},"🍆 JERK",Color3.fromRGB(180,140,200))
    local zeroG=createBtn(troll,{0.45,0,0,28},{0.275,0,0,ty2+90},"🪐 ZeroGravity: OFF",Color3.fromRGB(200,150,180))
    
    -- Controles de gravedad
    _G.zeroGravityValue = 0
    _G.zeroGravityDirection = 1
    
    local gravDisplay = Instance.new("TextBox", troll)
    gravDisplay.Size = UDim2.new(0, 30, 0, 28)
    gravDisplay.Position = UDim2.new(0.5, 75, 0, ty2+90)
    gravDisplay.BackgroundColor3 = Color3.fromRGB(255,255,255)
    gravDisplay.Text = "0"
    gravDisplay.TextColor3 = Color3.fromRGB(0,0,0)
    gravDisplay.Font = Enum.Font.GothamBold
    gravDisplay.TextSize = 20
    gravDisplay.TextScaled = true
    gravDisplay.ClearTextOnFocus = false
    Instance.new("UICorner", gravDisplay).CornerRadius = UDim.new(0, 4)
    
    local gravMinus = Instance.new("TextButton", troll)
    gravMinus.Size = UDim2.new(0, 20, 0, 20)
    gravMinus.Position = UDim2.new(0.5, 50, 0, ty2+94)
    gravMinus.Text = "-"
    gravMinus.TextColor3 = Color3.fromRGB(255,255,255)
    gravMinus.BackgroundColor3 = Color3.fromRGB(200,80,80)
    gravMinus.Font = Enum.Font.GothamBold
    gravMinus.TextSize = 16
    Instance.new("UICorner", gravMinus).CornerRadius = UDim.new(0, 4)
    
    local gravPlus = Instance.new("TextButton", troll)
    gravPlus.Size = UDim2.new(0, 20, 0, 20)
    gravPlus.Position = UDim2.new(0.5, 105, 0, ty2+94)
    gravPlus.Text = "+"
    gravPlus.TextColor3 = Color3.fromRGB(255,255,255)
    gravPlus.BackgroundColor3 = Color3.fromRGB(80,200,80)
    gravPlus.Font = Enum.Font.GothamBold
    gravPlus.TextSize = 16
    Instance.new("UICorner", gravPlus).CornerRadius = UDim.new(0, 4)
    
    local function updateDisplay()
        local displayValue = _G.zeroGravityValue * _G.zeroGravityDirection
        gravDisplay.Text = tostring(displayValue)
    end
    updateDisplay()
    
    gravMinus.MouseButton1Click:Connect(function()
        local current = _G.zeroGravityValue * _G.zeroGravityDirection
        local new = current - 5
        _G.zeroGravityValue = math.abs(new)
        _G.zeroGravityDirection = new < 0 and -1 or 1
        updateDisplay()
    end)
    
    gravPlus.MouseButton1Click:Connect(function()
        local current = _G.zeroGravityValue * _G.zeroGravityDirection
        local new = current + 5
        _G.zeroGravityValue = math.abs(new)
        _G.zeroGravityDirection = new < 0 and -1 or 1
        updateDisplay()
    end)
    
    gravDisplay.FocusLost:Connect(function(enterPressed)
        if enterPressed then
            local input = gravDisplay.Text:match("^%s*(.-)%s*$")
            local num = tonumber(input)
            if num then
                _G.zeroGravityValue = math.abs(num)
                _G.zeroGravityDirection = num < 0 and -1 or 1
            end
            updateDisplay()
        end
    end)
    
    -- Panel izquierdo (animaciones)
    local leftBtn=createBtn(main,{0,35,0,35},{0,-10,0.5,-17.5},"▶",Color3.fromRGB(180,130,90));leftBtn.TextSize=22;leftBtn.ZIndex=10
    local left=Instance.new("Frame",main);left.Size=UDim2.new(0,0,0,440);left.Position=UDim2.new(0,-255,0,90);left.BackgroundColor3=Color3.fromRGB(220,190,150);left.BorderSizePixel=0;left.Visible=false;left.ClipsDescendants=true;Instance.new("UICorner",left).CornerRadius=UDim.new(0,10)
    createLbl(left,{1,0,0,35},{0,0,0,0},"ANIMACIONES",Color3.fromRGB(200,120,80))

local leftStroke = Instance.new("UIStroke", left)
leftStroke.Thickness = 2
leftStroke.Color = Color3.fromRGB(210, 160, 130)
    
    local scroll=Instance.new("ScrollingFrame",left);scroll.Size=UDim2.new(0.9,0,0.85,0);scroll.Position=UDim2.new(0.05,0,0.1,0);scroll.BackgroundColor3=Color3.fromRGB(240,215,190);scroll.BorderSizePixel=0;scroll.ScrollBarThickness=6
    local ay=5
    local categorias={"Walk","Run","Idle","Jump","Fall","Autos","Extras"}
    for _,t in ipairs(categorias) do
        local an=Anims[t]
        if an then
            createLbl(scroll,{1,0,0,22},{0,0,0,ay},"=== "..t:upper().." ===",Color3.fromRGB(180,100,60),true)
            ay=ay+24
            for n,_ in pairs(an) do
                local nb=tostring(n)
                local b=createBtn(scroll,{0.9,0,0,24},{0.05,0,0,ay},nb,Color3.fromRGB(210,160,130))
                b.MouseButton1Click:Connect(function()setAnim(t,n)end)
                ay=ay+26
            end
            ay=ay+5
        end
    end
    scroll.CanvasSize=UDim2.new(0,0,0,ay)
    
    -- Panel derecho (move parts)
    local rightBtn=createBtn(main,{0,35,0,35},{1,-20,0.5,-17.5},"◀",Color3.fromRGB(180,130,90));rightBtn.TextSize=22;rightBtn.ZIndex=10
    local right=Instance.new("Frame",main);right.Size=UDim2.new(0,0,0,520);right.Position=UDim2.new(1,5,0,90);right.BackgroundColor3=Color3.fromRGB(220,190,150);right.BorderSizePixel=0;right.Visible=false;right.ClipsDescendants=true;Instance.new("UICorner",right).CornerRadius=UDim.new(0,10)
    createLbl(right,{1,0,0,35},{0,0,0,0},"MOVE PARTS",Color3.fromRGB(200,120,80))

local rightStroke = Instance.new("UIStroke", right)
rightStroke.Thickness = 2
rightStroke.Color = Color3.fromRGB(210, 160, 130)
    
    local ay2,bs=40,42
    local up=createBtn(right,{0,bs,0,bs},{0.5,-bs/2,0,ay2},"▲",Color3.fromRGB(210,160,130))
    local down=createBtn(right,{0,bs,0,bs},{0.5,-bs/2,0,ay2+bs+10},"▼",Color3.fromRGB(210,160,130))
    local leftBtn2=createBtn(right,{0,bs,0,bs},{0.5,-bs*1.5-10,0,ay2+bs/2},"◀",Color3.fromRGB(210,160,130))
    local rightBtn2=createBtn(right,{0,bs,0,bs},{0.5,bs/2+10,0,ay2+bs/2},"▶",Color3.fromRGB(210,160,130))
    local fwd=createBtn(right,{0,bs,0,bs},{0.5,-bs*1.5-10,0,ay2+bs*1.5+15},"+",Color3.fromRGB(150,180,150))
    local bwd=createBtn(right,{0,bs,0,bs},{0.5,bs/2+10,0,ay2+bs*1.5+15},"-",Color3.fromRGB(200,130,110))
    
    local objSpdL=createLbl(right,{1,0,0,22},{0,0,0,240},"⚡ OBJ: ".._G.floatSpeed,Color3.fromRGB(200,120,80),true)
    local objM=createBtn(right,{0,32,0,28},{0.3,0,0,265},"-",Color3.fromRGB(200,130,110))
    local objP=createBtn(right,{0,32,0,28},{0.6,0,0,265},"+",Color3.fromRGB(150,180,150))
    local sel=createBtn(right,{0.4,0,0,32},{0.05,0,0,300},"SELECCIONAR",Color3.fromRGB(210,160,130))
    local stop=createBtn(right,{0.4,0,0,32},{0.55,0,0,300},"DETENER",Color3.fromRGB(200,130,110))
    local follow=createBtn(right,{0.6,0,0,32},{0.2,0,0,340},"◯ SEGUIR: OFF",Color3.fromRGB(210,160,130))
    
    local tornadoBtn=createBtn(right,{0.6,0,0,32},{0.2,0,0,420},"🌪️ RingParts: OFF",Color3.fromRGB(210,160,130))
    local tornadoRadiusLabel=createLbl(right,{1,0,0,22},{0,0,0,450},"⚡ RADIUS: ".._G.tornadoConfig.radius,Color3.fromRGB(200,120,80),true)
    local tornadoRadiusSlider=Instance.new("Frame",right);tornadoRadiusSlider.Size=UDim2.new(0.8,0,0,12);tornadoRadiusSlider.Position=UDim2.new(0.1,0,0,470);tornadoRadiusSlider.BackgroundColor3=Color3.fromRGB(200,150,120);tornadoRadiusSlider.BorderSizePixel=0;Instance.new("UICorner",tornadoRadiusSlider).CornerRadius=UDim.new(0,6)
    local tornadoRadiusCircle=Instance.new("TextButton",tornadoRadiusSlider);tornadoRadiusCircle.Size=UDim2.new(0,16,0,16);tornadoRadiusCircle.Position=UDim2.new((_G.tornadoConfig.radius-10)/990,-8,0.5,-8);tornadoRadiusCircle.Text="";tornadoRadiusCircle.BackgroundColor3=Color3.fromRGB(255,215,190);tornadoRadiusCircle.BorderSizePixel=0;Instance.new("UICorner",tornadoRadiusCircle).CornerRadius=UDim.new(0,8)
    
    local distL=createLbl(right,{1,0,0,22},{0,0,0,375},"🎯 ".._G.followDistance.."m",Color3.fromRGB(200,120,80),true)
    local slider=Instance.new("Frame",right);slider.Size=UDim2.new(0.8,0,0,12);slider.Position=UDim2.new(0.1,0,0,400);slider.BackgroundColor3=Color3.fromRGB(200,150,120);slider.BorderSizePixel=0;Instance.new("UICorner",slider).CornerRadius=UDim.new(0,6)
    local circle=Instance.new("TextButton",slider);circle.Size=UDim2.new(0,16,0,16);circle.Position=UDim2.new((_G.followDistance-5)/145,-8,0.5,-8);circle.Text="";circle.BackgroundColor3=Color3.fromRGB(255,215,190);circle.BorderSizePixel=0;Instance.new("UICorner",circle).CornerRadius=UDim.new(0,8)
    
    right.Size=UDim2.new(0,300,0,500)  -- Más angosto y un poco más corto
    
    -- Variables de estado para desplegables
    local te,le,re,tre = false,false,false,false
    local selActive,drag = false,false
    local tornadoRadiusDrag = false
    
    -- Conexiones de los desplegables
    topBtn.MouseButton1Click:Connect(function()
        te = not te
        topBtn.Text = te and "▼ EXTRAS ▲" or "▲ EXTRAS ▼"
        if te then
            top.Visible = true
            TS:Create(top, TweenInfo.new(0.3), {Size = UDim2.new(0,280,0,th)}):Play()
        else
            TS:Create(top, TweenInfo.new(0.3), {Size = UDim2.new(0,280,0,0)}):Play()
            task.wait(0.3)
            top.Visible = false
        end
    end)
    
    leftBtn.MouseButton1Click:Connect(function()
        le = not le
        leftBtn.Text = le and "◀" or "▶"
        if le then
            left.Visible = true
            TS:Create(left, TweenInfo.new(0.3), {Size = UDim2.new(0,250,0,440)}):Play()
        else
            TS:Create(left, TweenInfo.new(0.3), {Size = UDim2.new(0,0,0,440)}):Play()
            task.wait(0.3)
            left.Visible = false
        end
    end)
    
    rightBtn.MouseButton1Click:Connect(function()
        re = not re
        rightBtn.Text = re and "▶" or "◀"
        if re then
            right.Visible = true
            TS:Create(right, TweenInfo.new(0.3), {Size = UDim2.new(0,300,0,500)}):Play()
        else
            TS:Create(right, TweenInfo.new(0.3), {Size = UDim2.new(0,0,0,500)}):Play()
            task.wait(0.3)
            right.Visible = false
        end
    end)
    
    trollBtn.MouseButton1Click:Connect(function()
        tre = not tre
        trollBtn.Text = tre and "👹 TROLL ▲" or "👹 TROLL ▼"
        if tre then
            troll.Visible = true
            TS:Create(troll, TweenInfo.new(0.3), {Size = UDim2.new(0.9,0,0,125)}):Play()
        else
            TS:Create(troll, TweenInfo.new(0.3), {Size = UDim2.new(0.9,0,0,0)}):Play()
            task.wait(0.3)
            troll.Visible = false
        end
    end)

local trollStroke = Instance.new("UIStroke", troll)
trollStroke.Thickness = 2
trollStroke.Color = Color3.fromRGB(210, 160, 130)
    
    -- Conexiones de los botones de escala y cierre
    sd.MouseButton1Click:Connect(function()
        scale.Scale = math.max(0.5, scale.Scale - 0.1)
    end)
    
    su.MouseButton1Click:Connect(function()
        scale.Scale = math.min(1.5, scale.Scale + 0.1)
    end)
    
    close.MouseButton1Click:Connect(function()
        _G.hiddenfling = false
        _G.isFlying = false
        main.Visible = false
        moon.Visible = true
        moon.Size = UDim2.new(0,0,0,0)
        TS:Create(moon, TweenInfo.new(0.3), {Size = UDim2.new(0,55,0,55)}):Play()
    end)
    
    moon.MouseButton1Click:Connect(function()
        main.Visible = true
        moon.Visible = false
    end)
    
    -- Conexiones de los otros botones
    rag.MouseButton1Click:Connect(function() toggleRagdoll(rag) end)
    inf.MouseButton1Click:Connect(function() toggleInfiniteJump(inf) end)
    noc.MouseButton1Click:Connect(function() toggleNoclip(noc) end)
    esp.MouseButton1Click:Connect(function() toggleESP(esp) end)
    rst.MouseButton1Click:Connect(function() resetStats(rst) end)
    
    bang.MouseButton1Click:Connect(function() toggleBang(bang) end)
    face.MouseButton1Click:Connect(function() toggleFaceBang(face) end)
    sit.MouseButton1Click:Connect(function() toggleSit(sit) end)
    suck.MouseButton1Click:Connect(function() toggleSuck(suck) end)
    jerk.MouseButton1Click:Connect(function() toggleJerk(jerk) end)
    zeroG.MouseButton1Click:Connect(function() toggleZeroG(zeroG) end)
    
    -- Botón TP back
    local tpBackBtn = Instance.new("TextButton", main)
    tpBackBtn.Size = UDim2.new(0, 70, 0, 24)  -- 60% más pequeño aprox
    tpBackBtn.Position = UDim2.new(0.5, -65, 0.6, 0)  
    tpBackBtn.Text = " TP BACK"
    tpBackBtn.TextColor3 = Color3.fromRGB(255,255,255)
    tpBackBtn.BackgroundColor3 = Color3.fromRGB(200,0,0)
    tpBackBtn.Font = Enum.Font.GothamBold
    tpBackBtn.TextSize = 16
    tpBackBtn.Visible = false
    tpBackBtn.ZIndex = 10
    Instance.new("UICorner", tpBackBtn).CornerRadius = UDim.new(0, 8)
    
    local deathPosition = nil
    
    local function onCharacterAdded(char)
        local hum = char:WaitForChild("Humanoid")
        hum.Died:Connect(function()
            local root = char:FindFirstChild("HumanoidRootPart")
            if root then
                deathPosition = root.Position
                tpBackBtn.Visible = true
            end
        end)
    end
    
    if LP.Character then onCharacterAdded(LP.Character) end
    LP.CharacterAdded:Connect(onCharacterAdded)
    
    tpBackBtn.MouseButton1Click:Connect(function()
        if deathPosition and LP.Character and LP.Character:FindFirstChild("HumanoidRootPart") then
            LP.Character.HumanoidRootPart.CFrame = CFrame.new(deathPosition)
            tpBackBtn.Visible = false
            deathPosition = nil
        end
    end)
    
    -- Conexiones de move parts
    sel.MouseButton1Click:Connect(function()
        selActive = not selActive
        sel.BackgroundColor3 = selActive and Color3.fromRGB(150,180,150) or Color3.fromRGB(210,160,130)
        sel.Text = selActive and "ACTIVO" or "SELECCIONAR"
    end)
    
    stop.MouseButton1Click:Connect(function()
        _G.moveDirection = Vector3.new(0,0,0)
        if _G.selectedPart and _G.selectedPart.Parent then processPart(_G.selectedPart) end
    end)
    
    up.MouseButton1Click:Connect(function()
        _G.moveDirection = getCamDir(Vector3.new(0,1,0))
        if next(_G.multiSelect) then processMultiPart() elseif _G.selectedPart then processPart(_G.selectedPart) end
    end)
    
    down.MouseButton1Click:Connect(function()
        _G.moveDirection = getCamDir(Vector3.new(0,-1,0))
        if next(_G.multiSelect) then processMultiPart() elseif _G.selectedPart then processPart(_G.selectedPart) end
    end)
    
    leftBtn2.MouseButton1Click:Connect(function()
        _G.moveDirection = getCamDir(Vector3.new(-1,0,0))
        if next(_G.multiSelect) then processMultiPart() elseif _G.selectedPart then processPart(_G.selectedPart) end
    end)
    
    rightBtn2.MouseButton1Click:Connect(function()
        _G.moveDirection = getCamDir(Vector3.new(1,0,0))
        if next(_G.multiSelect) then processMultiPart() elseif _G.selectedPart then processPart(_G.selectedPart) end
    end)
    
    fwd.MouseButton1Click:Connect(function()
        _G.moveDirection = getCamDir(Vector3.new(0,0,1))
        if next(_G.multiSelect) then processMultiPart() elseif _G.selectedPart then processPart(_G.selectedPart) end
    end)
    
    bwd.MouseButton1Click:Connect(function()
        _G.moveDirection = getCamDir(Vector3.new(0,0,-1))
        if next(_G.multiSelect) then processMultiPart() elseif _G.selectedPart then processPart(_G.selectedPart) end
    end)
    
    objM.MouseButton1Click:Connect(function()
        _G.floatSpeed = math.max(5, _G.floatSpeed-5)
        objSpdL.Text = "⚡ OBJ: ".._G.floatSpeed
        if next(_G.multiSelect) then processMultiPart() elseif _G.selectedPart then processPart(_G.selectedPart) end
    end)
    
    objP.MouseButton1Click:Connect(function()
        _G.floatSpeed = _G.floatSpeed+5
        objSpdL.Text = "⚡ OBJ: ".._G.floatSpeed
        if next(_G.multiSelect) then processMultiPart() elseif _G.selectedPart then processPart(_G.selectedPart) end
    end)
    
    follow.MouseButton1Click:Connect(function()
        _G.followCamera = not _G.followCamera
        follow.Text = _G.followCamera and "✓ SEGUIR: ON" or "◯ SEGUIR: OFF"
        follow.BackgroundColor3 = _G.followCamera and Color3.fromRGB(150,180,150) or Color3.fromRGB(210,160,130)
        if _G.selectedPart then
            if _G.followCamera then
                _G.initialRotation = _G.selectedPart.CFrame
                clearForces(_G.selectedPart)
            else
                processPart(_G.selectedPart)
            end
        end
    end)
    
    circle.MouseButton1Down:Connect(function() drag = true end)
    tornadoRadiusCircle.MouseButton1Down:Connect(function() tornadoRadiusDrag = true end)
    
    UIS.InputChanged:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseMovement or i.UserInputType == Enum.UserInputType.Touch then
            if drag then
                local x = i.Position.X
                local ap, sz = slider.AbsolutePosition, slider.AbsoluteSize
                local r = math.clamp((x - ap.X) / sz.X, 0, 1)
                local nd = math.floor(5 + r * 145)
                _G.followDistance = nd
                circle.Position = UDim2.new(r, -8, 0.5, -8)
                distL.Text = "🎯 "..nd.."m"
            end
            if tornadoRadiusDrag then
                local x = i.Position.X
                local ap, sz = tornadoRadiusSlider.AbsolutePosition, tornadoRadiusSlider.AbsoluteSize
                local r = math.clamp((x - ap.X) / sz.X, 0, 1)
                local nr = math.floor(10 + r * 990)
                _G.tornadoConfig.radius = nr
                tornadoRadiusCircle.Position = UDim2.new(r, -8, 0.5, -8)
                tornadoRadiusLabel.Text = "⚡ RADIO: "..nr
            end
        end
    end)
    
    UIS.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch then
            drag = false
            tornadoRadiusDrag = false
        end
    end)
    
    tornadoBtn.MouseButton1Click:Connect(function() toggleTornado(tornadoBtn) end)
    
    UIS.InputBegan:Connect(function(i)
        if selActive and (i.UserInputType == Enum.UserInputType.MouseButton1 or i.UserInputType == Enum.UserInputType.Touch) then
            local r = Camera:ViewportPointToRay(i.Position.X, i.Position.Y)
            local p = WS:FindPartOnRay(Ray.new(r.Origin, r.Direction * 1000), LP.Character)
            if p and not p.Anchored then
                if _G.selectedHighlight then _G.selectedHighlight:Destroy() end
                _G.selectedPart = p
                _G.initialRotation = p.CFrame
                local h = Instance.new("SelectionBox", p)
                h.Adornee = p
                h.Color3 = Color3.fromRGB(200,120,80)
                _G.selectedHighlight = h
                processPart(p)
                selActive = false
                sel.BackgroundColor3 = Color3.fromRGB(210,160,130)
                sel.Text = "SELECCIONAR"
            end
        end
    end)
end
-- [35] UI HELPERS
function createBtn(p,s,po,t,c)local b=Instance.new("TextButton",p);b.Size=UDim2.new(s[1],s[2],s[3],s[4]);b.Position=UDim2.new(po[1],po[2],po[3],po[4]);b.Text=t;b.TextColor3=Color3.fromRGB(255,255,255);b.BackgroundColor3=c;b.BorderSizePixel=0;b.Font=Enum.Font.GothamBold;b.TextSize=10;Instance.new("UICorner",b).CornerRadius=UDim.new(0,6);return b end
function createLbl(p,s,po,t,tc,trans)local l=Instance.new("TextLabel",p);l.Size=UDim2.new(s[1],s[2],s[3],s[4]);l.Position=UDim2.new(po[1],po[2],po[3],po[4]);l.Text=t;l.TextColor3=tc;l.BackgroundColor3=Color3.fromRGB(235,200,170);l.BorderSizePixel=0;l.Font=Enum.Font.GothamBold;l.TextSize=12;if trans then l.BackgroundTransparency=1 end;Instance.new("UICorner",l).CornerRadius=UDim.new(0,6);return l end
function createFrame(p,s,po,c)local f=Instance.new("Frame",p);f.Size=UDim2.new(s[1],s[2],s[3],s[4]);f.Position=UDim2.new(po[1],po[2],po[3],po[4]);f.BackgroundColor3=c;f.BorderSizePixel=0;Instance.new("UICorner",f).CornerRadius=UDim.new(0,10);return f end
function createBox(p,s,po,t,ph)local b=Instance.new("TextBox",p);b.Size=UDim2.new(s[1],s[2],s[3],s[4]);b.Position=UDim2.new(po[1],po[2],po[3],po[4]);b.Text=t;b.PlaceholderText=ph or "";b.TextColor3=Color3.fromRGB(0,0,0);b.PlaceholderColor3=Color3.fromRGB(150,100,70);b.BackgroundColor3=Color3.fromRGB(255,255,255);b.BorderSizePixel=0;b.Font=Enum.Font.Gotham;b.TextSize=12;Instance.new("UICorner",b).CornerRadius=UDim.new(0,6);return b end
CreateHub()
