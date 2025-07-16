-- Configurações
local espColor = {r = 255, g = 255, b = 0, a = 255} -- Amarelo
local aimlockKey = VK_LBUTTON -- Botão esquerdo do mouse (segurar para aimlock)
local aimlockFOV = 30 -- Campo de visão para o aimlock

-- Variáveis globais
local players = {}
local localPlayer
local closestEnemy = nil

-- Função para obter a posição do jogador local
function getLocalPlayer()
    -- (Depende do jogo, exemplo genérico)
    localPlayer = readPointer(getBaseAddress() + 0x123456) -- Substitua pelo offset correto
    return localPlayer
end

-- Função para obter lista de jogadores
function updatePlayers()
    players = {}
    for i = 0, 64 do -- Supondo 64 jogadores máximos
        local player = readPointer(getBaseAddress() + 0x789ABC + i * 0x100) -- Offset exemplo
        if player and player ~= localPlayer and isEnemy(player) then
            table.insert(players, player)
        end
    end
end

-- Verifica se o jogador é inimigo
function isEnemy(player)
    -- (Depende do jogo, exemplo genérico)
    local team = readInteger(player + 0x50) -- Offset do time
    return team ~= readInteger(localPlayer + 0x50)
end

-- Calcula distância entre dois jogadores
function getDistance(player1, player2)
    local pos1 = getPosition(player1)
    local pos2 = getPosition(player2)
    return math.sqrt((pos1.x - pos2.x)^2 + (pos1.y - pos2.y)^2 + (pos1.z - pos2.z)^2)
end

-- Obtém a posição 3D de um jogador
function getPosition(player)
    return {
        x = readFloat(player + 0x100), -- Offset X
        y = readFloat(player + 0x104), -- Offset Y
        z = readFloat(player + 0x108)  -- Offset Z
    }
end

-- Encontra o inimigo mais próximo
function findClosestEnemy()
    local minDist = math.huge
    closestEnemy = nil
    for _, enemy in ipairs(players) do
        local dist = getDistance(localPlayer, enemy)
        if dist < minDist then
            minDist = dist
            closestEnemy = enemy
        end
    end
    return closestEnemy
end

-- AimLock: Mira no inimigo mais próximo
function aimAtEnemy()
    if closestEnemy then
        local enemyPos = getPosition(closestEnemy)
        local viewAngles = getViewAngles()
        local newAngles = calculateAngles(localPlayer, enemyPos)
        setViewAngles(newAngles)
    end
end

-- Função para desenhar ESP (caixa amarela ao redor dos inimigos)
function drawESP()
    for _, enemy in ipairs(players) do
        local pos = getPosition(enemy)
        local screenPos = worldToScreen(pos)
        if screenPos then
            drawBox(screenPos.x, screenPos.y, 50, 100, espColor) -- Exemplo de caixa
            drawText(screenPos.x, screenPos.y - 20, "INIMIGO", espColor)
        end
    end
end

-- Loop principal
function mainLoop()
    localPlayer = getLocalPlayer()
    if not localPlayer then return end
    
    updatePlayers()
    findClosestEnemy()
    
    if isKeyPressed(aimlockKey) and closestEnemy then
        aimAtEnemy()
    end
    
    drawESP()
end

-- Executa o loop
setInterval(mainLoop, 10) -- 10ms de atualização
