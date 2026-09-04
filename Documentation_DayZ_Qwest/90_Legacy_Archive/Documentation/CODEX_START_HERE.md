# CODEX START HERE

## Editor Root Rule

- Каноническая рабочая папка редактора: `P:\Silver_77_Quests\Support\JSON_Quvest`.
- Legacy-папка `P:\Silver_77_Quests\JSON_Quvest` больше не считается отдельной живой копией редактора.
- Все относительные пути редактора считаются от папки реально запущенного `server.ps1`, а не от текущего `cwd`.
- Это касается `savePath`, `backupPath`, `editor-draft.json`, `item-stack-rules.json` и статики редактора.
- Текущий рабочий JSON редактора: `Silver_77_Quests.json`.
- Текущий backup рядом с ним: `Silver_77_Quests_BackUP.json`.
- Рабочие дефолтные пути сохранения редактора зафиксированы явно:
  - `P:\Silver_77_Quests\JSON_Quvest\Silver_77_Quests.json`
  - `P:\Silver_77_Quests\JSON_Quvest\Silver_77_Quests_BackUP.json`
- Даже если launcher идёт через `P:\Silver_77_Quests\Support\JSON_Quvest`, сохранять по умолчанию нужно именно в корень `P:\Silver_77_Quests\JSON_Quvest`, где лежит bat-обёртка.
- Если запуск идёт из `P:\Silver_77_Quests\JSON_Quvest`, это только совместимая обёртка на `Support\JSON_Quvest`.
- Если поведение редактора выглядит как “подмешалась старая версия”, сначала проверить активный editor root, а уже потом quest logic.

Это первая точка входа для нового Codex-агента по проекту `Silver_77_Quests`.

Если задача касается мода, квестов, JSON-конфига, NPC, trigger, split client/server или редактора квестов, сначала прочитать этот файл, а потом указанные ниже документы.

## Обязательный порядок чтения

1. `P:\Silver_77_Quests\Documentation\CODEX_START_HERE.md`
2. `P:\Silver_77_Quests\Documentation\CODEX_CONTROL_CONTEXT.md`
3. `P:\Silver_77_Quests\Documentation\CODEX_CONTEXT_2026-04-20.md`
4. `P:\Silver_77_Quests\Documentation\CODEX_WORKLOG.md`
5. `P:\Silver_77_Quests\Documentation\CODEX_SESSION_HANDOFF_2026-04-29.md`
6. `P:\Silver_77_Quests\Documentation\CODEX_SESSION_HANDOFF_2026-04-26.md`
7. `P:\Silver_77_Quests\Documentation\CODEX_SESSION_HANDOFF_2026-04-25.md` - только как предыдущий handoff, если нужно понять, с чего стартовала эта ветка.

После чтения нужно коротко подтвердить, что контекст принят, и только потом предлагать правки или делать изменения.

## Быстрый вход при дефиците контекста

Если новая сессия короткая и нужно максимально быстро войти в текущую точку проекта, допускается укороченный маршрут:

1. `P:\Silver_77_Quests\Documentation\CODEX_START_HERE.md`
2. `P:\Silver_77_Quests\Documentation\CODEX_FAST_HANDOFF_2026-04-27.md`

А уже потом, только при необходимости углубления:

3. `P:\Silver_77_Quests\Documentation\CODEX_SESSION_HANDOFF_2026-04-29.md`
4. `P:\Silver_77_Quests\Documentation\CODEX_CONTROL_CONTEXT.md`
5. `P:\Silver_77_Quests\Documentation\CODEX_SESSION_HANDOFF_2026-04-26.md`
6. `P:\Silver_77_Quests\Documentation\CODEX_WORKLOG.md`

## Текущая рабочая схема

- Основной корень проекта: `P:\Silver_77_Quests`
- В этом же корне лежат:
  - клиентский мод: `P:\Silver_77_Quests\Silver_77_Quests_Client`
  - серверный мод: `P:\Silver_77_Quests\Silver_77_Quests_Server`
  - legacy-монолит: `P:\Silver_77_Quests\scripts`
  - редактор квестов: `P:\Silver_77_Quests\Support\JSON_Quvest`
- Основной рабочей схемой считать split client/server.
- Legacy `scripts/` не удалять и не ломать без прямой причины.
- Корень `P:\Silver_77_Quests` в этой рабочей схеме не является git-репозиторием.
  - Не тратить стартовые шаги на `git status`, ветки, staging или commit-поток.
  - Считать проект обычной рабочей папкой с прямой правкой файлов.
- Для этой ветки обязателен режим маленьких законченных шагов.
  - Не тащить большой рефактор одним патчем, если его можно разбить на несколько завершённых подзадач.
  - После каждого небольшого слоя оставлять короткий checkpoint в handoff / start, если сессия длинная или рискованная.
- Русский текст и корректная UTF-8 кодировка являются обязательной частью проекта.
  - Любые строки вида `Р...`, `С...`, `Ð...` и прочая кракозябра считаются багом, а не косметикой.
  - При правках нельзя копировать битый русский текст из сломанного терминального вывода.

## Что считать актуальным прямо сейчас

- Новый минимум стартовых квестов уже внесен в серверный дефолт мода.
- Runtime JSON сервера по-прежнему живет в:
  - `profiles\Silver_77_Quests\Silver_77_Quests.json`
- Если этот профильный файл уже существует, изменения дефолта в коде сами не применятся.
- Редактор квестов теперь считается частью того же проекта на `P:`.
- Запускатор редактора уже починен:
  - основной путь: `P:\Silver_77_Quests\Support\JSON_Quvest`
  - старые запускаторы в `P:\Silver_77_Quests\JSON_Quvest` теперь работают как обертка и должны вести в актуальную папку
- При этом дефолтный saved JSON для текущей рабочей ветки теперь уже принят как:
  - `P:\Silver_77_Quests\JSON_Quvest\Silver_77_Quests.json`
  - текущий актуальный baseline: `version = 3`, 5 квестов, включая `quest_hunter_2`
- В редакторе уже работает фильтр квестов по trigger / NPC в левой колонке.
- Основная модель квестов больше не строится вокруг отдельной ручной `видимости`.
  - `trigger.questIds` = список и порядок квестов у NPC
  - игровая видимость считается по ролям и статусу квеста
  - роли: `offerTriggerIds`, `completionTriggerIds`, `rewardTriggerIds`
- Для offer / completion / reward уже заложен новый каркас `triggerActions`.
  - у действия есть `triggerId`, `actionType`, `dialogText`, `rewards`
  - `offer` хранит отдельный диалог NPC при взятии
  - `completion` хранит отдельный диалог и локальную награду промежуточного этапа
  - `reward` хранит отдельный диалог и финальную награду
- Важно для следующей сессии:
  - старую пользовательскую совместимость можно не сохранять
  - при необходимости допустимо пересоздать profile JSON и историю персонажей
  - но дефолтный набор квестов на базе уже известных старых квестов должен оставаться по смыслу

## Текущее состояние после последних правок

- Запуск редактора уже должен работать.
- В редакторе основной сценарий уже переведен на role-driven модель:
  - где взять
  - кому сдать
  - где получить награду
- Для `offer`, `completion` и `reward` уже есть отдельные action-карточки с диалогом по конкретному NPC.
- Новая базовая логика квеста:
  - `Offer` обязателен: где игрок берёт квест.
  - `Reward` обязателен: финальная точка, которая закрывает квест и выдаёт награду.
  - `Completion` опционален: один или несколько промежуточных этапов передачи / сдачи.
  - Если есть хотя бы один `Completion`, `Reward` не должен закрывать квест, пока все Completion-trigger не отмечены выполненными.
  - Если `Completion` нет, `Reward` принимает цели напрямую и закрывает квест.
- На клиенте и сервере уже заложена логика `reward_pending` и перехода к отдельному reward trigger.
- В split client/server добавлено сохранение выполненных Completion-trigger в прогрессе игрока через `completedCompletionTriggerIds`.
- Старый дефолтный список квестов не должен пропасть при миграции:
  - если у старого JSON нет ролей, они досеиваются из `trigger.questIds`
- Отдельные старые глобальные поля `turnInDialogText` / `rewardDialogText` больше не считать основной моделью.

## Update 2026-04-29 - жесткая роль на блок

- Последняя продуктовая фиксация по `NPC Flow`: один блок = одна роль.
- Нельзя смешивать `Offer + Reward` в одной карточке.
- Нельзя создавать два `Offer` у одного квеста.
- `Reward` тоже должен быть уникальным и обязательным.
- `Completion` может быть несколько.
- Один и тот же NPC / trigger может повторяться в квесте, но только как другой отдельный блок с другой ролью.
- `Offer`:
  - обязателен
  - несет главный диалог и стартовые требования
  - может выдавать несколько предметов
  - не является местом финальной награды
- `Completion`:
  - опционален
  - имеет собственный диалог
  - может принимать предметы
  - может выдавать локальную награду
  - не может закрывать квест
- `Reward`:
  - обязателен
  - является единственной финальной точкой закрытия
  - выдает финальную награду
  - не имеет права закрыть квест, пока не выполнены требования `Offer` и все `Completion`
- Пользователь явно разрешил при необходимости перестраивать JSON / draft под эту схему, не цепляясь за старый формат.

## Ближайшая продуктовая задача

Следующий практический шаг после чтения контекста:

1. Сначала добить редактор до строгой схемы `один блок = одна роль`.
2. Проверить, что в редакторе физически нельзя:
   - создать второй `Offer`
   - создать второй `Reward`
   - смешать `Offer + Reward` в одной карточке
3. Проверить, что один и тот же NPC / trigger можно повторить только как другой отдельный блок с другой ролью.
4. Проверить, что `Completion` поддерживает собственный диалог и локальную награду, но не закрывает квест.
5. Проверить, что `Reward` остается единственным финальным закрывающим блоком.
6. После этого вернуться к live-проверке видимости:
   - `not_started` -> offer
   - `active` -> completion; если completion нет или все completion выполнены, показывается reward
   - `reward_pending` -> reward
7. Только потом пересобирать оба PBO: `Silver_77_Quests_Client` и `Silver_77_Quests_Server`, и уже в игре проверять маршрут `A -> B -> reward`, диалоги и желтую подсветку reward trigger.

Важно:

- эту задачу вести по небольшим законченным слоям;
- после каждого слоя фиксировать фактически выполненное, чтобы при сжатии контекста план не расползался.

## С чего начать завтра

1. Сначала заново прочитать:
   - `P:\Silver_77_Quests\Documentation\CODEX_START_HERE.md`
   - `P:\Silver_77_Quests\Documentation\CODEX_CONTROL_CONTEXT.md`
   - `P:\Silver_77_Quests\Documentation\CODEX_CONTEXT_2026-04-20.md`
   - `P:\Silver_77_Quests\Documentation\CODEX_WORKLOG.md`
   - `P:\Silver_77_Quests\Documentation\CODEX_SESSION_HANDOFF_2026-04-29.md`
2. После чтения не уходить сразу в сборку, а сначала добить редактор в `P:\Silver_77_Quests\Support\JSON_Quvest`.
3. Первый практический фокус на завтра:
   - проверить и довести `NPC Flow` до жесткого правила `один блок = одна роль`;
   - убедиться, что в редакторе нельзя сделать два `Offer` и нельзя смешать `Offer + Reward` в одной карточке;
   - убедиться, что один и тот же NPC можно повторить только отдельным блоком для другой роли;
   - собрать сценарий вида:
     - первый блок = кто выдаёт квест (`Offer`)
     - промежуточные блоки = кому несут / передают (`Completion`)
     - последний блок = кто финально закрывает квест и выдаёт награду (`Reward`)
4. Только после этого переходить к live-проверке видимости, желтой подсветки reward trigger, а затем к пересборке `Silver_77_Quests_Client` и `Silver_77_Quests_Server`.

## Что не доделано

- Редактор ещё не доведён до финальной универсальной схемы NPC-блоков.
- Редактор ещё не доведён до жесткого правила `один блок = одна роль`.
- Нужно финально проверить, что один блок физически не может содержать одновременно `Offer + Reward`.
- Нужно финально проверить, что у одного квеста не может появиться второй `Offer` и второй `Reward`.
- Нужно финально проверить, что верхний picker `Открыть / добавить NPC` не подменяет собой локальный выбор NPC внутри каждого блока.
- Нужно убедиться, что логика редактора действительно строится блоками, а не старыми глобальными секциями.
- Старые отдельные блоки `Give Items`, `Objectives`, `Rewards` больше не должны оставаться смысловым центром настройки; их поведение нужно окончательно подчинить role-driven блокам.
- Нужно подтвердить, что `Completion` действительно живет как промежуточный блок со своим диалогом и локальной наградой, но без финального закрытия квеста.
- Нужен полный практический прогон после последних правок:
  - `Offer -> Reward`
  - `Offer -> Completion -> Reward`
  - цепочка с несколькими `Completion`
- Нужна реальная проверка, что `Reward` не закрывает квест, пока не выполнены все `Completion`.
- Нужна реальная проверка диалогов по ролям:
  - `offer`
  - `completion`
  - `reward`
- Нужна пересборка обоих PBO после последних правок и проверка реального результата в игре.
- Отдельная механика "именно тот самый конкретный выданный предмет" пока не реализована и остаётся отдельной идеей на будущее.

## Что нельзя предполагать молча

- Нельзя считать `M:` основной мастерской. Основной путь - `P:\Silver_77_Quests`.
- Нельзя запускать сборку, публикацию или сервер без прямой новой просьбы пользователя.
- Пользователь сам занимается build / rebuild / publish мода.
  - Зона Codex здесь: код, JSON-схема, редактор, валидация, handoff и проверка логики.
- Нельзя считать, что старый `profiles\Silver_77_Quests\Silver_77_Quests.json` сам обновится.
- Нельзя считать, что достаточно обновить только client или только server, если менялись DTO / RPC / UI-логика.
- Нельзя молча предполагать наличие git-репозитория в `P:\Silver_77_Quests`.

## Русский и кодировка

- Для редактора квестов, документации и всех пользовательских строк русский текст считать частью рабочего контракта, а не косметикой.
- Если в UI, `app.js`, `.md` или JSON появились последовательности вида `Р...`, `С...` или другой "кракозябры", это считать ошибкой кодировки и чинить сразу.
- Если есть выбор между "быстро дописать" и "сначала убедиться, что русский текст читается нормально", сначала всегда чинить кодировку.
- Новая сессия не должна молча продолжать работу поверх битого русского текста в UI, документации или JSON.
- Русские строки нельзя бездумно копировать из терминального вывода, если сам терминал уже показывает их битым текстом.
- При правках русских строк ориентироваться на реальные UTF-8 файлы проекта:
  - `P:\Silver_77_Quests\Support\JSON_Quvest\app.js`
  - `P:\Silver_77_Quests\Support\JSON_Quvest\README.md`
  - `P:\Silver_77_Quests\Support\JSON_Quvest\PROJECT_CONTEXT.md`
  - `P:\Silver_77_Quests\Documentation\*.md`
- Если есть сомнение, сначала открыть исходный файл и убедиться, что текст там читается нормально, и только потом переносить или редактировать строки.

## Рекомендуемый стартовый ответ нового агента

Пример безопасного старта:

`Сначала прочитал CODEX_START_HERE.md, CODEX_CONTROL_CONTEXT.md, CODEX_CONTEXT_2026-04-20.md, CODEX_WORKLOG.md и CODEX_SESSION_HANDOFF_2026-04-29.md. Контекст принят. Вижу, что P:\\Silver_77_Quests не является git-репозиторием, а build/publish пользователь делает сам. Мой текущий шаг - сначала добить строгую схему NPC Flow (один блок = одна роль, уникальные Offer/Reward), и только потом передать изменения на твою ручную сборку и игровой тест.`


## Fresh Checkpoint - 2026-04-29

1. Accepted editor baseline:
   - `P:\Silver_77_Quests\JSON_Quvest\Silver_77_Quests.json`
   - `version = 3`
   - `quests = 5`
   - includes `quest_hunter_2` with the real `Offer -> Completion -> Reward` chain
2. Default save targets are now fixed in config:
   - `P:\Silver_77_Quests\JSON_Quvest\Silver_77_Quests.json`
   - `P:\Silver_77_Quests\JSON_Quvest\Silver_77_Quests_BackUP.json`
   - `Support\JSON_Quvest\editor-config.local.json` is no longer required for the normal path
3. Editor/UI micro-layer already closed:
   - big red stage labels were added for role blocks
   - `Offer` / `Reward` stay single-trigger in editor/client/server
   - `node --check P:\Silver_77_Quests\Support\JSON_Quvest\app.js` passes after the latest label pass
4. Mod default baseline is now baked into runtime too:
   - `P:\Silver_77_Quests\Silver_77_Quests_Server\scripts\4_World\QuestServerManager.c`
   - `CreateDefaultQuestConfig()` now mirrors the accepted 5-quest JSON baseline instead of the old 4-quest setup
   - this means a clean profile should spawn the current accepted quest pack by default
5. Next real external step stays user-side:
   - rebuild `Silver_77_Quests_Client`
   - rebuild `Silver_77_Quests_Server`
   - test on a clean profile / clean generated `profiles\Silver_77_Quests\Silver_77_Quests.json`
6. Do not switch back to `SplitMods`:
   - current build source remains the root split folders `Silver_77_Quests_Client` and `Silver_77_Quests_Server`
   - `SplitMods\` is still only a reserve copy
7. Stack rules reference is active and should stay in the check list:
   - the editor stack reference is live via `/api/stack-rules`
   - active server route now reads `P:\Silver_77_Quests\JSON_Quvest\item-stack-rules.json`
   - `Support\JSON_Quvest\item-stack-rules.json` is no longer the live source
   - after editor checks, also verify that the stack reference loads, saves, and still shows the expected current rules
8. Server-side default quest file creation had one more important fix:
   - `P:\Silver_77_Quests\Silver_77_Quests_Server\scripts\4_World\QuestServerManager.c`
   - `Silver77_SaveQuestConfigFile(...)` no longer writes the default config through raw `OpenFile("$profile:...")`
   - it now uses `JsonFileLoader<Silver77_QuestConfig>.JsonSaveFile(...)`, the same safer path style already used for player quest data
   - if the default quest JSON is absent again after rebuild, inspect this area first and then check the server RPT

## Fresh Checkpoint - 2026-04-30

1. Another server fallback was added after the live failure report:
   - `P:\Silver_77_Quests\Silver_77_Quests_Server\scripts\4_World\QuestServerRPC.c`
   - `modded class PlayerBase`
   - `EEInit()` now prints a bootstrap log and calls `QuestServerManager.EnsureQuestNpcsSpawned();`
2. Purpose of this fallback:
   - if some other mod breaks the normal `MissionServer.OnInit()` super chain, the quest server should still get one more chance to load/create config and spawn NPCs when the first real player entity initializes
3. Current live blocker is still NOT solved:
   - on the live server, `profiles\Silver_77_Quests\Silver_77_Quests.json` still was not confirmed as auto-created
   - quest NPCs still were not confirmed as spawned
   - quests therefore still were not confirmed as usable in-game
4. Important code-level observation from the latest inspection:
   - built-in NPC classes in default trigger config are vanilla survivors (`SurvivorM_Mirek`, `SurvivorM_Boris`, `SurvivorM_Oliver`)
   - this makes broken quest initialization / missing fresh server PBO / broken hook chain more likely than a bad NPC classname in the default quest data
5. Do not over-read the existence of `profiles\Silver_77_Quests\players\`:
   - that folder can be created by player-data save paths
   - it is not enough proof that full quest init or NPC spawn already succeeded
6. First thing to demand in the next debugging session:
   - live RPT lines for:
     - `[Silver_77_Quests] MissionServer.OnInit called`
     - `[Silver_77_Quests] Loading quest config from:`
     - `[Silver_77_Quests] Config not found, creating default...`
     - `[Silver_77_Quests] Saving quest config to:`
     - `[Silver_77_Quests] Quest config saved via JsonSaveFile:`
     - `[Silver_77_Quests] PlayerBase.EEInit bootstrap for quest server:`
     - `[Silver_77_Quests] Quest NPC cache is empty, spawning configured NPCs now`
     - `[Silver_77_Quests] Spawned quest NPC ...`
7. New concrete finding from live RPT inspection:
   - `Silver_77_Quests_Server.pbo` physically exists on the live server and matches the locally built file by SHA256
   - but unlike other `-servermod` addons such as `CacheSpawner`, it was still not showing up in:
     - `ENGINE : FileSystem: Adding package ... Silver_77_Quests_Server.pbo`
     - `SCRIPT : Silver_77_Quests_Server`
   - this strongly points to addon registration / config module wiring, not quest JSON content
8. Config-level mitigation already applied after that finding:
   - `P:\Silver_77_Quests\Silver_77_Quests_Server\config.cpp`
   - removed `Silver_77_Quests_Client` from `requiredAddons[]`
   - reduced `dependencies[]` from `{"Game", "World", "Mission"}` to `{"World", "Mission"}`
   - reason: the server addon only defines `worldScriptModule` and `missionScriptModule`; the old config shape may have prevented clean script registration
9. Live retest after that config fix still failed in the same place:
   - `profiles\Silver_77_Quests\Silver_77_Quests.json` was still not created
   - quest NPCs were still absent in the world
   - the fresh RPT still did NOT contain:
     - `Adding package 'C:\Server\Dayz\Shitler_00\@Silver_77_Quests_Server\addons\Silver_77_Quests_Server.pbo'`
     - `SCRIPT       : Silver_77_Quests_Server`
     - any `[Silver_77_Quests] ...` runtime logs
10. Important contrast from the same live RPT:
   - `@CacheSpawner` from the same `-servermod` line does appear both in `Adding package ...` and in the `SCRIPT : ...` addon list
   - `@Silver_77_Quests_Server` does not
   - so the current blocker is now specifically “this server PBO is not being registered/loaded as an addon”, not “quest JSON logic is wrong”
11. Highest-value next debugging target:
   - treat `Silver_77_Quests_Server.pbo` as a package-registration problem
   - inspect/rebuild around PBO validity / addon registration / alternate pack method
   - do NOT spend the next session first on quest logic, NPC data, or profile JSON shape
12. Strongest concrete low-level clue so far:
   - direct PBO header inspection showed that BOTH live PBOs currently advertise the same internal prefix:
     - `@Silver_77_Quests_Client\addons\Silver_77_Quests_Client.pbo` -> `prefix = Silver_77_Quests_Server`
     - `@Silver_77_Quests_Server\addons\Silver_77_Quests_Server.pbo` -> `prefix = Silver_77_Quests_Server`
   - the client PBO should not be using the server prefix
13. Practical interpretation:
   - the build/pack step for the client PBO is very likely using the wrong Addon Builder prefix override
   - duplicate internal prefix between client/server is now the strongest working explanation for why the server addon never appears in `Adding package ...` / `SCRIPT ...`
14. If a specialist continues from here, the first thing to verify is:
   - when packing `Silver_77_Quests_Client.pbo`, the actual internal prefix must be `Silver_77_Quests` (or whatever the intended distinct client prefix is)
   - right now it is still `Silver_77_Quests_Server`

## Fresh Checkpoint - 2026-04-30 - runtime restored, potato quest data fix prepared

1. Live runtime is no longer the main blocker:
   - after rebuilding the client PBO with the correct distinct client prefix, the quest server came back to life;
   - `profiles\Silver_77_Quests\Silver_77_Quests.json` is now auto-created again;
   - quest NPCs are spawning in the world again;
   - quests can be accepted in-game again.
2. New gameplay bug found immediately after runtime recovery:
   - `quest_hunter_2` (`Offer -> Completion -> Reward`) advanced at `Rasputin_1_trigger` even though Rasputin did not actually take the potatoes;
   - the large potato hand-in on `quest_hunter_1` also did not behave as expected in live play.
3. Root cause analysis from source + live JSON:
   - `quest_hunter_2` was giving `PotatoSeed x12` but had `objectives = []`;
   - because the completion stage had no objectives, the server correctly allowed the completion trigger to progress without consuming items;
   - `quest_hunter_1` was still configured for `PotatoSeed x120`, while project docs/examples already treat the cumulative food hand-in as `Potato`.
4. Data fix was corrected again after the latest user review:
   - intended default-server item is still `PotatoSeed`, not peeled `Potato`;
   - `quest_hunter_2` still keeps the real completion objective, but it is now `PotatoSeed x12`;
   - `quest_hunter_1` objective is back on `PotatoSeed`;
   - the earlier `Potato` remap must be treated as a rejected hypothesis, not the accepted baseline.
5. Files updated for this fix:
   - `P:\Silver_77_Quests\Silver_77_Quests_Server\scripts\4_World\QuestServerManager.c`
   - `P:\Silver_77_Quests\JSON_Quvest\Silver_77_Quests.json`
   - `P:\Silver_77_Quests\JSON_Quvest\Silver_77_Quests_BackUP.json`
   - `P:\Silver_77_Quests\Support\JSON_Quvest\Silver_77_Quests.json`
   - `P:\Silver_77_Quests\Support\JSON_Quvest\Silver_77_Quests_BackUP.json`
   - `P:\Silver_77_Quests\Support\JSON_Quvest\editor-draft.json`
6. Still not closed in this pass:
   - the corrected `PotatoSeed` fix has not yet been rebuilt and retested live;
   - after rebuild, first verify:
     - Rasputin does not advance `quest_hunter_2` without consuming the `PotatoSeed x12`;
     - the same items can no longer silently bypass the chain;
     - `quest_hunter_1` now accepts the intended `PotatoSeed` class at Voron.
7. Additional client UX pass prepared after live feedback:
   - active chain quests should no longer disappear from NPC menus just because the next role is on another NPC;
   - `QuestClientManager.IsQuestVisibleForTrigger(...)` now keeps `active` / `reward_pending` quests visible on assigned chain NPCs;
   - `QuestUI.c` now builds a visible `Линия квеста` block plus `Контекст NPC` text inside the NPC dialog window;
   - button labels were also made stage-aware (`Взять`, `Передать предметы`, `Завершить этап`, `Получить награду`);
   - this UX layer still needs live verification after the next client rebuild.
8. New user correction after live UI review:
   - the current inline `Линия квеста` text inside the main description block is NOT the desired final UX;
   - the quest chain must move into a separate dedicated UI area/window in the NPC dialog;
   - the same dedicated chain area must also exist in the quest journal, not only in the NPC window.
9. Required next UX scope for chain quests:
   - separate chain panel with the full route:
     - who starts the quest;
     - intermediate NPC/trigger steps;
     - who finishes / gives reward;
   - completed chain steps should be visibly marked as completed;
   - future/unvisited dialog steps must not reveal full dialog text yet;
   - for unvisited stages, show only:
     - the NPC / trigger name;
     - the goal/purpose of that stage, if it was already communicated by the first NPC who issued the quest;
   - do not reveal the future stage's own dialog text before visiting that stage.
10. Required next dialog scope:
   - separate dedicated dialog area/window for the current NPC stage;
   - each stage should show only its own live dialog text for the current NPC interaction;
   - dialog history for already visited stages should be visible separately as past conversation history;
   - current implementation that mixes chain/context/dialog inside one description text must be treated as temporary and replaced.
   - partial progress already done:
     - the current stage dialog is no longer supposed to live at the bottom of the main description forever;
     - a dedicated `DialogText` area was added to the NPC quest layout as the first step, but it still needs a proper live rebuild/visual pass.
11. New data/editor requirements discovered during this live review:
   - item class names must not be shown raw to the player;
   - JSON/editor need a separate human-readable display string field for quest items (Russian label);
   - UI/journal/NPC windows should render that display label instead of raw class names whenever present.
12. Open data inconsistency that must be rechecked before the next content pass:
   - the latest potato assumptions may still be wrong for intended design;
   - live result showed the quest giving peeled `Potato`, not the originally intended seed-type item flow;
   - Rasputin dialog text also did not match the intended stage owner;
   - this strongly suggests another JSON/editor remap mismatch that should be inspected before more content edits are baked in.

## Next 7 Tasks - updated UX / data roadmap

1. Lock the NPC window to a strict UI contract.
   - left list = quest titles + state markers only;
   - center body = status / goal / progress / reward only;
   - right column = trigger route only;
   - bottom panel = one shared scrolling dialog journal only.
2. Replace the temporary left-side chain prototype with the final right-side trigger column.
   - current left `ChainPanel` is now only a temporary prototype and should not be treated as final UX;
   - the final trigger route belongs on the right side of the NPC window.
3. Build one unified lower dialog journal instead of separate popup/current-dialog windows.
   - the bottom area should accumulate dialog history in activation order;
   - current NPC dialog should appear there too, not in a separate top block;
   - presentation order should keep the current/live stage at the top, while older history sinks lower.
4. Add trigger-driven focus behavior.
   - clicking a trigger on the right should focus the matching dialog block in the lower journal;
   - the matching dialog block should be highlighted in gold;
   - no extra popup window is preferred for this flow.
5. Define ordering rules for secondary / optional stages.
   - primary chain keeps its authored route;
   - secondary stages without hard order should appear in the dialog journal by first visit / first activation order.
6. Mirror the same route/history model in the quest journal.
   - the journal should reuse the same trigger-route + dialog-history paradigm;
   - do not invent a different interaction model between NPC UI and journal.
7. Extend JSON/editor and then re-run live validation.
   - add player-facing item names instead of leaking raw `className`;
   - add explicit stage goal / display / presentation fields where needed;
   - re-check potato flow, Rasputin/Voron dialog ownership, and the full rebuilt UX in live play.

## Immediate Live Checklist

1. Rebuild `Silver_77_Quests_Client`.
2. Rebuild `Silver_77_Quests_Server` or update the live profile quest JSON from the accepted root baseline.
3. Verify `quest_hunter_2` gives `PotatoSeed x12`, not peeled `Potato`.
4. Verify Rasputin does not advance the chain until the `PotatoSeed x12` is actually handed in.
5. Verify Voron accepts the intended `PotatoSeed` class for the large potato quest.
6. Verify the current NPC dialog is not duplicated in the upper description body and is prepared to live in the one shared lower scrolling dialog journal.
7. Verify the main description area stays limited to status / objectives / NPC context, without dialog text or old inline chain text.
8. Verify Rasputin shows his own stage dialog, not a чужой/перепутанный dialog owner.
9. Verify the quest remains visible across the chain NPCs even after acceptance.
10. Verify the temporary route prototype is still readable until the final right-side trigger column replaces it.

## Fresh Checkpoint - 2026-05-01 - accept-flow stabilization + new UX contract

1. Live feedback from the user after the first NPC-chain-panel pass:
   - some quests, especially `quest_fisherman_2` / `Поставка медицины`, felt like they did not accept cleanly on the first click;
   - the NPC menu still felt visually mixed because the top area was showing quest description text that read like a second dialog copy.
2. `quest_fisherman_2` data itself was re-checked and still looks structurally valid:
   - same trigger is used for `offer` and `reward`;
   - objective is `BandageDressing x6`;
   - no obvious bad requirement chain was found in the JSON baseline.
3. A first client-side stabilizer has now been added:
   - `MissionGameplay.c` no longer keeps re-evaluating trigger focus every 0.5s while the quest UI or journal is open;
   - this freezes `m_CurrentTriggerId` / `m_CurrentQuestIds` for the open interaction and should reduce menu drift during accept / complete clicks.
4. The NPC window also stopped showing `quest.description` in the upper description body:
   - the intent is to keep live NPC dialog in the dedicated lower dialog block only;
   - the upper body should now stay focused on status / context / objectives / rewards.
5. User clarified the final UX contract more strictly:
   - no mixed text blocks by meaning;
   - each window area must have one hard responsibility only.
6. Final NPC-window contract to build toward:
   - left = quest list only;
   - center = status / objective / progress / reward;
   - right = trigger route;
   - bottom = one shared scrolling dialog journal.
7. Trigger interactions should work like this:
   - clicking a trigger on the right should not open a popup by default;
   - it should highlight the matching dialog block in the lower journal in gold and focus that part of the history.
8. Secondary / optional stages currently do not have a strict authored completion order:
   - for now, their dialog/history order should be the order in which the player first visited / activated them.
9. This new contract supersedes older wording about building multiple separate dialog windows.
10. The stabilization pass still needs a live retest before it can be marked fully solved.
11. Important presentation rule for the lower journal:
   - “what matters now” should be visible first;
   - the current stage belongs at the top of the lower journal;
   - older stages stay below it as past history.

## Current code state to continue from tomorrow

1. Already in code:
   - the temporary left-side `ChainPanel` prototype has been replaced by a real right-side `TriggerRouteListbox`;
   - `quest.description` is no longer injected into the upper NPC description body;
   - `MissionGameplay.c` now freezes trigger-context refresh while the quest UI or journal is open;
   - player progress now has a new `stageVisits` history model (`triggerId + actionType + firstActivatedTime`) and the server records it on `offer`, `completion`, and `reward`;
   - the lower NPC dialog block now starts from a real journal-builder that reads `stageVisits` and appends the current live stage when it is not yet recorded;
   - selecting a route entry on the right now changes the focus context for the lower journal.
2. Still temporary / not final:
   - the right-side trigger route is already in code, but the lower journal focus is still only a text-marker (`>>`) instead of true gold in-text highlighting;
   - the lower dialog area is still a multiline text block, not a true scrolling dialog journal widget yet;
   - richer per-stage goal/reward payload is still not rendered there yet.
3. Known unresolved bugs / risks:
   - `quest_fisherman_2` / `Поставка медицины` sometimes feels like it accepts only after repeated clicks;
   - this was already narrowed down away from clearly broken JSON structure and more toward accept-flow / UI feedback / trigger-context drift;
   - the new trigger-freeze stabilizer still needs a live rebuild and retest;
   - the current live UI build also has a layout-text encoding regression: menu title / close text / some placeholders render as mojibake after the recent `QuestMenu.layout` rewrite;
   - partial turn-in progression is currently broken: items are removed from the player's inventory, but deposited counters do not increase, so the quest state stays stuck;
   - `stageVisits` is additive and safe for old profiles, but it cannot reconstruct past history for already active quests retroactively;
   - the lower journal currently shows dialog history text only;
   - right-side route focus exists, but the journal still uses a text-marker focus (`>>`) rather than true gold-colored in-text highlighting;
   - richer stage payloads are still not implemented.
   - NPC window layout was just re-separated into distinct description / route / dialog panels, so this geometry still needs the first live visual pass.
4. Do not re-investigate from scratch tomorrow:
   - the final UX direction is already agreed;
   - no popup-first design is preferred right now;
   - use one shared scrolling dialog journal, right-side trigger route, and activation-order history for secondary stages.
5. If tomorrow's live test specifically targets the new dialog-history order:
   - clear `profiles\Silver_77_Quests\players\*` for a clean history run;
   - do not delete the server quest JSON unless we also want to regenerate quest content defaults.

## Update 2026-05-01 - NPC panels separated for cleaner live UX

1. `QuestMenu.layout` now has three distinct visual zones instead of overlapping content:
   - central description panel;
   - right-side trigger route panel;
   - lower dialog panel.
2. This step was specifically done to stop the lower dialog area from physically overlapping the right-side route list during live testing.
3. It is still not the final UX:
   - the lower journal is still a multiline text block, not a true scrolling widget;
   - route focus still uses `>>` instead of true gold in-text highlighting.
4. New live-test regressions found immediately after this pass:
   - `QuestMenu.layout` static texts currently have broken encoding (`mojibake`);
   - item hand-in progress is broken for partial-delivery quests;
   - the user also wants the two large action buttons reduced in size, with free space on the right reserved for one more future text box.
5. Important clarification from the latest live test:
   - the server appears to accept the needed partial hand-in amount correctly;
   - extra items are left in the inventory once the required amount is already consumed;
   - the actual broken layer is the client-side progress refresh after partial turn-in, because the UI keeps showing `0 / N`, leaves the submit button active, and falls into `waiting for server` timeout.
6. Current diagnostic layer already added in code:
   - client `QuestClientRPC.c` now logs the received player-progress payload length and decoded quest-entry count;
   - if synced player data arrives with empty `steamId`, the client now applies a local identity fallback before `ApplySyncedPlayerData(...)`;
   - `QuestClientManager.ApplySyncedPlayerData(...)` now logs how many non-zero objective-progress entries and stage visits were actually applied.

## Update 2026-05-01 - static NPC UI text encoding and panel contrast pass

1. `QuestMenu.layout` has been rewritten again and explicitly saved in `windows-1251` for DayZ UI compatibility.
2. This pass targets the specific `mojibake` visible in live screenshots for:
   - `КВЕСТЫ`;
   - `ОПИСАНИЕ КВЕСТА`;
   - `МАРШРУТ`;
   - `ДИАЛОГОВЫЙ ЖУРНАЛ`;
   - `ВЗЯТЬ КВЕСТ / СДАТЬ КВЕСТ / ЗАКРЫТЬ`.
3. At the same time, panel contrast was increased:
   - darker description / route / dialog backgrounds;
   - brighter description label;
   - gold-tinted route and dialog labels.
4. This update is UI-only.
   - no profile cleanup is needed for this visual test;
   - no server logic was changed by this step.
5. Still unresolved after this pass:
   - partial item hand-in still desyncs on the client after deposit;
   - lower dialog journal is still not a real scrolling widget;
   - route focus is still `>>`, not true gold in-text highlighting;
   - action buttons are still oversized and a future extra right-side helper text box is not implemented yet.
