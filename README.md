
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- 获取对Workspace和RunService的引用
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

-- 创建窗口
local Window = Rayfield:CreateWindow({
    Name = "TDR透视",
    Icon = 0,
    LoadingTitle = "加载中...",
    LoadingSubtitle = "高亮功能初始化",
    Theme = "Default",
    ToggleUIKeybind = "K",
    ConfigurationSaving = {
        Enabled = false,
    }
})

-- 创建主标签页
local MainTab = Window:CreateTab("高亮控制", nil)
local FixedItemsTab = Window:CreateTab("固定物品ESP", nil)
local CreatureTab = Window:CreateTab("生物ESP", nil) -- 新增的生物ESP标签页

-- ========== 高亮控制标签页 ==========
local MainSection = MainTab:CreateSection("高亮设置")

-- 高亮系统变量
local highlightSystem = {
    Enabled = false,
    TargetName = "你的组件名称",
    HighlightColor = Color3.new(1, 0, 0),
    ProcessedObjects = {}
}

-- 应用高亮效果到目标对象
local function applyHighlight(target)
    if highlightSystem.ProcessedObjects[target] then
        return
    end
    
    highlightSystem.ProcessedObjects[target] = true
    
    local existingHighlight = target:FindFirstChildOfClass("Highlight")
    if existingHighlight then
        existingHighlight:Destroy()
    end
    
    local highlight = Instance.new("Highlight")
    highlight.Name = "DynamicHighlight"
    highlight.FillColor = highlightSystem.HighlightColor
    highlight.OutlineColor = highlightSystem.HighlightColor
    highlight.FillTransparency = 0.5
    highlight.OutlineTransparency = 0
    highlight.Parent = target
    
    target.Destroying:Connect(function()
        highlightSystem.ProcessedObjects[target] = nil
    end)
end

-- 检查对象是否符合高亮条件
local function shouldHighlight(object)
    if string.find(object.Name:lower(), highlightSystem.TargetName:lower()) then
        if object:IsA("Model") or object:IsA("BasePart") then
            return true
        end
    end
    return false
end

-- 初始扫描
local function initialScan()
    for _, child in ipairs(workspace:GetChildren()) do
        if shouldHighlight(child) then
            applyHighlight(child)
        end
        
        if child:IsA("Model") then
            for _, descendant in ipairs(child:GetDescendants()) do
                if shouldHighlight(descendant) then
                    applyHighlight(descendant)
                end
            end
        end
    end
end

-- 设置监听器
local function setupListeners()
    -- 监听新对象的添加
    workspace.ChildAdded:Connect(function(child)
        wait(0.1)
        
        if shouldHighlight(child) then
            applyHighlight(child)
        end
        
        if child:IsA("Model") then
            for _, descendant in ipairs(child:GetDescendants()) do
                if shouldHighlight(descendant) then
                    applyHighlight(descendant)
                end
            end
            
            child.DescendantAdded:Connect(function(descendant)
                if shouldHighlight(descendant) then
                    applyHighlight(descendant)
                end
            end)
        end
    end)
end

-- 清除所有高亮效果
local function clearAllHighlights()
    for object, _ in pairs(highlightSystem.ProcessedObjects) do
        local highlight = object:FindFirstChildOfClass("Highlight")
        if highlight then
            highlight:Destroy()
        end
    end
    highlightSystem.ProcessedObjects = {}
end

-- 启动高亮系统
local function startHighlightSystem()
    if highlightSystem.Enabled then return end
    
    highlightSystem.Enabled = true
    initialScan()
    setupListeners()
    Rayfield:Notify({
        Title = "高亮系统",
        Content = "高亮功能已开启",
        Duration = 3,
    })
end

-- 停止高亮系统
local function stopHighlightSystem()
    if not highlightSystem.Enabled then return end
    
    highlightSystem.Enabled = false
    clearAllHighlights()
    Rayfield:Notify({
        Title = "高亮系统",
        Content = "高亮功能已关闭",
        Duration = 3,
    })
end

-- ========== AlmondWater高亮系统函数 ==========

-- AlmondWater高亮系统变量
local almondWaterHighlightSystem = {
    Enabled = false,
    TargetModelName = "AlmondWater",
    HighlightColor = Color3.new(1, 0, 0), -- 红色
    ProcessedModels = {}
}

-- AlmondWater传送系统变量
local almondWaterTeleportSystem = {
    Enabled = false,
    TeleportInterval = 2, -- 传送间隔（秒）
    TeleportThread = nil
}

-- 应用高亮效果到AlmondWater模型
local function applyHighlightToAlmondWater(model)
    if almondWaterHighlightSystem.ProcessedModels[model] then
        return
    end
    
    almondWaterHighlightSystem.ProcessedModels[model] = true
    
    local existingHighlight = model:FindFirstChildOfClass("Highlight")
    if existingHighlight then
        existingHighlight:Destroy()
    end
    
    local highlight = Instance.new("Highlight")
    highlight.Name = "AlmondWaterHighlight"
    highlight.FillColor = almondWaterHighlightSystem.HighlightColor
    highlight.OutlineColor = almondWaterHighlightSystem.HighlightColor
    highlight.FillTransparency = 0.5
    highlight.OutlineTransparency = 0
    highlight.Parent = model
    
    model.Destroying:Connect(function()
        almondWaterHighlightSystem.ProcessedModels[model] = nil
    end)
end

-- 检查并处理AlmondWater模型
local function processAlmondWaterModel(model)
    if model.Name == almondWaterHighlightSystem.TargetModelName and model:IsA("Model") then
        applyHighlightToAlmondWater(model)
        return true
    end
    return false
end

-- AlmondWater初始扫描
local function almondWaterInitialScan()
    for _, descendant in pairs(Workspace:GetDescendants()) do
        processAlmondWaterModel(descendant)
    end
end

-- 设置AlmondWater监听器
local function setupAlmondWaterListeners()
    -- 监听新加入的实例
    Workspace.DescendantAdded:Connect(function(descendant)
        RunService.Heartbeat:Wait() -- 延迟一帧处理
        if not almondWaterHighlightSystem.ProcessedModels[descendant] then
            processAlmondWaterModel(descendant)
        end
    end)
    
    -- 监听模型名称变化
    Workspace.DescendantAdded:Connect(function(descendant)
        if descendant:IsA("Model") then
            descendant:GetPropertyChangedSignal("Name"):Connect(function()
                if descendant.Name == almondWaterHighlightSystem.TargetModelName then
                    applyHighlightToAlmondWater(descendant)
                end
            end)
        end
    end)
end

-- 清除所有AlmondWater高亮效果
local function clearAlmondWaterHighlights()
    for model, _ in pairs(almondWaterHighlightSystem.ProcessedModels) do
        local highlight = model:FindFirstChildOfClass("Highlight")
        if highlight then
            highlight:Destroy()
        end
    end
    almondWaterHighlightSystem.ProcessedModels = {}
end

-- 启动AlmondWater高亮系统
local function startAlmondWaterHighlightSystem()
    if almondWaterHighlightSystem.Enabled then return end
    
    almondWaterHighlightSystem.Enabled = true
    almondWaterInitialScan()
    setupAlmondWaterListeners()
    Rayfield:Notify({
        Title = "AlmondWater高亮系统",
        Content = "AlmondWater高亮功能已开启",
        Duration = 3,
    })
end

-- 停止AlmondWater高亮系统
local function stopAlmondWaterHighlightSystem()
    if not almondWaterHighlightSystem.Enabled then return end
    
    almondWaterHighlightSystem.Enabled = false
    clearAlmondWaterHighlights()
    Rayfield:Notify({
        Title = "AlmondWater高亮系统",
        Content = "AlmondWater高亮功能已关闭",
        Duration = 3,
    })
end

-- ========== AlmondWater传送系统函数 ==========

-- 查找最近的AlmondWater模型
local function findNearestAlmondWater()
    local player = Players.LocalPlayer
    if not player or not player.Character then
        return nil
    end
    
    local character = player.Character
    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
    if not humanoidRootPart then
        return nil
    end
    
    local nearestAlmondWater = nil
    local nearestDistance = math.huge
    
    for model, _ in pairs(almondWaterHighlightSystem.ProcessedModels) do
        if model:IsA("Model") and model.PrimaryPart then
            local distance = (humanoidRootPart.Position - model.PrimaryPart.Position).Magnitude
            if distance < nearestDistance then
                nearestDistance = distance
                nearestAlmondWater = model
            end
        end
    end
    
    return nearestAlmondWater
end

-- 传送到AlmondWater
local function teleportToAlmondWater()
    local player = Players.LocalPlayer
    if not player or not player.Character then
        return false
    end
    
    local character = player.Character
    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
    if not humanoidRootPart then
        return false
    end
    
    local almondWater = findNearestAlmondWater()
    if not almondWater or not almondWater.PrimaryPart then
        return false
    end
    
    -- 传送到AlmondWater位置
    humanoidRootPart.CFrame = almondWater.PrimaryPart.CFrame + Vector3.new(0, 5, 0)
    return true
end

-- 启动AlmondWater传送系统
local function startAlmondWaterTeleportSystem()
    if almondWaterTeleportSystem.Enabled then return end
    
    almondWaterTeleportSystem.Enabled = true
    
    -- 创建传送循环
    almondWaterTeleportSystem.TeleportThread = task.spawn(function()
        while almondWaterTeleportSystem.Enabled do
            local success = teleportToAlmondWater()
            
            if success then
                Rayfield:Notify({
                    Title = "AlmondWater传送",
                    Content = "已传送到最近的AlmondWater",
                    Duration = 1,
                })
            else
                Rayfield:Notify({
                    Title = "AlmondWater传送",
                    Content = "未找到可传送的AlmondWater",
                    Duration = 1,
                })
            end
            
            -- 等待指定的间隔时间
            task.wait(almondWaterTeleportSystem.TeleportInterval)
        end
    end)
    
    Rayfield:Notify({
        Title = "AlmondWater传送系统"
        Content = "AlmondWater传送功能已开启，每" .. almondWaterTeleportSystem.TeleportInterval .. "秒传送一次",
        Duration = 5,
    })
end

-- 停止AlmondWater传送系统
local function stopAlmondWaterTeleportSystem()
    if not almondWaterTeleportSystem.Enabled then return end
    
    almondWaterTeleportSystem.Enabled = false
    
    -- 停止传送循环
    if almondWaterTeleportSystem.TeleportThread then
        task.cancel(almondWaterTeleportSystem.TeleportThread)
        almondWaterTeleportSystem.TeleportThread = nil
    end
    
    Rayfield:Notify({
        Title = "AlmondWater传送系统",
        Content = "AlmondWater传送功能已关闭",
        Duration = 3,
    })
end

-- ========== Skin Stealer高亮系统函数 ==========

-- Skin Stealer高亮系统变量
local skinStealerHighlightSystem = {
    Enabled = false,
    TargetModelName = "Skin Stealer",
    HighlightColor = Color3.new(1, 0, 0), -- 红色
    ProcessedModels = {}
}

-- 应用高亮效果到Skin Stealer模型
local function applyHighlightToSkinStealer(model)
    if skinStealerHighlightSystem.ProcessedModels[model] then
        return
    end
    
    skinStealerHighlightSystem.ProcessedModels[model] = true
    
    local existingHighlight = model:FindFirstChildOfClass("Highlight")
    if existingHighlight then
        existingHighlight:Destroy()
    end
    
    local highlight = Instance.new("Highlight")
    highlight.Name = "SkinStealerHighlight"
    highlight.FillColor = skinStealerHighlightSystem.HighlightColor
    highlight.OutlineColor = skinStealerHighlightSystem.HighlightColor
    highlight.FillTransparency = 0.5
    highlight.OutlineTransparency = 0
    highlight.Parent = model
    
    model.Destroying:Connect(function()
        skinStealerHighlightSystem.ProcessedModels[model] = nil
    end)
end

-- 检查并处理Skin Stealer模型
local function processSkinStealerModel(model)
    if model.Name == skinStealerHighlightSystem.TargetModelName and model:IsA("Model") then
        applyHighlightToSkinStealer(model)
        return true
    end
    return false
end

-- Skin Stealer初始扫描
local function skinStealerInitialScan()
    for _, descendant in pairs(Workspace:GetDescendants()) do
        processSkinStealerModel(descendant)
    end
end

-- 设置Skin Stealer监听器
local function setupSkinStealerListeners()
    -- 监听新加入的实例
    Workspace.DescendantAdded:Connect(function(descendant)
        RunService.Heartbeat:Wait() -- 延迟一帧处理
        if not skinStealerHighlightSystem.ProcessedModels[descendant] then
            processSkinStealerModel(descendant)
        end
    end)
    
    -- 监听模型名称变化
    Workspace.DescendantAdded:Connect(function(descendant)
        if descendant:IsA("Model") then
            descendant:GetPropertyChangedSignal("Name"):Connect(function()
                if descendant.Name == skinStealerHighlightSystem.TargetModelName then
                    applyHighlightToSkinStealer(descendant)
                end
            end)
        end
    end)
end

-- 清除所有Skin Stealer高亮效果
local function clearSkinStealerHighlights()
    for model, _ in pairs(skinStealerHighlightSystem.ProcessedModels) do
        local highlight = model:FindFirstChildOfClass("Highlight")
        if highlight then
            highlight:Destroy()
        end
    end
    skinStealerHighlightSystem.ProcessedModels = {}
end

-- 启动Skin Stealer高亮系统
local function startSkinStealerHighlightSystem()
    if skinStealerHighlightSystem.Enabled then return end
    
    skinStealerHighlightSystem.Enabled = true
    skinStealerInitialScan()
    setupSkinStealerListeners()
    Rayfield:Notify({
        Title = "Skin Stealer高亮系统",
        Content = "Skin Stealer高亮功能已开启",
        Duration = 3,
    })
end

-- 停止Skin Stealer高亮系统
local function stopSkinStealerHighlightSystem()
    if not skinStealerHighlightSystem.Enabled then return end
    
    skinStealerHighlightSystem.Enabled = false
    clearSkinStealerHighlights()
    Rayfield:Notify({
        Title = "Skin Stealer高亮系统",
        Content = "Skin Stealer高亮功能已关闭",
        Duration = 3,
    })
end

-- ========== 高亮控制标签页GUI控件 ==========

-- 创建开关控件
local Toggle = MainTab:CreateToggle({
    Name = "高亮功能开关",
    CurrentValue = false,
    Flag = "HighlightToggle",
    Callback = function(Value)
        if Value then
            startHighlightSystem()
        else
            stopHighlightSystem()
        end
    end,
})

-- 创建目标名称输入框
local Input = MainTab:CreateInput({
    Name = "目标名称",
    PlaceholderText = "输入要高亮的组件名称",
    RemoveTextAfterFocusLost = false,
    Callback = function(Text)
        highlightSystem.TargetName = Text
        if highlightSystem.Enabled then
            -- 重新扫描
            clearAllHighlights()
            initialScan()
        end
    end,
})

-- 创建颜色选择器
local ColorPicker = MainTab:CreateColorPicker({
    Name = "高亮颜色",
    Color = Color3.new(1, 0, 0),
    Flag = "HighlightColor",
    Callback = function(Color)
        highlightSystem.HighlightColor = Color
        if highlightSystem.Enabled then
            -- 更新所有已高亮对象的颜色
            for object, _ in pairs(highlightSystem.ProcessedObjects) do
                local highlight = object:FindFirstChildOfClass("Highlight")
                if highlight then
                    highlight.FillColor = Color
                    highlight.OutlineColor = Color
                end
            end
        end
    end
})

-- 创建按钮部分
local ButtonSection = MainTab:CreateSection("控制按钮")

-- 立即扫描按钮
local ScanButton = MainTab:CreateButton({
    Name = "立即扫描",
    Callback = function()
        if highlightSystem.Enabled then
            clearAllHighlights()
            initialScan()
            Rayfield:Notify({
                Title = "高亮系统",
                Content = "已完成重新扫描",
                Duration = 3,
            })
        else
            Rayfield:Notify({
                Title = "高亮系统",
                Content = "请先开启高亮功能",
                Duration = 3,
            })
        end
    end,
})

-- 清除所有高亮按钮
local ClearButton = MainTab:CreateButton({
    Name = "清除所有高亮",
    Callback = function()
        clearAllHighlights()
        Rayfield:Notify({
            Title = "高亮系统",
            Content = "已清除所有高亮效果",
            Duration = 3,
        })
    end,
})

-- 信息显示部分
local InfoSection = MainTab:CreateSection("系统信息")

-- 状态显示标签
local StatusLabel = MainTab:CreateLabel("当前状态: 高亮功能未开启")
local ObjectsLabel = MainTab:CreateLabel("已高亮对象: 0")

-- ========== 固定物品ESP标签页GUI控件 ==========

-- AlmondWater高亮设置部分
local AlmondWaterSection = FixedItemsTab:CreateSection("AlmondWater高亮设置")

-- 创建AlmondWater高亮开关控件
local AlmondWaterToggle = FixedItemsTab:CreateToggle({
    Name = "AlmondWater高亮开关",
    CurrentValue = false,
    Flag = "AlmondWaterHighlightToggle",
    Callback = function(Value)
        if Value then
            startAlmondWaterHighlightSystem()
        else
            stopAlmondWaterHighlightSystem()
        end
    end,
})

-- 创建AlmondWater高亮颜色选择器
local AlmondWaterColorPicker = FixedItemsTab:CreateColorPicker({
    Name = "AlmondWater高亮颜色",
    Color = Color3.new(1, 0, 0),
    Flag = "AlmondWaterHighlightColor",
    Callback = function(Color)
        almondWaterHighlightSystem.HighlightColor = Color
        if almondWaterHighlightSystem.Enabled then
            -- 更新所有已高亮AlmondWater模型的颜色
            for model, _ in pairs(almondWaterHighlightSystem.ProcessedModels) do
                local highlight = model:FindFirstChildOfClass("Highlight")
                if highlight then
                    highlight.FillColor = Color
                    highlight.OutlineColor = Color
                end
            end
        end
    end
})

-- AlmondWater传送设置部分
local AlmondWaterTeleportSection = FixedItemsTab:CreateSection("AlmondWater传送设置")

-- 创建AlmondWater传送开关控件
local AlmondWaterTeleportToggle = FixedItemsTab:CreateToggle({
    Name = "AlmondWater自动传送",
    CurrentValue = false,
    Flag = "AlmondWaterTeleportToggle",
    Callback = function(Value)
        if Value then
            startAlmondWaterTeleportSystem()
        else
            stopAlmondWaterTeleportSystem()
        end
    end,
})

-- 创建传送间隔输入框
local TeleportIntervalInput = FixedItemsTab:CreateInput({
    Name = "传送间隔(秒)",
    PlaceholderText = "输入传送间隔时间",
    RemoveTextAfterFocusLost = false,
    Callback = function(Text)
        local interval = tonumber(Text)
        if interval and interval > 0 then
            almondWaterTeleportSystem.TeleportInterval = interval
            Rayfield:Notify({
                Title = "AlmondWater传送系统",
                Content = "传送间隔已设置为 " .. interval .. " 秒",
                Duration = 3,
            })
        else
            Rayfield:Notify({
                Title = "AlmondWater传送系统",
                Content = "请输入有效的正数作为间隔时间",
                Duration = 3,
            })
        end
    end,
})

-- 立即传送按钮
local TeleportButton = FixedItemsTab:CreateButton({
    Name = "立即传送到AlmondWater",
    Callback = function()
        local success = teleportToAlmondWater()
        if success then
            Rayfield:Notify({
                Title = "AlmondWater传送",
                Content = "已成功传送到最近的AlmondWater",
                Duration = 3,
            })
        else
            Rayfield:Notify({
                Title = "AlmondWater传送",
                Content = "传送失败，未找到可用的AlmondWater",
                Duration = 3,
            })
        end
    end,
})

-- AlmondWater控制按钮部分
local AlmondWaterButtonSection = FixedItemsTab:CreateSection("AlmondWater控制")

-- 清除AlmondWater高亮按钮
local ClearAlmondWaterButton = FixedItemsTab:CreateButton({
    Name = "清除AlmondWater高亮",
    Callback = function()
        clearAlmondWaterHighlights()
        Rayfield:Notify({
            Title = "AlmondWater高亮系统",
            Content = "已清除所有AlmondWater高亮效果",
            Duration = 3,
        })
    end,
})

-- AlmondWater信息显示部分
local AlmondWaterInfoSection = FixedItemsTab:CreateSection("AlmondWater系统信息")

-- AlmondWater状态显示标签
local AlmondWaterStatusLabel = FixedItemsTab:CreateLabel("AlmondWater状态: 未开启")
local AlmondWaterTeleportStatusLabel = FixedItemsTab:CreateLabel("传送状态: 未开启")
local AlmondWaterObjectsLabel = FixedItemsTab:CreateLabel("已高亮AlmondWater: 0")

-- ========== 生物ESP标签页GUI控件 ==========

-- Skin Stealer高亮设置部分
local SkinStealerSection = CreatureTab:CreateSection("Skin Stealer高亮设置")

-- 创建Skin Stealer高亮开关控件
local SkinStealerToggle = CreatureTab:CreateToggle({
    Name = "Skin Stealer高亮开关",
    CurrentValue = false,
    Flag = "SkinStealerHighlightToggle",
    Callback = function(Value)
        if Value then
            startSkinStealerHighlightSystem()
        else
            stopSkinStealerHighlightSystem()
        end
    end,
})

-- 创建Skin Stealer高亮颜色选择器
local SkinStealerColorPicker = CreatureTab:CreateColorPicker({
    Name = "Skin Stealer高亮颜色",
    Color = Color3.new(1, 0, 0),
    Flag = "SkinStealerHighlightColor",
    Callback = function(Color)
        skinStealerHighlightSystem.HighlightColor = Color
        if skinStealerHighlightSystem.Enabled then
            -- 更新所有已高亮Skin Stealer模型的颜色
            for model, _ in pairs(skinStealerHighlightSystem.ProcessedModels) do
                local highlight = model:FindFirstChildOfClass("Highlight")
                if highlight then
                    highlight.FillColor = Color
                    highlight.OutlineColor = Color
                end
            end
        end
    end
})

-- Skin Stealer控制按钮部分
local SkinStealerButtonSection = CreatureTab:CreateSection("Skin Stealer控制")

-- 清除Skin Stealer高亮按钮
local ClearSkinStealerButton = CreatureTab:CreateButton({
    Name = "清除Skin Stealer高亮",
    Callback = function()
        clearSkinStealerHighlights()
        Rayfield:Notify({
            Title = "Skin Stealer高亮系统",
            Content = "已清除所有Skin Stealer高亮效果",
            Duration = 3,
        })
    end,
})

-- Skin Stealer信息显示部分
local SkinStealerInfoSection = CreatureTab:CreateSection("Skin Stealer系统信息")

-- Skin Stealer状态显示标签
local SkinStealerStatusLabel = CreatureTab:CreateLabel("Skin Stealer状态: 未开启")
local SkinStealerObjectsLabel = CreatureTab:CreateLabel("已高亮Skin Stealer: 0")

-- ========== 状态更新函数 ==========

-- 更新状态显示的函数
local function updateStatusDisplay()
    -- 更新高亮控制标签页的状态
    StatusLabel:Set("当前状态: " .. (highlightSystem.Enabled and "高亮功能已开启" or "高亮功能未开启"))
    
    local count = 0
    for _ in pairs(highlightSystem.ProcessedObjects) do
        count = count + 1
    end
    ObjectsLabel:Set("已高亮对象: " .. count)
    
    -- 更新固定物品ESP标签页的状态
    AlmondWaterStatusLabel:Set("AlmondWater状态: " .. (almondWaterHighlightSystem.Enabled and "已开启" or "未开启"))
    AlmondWaterTeleportStatusLabel:Set("传送状态: " .. (almondWaterTeleportSystem.Enabled and "已开启" or "未开启"))
    
    local almondWaterCount = 0
    for _ in pairs(almondWaterHighlightSystem.ProcessedModels) do
        almondWaterCount = almondWaterCount + 1
    end
    AlmondWaterObjectsLabel:Set("已高亮AlmondWater: " .. almondWaterCount)
    
    -- 更新生物ESP标签页的状态
    SkinStealerStatusLabel:Set("Skin Stealer状态: " .. (skinStealerHighlightSystem.Enabled and "已开启" or "未开启"))
    
    local skinStealerCount = 0
    for _ in pairs(skinStealerHighlightSystem.ProcessedModels) do
        skinStealerCount = skinStealerCount + 1
    end
    SkinStealerObjectsLabel:Set("已高亮Skin Stealer: " .. skinStealerCount)
end

-- 定期更新状态显示
spawn(function()
    while true do
        updateStatusDisplay()
        wait(1)
    end
end)

Rayfield:Notify({
    Title = "高亮控制系统",
    Content = "系统加载完成！使用K键切换界面",
    Duration = 6,
})

