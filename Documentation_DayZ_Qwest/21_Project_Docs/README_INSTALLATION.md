# Silver_77_Quests - Полная инструкция по установке

## ⚡ Быстрая установка (для пользователей)

1. Распакуй `@Silver_77_Quests` в папку с модами сервера
2. Добавь в `serverDZ.cfg`: `mods = @Silver_77_Quests;`
3. Запусти сервер - готово!

Мод автоматически создаст минимальный стартовый конфиг квестов. Никакой ручной настройки не требуется!

## 🔧 Сборка мода (для разработчиков)

### Через DayZ Tools Addon Builder:
1. Открой DayZ Tools → Addon Builder
2. Source Directory: путь к папке `Silver_77_Quests`
3. Destination Folder: путь для `@Silver_77_Quests`
4. PBO Name Prefix: `Silver_77_Quests` (из $PBOPREFIX$)
5. Нажми "Pack"

### Через Mikero's Tools (PboProject):
```bash
PboProject.exe -P "путь\к\Silver_77_Quests"
```

---

## Структура мода

```
Silver_77_Quests/
├── config.cpp                          (главный конфиг мода)
├── gui/
│   └── QuestMenu.layout               (интерфейс меню)
└── scripts/
    ├── 3_Game/
    │   ├── QuestData.c                (структуры данных квестов)
    │   └── PlayerQuestData.c          (данные прогресса игрока)
    ├── 4_World/
    │   └── QuestManager.c             (менеджер квестов + 2 примера)
    └── 5_Mission/
        ├── QuestTrigger.c             (триггеры и зоны)
        ├── QuestUI.c                  (UI меню)
        └── mission/
            └── MissionGameplay.c      (интеграция в игру)
```

## Установка

### Для пользователей:
1. Распакуй `@Silver_77_Quests` в папку с модами сервера
2. Добавь мод в `serverDZ.cfg`:
```
mods = @Silver_77_Quests;
```
3. Запусти сервер

### Для разработчиков:
1. Собери PBO через DayZ Tools Addon Builder
2. Скопируй `@Silver_77_Quests` в папку с модами
3. Добавь в `serverDZ.cfg`

При первом запуске мод автоматически создаст:
- `profiles/Silver_77_Quests/Silver_77_Quests.json` - минимальный стартовый конфиг квестов
- `profiles/Silver_77_Quests/players/` - папка для сохранений игроков

## Тестирование (без настройки)

1. Запусти сервер
2. Зайди в игру
3. Подойди к NPC/триггеру
4. Нажми F - откроется меню квестов
5. Возьми квест, собери предметы, сдай - получи награду!

## Настройка координат триггеров (опционально)

Мод работает сразу с координатами по умолчанию около NPC рыбака и охотника.

Если хочешь изменить координаты под свою карту:

1. Останови сервер
2. Открой: `profiles/Silver_77_Quests/Silver_77_Quests.json`
3. Найди секцию `"triggers"` и измени `"position"`:

```json
"triggers": [
  {
    "id": "fisherman_trigger",
    "position": [13092.814453, 117.007767, 13084.485352],  ← ТВОИ КООРДИНАТЫ
    "radius": 2.0,
    "focusHeight": 1.2,
    "focusRadius": 1.0,
    "questIds": ["quest_fisherman_1"],
    "hintText": "[F] Коля Ворон",
    "spawnNpc": true,
    "npcClassName": "SurvivorM_Mirek",
    "npcPosition": [13092.814453, 117.007767, 13084.485352],
    "npcOrientation": [215.0, 0.0, 0.0],
    "npcLoadout": ["FlatCap_BrownCheck", "HuntingJacket_Brown", "Jeans_Blue", "WorkingGloves_Brown", "HikingBootsLow_Black"],
    "npcHandsItem": "FarmingHoe",
    "npcBackItems": ["HuntingBag", "Izh43Shotgun"]
  }
]
```

Если `spawnNpc` включён, поменяй и `npcPosition` на те же координаты. Можно оставить `npcPosition` пустым массивом `[]`, тогда NPC и наведение будут использовать `position` триггера.

Если мод уже запускался раньше, изменения дефолтных координат в PBO не перезапишут существующий JSON в `profiles`. Нужно вручную поменять `profiles/Silver_77_Quests/Silver_77_Quests.json` или удалить этот файл, чтобы мод создал его заново.

4. Запусти сервер

### Как узнать координаты?
1. Зайди в игру
2. Встань в нужном месте
3. Открой консоль (`) и введи: `getpos`
4. Скопируй координаты X Y Z


## Как работает система

1. **Игрок подходит к триггеру** (в радиусе зоны)
2. **Нажимает F** - открывается меню квестов
3. **Выбирает квест** из списка
4. **Видит описание**, цели и награды
5. **Кнопки "Взять" / "Сдать"** активны по условиям

### Условия кнопок:
- **Взять**: квест доступен, не активен, прошел cooldown (для повторяемых)
- **Сдать**: квест активен, все предметы собраны

## Настройка квестов (опционально)

Мод уже содержит 2 примера квестов. Если хочешь добавить свои:

Открой файл: `profiles/Silver_77_Quests/Silver_77_Quests.json`

### Пример квеста (в JSON):

```json
{
  "id": "quest_my_1",
  "name": "Название квеста",
  "description": "Описание задания",
  "repeatable": true,
  "cooldownSeconds": 3600,
  "requiresPrevious": "",
  "requiredQuestIds": [],
  "giveItems": [
    {
      "className": "TaloonBag_Blue",
      "quantity": 1,
      "spawnOnGround": false
    }
  ],
  "objectives": [
    {
      "type": "item",
      "className": "Carp",
      "quantity": 3,
      "removeOnComplete": true,
      "useItemQuantity": false
    }
  ],
  "rewards": [
    {
      "className": "FishingRod",
      "quantity": 1
    }
  ]
}
```

Подробнее: `README_JSON_CONFIG.md`

## Повторяемые квесты

```cpp
myQuest.repeatable = true;           // Можно брать снова
myQuest.cooldownSeconds = 3600;      // Время до повторного взятия (в секундах)
```

- `0` = без cooldown (сразу можно взять снова)
- `3600` = 1 час
- `86400` = 24 часа

## Тестирование

1. Запусти сервер
2. Зайди в игру
3. Подойди к NPC/триггеру
4. Нажми F - должно открыться меню
5. Возьми квест - получишь предметы
6. Собери нужные предметы
7. Вернись и сдай квест - получишь награду

## Логи

Проверь логи сервера на наличие:
```
[Silver_77_Quests] QuestManager initialized
[Silver_77_Quests] Loaded 2 quests
[Silver_77_Quests] QuestTriggerManager initialized with 2 triggers
```

## Файлы данных

Мод автоматически создаст:
- `profiles/Silver_77_Quests/Silver_77_Quests.json` - конфиг квестов и триггеров
- `profiles/Silver_77_Quests/players/[SteamID].json` - прогресс каждого игрока
