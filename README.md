local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")
local Players = game:GetService("Players")

local player = Players.LocalPlayer

--================================================
-- 1. 그림자
--================================================
local ok1, err1 = pcall(function()
    Lighting.Technology = Enum.Technology.Future
    Lighting.GlobalShadows = true
    Lighting.ShadowSoftness = 0.2
    for _, obj in ipairs(Workspace:GetDescendants()) do
        if obj:IsA("BasePart") then obj.CastShadow = true end
    end
end)
if not ok1 then warn("그림자 설정 에러:", err1) end

--================================================
-- 2. 기본 조명 (고정값)
--================================================
local ok2, err2 = pcall(function()
    Lighting.Brightness = 2
    Lighting.EnvironmentDiffuseScale = 1
    Lighting.EnvironmentSpecularScale = 1
    Lighting.Ambient = Color3.fromRGB(70, 70, 70)
    Lighting.OutdoorAmbient = Color3.fromRGB(95, 95, 95)
end)
if not ok2 then warn("기본 조명 에러:", err2) end

--================================================
-- 3. 크롬(빛 반사) - 맵 + 총(Tool)
--================================================
local ok3, err3 = pcall(function()
    local materialReflectance = {
        [Enum.Material.Metal] = 0.7, [Enum.Material.DiamondPlate] = 0.7, [Enum.Material.Foil] = 0.7,
        [Enum.Material.Glass] = 0.6, [Enum.Material.Ice] = 0.6, [Enum.Material.Marble] = 0.5,
        [Enum.Material.SmoothPlastic] = 0.25, [Enum.Material.Plastic] = 0.15, [Enum.Material.Concrete] = 0.1,
    }

    local function applyMapChrome(obj)
        if obj:IsA("BasePart") then
            obj.Reflectance = materialReflectance[obj.Material] or 0.08
        end
    end
    for _, obj in ipairs(Workspace:GetDescendants()) do applyMapChrome(obj) end
    Workspace.DescendantAdded:Connect(applyMapChrome)

    local GUN_REFLECTANCE = 0.55
    local function applyGunChrome(tool)
        if not tool:IsA("Tool") and not tool:IsA("Accessory") then return end
        for _, part in ipairs(tool:GetDescendants()) do
            if part:IsA("BasePart") then part.Reflectance = GUN_REFLECTANCE end
        end
        tool.DescendantAdded:Connect(function(child)
            if child:IsA("BasePart") then child.Reflectance = GUN_REFLECTANCE end
        end)
    end
    local function hookContainer(container)
        for _, child in ipairs(container:GetChildren()) do applyGunChrome(child) end
        container.ChildAdded:Connect(applyGunChrome)
    end
    hookContainer(player:WaitForChild("Backpack"))
    if player.Character then hookContainer(player.Character) end
    player.CharacterAdded:Connect(hookContainer)
end)
if not ok3 then warn("크롬 반사 에러:", err3) end

--================================================
-- 4. PostEffect들
--================================================
local ok4, err4 = pcall(function()
    local cc = Lighting:FindFirstChild("RivalsStyleCC") or Instance.new("ColorCorrectionEffect")
    cc.Name = "RivalsStyleCC"
    cc.Contrast = 0.1
    cc.Saturation = 0.05
    cc.Parent = Lighting

    local bloom = Lighting:FindFirstChild("RivalsStyleBloom") or Instance.new("BloomEffect")
    bloom.Name = "RivalsStyleBloom"
    bloom.Intensity = 0.4
    bloom.Size = 24
    bloom.Threshold = 1.5
    bloom.Parent = Lighting

    local dof = Lighting:FindFirstChild("RivalsStyleDOF") or Instance.new("DepthOfFieldEffect")
    dof.Name = "RivalsStyleDOF"
    dof.FarIntensity = 0.2
    dof.FocusDistance = 50
    dof.InFocusRadius = 30
    dof.NearIntensity = 0
    dof.Parent = Lighting
end)
if not ok4 then warn("PostEffect 에러:", err4) end

print("스크립트 로드 완료 (그림자/크롬반사/PostEffect, 안개·시간전환 제외)")
