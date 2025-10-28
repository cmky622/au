local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- 获取对Workspace和RunService的引用
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")

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
local FixedItemsTab = Window:CreateTab("固定物品ESP", nil) -- 新增的固定物品ESP标签页

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
local AlmondWaterObjectsLabel = FixedItemsTab:CreateLabel("已高亮AlmondWater: 0")

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
    
    local almondWaterCount = 0
    for _ in pairs(almondWaterHighlightSystem.ProcessedModels) do
        almondWaterCount = almondWaterCount + 1
    end
    AlmondWaterObjectsLabel:Set("已高亮AlmondWater: " .. almondWaterCount)
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
