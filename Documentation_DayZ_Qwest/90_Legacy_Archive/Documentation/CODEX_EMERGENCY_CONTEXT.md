# Codex Emergency Context

Дата: 20.04.2026
Проект: `D:\Dayz\Silver_77_Quests`

Актуальный срез после последних правок: `Documentation/CODEX_CONTEXT_2026-04-20.md`. Читать его первым, потому что ниже есть исторические детали, которые могут быть старее текущего split-состояния.

## Коротко

`Silver_77_Quests` - мод квестов для DayZ.

Система уже умеет:

- загружать квесты/триггеры из серверного JSON;
- создавать дефолтный JSON при первом запуске;
- синхронизировать конфиг и прогресс с клиента через RPC;
- открывать меню NPC по `F` в зоне триггера;
- открывать журнал активных квестов по `J`;
- принимать и сдавать item-квесты;
- сдавать накопительные item-квесты частями через `allowPartialTurnIn`;
- выдавать стартовые предметы и награды;
- удалять предметы-цели при сдаче;
- хранить прогресс игроков в `$profile`.

## Главные файлы

- `config.cpp` - подключает `3_Game`, `4_World`, `5_Mission`.
- `scripts/3_Game/QuestData.c` - структуры квестов, целей, наград, триггеров.
- `scripts/3_Game/PlayerQuestData.c` - структура прогресса игрока.
- `scripts/4_World/QuestManager.c` - загрузка JSON, логика квестов, награды, прогресс, RPC sync helpers.
- `scripts/4_World/QuestPlayerRPC.c` - RPC обработка клиента/сервера.
- `scripts/5_Mission/QuestTrigger.c` - триггерные зоны NPC.
- `scripts/5_Mission/QuestUI.c` - меню NPC по `F`.
- `scripts/5_Mission/QuestJournalUI.c` - журнал активных квестов по `J`.
- `scripts/5_Mission/mission/MissionGameplay.c` - клиентская интеграция, hotkeys, hint, открытие меню.
- `gui/QuestMenu.layout` - layout меню NPC.
- `gui/QuestJournal.layout` - layout журнала.
- `gui/layouts/QuestHint.layout` - подсказка у NPC.
- `Documentation/STARTER_QUEST_CONFIG.json` - стартовый пример серверного конфига.
- `Documentation/CODEX_WORKLOG.md` - длинная история работы.
- `Documentation/CODEX_CONTROL_CONTEXT.md` - контрольный контекст для продолжения после перезапуска.
- `Documentation/SPLIT_CLIENT_SERVER.md` - текущая схема client/server split.
- `Documentation/RUSSIAN_ENCODING.md` - почему ломается русский и как не портить кодировку.
- `build_split_mods.bat` - сборка split-модов.

## Важное поведение

Журнал по `J` намеренно НЕ блокирует движение персонажа.

В `QuestJournalUI.c` строки с `ChangeGameFocus` и `ShowUICursor` должны оставаться закомментированными, если пользователь не попросит обратное. Идея: журнал можно читать на ходу.

Журнал должен закрываться по `ESC`. Переключение активных квестов колесом мыши добавлено через `QuestJournalUI.OnMouseWheel`, но это нужно проверить в игре: без game focus колесо может не доходить до UI.

Основное меню NPC по `F` должно блокировать управление и показывать курсор. В `QuestUI.c` это делается в `OnShow/OnHide`.

## JSON

Рабочий серверный конфиг находится НЕ в папке мода, а в профиле сервера:

`profiles/Silver_77_Quests/Silver_77_Quests.json`

В коде дефолтный конфиг создается в:

`scripts/4_World/QuestManager.c` -> `CreateDefaultQuestConfig()`

Важно: если серверный JSON уже существует, изменения дефолта в PBO сами не применятся. Нужно править существующий JSON вручную или удалить его перед запуском сервера.

Текущий локальный шаблон:

`Documentation/STARTER_QUEST_CONFIG.json`

Он валиден и проверен через PowerShell `ConvertFrom-Json`.

## Текущие квесты в дефолте кода

1. `quest_fisherman_1`
   - название: `Картошечка с маслицем`
   - цели: `PotatoSeed` x20, `PleurotusMushroom` x1, `MacrolepiotaMushroom` x2, `BoletusMushroom` x1
   - все цели с `allowPartialTurnIn = true`
   - стартовый предмет: `SteakKnife` x1
   - награда: `Ammo_12gaPellets` x7
   - repeatable: true
   - cooldown: 43200

2. `quest_hunter_1`
   - название: `Рыба это вам не картошка!`
   - цель: `Carp` x6
   - цель с `allowPartialTurnIn = true`
   - стартовый предмет: `HuntingKnife`
   - награда: `Ammo_12gaPellets` x7
   - repeatable: true
   - cooldown: 43200

## Триггеры

Коля Ворон:

- id: `fisherman_trigger`
- position: `13092.814453 117.007767 13084.485352`
- radius: `1.4`
- questIds: `quest_fisherman_1`
- hint: `[F] Коля Ворон`

Рыбак Гаврила:

- id: `hunter_trigger`
- position: `13091.663086 116.755630 13088.637695`
- radius: `1.4`
- questIds: `quest_hunter_1`
- hint: `[F] Рыбак Гаврила перец`

## Split client/server

Первый split уже создан:

- client: `Silver_77_Quests_Client`;
- server: `Silver_77_Quests_Server`;
- сборка: `build_split_mods.bat`.

Старая папка `SplitMods/` оставлена как резервная копия. Для ручной сборки через Addon Builder использовать корневые папки выше.

Запуск сервера:

```bat
-mod=@Silver_77_Quests_Client
-serverMod=@Silver_77_Quests_Server
```

В Workshop публиковать только `@Silver_77_Quests_Client`. Серверный `@Silver_77_Quests_Server` не публиковать.

Монолитный корневой мод оставлен как резерв. Не запускать монолит одновременно со split.

После обрыва связи были восстановлены русские строки в split UI и серверном дефолтном конфиге. Ожидаемый серверный лог:

```text
[Silver_77_Quests] QuestServerManager initialized
```

## Русский текст

Файлы держать в UTF-8. Русский ломается, когда UTF-8 читают как Windows-1251/OEM/ANSI и потом сохраняют обратно.

Если в файле видны строки вида `Р’С‹Р±РµСЂРёС‚Рµ`, это mojibake. Проверять через:

```powershell
Get-Content -Encoding UTF8 path\to\file
```

Не переписывать русские файлы через PowerShell `>` или `Out-File` без явной UTF-8 кодировки.

## Что пользователь сообщил последним

- После изменений меню/журнал по `J` открывается.
- То, что персонаж может двигаться при открытом `J`, это хорошо и должно остаться.
- Пользователь менял квесты, названия и содержимое, вероятно в серверном JSON.
- Нужно помочь переписать стартовый JSON для настройки квестов, но текущий серверный `profiles/Silver_77_Quests/Silver_77_Quests.json` пока не предоставлен.

## Что делать после перезапуска

1. Прочитать этот файл.
2. Для продолжения завтрашней работы открыть `Documentation/CODEX_CONTROL_CONTEXT.md`.
3. Если задача про историю, открыть `Documentation/CODEX_WORKLOG.md`.
4. Если задача про квесты/JSON, открыть:
   - `scripts/4_World/QuestManager.c`
   - `Documentation/STARTER_QUEST_CONFIG.json`
   - серверный `profiles/Silver_77_Quests/Silver_77_Quests.json`, если пользователь даст путь или содержимое.
5. Если задача про `J`, открыть `scripts/5_Mission/QuestJournalUI.c` и помнить, что движение при открытом журнале нужно сохранить.
6. Если задача про `F`/NPC, открыть:
   - `scripts/5_Mission/mission/MissionGameplay.c`
   - `scripts/5_Mission/QuestUI.c`
   - `scripts/5_Mission/QuestTrigger.c`
7. Если задача про защиту мода, читать `Documentation/SPLIT_CLIENT_SERVER.md`: первый split уже создан, дальше нужно собрать и проверить PBO.

## Не ломать

- Не включать блокировку движения для `J` без прямой просьбы.
- Не убирать RPC путь принятия/сдачи квеста: клиент должен отправлять запрос, сервер выполнять.
- Не рассчитывать, что дефолтный JSON перезапишет существующий серверный конфиг.
- Для рыбы/мяса `useItemQuantity = false`, чтобы предметы считались штуками и удалялись целиком.
- Накопительная сдача включается только через `allowPartialTurnIn = true`; старые квесты должны оставаться в обычном режиме.
- При изменении trigger `questIds` помнить: меню NPC показывает только квесты текущего триггера. Пустой список работает как fallback и показывает все.
- Не публиковать `@Silver_77_Quests_Server` в Workshop.
- Не сохранять файлы с русскими строками в ANSI/Windows-1251.

## Быстрая проверка

После сборки PBO в игре проверить:

1. Подойти к NPC.
2. Увидеть подсказку `[F] ...`.
3. Нажать `F`, открыть меню NPC.
4. Взять квест.
5. Нажать `J`, увидеть активный квест в журнале.
6. При открытом `J` персонаж должен продолжать двигаться.
7. Собрать предметы.
8. Вернуться к NPC и сдать квест.
