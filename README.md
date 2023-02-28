# ProfileService туториал

Это подробный туториал по ProfileService. Для чайников - это продвинутый и улучшенный DataStore с встроенной защитой от потери данных, и от дюпов данных.

После предыдущих шагов, вставляем в созданный нами модуль данное:

```lua
local PlayerDataHandler = {}

local mainStats = {
	 Cash = 100,
   Diamonds = 50
}
-- данная таблица, это наши данные. Сюда может идти так и таблица, boolValue и т.д, в нашем случае, мы используем только валюту.

local ProfileService = require(game.ServerScriptService:WaitForChild("ProfileService")) -- получаем модуль ProfileService
local Players = game:GetService("Players") -- получаем сервис игроков

local ProfileStore = ProfileService.GetProfileStore( -- создаем датастор
	"DataStoreName", -- ставим имя нашему датастор
	mainStats -- вставляем данные для нашего датастора
)

local Profiles = {} -- в данной таблице будут записаны профили игроково

local function PlayerAdded(player) -- создаем функцию при заходе игрока
	local profile = ProfileStore:LoadProfileAsync("Player_"..player.UserId) -- создаем ключ для нашего датастора, и так же загружаем профиль

	if profile then -- проверяет загрузился ли профиль
		profile:AddUserId() -- добавляет айди игрока в таблицу
		profile:Reconcile() -- заполняет не достающие переменные в mainStats

		profile:ListenToRelease(function()
			Profiles[player] = nil
			player:Kick()
		end)

		if not player:IsDescendantOf(Players) then
      -- если игрока нету на сервере, то его данные удаляются с сервера (не бойтесь, данный датастор удаляет данные из сервера, и при заходе возращает их)
			profile:Release()
		else
      -- загружает данные игрока, если он на сервере
			Profiles[player] = profile
			print(Profiles[player].Data)
			wait(.1)

		end
	else
    -- если что-то пошло по жопе, то игрока кикает
		player:Kick()
	end
end

function PlayerDataHandler:Init()
  -- данная функция загружает сам модуль
	for _, v in Players:GetPlayers() do
		task.spawn(PlayerAdded, v)
	end

	game.Players.PlayerAdded:Connect(PlayerAdded) -- если игрок зашел, то он активирует функцию PlayerAdded

	game.Players.PlayerRemoving:Connect(function(player) -- если игрок выходит, то датастор удаляет данные с сервера.
		if Profiles[player] then
			Profiles[player]:Release()
		end
	end)
end

local function GetProfile(player) -- данная функция получает профиль игрока
	assert(Profiles[player], string.format("Profile does not exist for %s", player.UserId)) -- пишет ошибку в оутпуте, если профиль игрока не найден

	return Profiles[player]
end

function PlayerDataHandler:Get(player, key) -- данная функция получает данные игрока
	local profile = GetProfile(player) -- получает профиль игрока
	assert(profile.Data[key], string.format("Data does not exist for %s", key)) -- пишет ошибку в оутпуте, если ключ для данных не найден

	return profile.Data[key]
end

function PlayerDataHandler:Set(player, key, value) -- удаляет старые данные игрока, и ставит новые заданные
	local profile = GetProfile(player) -- получает профиль игрока
	assert(profile.Data[key], string.format("Data does not exist for %s", key)) -- пишет ошибку в оутпуте, если ключ для данных не найден

	assert(type(profile.Data[key]) == type(value))

	profile.Data[key] = value -- заменяем старые данные
end

function PlayerDataHandler:Update(player, key, callback) -- обновляем данные игрока
	local profile = GetProfile(player) -- получаем профиль игрока

	local oldData = self:Get(player, key) -- получаем старые данные игрока
	local newData = callback(oldData)

	self:Set(player, key, newData) -- заменяем старые данные игрока, на новые которые были заданы
end

return PlayerDataHandler
```

После чего, вы должны добавить серверный скрипт в ``ServerScriptService``, и назвать его ``PlayerDataService``. Затем вставляем туда данное:
```lua
local PlayerDataHandler = require(game.ServerScriptService:WaitForChild("PlayerDataHandler")) -- получаем модуль с которым будем работать далее (так надо делать постоянно когда работаете с ProfileService в каждом ИМЕННО серверным скриптом.

PlayerDataHandler:Init() -- запускает модуль, заметьте, :Init() появился не сам по себе, а тогда когда мы указали эту функцию в ``PlayerDataHandler``
```

На этом вы закончили устанавливать ProfileService в свою игру. Данный модуль полностью кастомный, так что делайте с ним что хотите.

# Но вот как мне например изменить значение?

Все очень легко! Для этого создаем серверный скрипт, и например вы хотите чтобы при каждом клике на кнопку, у вас добавлялось +100 к значению.

```lua
local ClickDetector = script.Parent.ClickDetector
local PlayerDataHandler = require(game.ServerScriptService
