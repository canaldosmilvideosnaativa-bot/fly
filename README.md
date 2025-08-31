-- LocalScript em StarterPlayerScripts
local player = game.Players.LocalPlayer
local uis = game:GetService("UserInputService")
local runService = game:GetService("RunService")

local flying = false
local speed = 80

-- GUI
local screenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0,220,0,170)
frame.Position = UDim2.new(0.4,0,0.35,0)
frame.BackgroundColor3 = Color3.fromRGB(50,50,50)
frame.Active = true
frame.Draggable = true
frame.Parent = screenGui

local flyButton = Instance.new("TextButton", frame)
flyButton.Size = UDim2.new(0,200,0,40)
flyButton.Position = UDim2.new(0,10,0,10)
flyButton.Text = "VOAR (ou F)"
flyButton.BackgroundColor3 = Color3.fromRGB(80,80,80)
flyButton.TextColor3 = Color3.new(1,1,1)

-- SLIDER
local sliderBack = Instance.new("Frame", frame)
sliderBack.Size = UDim2.new(0,200,0,20)
sliderBack.Position = UDim2.new(0,10,0,110)
sliderBack.BackgroundColor3 = Color3.fromRGB(80,80,80)

local sliderFill = Instance.new("Frame", sliderBack)
sliderFill.Size = UDim2.new(0.5,0,1,0)
sliderFill.BackgroundColor3 = Color3.fromRGB(0,170,255)

local sliderButton = Instance.new("TextButton", sliderBack)
sliderButton.Size = UDim2.new(0,20,1,0)
sliderButton.Position = UDim2.new(0.5, -10, 0, 0)
sliderButton.Text = ""
sliderButton.BackgroundColor3 = Color3.fromRGB(200,200,200)

local speedLabel = Instance.new("TextLabel", frame)
speedLabel.Size = UDim2.new(0,200,0,20)
speedLabel.Position = UDim2.new(0,10,0,135)
speedLabel.BackgroundTransparency = 1
speedLabel.TextColor3 = Color3.new(1,1,1)
speedLabel.Text = "Velocidade: "..speed

-- Slider
local dragging = false
sliderButton.MouseButton1Down:Connect(function() dragging = true end)
uis.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
end)

runService.RenderStepped:Connect(function()
	if dragging then
		local mouse = uis:GetMouseLocation().X
		local x = math.clamp(mouse - sliderBack.AbsolutePosition.X, 0, sliderBack.AbsoluteSize.X)
		local percent = x / sliderBack.AbsoluteSize.X
		sliderButton.Position = UDim2.new(0, x - sliderButton.AbsoluteSize.X/2, 0, 0)
		sliderFill.Size = UDim2.new(percent, 0, 1, 0)
		speed = math.floor(20 + (percent * 180))
		speedLabel.Text = "Velocidade: "..speed
	end
end)

-- Teclas
local keysDown = {}
uis.InputBegan:Connect(function(input, gp)
	if not gp and input.UserInputType == Enum.UserInputType.Keyboard then
		keysDown[input.KeyCode] = true
		if input.KeyCode == Enum.KeyCode.F then
			toggleFly()
		end
	end
end)
uis.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.Keyboard then
		keysDown[input.KeyCode] = false
	end
end)

-- Função para iniciar/parar voo
function toggleFly()
	flying = not flying
	local character = player.Character
	if character and character:FindFirstChild("HumanoidRootPart") then
		local hrp = character.HumanoidRootPart

		if flying then
			-- Usar LinearVelocity (mais moderno e não "suspeito")
			local lv = Instance.new("LinearVelocity")
			lv.Name = "FlyVelocity"
			lv.Attachment0 = Instance.new("Attachment", hrp)
			lv.MaxForce = math.huge
			lv.VectorVelocity = Vector3.zero
			lv.Parent = hrp

			-- Gyro moderno (AlignOrientation)
			local ao = Instance.new("AlignOrientation")
			ao.Name = "FlyGyro"
			ao.MaxAngularVelocity = math.huge
			ao.Responsiveness = 200
			ao.Attachment0 = hrp:FindFirstChildOfClass("Attachment")
			ao.CFrame = workspace.CurrentCamera.CFrame
			ao.Parent = hrp
		else
			if hrp:FindFirstChild("FlyVelocity") then hrp.FlyVelocity:Destroy() end
			if hrp:FindFirstChild("FlyGyro") then hrp.FlyGyro:Destroy() end
		end
	end
end

flyButton.MouseButton1Click:Connect(toggleFly)

-- Loop
runService.RenderStepped:Connect(function()
	if flying then
		local character = player.Character
		if character and character:FindFirstChild("HumanoidRootPart") then
			local hrp = character.HumanoidRootPart
			local camCF = workspace.CurrentCamera.CFrame
			local direction = Vector3.zero

			if keysDown[Enum.KeyCode.W] then direction += camCF.LookVector end
			if keysDown[Enum.KeyCode.S] then direction -= camCF.LookVector end
			if keysDown[Enum.KeyCode.A] then direction -= camCF.RightVector end
			if keysDown[Enum.KeyCode.D] then direction += camCF.RightVector end
			if keysDown[Enum.KeyCode.E] then direction += Vector3.new(0,1,0) end
			if keysDown[Enum.KeyCode.Q] then direction -= Vector3.new(0,1,0) end

			if hrp:FindFirstChild("FlyVelocity") then
				if direction.Magnitude > 0 then
					hrp.FlyVelocity.VectorVelocity = direction.Unit * speed
				else
					hrp.FlyVelocity.VectorVelocity = Vector3.zero
				end
			end

			if hrp:FindFirstChild("FlyGyro") then
				hrp.FlyGyro.CFrame = workspace.CurrentCamera.CFrame
			end
		end
	end
end)
