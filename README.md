-- EGG KAITUN V114 - EXACT V113 BASE + PICKUP FORENSIC + AIR1000
-- EXACT V85/V93-STYLE DYNAMIC REPLAY
-- Diagnóstico: preserva pickup do V113 e rastreia passivamente a aproximação/pickup. One death = STOP.

local Players=game:GetService("Players")
local RS=game:GetService("ReplicatedStorage")
local WS=game:GetService("Workspace")
local RunService=game:GetService("RunService")
local TweenService=game:GetService("TweenService")
local HttpService=game:GetService("HttpService")
local LP=Players.LocalPlayer

local CFG={
 Chicken=Vector3.new(545,71,-365),
 ForestTrigger=Vector3.new(592.974,70.683,-332.401),
 ChickenSpeed=120, FlightSpeed=1000,
 ChickenAfterWait=.06, ForestAfterTPWait=.12, AskPostWait=.70,
 ForestPromptOffsets={0,.067143,.114738},
 BeginTimeout=6, EggTPHeight=3,
 PromptLeadBeforeEnd=.159, EscapeAfterLastPrompt=.180,
 TargetPromptOffsets={0,.055519,.208221,.292291,.358595,.414449,.508160,.610433,.671248,.778416,.988710,1.058775,1.157651,1.310894},
 TargetPromptSearchRadius=18, PickupToFlightDelay=.12, BaseOffsetY=2.5, BaseAirHeight=28.3,
 BaseHorizontalArrival=32, BaseFallTimeout=3.2, BaseAirArrival=20,
 RollbackJumpDistance=220, RollbackDistanceIncrease=170,
 AutoRepeat=true,CycleDelay=.30,ReportMovementInterval=.15,MaxMovementSamples=220
}

local S={Running=true,Dead=false,Character=nil,Humanoid=nil,Root=nil,CurrentTween=nil,
 BasePlot=nil,BaseReturn=nil,Mode="INIT",Target=nil,PendingAuth=false,AuthDone=false,
 AuthTargetPosition=nil,AuthBeginAt=0,AuthDuration=2.5,Ragdoll=false,RagdollCount=0,
 FirstBeginAt=0,FirstEndAt=0,CachedTargetPrompt=nil,PromptFireCount=0,
 FlightActive=false,FlightInterrupted=false,FlightRollback=false,FlightSuccess=false,
 FlightFailure=nil,FlightGeneration=0,AskDuration=0,AskResult=nil,Relocates=0,
 Events={},Movement={},StartClock=os.clock(),LastArea=nil}

local function now() return os.clock()-S.StartClock end
local function vec(v) if typeof(v)~="Vector3" then return "nil" end return string.format("(%.3f, %.3f, %.3f)",v.X,v.Y,v.Z) end
local function alive() return not S.Dead and S.Character and S.Character.Parent and S.Humanoid and S.Humanoid.Parent and S.Humanoid.Health>0 and S.Root and S.Root.Parent end
local function state() if not S.Humanoid then return "nil" end local ok,v=pcall(function() return S.Humanoid:GetState() end) return ok and tostring(v) or "nil" end
local function area() for _,h in ipairs({LP,S.Character}) do if h then for _,k in ipairs({"AreaId","Area","CurrentArea","Zone","ZoneId"}) do local ok,v=pcall(function() return h:GetAttribute(k) end) if ok and v~=nil then return tostring(v) end end end end return "nil" end
local function context() if not alive() then return "dead/root=nil" end return string.format("pos=%s | area=%s | state=%s | speed=%.1f | hp=%.1f | rag=%s",vec(S.Root.Position),area(),state(),S.Root.AssemblyLinearVelocity.Magnitude,S.Humanoid.Health,tostring(S.Ragdoll)) end
local function log(n,d) d=tostring(d or ""); S.Events[#S.Events+1]={t=now(),name=n,detail=d}; print(string.format("[V114 %.6f] %-28s | %s",now(),n,d)) end
local function report()
 local l={"==========================================================================","V114 - EXACT V113 BASE + PICKUP FORENSIC + AIR1000","==========================================================================",string.format("Duration: %.6f",now()),"Final: "..context(),string.format("Mode=%s | Dead=%s | Relocates=%d | Ragdolls=%d | AskDuration=%.6f | AskResult=%s | PromptFires=%d",S.Mode,tostring(S.Dead),S.Relocates,S.RagdollCount,S.AskDuration,tostring(S.AskResult),S.PromptFireCount)}
 if S.Target then l[#l+1]=string.format("TARGET uid=%s | asset=%s | rate=%s | area=%s | pos=%s",S.Target.Key,S.Target.Asset,S.Target.Rate,S.Target.Area,vec(S.Target.Position)) end
 l[#l+1]="";l[#l+1]="TIMELINE"
 for _,e in ipairs(S.Events) do l[#l+1]=string.format("[%.6f] %-28s | %s",e.t,e.name,e.detail) end
 l[#l+1]="";l[#l+1]="MOVEMENT"
 for i,m in ipairs(S.Movement) do l[#l+1]=string.format("#%04d [%.6f] pos=%s | area=%s | state=%s | speed=%.1f | mode=%s",i,m.t,vec(m.pos),m.area,m.state,m.speed,m.mode) end
 l[#l+1]="==========================================================================";l[#l+1]="END V113"
 return table.concat(l,"\n")
end
local function copyReport() local r=report();print(r);if type(setclipboard)=="function" then pcall(setclipboard,r) elseif type(toclipboard)=="function" then pcall(toclipboard,r) end end

pcall(function() local pg=LP:FindFirstChild("PlayerGui");if pg then local x=pg:FindFirstChild("EggKaitunV110");if x then x:Destroy() end end end)
local Gui=Instance.new("ScreenGui");Gui.Name="EggKaitunV110";Gui.ResetOnSpawn=false;Gui.IgnoreGuiInset=true;Gui.Parent=LP:WaitForChild("PlayerGui")
local Frame=Instance.new("Frame");Frame.Size=UDim2.fromOffset(390,172);Frame.Position=UDim2.new(.5,-195,0,28);Frame.BackgroundColor3=Color3.fromRGB(18,18,22);Frame.BorderSizePixel=0;Frame.Active=true;Frame.Draggable=true;Frame.Parent=Gui
Instance.new("UICorner",Frame).CornerRadius=UDim.new(0,12)
local Title=Instance.new("TextLabel",Frame);Title.BackgroundTransparency=1;Title.Size=UDim2.new(1,-20,0,28);Title.Position=UDim2.fromOffset(10,7);Title.Font=Enum.Font.GothamBold;Title.TextSize=15;Title.TextColor3=Color3.new(1,1,1);Title.TextXAlignment=Enum.TextXAlignment.Left;Title.Text="🥚 V114 • PICKUP FORENSIC + AIR1000"
local Status=Instance.new("TextLabel",Frame);Status.BackgroundTransparency=1;Status.Size=UDim2.new(1,-20,0,55);Status.Position=UDim2.fromOffset(10,39);Status.Font=Enum.Font.GothamSemibold;Status.TextSize=13;Status.TextColor3=Color3.fromRGB(235,235,240);Status.TextWrapped=true;Status.TextXAlignment=Enum.TextXAlignment.Left;Status.TextYAlignment=Enum.TextYAlignment.Top;Status.Text="STATUS: INICIANDO"
local TargetLabel=Instance.new("TextLabel",Frame);TargetLabel.BackgroundTransparency=1;TargetLabel.Size=UDim2.new(1,-20,0,25);TargetLabel.Position=UDim2.fromOffset(10,96);TargetLabel.Font=Enum.Font.Gotham;TargetLabel.TextSize=12;TargetLabel.TextColor3=Color3.fromRGB(200,200,205);TargetLabel.TextXAlignment=Enum.TextXAlignment.Left;TargetLabel.Text="ALVO: --"
local Copy=Instance.new("TextButton",Frame);Copy.Size=UDim2.fromOffset(105,30);Copy.Position=UDim2.new(1,-225,1,-40);Copy.BackgroundColor3=Color3.fromRGB(50,90,145);Copy.Font=Enum.Font.GothamBold;Copy.TextSize=12;Copy.TextColor3=Color3.new(1,1,1);Copy.Text="COPIAR LOG";Instance.new("UICorner",Copy).CornerRadius=UDim.new(0,8)
local Stop=Instance.new("TextButton",Frame);Stop.Size=UDim2.fromOffset(105,30);Stop.Position=UDim2.new(1,-112,1,-40);Stop.BackgroundColor3=Color3.fromRGB(155,45,45);Stop.Font=Enum.Font.GothamBold;Stop.TextSize=12;Stop.TextColor3=Color3.new(1,1,1);Stop.Text="PARAR";Instance.new("UICorner",Stop).CornerRadius=UDim.new(0,8)
local function status(t) Status.Text="STATUS: "..tostring(t);print("[V114] "..tostring(t)) end
local function targetText(t) TargetLabel.Text="ALVO: "..tostring(t) end

local Char=LP.Character or LP.CharacterAdded:Wait();local Hum=Char:WaitForChild("Humanoid");local Root=Char:WaitForChild("HumanoidRootPart");S.Character=Char;S.Humanoid=Hum;S.Root=Root
local function zero() if alive() then S.Root.AssemblyLinearVelocity=Vector3.zero;S.Root.AssemblyAngularVelocity=Vector3.zero end end
local function cancel() if S.CurrentTween then pcall(function() S.CurrentTween:Cancel() end) end;S.CurrentTween=nil end
local function stop(r) if not S.Running then return end;S.Running=false;cancel();S.Mode="STOPPED";status(r or "PARADO");task.defer(function() task.wait(.05);copyReport() end) end
Hum.Died:Connect(function()if not S.Dead then S.Dead=true;log("DIED","uma morte -> STOP");stop("MORREU - DESLIGADO")end end)

local function fullname(o) if not o then return "nil" end local ok,v=pcall(function()return o:GetFullName()end)return ok and v or tostring(o)end
local function wpos(o) if not o then return nil end;if o:IsA("BasePart")then return o.Position elseif o:IsA("Attachment")then return o.WorldPosition elseif o:IsA("Model")then local ok,p=pcall(function()return o:GetPivot()end);if ok then return p.Position end end;local p=o:FindFirstChildWhichIsA("BasePart",true);return p and p.Position end
local function remote(q,c) q=q:lower();for _,o in ipairs(RS:GetDescendants())do if not c or o.ClassName==c then local n=o.Name:lower();local f=fullname(o):lower();if n:find(q,1,true)or f:find(q,1,true)then return o end end end end
local function tp(p)if not alive()then return false end;S.Root.CFrame=CFrame.new(p);zero();return true end

local function plotCenter(p)local x=p and p:FindFirstChild("GridCenter",true);return(x and wpos(x))or wpos(p)end
local function detectBase()
 local assets=WS:FindFirstChild("ClientRenderedAssets");local plots=WS:FindFirstChild("Plots");if not plots then return end
 local c={}
 for _,p in ipairs(plots:GetChildren())do local q=plotCenter(p);if q then c[#c+1]={Plot=p,Position=q,Votes=0,TotalDistance=0}end end
 for _,a in ipairs(assets and assets:GetChildren()or{})do if tonumber(a:GetAttribute("OwnerUserId"))==LP.UserId then local q=wpos(a);if q then local b,bd=nil,math.huge;for _,x in ipairs(c)do local d=(x.Position-q).Magnitude;if d<bd then b=x;bd=d end end;if b then b.Votes+=1;b.TotalDistance+=bd end end end end
 local b
 for _,x in ipairs(c)do if x.Votes>0 and(not b or x.Votes>b.Votes or(x.Votes==b.Votes and x.TotalDistance/x.Votes<b.TotalDistance/b.Votes))then b=x end end
 if not b then local bd=math.huge;for _,x in ipairs(c)do local d=(x.Position-Root.Position).Magnitude;if d<bd then b=x;bd=d end end end
 if not b then return end
 local tu=b.Plot:FindFirstChild("ToUpdate");local sp=tu and tu:FindFirstChild("StarterPen");local gc=sp and sp:FindFirstChild("GridCenter");local ret=(gc and wpos(gc))or b.Position;ret+=Vector3.new(0,CFG.BaseOffsetY,0)
 S.BasePlot=b.Plot;S.BaseReturn=ret;log("BASE_FOUND",string.format("%s | votes=%d | return=%s",b.Plot.Name,b.Votes,vec(ret)));return b.Plot,ret
end

local SnapshotRF=remote("AskFieldEggSnapshot","RemoteFunction")
local CarryRF=remote("AskFieldEggCarry","RemoteFunction")
local RefreshRE
for _,o in ipairs(RS:GetDescendants())do if o:IsA("RemoteEvent")then local f=fullname(o):lower();if f:find("rigsync",1,true)and f:find("refresh",1,true)then RefreshRE=o;break end end end
log("REMOTES",string.format("Snapshot=%s | Carry=%s | Refresh=%s",fullname(SnapshotRF),fullname(CarryRF),fullname(RefreshRE)))

local function snap()
 if not SnapshotRF or not SnapshotRF.Parent then SnapshotRF=remote("AskFieldEggSnapshot","RemoteFunction")end;if not SnapshotRF then return end
 local ok,p=pcall(function()return table.pack(SnapshotRF:InvokeServer())end);if not ok then return end
 for i=1,p.n do local v=p[i];if type(v)=="table"and type(v.Records)=="table"then return v elseif type(v)=="string"and(v:sub(1,1)=="{"or v:sub(1,1)=="[")then local a,d=pcall(function()return HttpService:JSONDecode(v)end);if a and type(d)=="table"and type(d.Records)=="table"then return d end end end
end
local function rpos(r)if type(r)~="table"then return end;if typeof(r.BottomCFrame)=="CFrame"then return r.BottomCFrame.Position end;if typeof(r.BoundsCFrame)=="CFrame"then return r.BoundsCFrame.Position end;if typeof(r.CFrame)=="CFrame"then return r.CFrame.Position end;return typeof(r.Position)=="Vector3"and r.Position end
local function first(r,k)for _,x in ipairs(k)do if r[x]~=nil then return r[x]end end end
local cache={}
local function cfg(cat)if not cat then return end;cat=tostring(cat);if cache[cat]~=nil then return cache[cat]or nil end;local d=RS:FindFirstChild("Data");local a=d and d:FindFirstChild("Assets");local cs=a and a:FindFirstChild("Configs");local m=cs and cs:FindFirstChild(cat);if not m or not m:IsA("ModuleScript")then cache[cat]=false;return end;local ok,v=pcall(require,m);cache[cat]=ok and v or false;return ok and v or nil end
local function rate(r)
 for _,k in ipairs({"EarningRate","EarningsRate","Income","IncomeRate","MoneyPerSecond","CashPerSecond","Rate"})do local n=tonumber(r[k]);if n then return n end end
 local c=cfg(first(r,{"AssetCategory","Category","AssetType","Type"}));if type(c)~="table"then return 0 end;if type(c.EarningRate)=="number"then return c.EarningRate end
 local key=first(r,{"AssetId","AssetName","Name","Asset","Id","ID"});if key~=nil then local d=c[key]or c[tostring(key)];if type(d)=="table"and type(d.EarningRate)=="number"then return d.EarningRate end end
 return 0
end
local function key(r,i)local v=first(r,{"Uid","UID","GUID","Guid","UUID","Uuid","EggId","AssetId","Id","ID"});if v~=nil then return tostring(v)end;return type(i)=="string"and i or tostring(i)end
local function best(s)local b;for i,r in pairs(s.Records or{})do if type(r)=="table"and r.State=="Slot"then local p=rpos(r);if p then local z={Key=key(r,i),Record=r,Position=p,Rate=rate(r),Distance=(p-Root.Position).Magnitude,Asset=tostring(first(r,{"AssetName","AssetCategory","Name","Asset","AssetId"})or"?"),Area=tostring(r.AreaId or r.Area or"?")};if not b or z.Rate>b.Rate or(z.Rate==b.Rate and z.Distance<b.Distance)then b=z end end end end;return b end
local function carry(r,i)
 if type(r)~="table"then return end
 local uid=r.Uid or r.UID or r.EggUid or r.EggUID or r.Id or r.ID
 local slot=r.FirstAreaSlotKey or r.SlotKey or r.RecordKey or r.Key or r.Slot or r.SlotId
 if not slot and type(i)=="string"and i:find("Forest:",1,true)then slot=i end
 if not slot and r.NestId then slot=tostring(r.NestId)end
 if slot then slot=tostring(slot);if slot:find("Slot_",1,true)and not slot:find("Forest:",1,true)then slot="Forest:"..slot end end
 if uid and slot then return{Uid=tostring(uid),FirstAreaSlotKey=tostring(slot)}end
end
local function forest(s)
 local root=s.Records or s;local b,bd,seen=nil,math.huge,{}
 local function walk(t)if type(t)~="table"or seen[t]then return end;seen[t]=true;for i,v in pairs(t)do if type(v)=="table"then local p=rpos(v);local c=carry(v,i);if p and c and(v.State==nil or v.State=="Slot")then local d=(p-CFG.ForestTrigger).Magnitude;if d<bd then bd=d;b={Position=p,Carry=c,Record=v}end end;walk(v)end end end
 walk(root);return b
end

local function eggPrompt(p)if not p or not p:IsA("ProximityPrompt")then return false end;if p.Name=="CarryAreaEgg"then return true end;local t=(tostring(p.ActionText).." "..tostring(p.ObjectText)):lower();return t:find("steal",1,true)and t:find("egg",1,true)end
local function ppos(p)local x=p and p.Parent;if x and x:IsA("BasePart")then return x.Position end;return x and wpos(x)end
local function findPrompt(pos,r)
 local b,bd=nil,r or math.huge
 for _,x in ipairs(WS:GetDescendants())do if x:IsA("ProximityPrompt")and eggPrompt(x)then local p=ppos(x);if p then local d=(p-pos).Magnitude;if d<=bd then b=x;bd=d end end end end
 return b,bd
end
local function fire(p,label,nom,base)
 if not p or not p.Parent or type(fireproximityprompt)~="function"then log("PROMPT_MISS",label);return false end
 local t=os.clock();local ok=pcall(function()fireproximityprompt(p)end);if ok then S.PromptFireCount+=1 end
 log("PROMPT_FIRE",string.format("%s | nominal=%.6f | actual=%.6f | drift=%.6f | ok=%s | pos=%s",label,tonumber(nom)or 0,base and t-base or 0,base and(t-base-(tonumber(nom)or 0))or 0,tostring(ok),vec(ppos(p))))
 return ok
end

if RefreshRE then RefreshRE.OnClientEvent:Connect(function(a,...)
 if not S.Running then return end;local d;if type(a)=="string"and a:sub(1,1)=="{"then local ok,v=pcall(function()return HttpService:JSONDecode(a)end);if ok and type(v)=="table"then d=v end end;if not d then return end
 local act=tostring(d.Action or"");local low=act:lower()
 if low:find("relocate",1,true)then S.Relocates+=1;log("RELOCATE",string.format("seq=%s | %s",tostring(d.Sequence),context()));return end
 if low:find("walk",1,true)then log("RIG_SETWALK",string.format("seq=%s | %s",tostring(d.Sequence),context()));return end
 if act=="EndRagdoll"then S.Ragdoll=false;if S.FirstEndAt==0 and S.FirstBeginAt>0 then S.FirstEndAt=now()end;log("ENDRAGDOLL",tostring(d.Sequence));return end
 if act~="BeginRagdoll"then return end
 S.Ragdoll=true;S.RagdollCount+=1;local dur=CFG.RagdollDurationFallback;if type(d.Arguments)=="table"and type(d.Arguments[1])=="number"then dur=d.Arguments[1]end
 log("BEGINRAGDOLL",string.format("hit=%d | seq=%s | duration=%.3f | mode=%s | %s",S.RagdollCount,tostring(d.Sequence),dur,S.Mode,context()))
 if S.PendingAuth and not S.AuthDone and S.AuthTargetPosition then
  local t=S.AuthTargetPosition;local st=os.clock();S.Root.CFrame=CFrame.new(t);zero();local latency=os.clock()-st
  S.PendingAuth=false;S.AuthDone=true;S.AuthBeginAt=os.clock();S.AuthDuration=dur;S.FirstBeginAt=now();S.Mode="AUTH_TP_DONE"
  log("BEGIN_AUTH_TP",string.format("seq=%s | target=%s | after=%s | writeLatency=%.6f",tostring(d.Sequence),vec(t),vec(S.Root.Position),latency))
 end
end)end

local function tween(dest,speed,label)
 if not alive()then return false,"dead"end;local dist=(dest-Root.Position).Magnitude;if dist<=2 then return true,"already"end
 local tw=TweenService:Create(Root,TweenInfo.new(math.max(dist/speed,.02),Enum.EasingStyle.Linear),{CFrame=CFrame.new(dest)});S.CurrentTween=tw;local done=false;local pb;local c=tw.Completed:Connect(function(x)pb=x;done=true end);tw:Play();log("TWEEN_START",string.format("%s | dist=%.1f | speed=%.0f",label,dist,speed))
 while S.Running and alive()and not done do RunService.Heartbeat:Wait()end;c:Disconnect();if S.CurrentTween==tw then S.CurrentTween=nil end;if not S.Running then pcall(function()tw:Cancel()end);return false,"stopped"end;if not alive()then return false,"dead"end;local rem=(dest-Root.Position).Magnitude;log("TWEEN_END",string.format("%s | playback=%s | remaining=%.1f | %s",label,tostring(pb),rem,context()));return rem<=10 or pb==Enum.PlaybackState.Completed,"remaining="..math.floor(rem)
end

local function auth(target,fr)
 if not CarryRF or not CarryRF.Parent then CarryRF=remote("AskFieldEggCarry","RemoteFunction")end;if not CarryRF or not RefreshRE then return false,"auth remote missing"end
 local p,d=findPrompt(target.Position,CFG.TargetPromptSearchRadius);if p then S.CachedTargetPrompt=p;log("TARGET_PROMPT_CACHED",string.format("%s | pairDist=%.2f",fullname(p),d))else log("TARGET_PROMPT_CACHE_MISS","burst usara fallback")end
 S.PendingAuth=true;S.AuthDone=false;S.AuthTargetPosition=target.Position+Vector3.new(0,CFG.EggTPHeight,0);S.AuthBeginAt=0;S.AuthDuration=2.5;S.Mode="WAIT_AUTH";status("ASK FIELD EGG CARRY • BACKGROUND")
 task.spawn(function()
  local st=os.clock();local ok,res=pcall(function()return CarryRF:InvokeServer({FirstAreaSlotKey=fr.Carry.FirstAreaSlotKey,Uid=fr.Carry.Uid})end);S.AskDuration=os.clock()-st;S.AskResult=res;log("ASK_RESULT_ASYNC",string.format("ok=%s | result=%s | duration=%.6f",tostring(ok),tostring(res),S.AskDuration))
 end)
 local endAt=os.clock()+CFG.AskPostWait;while S.Running and alive()and os.clock()<endAt do RunService.Heartbeat:Wait()end
 local fp=findPrompt(CFG.ForestTrigger,9);if not fp then S.PendingAuth=false;return false,"Forest prompt missing"end
 local base=os.clock();for i,off in ipairs(CFG.ForestPromptOffsets)do while S.Running and alive()and os.clock()<base+off do RunService.Heartbeat:Wait()end;if not fire(fp,"FOREST "..i.."/3",off,base)then end end
 status("FOREST 3X OK • ESPERANDO BEGINRAGDOLL");local dl=os.clock()+CFG.BeginTimeout;while S.Running and alive()and not S.AuthDone and os.clock()<dl do RunService.Heartbeat:Wait()end;if not S.AuthDone then S.PendingAuth=false;return false,"BeginRagdoll timeout"end;return true
end

local function burst(target)
 local firstAt=S.AuthBeginAt+math.max(S.AuthDuration-CFG.PromptLeadBeforeEnd,0)
 log("BURST_ARM",string.format("firstIn=%.6f | duration=%.3f",math.max(firstAt-os.clock(),0),S.AuthDuration))
 while S.Running and alive()and os.clock()<firstAt do RunService.Heartbeat:Wait()end
 if not S.Running or not alive()then return false,"stopped/dead"end
 S.Mode="BURST";local done=0;local ev=Instance.new("BindableEvent");local base=firstAt
 for i,off in ipairs(CFG.TargetPromptOffsets)do task.spawn(function()
  local at=base+off
  while S.Running and alive()and os.clock()<at do RunService.Heartbeat:Wait()end
  if S.Running and alive()then
   local p=findPrompt(Root.Position,CFG.TargetPromptSearchRadius)
   local pd=p and (ppos(p)-Root.Position).Magnitude or math.huge
   local td=p and (ppos(p)-target.Position).Magnitude or math.huge
   if p and pd<=CFG.TargetPromptSearchRadius then
    fire(p,string.format("BURST %02d/14",i),off,base)
    log("BURST_CONTEXT",string.format("n=%02d | promptPlayer=%.2f | promptTarget=%.2f | playerTarget=%.2f",i,pd,td,(target.Position-Root.Position).Magnitude))
   else
    log("BURST_MISS",string.format("n=%02d | promptPlayer=%.2f | promptTarget=%.2f | playerTarget=%.2f",i,pd,td,(target.Position-Root.Position).Magnitude))
   end
  end
  done+=1;ev:Fire()
 end)end
 while S.Running and alive()and done<14 do ev.Event:Wait()end
 ev:Destroy()
 if not S.Running or not alive()then return false,"stopped/dead"end
 log("BURST_DONE","count="..done)
 local escape=base+CFG.TargetPromptOffsets[#CFG.TargetPromptOffsets]+CFG.EscapeAfterLastPrompt
 while S.Running and alive()and os.clock()<escape do RunService.Heartbeat:Wait()end
 return true
end

local function snapshotTargetState(uid)
 local s=snap();if not s then return nil end
 for _,r in pairs(s.Records or {})do
  if type(r)=="table" then
   local u=r.Uid or r.UID or r.EggUid or r.EggUID or r.Id or r.ID
   if u and tostring(u)==tostring(uid) then return tostring(r.State or "?"),r end
  end
 end
 return nil
end

local function pickupForensic(target, phase)
 local p = findPrompt(Root.Position, CFG.TargetPromptSearchRadius)
 local pp = p and ppos(p)
 local uidState, rec = snapshotTargetState(target.Key)
 local rp = rec and rpos(rec)
 log("PICKUP_FORENSIC", string.format(
  "phase=%s | uidState=%s | playerToTarget=%.2f | playerToPrompt=%.2f | promptToTarget=%.2f | recordPos=%s | %s",
  tostring(phase), tostring(uidState),
  (target.Position-Root.Position).Magnitude,
  pp and (pp-Root.Position).Magnitude or math.huge,
  pp and (pp-target.Position).Magnitude or math.huge,
  rp and vec(rp) or "nil", context()
 ))
end

local function pickupGate(target)
 if not alive()then return false,"dead"end
 S.Mode="PICKUP_GATE";status("PICKUP • VOLTANDO AO OVO");pickupForensic(target,"GATE_START")
 local timeout=os.clock()+2.0
 local reached=false
 while S.Running and alive() and os.clock()<timeout do
  local p,pd=findPrompt(Root.Position,CFG.TargetPromptSearchRadius)
  if p and pd<=8.0 then
   reached=true
   log("PICKUP_RANGE",string.format("playerPrompt=%.2f | playerTarget=%.2f | prompt=%s",pd,(target.Position-Root.Position).Magnitude,fullname(p)))
   zero()
   pickupForensic(target,"PRE_FIRE")
   local fired=fire(p,"PICKUP_CONFIRM",0,os.clock())
   pickupForensic(target,"POST_FIRE")
   if not fired then return false,"pickup fire failed" end

   -- V111: confirmação curta. Não fica parado esperando um segundo Ragdoll.
   -- Se o prompt sumir ou o estado do UID mudar, seguimos imediatamente.
   local confirmEnd=os.clock()+0.45
   while S.Running and alive() and os.clock()<confirmEnd do
    local pp,pd2=findPrompt(Root.Position,CFG.TargetPromptSearchRadius)
    if not pp or pd2>8.0 then
     log("PICKUP_CONFIRMED",string.format("promptGoneOrOutOfRange | playerPrompt=%.2f",pd2 or math.huge))
     local settleEnd=os.clock()+CFG.PickupToFlightDelay
     while S.Running and alive() and os.clock()<settleEnd do RunService.Heartbeat:Wait() end
     if not S.Running or not alive() then return false,"stopped/dead" end
     zero()
     local preSt=select(1,snapshotTargetState(target.Key)); log("PICKUP_SETTLED",string.format("delay=%.3f | uidState=%s | state=%s | %s",CFG.PickupToFlightDelay,tostring(preSt),state(),context()))
     return true
    end
    local st=snapshotTargetState(target.Key)
    if st and st~="Slot" then
     log("PICKUP_CONFIRMED",string.format("uidState=%s",st))
     return true
    end
    RunService.Heartbeat:Wait()
   end

   -- O prompt pode continuar visível mesmo depois do servidor aceitar.
   -- Não repetimos o prompt e não esperamos outro BeginRagdoll.
   log("PICKUP_TRANSITION","confirmation window ended -> MICRO_DELAY")
   -- V112: pequeno intervalo para o servidor consolidar Carried.
   -- Não reposiciona, não dispara outro prompt e não espera outro Ragdoll.
   local settleEnd=os.clock()+CFG.PickupToFlightDelay
   while S.Running and alive() and os.clock()<settleEnd do
    RunService.Heartbeat:Wait()
   end
   if not S.Running or not alive() then return false,"stopped/dead" end
   zero()
   local preSt=select(1,snapshotTargetState(target.Key))
   log("PICKUP_SETTLED",string.format("delay=%.3f | uidState=%s | state=%s | %s",CFG.PickupToFlightDelay,tostring(preSt),state(),context()))
   return true
  end

  local stateNow=state()
  if stateNow~="Enum.HumanoidStateType.Physics" and stateNow~="Enum.HumanoidStateType.Freefall" then
   local tp=findPrompt(target.Position,CFG.TargetPromptSearchRadius)
   local dest=(tp and ppos(tp)) or target.Position
   local flat=Vector3.new(dest.X,Root.Position.Y,dest.Z)
   if (flat-Root.Position).Magnitude>2 then
    local tw=TweenService:Create(Root,TweenInfo.new(math.max((flat-Root.Position).Magnitude/325,.03),Enum.EasingStyle.Linear),{CFrame=CFrame.new(flat)})
    S.CurrentTween=tw;tw:Play();log("PICKUP_APPROACH",string.format("dist=%.2f | speed=325 | dest=%s",(flat-Root.Position).Magnitude,vec(flat)))
    while S.Running and alive() and tw.PlaybackState ~= Enum.PlaybackState.Completed do RunService.Heartbeat:Wait() end
    pcall(function()tw:Cancel()end)
    if S.CurrentTween==tw then S.CurrentTween=nil end
    pickupForensic(target,"AFTER_APPROACH")
   end
  end
  RunService.Heartbeat:Wait()
 end
 return false,reached and "pickup timeout after approach" or "pickup range timeout"
end

local function startFlightUidTrace(target)
 task.spawn(function()
  local schedule={0,0.10,0.25,0.50,0.75,1.00,1.50,2.00,2.50,3.00}
  local t0=os.clock()
  for _,dt in ipairs(schedule) do
   while S.Running and alive() and os.clock() < t0+dt do RunService.Heartbeat:Wait() end
   if not S.Running or not alive() then break end
   local st,rec=snapshotTargetState(target.Key)
   local rp=rec and rpos(rec)
   local pd=(target.Position-Root.Position).Magnitude
   local rs=rp and ((rp-Root.Position).Magnitude) or math.huge
   log("FLIGHT_UID_TRACE",string.format("t=%.3f | uidState=%s | recordPos=%s | playerToRecord=%.2f | playerToTarget=%.2f | %s",os.clock()-t0,tostring(st),rp and vec(rp) or "nil",rs,pd,context()))
  end
 end)
end

local function flight()
 -- Safety diagnostic: if the server state stops being Carried during flight, cancel immediately.
 local dest=S.BaseReturn+Vector3.new(0,CFG.BaseAirHeight,0);local initial=(dest-Root.Position).Magnitude;S.Mode="FLIGHT";S.FlightActive=true
 startFlightUidTrace(S.Target)
 log("FLIGHT_START",string.format("dist=%.1f | speed=%d | target=%s | %s",initial,CFG.FlightSpeed,vec(dest),context()));status(string.format("AIR1000 • %.0f STUDS",initial))
 local tw=TweenService:Create(Root,TweenInfo.new(math.max(initial/CFG.FlightSpeed,.03),Enum.EasingStyle.Linear),{CFrame=CFrame.new(dest)});S.CurrentTween=tw;local done=false;local pb;local c=tw.Completed:Connect(function(x)pb=x;done=true end);local lp=Root.Position;local ld=(dest-lp).Magnitude;tw:Play()
 while S.Running and alive()and not done do
  RunService.Heartbeat:Wait()
  local p=Root.Position;local d=(dest-p).Magnitude;local jump=(p-lp).Magnitude
  -- Never continue the return flight after the target UID is no longer Carried.
  local uidState=snapshotTargetState(S.Target and S.Target.Key)
  if uidState and uidState~="Carried" then
   S.FlightInterrupted=true
   S.FlightFailure="pickup lost: "..tostring(uidState)
   log("FLIGHT_PICKUP_LOST",string.format("uidState=%s | rem=%.1f | area=%s | %s",tostring(uidState),d,area(),context()))
   pcall(function()tw:Cancel()end)
   break
  end
  if jump>=CFG.RollbackJumpDistance and d>=ld+CFG.RollbackDistanceIncrease then S.FlightRollback=true;log("ROLLBACK_PHYSICAL",string.format("jump=%.1f | rem %.1f -> %.1f | %s",jump,ld,d,context()));pcall(function()tw:Cancel()end);break end
  lp=p;ld=d
 end
 c:Disconnect();if S.CurrentTween==tw then S.CurrentTween=nil end;S.FlightActive=false;if not S.Running or not alive()then return false,"dead/stopped"end;if S.FlightRollback then return false,"physical rollback"end
 local rem=(dest-Root.Position).Magnitude;log("FLIGHT_END",string.format("playback=%s | remaining=%.1f | %s",tostring(pb),rem,context()));if pb~=Enum.PlaybackState.Completed and rem>CFG.BaseAirArrival then return false,"flight incomplete"end
 S.Mode="BASE_FALL";local dl=os.clock()+CFG.BaseFallTimeout;while S.Running and alive()and os.clock()<dl do local p=Root.Position;if Vector2.new(p.X-S.BaseReturn.X,p.Z-S.BaseReturn.Z).Magnitude<=CFG.BaseHorizontalArrival and p.Y<=S.BaseReturn.Y+6 then S.FlightSuccess=true;S.Mode="SUCCESS";log("CYCLE_SUCCESS","AIR1000 + base OK");return true end;RunService.Heartbeat:Wait()end;return false,"base fall timeout"
end

task.spawn(function()local last=0;while S.Running do RunService.Heartbeat:Wait();if alive()then local t=os.clock();local a=area();if a~=S.LastArea then log("AREA_CHANGE",string.format("%s -> %s | %s",tostring(S.LastArea),a,vec(Root.Position)));S.LastArea=a end;if t-last>=CFG.ReportMovementInterval then last=t;S.Movement[#S.Movement+1]={t=now(),pos=Root.Position,area=a,state=state(),speed=Root.AssemblyLinearVelocity.Magnitude,hp=Hum.Health,mode=S.Mode};if #S.Movement>CFG.MaxMovementSamples then table.remove(S.Movement,1)end end end end end)

local function cycle()
 if not alive()then return false,"dead"end
 if not S.BaseReturn then status("DETECTANDO BASE");local p,r=detectBase();if not p or not r then return false,"base nao encontrada"end end
 S.Mode="SNAPSHOT";status("PEGANDO SNAPSHOT");local s=snap();if not s then return false,"snapshot falhou"end
 local target=best(s);if not target then return false,"nenhum ovo Slot"end;local fr=forest(s);if not fr then return false,"Forest carry record nao encontrado"end
 S.Target=target;targetText(string.format("%s | %.0f/s | %s",target.Asset,target.Rate,target.Area));log("TARGET",string.format("uid=%s | asset=%s | rate=%.0f | area=%s | pos=%s",target.Key,target.Asset,target.Rate,target.Area,vec(target.Position)))
 S.Mode="CHICKEN";status("1/5 • TWEEN120 GALINHA");local ok,reason=tween(CFG.Chicken,CFG.ChickenSpeed,"CHICKEN120");if not ok then return false,reason end;zero();local u=os.clock()+CFG.ChickenAfterWait;while S.Running and alive()and os.clock()<u do RunService.Heartbeat:Wait()end
 S.Mode="FOREST";status("2/5 • TP FOREST");tp(CFG.ForestTrigger);log("TP_FOREST",vec(CFG.ForestTrigger));u=os.clock()+CFG.ForestAfterTPWait;while S.Running and alive()and os.clock()<u do RunService.Heartbeat:Wait()end
 status("3/5 • AUTH TP");ok,reason=auth(target,fr);if not ok then return false,reason end
 status("4/6 • BURST 14X");ok,reason=burst(target);if not ok then return false,reason end
 status("5/6 • CONFIRMANDO PICKUP");ok,reason=pickupGate(target);if not ok then return false,reason end
 status("6/6 • AIR1000");pickupForensic(target,"PRE_FLIGHT");return flight()
end

Stop.MouseButton1Click:Connect(function()log("STOP_BUTTON","usuario parou");stop("PARADO PELO BOTAO")end)
Copy.MouseButton1Click:Connect(function()copyReport();Copy.Text="COPIADO";task.delay(.8,function()if Copy and Copy.Parent then Copy.Text="COPIAR LOG"end end)end)
log("MONITOR_START",context());status("V114 LIGADO")

task.spawn(function()
 task.wait(.25)
 while S.Running and alive()do
  S.FlightSuccess=false;S.FlightRollback=false;S.FlightInterrupted=false;S.FlightFailure=nil;S.AuthDone=false;S.PendingAuth=false;S.Ragdoll=false;S.RagdollCount=0;S.FirstBeginAt=0;S.FirstEndAt=0;S.CachedTargetPrompt=nil;S.PromptFireCount=0
  local ok,reason=cycle();if not S.Running or not alive()then break end
  if ok then log("CYCLE_DONE",tostring(reason));status("SUCESSO | NOVO CICLO");if not CFG.AutoRepeat then stop("SUCESSO - FINALIZADO");break end;local u=os.clock()+CFG.CycleDelay;while S.Running and alive()and os.clock()<u do RunService.Heartbeat:Wait()end
  else log("CYCLE_FAIL",tostring(reason));stop("FALHA: "..tostring(reason));break end
 end
end)
