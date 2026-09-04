# Codex Context Snapshot 2026-04-20

Перед чтением этого файла сначала открыть:

- `Documentation/CODEX_START_HERE.md`

Дата: 20.04.2026

Проект: `D:\Dayz\Silver_77_Quests`

## Как работать с пользователем

- Отвечать по-русски.
- Беречь русский текст и кодировку: файлы держать в UTF-8, не сохранять как ANSI/Windows-1251/OEM.
- Пользователь сам собирает PBO и сам заменяет моды на сервере. Codex не запускает сборку, публикацию и замену без прямой новой просьбы.
- Не перетирать существующий серверный JSON. Если рабочий `Silver_77_Quests.json` уже есть, править его вручную или удалить для пересоздания.

## Текущая архитектура

Используется split:

- клиентский мод: `Silver_77_Quests_Client`;
- серверный приватный мод: `Silver_77_Quests_Server`;
- старый `SplitMods/` и корневой монолитный мод считать резервом, не основным рабочим путем.

Запуск сервера:

```bat
-mod=@Silver_77_Quests_Client
-serverMod=@Silver_77_Quests_Server
```

Workshop: публиковать только клиентский мод. Серверный `@Silver_77_Quests_Server` не публиковать.

## Сборка

Главное для Addon Builder:

- client source: `D:\Dayz\Silver_77_Quests\Silver_77_Quests_Client`;
- client prefix: `Silver_77_Quests`;
- server source: `D:\Dayz\Silver_77_Quests\Silver_77_Quests_Server`;
- server prefix: `Silver_77_Quests_Server`;
- server destination: `D:\Dayz\Mods_DONE\@Silver_77_Quests_Server\addons`.

Если серверный PBO собрать с prefix `Silver_77_Quests` вместо `Silver_77_Quests_Server`, серверные скрипты могут не подхватиться. Симптом: PBO собран успешно, но в RPT нет `MissionServer.OnInit called`, нет `QuestServerManager initialized`, рабочий JSON не создается.

`build_split_mods.bat` уже поправлен: Addon Builder получает `-exclude="%ROOT_DIR%exclude.lst"`.

## Рабочий JSON

Путь:

```text
profiles\Silver_77_Quests\Silver_77_Quests.json
```

В коде:

- `Silver_77_Quests_Server/scripts/4_World/QuestServerManager.c`;
- `SILVER77_QUEST_CONFIG_VERSION = 2`;
- `SILVER77_QUEST_CONFIG_PATH = "$profile:Silver_77_Quests/Silver_77_Quests.json"`;
- `CreateDefaultQuestConfig()` создает дефолт;
- `LoadQuestConfig()` создает файл, если его нет;
- `Silver77_MigrateQuestConfig()` добавляет NPC-настройки в старые конфиги версии ниже 2;
- `Silver77_SaveQuestConfigFile()` пишет JSON через `JsonSerializer + OpenFile` и логирует ошибки;
- `EnsureServerQuestConfigLoaded()` страхует случай, когда объект конфига есть, а файла в профиле нет.

Ожидаемые RPT-строки при нормальном старте:

```text
[Silver_77_Quests] MissionServer.OnInit called
[Silver_77_Quests] Loading quest config from: $profile:Silver_77_Quests/Silver_77_Quests.json
[Silver_77_Quests] Config not found, creating default...
[Silver_77_Quests] Config created at: $profile:Silver_77_Quests/Silver_77_Quests.json
[Silver_77_Quests] QuestServerManager initialized
```

## Дефолтные квесты

`quest_fisherman_1`:

- имя: `Картошечка с маслицем`;
- стартовый предмет: `SteakKnife` x1;
- цели: `PotatoSeed` x50, `PleurotusMushroom` x1, `MacrolepiotaMushroom` x2, `BoletusMushroom` x1;
- все цели с `allowPartialTurnIn = true`;
- награда: `Ammo_12gaPellets` x100;
- repeatable: `true`;
- cooldown: `43200`.

`quest_hunter_1`:

- имя: `Рыба это вам не картошка!`;
- стартовый предмет: `HuntingKnife` x1;
- цель: `Carp` x6;
- цель с `allowPartialTurnIn = true`;
- награда: `Ammo_12gaPellets` x7;
- repeatable: `true`;
- cooldown: `43200`.

## Дефолтные NPC

Коля Ворон:

- trigger id: `fisherman_trigger`;
- questIds: `quest_fisherman_1`;
- position/npcPosition: `13092.814453, 117.007767, 13084.485352`;
- radius: `2.0`;
- focusHeight: `1.2`;
- focusRadius: `1.0`;
- hint: `[F] Коля Ворон`;
- npcClassName: `SurvivorM_Mirek`;
- npcOrientation: `[215.0, 0.0, 0.0]`;
- npcLoadout: `FlatCap_BrownCheck`, `HuntingJacket_Brown`, `Jeans_Blue`, `WorkingGloves_Brown`, `HikingBootsLow_Black`;
- npcHandsItem: `FarmingHoe`;
- npcBackItems: `HuntingBag`, `Izh43Shotgun`.

Рыбак Гаврила:

- trigger id: `hunter_trigger`;
- questIds: `quest_hunter_1`;
- position/npcPosition: `13091.663086, 116.755630, 13088.637695`;
- radius: `2.0`;
- focusHeight: `1.2`;
- focusRadius: `1.0`;
- hint: `[F] Рыбак Гаврила перец`;
- npcClassName: `SurvivorM_Boris`;
- npcOrientation: `[300.0, 0.0, 0.0]`;
- npcLoadout: `BeanieHat_Green`, `Raincoat_Green`, `HunterPants_Summer`, `WorkingGloves_Black`, `Wellies_Green`;
- npcHandsItem: `FishingRod`;
- npcBackItems: `DryBag_Green`, `Izh43Shotgun`.

Пользователь проверил в игре: NPC спавнятся, одежда и предметы работают. Охотник визуально ориентирован удачно. Рыбак был довернут с `35.0` на `215.0`; это нужно проверить после следующей сборки/обновления.

## Что уже реализовано

- Серверная авторитетная обработка принятия и сдачи квестов через RPC.
- Синхронизация серверного конфига и прогресса игрока на клиент.
- Сохранение прогресса по SteamID в `$profile`.
- Частичная сдача предметов через `allowPartialTurnIn`.
- Проверка предыдущих квестов через `requiresPrevious` и `requiredQuestIds`.
- Cooldown повторяемых квестов.
- Выдача стартовых предметов и наград; если не помещается, предмет появляется рядом с игроком.
- Удаление предметов-целей при сдаче.
- Видимые серверные NPC на точках квестов.
- Настраиваемые `radius`, `focusHeight`, `focusRadius`, `npcPosition`, `npcOrientation`, `npcLoadout`, `npcHandsItem`, `npcBackItems`.
- Подсказка и открытие меню по `F` завязаны на наведение на виртуальную точку NPC.
- Журнал квестов по `J`; движение игрока при открытом журнале сохранять.

## Что не забыть

- Если после удаления JSON в профиле файл не создается, сначала проверить server Addon prefix и RPT.
- Если в RPT нет строк `[Silver_77_Quests]`, почти наверняка серверный PBO не подхватился или собран с неверным prefix.
- Если есть `OpenFile WRITE failed`, смотреть права/путь профиля.
- При изменении дефолта помнить: существующий рабочий JSON сам не обновится.
- При задачах по квестам/JSON читать `Documentation/STARTER_QUEST_CONFIG.json` и `Documentation/README_JSON_CONFIG.md`.
- При задачах по сборке читать `Documentation/BUILD.md` и `Documentation/SPLIT_CLIENT_SERVER.md`.

## Актуализация 25.04.2026

### Новый минимальный дефолт сервера

Базовым стартовым набором теперь считаются 4 квеста:

- `quest_hunter_1` — `Картошечка с маслицем`;
- `quest_fisherman_1` — `Рыба это вам не картошка!`;
- `quest_Rasputin_1` — `Взаимовыручка прежде всего!`;
- `quest_fisherman_2` — `Поставка медицины`.

Базовые триггеры теперь такие:

- `hunter_trigger` -> `quest_hunter_1`;
- `fisherman_trigger` -> `quest_fisherman_2`, `quest_fisherman_1`;
- `Rasputin_1_trigger` -> `quest_Rasputin_1`.

Это уже внесено в:

- `Silver_77_Quests_Server/scripts/4_World/QuestServerManager.c`;
- `SplitMods/Silver_77_Quests_Server/scripts/4_World/QuestServerManager.c`;
- `scripts/4_World/QuestManager.c`;
- `Documentation/STARTER_QUEST_CONFIG.json`.

### Что важно для пересоздания на сервере

- Источник истины в рантайме по-прежнему: `profiles/Silver_77_Quests/Silver_77_Quests.json`.
- Новый дефолт будет создан только если этого файла нет.
- Текущий согласованный план пользователя: удалить профильный JSON и дать серверу создать его заново.

### Что нужно пересобирать

Для split-схемы с отдельными `Silver_77_Quests_Client` и `Silver_77_Quests_Server` под этот апдейт нужен только server rebuild/update:

- клиентский мод не менялся;
- серверный код дефолтной генерации менялся.

### Состояние редактора / парсера

Рабочая мастерская для JSON сейчас:

```text
M:\GITS_VERSE\Neyro_01\Sborka_Json\JSON_Quvest
```

Там:

- новый минимальный набор уже лежит как дефолтный `Silver_77_Quests.json`;
- редактор синхронизирован с этим набором;
- добавлена отдельная заготовка под справочник стеков через `item-stack-rules.json` и `/api/stack-rules`.

Важно: справочник стеков пока только хранит ручные правила и UI для их ввода. Автоматическое преобразование "наборов" в рабочее `quantity` в квестовых objective еще не подключено.
