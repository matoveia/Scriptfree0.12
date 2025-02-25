local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = game.Workspace.CurrentCamera
local Mouse = LocalPlayer:GetMouse()
local RayParams = RaycastParams.new()

-- Configuração do Raycast
RayParams.IgnoreWater = true
RayParams.FilterType = Enum.RaycastFilterType.Blacklist
RayParams.FilterDescendantsInstances = {LocalPlayer.Character}

-- Nome exato da arma
local ArmaPermitida = "Arisaka 7.7mm"
local AimbotEnabled = false
local DistanceLimit = 1000 -- Distância máxima do Raycast

-- Função para verificar se está segurando a arma certa
local function TemArmaCerta()
    local Character = LocalPlayer.Character
    if Character and Character:FindFirstChildOfClass("Tool") then
        local Arma = Character:FindFirstChildOfClass("Tool")
        return Arma.Name == ArmaPermitida
    end
    return false
end

-- Função para encontrar o jogador mais próximo do centro da mira
local function GetClosestPlayer()
    local Target = nil
    local ShortestDistance = math.huge

    for _, Player in pairs(Players:GetPlayers()) do
        if Player ~= LocalPlayer and Player.Character and Player.Character:FindFirstChild("Head") then
            local Head = Player.Character.Head
            local EnemyPos, OnScreen = Camera:WorldToViewportPoint(Head.Position)

            if OnScreen then
                local DistanceToCenter = (Vector2.new(EnemyPos.X, EnemyPos.Y) - Mouse.Position).Magnitude
                if DistanceToCenter < ShortestDistance then
                    ShortestDistance = DistanceToCenter
                    Target = Head
                end
            end
        end
    end
    return Target
end

-- Função para disparar através das paredes
local function ShootRay(Target)
    if TemArmaCerta() and Target then
        local StartPosition = Camera.CFrame.Position
        local Direction = (Target.Position - StartPosition).unit * DistanceLimit

        -- Raycast para detectar o alvo
        local RaycastResult = workspace:Raycast(StartPosition, Direction, RayParams)

        -- Verifica se o raio atingiu um jogador
        if RaycastResult then
            local HitPart = RaycastResult.Instance
            if HitPart and HitPart.Parent and HitPart.Parent:FindFirstChild("Humanoid") then
                print("Jogador atingido através da parede!")
            end
        end
    end
end

-- Função para puxar a mira para o alvo suavemente
local function SmoothAim(Target, Speed)
    if Target then
        local TargetPos = Target.Position
        local CurrentCFrame = Camera.CFrame
        local NewDirection = (TargetPos - CurrentCFrame.Position).unit

        -- Suaviza a transição para a mira
        local SmoothedCFrame = CFrame.new(CurrentCFrame.Position, CurrentCFrame.Position + (CurrentCFrame.LookVector:Lerp(NewDirection, Speed)))
        Camera.CFrame = SmoothedCFrame
    end
end

-- Ativar mira ao atirar
Mouse.Button1Down:Connect(function()
    if AimbotEnabled and TemArmaCerta() then
        local Target = GetClosestPlayer()
        if Target then
            SmoothAim(Target, 0.9) -- Puxa 90% para a cabeça
            ShootRay(Target) -- Atira através das paredes
        end
    end
end)

-- Função principal do Aimbot (ativa continuamente)
local function Aimbot()
    while true do
        if AimbotEnabled and TemArmaCerta() then
            local Target = GetClosestPlayer()
            if Target then
                SmoothAim(Target, 1) -- 100% de precisão na mira
            end
        end
        task.wait(0.01)
    end
end

-- Criar botão de ativação/desativação
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
local ToggleButton = Instance.new("TextButton", ScreenGui)

ToggleButton.Size = UDim2.new(0, 200, 0, 50)
ToggleButton.Position = UDim2.new(0.1, 0, 0.1, 0)
ToggleButton.Text = "Aimbot OFF"
ToggleButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
ToggleButton.TextColor3 = Color3.new(1, 1, 1)
ToggleButton.BackgroundTransparency = 0.2  

ToggleButton.MouseButton1Click:Connect(function()
    AimbotEnabled = not AimbotEnabled
    if AimbotEnabled then
        ToggleButton.Text = "Aimbot ON"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
    else
        ToggleButton.Text = "Aimbot OFF"
        ToggleButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
    end
end)

-- Iniciar Aimbot
task.spawn(Aimbot)
