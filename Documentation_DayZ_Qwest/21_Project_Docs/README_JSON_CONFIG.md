# Silver_77_Quests - Настройка через JSON

## Структура файлов

После первого запуска сервера создастся:
```
profiles/
└── Silver_77_Quests/
    ├── Silver_77_Quests.json    (главный конфиг - квесты и триггеры)
    └── players/
        └── [SteamID].json       (прогресс каждого игрока)
```

## Главный конфиг: Silver_77_Quests.json

Весь мод настраивается через один JSON файл!

### Структура файла:

```json
{
  "version": 2,
  "quests": [
    {
      "id": "quest_fisherman_1",
      "name": "Помощь рыбаку",
      "description": "Рыбак у озера просит принести ему 3 карпа для ужина.",
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
  ],
  "triggers": [
    {
      "id": "fisherman_trigger",
      "position": [13092.814453, 117.007767, 13084.485352],
      "radius": 2.0,
      "focusHeight": 1.2,
      "focusRadius": 1.0,
      "questIds": ["quest_fisherman_1"],
      "hintText": "[F] Коля Ворон",
      "spawnNpc": true,
      "npcClassName": "SurvivorM_Mirek",
      "npcPosition": [13092.814453, 117.007767, 13084.485352],
      "npcOrientation": [215.0, 0.0, 0.0],
      "npcLoadout": [
        "FlatCap_BrownCheck",
        "HuntingJacket_Brown",
        "Jeans_Blue",
        "WorkingGloves_Brown",
        "HikingBootsLow_Black"
      ],
      "npcHandsItem": "FarmingHoe",
      "npcBackItems": ["HuntingBag", "Izh43Shotgun"]
    }
  ]
}
```

## Параметры квеста

- **id**: Уникальный идентификатор квеста
- **name**: Название квеста (отображается в UI)
- **description**: Описание задания
- **repeatable**: `true` = можно повторять, `false` = только один раз
- **cooldownSeconds**: Время до повторного взятия (в секундах)
  - `0` = без задержки
  - `3600` = 1 час
  - `86400` = 24 часа
- **requiresPrevious**: ID квеста, который нужно выполнить перед этим (или `""`)
- **requiredQuestIds**: список ID квестов, которые все должны быть выполнены перед взятием этого квеста. Для квеста без условий указывай `[]`

### giveItems - Предметы при взятии квеста

```json
"giveItems": [
  {
    "className": "TaloonBag_Blue",
    "quantity": 1,
    "spawnOnGround": false
  }
]
```
- **className**: Класс предмета из DayZ
- **quantity**: Количество
- **spawnOnGround**: `false` = в инвентарь, `true` = на землю

### objectives - Цели квеста

```json
"objectives": [
  {
    "type": "item",
    "className": "Carp",
    "quantity": 3,
    "removeOnComplete": true,
    "useItemQuantity": false,
    "allowPartialTurnIn": false
  }
]
```
- **type**: Тип цели (пока только `"item"`)
- **className**: Класс предмета
- **quantity**: Сколько нужно
- **removeOnComplete**: Удалить предметы при сдаче
- **useItemQuantity**: Как считать предметы
  - `false` = считать каждый предмет / стак как 1 и удалять целые предметы. Для рыбы, мяса, банок, инструментов.
  - `true` = считать внутреннюю quantity предмета. Для патронов, денег в стаках, `Firewood`, `WoodenStick`, жидкостей и других количественных предметов.
- **allowPartialTurnIn**: Можно ли сдавать цель частями
  - `false` = старое поведение: для сдачи квеста нужно принести все количество сразу.
  - `true` = накопительное поведение: предметы можно вносить частями, прогресс сохраняется в файле игрока.

### Пример накопительной цели

```json
{
  "type": "item",
  "className": "Potato",
  "quantity": 50,
  "removeOnComplete": true,
  "useItemQuantity": false,
  "allowPartialTurnIn": true
}
```

Такой квест можно сдавать партиями: например 10 картошек сейчас, потом 20 и еще 20. Награда выдастся только когда накоплено `50 / 50`.

### rewards - Награды

```json
"rewards": [
  {
    "className": "FishingRod",
    "quantity": 1
  }
]
```

## Параметры триггера

```json
"triggers": [
  {
    "id": "fisherman_trigger",
    "position": [13092.814453, 117.007767, 13084.485352],
    "radius": 2.0,
    "focusHeight": 1.2,
    "focusRadius": 1.0,
    "questIds": ["quest_fisherman_1"],
    "hintText": "[F] Коля Ворон",
    "spawnNpc": true,
    "npcClassName": "SurvivorM_Mirek",
    "npcPosition": [13092.814453, 117.007767, 13084.485352],
    "npcOrientation": [215.0, 0.0, 0.0],
    "npcLoadout": [
      "FlatCap_BrownCheck",
      "HuntingJacket_Brown",
      "Jeans_Blue",
      "WorkingGloves_Brown",
      "HikingBootsLow_Black"
    ],
    "npcHandsItem": "FarmingHoe",
    "npcBackItems": ["HuntingBag", "Izh43Shotgun"]
  }
]
```

- **id**: Уникальный ID триггера
- **position**: Координаты [X, Y, Z]
- **radius**: Радиус зоны в метрах
- **focusHeight**: Высота точки наведения над `position` в метрах. Для груди/пояса NPC обычно `1.2`-`1.6`.
- **focusRadius**: Радиус виртуальной точки наведения в метрах. Меньше = строже, больше = легче поймать подсказку.
- **questIds**: Массив ID квестов, доступных в этой точке
- **hintText**: Текст подсказки у точки взаимодействия
- **spawnNpc**: `true` = сервер заспавнит видимого персонажа на этой точке
- **npcClassName**: Класс ванильного персонажа, например `SurvivorM_Mirek`
- **npcPosition**: Координаты NPC. Если массив пустой, используется `position` триггера. Когда NPC включён и `npcPosition` заполнен, клиентская точка наведения тоже берётся от NPC
- **npcOrientation**: Поворот NPC `[yaw, pitch, roll]`
- **npcLoadout**: Одежда и снаряжение, которые надеваются через attachment-слоты
- **npcHandsItem**: Предмет в руках NPC
- **npcBackItems**: Предметы за плечами/на спине, например рюкзак или длинное оружие

### Как узнать координаты?

1. Зайди в игру
2. Встань в нужном месте
3. Открой консоль (`) и введи: `getpos`
4. Скопируй координаты X Y Z в формат: `[X, Y, Z]`

## Добавление нового квеста

1. Открой `profiles/Silver_77_Quests/Silver_77_Quests.json`
2. Добавь новый квест в массив `"quests"`
3. Добавь триггер в массив `"triggers"`
4. Перезапусти сервер

## Пример: Добавить квест торговца

```json
{
  "id": "quest_trader_1",
  "name": "Поставка для торговца",
  "description": "Торговец просит принести 10 консервов.",
  "repeatable": true,
  "cooldownSeconds": 7200,
  "requiresPrevious": "",
  "requiredQuestIds": [],
  "giveItems": [],
  "objectives": [
    {
      "type": "item",
      "className": "BakedBeansCan",
      "quantity": 10,
      "removeOnComplete": true,
      "useItemQuantity": false
    }
  ],
  "rewards": [
    {
      "className": "Money",
      "quantity": 1000
    }
  ]
}
```

И триггер:
```json
{
  "id": "trader_trigger",
  "position": [5000, 0, 5000],
  "radius": 5.0,
  "focusHeight": 1.2,
  "focusRadius": 1.0,
  "questIds": ["quest_trader_1"],
  "hintText": "[F] Поговорить",
  "spawnNpc": true,
  "npcClassName": "SurvivorM_Boris",
  "npcPosition": [5000, 0, 5000],
  "npcOrientation": [90.0, 0.0, 0.0],
  "npcLoadout": ["BeanieHat_Green", "Raincoat_Green", "Jeans_Blue", "HikingBoots_Black"],
  "npcHandsItem": "FishingRod",
  "npcBackItems": ["DryBag_Green", "Izh43Shotgun"]
}
```

## Пример: Последовательность квестов

Если квест должен открыться только после нескольких других квестов, добавь их ID в `requiredQuestIds`:

```json
{
  "id": "quest_medic_final",
  "name": "Большая просьба медика",
  "description": "Медик доверит это задание только после всей предыдущей цепочки.",
  "repeatable": false,
  "cooldownSeconds": 0,
  "requiresPrevious": "",
  "requiredQuestIds": [
    "quest_medic_1",
    "quest_medic_2",
    "quest_medic_3",
    "quest_hunter_medic_1"
  ],
  "giveItems": [],
  "objectives": [],
  "rewards": []
}
```

Все квесты из `requiredQuestIds` должны быть выполнены хотя бы один раз. Если хотя бы один ещё не выполнен, сервер не даст взять квест, а в меню будет показано, какие условия ещё не закрыты. Для повторяемых квестов повторное взятие не сбрасывает доступ к следующим квестам, если этот квест уже когда-то был завершён.

## Горячая перезагрузка

После изменения JSON файла нужно перезапустить сервер.
