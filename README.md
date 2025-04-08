-- Version: 4.8 (Dynamic Updates + Persistent Toggles + Real-Time Control)
local activeClickDetectors = {} -- Tracks toggle states
local buttonMap = {} -- Maps buttons to their corresponding ClickDetectors
local lastFireTime = {} -- Stores last fire time for each ClickDetector
local debounceTime = 0.5 -- Minimum time between firing ClickDetectors

-- Function to create the GUI
local function createGUI()
    local UI = game.Players.LocalPlayer.PlayerGui:FindFirstChild("UI")

    -- Destroy existing GUI if it already exists
    if UI then
        UI:Destroy()
    end

    -- UI setup
    UI = Instance.new("ScreenGui")
    local Main = Instance.new("Frame")
    local ScrollingFrame = Instance.new("ScrollingFrame")
    local UIListLayout = Instance.new("UIListLayout")
    local Title = Instance.new("TextLabel")

    UI.Name = "UI"
    UI.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

    Main.Name = "Main"
    Main.Parent = UI
    Main.BackgroundColor3 = Color3.fromRGB(48, 48, 48)
    Main.Size = UDim2.new(0, 250, 0, 400)
    Main.Position = UDim2.new(0.3, 0, 0.3, 0)
    Main.Active = true
    Main.Draggable = true
    Main.BorderSizePixel = 2

    Title.Name = "Title"
    Title.Parent = Main
    Title.Size = UDim2.new(1, 0, 0, 30)
    Title.Text = "ClickDetector Manager"
    Title.TextColor3 = Color3.fromRGB(255, 255, 255)
    Title.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    Title.Font = Enum.Font.SourceSansBold
    Title.TextSize = 16
    Title.BorderSizePixel = 1

    ScrollingFrame.Parent = Main
    ScrollingFrame.Size = UDim2.new(1, -10, 1, -50)
    ScrollingFrame.Position = UDim2.new(0, 5, 0, 40)
    ScrollingFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
    ScrollingFrame.ScrollBarThickness = 10
    ScrollingFrame.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    ScrollingFrame.BorderSizePixel = 1
    ScrollingFrame.AutomaticCanvasSize = Enum.AutomaticSize.Y

    UIListLayout.Parent = ScrollingFrame
    UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
    UIListLayout.Padding = UDim.new(0, 5)
    UIListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center

    -- Update the canvas size
    local function updateCanvasSize()
        -- Update canvas size only if there's a change in content
        ScrollingFrame.CanvasSize = UDim2.new(0, 0, 0, UIListLayout.AbsoluteContentSize.Y + 10)
    end

    -- Create a button for a ClickDetector
    local function createClickDetectorButton(cd)
        if buttonMap[cd] then return end -- Avoid duplicates

        local button = Instance.new("TextButton")
        button.Size = UDim2.new(1, -20, 0, 30)
        button.Text = cd.Parent and cd.Parent.Name or "Unknown"
        button.Parent = ScrollingFrame
        button.BackgroundColor3 = activeClickDetectors[cd] and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
        button.TextColor3 = Color3.fromRGB(0, 0, 0)
        button.Font = Enum.Font.SourceSans
        button.TextSize = 14
        button.BorderSizePixel = 2
        button.AutoButtonColor = true

        button.MouseButton1Click:Connect(function()
            if activeClickDetectors[cd] then
                activeClickDetectors[cd] = nil -- Stop firing
                button.BackgroundColor3 = Color3.fromRGB(255, 0, 0) -- Indicate it's off
            else
                activeClickDetectors[cd] = true -- Start firing
                button.BackgroundColor3 = Color3.fromRGB(0, 255, 0) -- Indicate it's on

                -- Trigger firing at intervals with a debounce
                local function debounceFire()
                    local currentTime = tick()
                    if not lastFireTime[cd] or currentTime - lastFireTime[cd] > debounceTime then
                        fireclickdetector(cd)
                        lastFireTime[cd] = currentTime
                    end
                end

                -- Use a heartbeat connection to fire periodically
                local heartbeatConnection
                heartbeatConnection = game:GetService("RunService").Heartbeat:Connect(function()
                    if not activeClickDetectors[cd] then
                        heartbeatConnection:Disconnect()
                    else
                        debounceFire()
                    end
                end)
            end
        end)

        buttonMap[cd] = button -- Track the button for this ClickDetector
        updateCanvasSize()
    end

    -- Remove a button when the ClickDetector is removed
    local function removeClickDetectorButton(cd)
        local button = buttonMap[cd]
        if button then
            button:Destroy()
            buttonMap[cd] = nil
            activeClickDetectors[cd] = nil -- Clear toggle state
            updateCanvasSize()
        end
    end

    -- Scan for all ClickDetectors and update the GUI
    local function scanForClickDetectors()
        for _, descendant in pairs(workspace:GetDescendants()) do
            if descendant:IsA("ClickDetector") then
                createClickDetectorButton(descendant)
            end
        end
    end

    -- Initial scan
    scanForClickDetectors()

    -- Watch for changes
    workspace.DescendantAdded:Connect(function(descendant)
        if descendant:IsA("ClickDetector") then
            createClickDetectorButton(descendant)
        end
    end)

    workspace.DescendantRemoving:Connect(function(descendant)
        if descendant:IsA("ClickDetector") then
            removeClickDetectorButton(descendant)
        end
    end)
end

-- Create the GUI
createGUI()
