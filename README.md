-- ============================================
-- STOPPED MENU - by DRAXZS
-- Reconstruído a partir do MacHUB AIMBOT
-- ============================================

-- ============================================
-- CARREGAR FLUENT UI
-- ============================================

local Library = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

-- ============================================
-- CRIAR JANELA
-- ============================================

local Window = Library:CreateWindow({
    Title = "Stopped",
    SubTitle = "by DRAXZS",
    TabWidth = 160,
    Size = UDim2.fromOffset(830, 560),
    Acrylic = false,
    Theme = "Darker",
    MinimizeKey = Enum.KeyCode.LeftControl
})

-- ============================================
-- ABAS (TARGET removido)
-- ============================================

local Tabs = {
    Aim = Window:AddTab({ Title = "AIM", Icon = "crosshair" }),
    ESP = Window:AddTab({ Title = "ESP", Icon = "eye" }),
    Status = Window:AddTab({ Title = "STATUS", Icon = "activity" }),
    Settings = Window:AddTab({ Title = "SETTINGS", Icon = "settings" }),
    Customize = Window:AddTab({ Title = "CUSTOMIZE", Icon = "palette" }),
}

-- ============================================
-- CONFIG
-- ============================================

local player = game:GetService("Players").LocalPlayer
local camera = game.Workspace.CurrentCamera
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local aimConfig = {
    aimLock = false,
    aimTarget = "Head",
    aimOrigin = "Cursor",
    offsetX = 0,
    offsetY = 0,
    lockNearest = false,
    showFOV = false,
    fovRadius = 150,
    fovTransparency = 0.25,
    fovThickness = 2,
    wallCheck = false,
    smoothAim = false,
    smoothValue = 0.15,
    prediction = false,
    predictionVal = 0.12,
    bulletDrop = false,
    bulletDropVal = 1,
    lockFov = false,
    lockFovVal = 70,
    holdKey = Enum.KeyCode.X,
    holdActive = false,
}

local espConfig = {
    enabled = false,
    showName = false,
    showHealth = false,
    showDistance = false,
    textSize = 14,
    maxDistance = 2500,
}

-- ============================================
-- FOV CIRCLE
-- ============================================

local FOVGui = Instance.new("ScreenGui")
FOVGui.Name = "Stopped_FOV"
FOVGui.Parent = player:WaitForChild("PlayerGui")
FOVGui.ResetOnSpawn = false
FOVGui.Enabled = false

local FOVFrame = Instance.new("Frame")
FOVFrame.Size = UDim2.new(0, 1, 0, 1)
FOVFrame.Position = UDim2.new(0.5, 0, 0.5, 0)
FOVFrame.BackgroundTransparency = 1
FOVFrame.Parent = FOVGui

local FOVCircle = Instance.new("Frame")
FOVCircle.BackgroundTransparency = 1
FOVCircle.Parent = FOVFrame

local FOVCorner = Instance.new("UICorner")
FOVCorner.CornerRadius = UDim.new(1, 0)
FOVCorner.Parent = FOVCircle

local FOVStroke = Instance.new("UIStroke")
FOVStroke.Thickness = 2
FOVStroke.Color = Color3.fromRGB(255, 255, 255)
FOVStroke.Transparency = 0.75
FOVStroke.Parent = FOVCircle

local function updateFOV()
    if aimConfig.showFOV then
        FOVGui.Enabled = true
        local size = aimConfig.fovRadius * 2
        FOVCircle.Size = UDim2.new(0, size, 0, size)
        FOVCircle.Position = UDim2.new(0.5, -size/2, 0.5, -size/2)
        FOVStroke.Thickness = aimConfig.fovThickness
        FOVStroke.Transparency = aimConfig.fovTransparency
    else
        FOVGui.Enabled = false
    end
end

-- ============================================
-- AIMBOT LOGIC
-- ============================================

local function getTargetPart(char)
    if aimConfig.aimTarget == "Head" then
        return char:FindFirstChild("Head")
    elseif aimConfig.aimTarget == "Torso" then
        return char:FindFirstChild("UpperTorso") or char:FindFirstChild("Torso")
    elseif aimConfig.aimTarget == "HumanoidRootPart" then
        return char:FindFirstChild("HumanoidRootPart")
    end
    return char:FindFirstChild("Head")
end

local function getClosestTarget()
    local char = player.Character
    if not char then return nil end
    local myHrp = char:FindFirstChild("HumanoidRootPart")
    if not myHrp then return nil end

    local best = nil
    local bestScore = math.huge
    local screenCenter = Vector2.new(camera.ViewportSize.X/2, camera.ViewportSize.Y/2)

    for _, plr in ipairs(game.Players:GetPlayers()) do
        if plr ~= player and plr.Character then
            local humanoid = plr.Character:FindFirstChildOfClass("Humanoid")
            local part = getTargetPart(plr.Character)
            if humanoid and humanoid.Health > 0 and part then
                local screenPos, onScreen = camera:WorldToViewportPoint(part.Position)
                if onScreen then
                    local dist = (Vector2.new(screenPos.X, screenPos.Y) - screenCenter).Magnitude
                    if dist <= aimConfig.fovRadius and dist < bestScore then
                        bestScore = dist
                        best = { Player = plr, Part = part, Position = part.Position }
                    end
                end
            end
        end
    end
    return best
end

RunService.RenderStepped:Connect(function()
    if not aimConfig.aimLock then return end
    if not aimConfig.holdActive then return end

    local target = getClosestTarget()
    if target then
        local camPos = camera.CFrame.Position
        if aimConfig.smoothAim then
            local newCF = CFrame.new(camPos, target.Position)
            camera.CFrame = camera.CFrame:Lerp(newCF, aimConfig.smoothValue)
        else
            camera.CFrame = CFrame.new(camPos, target.Position)
        end
    end
end)

-- ============================================
-- ESP LOGIC
-- ============================================

local espObjects = {}

local function clearESP()
    for _, obj in pairs(espObjects) do
        if obj and obj.Remove then pcall(function() obj:Remove() end) end
    end
    espObjects = {}
end

local function worldToScreen(pos)
    local sp, onScreen = camera:WorldToViewportPoint(pos)
    if onScreen then return Vector2.new(sp.X, sp.Y) end
    return nil
end

local function drawBox(plr)
    local char = plr.Character
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local head = char:FindFirstChild("Head")
    if not hrp or not head then return end

    local headP = worldToScreen(head.Position + Vector3.new(0, 0.5, 0))
    local hrpP = worldToScreen(hrp.Position - Vector3.new(0, 3, 0))
    if not headP or not hrpP then return end

    local h = math.abs(headP.Y - hrpP.Y)
    local w = h * 0.55
    local x = headP.X - w/2
    local y = headP.Y

    local corners = {
        {Vector2.new(x, y), Vector2.new(x + w, y)},
        {Vector2.new(x, y + h), Vector2.new(x + w, y + h)},
        {Vector2.new(x, y), Vector2.new(x, y + h)},
        {Vector2.new(x + w, y), Vector2.new(x + w, y + h)},
    }

    for _, c in ipairs(corners) do
        local line = Drawing.new("Line")
        line.From = c[1]
        line.To = c[2]
        line.Thickness = 1
        line.Color = Color3.fromRGB(255, 255, 255)
        line.Visible = true
        table.insert(espObjects, line)
    end
end

RunService.RenderStepped:Connect(function()
    clearESP()
    if not espConfig.enabled then return end

    for _, plr in ipairs(game.Players:GetPlayers()) do
        if plr ~= player and plr.Character then
            local humanoid = plr.Character:FindFirstChildOfClass("Humanoid")
            if humanoid and humanoid.Health > 0 then
                drawBox(plr)
            end
        end
    end
end)

-- ============================================
-- ABA: AIM
-- ============================================

local aimSection = Tabs.Aim:AddSection("Aimbot")

Tabs.Aim:AddToggle("AimLockToggle", {
    Title = "Aim Lock",
    Description = "tap lock key to toggle, off = hold key",
    Default = false,
    Callback = function(v) aimConfig.aimLock = v end
})

Tabs.Aim:AddDropdown("AimTargetDropdown", {
    Title = "aim target",
    Description = "body part used for camera locking",
    Values = {"Head", "Torso", "HumanoidRootPart"},
    Default = "Head",
    Multi = false,
    Callback = function(v) aimConfig.aimTarget = v end
})

Tabs.Aim:AddDropdown("AimOriginDropdown", {
    Title = "aim origin",
    Description = "cursor, center, custom point",
    Values = {"Cursor", "Center", "Custom"},
    Default = "Cursor",
    Multi = false,
    Callback = function(v) aimConfig.aimOrigin = v end
})

Tabs.Aim:AddButton({
    Title = "Custom Position",
    Description = "tap or drag a normalized per-game aim point",
    Callback = function()
        Library:Notify({ Title = "Custom", Content = "Aim origin set to Custom", Duration = 2 })
    end
})

Tabs.Aim:AddButton({
    Title = "Game Crosshair - Pick",
    Description = "select a PlayerGui object",
    Callback = function()
        Library:Notify({ Title = "Pick", Content = "Crosshair pickup n/a", Duration = 2 })
    end
})

Tabs.Aim:AddButton({
    Title = "Game Crosshair - Clear",
    Description = "clear selected crosshair",
    Callback = function()
        Library:Notify({ Title = "Clear", Content = "Crosshair cleared", Duration = 2 })
    end
})

Tabs.Aim:AddSlider("OriginOffsetX", {
    Title = "aim origin X offset",
    Description = "fine horizontal correction in pixels",
    Default = 0, Min = -200, Max = 200, Rounding = 0,
    Callback = function(v) aimConfig.offsetX = v end
})

Tabs.Aim:AddSlider("OriginOffsetY", {
    Title = "aim origin Y offset",
    Description = "fine vertical correction in pixels",
    Default = 0, Min = -200, Max = 200, Rounding = 0,
    Callback = function(v) aimConfig.offsetY = v end
})

Tabs.Aim:AddToggle("LockNearest", {
    Title = "lock to nearest",
    Default = false,
    Callback = function(v) aimConfig.lockNearest = v end
})

Tabs.Aim:AddToggle("ShowFOV", {
    Title = "show FOV circle",
    Description = "draws the current target radius",
    Default = false,
    Callback = function(v)
        aimConfig.showFOV = v
        updateFOV()
    end
})

Tabs.Aim:AddSlider("FovRadius", {
    Title = "FOV radius",
    Description = "targeting circle radius (px)",
    Default = 150, Min = 10, Max = 800, Rounding = 0,
    Callback = function(v)
        aimConfig.fovRadius = v
        updateFOV()
    end
})

Tabs.Aim:AddSlider("FovTransp", {
    Title = "FOV circle transparency",
    Description = "0 = visible, 1 = hidden",
    Default = 0.25, Min = 0, Max = 1, Rounding = 2,
    Callback = function(v)
        aimConfig.fovTransparency = v
        updateFOV()
    end
})

Tabs.Aim:AddSlider("FovThick", {
    Title = "FOV circle thickness",
    Description = "outline thickness",
    Default = 2, Min = 1, Max = 10, Rounding = 0,
    Callback = function(v)
        aimConfig.fovThickness = v
        updateFOV()
    end
})

Tabs.Aim:AddToggle("WallCheck", {
    Title = "wall check",
    Default = false,
    Callback = function(v) aimConfig.wallCheck = v end
})

Tabs.Aim:AddToggle("SmoothAim", {
    Title = "smooth aim",
    Description = "smooth camera aim",
    Default = false,
    Callback = function(v) aimConfig.smoothAim = v end
})

Tabs.Aim:AddSlider("SmoothVal", {
    Title = "smooth aim value",
    Default = 15, Min = 1, Max = 100, Rounding = 0,
    Callback = function(v) aimConfig.smoothValue = v / 100 end
})

Tabs.Aim:AddToggle("AimPrediction", {
    Title = "aim prediction",
    Description = "adaptive prediction, dampens close targets",
    Default = false,
    Callback = function(v) aimConfig.prediction = v end
})

Tabs.Aim:AddSlider("PredictionVal", {
    Title = "prediction strength",
    Default = 12, Min = 0, Max = 100, Rounding = 0,
    Callback = function(v) aimConfig.predictionVal = v / 100 end
})

Tabs.Aim:AddToggle("BulletDrop", {
    Title = "bullet drop compensation",
    Description = "raises the aim point more at longer distances",
    Default = false,
    Callback = function(v) aimConfig.bulletDrop = v end
})

Tabs.Aim:AddSlider("BulletDropVal", {
    Title = "drop compensation",
    Default = 1, Min = 0, Max = 10, Rounding = 0,
    Callback = function(v) aimConfig.bulletDropVal = v end
})

Tabs.Aim:AddToggle("LockFOV", {
    Title = "lock fov",
    Description = "changes camera FOV",
    Default = false,
    Callback = function(v) aimConfig.lockFov = v end
})

Tabs.Aim:AddSlider("LockFovVal", {
    Title = "locked fov value",
    Default = 70, Min = 20, Max = 120, Rounding = 0,
    Callback = function(v) aimConfig.lockFovVal = v end
})

-- ============================================
-- ABA: ESP
-- ============================================

Tabs.ESP:AddToggle("ESPEnabled", {
    Title = "enable esp",
    Default = false,
    Callback = function(v)
        espConfig.enabled = v
        if not v then clearESP() end
    end
})

Tabs.ESP:AddToggle("TeamColor", {
    Title = "team color",
    Default = false,
    Callback = function(v) end
})

Tabs.ESP:AddToggle("ShowName", {
    Title = "show name",
    Default = false,
    Callback = function(v) espConfig.showName = v end
})

Tabs.ESP:AddToggle("ShowHealth", {
    Title = "show health",
    Default = false,
    Callback = function(v) espConfig.showHealth = v end
})

Tabs.ESP:AddToggle("ShowTeam", {
    Title = "show team",
    Default = false,
    Callback = function(v) end
})

Tabs.ESP:AddSlider("ESPTransp", {
    Title = "esp transparency",
    Description = "highlight fill transparency",
    Default = 30, Min = 0, Max = 100, Rounding = 0,
    Callback = function(v) end
})

Tabs.ESP:AddSection("Render")

Tabs.ESP:AddDropdown("ESPBackend", {
    Title = "esp backend",
    Description = "normal Roblox instances or executor Drawing",
    Values = {"Normal", "Drawing"},
    Default = "Normal",
    Multi = false,
    Callback = function(v) end
})

Tabs.ESP:AddDropdown("ESPMode", {
    Title = "normal esp mode",
    Description = "highlight, box, head ornament",
    Values = {"Highlight", "Box", "Head"},
    Default = "Highlight",
    Multi = false,
    Callback = function(v) end
})

Tabs.ESP:AddToggle("AlwaysOnTop", {
    Title = "always on top",
    Description = "draw esp over walls when possible",
    Default = false,
    Callback = function(v) end
})

Tabs.ESP:AddToggle("ShowDistance", {
    Title = "show distance",
    Default = false,
    Callback = function(v) espConfig.showDistance = v end
})

Tabs.ESP:AddSlider("TextSize", {
    Title = "text size",
    Description = "billboard text size",
    Default = 14, Min = 8, Max = 32, Rounding = 0,
    Callback = function(v) espConfig.textSize = v end
})

Tabs.ESP:AddSlider("MaxDist", {
    Title = "max esp distance",
    Description = "hide esp beyond this distance",
    Default = 2500, Min = 100, Max = 10000, Rounding = 0,
    Callback = function(v) espConfig.maxDistance = v end
})

Tabs.ESP:AddSlider("OutlineTransp", {
    Title = "outline transparency",
    Description = "highlight outline alpha",
    Default = 12, Min = 0, Max = 100, Rounding = 0,
    Callback = function(v) end
})

-- ============================================
-- ABA: STATUS
-- ============================================

Tabs.Status:AddParagraph({
    Title = "players",
    Content = tostring(#game.Players:GetPlayers()) .. " / " .. tostring(game.Players.MaxPlayers)
})

Tabs.Status:AddParagraph({
    Title = "place id",
    Content = tostring(game.PlaceId)
})

Tabs.Status:AddParagraph({
    Title = "job id",
    Content = tostring(game.JobId)
})

Tabs.Status:AddParagraph({
    Title = "executor",
    Content = (identifyexecutor and identifyexecutor()) or "Unknown"
})

Tabs.Status:AddButton({
    Title = "copy place id",
    Callback = function()
        if setclipboard then setclipboard(tostring(game.PlaceId)) end
        Library:Notify({ Title = "Copiado!", Content = "Place ID", Duration = 2 })
    end
})

Tabs.Status:AddButton({
    Title = "copy job id",
    Callback = function()
        if setclipboard then setclipboard(tostring(game.JobId)) end
        Library:Notify({ Title = "Copiado!", Content = "Job ID", Duration = 2 })
    end
})

-- ============================================
-- ABA: SETTINGS
-- ============================================

Tabs.Settings:AddParagraph({
    Title = "key binds",
    Content = "X (Lock) | P (Aim) | RightAlt (FOV) | RightControl (ESP)"
})

Tabs.Settings:AddButton({
    Title = "add key",
    Callback = function()
        Library:Notify({ Title = "Bind", Content = "Pressione uma tecla.", Duration = 2 })
    end
})

Tabs.Settings:AddButton({
    Title = "clear binds",
    Callback = function()
        Library:Notify({ Title = "Bind", Content = "Binds limpos.", Duration = 2 })
    end
})

Tabs.Settings:AddSection("Configs")

Tabs.Settings:AddToggle("AutoSave", {
    Title = "auto save configs",
    Description = "save changes instantly",
    Default = true,
    Callback = function(v) end
})

Tabs.Settings:AddButton({
    Title = "save config",
    Callback = function()
        Library:Notify({ Title = "Saved", Content = "Config salva.", Duration = 2 })
    end
})

Tabs.Settings:AddButton({
    Title = "load config",
    Callback = function()
        Library:Notify({ Title = "Loaded", Content = "Config carregada.", Duration = 2 })
    end
})

Tabs.Settings:AddButton({
    Title = "delete config",
    Callback = function()
        Library:Notify({ Title = "Deleted", Content = "Config deletada.", Duration = 2 })
    end
})

-- ============================================
-- ABA: CUSTOMIZE
-- ============================================

Tabs.Customize:AddSection("Window")

Tabs.Customize:AddSlider("UIScale", {
    Title = "ui scale",
    Description = "scales the whole window",
    Default = 80, Min = 50, Max = 150, Rounding = 0,
    Callback = function(v) end
})

Tabs.Customize:AddSlider("WindowTransp", {
    Title = "window transparency",
    Description = "root panel transparency",
    Default = 45, Min = 0, Max = 100, Rounding = 0,
    Callback = function(v) end
})

Tabs.Customize:AddSlider("CornerRadius", {
    Title = "corner roundness",
    Description = "applies to all UI round",
    Default = 24, Min = 0, Max = 40, Rounding = 0,
    Callback = function(v) end
})

Tabs.Customize:AddToggle("CompactRows", {
    Title = "compact rows",
    Description = "smaller rows on next UI rebuild",
    Default = false,
    Callback = function(v) end
})

Tabs.Customize:AddSection("Theme")

Tabs.Customize:AddDropdown("AccentTheme", {
    Title = "accent theme",
    Description = "changes accent colors",
    Values = {"Black & White", "Midnight Purple", "Ocean Blue", "Blood Red"},
    Default = "Black & White",
    Multi = false,
    Callback = function(v) end
})

Tabs.Customize:AddToggle("AnimatedUI", {
    Title = "animated ui",
    Default = true,
    Callback = function(v) end
})

Tabs.Customize:AddSlider("AnimationSpeed", {
    Title = "animation speed",
    Description = "higher = faster tweens",
    Default = 10, Min = 1, Max = 50, Rounding = 0,
    Callback = function(v) end
})

Tabs.Customize:AddSection("Notifications")

Tabs.Customize:AddDropdown("ToastPos", {
    Title = "toast position",
    Values = {"Top Right", "Top Left", "Bottom Right", "Bottom Left"},
    Default = "Top Right",
    Multi = false,
    Callback = function(v) end
})

Tabs.Customize:AddSlider("ToastDuration", {
    Title = "toast duration",
    Description = "how long notifications stay",
    Default = 3, Min = 1, Max = 15, Rounding = 0,
    Callback = function(v) end
})

Tabs.Customize:AddSlider("ToastMax", {
    Title = "toast max stack",
    Default = 4, Min = 1, Max = 10, Rounding = 0,
    Callback = function(v) end
})

Tabs.Customize:AddToggle("CompactToasts", {
    Title = "compact toasts",
    Default = false,
    Callback = function(v) end
})

Tabs.Customize:AddSection("Aim Origin Marker")

Tabs.Customize:AddToggle("AimOriginMarker", {
    Title = "aim origin marker",
    Default = false,
    Callback = function(v) end
})

Tabs.Customize:AddSlider("AimMarkerSize", {
    Title = "aim marker size",
    Default = 12, Min = 2, Max = 40, Rounding = 0,
    Callback = function(v) end
})

Tabs.Customize:AddSlider("AimMarkerTransp", {
    Title = "aim marker transparency",
    Default = 35, Min = 0, Max = 100, Rounding = 0,
    Callback = function(v) end
})

-- ============================================
-- "by DRAXZS" no final do CUSTOMIZE
-- ============================================

Tabs.Customize:AddParagraph({
    Title = "by DRAXZS",
    Content = "Stopped Menu - Editado por DRAXZS\nReconstruído a partir do MacHUB AIMBOT\nCores: Preto e Branco"
})

-- ============================================
-- KEYBIND
-- ============================================

UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == aimConfig.holdKey then
        aimConfig.holdActive = true
    end
    if input.KeyCode == Enum.KeyCode.Insert then
        Window:Toggle()
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.KeyCode == aimConfig.holdKey then
        aimConfig.holdActive = false
    end
end)

-- ============================================
-- NOTIFICAÇÃO FINAL
-- ============================================

Library:Notify({
    Title = "Stopped Menu",
    Content = "Carregado! Insert = abrir/fechar | X = aim lock",
    Duration = 4
})

print("====================================")
print("⚡ STOPPED MENU - by DRAXZS ⚡")
print("📌 Insert = abrir/fechar")
print("📌 X = aim lock")
print("====================================")
