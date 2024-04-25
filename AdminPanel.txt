
local ScreenGui = Instance.new("ScreenGui")
local Frame = Instance.new("Frame")
local TextBox = Instance.new("TextBox")
local TextLabel = Instance.new("TextLabel")
local CloseButton = Instance.new("TextButton")
local version = "0.02"
ScreenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
ScreenGui.ResetOnSpawn = false
local outlinesespz = {}
Frame.Parent = ScreenGui
Frame.BackgroundColor3 = Color3.new(0, 0, 0)
Frame.Size = UDim2.new(0.5, 0, 0.5, 0)
Frame.Position = UDim2.new(0.5, 0, 0.5, 0)
Frame.AnchorPoint = Vector2.new(0.5, 0.5)
Frame.BackgroundTransparency = 0.4
Frame.BorderSizePixel = 0
Frame.Selectable = true
Frame.Active = true
Frame.Draggable = true

local FrameUICorner = Instance.new("UICorner")
FrameUICorner.CornerRadius = UDim.new(0, 10)
FrameUICorner.Parent = Frame

TextBox.Parent = Frame
TextBox.BackgroundColor3 = Color3.fromRGB(169, 169, 169)
TextBox.Size = UDim2.new(1, -20, 0, 20)
TextBox.Position = UDim2.new(0.5, 0, 1, -10)
TextBox.AnchorPoint = Vector2.new(0.5, 1)
TextBox.BackgroundTransparency = 0.1
TextBox.BorderSizePixel = 0
TextBox.Text = ""
TextBox.TextXAlignment = Enum.TextXAlignment.Left
local TextBoxUICorner = Instance.new("UICorner")
TextBoxUICorner.CornerRadius = UDim.new(0, 4)
TextBoxUICorner.Parent = TextBox

TextLabel.Parent = Frame
TextLabel.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
TextLabel.BackgroundTransparency = 1
TextLabel.Size = UDim2.new(1, -20, 1, -50)
TextLabel.Position = UDim2.new(0, 10, 0, 10)
TextLabel.TextYAlignment = Enum.TextYAlignment.Bottom
TextLabel.Text = ""
TextLabel.TextColor3 = Color3.new(1, 1, 1)
TextLabel.TextWrapped = true
TextLabel.Font = Enum.Font.Code
TextLabel.TextSize = 14

CloseButton.Parent = Frame
CloseButton.BackgroundColor3 = Color3.new(1, 0, 0)
CloseButton.Size = UDim2.new(0, 25, 0, 25)
CloseButton.Position = UDim2.new(0.9, 5, 0, 5)
CloseButton.Text = ""
CloseButton.TextColor3 = Color3.new(1, 1, 1)
CloseButton.Font = Enum.Font.Code
CloseButton.TextSize = 14

local CloseButtonUICorner = Instance.new("UICorner")
CloseButtonUICorner.CornerRadius = UDim.new(0.3, 0)
CloseButtonUICorner.Parent = CloseButton

CloseButton.MouseButton1Click:Connect(function()
    ScreenGui:Destroy()
end)



local Commands = {}

function RegisterCommand(commandName, commandFunction)
    Commands[commandName] = commandFunction
end

function ExecuteCommand(commandName, ...)
    if Commands[commandName] then
        return Commands[commandName](...)
    else
        return "Command not found."
    end
end


TextLabel.TextXAlignment = Enum.TextXAlignment.Left
TextBox.FocusLost:Connect(function(enterPressed)
    if enterPressed then
        local input = TextBox.Text
        TextBox.Text = ""
        local commandName, args = input:match("^(%S+)%s*(.*)")
        local output = ExecuteCommand(commandName, unpack(args:split(" ")))
        local time = os.date("%X")
        TextLabel.Text = TextLabel.Text .. time .. "~ " .. output .. "\n"
    end
end)


RegisterCommand("version", function()
    return "Version: "..version
end)

RegisterCommand("respawn", function()
    local seconds = 0
    local done = false
    
    local function timer()
        while wait(1) do
            seconds = seconds + 1
            if done == true then
                break
            end
        end
    end
    
    local location = game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame
    
    
    game.Players.LocalPlayer.Character:BreakJoints()
    coroutine.wrap(timer)()
    game.Players.LocalPlayer.CharacterAdded:Connect(function()
        wait(0.3) 
        game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = location
        
        done = true
    end)
    task.wait()
    
    return "Respawning player..."
end)

RegisterCommand("clear", function()
TextLabel.Text = ""
    return "Sucesfully cleared!"
end)

RegisterCommand("espz", function(pn)
local playerNameInput = pn



local function findPlayer(partialName)
    for _, player in ipairs(game.Players:GetPlayers()) do
        if string.find(string.lower(player.Name), string.lower(partialName), 1, true) then
            return player
        end
    end
    return nil
end

local function addOutline(character)
    local outlineParts = {}
    for _, part in ipairs(character:GetChildren()) do
        if part:IsA("BasePart") then
            local outline = Instance.new("SelectionBox")
            outline.Adornee = part
            outline.Color3 = Color3.new(1, 0, 0)
            outline.LineThickness = 0.05
            outline.Transparency = 0.5
            outline.Visible = true
            outline.Parent = part
            table.insert(outlineParts, outline)
        end
    end
    table.insert(outlinesespz, outlineParts)
end



local partialName = pn
local foundPlayer = findPlayer(partialName)
if foundPlayer then
    addOutline(foundPlayer.Character)
else
    print("Player not found.")
end

return "Outline succesfully added!"
end)


RegisterCommand("un.espz", function()
    for _, outlineParts in ipairs(outlinesespz) do
        for _, outline in ipairs(outlineParts) do
            outline:Destroy()
        end
    end
    outlinesespz = {}
return "Outlines sucesfully deleted!"
end)

RegisterCommand("tp", function(pn)
local function findPlayer(partialName)
    for _, player in ipairs(game.Players:GetPlayers()) do
        if string.find(string.lower(player.Name), string.lower(partialName), 1, true) then
            return player
        end
    end
    return nil
end

local function teleportToPlayer(player)
    game.Players.LocalPlayer.Character:MoveTo(player.Character.HumanoidRootPart.Position)
end

local partialName = pn
local foundPlayer = findPlayer(partialName)
if foundPlayer then
    teleportToPlayer(foundPlayer)
else
    print("Player not found.")
end
return "Succesfully teleported to: "..pn
end)

RegisterCommand("load:", function(url)
    local content = game:HttpGet(url)
    if content then
        local script, errorMessage = loadstring(content)
        if script then
            script()
            return "Loading > " .. url
        else
            return "Error: Invalid script"
        end
    else
        return "Error: Invalid URL or unable to retrieve content"
    end
end)

RegisterCommand("alone", function()
    local function quarantineEnvironment()
        local quarantine = Instance.new("Folder")
        quarantine.Name = "Quarantine"

        local function deleteAllParts()
            for _, player in ipairs(game.Players:GetPlayers()) do
                if player ~= game.Players.LocalPlayer then
                    player.Character:Destroy()
                    return player.Name .. ", has been removed!"
                end
            end
        end

        game.Players.PlayerAdded:Connect(function(newPlayer)
            newPlayer.Character:Destroy()
            return newPlayer.Name .. ", has been removed!"
        end)

        while true do
            wait(5)
            deleteAllParts()
        end
    end

    quarantineEnvironment()
end)

RegisterCommand("help", function(page, cmd)
    if page == "1" and cmd == nil then
        return "version,respawn,clear,espz,un.espz"
    end
    if page == "2" and cmd == nil then
        return "tp,load:,alone"
    end
    if page == "1" and cmd == "version" then
        return "Displays the executor version. {noargs}"
    end
    if page == "1" and cmd == "respawn" then
        return "Respawns the player. {noargs}"
    end
    if page == "1" and cmd == "clear" then
        return "Clears the console. {noargs}"
    end
    if page == "1" and cmd == "espz" then
        return "Adds outlines to a player {espz plrname}"
    end
    if page == "1" and cmd == "un.espz" then
        return "Removes the outlines. {noargs}"
    end
    if page == "2" and cmd == "tp" then
        return "Teleports to a player. {tp plrname}"
    end
    if page == "2" and cmd == "load:" then
        return "Loads a link with a script."
    end
    if page == "2" and cmd == "alone" then
        return "Not working"
    end
    return "Enter a page and a command to view" 
end)

