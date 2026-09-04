# CODEX SESSION HANDOFF 2026-04-26

## Зачем этот файл

Это короткий handoff после сессии, где был завершен переход редактора и логики квестов на role-driven модель `offer / completion / reward`.

## Update 2026-04-26 - финальная схема Offer / Completion / Reward

- Базовая схема теперь простая:
  - `Offer` обязателен: где игрок берёт квест.
  - `Reward` обязателен: где квест финально закрывается и выдаётся награда.
  - `Completion` опционален: один или несколько промежуточных этапов передачи / сдачи.
- Если Completion есть, Reward не закрывает квест и не выдаёт награду, пока все Completion-trigger не выполнены.
- Если Completion нет, Reward принимает цели напрямую и закрывает квест.
- У каждого действия теперь свой диалог в `triggerActions`:
  - `actionType = "offer"`
  - `actionType = "completion"`
  - `actionType = "reward"`
- В split runtime добавлен прогресс `completedCompletionTriggerIds`, чтобы сервер и клиент знали, какие промежуточные NPC уже пройдены.
- Старые profile JSON / историю персонажа можно пересоздавать; обратную совместимость не считать приоритетом.

## Что уже сделано

- Починен запускатор редактора в `P:\Silver_77_Quests\Support\JSON_Quvest`.
- Старые запускаторы в `P:\Silver_77_Quests\JSON_Quvest` теперь работают как обертка и должны вести в актуальную папку.
- Для редактора добавлены anti-cache / versioned assets, чтобы не открывалась старая страница из кэша.
- Проверено, что `http://127.0.0.1:4173/api/health` отвечает `{"ok":true}`.
- Проверено, что сервер редактора уже отдает обновленный `app.js`.

## Что изменилось в модели квестов

- Отдельную ручную `видимость` больше не считать основной моделью.
- `trigger.questIds` теперь трактовать как:
  - список квестов у NPC
  - порядок квестов у NPC
- Игровая видимость считается по ролям и состоянию квеста:
  - `not_started` -> `offerTriggerIds`
  - `active` -> невыполненные `completionTriggerIds`
  - `active` без Completion или после всех Completion -> `rewardTriggerIds`
  - `reward_pending` -> `rewardTriggerIds`
- Основные поля роли:
  - `offerTriggerIds`
  - `completionTriggerIds`
  - `rewardTriggerIds`
  - `hideUntilRequirementsComplete`

## Новый каркас действий по trigger

- В квесте добавлен `triggerActions`.
- Один action хранит:
  - `triggerId`
  - `actionType`
  - `dialogText`
  - `rewards`
- Сейчас action-типы:
  - `offer`
  - `completion`
  - `reward`
- Стартовый диалог теперь тоже action-диалог `offer`; старое `description` остаётся fallback / старым текстом квеста.
- Старые глобальные `turnInDialogText` / `rewardDialogText` больше не считать активной моделью.

## Что изменилось в редакторе

- В левой колонке уже работает фильтр квестов по trigger / NPC.
- Список квестов внутри trigger идет в порядке `trigger.questIds`.
- `NPC Flow` теперь строится как набор универсальных NPC-блоков.
- По умолчанию в `NPC Flow` показываются только активные блоки, уже участвующие в квесте.
- Новый NPC теперь можно открыть прямо внутри `NPC Flow` через встроенный picker `Открыть / добавить NPC`.
- После включения роли `offer / completion / reward` выбранный NPC автоматически попадает в активные блоки.
- Для обзора и точечной настройки в `NPC Flow` есть режимы:
  - `Активные блоки`
  - `Выбранный NPC`
  - `Все NPC`
- Черновики редактора теперь сравниваются по `updatedAt`:
  - свежий localStorage-черновик больше не должен проигрывать старому файловому
  - legacy-черновики без `updatedAt` больше не должны автоматически оживать
- В карточке квеста основной сценарий теперь:
  - где взять
  - кому сдать
  - где получить награду
- Для `offer`, `completion` и `reward` уже есть отдельные action-карточки:
  - диалог для конкретного NPC
  - локальный список наград для конкретного NPC
- Общая награда квеста остается как fallback по умолчанию.

## Что сделано в рантайме мода

- На клиенте и сервере уже заложена role-driven видимость по статусу.
- Уже есть статус `reward_pending`.
- Уже есть логика отдельного reward trigger.
- Completion-этапы теперь отмечаются в прогрессе игрока, а Reward проверяет эту цепочку перед закрытием.
- Уже есть желтая подсветка для reward-сценария из предыдущей ветки.
- Клиент и сервер теперь автоматически досеивают роли из старых `trigger.questIds`, если в JSON еще нет новых role-полей.
  - это важно для дефолтного списка старых квестов
  - смысл старого набора должен сохраниться даже после перехода на новую схему

## Что пользователь явно разрешил

- Не тратить силы на обратную совместимость со старой пользовательской историей персонажей и старым profile JSON, если это мешает новой модели.
- При необходимости пользователь готов:
  - удалить старые квесты
  - пересоздать server profile JSON
  - пересоздать историю персонажей
- Но дефолтный список квестов на базе уже известных старых квестов должен остаться и быть понятным.

## Что еще не доведено до финала

- Не выполнен полноценный игровой тест после последних правок.
- Не собраны новые PBO после этого конкретного role-driven апдейта.
- Логика "именно тот самый конкретный предмет, который выдали" еще не реализована как отдельная идентичность экземпляра.
  - сейчас маршрут `A -> B -> reward` строится на логике ролей и целевых предметов
  - но не на уникальном instance-id выданного предмета

## Ключевые файлы этой ветки

- `P:\Silver_77_Quests\Support\JSON_Quvest\app.js`
- `P:\Silver_77_Quests\Support\JSON_Quvest\index.html`
- `P:\Silver_77_Quests\Support\JSON_Quvest\server.ps1`
- `P:\Silver_77_Quests\Support\JSON_Quvest\start-editor.ps1`
- `P:\Silver_77_Quests\Support\JSON_Quvest\editor-config.local.json`
- `P:\Silver_77_Quests\JSON_Quvest\start-editor.ps1`
- `P:\Silver_77_Quests\JSON_Quvest\start-editor.cmd`
- `P:\Silver_77_Quests\Silver_77_Quests_Client\scripts\3_Game\QuestData.c`
- `P:\Silver_77_Quests\Silver_77_Quests_Client\scripts\4_World\QuestClientManager.c`
- `P:\Silver_77_Quests\Silver_77_Quests_Client\scripts\5_Mission\QuestUI.c`
- `P:\Silver_77_Quests\Silver_77_Quests_Server\scripts\4_World\QuestServerManager.c`
- `P:\Silver_77_Quests\Silver_77_Quests_Server\scripts\4_World\QuestServerRPC.c`

## Что проверять следующим шагом

1. Пересобрать оба PBO:
   - `Silver_77_Quests_Client`
   - `Silver_77_Quests_Server`
2. Протестировать в игре один обычный дефолтный квест из старого списка.
3. Протестировать маршрут `A -> B -> reward`.
4. Проверить:
   - берется ли квест только у нужного NPC
   - сдается ли только у нужного NPC
   - переходит ли в `reward_pending`, если есть Completion и награда в другом месте
   - закрывается ли квест сразу у Reward, если Completion нет
   - есть ли желтая подсветка у reward trigger
   - показываются ли offer / completion / reward диалоги у нужного NPC
5. После игрового теста продолжать courier / delivery логику дальше.

## Что не забыть в новой сессии

- Если менялись DTO / RPC / UI / server logic, клиент и сервер нужно обновлять вместе.
- Если на сервере уже есть `profiles\Silver_77_Quests\Silver_77_Quests.json`, новые дефолты из кода сами не применятся.
- Если редактор вдруг снова открывает старую страницу, первым делом проверять:
  - что запуск шел не из старой копии
  - что страница не сидит на кэше
  - что `http://127.0.0.1:4173/app.js?...` отдает обновленный скрипт

## Hotfix 2026-04-26 - quest sync crash

- Client crash report showed `String CORRUPTED - FIX OnStoreLoad() !!!` during `questclientrpc.c` when reading synced quest config.
- After the role-driven schema expansion, binary RPC transport became too brittle for the synced quest payload.
- Runtime sync was hardened:
  - server sends quest config as JSON string payload;
  - server sends player progress as JSON string payload;
  - client writes the JSON payload into temp files in `$profile:Silver_77_Quests` and loads them via `JsonFileLoader`.
- Files touched:
  - `P:\Silver_77_Quests\Silver_77_Quests_Client\scripts\4_World\QuestClientRPC.c`
  - `P:\Silver_77_Quests\Silver_77_Quests_Server\scripts\4_World\QuestServerManager.c`
- Build warning:
  - `build_split_mods.bat` currently may still print `[OK]` after `AddonBuilder` fatal `ArgumentNullException`;
  - next session should either fix that build step or verify actual PBO timestamps / fresh deployment manually.

## Follow-up 2026-04-26 - config payload was still too large

- Fresh client log after the first hotfix still showed `QuestTriggerManager initialized with 0 triggers` and `Failed to read quest config sync RPC`.
- Player progress sync was arriving, so the issue narrowed to quest config transport size/format.
- Config sync is now chunked:
  - server sends `SILVER77_QUEST_RPC_CONFIG_DATA` as `Param3<int, int, string>` chunks;
  - client reassembles those chunks and then loads quest config JSON.
- Template alignment also done:
  - `Support/JSON_Quvest/Silver_77_Quests.json` -> `version 3`
  - `Documentation/STARTER_QUEST_CONFIG.json` -> `version 3`
  - role fields are explicit in both template JSON files.
- Build script note changed:
  - `build_split_mods.bat` now correctly fails on AddonBuilder `[FATAL]`;
  - current known issue is no longer false success, but the underlying AddonBuilder `ArgumentNullException` itself, which may still need separate fixing.

## Editor UX update 2026-04-26 - trigger-first quest flow cards

- Quest editor no longer has to be mentally read as:
  - one global block with trigger role checkboxes
  - another separate global block with NPC action details
- New intended reading model in `Support/JSON_Quvest`:
  - one NPC / trigger card
  - role checkboxes inside that card
  - detailed settings open directly below the same NPC card
- Visual change:
  - trigger cards alternate between two accent tones for readability
  - offer / completion / reward detail blocks stay visually attached to the same card
- Important scope note:
  - this was an editor presentation / workflow change only
  - runtime schema still uses the same role arrays plus `triggerActions`
- Offer-stage note:
  - offer dialog is now per NPC through `triggerActions`
  - giveItems / objectives still remain quest-level settings shown inside the Offer block

## Session cutoff 2026-04-26 - latest product decisions and unfinished editor work

- Final quest rule fixed by the user:
  - `Offer` is mandatory: where the quest is taken.
  - `Reward` is mandatory: final closure point and reward issuing point.
  - `Completion` is optional: zero, one, or several intermediate handoff stages.
- Runtime rule fixed by the user:
  - `Reward` must be the final closer.
  - if there is at least one `Completion`, `Reward` must not close the quest and must not give the final reward until every `Completion` trigger is completed;
  - if there are no `Completion` stages, `Reward` may accept the quest objectives directly and finish the quest.
- Each role has its own NPC dialog:
  - `offer` dialog when taking the quest;
  - `completion` dialog on intermediate handoff;
  - `reward` dialog before final reward.
- Important user clarification:
  - even intermediate `Completion` stages may take something from the player and may issue something as part of the chain;
  - but final quest closure should still happen only through `Reward`.
- Editor UX target is now stricter than the first trigger-flow revision:
  - the editor should operate with universal NPC blocks;
  - in every block the user must be able to choose any needed NPC / trigger for that exact block;
  - the top picker `Открыть / добавить NPC` is not enough by itself;
  - every concrete card must also have its own local trigger picker (`NPC / Trigger этого блока`).
- Intended editor reading model:
  - block 1: choose who gives the quest and enable `Offer`;
  - block 2..N: choose who receives intermediate handoff(s) and enable `Completion`;
  - final block: choose who closes the quest and enable `Reward`;
  - even if giver and closer are the same NPC, both roles should still be configured explicitly.
- User also wants the old standalone blocks to stop being the mental center of the editor:
  - `Give Items`
  - `Objectives`
  - `Rewards`
  Their logic should increasingly live under the role-driven NPC blocks, according to enabled checkboxes.
- Current honest state at session cutoff:
  - role arrays and `triggerActions` are already in place;
  - split client/server runtime already tracks `completedCompletionTriggerIds`;
  - left quest filter by trigger/NPC already works;
  - top `NPC Flow` picker exists;
  - per-card picker text and base rendering exist in `app.js`;
  - but the final universal block workflow still needs an in-editor verification pass and likely more cleanup in `app.js`.
- Path / file rule for the editor is fixed and should not be rediscovered again:
  - editor must resolve paths relative to its own folder `P:\Silver_77_Quests\Support\JSON_Quvest`, not relative to launch `cwd`;
  - working quest JSON: `P:\Silver_77_Quests\Support\JSON_Quvest\Silver_77_Quests.json`;
  - visible backup next to it: `P:\Silver_77_Quests\Support\JSON_Quvest\Silver_77_Quests_BackUP.json`;
  - file draft: `P:\Silver_77_Quests\Support\JSON_Quvest\editor-draft.json`;
  - legacy `.local` content is historical and should not become the main working source again.
- Backward compatibility rule remains unchanged:
  - old player history / old profile JSON are not a priority if they block the new model;
  - but the default quest set should still preserve the known meaning of the old starter quest list.
- Still not implemented as a separate mechanic:
  - unique identity tracking of the exact issued item instance ("bring back the exact item that was given"), as opposed to role-based delivery logic.

## Update 2026-04-29 - stricter editor rule than the 26.04 pass

- The product model became stricter after this handoff:
  - one `NPC Flow` block = one role
  - no mixed `Offer + Reward` block
  - no second `Offer`
  - `Reward` is also unique and mandatory
  - `Completion` may be many
- The same NPC / trigger may still appear several times inside one quest, but only as different separate blocks with different roles.
- `Completion` semantics were clarified further:
  - every Completion has its own dialog
  - every Completion may accept required items
  - every Completion may issue its own local reward
  - Completion must not close the quest finally
- `Reward` semantics were clarified further:
  - Reward is the only final closer
  - Reward must not close/pay out until Offer requirements and all Completion stages are satisfied
- The user explicitly allowed reshaping JSON / draft structure if needed for a cleaner editor model.
- Treat this 26.04 handoff as architectural background, but treat `CODEX_SESSION_HANDOFF_2026-04-29.md` as the fresher product override.
