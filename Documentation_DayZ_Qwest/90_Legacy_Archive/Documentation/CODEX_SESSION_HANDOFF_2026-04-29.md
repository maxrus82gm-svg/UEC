# CODEX SESSION HANDOFF 2026-04-29

## Зачем этот файл

Это свежий handoff после уточнения продуктовой модели `Offer / Completion / Reward`.

Главная цель этого файла: не дать новой сессии ошибочно считать, что текущий `NPC Flow` уже доведен до финальной схемы. Базовая role-driven модель уже существует, но последние продуктовые правила стали жестче и теперь должны быть зафиксированы отдельно.

## Новая жесткая фиксация NPC Flow

- Один блок `NPC Flow` = одна роль.
- Нельзя держать в одном и том же блоке сразу две роли, например `Offer + Reward`.
- У одного квеста не должно быть возможности создать два `Offer`.
- `Reward` тоже должен быть уникальным и обязательным.
- `Completion` может быть ноль, один или несколько.
- Один и тот же NPC / trigger может встречаться в квесте несколько раз, но только как разные отдельные блоки с разными ролями.

Пример допустимой схемы:

- блок 1 = `Offer` у рыбака
- блок 2 = `Completion` у медика
- блок 3 = `Reward` снова у рыбака

Пример недопустимой схемы:

- один блок одновременно `Offer + Reward`
- два разных блока `Offer`
- отсутствие `Reward`

## Смысл трех ролей

### Offer

- Обязателен.
- Это главный стартовый блок квеста.
- В нем лежат главный диалог, стартовые требования и выдача предметов для прохождения.
- `Offer` может выдавать один или несколько предметов.
- `Offer` не должен быть местом финальной награды.

### Completion

- Не обязателен.
- Может быть несколько разных блоков.
- Каждый `Completion` имеет свой собственный диалог.
- Каждый `Completion` может принимать нужные предметы у игрока.
- Каждый `Completion` может иметь свою локальную награду.
- `Completion` не имеет права закрывать квест окончательно.

### Reward

- Обязателен.
- Это единственная финальная точка закрытия квеста.
- У `Reward` свой собственный диалог.
- `Reward` выдает финальную награду.
- `Reward` не должен закрывать квест и выдавать финальную награду, пока не выполнены:
  - требования самого `Offer`
  - все обязательные `Completion`

## Что пользователь явно разрешил

- Не держаться за старую структуру JSON, если новая схема получается чище.
- Перестраивать draft и шаблонный JSON под новую модель редактора.
- Не считать обратную совместимость со старыми profile / history приоритетом, если она мешает новой схеме.

## Что уже есть к этому моменту

- Role-driven поля и `triggerActions` уже существуют.
- В рантайме уже есть `reward_pending`.
- В рантайме уже есть отслеживание `completedCompletionTriggerIds`.
- В моде и клиенте уже есть базовая role-driven видимость.
- В UI уже существует `NPC Flow` и локальные карточки ролей.
- Желтая подсветка reward-сценария уже заложена.

## Что нельзя молча предполагать

- Нельзя считать, что редактор уже полностью запрещает второй `Offer`.
- Нельзя считать, что редактор уже физически не дает смешать `Offer + Reward` в одном блоке.
- Нельзя считать, что текущий JSON / draft уже на 100% приведены к новой жесткой схеме только потому, что базовая role-driven модель уже была внедрена.
- Нельзя уходить сразу в сборку PBO, пока не проверено, что редактор реально держит эти ограничения.

## Ближайшая практическая задача

Сначала добить редактор и схему данных, и только потом собирать мод.

Порядок:

1. Проверить и довести `NPC Flow` до правила `один блок = одна роль`.
2. Запретить второй `Offer`.
3. Запретить второй `Reward`.
4. Запретить смешивание `Offer + Reward` в одной карточке.
5. Сохранить возможность повторять один и тот же NPC только как другой отдельный блок.
6. Проверить, что `Completion` действительно настраивается как промежуточная сдача со своим диалогом и локальной наградой.
7. Проверить, что `Reward` остается только финальным закрывающим блоком.
8. Только после этого переходить к live-проверке видимости, желтой подсветки и сборке client/server.

## Что особенно важно для следующего агента

- Сейчас главный фокус снова на редакторе, а не на сборке.
- Новый агент должен читать старые handoff-файлы как фон, но считать именно этот файл самым свежим продуктовым срезом.
- Если новая сессия короткая, после `CODEX_START_HERE.md` открыть именно этот handoff первым.

## Точное состояние кода на момент остановки

Быстрая разведка по живым файлам уже сделана. Важно не повторять её с нуля, а продолжать отсюда.

### Что уже есть в моде

- В `Silver_77_Quests_Client\scripts\3_Game\QuestData.c` уже есть поля:
  - `hideUntilRequirementsComplete`
  - `offerTriggerIds`
  - `completionTriggerIds`
  - `rewardTriggerIds`
  - `triggerActions`
- В рантайме уже есть:
  - `reward_pending`
  - `completedCompletionTriggerIds`
  - отдельная логика completion / reward
- Сервер и клиент уже умеют считать role-driven видимость и route `Offer -> Completion -> Reward`.

### Что сейчас мешает новой строгой схеме

- В сервере и клиенте `offerTriggerIds` и `rewardTriggerIds` всё ещё трактуются как обычные массивы без жёсткого ограничения на один trigger.
- В редакторе уже написана более правильная карточка `renderQuestRoleFlowCard(...)`, но активная секция `NPC Flow` всё ещё рендерит старую многорольную карточку `renderQuestTriggerFlowCard(...)`.
- В `seedQuestRolesFromAssignments(...)` редактор всё ещё досеивает `offerTriggerIds` и `rewardTriggerIds` сразу всеми `assignedTriggerIds`, а не только одним trigger.
- В `normalizeQuest(...)` редактор пока не подрезает `offerTriggerIds` / `rewardTriggerIds` до одного элемента.
- В `validateData(...)` пока нет жёсткой ошибки на второй `Offer` / второй `Reward`.

### Уже найденные точки входа по файлам

- Сервер:
  - `P:\Silver_77_Quests\Silver_77_Quests_Server\scripts\4_World\QuestServerManager.c`
  - ключевые зоны:
    - `SeedDefaultQuestRoles(...)`
    - `NormalizeQuestConfig(...)`
    - `GetQuestOfferTriggerIds(...)`
    - `GetQuestRewardTriggerIds(...)`
    - `CanAcceptQuest(...)`
    - `CanClaimReward(...)`
    - `CompleteQuest(...)`
- Клиент:
  - `P:\Silver_77_Quests\Silver_77_Quests_Client\scripts\4_World\QuestClientManager.c`
  - ключевые зоны:
    - `Silver77_ClientSeedQuestRoles(...)`
    - `Silver77_ClientNormalizeQuestConfig(...)`
    - `GetQuestOfferTriggerIds(...)`
    - `GetQuestRewardTriggerIds(...)`
    - `IsQuestVisibleForTrigger(...)`
    - `CanClaimReward(...)`
    - `ShouldHighlightQuestAsReward(...)`
- Редактор:
  - `P:\Silver_77_Quests\Support\JSON_Quvest\app.js`
  - ключевые зоны:
    - `handleQuestTriggerToggleChange(...)`
    - `canAssignRoleToTrigger(...)`
    - `getQuestFlowVisibleTriggers(...)`
    - `renderQuestRoleFlowCard(...)`
    - `renderQuestTriggerFlowSection(...)`
    - `seedQuestRolesFromAssignments(...)`
    - `normalizeQuest(...)`
    - `validateData(...)`

## Пошаговый план следующей сессии

### Шаг 1. Сначала зажать роль-модель в редакторе

Смысл: перестать собирать неправильный JSON прямо в UI.

Сделать:

1. В `normalizeQuest(...)` подрезать:
   - `offerTriggerIds` -> максимум один trigger
   - `rewardTriggerIds` -> максимум один trigger
2. В `seedQuestRolesFromAssignments(...)` досевать:
   - `Offer` только первым `assignedTriggerId`
   - `Reward` только первым `assignedTriggerId`
3. В `validateData(...)` добавить ошибки:
   - если `offerTriggerIds.length > 1`
   - если `rewardTriggerIds.length > 1`
4. В `NPC Flow` переключить активный рендер на `renderQuestRoleFlowCard(...)`, а не на старую `renderQuestTriggerFlowCard(...)`.
5. Сохранить правило:
   - один блок = одна роль
   - повтор того же NPC допустим только отдельным блоком для другой роли

### Шаг 2. Потом зажать ту же модель в runtime мода

Смысл: даже если старый JSON где-то останется, клиент/сервер всё равно нормализуют его к новой схеме.

Сделать:

1. В `QuestServerManager.c`:
   - после seed/normalize приводить `offerTriggerIds` к одному trigger
   - после seed/normalize приводить `rewardTriggerIds` к одному trigger
2. В `QuestClientManager.c`:
   - сделать ту же нормализацию
3. Не ломать `completionTriggerIds`: они остаются множественными

### Шаг 3. Только после этого переходить к точечной логике UI

Проверить:

- что `Offer + Reward` не могут жить в одной карточке
- что второй `Offer` нельзя включить
- что второй `Reward` нельзя включить
- что один и тот же NPC можно повторить отдельным блоком для другой роли
- что `Completion` всё ещё имеет свой диалог и локальную награду

## Что не делать в следующей сессии первым делом

- Не тратить шаги на `git status`: тут нет git-репозитория.
- Не уходить сразу в build / rebuild / publish: это делает пользователь сам.
- Не начинать с крупного рефактора сервера до того, как редактор перестанет собирать старую многорольную схему.

## Checkpoint - первый маленький слой уже выполнен

В `P:\Silver_77_Quests\Support\JSON_Quvest\app.js` уже зафиксирован первый реальный слой новой схемы:

- добавлены helper-функции:
  - `normalizeSingleTriggerRoleIds(...)`
  - `updateSingleTriggerRoleMembership(...)`
- `handleQuestTriggerToggleChange(...)` теперь обрабатывает `Offer` и `Reward` как одиночные роли, а не как обычные массивы
- `normalizeQuest(...)` теперь подрезает:
  - `offerTriggerIds` до одного trigger
  - `rewardTriggerIds` до одного trigger
- `seedQuestRolesFromAssignments(...)` теперь досеивает:
  - `Offer` только первым `assignedTriggerId`
  - `Reward` только первым `assignedTriggerId`
- `validateData(...)` уже выдает ошибки:
  - `Слишком много offer trigger`
  - `Слишком много reward trigger`
- `renderQuestTriggerFlowSection(...)` уже переключен с многорольной карточки `renderQuestTriggerFlowCard(...)` на строгую `renderQuestRoleFlowCard(...)`

Проверка:

- `node --check P:\Silver_77_Quests\Support\JSON_Quvest\app.js` проходит

Что осталось следующим слоем:

- добить runtime-нормализацию в `QuestServerManager.c` и `QuestClientManager.c`
- проверить, не осталось ли в редакторе старых живых мест, которые всё ещё опираются на многорольную карточку
- потом уже отдельно прогонять поведение UI и данных на живом JSON

## Checkpoint - второй маленький слой уже выполнен

После первого editor-слоя уже закрыт и следующий runtime-слой.

- В `P:\Silver_77_Quests\Silver_77_Quests_Server\scripts\4_World\QuestServerManager.c`:
  - добавлен helper `Silver77_ServerNormalizeSingleTriggerRoleIds(...)`;
  - `SeedDefaultQuestRoles(...)` теперь сначала подрезает `offerTriggerIds` и `rewardTriggerIds` до одного trigger, а если роль пустая, досеивает только первым `assignedTriggerId`;
  - `NormalizeQuestConfig(...)` тоже подрезает `offerTriggerIds` и `rewardTriggerIds` до одного trigger;
  - `GetQuestOfferTriggerIds(...)` и `GetQuestRewardTriggerIds(...)` теперь отдают только первый валидный trigger через `AppendFirstValidTriggerId(...)`.
- В `P:\Silver_77_Quests\Silver_77_Quests_Client\scripts\4_World\QuestClientManager.c` сделан симметричный слой:
  - добавлен `Silver77_ClientNormalizeSingleTriggerRoleIds(...)`;
  - `Silver77_ClientSeedQuestRoles(...)` и `Silver77_ClientNormalizeQuestConfig(...)` теперь держат `Offer` и `Reward` только как single-trigger роли;
  - client getters для `offer/reward` тоже возвращают только первый валидный trigger.
- `completionTriggerIds` специально не менялись и остаются множественными.

Дополнительно зафиксирован небольшой cleanup в `P:\Silver_77_Quests\Support\JSON_Quvest\app.js`:

- живые мутации `offer/reward` при remove / hide / rename trigger теперь идут через single-role helper-ы и не раздувают массивы обратно;
- старая многорольная карточка и старый validator после этого слоя уже физически вырезаны из live-path `app.js`;
- активными рабочими версиями остаются:
  - `validateData(...)`
  - `validateTriggerRoleArray(...)`
  - `renderQuestRoleFlowCard(...)`
- дополнительный cleanup по `app.js` тоже уже сделан:
  - убран физический dead-code хвост старого validator / старой trigger-flow карточки;
  - снят ещё один дубль по имени `buildQuestRoleTextForTrigger(...)`, чтобы в файле не оставалось скрытых function override.

Проверка:

- `node --check P:\Silver_77_Quests\Support\JSON_Quvest\app.js` проходит.

Что решено этим слоем:

- runtime больше не должен молча принимать множественные `Offer` / `Reward` trigger'ы как рабочую норму;
- даже если старый JSON где-то остался, client/server теперь сами поджимают его к single-trigger схеме;
- в редакторе больше нет живой записи, которая после rename/remove trigger снова раздувает `offerTriggerIds` или `rewardTriggerIds`.

Что ещё не решено:

- нужен реальный ручной прогон на живом JSON редактора;
- игровой build / rebuild / publish и in-game проверка по-прежнему остаются на стороне пользователя.

## Checkpoint - редакторский baseline и savePath зафиксированы

После ручного прохода пользователя по всем квестам принято текущее состояние editor JSON как новый baseline.

- Текущий принятый baseline JSON:
  - `P:\Silver_77_Quests\JSON_Quvest\Silver_77_Quests.json`
  - `version = 3`
  - `quests = 5`
  - в baseline уже есть `quest_hunter_2`
  - в редакторе уже заведены отдельные диалоги и второстепенные награды по новой схеме
- Дефолтные пути сохранения закреплены как рабочие:
  - `P:\Silver_77_Quests\JSON_Quvest\Silver_77_Quests.json`
  - `P:\Silver_77_Quests\JSON_Quvest\Silver_77_Quests_BackUP.json`
- `Support\JSON_Quvest\editor-config.json` теперь тоже смотрит на эти absolute path, а временный local override больше не нужен.
- В UI добавлена более явная крупная маркировка role-блоков:
  - `СТАРТ КВЕСТА`
  - `ПРОМЕЖУТОЧНОЕ ЗАДАНИЕ`
  - `ЗАВЕРШАЮЩИЙ`
  - стиль: красный текст, крупный размер для быстрого визуального считывания цепочки.

Что считать выполненным на этом checkpoint:

- role-driven блоки в редакторе визуально работают;
- маршрут с передачей и возвратом за наградой пользователь руками уже собрал в редакторе;
- текущий `version=3` JSON принят как дефолтный baseline для следующей сессии;
- рабочие savePath / backupPath больше не должны зависеть от случайного local override.


## Checkpoint - default mod baseline now matches the accepted JSON

Closed in this pass:

1. `QuestServerManager.c` default config was no longer left on the outdated built-in seed.
2. `P:\Silver_77_Quests\Silver_77_Quests_Server\scripts\4_World\QuestServerManager.c`
   - `CreateDefaultQuestConfig()` now mirrors the accepted `version=3`, `quests=5` JSON baseline;
   - includes `quest_hunter_2`;
   - includes the real `completion` trigger on `Rasputin_1_trigger`;
   - uses the current trigger quest order from the accepted editor JSON.
3. Editor/UI checkpoint also confirmed in the same pass:
   - large red role-stage labels are already wired in;
   - `node --check P:\Silver_77_Quests\Support\JSON_Quvest\app.js` passes after that UI pass.

What is solved now:

- accepted quest content is no longer only in editor JSON;
- a clean server profile should now bootstrap the same quest pack from mod defaults;
- working save/backup paths stay anchored to the root `P:\Silver_77_Quests\JSON_Quvest` folder.

What is still not solved:

- no DayZ rebuild was run in this session;
- no in-game validation yet on a clean profile after baking the new default config;
- `SplitMods\` was not resynced and should still be treated only as reserve, not as the active build source.
- stack rules route is now rewired to the root live file `P:\Silver_77_Quests\JSON_Quvest\item-stack-rules.json`; `Support\JSON_Quvest\item-stack-rules.json` is no longer the active source.
- default quest config save path was also hardened after a live failure report:
  - `Silver77_SaveQuestConfigFile(...)` now saves through `JsonFileLoader<Silver77_QuestConfig>.JsonSaveFile(...)`
  - the previous raw `OpenFile("$profile:...")` route is no longer used for quest-config creation

## Checkpoint - live server runtime is still the main open blocker

Closed in code, but not confirmed live yet:

1. `P:\Silver_77_Quests\Silver_77_Quests_Server\scripts\4_World\QuestServerRPC.c`
   - `modded class PlayerBase`
   - `EEInit()` now acts as one more bootstrap path for the quest server
   - on server, for a real player with identity, it logs and calls `QuestServerManager.EnsureQuestNpcsSpawned();`
2. Why this was added:
   - the live symptoms suggested that another mod may be breaking the normal mission init chain
   - this fallback is meant to give quest config load + NPC spawn another chance even if `MissionServer.OnInit()` never reaches our layer

Current unresolved state:

1. Live server still did not prove automatic creation of:
   - `profiles\Silver_77_Quests\Silver_77_Quests.json`
2. Live server still did not prove quest NPC spawn.
3. Manually dropping the quest JSON into profile was not enough to make the quest NPCs appear.
4. Therefore the real blocker remains server runtime bootstrap / hook execution / fresh PBO pickup, not editor JSON structure.

Important observation from source inspection:

1. Default trigger NPC classes are valid vanilla survivor classes:
   - `SurvivorM_Mirek`
   - `SurvivorM_Boris`
   - `SurvivorM_Oliver`
2. This makes broken hook execution or stale live deploy more likely than a bad built-in NPC classname.
3. The existence of `profiles\Silver_77_Quests\players\` alone is not enough proof of full init, because player-data code can create that folder separately.

Next session priority:

1. Get live RPT lines for:
   - `MissionServer.OnInit called`
   - `Loading quest config from:`
   - `Config not found, creating default...`
   - `Saving quest config to:`
   - `Quest config saved via JsonSaveFile:`
   - `PlayerBase.EEInit bootstrap for quest server:`
   - `Quest NPC cache is empty, spawning configured NPCs now`
   - `Spawned quest NPC ...`
2. If those lines are absent, treat the problem first as fresh-server-PBO or hook-chain failure, not as a quest JSON content bug.

## Checkpoint - live RPT now points at addon registration, not JSON data

1. Direct inspection of the live server path confirmed:
   - `\\192.168.0.77\Server\Dayz\Shitler_00\@Silver_77_Quests_Server\addons\Silver_77_Quests_Server.pbo` exists
   - its SHA256 matches the local built file in `P:\Mods_DONE\@Silver_77_Quests_Server\addons\Silver_77_Quests_Server.pbo`
2. Direct inspection of the built PBO content confirmed:
   - it contains `config.cpp`
   - `scripts\4_World\QuestServerManager.c`
   - `scripts\4_World\QuestServerRPC.c`
   - `scripts\5_Mission\mission\MissionServer.c`
3. But live RPT still showed an important mismatch:
   - `CacheSpawner` from `-servermod` appears in `Adding package ...` and in the `SCRIPT : ...` list
   - `Silver_77_Quests_Server` did not show in either place
   - at the same time, `Silver_77_Quests_Client` did show normally
4. Working interpretation:
   - the blocker is now most likely server addon registration / script module wiring
   - not the contents of `Silver_77_Quests.json`
5. Config mitigation already applied after this finding:
   - `P:\Silver_77_Quests\Silver_77_Quests_Server\config.cpp`
   - removed `Silver_77_Quests_Client` from `requiredAddons[]`
   - changed `dependencies[]` from `{"Game", "World", "Mission"}` to `{"World", "Mission"}`
6. Why this specific fix was chosen:
   - server addon only defines `worldScriptModule` and `missionScriptModule`
   - the old config declared `Game` dependency without a matching `gameScriptModule`
   - it also declared a direct patch dependency on `Silver_77_Quests_Client`, which may be unsafe/noisy across the `-mod` / `-servermod` split

## Checkpoint - config.cpp mitigation did not restore server addon loading

1. After rebuilding and redeploying the server PBO with the new `config.cpp`, the live result was still negative:
   - `profiles\Silver_77_Quests\Silver_77_Quests.json` still was not generated
   - quest NPCs still were not present in the world
2. Fresh live RPT still did NOT show:
   - `Adding package 'C:\Server\Dayz\Shitler_00\@Silver_77_Quests_Server\addons\Silver_77_Quests_Server.pbo'`
   - `SCRIPT       : Silver_77_Quests_Server`
   - any `[Silver_77_Quests] ...` lines
3. But the same fresh RPT DID still show:
   - `@CacheSpawner` from the same `-servermod` line entering `Adding package ...`
   - `SCRIPT       : CacheSpawner`
   - `Silver_77_Quests_Client` loading normally from `-mod`
4. Therefore the current best reading is:
   - the live blocker remains at the level of `Silver_77_Quests_Server.pbo` registration/loading as an addon package
   - not at the level of quest-config save logic, NPC spawn data, or role-chain quest logic
5. This is now the first thing to carry into the next session.

## Checkpoint - duplicate internal PBO prefix is now the strongest root-cause candidate

1. Direct live binary-header inspection of the deployed PBOs showed:
   - `\\192.168.0.77\Server\Dayz\Shitler_00\@Silver_77_Quests_Client\addons\Silver_77_Quests_Client.pbo`
     - `prefix = Silver_77_Quests_Server`
   - `\\192.168.0.77\Server\Dayz\Shitler_00\@Silver_77_Quests_Server\addons\Silver_77_Quests_Server.pbo`
     - `prefix = Silver_77_Quests_Server`
2. This is not expected:
   - client and server PBOs should not advertise the same internal prefix
   - the client PBO especially should not carry the server prefix
3. Why this matters:
   - duplicate internal prefix is now the strongest candidate for why:
     - `Silver_77_Quests_Client` appears in `RPT`
     - `Silver_77_Quests_Server` never appears in `Adding package ...`
     - `Silver_77_Quests_Server` never appears in `SCRIPT : ...`
4. Most likely operational cause:
   - Addon Builder pack step for the client mod is using the wrong manual `Addon prefix` override or cached prefix value
5. Highest-value next manual fix outside Codex:
   - repack the client addon with the correct distinct client prefix
   - then redeploy both client/server PBOs and re-check whether `Silver_77_Quests_Server` finally enters `Adding package ...`
## Update 2026-04-30 - runtime restored after correct client prefix rebuild

1. After rebuilding `Silver_77_Quests_Client.pbo` with the correct distinct client prefix (`Silver_77_Quests`) and redeploying both PBOs, the quest runtime came back.
2. Live confirmation:
   - `profiles\Silver_77_Quests\Silver_77_Quests.json` is auto-created again;
   - quest NPCs are visible in the world again;
   - quests can be accepted again.
3. This closes the previous top-level blocker around live server addon registration enough to continue gameplay debugging.

## Update 2026-04-30 - potato quest hand-in bug isolated and patched in data

1. New live symptom after runtime recovery:
   - `quest_hunter_2` progressed at Rasputin without actually consuming the potatoes;
   - the large potato hand-in on `quest_hunter_1` did not behave as expected.
2. Root cause found:
   - `quest_hunter_2` gave `PotatoSeed x12` but had `objectives = []`, so the completion trigger had nothing to enforce;
   - `quest_hunter_1` still targeted `PotatoSeed x120`, while project docs/examples already model the cumulative quest item as `Potato`.
3. Fix prepared, then corrected again after live user review:
   - intended baseline item stays `PotatoSeed`, not peeled `Potato`;
   - `quest_hunter_1` objective is back on `PotatoSeed`;
   - `quest_hunter_2` give item is back on `PotatoSeed x12`;
   - `quest_hunter_2` still keeps the real completion objective, now `PotatoSeed x12` with `allowPartialTurnIn = true`.
4. Files updated:
   - `P:\Silver_77_Quests\Silver_77_Quests_Server\scripts\4_World\QuestServerManager.c`
   - `P:\Silver_77_Quests\JSON_Quvest\Silver_77_Quests.json`
   - `P:\Silver_77_Quests\JSON_Quvest\Silver_77_Quests_BackUP.json`
   - `P:\Silver_77_Quests\Support\JSON_Quvest\Silver_77_Quests.json`
   - `P:\Silver_77_Quests\Support\JSON_Quvest\Silver_77_Quests_BackUP.json`
   - `P:\Silver_77_Quests\Support\JSON_Quvest\editor-draft.json`
5. Still pending after this patch:
   - rebuild/redeploy and live retest that Rasputin only progresses after the `PotatoSeed` items are really handed in;
   - confirm Voron now accepts the intended `PotatoSeed` class for the `x120` quest.
6. Additional UX issue reported right after that:
   - chain quests were disappearing from the original giver / secondary NPC menus once accepted, which made the route feel lost unless the player opened the journal.
7. Prepared client-side UX fix:
   - `QuestClientManager.IsQuestVisibleForTrigger(...)` now keeps `active` and `reward_pending` quests visible on assigned chain NPCs;
   - `QuestUI.c` now adds a visible `Линия квеста` block in the NPC dialog window;
   - the same NPC window now also shows `Контекст NPC`, so the player can see where the next stage is even if this NPC cannot act right now;
   - button captions are now stage-aware (`ВЗЯТЬ КВЕСТ`, `ПЕРЕДАТЬ ПРЕДМЕТЫ`, `ЗАВЕРШИТЬ ЭТАП`, `ПОЛУЧИТЬ НАГРАДУ`).
8. Still pending after the UX patch:
   - rebuild/redeploy the client PBO;
   - confirm in live play that both primary and secondary chain NPCs keep the quest visible and that the route text is actually readable in the dialog window.
9. User reviewed that result live and clarified the real target UX:
   - the current inline chain text inside the main quest description is not acceptable as a final solution;
   - `Линия квеста` must move into a separate dedicated UI block/window in the NPC interaction menu;
   - the same separate chain block/window must also exist in the quest journal.
10. Additional mandatory behavior for that future chain block:
   - show the full route of the quest chain;
   - mark completed stages;
   - for stages whose dialogs have not been visited yet, show only:
     - NPC / trigger name;
     - the stage goal if that goal was already received from the first NPC who issued the quest;
   - do not reveal the future stage's own dialog text before visiting it;
   - keep visited dialog stages available as history.
11. Additional mandatory behavior for dialogs:
   - dialog text for the current interaction must live in its own dedicated dialog area/window;
   - each NPC stage must show its own stage-specific dialog only when interacting with that NPC;
   - current mixed description/context/dialog rendering is temporary and should be replaced.
   - partial implementation already started:
     - `QuestMenu.layout` now has a dedicated `DialogText` block for NPC-stage dialog;
     - `QuestUI.c` was adjusted so dialog is being moved out of the main description flow;
     - this still needs a proper rebuild and live visual verification.
12. Additional data/editor requirement:
   - raw item class names should not be shown to players;
   - JSON/editor need a separate human-readable display-name field for items so UI can render Russian labels instead of `className`.
13. Important open content mismatch discovered during the same live check:
   - the latest potato flow may still be mapped incorrectly;
   - live behavior showed peeled `Potato` being given, while the earlier design intent sounded closer to another item flow;
   - Rasputin's dialog text also did not match the intended stage owner;
   - treat this as a likely JSON/editor remap/content-mapping issue before doing another broad content bake.

## Next 7 Tasks - updated UX / data execution order

1. Lock the NPC window to a hard UI contract.
   - left list = quest titles + state markers only;
   - center body = status / goal / progress / reward only;
   - right column = trigger route only;
   - bottom panel = one shared scrolling dialog journal only.
2. Replace the temporary left-side chain prototype with the final right-side trigger column.
   - the current left `ChainPanel` is only a prototype;
   - the final trigger route belongs on the right side of the NPC window.
3. Build one unified lower dialog journal instead of multiple dialog windows.
   - current NPC dialog must live in the same lower journal as the history;
   - prefer one shared scroll area over popups;
   - presentation order should keep the current/live stage at the top, while older history stays lower.
4. Add trigger-driven focus behavior.
   - clicking a trigger on the right should focus and highlight the matching dialog block in gold inside the lower journal;
   - do not replace the whole journal with one isolated dialog by default.
5. Define ordering rules for secondary / optional stages.
   - authored primary chain keeps its route;
   - secondary stages without strict order should enter the lower journal by first visit / first activation order.
6. Mirror the same route/history model in the quest journal.
   - the journal should reuse the same trigger-route + scrolling dialog-history paradigm;
   - do not invent a different interaction model for the journal.
7. Extend JSON/editor and then re-run live validation.
   - add human-readable item display names;
   - add explicit player-facing stage goal / presentation fields where needed;
   - re-check potato flow, Rasputin/Voron dialog ownership, and the rebuilt full route UX.

## Immediate Live Checklist

1. Rebuild `Silver_77_Quests_Client`.
2. Rebuild `Silver_77_Quests_Server` or refresh the live profile quest JSON from the accepted root baseline.
3. Confirm `quest_hunter_2` gives `PotatoSeed x12`.
4. Confirm Rasputin blocks completion until the `PotatoSeed x12` is actually deposited.
5. Confirm Voron accepts `PotatoSeed` for the large potato quest.
6. Confirm the current NPC dialog is no longer duplicated in the upper description body and is ready to live in the one shared lower scrolling dialog journal.
7. Confirm the main description now stays focused on quest state / goals / NPC context, without dialog text or the old inline chain block.
8. Confirm Rasputin's dialog owner is correct for his stage.
9. Confirm the quest remains visible on all participating chain NPCs after acceptance.
10. Confirm the temporary route prototype remains readable until the final right-side trigger column replaces it.

## Update 2026-05-01 - accept-flow stabilization + final UX contract locked

1. User feedback after the first NPC-chain-panel pass:
   - `quest_fisherman_2` / `Поставка медицины` sometimes felt like it needed repeated clicks to accept;
   - the top body of the NPC window still looked like a second dialog copy.
2. The quest data was re-checked first:
   - no obvious broken requirement chain was found in the accepted JSON baseline for `quest_fisherman_2`;
   - the issue looked more like interaction drift / feedback than a clearly invalid quest definition.
3. A first code-level stabilizer has already been added on the client:
   - `MissionGameplay.OnUpdate()` now stops re-running trigger focus checks while the quest UI or journal is open;
   - this keeps the current trigger / quest context stable for the active interaction instead of letting it drift every 0.5 seconds.
4. The NPC window also stopped injecting `quest.description` into the upper description body:
   - live dialog is intended to stay in the dedicated lower dialog block only.
5. User then clarified the final direction more strictly:
   - every window area must have one hard responsibility only;
   - no mixed meaning blocks should remain.
6. Final NPC-window contract from the user discussion:
   - left = quest list only;
   - center = status / objective / progress / reward;
   - right = trigger route;
   - bottom = one shared scrolling dialog journal.
7. Trigger interactions should work like this:
   - clicking a trigger on the right should highlight the related dialog/history block in gold;
   - the lower journal should keep the whole history visible instead of replacing it with a popup-first view.
8. Secondary / optional stages do not currently have strict authored order:
   - for now they should appear in history in first visit / first activation order.
9. Important presentation rule for the lower journal:
   - the player should see “what matters now” first;
   - current stage belongs at the top of the lower journal;
   - older stages remain below it as past history.
10. Current code state already in place:
   - the temporary left-side `ChainPanel` prototype has been replaced by a real right-side `TriggerRouteListbox`;
   - `quest.description` is no longer shown in the upper description body;
   - trigger-context refresh is frozen while quest UI/journal is open;
   - player progress now includes `stageVisits` and the server records stage activation order on `offer`, `completion`, and `reward`;
   - the lower NPC dialog block now starts from a journal-builder that reads `stageVisits` and appends the current live stage if it is not yet recorded;
   - selecting a route entry on the right now changes the focus context for the lower journal.
11. Still unresolved / needs rebuild + live retest:
   - repeated-click feeling on `quest_fisherman_2` / `Поставка медицины`;
   - the current lower dialog block is not yet a true shared scrolling history widget;
   - the right-side trigger route is now implemented, but journal focus is still a text-marker (`>>`) rather than true gold-colored in-text highlighting;
   - old active profiles cannot magically reconstruct past dialog history before `stageVisits` existed;
   - current journal output still covers dialog text only, not richer per-stage reward/goal payloads yet;
   - NPC window geometry was just re-separated into distinct description / route / dialog panels and needs a first live visual pass.
12. Important test note for the next session:
   - if the next live pass is specifically about dialog-history order, clear `profiles\Silver_77_Quests\players\*` first;
   - only clear `profiles\Silver_77_Quests\Silver_77_Quests.json` if quest content defaults themselves also need a fresh rebuild/regeneration.

## Update 2026-05-01 - NPC panels separated for cleaner live UX

1. `QuestMenu.layout` now separates the NPC window into:
   - central description panel;
   - right-side trigger route panel;
   - lower dialog panel.
2. This removes the old physical overlap between route and dialog areas, which was making live screenshots harder to interpret.
3. This is still a midpoint, not the finish line:
   - the lower journal is still a multiline text block, not a true scrolling widget;
   - route focus still uses a text marker (`>>`) instead of true gold in-text highlighting.
4. Important clarification from the latest live test:
   - the server appears to accept the needed partial hand-in amount correctly;
   - extra items are left in the inventory once the required amount is already consumed;
   - the actual broken layer is the client-side progress refresh after partial turn-in, because the UI keeps showing `0 / N`, leaves the submit button active, and falls into `waiting for server` timeout.
5. Diagnostic layer already added after that clarification:
   - client `QuestClientRPC.c` now logs received player-progress payload length and decoded quest-entry count;
   - if synced player data arrives with empty `steamId`, the client now applies a local identity fallback before `ApplySyncedPlayerData(...)`;
   - `QuestClientManager.ApplySyncedPlayerData(...)` now logs non-zero objective-progress count and stage-visit count for the applied payload.

## Update 2026-05-01 - static NPC UI text encoding and panel contrast pass

1. `QuestMenu.layout` was rewritten and then explicitly saved as `windows-1251`.
2. Goal of this pass:
   - remove the live `mojibake` from static NPC window labels/buttons;
   - make description / route / dialog zones read as clearly separated blocks.
3. Concrete visual changes:
   - darker panel backgrounds;
   - brighter `ОПИСАНИЕ КВЕСТА` label;
   - gold-toned `МАРШРУТ` and `ДИАЛОГОВЫЙ ЖУРНАЛ` labels.
4. This is a UI-only step:
   - no profile reset needed for this check;
   - no quest-state or deposit logic changed here.
5. Still open after this pass:
   - client-side partial hand-in refresh bug;
   - no real scroll widget for the lower journal yet;
   - route focus still uses `>>` instead of gold in-text highlight;
   - button downsizing / extra right-side helper textbox still pending.
