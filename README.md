-- ตรวจสอบผู้เล่นและ PlayerGui
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- ลบ UI เก่าออกก่อน (ถ้ามี)
if PlayerGui:FindFirstChild("MM2CheatMenu") then
    PlayerGui.MM2CheatMenu:Destroy()
end

-- สร้าง ScreenGui หลัก
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MM2CheatMenu"
ScreenGui.Parent = PlayerGui
ScreenGui.ResetOnSpawn = false

-- สร้างปุ่มย่อ/ขยาย (Toggle Button) เพื่อเปิด-ปิดเมนู
local ToggleButton = Instance.new("TextButton")
ToggleButton.Name = "ToggleButton"
ToggleButton.Parent = ScreenGui
ToggleButton.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
ToggleButton.Position = UDim2.new(0.05, 0, 0.1, 0)
ToggleButton.Size = UDim2.new(0, 110, 0, 40)
ToggleButton.Font = Enum.Font.GothamBold
ToggleButton.Text = "MM2 Menu"
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.TextSize = 14
ToggleButton.Active = true
ToggleButton.Draggable = true

local ToggleCorner = Instance.new("UICorner")
ToggleCorner.CornerRadius = UDim.new(0, 8)
ToggleCorner.Parent = ToggleButton

-- สร้างกรอบหน้าต่างหลัก (Main Frame)
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Parent = ScreenGui
MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
MainFrame.Position = UDim2.new(0.05, 0, 0.17, 0)
MainFrame.Size = UDim2.new(0, 220, 0, 240)
MainFrame.Visible = true
MainFrame.Active = true
MainFrame.Draggable = true

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 10)
MainCorner.Parent = MainFrame

-- หัวข้อเมนู
local TitleLabel = Instance.new("TextLabel")
TitleLabel.Parent = MainFrame
TitleLabel.BackgroundTransparency = 1
TitleLabel.Size = UDim2.new(1, 0, 0, 35)
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.Text = "MM2 Hub[span_1](start_span)[span_1](end_span)"
TitleLabel.TextColor3 = Color3.fromRGB(255, 170, 0)
TitleLabel.TextSize = 16

-- ฟังก์ชันเปิด/ปิดหน้าต่างเมื่อกดปุ่ม Toggle
local isOpen = true
ToggleButton.MouseButton1Click:Connect(function()
    isOpen = not isOpen
    MainFrame.Visible = isOpen
    ToggleButton.Text = isOpen and "Hide Menu" or "Show Menu"
end)

-- ฟังก์ชันสร้างปุ่มตัวเลือกในเมนู
local function createButton(name, posY, callback)
    local btn = Instance.new("TextButton")
    btn.Parent = MainFrame
    btn.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
    btn.Position = UDim2.new(0.1, 0, 0, posY)
    btn.Size = UDim2.new(0, 176, 0, 38)
    btn.Font = Enum.Font.GothamSemibold
    btn.Text = name
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.TextSize = 13
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 6)
    corner.Parent = btn
    
    btn.MouseButton1Click:Connect(callback)
    return btn
end

-- ตัวแปรสถานะฟังก์ชันต่างๆ
local lockMurdererHead = false
local autoGrabGun = false
local revealRoles = false

----------------------------------------------------------------
-- 1. ล็อคหัวฆาตกร (Lock Murderer Head)
----------------------------------------------------------------
createButton("ล็อคหัวฆาตกร: ปิด", 45, function(btn)
    lockMurdererHead = not lockMurdererHead
    btn.Text = lockMurdererHead and "ล็อคหัวฆาตกร: เปิด" or "ล็อคหัวฆaตกร: ปิด"
    btn.TextColor3 = lockMurdererHead and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 255, 255)
end)

game:GetService("RunService").RenderStepped:Connect(function()
    if lockMurdererHead then
        pcall(function()
            local camera = workspace.CurrentCamera
            for _, player in ipairs(Players:GetPlayers()) do
                if player ~= LocalPlayer and player.Character then
                    local tool = player.Character:FindFirstChildOfClass("Tool") or player.Backpack:FindFirstChildOfClass("Tool")
                    if tool and (tool.Name == "Knife" or tool.Name == "Axe" or tool.Name == "Cutter") then
                        local head = player.Character:FindFirstChild("Head")
                        if head then
                            camera.CFrame = CFrame.new(camera.CFrame.Position, head.Position)
                        end
                    end
                end
            end
        end)
    end
end)

----------------------------------------------------------------
-- 2. ดึงปืนจากนายอำเภอที่ตายอัตโนมัติ (Auto Grab Gun)
----------------------------------------------------------------
createButton("ดึงปืนอัตโนมัติ: ปิด", 90, function(btn)
    autoGrabGun = not autoGrabGun
    btn.Text = autoGrabGun and "ดึงปืนอัตโนมัติ: เปิด" or "ดึงปืนอัตโนมัติ: ปิด"
    btn.TextColor3 = autoGrabGun and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 255, 255)
end)

task.spawn(function()
    while true do
        task.wait(0.5)
        if autoGrabGun then
            pcall(function()
                for _, obj in ipairs(workspace:GetChildren()) do
                    if obj.Name == "GunDrop" and obj:IsDescendantOf(workspace) then
                        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                            LocalPlayer.Character.HumanoidRootPart.CFrame = obj.CFrame
                        end
                    end
                end
            end)
        end
    end
end)

----------------------------------------------------------------
-- 3. เปิดโปงบทบาท (Reveal Roles)
----------------------------------------------------------------
createButton("เปิดโปงบทบาท: ปิด", 135, function(btn)
    revealRoles = not revealRoles
    btn.Text = revealRoles and "เปิดโปงบทบาท: เปิด" or "เปิดโปงบทบาท: ปิด"
    btn.TextColor3 = revealRoles and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 255, 255)
end)

task.spawn(function()
    while true do
        task.wait(1)
        if revealRoles then
            pcall(function()
                for _, player in ipairs(Players:GetPlayers()) do
                    if player ~= LocalPlayer and player.Character then
                        local character = player.Character
                        local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
                        if humanoidRootPart and not humanoidRootPart:FindFirstChild("RoleBillboard") then
                            local billboard = Instance.new("BillboardGui")
                            billboard.Name = "RoleBillboard"
                            billboard.Size = UDim2.new(0, 100, 0, 40)
                            billboard.StudsOffset = Vector3.new(0, 3, 0)
                            billboard.AlwaysOnTop = true
                            billboard.Parent = humanoidRootPart
                            
                            local label = Instance.new("TextLabel")
                            label.Size = UDim2.new(1, 0, 1, 0)
                            label.BackgroundTransparency = 1
                            label.TextColor3 = Color3.fromRGB(255, 255, 255)
                            label.TextStrokeTransparency = 0
                            label.TextSize = 14
                            label.Font = Enum.Font.GothamBold
                            label.Parent = billboard
                            
                            -- เช็คบทบาทจากไอเทมในตัว
                            task.spawn(function()
                                while billboard and billboard.Parent do
                                    local role = "ผู้บริสุทธิ์ (Innocent)"
                                    local roleColor = Color3.fromRGB(0, 255, 0)
                                    
                                    local pTool = character:FindFirstChildOfClass("Tool") or player.Backpack:FindFirstChildOfClass("Tool")
                                    if pTool then
                                        if pTool.Name == "Knife" or pTool.Name == "Axe" then
                                            role = "ฆาตกร (Murderer)"
                                            roleColor = Color3.fromRGB(255, 0, 0)
                                        elseif pTool.Name == "Gun" then
                                            role = "นายอำเภอ (Sheriff)"
                                            roleColor = Color3.fromRGB(0, 170, 255)
                                        end
                                    end
                                    label.Text = player.Name .. "\n[" .. role .. "]"
                                    label.TextColor3 = roleColor
                                    task.wait(1)
                                end
                            end)
                        end
                    end
                end
            end)
        else
            -- ลบป้ายบทบาทออกเมื่อปิด
            pcall(function()
                for _, player in ipairs(Players:GetPlayers()) do
                    if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                        local hrp = player.Character.HumanoidRootPart
                        if hrp:FindFirstChild("RoleBillboard") then
                            hrp.RoleBillboard:Destroy()
                        end
                    end
                end
            end)
        end
    end
end)

-- ปุ่มเครดิตด้านล่าง
local CreditLabel = Instance.new("TextLabel")
CreditLabel.Parent = MainFrame
CreditLabel.BackgroundTransparency = 1
CreditLabel.Position = UDim2.new(0, 0, 1, -25)
CreditLabel.Size = UDim2.new(1, 0, 0, 20)
CreditLabel.Font = Enum.Font.Gotham
CreditLabel.Text = "Status: Active"
CreditLabel.TextColor3 = Color3.fromRGB(150, 150, 150)
CreditLabel.TextSize = 11
