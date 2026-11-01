# Primehub
script
--========================================
-- PRIME HUB | AISLES OF THE LOCUST
-- VERSION 1.2
--========================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

local LP = Players.LocalPlayer

local Settings = {
    Players = false,
    Items = false,
    Locust = false
}

local ESP = {}
local Positions = {}
local MovingUntil = {}
local LocustTarget = nil
local LocustModels = {}
local LastScan = 0

--========================================
-- GUI
--========================================

local GUI = Instance.new("ScreenGui")
GUI.Name = "PRIME_HUB"
GUI.ResetOnSpawn = false

pcall(function()
    GUI.Parent = game:GetService("CoreGui")
end)

if not GUI.Parent then
    GUI.Parent = LP:WaitForChild("PlayerGui")
end

local Main = Instance.new("Frame")
Main.Size = UDim2.fromOffset(245, 245)
Main.Position = UDim2.new(0.5, -122, 0.5, -122)
Main.BackgroundColor3 = Color3.fromRGB(13, 13, 17)
Main.BorderSizePixel = 0
Main.Active = true
Main.Draggable = true
Main.Parent = GUI

Instance.new("UICorner", Main).CornerRadius =
    UDim.new(0, 12)

local Border = Instance.new("UIStroke", Main)
Border.Color = Color3.fromRGB(220, 170, 45)
Border.Thickness = 2

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 45)
Title.BackgroundTransparency = 1
Title.Text = "PRIME HUB"
Title.TextColor3 = Color3.fromRGB(235, 185, 65)
Title.TextSize = 23
Title.Font = Enum.Font.GothamBold
Title.Parent = Main

local Subtitle = Instance.new("TextLabel")
Subtitle.Position = UDim2.fromOffset(0, 38)
Subtitle.Size = UDim2.new(1, 0, 0, 20)
Subtitle.BackgroundTransparency = 1
Subtitle.Text = "AISLES OF THE LOCUST"
Subtitle.TextColor3 = Color3.fromRGB(150, 150, 150)
Subtitle.TextSize = 10
Subtitle.Parent = Main

--========================================
-- TOGGLES
--========================================

local function CreateButton(name, setting, y)

    local button = Instance.new("TextButton")
    button.Position = UDim2.fromOffset(15, y)
    button.Size = UDim2.new(1, -30, 0, 38)
    button.BackgroundColor3 = Color3.fromRGB(35, 35, 42)
    button.TextColor3 = Color3.new(1, 1, 1)
    button.TextSize = 13
    button.Font = Enum.Font.GothamBold
    button.Parent = Main

    Instance.new("UICorner", button).CornerRadius =
        UDim.new(0, 7)

    local function Update()
        button.Text = name .. " [" ..
            (Settings[setting] and "ON" or "OFF") .. "]"

        button.BackgroundColor3 = Settings[setting]
            and Color3.fromRGB(115, 75, 20)
            or Color3.fromRGB(35, 35, 42)
    end

    button.MouseButton1Click:Connect(function()
        Settings[setting] = not Settings[setting]
        Update()
    end)

    Update()
end

CreateButton("PLAYER ESP", "Players", 70)
CreateButton("ITEM ESP", "Items", 115)
CreateButton("THE LOCUST ESP", "Locust", 160)

--========================================
-- ESP FUNCTIONS
--========================================

local function GetPart(obj)

    if obj:IsA("BasePart") then
        return obj
    end

    if obj:IsA("Model") then
        return obj.PrimaryPart
            or obj:FindFirstChild("HumanoidRootPart")
            or obj:FindFirstChildWhichIsA("BasePart", true)
    end

end

local function RemoveESP(obj)

    local data = ESP[obj]
    if not data then return end

    if data.Highlight then
        data.Highlight:Destroy()
    end

    if data.Billboard then
        data.Billboard:Destroy()
    end

    ESP[obj] = nil
end

local function CreateESP(obj, label, color)

    if not obj or not obj.Parent then return end

    if ESP[obj] then
        ESP[obj].Label = label
        return
    end

    local part = GetPart(obj)
    if not part then return end

    local highlight = Instance.new("Highlight")
    highlight.Name = "PrimeHighlight"
    highlight.Adornee = obj
    highlight.FillColor = color
    highlight.OutlineColor = color
    highlight.FillTransparency = 0.65
    highlight.DepthMode =
        Enum.HighlightDepthMode.AlwaysOnTop
    highlight.Parent = obj

    local billboard = Instance.new("BillboardGui")
    billboard.Name = "PrimeTag"
    billboard.Adornee = part
    billboard.Size = UDim2.fromOffset(180, 45)
    billboard.StudsOffset = Vector3.new(0, 3, 0)
    billboard.AlwaysOnTop = true
    billboard.Parent = GUI

    local text = Instance.new("TextLabel")
    text.Size = UDim2.fromScale(1, 1)
    text.BackgroundTransparency = 1
    text.TextColor3 = color
    text.TextStrokeTransparency = 0
    text.TextSize = 14
    text.Font = Enum.Font.GothamBold
    text.Parent = billboard

    ESP[obj] = {
        Highlight = highlight,
        Billboard = billboard,
        Text = text,
        Label = label
    }
end

--========================================
-- LOCUST DETECTION
--========================================

local function RefreshLocusts()

    LocustModels = {}

    for _, obj in ipairs(Workspace:GetDescendants()) do

        if obj:IsA("Model")
        and obj.Name:lower():find("locust") then

            table.insert(LocustModels, obj)
        end
    end
end

local function DetectMovingLocust()

    local now = os.clock()
    local best = nil
    local bestSpeed = 1.5

    for _, obj in ipairs(LocustModels) do

        if not obj.Parent then
            continue
        end

        local part = GetPart(obj)

        if part then

            local position = obj:GetPivot().Position
            local previous = Positions[obj]

            if previous then

                local elapsed = now - previous.Time

                if elapsed > 0 then

                    local speed =
                        (position - previous.Position).Magnitude
                        / elapsed

                    if speed > 1.5 then

                        MovingUntil[obj] = now + 0.8

                    end
                end
            end

            Positions[obj] = {
                Position = position,
                Time = now
            }

            if MovingUntil[obj]
            and MovingUntil[obj] > now then

                local previousPosition = Positions[obj].Position

                if best == nil or MovingUntil[obj] > MovingUntil[best] then
                    best = obj
                end
            end
        end
    end

    return best
end

--========================================
-- MAIN LOOP
--========================================

task.spawn(function()

    while GUI.Parent do

        local now = os.clock()

        if now - LastScan >= 0.5 then
            RefreshLocusts()
            LastScan = now
        end

        if Settings.Locust then

            local target = DetectMovingLocust()

            if target ~= LocustTarget then

                if LocustTarget then
                    RemoveESP(LocustTarget)
                end

                LocustTarget = target
            end

            if LocustTarget then
                CreateESP(
                    LocustTarget,
                    "THE LOCUST",
                    Color3.fromRGB(255, 20, 20)
                )
            end

        else

            if LocustTarget then
                RemoveESP(LocustTarget)
                LocustTarget = nil
            end

        end

        -- PLAYERS
        if Settings.Players then

            for _, player in ipairs(Players:GetPlayers()) do

                if player ~= LP and player.Character then
                    CreateESP(
                        player.Character,
                        player.Name,
                        Color3.fromRGB(50, 170, 255)
                    )
                end
            end
        end

        -- ITEMS
        if Settings.Items then

            for _, obj in ipairs(Workspace:GetDescendants()) do

                if obj:IsA("Model") then

                    local name = obj.Name:lower()

                    if name:find("item")
                    or name:find("key")
                    or name:find("battery") then

                        CreateESP(
                            obj,
                            obj.Name,
                            Color3.fromRGB(255, 210, 50)
                        )
                    end
                end
            end
        end

        -- DISTANCE
        local character = LP.Character
        local root = character and
            character:FindFirstChild("HumanoidRootPart")

        for obj, data in pairs(ESP) do

            local part = GetPart(obj)

            if not obj.Parent then
                RemoveESP(obj)

            elseif part and root then

                local distance =
                    (root.Position - part.Position).Magnitude

                data.Text.Text =
                    data.Label .. "\n" ..
                    math.floor(distance) .. " studs"
            end
        end

        task.wait(0.1)
    end
end)

print("PRIME HUB v1.2 LOADED")
