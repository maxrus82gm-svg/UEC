# Split Client/Server Build

Дата: 18.04.2026

## Что это

Подготовлена первая split-версия мода:

- публичный клиентский мод: `@Silver_77_Quests_Client`;
- приватный серверный мод: `@Silver_77_Quests_Server`.

Цель первого этапа: не публиковать серверную логику, JSON-загрузку, выдачу наград, удаление предметов, cooldown и сохранение прогресса.

## Где исходники

Клиент:

`Silver_77_Quests_Client`

Сервер:

`Silver_77_Quests_Server`

Монолитная версия в корне проекта пока оставлена как рабочая резервная копия.
Старая папка `SplitMods/` оставлена только как резервная копия. Основная понятная структура теперь лежит прямо в корне проекта:

```text
Silver_77_Quests/
├── Silver_77_Quests_Client/
├── Silver_77_Quests_Server/
├── Documentation/
├── scripts/      старый монолитный резерв
└── gui/          старый монолитный резерв
```

## Что лежит в клиентском моде

- общие DTO-структуры:
  - `scripts/3_Game/QuestData.c`
  - `scripts/3_Game/PlayerQuestData.c`
- клиентское состояние и RPC-запросы:
  - `scripts/4_World/QuestClientManager.c`
  - `scripts/4_World/QuestClientRPC.c`
- UI и клиентская интеграция:
  - `scripts/5_Mission/QuestUI.c`
  - `scripts/5_Mission/QuestJournalUI.c`
  - `scripts/5_Mission/QuestTrigger.c`
  - `scripts/5_Mission/mission/MissionGameplay.c`
- layouts:
  - `gui/QuestMenu.layout`
  - `gui/QuestJournal.layout`
  - `gui/layouts/QuestHint.layout`

Клиентский мод можно публиковать в Workshop.

## Что лежит в серверном моде

- приватная серверная логика:
  - `scripts/4_World/QuestServerManager.c`
  - `scripts/4_World/QuestServerRPC.c`
  - `scripts/5_Mission/mission/MissionServer.c`

Серверный мод не публиковать. Он должен лежать только на сервере.

## Как собрать

Использовать:

```bat
build_split_mods.bat
```

Скрипт собирает:

```text
..\Mods_DONE\@Silver_77_Quests_Client
..\Mods_DONE\@Silver_77_Quests_Server
```

Если `AddonBuilder.exe` лежит в другом месте, изменить переменную `DAYZ_TOOLS` внутри `build_split_mods.bat`.

Для ручной сборки через DayZ Addon Builder:

```text
Клиентский PBO:
Addon source directory: D:\Dayz\Silver_77_Quests\Silver_77_Quests_Client
Destination directory:  D:\Dayz\Mods_DONE\@Silver_77_Quests_Client\addons
Addon prefix:           Silver_77_Quests

Серверный PBO:
Addon source directory: D:\Dayz\Silver_77_Quests\Silver_77_Quests_Server
Destination directory:  D:\Dayz\Mods_DONE\@Silver_77_Quests_Server\addons
Addon prefix:           Silver_77_Quests_Server
```

## Как запускать сервер

Пример:

```bat
-mod=@Silver_77_Quests_Client
-serverMod=@Silver_77_Quests_Server
```

В Workshop публикуется только:

```text
@Silver_77_Quests_Client
```

## Важные технические решения

1. Общие структуры лежат в клиентском моде.

Серверный мод зависит от клиентского CfgPatches:

```cpp
requiredAddons[] = {"DZ_Data", "DZ_Scripts", "Silver_77_Quests_Client"};
```

Так мы избегаем дубля классов `Silver77_QuestConfig`, `PlayerQuestData` и т.п. при загрузке сервером одновременно `-mod` и `-serverMod`.

2. Клиентский менеджер называется `QuestClientManager`.

Он только:

- хранит синхронизированный конфиг и прогресс;
- обновляет UI;
- отправляет RPC-запросы на сервер.

Он не выдает награды и не удаляет предметы.

3. Серверный менеджер называется `QuestServerManager`.

Он:

- загружает серверный JSON;
- принимает/сдает квесты;
- проверяет предметы;
- удаляет цели;
- выдает награды;
- сохраняет прогресс;
- отправляет клиенту синхронизацию.

4. На первом этапе клиент все еще получает синхронизированный конфиг.

Это быстрее и безопаснее для первого split-теста. Второй этап защиты: убрать полный конфиг с клиента и перенести проверку зон NPC на сервер.

## Что проверить в игре

1. Сервер стартует с client+server split-модами.
2. Клиент подключается только с `@Silver_77_Quests_Client`.
3. Серверный `@Silver_77_Quests_Server` не требуется клиенту.
4. В логах сервера есть:
   - `[Silver_77_Quests] MissionServer.OnInit called`
   - `[Silver_77_Quests] QuestServerManager initialized`
5. Клиент получает конфиг и прогресс.
6. Подсказка возле NPC появляется.
7. `F` открывает меню NPC.
8. `J` открывает журнал.
9. `ESC` закрывает журнал.
10. Накопительная сдача работает.

## Риски первого теста

1. Возможная ошибка сигнатуры:

```c
override bool OnMouseWheel(Widget w, int x, int y, int wheel)
```

Если DayZ ожидает другую сигнатуру, исправить в `QuestJournalUI.c`.

2. Возможная ошибка константы:

```c
KeyCode.KC_ESCAPE
```

Если имя другое, исправить в `MissionGameplay.c`.

3. Если серверный PBO не видит общие структуры, проверить:

- загружен ли клиентский мод в `-mod`;
- есть ли `Silver_77_Quests_Client` в `requiredAddons[]` серверного `config.cpp`;
- не запущен ли старый монолитный `@Silver_77_Quests` одновременно со split-версией.

## Что делать дальше

После успешного первого split:

1. убрать полный конфиг с клиента;
2. перенести проверку NPC-зон на сервер;
3. отправлять клиенту только текущую подсказку и доступные квесты;
4. оставить координаты, полный список квестов и награды только на сервере.
