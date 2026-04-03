-- RAYFIELD
local Rayfield = loadstring(game:HttpGet("https://sirius.menu/rayfield"))()

-- JANELA COM KEY
local Window = Rayfield:CreateWindow({
Name = "XITHUB",
LoadingTitle = "Carregando XITHUB...",
KeySystem = true,
KeySettings = {
Title = "XITHUB KEY",
Subtitle = "Digite a Key",
Note = "sistema de Key em desenvolvimento. KEY ATUAL:x",
FileName = "XITHUB",
SaveKey = false,
GrabKeyFromSite = false,
Key = {"x"}
}
})

-- SERVIÇOS
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local UIS = game:GetService("UserInputService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- VARIÁVEIS
local AIM=false
local FOV=120
local Pulse=false
local Rainbow=false

local ESP=false
local ShowHP=false
local Highlight=false

local fly=false

-- =========================
-- ABAS
-- =========================
local Combat = Window:CreateTab("🎯 Combate")
local Visual = Window:CreateTab("💎 Visual")
local PlayerTab = Window:CreateTab("⚡ Player")

-- =========================
-- 🎯 FOV CIRCLE
-- =========================
local circle = Drawing.new("Circle")
circle.Visible=false
circle.Radius=FOV
circle.Thickness=2
circle.Color=Color3.fromRGB(0,170,255)

-- =========================
-- 🎯 AIMBOT
-- =========================
Combat:CreateToggle({
Name="Aimbot",
CurrentValue=false,
Callback=function(v)
AIM=v
circle.Visible=v
end
})

Combat:CreateSlider({
Name="FOV",
Range={50,300},
Increment=1,
CurrentValue=120,
Callback=function(v)
FOV=v
circle.Radius=v
end
})

Combat:CreateToggle({
Name="Pulse FOV",
CurrentValue=false,
Callback=function(v) Pulse=v end
})

Combat:CreateToggle({
Name="Rainbow FOV",
CurrentValue=false,
Callback=function(v) Rainbow=v end
})

-- LOOP AIMBOT
RunService.RenderStepped:Connect(function()

circle.Position=Vector2.new(camera.ViewportSize.X/2,camera.ViewportSize.Y/2)

if Pulse then
circle.Radius = FOV + math.sin(tick()*5)*20
end

if Rainbow then
circle.Color = Color3.fromHSV(tick()%5/5,1,1)
end

if AIM then
local target=nil
local dist=math.huge

for _,v in pairs(Players:GetPlayers()) do
if v~=player and v.Character and v.Character:FindFirstChild("Head") then
local pos,vis=camera:WorldToViewportPoint(v.Character.Head.Position)
if vis then
local mag=(Vector2.new(pos.X,pos.Y)-circle.Position).Magnitude
if mag<dist and mag<=circle.Radius then
dist=mag
target=v
end
end
end
end

if target and target.Character then
camera.CFrame=CFrame.new(camera.CFrame.Position,target.Character.Head.Position)
end

end

end)

-- =========================
-- 👁 ESP + VIDA + HIGHLIGHT
-- =========================
Visual:CreateToggle({Name="ESP Azul",CurrentValue=false,Callback=function(v)ESP=v end})
Visual:CreateToggle({Name="Mostrar Vida ❤️",CurrentValue=false,Callback=function(v)ShowHP=v end})
Visual:CreateToggle({Name="Highlight Azul",CurrentValue=false,Callback=function(v)Highlight=v end})

local highlights={}

RunService.RenderStepped:Connect(function()
for _,v in pairs(Players:GetPlayers()) do
if v~=player and v.Character and v.Character:FindFirstChild("Head") then

local char=v.Character
local hum=char:FindFirstChildOfClass("Humanoid")

-- ESP
if ESP then
if not char:FindFirstChild("ESP_NAME") then
local bill=Instance.new("BillboardGui",char)
bill.Name="ESP_NAME"
bill.Size=UDim2.new(0,200,0,40)
bill.Adornee=char.Head
bill.AlwaysOnTop=true

local txt=Instance.new("TextLabel",bill)
txt.Size=UDim2.new(1,0,1,0)
txt.BackgroundTransparency=1
txt.TextScaled=true
txt.Font=Enum.Font.GothamBold
txt.TextColor3=Color3.fromRGB(0,170,255)
txt.Text=v.Name.."  @"..v.DisplayName
end
else
if char:FindFirstChild("ESP_NAME") then char.ESP_NAME:Destroy() end
end

-- VIDA
if ShowHP and hum then
if not char:FindFirstChild("HP_BAR") then
local hp=Instance.new("BillboardGui",char)
hp.Name="HP_BAR"
hp.Size=UDim2.new(0,200,0,20)
hp.Adornee=char.Head
hp.StudsOffset=Vector3.new(0,2,0)
hp.AlwaysOnTop=true

local bg=Instance.new("Frame",hp)
bg.Size=UDim2.new(1,0,0.3,0)
bg.BackgroundColor3=Color3.fromRGB(50,50,50)

local bar=Instance.new("Frame",bg)
bar.Name="HP"
bar.Size=UDim2.new(1,0,1,0)
bar.BackgroundColor3=Color3.fromRGB(0,170,255)
end

char.HP_BAR.Frame.HP.Size=UDim2.new(hum.Health/hum.MaxHealth,0,1,0)
else
if char:FindFirstChild("HP_BAR") then char.HP_BAR:Destroy() end
end

-- HIGHLIGHT
if Highlight then
if not highlights[v] then
local hl=Instance.new("Highlight",char)
hl.FillColor=Color3.fromRGB(0,170,255)
hl.FillTransparency=0.5
hl.OutlineColor=Color3.new(1,1,1)
highlights[v]=hl
end
else
if highlights[v] then highlights[v]:Destroy() highlights[v]=nil end
end

end
end
end)

-- =========================
-- ⚡ PLAYER
-- =========================

-- FPS BOOST
local fps=false
PlayerTab:CreateToggle({
Name="FPS BOOST",
CurrentValue=false,
Callback=function(v)
fps=v

if fps then
Lighting.GlobalShadows=false
Lighting.FogEnd=100000
Lighting.Brightness=3

for _,v in pairs(workspace:GetDescendants()) do
if v:IsA("BasePart") then
v.Material=Enum.Material.SmoothPlastic
v.CastShadow=false
elseif v:IsA("Decal") or v:IsA("Texture") then
v:Destroy()
elseif v:IsA("ParticleEmitter") then
v.Enabled=false
end
end
end
end
})

-- VELOCIDADE
PlayerTab:CreateInput({
Name="Velocidade",
PlaceholderText="Ex: 50",
RemoveTextAfterFocusLost=true,
Callback=function(txt)
local num=tonumber(txt)
if num then
local hum=player.Character and player.Character:FindFirstChildOfClass("Humanoid")
if hum then hum.WalkSpeed=num end
end
end
})

-- PULO
PlayerTab:CreateInput({
Name="Pulo",
PlaceholderText="Ex: 100",
RemoveTextAfterFocusLost=true,
Callback=function(txt)
local num=tonumber(txt)
if num then
local hum=player.Character and player.Character:FindFirstChildOfClass("Humanoid")
if hum then hum.JumpPower=num end
end
end
})

-- NOTIFICAÇÃO
Rayfield:Notify({
Title="XITHUB",
Content="XITHUB CARREGADO COM SUCESS✅
Obrigado por usar",
Duration=4
})

PlayerTab:CreateToggle({
Name="Noclip",
CurrentValue=false,
Callback=function(v)
_G.Noclip=v

RunService.Stepped:Connect(function()
if _G.Noclip and player.Character then
for _,part in pairs(player.Character:GetDescendants()) do
if part:IsA("BasePart") then
part.CanCollide=false
end
end
end
end)

end
})

local VirtualUser = game:GetService("VirtualUser")

player.Idled:Connect(function()
VirtualUser:Button2Down(Vector2.new(0,0),workspace.CurrentCamera.CFrame)
task.wait(1)
VirtualUser:Button2Up(Vector2.new(0,0),workspace.CurrentCamera.CFrame)
end)

PlayerTab:CreateInput({
Name="TP para jogador",
PlaceholderText="Nome do player",
RemoveTextAfterFocusLost=true,
Callback=function(txt)
for _,v in pairs(game.Players:GetPlayers()) do
if string.find(string.lower(v.Name), string.lower(txt)) then
if v.Character and v.Character:FindFirstChild("HumanoidRootPart") then
player.Character.HumanoidRootPart.CFrame =
v.Character.HumanoidRootPart.CFrame + Vector3.new(0,3,0)
end
end
end
end
})
PlayerTab:CreateButton({
    Name = "Abrir Fly Menu",
    Callback = function()

        local player = game.Players.LocalPlayer

        if player.PlayerGui:FindFirstChild("FlyGui") then
            player.PlayerGui.FlyGui:Destroy()
        end

        local flying = false
        local speed = 50

        local gui = Instance.new("ScreenGui")
        gui.Name = "FlyGui"
        gui.Parent = player.PlayerGui

        local frame = Instance.new("Frame", gui)
        frame.Size = UDim2.new(0, 140, 0, 120)
        frame.Position = UDim2.new(0, 20, 0.4, 0)
        frame.BackgroundColor3 = Color3.fromRGB(20,20,20)
        Instance.new("UICorner", frame)

        local flyBtn = Instance.new("TextButton", frame)
        flyBtn.Size = UDim2.new(1, -10, 0, 40)
        flyBtn.Position = UDim2.new(0, 5, 0, 5)
        flyBtn.Text = "FLY: OFF"
        flyBtn.BackgroundColor3 = Color3.fromRGB(150,0,0)
        flyBtn.TextColor3 = Color3.new(1,1,1)

        local speedBox = Instance.new("TextBox", frame)
        speedBox.Size = UDim2.new(1, -10, 0, 30)
        speedBox.Position = UDim2.new(0, 5, 0, 55)
        speedBox.Text = "50"
        speedBox.BackgroundColor3 = Color3.fromRGB(40,40,40)
        speedBox.TextColor3 = Color3.new(1,1,1)

        -- BOTÃO MINIMIZAR
        local miniBtn = Instance.new("TextButton", gui)
        miniBtn.Size = UDim2.new(0, 80, 0, 20)
        miniBtn.Position = UDim2.new(0, 20, 0.4, -25)
        miniBtn.Text = "Minimize"
        miniBtn.BackgroundColor3 = Color3.fromRGB(40,40,40)
        miniBtn.TextColor3 = Color3.new(1,1,1)
        miniBtn.TextSize = 12
        Instance.new("UICorner", miniBtn)

        local bv = Instance.new("BodyVelocity")
        local bg = Instance.new("BodyGyro")

        speedBox.FocusLost:Connect(function()
            speed = tonumber(speedBox.Text) or 50
        end)

        flyBtn.MouseButton1Click:Connect(function()
            local char = player.Character
            local root = char and char:FindFirstChild("HumanoidRootPart")
            if not root then return end

            flying = not flying

            if flying then
                flyBtn.Text = "FLY: ON"
                flyBtn.BackgroundColor3 = Color3.fromRGB(0,150,0)

                bv.Parent = root
                bg.Parent = root
                bv.MaxForce = Vector3.new(9e9,9e9,9e9)
                bg.MaxTorque = Vector3.new(9e9,9e9,9e9)

                char.Humanoid.PlatformStand = true
            else
                flyBtn.Text = "FLY: OFF"
                flyBtn.BackgroundColor3 = Color3.fromRGB(150,0,0)

                bv.Parent = nil
                bg.Parent = nil

                char.Humanoid.PlatformStand = false
            end
        end)

        -- FUNÇÃO MINIMIZAR
        miniBtn.MouseButton1Click:Connect(function()
            frame.Visible = not frame.Visible
            miniBtn.Text = frame.Visible and "Minimize" or "Open"
        end)

        game:GetService("RunService").RenderStepped:Connect(function()
            if flying and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                local root = player.Character.HumanoidRootPart
                local hum = player.Character:FindFirstChild("Humanoid")
                local cam = workspace.CurrentCamera

                bg.CFrame = cam.CFrame

                if hum.MoveDirection.Magnitude > 0 then
                    bv.Velocity = cam.CFrame.LookVector * speed
                else
                    bv.Velocity = Vector3.new(0,0,0)
                end
            end
        end)

    end
})
