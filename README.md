--[[=====================================================
 Script Educacional v2.7
 Rayfield UI | ESP Premium | Aimbot | Config Completa
=======================================================]]

--// SERVICES
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

--// LOAD RAYFIELD
local Rayfield = loadstring(game:HttpGet("https://sirius.menu/rayfield"))()

--// STATE
local State = {
	ESP = false,
	Aimbot = false,
	AimLock = false,
	TeamCheck = true,
	Smooth = 0.25,

	HoldKey = Enum.UserInputType.MouseButton2,
	Holding = false,
	WaitingKey = false,

	Colors = {
		Enemy = Color3.fromRGB(255,70,70),
		Friendly = Color3.fromRGB(70,255,70),
		Tools = Color3.fromRGB(220,220,220)
	},

	ESPSettings = {
		Box = true,
		Outline = true,
		HealthBar = true,
		Name = true,
		Distance = true,
		Tools = true,
		ScaleByDistance = true
	}
}

--// WINDOW
local Window = Rayfield:CreateWindow({
	Name = "Mika V1",
	LoadingTitle = "Loading",
	LoadingSubtitle = "ESP / AIMBOT",
	ConfigurationSaving = { Enabled = false }
})

local MainTab = Window:CreateTab("Main", 4483362458)
local ESPTab = Window:CreateTab("ESP Visual", 4483362458)
local ConfigTab = Window:CreateTab("Configurações", 4483362458)

--// INPUT (KEYBIND)
UIS.InputBegan:Connect(function(i,g)
	if g then return end

	if State.WaitingKey then
		State.HoldKey = i.UserInputType ~= Enum.UserInputType.Keyboard and i.UserInputType or i.KeyCode
		State.WaitingKey = false
		Rayfield:Notify({
			Title = "Keybind",
			Content = "Tecla definida: "..tostring(State.HoldKey),
			Duration = 3
		})
		return
	end

	if i.UserInputType == State.HoldKey or i.KeyCode == State.HoldKey then
		State.Holding = true
	end
end)

UIS.InputEnded:Connect(function(i)
	if i.UserInputType == State.HoldKey or i.KeyCode == State.HoldKey then
		State.Holding = false
	end
end)

--// TEAM CHECK
local function IsEnemy(plr)
	if not State.TeamCheck then return true end
	if not plr.Team or not LocalPlayer.Team then return false end
	return plr.Team ~= LocalPlayer.Team
end

--// UTILS
local function StudsDistance(pos)
	return math.floor((Camera.CFrame.Position - pos).Magnitude)
end

local function GetScale(dist)
	if not State.ESPSettings.ScaleByDistance then return 1 end
	return math.clamp(1 / (dist / 60), 0.6, 1.2)
end

--// ===================== ESP =====================
local ESP = { Cache = {} }

function ESP:Clear(plr)
	if self.Cache[plr] then
		for _,d in pairs(self.Cache[plr]) do
			pcall(function() d:Remove() end)
		end
		self.Cache[plr] = nil
	end
end

RunService.RenderStepped:Connect(function()
	if not State.ESP then
		for p in pairs(ESP.Cache) do ESP:Clear(p) end
		return
	end

	for _,plr in ipairs(Players:GetPlayers()) do
		if plr == LocalPlayer then continue end

		local char = plr.Character
		local hrp = char and char:FindFirstChild("HumanoidRootPart")
		local hum = char and char:FindFirstChildOfClass("Humanoid")
		if not (hrp and hum and hum.Health > 0) then
			ESP:Clear(plr)
			continue
		end

		local pos, on = Camera:WorldToViewportPoint(hrp.Position)
		if not on then
			ESP:Clear(plr)
			continue
		end

		local dist = StudsDistance(hrp.Position)
		local scale = GetScale(dist)

		ESP.Cache[plr] = ESP.Cache[plr] or {}
		local c = ESP.Cache[plr]

		local boxSize = Vector2.new(40,65) * scale
		local boxPos = Vector2.new(pos.X - boxSize.X/2, pos.Y - boxSize.Y/2)

		if State.ESPSettings.Box then
			if not c.Box then
				c.Box = Drawing.new("Square")
				c.Box.Thickness = 1
				c.Box.Filled = false
			end
			c.Box.Size = boxSize
			c.Box.Position = boxPos
			c.Box.Color = IsEnemy(plr) and State.Colors.Enemy or State.Colors.Friendly
			c.Box.Visible = true
		end

		if State.ESPSettings.Outline then
			if not c.Outline then
				c.Outline = Drawing.new("Square")
				c.Outline.Thickness = 3
				c.Outline.Filled = false
				c.Outline.Color = Color3.new(0,0,0)
			end
			c.Outline.Size = boxSize
			c.Outline.Position = boxPos
			c.Outline.Visible = true
		end

		if State.ESPSettings.HealthBar then
			if not c.Health then
				c.Health = Drawing.new("Line")
				c.Health.Thickness = 2
			end
			local hp = hum.Health / hum.MaxHealth
			c.Health.From = Vector2.new(boxPos.X - 5, boxPos.Y + boxSize.Y)
			c.Health.To = Vector2.new(boxPos.X - 5, boxPos.Y + boxSize.Y * (1 - hp))
			c.Health.Color = Color3.fromRGB(80,255,80)
			c.Health.Visible = true
		end

		if State.ESPSettings.Name then
			if not c.Name then
				c.Name = Drawing.new("Text")
				c.Name.Size = 13
				c.Name.Center = true
				c.Name.Outline = true
			end
			c.Name.Text = plr.Name
			c.Name.Position = Vector2.new(pos.X, boxPos.Y - 14)
			c.Name.Color = State.Colors.Friendly
			c.Name.Visible = true
		end

		if State.ESPSettings.Distance then
			if not c.Distance then
				c.Distance = Drawing.new("Text")
				c.Distance.Size = 12
				c.Distance.Center = true
				c.Distance.Outline = true
			end
			c.Distance.Text = dist.."m"
			c.Distance.Position = Vector2.new(pos.X, boxPos.Y + boxSize.Y + 2)
			c.Distance.Color = Color3.fromRGB(200,200,200)
			c.Distance.Visible = true
		end

		if State.ESPSettings.Tools then
			if not c.Tools then
				c.Tools = Drawing.new("Text")
				c.Tools.Size = 12
				c.Tools.Center = true
				c.Tools.Outline = true
			end

			local toolName = "Nenhuma"
			for _,v in ipairs(char:GetChildren()) do
				if v:IsA("Tool") then
					toolName = v.Name
					break
				end
			end

			c.Tools.Text = toolName
			c.Tools.Position = Vector2.new(pos.X, boxPos.Y + boxSize.Y + 14)
			c.Tools.Color = State.Colors.Tools
			c.Tools.Visible = true
		end
	end
end)

Players.PlayerRemoving:Connect(function(p)
	ESP:Clear(p)
end)

--// ===================== AIMBOT =====================
local function GetTarget()
	local best, dist
	local center = Camera.ViewportSize / 2

	for _,plr in ipairs(Players:GetPlayers()) do
		if plr ~= LocalPlayer and IsEnemy(plr) then
			local char = plr.Character
			local head = char and char:FindFirstChild("Head")
			local hum = char and char:FindFirstChildOfClass("Humanoid")

			if head and hum and hum.Health > 0 then
				local pos,on = Camera:WorldToViewportPoint(head.Position)
				if on then
					local d = (Vector2.new(pos.X,pos.Y)-center).Magnitude
					if not dist or d < dist then
						dist = d
						best = head
					end
				end
			end
		end
	end
	return best
end

RunService.RenderStepped:Connect(function()
	if not State.Aimbot then return end
	if not State.AimLock and not State.Holding then return end

	local target = GetTarget()
	if not target then return end

	local cf = CFrame.lookAt(Camera.CFrame.Position, target.Position)
	Camera.CFrame = Camera.CFrame:Lerp(cf, State.Smooth)
end)

--// ===================== UI =====================
MainTab:CreateToggle({ Name="ESP", Callback=function(v) State.ESP=v end })
MainTab:CreateToggle({ Name="Aimbot", Callback=function(v) State.Aimbot=v end })
MainTab:CreateToggle({ Name="Team Check", CurrentValue=true, Callback=function(v) State.TeamCheck=v end })
MainTab:CreateSlider({
	Name="Aim Smooth",
	Range={0.05,1},
	Increment=0.05,
	CurrentValue=0.25,
	Callback=function(v) State.Smooth=v end
})

for k,_ in pairs(State.ESPSettings) do
	ESPTab:CreateToggle({
		Name = k,
		CurrentValue = true,
		Callback = function(v) State.ESPSettings[k] = v end
	})
end

ConfigTab:CreateToggle({
	Name="Aimbot Grudar",
	Callback=function(v) State.AimLock=v end
})

ConfigTab:CreateButton({
	Name="Escolher Tecla do Aimbot",
	Callback=function()
		State.WaitingKey = true
		Rayfield:Notify({
			Title="Keybind",
			Content="Pressione qualquer tecla...",
			Duration=3
		})
	end
})

ConfigTab:CreateColorPicker({
	Name="Cor Inimigo",
	Color=State.Colors.Enemy,
	Callback=function(c) State.Colors.Enemy=c end
})

ConfigTab:CreateColorPicker({
	Name="Cor Aliado",
	Color=State.Colors.Friendly,
	Callback=function(c) State.Colors.Friendly=c end
})

ConfigTab:CreateColorPicker({
	Name="Cor Tools",
	Color=State.Colors.Tools,
	Callback=function(c) State.Colors.Tools=c end
})

Rayfield:Notify({
	Title="Loaded",
	Content="ESP Premium + Aimbot carregado",
	Duration=4
})

--// ===================== TROLL TAB =====================
local TrollTab = Window:CreateTab("Troll", 4483362458)

--// ===================== NOCLIP =====================
State.Noclip = false
RunService.Stepped:Connect(function()
	if State.Noclip and LocalPlayer.Character then
		for _,v in pairs(LocalPlayer.Character:GetDescendants()) do
			if v:IsA("BasePart") then
				v.CanCollide = false
			end
		end
	end
end)

TrollTab:CreateToggle({
	Name = "Noclip",
	CurrentValue = false,
	Callback = function(v)
		State.Noclip = v
	end
})

--// ===================== FLY =====================
State.Fly = {
	Enabled = false,
	Speed = 60
}

local BV, BG
RunService.RenderStepped:Connect(function()
	if not State.Fly.Enabled then return end
	local char = LocalPlayer.Character
	local hrp = char and char:FindFirstChild("HumanoidRootPart")
	if not hrp then return end

	local cam = workspace.CurrentCamera
	local move = Vector3.zero

	if UIS:IsKeyDown(Enum.KeyCode.W) then move += cam.CFrame.LookVector end
	if UIS:IsKeyDown(Enum.KeyCode.S) then move -= cam.CFrame.LookVector end
	if UIS:IsKeyDown(Enum.KeyCode.A) then move -= cam.CFrame.RightVector end
	if UIS:IsKeyDown(Enum.KeyCode.D) then move += cam.CFrame.RightVector end
	if UIS:IsKeyDown(Enum.KeyCode.Space) then move += cam.CFrame.UpVector end
	if UIS:IsKeyDown(Enum.KeyCode.LeftControl) then move -= cam.CFrame.UpVector end

	BV.Velocity = move.Unit * State.Fly.Speed
end)

TrollTab:CreateToggle({
	Name = "Fly",
	CurrentValue = false,
	Callback = function(v)
		State.Fly.Enabled = v
		local char = LocalPlayer.Character
		local hrp = char and char:FindFirstChild("HumanoidRootPart")
		if not hrp then return end

		if v then
			BV = Instance.new("BodyVelocity", hrp)
			BV.MaxForce = Vector3.new(9e9,9e9,9e9)
			BV.Velocity = Vector3.zero

			BG = Instance.new("BodyGyro", hrp)
			BG.MaxTorque = Vector3.new(9e9,9e9,9e9)
			BG.CFrame = hrp.CFrame
		else
			if BV then BV:Destroy() BV = nil end
			if BG then BG:Destroy() BG = nil end
		end
	end
})

TrollTab:CreateSlider({
	Name = "Fly Speed",
	Range = {20, 150},
	Increment = 5,
	CurrentValue = 60,
	Callback = function(v)
		State.Fly.Speed = v
	end
})

--// ===================== PUXAR TODAS AS TOOLS =====================
TrollTab:CreateButton({
	Name = "Puxar todas as Tools",
	Callback = function()
		for _,v in pairs(workspace:GetDescendants()) do
			if v:IsA("Tool") then
				v.Parent = LocalPlayer.Backpack
			end
		end
		Rayfield:Notify({
			Title = "Tools",
			Content = "Todas as tools possíveis foram puxadas",
			Duration = 3
		})
	end
})

--// ===================== CHAT GLOBAL =====================
local TextChatService = game:GetService("TextChatService")
local GlobalMessage = ""

TrollTab:CreateInput({
	Name = "Mensagem Global",
	PlaceholderText = "Digite a mensagem...",
	RemoveTextAfterFocusLost = false,
	Callback = function(txt)
		GlobalMessage = txt
	end
})

TrollTab:CreateButton({
	Name = "Enviar para Todos",
	Callback = function()
		if GlobalMessage == "" then return end
		local channel = TextChatService.ChatInputBarConfiguration.TargetTextChannel
		if channel then
			channel:SendAsync(GlobalMessage)
		end
	end
})

--// ===================== BIG HEAD SYSTEM (ADICIONADO) =====================
State.BigHead = {
	Enabled = false,
	Size = 2,
	Transparency = 0
}

--// ===================== SAVEINSTANCE BUTTON =====================

ConfigTab:CreateButton({
	Name = "Save Instance (Mapa)",
	Callback = function()
		if typeof(saveinstance) == "function" then
			Rayfield:Notify({
				Title = "SaveInstance",
				Content = "Salvando o jogo...",
				Duration = 3
			})

			-- chamada padrão
			saveinstance({
				Mode = "full",
				SavePlayers = true,
				IgnoreArchivable = false
			})

			Rayfield:Notify({
				Title = "SaveInstance",
				Content = "SaveInstance executado com sucesso",
				Duration = 4
			})
		else
			Rayfield:Notify({
				Title = "Erro",
				Content = "Seu executor não suporta saveinstance()",
				Duration = 4
			})
		end
	end
})

RunService.RenderStepped:Connect(function()
	if not State.BigHead.Enabled then return end

	for _,plr in ipairs(Players:GetPlayers()) do
		if plr ~= LocalPlayer then
			local char = plr.Character
			local head = char and char:FindFirstChild("Head")
			if head then
				head.Size = Vector3.new(
					State.BigHead.Size,
					State.BigHead.Size,
					State.BigHead.Size
				)
				head.Transparency = State.BigHead.Transparency
			end
		end
	end
end)

ConfigTab:CreateToggle({
	Name = "Big Head (Outros)",
	Callback = function(v)
		State.BigHead.Enabled = v
	end
})

ConfigTab:CreateSlider({
	Name = "Tamanho da Cabeça",
	Range = {1, 6},
	Increment = 0.1,
	CurrentValue = 2,
	Callback = function(v)
		State.BigHead.Size = v
	end
})

ConfigTab:CreateSlider({
	Name = "Transparência da Cabeça",
	Range = {0, 1},
	Increment = 0.05,
	CurrentValue = 0,
	Callback = function(v)
		State.BigHead.Transparency = v
	end
})
