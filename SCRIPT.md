-- LocalScript Roblox

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- ScreenGui
local ScreenGui = Instance.new("ScreenGui", PlayerGui)
ScreenGui.ResetOnSpawn = false

-- Helper
local function create(class, props)
    local obj = Instance.new(class)
    for i,v in pairs(props) do obj[i]=v end
    return obj
end

-- Main Frame
local MainFrame = create("Frame", {
    Parent = ScreenGui,
    Size = UDim2.new(0.7,0,0.7,0),
    Position = UDim2.new(0.15,0,0.15,0),
    BackgroundColor3 = Color3.fromRGB(30,30,30)
})
create("UICorner",{Parent=MainFrame, CornerRadius=UDim.new(0,10)})
create("UIStroke",{Parent=MainFrame, Color=Color3.new(1,1,1), Thickness=1})

-- TitleBar
local TitleBar = create("Frame",{Parent=MainFrame, Size=UDim2.new(1,0,0,30), BackgroundColor3=Color3.fromRGB(40,40,40)})
create("UICorner",{Parent=TitleBar, CornerRadius=UDim.new(0,10)})
create("TextLabel",{Parent=TitleBar, Size=UDim2.new(0.9,0,1,0), Text="Menu Exploração", TextColor3=Color3.new(1,1,1), BackgroundTransparency=1, Font=Enum.Font.SourceSansBold, TextScaled=true, TextXAlignment=Enum.TextXAlignment.Left, Position=UDim2.new(0.05,0,0,0)})

-- Min/Max buttons
local MinimizeButton = create("TextButton",{Parent=TitleBar, Size=UDim2.new(0,30,0,30), Position=UDim2.new(0.85,0,0,0), Text="-", BackgroundColor3=Color3.fromRGB(200,50,50), TextColor3=Color3.new(1,1,1)})
local MaximizeButton = create("TextButton",{Parent=TitleBar, Size=UDim2.new(0,30,0,30), Position=UDim2.new(0.9,0,0,0), Text="+", BackgroundColor3=Color3.fromRGB(50,200,50), TextColor3=Color3.new(1,1,1)})

local function toggleMenu()
    if MainFrame.Size == UDim2.new(0.7,0,0.7,0) then
        MainFrame:TweenSize(UDim2.new(0,50,0,30),"Out","Quad",0.3,true)
    else
        MainFrame:TweenSize(UDim2.new(0.7,0,0.7,0),"Out","Quad",0.3,true)
    end
end
MinimizeButton.MouseButton1Click:Connect(toggleMenu)
MaximizeButton.MouseButton1Click:Connect(toggleMenu)

-- Drag
local dragging, dragInput, mousePos, framePos = false,nil,nil,nil
TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging=true
        mousePos=input.Position
        framePos=MainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState==Enum.UserInputState.End then dragging=false end
        end)
    end
end)
TitleBar.InputChanged:Connect(function(input)
    if input.UserInputType==Enum.UserInputType.MouseMovement then dragInput=input end
end)
RunService.RenderStepped:Connect(function()
    if dragging and dragInput then
        local delta = UserInputService:GetMouseLocation()-mousePos
        MainFrame.Position = UDim2.new(framePos.X.Scale, framePos.X.Offset+delta.X, framePos.Y.Scale, framePos.Y.Offset+delta.Y)
    end
end)

-- Tabs
local TabsFrame = create("Frame",{Parent=MainFrame, Size=UDim2.new(0.3,0,1,0), Position=UDim2.new(0,0,0,30), BackgroundColor3=Color3.fromRGB(25,25,25)})
local ContentFrame = create("Frame",{Parent=MainFrame, Size=UDim2.new(0.7,0,1,0), Position=UDim2.new(0.3,0,0,30), BackgroundColor3=Color3.fromRGB(35,35,35)})

-- Função criar tabs
local function createTab(name, options)
    local tabButton = create("TextButton",{Parent=TabsFrame, Size=UDim2.new(1,0,0,40), Text=name, BackgroundColor3=Color3.fromRGB(50,50,50), TextColor3=Color3.new(1,1,1)})
    tabButton.MouseButton1Click:Connect(function()
        for _,v in pairs(ContentFrame:GetChildren()) do if v:IsA("Frame") then v:Destroy() end end
        for i,opt in pairs(options) do
            local optFrame = create("Frame",{Parent=ContentFrame, Size=UDim2.new(1,-20,0,50), Position=UDim2.new(0,10,0,(i-1)*55), BackgroundColor3=Color3.fromRGB(45,45,45)})
            create("UICorner",{Parent=optFrame, CornerRadius=UDim.new(0,5)})
            create("TextLabel",{Parent=optFrame, Size=UDim2.new(0.55,0,1,0), Text=opt.Name, TextColor3=Color3.new(1,1,1), BackgroundTransparency=1, TextScaled=true})
            if opt.Slider then
                local slider = create("TextBox",{Parent=optFrame, Size=UDim2.new(0.35,0,0.6,0), Position=UDim2.new(0.6,0,0.2,0), Text=tostring(opt.Value), BackgroundColor3=Color3.fromRGB(100,100,200), TextColor3=Color3.new(1,1,1), TextScaled=true})
                slider.FocusLost:Connect(function()
                    local num = tonumber(slider.Text)
                    if num then opt.Value=math.clamp(num,0,100) opt.Function(opt.Value) end
                end)
            else
                local button = create("TextButton",{Parent=optFrame, Size=UDim2.new(0.35,0,0.8,0), Position=UDim2.new(0.6,0,0.1,0), Text=opt.Toggle and "Ativar" or "Ação", BackgroundColor3=Color3.fromRGB(100,100,200), TextColor3=Color3.new(1,1,1)})
                if opt.Toggle then
                    local activated=false
                    button.MouseButton1Click:Connect(function()
                        activated=not activated
                        button.Text=activated and "Desativar" or "Ativar"
                        opt.Function(activated)
                    end)
                else
                    button.MouseButton1Click:Connect(opt.Function)
                end
            end
        end
    end)
end

-- Variáveis Fly/NoClip/ESP
local flyEnabled=false
local flySpeed=50
local noClipEnabled=false
local espEnabled=false
local flyBodyVelocity=nil
local flyBodyGyro=nil

-- Funções Fly
local function Fly(active)
    flyEnabled=active
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        local hrp = char.HumanoidRootPart
        if active then
            flyBodyVelocity = Instance.new("BodyVelocity")
            flyBodyVelocity.MaxForce = Vector3.new(1e5,1e5,1e5)
            flyBodyVelocity.Velocity = Vector3.new(0,0,0)
            flyBodyVelocity.Parent = hrp
            flyBodyGyro = Instance.new("BodyGyro")
            flyBodyGyro.MaxTorque = Vector3.new(1e5,1e5,1e5)
            flyBodyGyro.CFrame = hrp.CFrame
            flyBodyGyro.Parent = hrp
        else
            if flyBodyVelocity then flyBodyVelocity:Destroy() end
            if flyBodyGyro then flyBodyGyro:Destroy() end
        end
    end
end

local function FlySpeed(val)
    flySpeed=val
end

local function NoClip(active)
    noClipEnabled=active
    local char=LocalPlayer.Character
    if char then
        for _,p in pairs(char:GetDescendants()) do
            if p:IsA("BasePart") then
                p.CanCollide = not active
            end
        end
    end
end

-- Função ESP (apenas players visíveis)
local function ESP(active)
    espEnabled=active
    for _,plr in pairs(Players:GetPlayers()) do
        if plr~=LocalPlayer then
            local char=plr.Character
            if char then
                local hrp=char:FindFirstChild("HumanoidRootPart")
                if hrp then
                    local highlight = hrp:FindFirstChild("ESP_Box")
                    local nameTag = hrp:FindFirstChild("ESP_Name")
                    if active then
                        if not highlight then
                            highlight = Instance.new("BoxHandleAdornment")
                            highlight.Name = "ESP_Box"
                            highlight.Adornee = hrp
                            highlight.Size = Vector3.new(2,3,1)
                            highlight.Color3 = Color3.fromRGB(255,0,0)
                            highlight.Transparency = 0
                            highlight.AlwaysOnTop = true
                            highlight.ZIndex = 10
                            highlight.Parent = hrp
                        end
                        if not nameTag then
                            nameTag = Instance.new("BillboardGui")
                            nameTag.Name = "ESP_Name"
                            nameTag.Adornee = hrp
                            nameTag.Size = UDim2.new(0,100,0,50)
                            nameTag.StudsOffset = Vector3.new(0,3,0)
                            local txt = Instance.new("TextLabel", nameTag)
                            txt.Size = UDim2.new(1,0,1,0)
                            txt.BackgroundTransparency = 1
                            txt.Text = plr.Name
                            txt.TextColor3 = Color3.fromRGB(255,0,0)
                            txt.TextScaled = true
                            nameTag.Parent = hrp
                        end
                    else
                        if highlight then highlight:Destroy() end
                        if nameTag then nameTag:Destroy() end
                    end
                end
            end
        end
    end
end

-- Criar tabs
createTab("FLY", {
    {Name="Ativar Fly", Toggle=true, Function=Fly},
    {Name="Velocidade", Slider=true, Value=50, Function=FlySpeed},
    {Name="No Clip", Toggle=true, Function=NoClip}
})

createTab("ESP", {
    {Name="Ativar ESP", Toggle=true, Function=ESP}
})

-- Fly update
RunService.RenderStepped:Connect(function()
    if flyEnabled and flyBodyVelocity and flyBodyGyro then
        local char=LocalPlayer.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            local hrp=char.HumanoidRootPart
            local moveDir=Vector3.new()
            if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveDir=moveDir+hrp.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveDir=moveDir-hrp.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveDir=moveDir-hrp.CFrame.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveDir=moveDir+hrp.CFrame.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.Space) then moveDir=moveDir+Vector3.new(0,1,0) end
            if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then moveDir=moveDir-Vector3.new(0,1,0) end
            flyBodyVelocity.Velocity = moveDir.Unit * flySpeed
            flyBodyGyro.CFrame = CFrame.new(hrp.Position, hrp.Position + hrp.CFrame.LookVector)
        end
    end
end)
