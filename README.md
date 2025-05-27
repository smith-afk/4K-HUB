local player = game.Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TextChatService = game:GetService("TextChatService")
local camera = workspace.CurrentCamera

-- Criar GUI
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "FlyGui"

local button = Instance.new("TextButton")
button.Parent = gui
button.Size = UDim2.new(0, 100, 0, 40)
button.Position = UDim2.new(0.05, 0, 0.1, 0)
button.Text = "Fly"
button.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
button.TextColor3 = Color3.fromRGB(255, 255, 255)
button.TextScaled = true
button.Font = Enum.Font.SourceSansBold

-- Variáveis
local flying = false
local speed = 100
local connection
local bv, bg
local animationTrack

-- ID da animação de voo
local animationId = "rbxassetid://507766388" -- Substitua pelo seu ID

local function startFlying()
	if flying then return end
	flying = true
	local character = player.Character or player.CharacterAdded:Wait()
	local hrp = character:WaitForChild("HumanoidRootPart")
	local humanoid = character:WaitForChild("Humanoid")

	-- Criar BodyVelocity e BodyGyro
	bv = Instance.new("BodyVelocity")
	bv.MaxForce = Vector3.new(9e9, 9e9, 9e9)
	bv.Velocity = Vector3.zero
	bv.P = 1000
	bv.Parent = hrp

	bg = Instance.new("BodyGyro")
	bg.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
	bg.P = 10000
	bg.CFrame = hrp.CFrame
	bg.Parent = hrp

	-- Tocar animação
	local anim = Instance.new("Animation")
	anim.AnimationId = animationId
	animationTrack = humanoid:LoadAnimation(anim)
	animationTrack.Priority = Enum.AnimationPriority.Action
	animationTrack:Play()

	-- Atualizar voo com inclinação
	connection = RunService.RenderStepped:Connect(function()
		local direction = camera.CFrame.LookVector
		bv.Velocity = direction * speed

		local tiltAngle = math.rad(30)
		local flatLook = Vector3.new(direction.X, 0, direction.Z).Unit
		local tiltDirection = CFrame.new(hrp.Position, hrp.Position + flatLook) * CFrame.Angles(-tiltAngle, 0, 0)
		bg.CFrame = tiltDirection
	end)

	button.Text = "Unfly"
end

local function stopFlying()
	if not flying then return end
	flying = false
	if connection then connection:Disconnect() end
	if bv then bv:Destroy() end
	if bg then bg:Destroy() end
	if animationTrack then
		animationTrack:Stop()
		animationTrack:Destroy()
	end
	button.Text = "Fly"
end

local function toggleFly()
	if flying then
		stopFlying()
	else
		startFlying()
	end
end

button.MouseButton1Click:Connect(toggleFly)

-- Suporte a comandos no chat (TextChatService)
TextChatService.OnIncomingMessage = function(message)
	if message.TextSource and message.TextSource.UserId == player.UserId then
		local msg = message.Text:lower()
		if msg == ";fly" then
			startFlying()
		elseif msg == ";unfly" then
			stopFlying()
		end
	end
end
