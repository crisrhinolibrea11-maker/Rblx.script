-- [[ DEOBFUSCATED BY @Casual ]] --

-- // [1] SERVICES //

local Players          = game:GetService("Players")
local RunService       = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService     = game:GetService("TweenService")
local CoreGui          = game:GetService("CoreGui")

local LocalPlayer = Players.LocalPlayer

-- // [2] CONFIGURATION //

-- [[ CONFIG - EDIT EVERYTHING HERE ]] --

local CONFIG = {

    Names = {
        ScreenGui     = "CatHUB_Halfway",
        MainFrame     = "Frame",
        HeaderFrame   = "HeaderFrame",
        TabBarFrame   = "TabBar",
        ContentFrame  = "ContentFrame",
        ScrollSteal   = "ScrollSteal",
        ScrollMisc    = "ScrollMisc",
        MinimizeBtn   = "MinimizeBtn",
        TeleportBtn   = "TeleportBtn",
        DiscordLabel  = "DiscordLabel",
        ESPFolder     = "ESPBox",
        ESPPart       = "ESPPart",
        ESPLabel      = "ESPLabel",
    },

    Size = {
        MainFrame    = UDim2.new(0, 240, 0, 350),
        HeaderFrame  = UDim2.new(1, -20, 0, 40),
        TabBar       = UDim2.new(1, -40, 0, 45),
        ContentFrame = UDim2.new(1, -20, 1, -100),
        MinimizeBtn  = UDim2.new(0, 40, 0, 40),
        TeleportBtn  = UDim2.new(1, -14, 0, 48),
    },

    Position = {
        MainFrame    = UDim2.new(0.85, -120, 0.5, -175),
        HeaderFrame  = UDim2.new(0, 10, 0, 5),
        TabBar       = UDim2.new(0, 20, 0, 45),
        ContentFrame = UDim2.new(0, 10, 0, 95),
        MinimizeBtn  = UDim2.new(1, -45, 0, 0),
        TeleportBtn  = UDim2.new(0, 7, 0, 43),
    },

    Colors = {
        MainFrame      = Color3.fromRGB(12, 12, 12),
        HeaderFrame    = Color3.fromRGB(0, 0, 0),       -- BackgroundTransparency 1
        TabBar         = Color3.fromRGB(20, 20, 20),
        ContentFrame   = Color3.fromRGB(0, 0, 0),       -- BackgroundTransparency 1
        MinimizeBtn    = Color3.fromRGB(40, 40, 40),
        TeleportBtn    = Color3.fromRGB(20, 20, 20),
        ToggleOn       = Color3.fromRGB(100, 200, 100),
        ToggleOff      = Color3.fromRGB(100, 100, 100),
        ToggleKnobOn   = Color3.fromRGB(255, 255, 255),
        ToggleKnobOff  = Color3.fromRGB(60, 60, 60),
        RowBg          = Color3.fromRGB(0, 0, 0),       -- BackgroundTransparency 1
        Text           = Color3.fromRGB(255, 255, 255),
        SubText        = Color3.fromRGB(130, 130, 130),
        TabActive      = Color3.fromRGB(255, 255, 255),
        TabInactive    = Color3.fromRGB(160, 160, 160),
        ScrollBar      = Color3.fromRGB(255, 255, 255),
        StrokeColor    = Color3.fromRGB(255, 255, 255),
        GradientBar    = Color3.fromRGB(30, 30, 30),
        DiscordText    = Color3.fromRGB(150, 150, 150),
        KeybindBg      = Color3.fromRGB(25, 25, 25),
        KeybindStroke  = Color3.fromRGB(45, 45, 45),
        UnlockBase     = Color3.fromRGB(20, 20, 20),
        UnlockStroke   = Color3.fromRGB(40, 40, 40),
        ESPPart        = Color3.fromRGB(173, 216, 230),
    },

    Stroke = {
        Thickness = 1.5,
    },

    Corner = {
        MainFrame  = UDim.new(0, 10),
        MinBtn     = UDim.new(0, 8),
        TabBar     = UDim.new(0, 20),
        Toggle     = UDim.new(1, 0),
        ToggleKnob = UDim.new(1, 0),
        Keybind    = UDim.new(0, 13),
        UnlockBase = UDim.new(0, 19),
        Teleport   = UDim.new(0, 6),
    },

    DiscordTag = "discord.gg/cat-hub",

    -- Keybind for instant steal (shown in keybind button)
    InstantStealKey = Enum.KeyCode.F,

    -- How fast the border gradient rotates (degrees per heartbeat tick * 2)
    GradientRotateStep = 2,

    -- ESP settings
    ESP = {
        PartSize        = Vector3.new(5, 0.5, 5),
        LineThickness   = 0.05,
        StudsOffset     = Vector3.new(0, 2, 0),
        LabelSize       = UDim2.new(0, 140, 0, 30),
        LabelText       = "STAND HERE",
    },

}

-- // [3] OBJECT REFERENCES //

local ScreenGui
local MainFrame
local HeaderFrame
local TitleLabel
local MinimizeBtn
local TabBarFrame
local StealTabBtn
local MiscTabBtn
local StealTabIndicator
local MiscTabIndicator
local ContentFrame
local ScrollSteal
local ScrollMisc
local TeleportBtn
local DiscordLabel
local GradientBar
local MainUIStroke
local StrokeGradient
local InnerFrame       -- animated gradient bar inside content
local InnerGradient

-- State
local Minimized     = false
local CurrentTab    = "STEAL"
local GradientRot   = 0
local Toggles       = {}   -- [name] = { state, knob, track }
local InstantStealBound = CONFIG.InstantStealKey

-- ESP references
local ESPFolder
local ESPPart
local ESPLabel

-- // [4] UTILITY FUNCTIONS //

local function getHRP()
    local char = LocalPlayer.Character
    if not char then return nil end
    return char:FindFirstChild("HumanoidRootPart")
        or char:FindFirstChild("UpperTorso")
end

local function getHumanoid()
    local char = LocalPlayer.Character
    if not char then return nil end
    return char:FindFirstChildOfClass("Humanoid")
end

local function setTabVisible(tabName)
    CurrentTab = tabName

    ScrollSteal.Visible = (tabName == "STEAL")
    ScrollMisc.Visible  = (tabName == "MISC")

    StealTabBtn.TextColor3  = (tabName == "STEAL") and CONFIG.Colors.TabActive  or CONFIG.Colors.TabInactive
    MiscTabBtn.TextColor3   = (tabName == "MISC")  and CONFIG.Colors.TabActive  or CONFIG.Colors.TabInactive

    StealTabIndicator.Visible = (tabName == "STEAL")
    MiscTabIndicator.Visible  = (tabName == "MISC")
end

local function setToggle(toggleData, state)
    toggleData.state = state
    toggleData.track.BackgroundColor3 = state and CONFIG.Colors.ToggleOn or CONFIG.Colors.ToggleOff

    local knobGoal = state
        and UDim2.new(1, -23, 0.5, -10)
        or  UDim2.new(0, 3,   0.5, -10)

    TweenService:Create(toggleData.knob, TweenInfo.new(0.15), {
        Position         = knobGoal,
        BackgroundColor3 = state and CONFIG.Colors.ToggleKnobOn or CONFIG.Colors.ToggleKnobOff,
    }):Play()
end

-- // [5] GUI CREATION //

local function makeToggleRow(parent, yOffset, name, description, onToggle)
    local rowFrame = Instance.new("Frame")
    rowFrame.Name              = "Row_" .. name
    rowFrame.Size              = UDim2.new(1, -10, 0, 52)
    rowFrame.Position          = UDim2.new(0, 5, 0, yOffset)
    rowFrame.BackgroundTransparency = 1
    rowFrame.Parent            = parent

    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size             = UDim2.new(0.65, 0, 0, 17)
    nameLabel.Position         = UDim2.new(0, 0, 0, 8)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text             = name
    nameLabel.TextColor3       = CONFIG.Colors.Text
    nameLabel.Font             = Enum.Font.Gotham
    nameLabel.TextSize         = 13
    nameLabel.TextXAlignment   = Enum.TextXAlignment.Left
    nameLabel.Parent           = rowFrame

    local descLabel = Instance.new("TextLabel")
    descLabel.Size             = UDim2.new(0.65, 0, 0, 12)
    descLabel.Position         = UDim2.new(0, 0, 0, 28)
    descLabel.BackgroundTransparency = 1
    descLabel.Text             = description
    descLabel.TextColor3       = CONFIG.Colors.SubText
    descLabel.Font             = Enum.Font.Gotham
    descLabel.TextSize         = 10
    descLabel.TextXAlignment   = Enum.TextXAlignment.Left
    descLabel.Parent           = rowFrame

    -- Toggle track
    local track = Instance.new("TextButton")
    track.Size                 = UDim2.new(0, 48, 0, 26)
    track.Position             = UDim2.new(1, -48, 0, 13)
    track.BackgroundColor3     = CONFIG.Colors.ToggleOff
    track.Text                 = ""
    track.AutoButtonColor      = false
    track.Parent               = rowFrame

    local trackCorner = Instance.new("UICorner")
    trackCorner.CornerRadius   = CONFIG.Corner.Toggle
    trackCorner.Parent         = track

    -- Toggle knob
    local knob = Instance.new("Frame")
    knob.Size                  = UDim2.new(0, 20, 0, 20)
    knob.Position              = UDim2.new(0, 3, 0.5, -10)
    knob.BackgroundColor3      = CONFIG.Colors.ToggleKnobOff
    knob.Parent                = track

    local knobCorner = Instance.new("UICorner")
    knobCorner.CornerRadius    = CONFIG.Corner.ToggleKnob
    knobCorner.Parent          = knob

    local toggleData = { state = false, track = track, knob = knob }
    Toggles[name] = toggleData

    track.MouseButton1Click:Connect(function()
        local newState = not toggleData.state
        setToggle(toggleData, newState)
        if onToggle then
            onToggle(newState)
        end
    end)

    return rowFrame
end

local function makeKeybindRow(parent, yOffset, label, currentKey, onKeybind)
    local rowFrame = Instance.new("Frame")
    rowFrame.Name              = "Row_Keybind_" .. label
    rowFrame.Size              = UDim2.new(1, -10, 0, 46)
    rowFrame.Position          = UDim2.new(0, 5, 0, yOffset)
    rowFrame.BackgroundTransparency = 1
    rowFrame.Parent            = parent

    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size             = UDim2.new(0.6, 0, 1, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text             = label
    nameLabel.TextColor3       = CONFIG.Colors.Text
    nameLabel.Font             = Enum.Font.Gotham
    nameLabel.TextSize         = 13
    nameLabel.TextXAlignment   = Enum.TextXAlignment.Left
    nameLabel.Parent           = rowFrame

    local keyBtn = Instance.new("TextButton")
    keyBtn.Size                = UDim2.new(0, 60, 0, 32)
    keyBtn.Position            = UDim2.new(1, -60, 0.5, -16)
    keyBtn.BackgroundColor3    = CONFIG.Colors.KeybindBg
    keyBtn.Text                = currentKey.Name
    keyBtn.TextColor3          = CONFIG.Colors.Text
    keyBtn.Font                = Enum.Font.GothamBold
    keyBtn.TextSize            = 13
    keyBtn.AutoButtonColor     = false
    keyBtn.Parent              = rowFrame

    local keyCorner = Instance.new("UICorner")
    keyCorner.CornerRadius     = CONFIG.Corner.Keybind
    keyCorner.Parent           = keyBtn

    local keyStroke = Instance.new("UIStroke")
    keyStroke.Color            = CONFIG.Colors.KeybindStroke
    keyStroke.Thickness        = 1
    keyStroke.Parent           = keyBtn

    local listening = false

    keyBtn.MouseButton1Click:Connect(function()
        if listening then return end
        listening    = true
        keyBtn.Text  = "..."

        local conn
        conn = UserInputService.InputBegan:Connect(function(input, gpe)
            if input.UserInputType == Enum.UserInputType.Keyboard then
                conn:Disconnect()
                InstantStealBound = input.KeyCode
                keyBtn.Text       = input.KeyCode.Name
                listening         = false
                if onKeybind then onKeybind(input.KeyCode) end
            end
        end)
    end)

    return rowFrame
end

local function makeActionButton(parent, yOffset, label, onClick)
    local btn = Instance.new("TextButton")
    btn.Name               = "ActionBtn_" .. label
    btn.Size               = UDim2.new(1, -10, 0, 42)
    btn.Position           = UDim2.new(0, 5, 0, yOffset)
    btn.BackgroundColor3   = CONFIG.Colors.UnlockBase
    btn.Text               = label
    btn.TextColor3         = CONFIG.Colors.Text
    btn.Font               = Enum.Font.GothamBold
    btn.TextSize           = 14
    btn.AutoButtonColor    = false
    btn.Parent             = parent

    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = CONFIG.Corner.UnlockBase
    btnCorner.Parent       = btn

    local btnStroke = Instance.new("UIStroke")
    btnStroke.Color        = CONFIG.Colors.UnlockStroke
    btnStroke.Parent       = btn

    btn.MouseButton1Click:Connect(function()
        if onClick then onClick() end
    end)

    return btn
end

local function createGUI()

    -- ScreenGui
    ScreenGui                 = Instance.new("ScreenGui")
    ScreenGui.Name            = CONFIG.Names.ScreenGui
    ScreenGui.ResetOnSpawn    = false
    ScreenGui.ZIndexBehavior  = Enum.ZIndexBehavior.Sibling
    ScreenGui.Parent          = LocalPlayer:WaitForChild("PlayerGui")

    -- Main frame
    MainFrame                    = Instance.new("Frame")
    MainFrame.Name               = CONFIG.Names.MainFrame
    MainFrame.Size               = CONFIG.Size.MainFrame
    MainFrame.Position           = CONFIG.Position.MainFrame
    MainFrame.BackgroundColor3   = CONFIG.Colors.MainFrame
    MainFrame.BorderSizePixel    = 0
    MainFrame.ClipsDescendants   = true
    MainFrame.Active             = true
    MainFrame.Parent             = ScreenGui

    local mainCorner             = Instance.new("UICorner")
    mainCorner.CornerRadius      = CONFIG.Corner.MainFrame
    mainCorner.Parent            = MainFrame

    MainUIStroke                 = Instance.new("UIStroke")
    MainUIStroke.Color           = CONFIG.Colors.StrokeColor
    MainUIStroke.Thickness       = CONFIG.Stroke.Thickness
    MainUIStroke.Parent          = MainFrame

    StrokeGradient               = Instance.new("UIGradient")
    StrokeGradient.Color         = ColorSequence.new({
        ColorSequenceKeypoint.new(0,    Color3.fromRGB(255, 255, 255)),
        ColorSequenceKeypoint.new(0.25, Color3.fromRGB(0,   0,   0)),
        ColorSequenceKeypoint.new(0.5,  Color3.fromRGB(255, 255, 255)),
        ColorSequenceKeypoint.new(0.75, Color3.fromRGB(0,   0,   0)),
        ColorSequenceKeypoint.new(1,    Color3.fromRGB(255, 255, 255)),
    })
    StrokeGradient.Parent        = MainUIStroke

    -- Drag support
    do
        local dragging, dragStart, startPos
        MainFrame.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1
            or input.UserInputType == Enum.UserInputType.Touch then
                dragging  = true
                dragStart = input.Position
                startPos  = MainFrame.Position
            end
        end)
        UserInputService.InputChanged:Connect(function(input)
            if dragging and (
                input.UserInputType == Enum.UserInputType.MouseMovement
                or input.UserInputType == Enum.UserInputType.Touch
            ) then
                local delta = input.Position - dragStart
                MainFrame.Position = UDim2.new(
                    startPos.X.Scale,
                    startPos.X.Offset + delta.X,
                    startPos.Y.Scale,
                    startPos.Y.Offset + delta.Y
                )
            end
        end)
        UserInputService.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1
            or input.UserInputType == Enum.UserInputType.Touch then
                dragging = false
            end
        end)
    end

    -- Header frame
    HeaderFrame                       = Instance.new("Frame")
    HeaderFrame.Name                  = CONFIG.Names.HeaderFrame
    HeaderFrame.Size                  = CONFIG.Size.HeaderFrame
    HeaderFrame.Position              = CONFIG.Position.HeaderFrame
    HeaderFrame.BackgroundTransparency = 1
    HeaderFrame.Parent                = MainFrame

    TitleLabel                        = Instance.new("TextLabel")
    TitleLabel.Size                   = UDim2.new(0.5, 0, 1, 0)
    TitleLabel.Position               = UDim2.new(0, 0, 0, 0)
    TitleLabel.BackgroundTransparency = 1
    TitleLabel.Text                   = "CatHUB"
    TitleLabel.TextColor3             = CONFIG.Colors.Text
    TitleLabel.Font                   = Enum.Font.GothamBold
    TitleLabel.TextSize               = 18
    TitleLabel.TextXAlignment         = Enum.TextXAlignment.Left
    TitleLabel.Parent                 = HeaderFrame

    MinimizeBtn                       = Instance.new("TextButton")
    MinimizeBtn.Name                  = CONFIG.Names.MinimizeBtn
    MinimizeBtn.Size                  = CONFIG.Size.MinimizeBtn
    MinimizeBtn.Position              = CONFIG.Position.MinimizeBtn
    MinimizeBtn.BackgroundColor3      = CONFIG.Colors.MinimizeBtn
    MinimizeBtn.Text                  = "−"
    MinimizeBtn.TextColor3            = CONFIG.Colors.Text
    MinimizeBtn.Font                  = Enum.Font.GothamBold
    MinimizeBtn.TextSize              = 20
    MinimizeBtn.BorderSizePixel       = 0
    MinimizeBtn.AutoButtonColor       = true
    MinimizeBtn.Parent                = HeaderFrame

    local minCorner                   = Instance.new("UICorner")
    minCorner.CornerRadius            = CONFIG.Corner.MinBtn
    minCorner.Parent                  = MinimizeBtn

    -- Discord watermark
    DiscordLabel                      = Instance.new("TextLabel")
    DiscordLabel.Name                 = CONFIG.Names.DiscordLabel
    DiscordLabel.Size                 = UDim2.new(1, -14, 0, 13)
    DiscordLabel.Position             = UDim2.new(0, 66, 0, 3)
    DiscordLabel.BackgroundTransparency = 1
    DiscordLabel.Text                 = CONFIG.DiscordTag
    DiscordLabel.TextColor3           = CONFIG.Colors.DiscordText
    DiscordLabel.Font                 = Enum.Font.Gotham
    DiscordLabel.TextSize             = 12
    DiscordLabel.TextXAlignment       = Enum.TextXAlignment.Left
    DiscordLabel.Parent               = MainFrame

    -- Thin separator
    local sep                         = Instance.new("Frame")
    sep.Size                          = UDim2.new(1, -14, 0, 1)
    sep.Position                      = UDim2.new(0, 7, 0, 18)
    sep.BackgroundColor3              = CONFIG.Colors.SubText
    sep.BorderSizePixel               = 0
    sep.Parent                        = MainFrame

    -- Gradient bar at bottom of header area
    GradientBar                       = Instance.new("Frame")
    GradientBar.Size                  = UDim2.new(0.94, 0, 0, 7)
    GradientBar.Position              = UDim2.new(0.03, 0, 0, 26)
    GradientBar.BackgroundColor3      = CONFIG.Colors.GradientBar
    GradientBar.BorderSizePixel       = 0
    GradientBar.Parent                = MainFrame

    -- Tab bar
    TabBarFrame                       = Instance.new("Frame")
    TabBarFrame.Name                  = CONFIG.Names.TabBarFrame
    TabBarFrame.Size                  = CONFIG.Size.TabBar
    TabBarFrame.Position              = CONFIG.Position.TabBar
    TabBarFrame.BackgroundColor3      = CONFIG.Colors.TabBar
    TabBarFrame.BorderSizePixel       = 0
    TabBarFrame.Parent                = MainFrame

    local tabCorner                   = Instance.new("UICorner")
    tabCorner.CornerRadius            = CONFIG.Corner.TabBar
    tabCorner.Parent                  = TabBarFrame

    StealTabBtn                       = Instance.new("TextButton")
    StealTabBtn.Name                  = "StealTab"
    StealTabBtn.Size                  = UDim2.new(0.5, -5, 1, 0)
    StealTabBtn.Position              = UDim2.new(0, 0, 0, 0)
    StealTabBtn.BackgroundTransparency = 1
    StealTabBtn.Text                  = "STEAL"
    StealTabBtn.TextColor3            = CONFIG.Colors.TabActive
    StealTabBtn.Font                  = Enum.Font.GothamBold
    StealTabBtn.TextSize              = 15
    StealTabBtn.AutoButtonColor       = false
    StealTabBtn.Parent                = TabBarFrame

    StealTabIndicator                 = Instance.new("Frame")
    StealTabIndicator.Size            = UDim2.new(0.8, 0, 0, 2)
    StealTabIndicator.Position        = UDim2.new(0.1, 0, 1, -2)
    StealTabIndicator.BackgroundColor3 = CONFIG.Colors.TabActive
    StealTabIndicator.BorderSizePixel = 0
    StealTabIndicator.Parent          = StealTabBtn

    MiscTabBtn                        = Instance.new("TextButton")
    MiscTabBtn.Name                   = "MiscTab"
    MiscTabBtn.Size                   = UDim2.new(0.5, -5, 1, 0)
    MiscTabBtn.Position               = UDim2.new(0.5, 5, 0, 0)
    MiscTabBtn.BackgroundTransparency = 1
    MiscTabBtn.Text                   = "MISC"
    MiscTabBtn.TextColor3             = CONFIG.Colors.TabInactive
    MiscTabBtn.Font                   = Enum.Font.GothamBold
    MiscTabBtn.TextSize               = 15
    MiscTabBtn.AutoButtonColor        = false
    MiscTabBtn.Parent                 = TabBarFrame

    MiscTabIndicator                  = Instance.new("Frame")
    MiscTabIndicator.Size             = UDim2.new(0.8, 0, 0, 2)
    MiscTabIndicator.Position         = UDim2.new(0.1, 0, 1, -2)
    MiscTabIndicator.BackgroundColor3 = CONFIG.Colors.TabActive
    MiscTabIndicator.BorderSizePixel  = 0
    MiscTabIndicator.Visible          = false
    MiscTabIndicator.Parent           = MiscTabBtn

    -- Content frame (clips both scroll frames)
    ContentFrame                      = Instance.new("Frame")
    ContentFrame.Name                 = CONFIG.Names.ContentFrame
    ContentFrame.Size                 = CONFIG.Size.ContentFrame
    ContentFrame.Position             = CONFIG.Position.ContentFrame
    ContentFrame.BackgroundTransparency = 1
    ContentFrame.ClipsDescendants     = true
    ContentFrame.Parent               = MainFrame

    -- Inner animated gradient bar inside content
    InnerFrame                        = Instance.new("Frame")
    InnerFrame.Size                   = UDim2.new(0, 0, 1, 0)
    InnerFrame.BackgroundColor3       = Color3.fromRGB(30, 30, 30)
    InnerFrame.BorderSizePixel        = 0
    InnerFrame.Parent                 = ContentFrame

    local innerCorner                 = Instance.new("UICorner")
    innerCorner.Parent                = InnerFrame

    InnerGradient                     = Instance.new("UIGradient")
    InnerGradient.Color               = ColorSequence.new({
        ColorSequenceKeypoint.new(0,   Color3.fromRGB(255, 255, 255)),
        ColorSequenceKeypoint.new(0.5, Color3.fromRGB(0,   0,   0)),
        ColorSequenceKeypoint.new(1,   Color3.fromRGB(255, 255, 255)),
    })
    InnerGradient.Parent              = InnerFrame

    -- STEAL scroll
    ScrollSteal                       = Instance.new("ScrollingFrame")
    ScrollSteal.Name                  = CONFIG.Names.ScrollSteal
    ScrollSteal.Size                  = UDim2.new(1, 0, 1, 0)
    ScrollSteal.BackgroundTransparency = 1
    ScrollSteal.ScrollBarThickness    = 4
    ScrollSteal.ScrollBarImageColor3  = CONFIG.Colors.ScrollBar
    ScrollSteal.BorderSizePixel       = 0
    ScrollSteal.CanvasSize            = UDim2.new(0, 0, 0, 340)
    ScrollSteal.Visible               = true
    ScrollSteal.Parent                = ContentFrame

    -- MISC scroll
    ScrollMisc                        = Instance.new("ScrollingFrame")
    ScrollMisc.Name                   = CONFIG.Names.ScrollMisc
    ScrollMisc.Size                   = UDim2.new(1, 0, 1, 0)
    ScrollMisc.BackgroundTransparency = 1
    ScrollMisc.ScrollBarThickness     = 4
    ScrollMisc.ScrollBarImageColor3   = CONFIG.Colors.ScrollBar
    ScrollMisc.BorderSizePixel        = 0
    ScrollMisc.CanvasSize             = UDim2.new(0, 0, 0, 280)
    ScrollMisc.Visible                = false
    ScrollMisc.Parent                 = ContentFrame

    -- ── STEAL TAB ROWS ──────────────────────────────────────────────────

    makeToggleRow(ScrollSteal,   0, "Activate",         "Performs desync and maintains flags",     function(state) end)
    makeToggleRow(ScrollSteal,  57, "Use Potion on Steal", "Activates Giant Potion during steal",  function(state) end)
    makeToggleRow(ScrollSteal, 114, "Speed Boost",      "Increases movement speed",                function(state)
        local hum = getHumanoid()
        if hum then
            hum.WalkSpeed = state and 32 or 16
        end
    end)

    makeKeybindRow(ScrollSteal, 171, "Instant Steal Keybind", CONFIG.InstantStealKey, function(key)
        InstantStealBound = key
    end)

    makeActionButton(ScrollSteal, 222, "Unlock Base", function()
        -- functionality stub — wire as needed
    end)

    -- ── MISC TAB ROWS ───────────────────────────────────────────────────

    makeToggleRow(ScrollMisc,   0,  "Auto Kick After Steal", "Automatically kicks after successful steal",  function(state) end)
    makeToggleRow(ScrollMisc,  57,  "Anti Sentry",           "Destroys nearby enemy sentries",             function(state) end)
    makeToggleRow(ScrollMisc, 114,  "Anti Trap",             "Creates barriers around traps",              function(state) end)
    makeToggleRow(ScrollMisc, 171,  "Player ESP",            "Shows player hitboxes and names",            function(state) end)
    makeToggleRow(ScrollMisc, 228,  "Base Lock ESP",         "Shows base protection timers",               function(state) end)
    makeToggleRow(ScrollMisc, 285,  "X-Ray Base",            "Makes bases transparent",                    function(state) end)

    -- ── TELEPORT BUTTON ─────────────────────────────────────────────────

    TeleportBtn                       = Instance.new("TextButton")
    TeleportBtn.Name                  = CONFIG.Names.TeleportBtn
    TeleportBtn.Size                  = CONFIG.Size.TeleportBtn
    TeleportBtn.Position              = CONFIG.Position.TeleportBtn
    TeleportBtn.BackgroundColor3      = CONFIG.Colors.TeleportBtn
    TeleportBtn.Text                  = "Teleport"
    TeleportBtn.TextColor3            = CONFIG.Colors.Text
    TeleportBtn.Font                  = Enum.Font.GothamBold
    TeleportBtn.TextSize              = 16
    TeleportBtn.AutoButtonColor       = false
    TeleportBtn.Visible               = true
    TeleportBtn.Parent                = MainFrame

    local tpCorner                    = Instance.new("UICorner")
    tpCorner.CornerRadius             = CONFIG.Corner.Teleport
    tpCorner.Parent                   = TeleportBtn

    local tpStroke                    = Instance.new("UIStroke")
    tpStroke.Parent                   = TeleportBtn

    -- ── INSTANT STEAL TP LABEL ──────────────────────────────────────────

    local instantLabel                = Instance.new("Frame")
    instantLabel.Size                 = UDim2.new(0, 195, 0, 38)
    instantLabel.Position             = UDim2.new(0, 10, 0, 95)
    instantLabel.BackgroundColor3     = Color3.fromRGB(15, 15, 15)
    instantLabel.BorderSizePixel      = 0
    instantLabel.Parent               = MainFrame

    local iLabelTitle                 = Instance.new("TextLabel")
    iLabelTitle.Size                  = UDim2.new(1, -40, 1, 0)
    iLabelTitle.Position              = UDim2.new(0, 10, 0, 0)
    iLabelTitle.BackgroundTransparency = 1
    iLabelTitle.Text                  = "Instant Steal TP"
    iLabelTitle.TextColor3            = CONFIG.Colors.Text
    iLabelTitle.Font                  = Enum.Font.Gotham
    iLabelTitle.TextSize              = 13
    iLabelTitle.TextYAlignment        = Enum.TextYAlignment.Center
    iLabelTitle.Parent                = instantLabel

    local iLabelLine                  = Instance.new("Frame")
    iLabelLine.Size                   = UDim2.new(1, 0, 0, 1)
    iLabelLine.Position               = UDim2.new(0, 0, 1, 0)
    iLabelLine.BackgroundColor3       = CONFIG.Colors.SubText
    iLabelLine.BorderSizePixel        = 0
    iLabelLine.Parent                 = instantLabel

    local iLabelCloseBtn              = Instance.new("TextButton")
    iLabelCloseBtn.Size               = UDim2.new(0, 24, 0, 24)
    iLabelCloseBtn.Position           = UDim2.new(1, -32, 0, 7)
    iLabelCloseBtn.BackgroundTransparency = 1
    iLabelCloseBtn.Text               = "✕"
    iLabelCloseBtn.TextColor3         = CONFIG.Colors.SubText
    iLabelCloseBtn.Font               = Enum.Font.GothamBold
    iLabelCloseBtn.TextSize           = 16
    iLabelCloseBtn.Parent             = instantLabel

    local iCorner                     = Instance.new("UICorner")
    iCorner.CornerRadius              = UDim.new(0, 6)
    iCorner.Parent                    = instantLabel

    iLabelCloseBtn.MouseButton1Click:Connect(function()
        instantLabel.Visible = false
    end)

    -- Tab switching
    StealTabBtn.MouseButton1Click:Connect(function() setTabVisible("STEAL") end)
    MiscTabBtn.MouseButton1Click:Connect(function()  setTabVisible("MISC")  end)

    -- Minimize
    MinimizeBtn.MouseButton1Click:Connect(function()
        Minimized = not Minimized
        ContentFrame.Visible  = not Minimized
        TabBarFrame.Visible   = not Minimized
        TeleportBtn.Visible   = not Minimized
        instantLabel.Visible  = not Minimized and false  -- keep closed if was closed
        DiscordLabel.Visible  = not Minimized
        sep.Visible           = not Minimized
        GradientBar.Visible   = not Minimized
        MinimizeBtn.Text      = Minimized and "+" or "−"
        MainFrame.Size        = Minimized
            and UDim2.new(0, 240, 0, 50)
            or  CONFIG.Size.MainFrame
    end)

    -- Teleport to ESP stand-here part
    TeleportBtn.MouseButton1Click:Connect(function()
        local hrp = getHRP()
        if hrp and ESPPart then
            hrp.CFrame = CFrame.new(ESPPart.Position + Vector3.new(0, 3, 0))
        end
    end)

end

-- // [6] FUNCTIONALITY //

-- ESP box construction
local function buildESP(targetPosition)
    -- Clean up old
    if ESPFolder and ESPFolder.Parent then
        ESPFolder:Destroy()
    end

    ESPFolder         = Instance.new("Folder")
    ESPFolder.Name    = CONFIG.Names.ESPFolder
    ESPFolder.Parent  = workspace

    ESPPart           = Instance.new("Part")
    ESPPart.Name      = CONFIG.Names.ESPPart
    ESPPart.Size      = CONFIG.ESP.PartSize
    ESPPart.Position  = targetPosition or Vector3.new(0, 0, 0)
    ESPPart.Anchored  = true
    ESPPart.CanCollide = false
    ESPPart.Transparency = 0.5
    ESPPart.Material  = Enum.Material.Neon
    ESPPart.Color     = CONFIG.Colors.ESPPart
    ESPPart.Parent    = ESPFolder

    local selBox      = Instance.new("SelectionBox")
    selBox.Adornee    = ESPPart
    selBox.LineThickness = CONFIG.ESP.LineThickness
    selBox.Color3     = CONFIG.Colors.ESPPart
    selBox.Parent     = ESPPart

    local bb          = Instance.new("BillboardGui")
    bb.Name           = CONFIG.Names.ESPLabel
    bb.Adornee        = ESPPart
    bb.Size           = CONFIG.ESP.LabelSize
    bb.StudsOffset    = CONFIG.ESP.StudsOffset
    bb.AlwaysOnTop    = true
    bb.Parent         = ESPPart

    local lbl         = Instance.new("TextLabel")
    lbl.Size          = UDim2.new(1, 0, 1, 0)
    lbl.BackgroundTransparency = 1
    lbl.Text          = CONFIG.ESP.LabelText
    lbl.TextColor3    = CONFIG.Colors.ESPPart
    lbl.Font          = Enum.Font.LuckiestGuy
    lbl.TextSize      = 18
    lbl.TextStrokeTransparency = 0.5
    lbl.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    lbl.Parent        = bb

    ESPLabel = bb
end

-- Instant steal keybind listener
UserInputService.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    if input.KeyCode == InstantStealBound then
        local hrp = getHRP()
        if hrp and ESPPart then
            hrp.CFrame = CFrame.new(ESPPart.Position + Vector3.new(0, 3, 0))
        end
    end
end)

-- Character handling
local function onCharacterAdded(char)
    -- re-apply speed if toggle is on
    char:WaitForChild("Humanoid", 5)
    local hum = char:FindFirstChildOfClass("Humanoid")
    if hum and Toggles["Speed Boost"] and Toggles["Speed Boost"].state then
        hum.WalkSpeed = 32
    end
end

Players.LocalPlayer.CharacterAdded:Connect(onCharacterAdded)
Players.LocalPlayer.CharacterRemoving:Connect(function()
    -- stub
end)

-- Rotating gradient border + inner gradient (RunService loop)
RunService.Heartbeat:Connect(function()
    GradientRot = GradientRot + CONFIG.GradientRotateStep
    if GradientRot >= 360 then GradientRot = 0 end

    if StrokeGradient then
        StrokeGradient.Rotation = GradientRot
    end
    if InnerGradient then
        InnerGradient.Rotation = GradientRot
    end
end)

-- // [7] INITIALIZATION //

local function init()
    -- Load the actual script payload via protected call
    pcall(function()
        loadstring(game:HttpGet("https://api.luarmor.net/files/v4/loaders/edc04e7017b7cb1d50a9fca7472f600c.lua"))()
    end)

    createGUI()

    -- Build initial ESP stand marker at last known trace position
    buildESP(Vector3.new(-349.56, -7, 27.05))

    setTabVisible("STEAL")
end

init()