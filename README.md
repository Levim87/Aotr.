pcall(function()
    while not (game:IsLoaded() and game.Players.LocalPlayer.Character and workspace.Titans:GetChildren()) do
        task.wait()
    end

    local Titans = workspace:WaitForChild("Titans")
    local Remotes = game.ReplicatedStorage:WaitForChild("Assets"):WaitForChild("Remotes")

    -- Player Objects
    local Player = game.Players.LocalPlayer
    local Character = Player.Character or Player.CharacterAdded:Wait()
    local Humanoid = Character:WaitForChild("Humanoid")
    local HumanoidRootPart = Character:WaitForChild("HumanoidRootPart")
    local PlayerGui = Player:WaitForChild("PlayerGui")

    -- Variables
    local Total = 0
    local TitansStorage = {}
    local TitansProcessed = 0

    task.spawn(function()
        for _, Titan in ipairs(Titans:GetChildren()) do
            if Titan:IsA("Model") and Titan:FindFirstChild("Humanoid") then
                TitansStorage[Titan.Name] = Titan
            end
        end

        while true do
            Total = #Titans:GetChildren()
            task.wait()
        end
    end)

    task.spawn(function()
        local Interface = PlayerGui:FindFirstChild("Interface")
        if Interface then
            Interface.ChildAdded:Connect(function(Child)
                if Child.Name == "Numbers" then
                    task.spawn(function()
                        while Child.Parent do
                            Child:Destroy()
                            task.wait()
                        end
                    end)
                end
            end)
        end

        for _, Titan in pairs(TitansStorage) do
            local Nape = Titan.Hitboxes.Hit.Nape
            if Nape then
                Nape.ChildAdded:Connect(function(Child)
                    if Child.Name == "Blood" or Child.Name == "Hit" then
                        task.spawn(function()
                            while Child.Parent do
                                Child:Destroy()
                                task.wait()
                            end
                        end)
                    end
                end)
            end
        end
    end)

    task.spawn(function()
        for _, Titan in pairs(TitansStorage) do
            local Nape = Titan.Hitboxes.Hit.Nape
            TitansProcessed = TitansProcessed + 1
            task.spawn(function()
                while Nape and Nape.Parent and Nape.Parent.Parent do
                    Nape.Position = HumanoidRootPart.Position
                    task.wait()
                end
            end)
        end

        local index = 0

        Remotes.GET:InvokeServer("Functions", "Retry", "Add")

        while TitansProcessed ~= Total do
            task.wait()
        end

        Remotes.GET:InvokeServer("S_Skills", "Usage", "23")
        Remotes.GET:InvokeServer("S_Skills", "Usage", "14")

        while true do
            for _, v in pairs(Titans:GetChildren()) do
                if v:FindFirstChild("Humanoid") then
                    index = index + 1
                end
            end

            Notify("Titans Remaining", tostring(index), 5)
            task.wait(2)

            index = 0
        end
    end)

    -- Additional Optimization
    local Lighting = game:GetService("Lighting")
    for _, LightObj in pairs(Lighting:GetChildren()) do
        LightObj:Destroy()
    end

    local UselessShit = {"Climbable", "Debris", "Points", "Background", "Platforms", "Props", "Reloads", "Trees", "U_Buildings", "Tree_Colliders", "Fake", "Torrential_Steel", "Drill_Thrust"}
    for _, Obj in ipairs(game:GetDescendants()) do
        if Obj:IsA("ParticleEmitter") or table.find(UselessShit, Obj.Name) then
            Obj:Destroy()
        end
    end
end)
