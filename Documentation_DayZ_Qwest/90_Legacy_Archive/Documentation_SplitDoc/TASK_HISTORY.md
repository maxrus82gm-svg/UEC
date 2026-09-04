# TASK HISTORY

История завершённых и зафиксированных задач, вынесенная из `AGENT_TASK_LOOP.md`.

## Подтверждённая история

- `TASK 001` — Проверка зависимости клиент/сервер — принято.
- `TASK 002` — Проверка соответствия имени клиентского мода в `CfgPatches` — принято.
- `TASK 003` — Инвентаризация дубликатов монолитной и split-версии — принято с уточнением.
- `TASK 004` — Проверка содержимого резервных и старых папок без изменений — принято.
- `TASK 005` — Проверка корневых `config.cpp` и `mod.cpp` — принято.
- `TASK 006` — Проверка корневой папки `gui` — принято с замечанием по кодировке.
- `TASK 007` — Проверка кодировки файлов с кириллицей — принято.
- `TASK 008` — Поиск нормальной версии `QuestMenu.layout` — принято.
- `TASK 009` — Сравнение `QuestMenu.layout` перед восстановлением — принято.
- `TASK 010` — Составление карты повреждённых строк `QuestMenu.layout` — принято.
- `TASK 011` — Попытка восстановления text-строк — остановлено безопасно из-за риска кодировки.
- `TASK 012` — Выбор безопасного способа восстановления `QuestMenu.layout` — принято.
- `TASK 013` — Подготовка безопасного скрипта восстановления `QuestMenu.layout` — принято предварительно.
- `TASK 014` — Code review скрипта восстановления `QuestMenu.layout` — принято.
- `TASK 015` — Финальная проверка кода `restore_questmenu_text.py` перед запуском — принято.
- `TASK 016` — Ручной запуск `restore_questmenu_text.py` и проверка результата — принято.
- `TASK 017` — Проверка восстановленного `QuestMenu.layout` без изменений — принято.
- `TASK 018` — Уборка временных Python/cache/backup файлов после восстановления — принято.
- `TASK 019` — Проверка результата восстановления `QuestMenu.layout` в проекте без изменений — принято.
- `TASK 020` — Диагностика `CloseButton` и `CloseButtonText` без изменений — принято.
- `TASK 021` — Подключение `CloseButtonText` в `QuestUI.c` — принято.
- `TASK 022` — Сравнение стилей `CloseButton` с рабочими кнопками — принято.
- `TASK 023` — Включение `CloseButton` и `CloseButtonText` в `UpdateButtons` — принято.
- `TASK 024` — Ручная проверка `CloseButton` в игре — принято.
- `TASK 059` — Диагностика `OnRPC case` без вызова `Silver77_HandleQuestPlayerData(ctx)` — принято как диагностический шаг.
- `TASK 060` — Перенос `ctx.Read` player data прямо в `OnRPC case` — принято как архитектурная правка.
- `TASK 061` — Диагностическая проверка `Param1<string>` вместо `Param3` — временный тест, позже откатан.
- `TASK 063` — Возврат player data RPC на `Param3` после диагностики — принято.
- `TASK 064` — Player data RPC переведён на `chunked sync` как config sync — принято. Исправлена ошибка `String CORRUPTED - FIX OnStoreLoad()`. В игре подтверждено: UI получает реальные статусы квестов и показывает `УЖЕ АКТИВЕН`.
- `TASK 065` — DayZ layout viewer — начата разработка отдельного read-only просмотрщика `.layout` файлов в `DayZ_layout/`.
- `TASK 065 FIX 1–4` — несколько неудачных подходов к viewer: plain text viewer, XML/DOMParser, неверный формат `position[] / size[] / color[]` — отклонено.
- `TASK 065 FIX 5` — парсер переведён на stack-подход для вложенных виджетов — частично принято как промежуточный шаг.
- `TASK 065 FIX 6` — добавлено ядро дерева `parent/children` и `absX/absY` — частично принято как шаг по ядру.
- `TASK 065 FIX 7` — принято. Ядро DayZ layout viewer доведено: `parentId/parentName`, `text/font/props`, безопасный `absX/absY`, подписи `node.name`, transparent border без `color`.
- `TASK 078` — выполнена, принята условно. Синхронизирован актуальный JSON из редактора в server profile с backup. Расширен `QuestMenu`: `QuestPanel 980x560`, `QuestListbox 250x450`, `DescriptionPanel/DialogPanel` шире, `RoutePanel 210x334`. Убраны длинные префиксы и `[F]` из квестового UI, `DialogPanel` показывает текущий `triggerActions[].dialogText`, фон стал плотнее, `QuestMenu.layout` приведён к `UTF-8` без BOM. Полноценный scroll для `DescriptionPanel/DialogPanel` не реализован и остаётся возможной отдельной задачей.
- `TASK 079` — принято. Проведена аналитика по разделению `AGENT_TASK_LOOP.md` на отдельные rule-документы. Подтверждено, что loop-файл перегружен и должен стать коротким рабочим shell-документом. Принята структура `SplitDoc`: `AGENT_RULES`, `ENCODING_RULES`, `DAYZ_RPC_SYNC_RULES`, `QUEST_EDITOR_RULES`, `QUEST_JSON_CONTRACT`, `QUEST_UI_RULES`, `DAYZ_LAYOUT_VIEWER_RULES`, `SERVER_PROFILE_RULES`, `TASK_HISTORY`.
- `TASK 080` — выполнена агентом, ожидает review. Создана структура `SplitDoc`, постоянные правила вынесены по темам, `AGENT_TASK_LOOP.md` сокращён до рабочего shell-документа. Зафиксированы новые правила о том, как ChatGPT формирует `БЛОК 1` и как агент при разрешении задачи обновляет `AGENT REPORT`, `REVIEW / STATUS`, `TASK_HISTORY` и новые правила.
- `TASK 101` — принято после runtime-проверки пользователя. Commit baseline: `2fc3212f8a381efd7d33bdd60ff2491f5b5e3800`. Для `QuestMenu` выбран компромиссный рабочий вариант: `Description/Dialog` увеличены по вертикали, `Scrollbar V` отключён, стек остаётся `ScrollWidget -> MultilineTextWidget`, `QuestUI.c` не менялся. `wrapper/mask/SizeToChild/RichText/EditBox` не используются. Известное ограничение: очень длинный текст может обрезаться; тема `scroll/clipping` отложена.
- `TASK 102` — выполнена как документационная фиксация runtime-accepted baseline после `TASK 101`. В `AGENT_TASK_LOOP.md` и `TASK_HISTORY.md` записано, что commit `2fc3212f8a381efd7d33bdd60ff2491f5b5e3800` принят как текущий рабочий вариант `QuestMenu`.
- `TASK 103` — выполнена документационная правка. `AGENT_RULES.md` переименован/заменён на `START.md`; `START.md` теперь главный стартовый документ правил. Добавлены правила Git review: после отчёта Codex REVIEW должен начинаться с `СТАТУС КОММИТА`, commit message указывается как есть, неподтверждённый message отмечается честно, ассистент не утверждает, что видит локальные изменения, а проверяет GitHub/push. В правило добавлен рабочий путь проекта `P:\Silver_77_Quests`.
- `TASK 104` — выполнена документационная правка. В `START.md` в базовый формат `AGENT REPORT` добавлен обязательный пункт `RULES PATH` с путём к основным правилам и `SplitDoc`: `P:\Silver_77_Quests\Documentation`.
- `TASK 105` — выполнена документационная правка. Подтверждено, что `RULES PATH` с путём `P:\Silver_77_Quests\Documentation` остаётся в каноническом формате отчёта `START.md`; верх `AGENT_TASK_LOOP.md` сжат до ссылки на `START.md` и сохранённой матрицы чтения `SplitDoc`.
- `TASK 106` — выполнена документационная правка. В `START.md` добавлено правило выборочного чтения: ChatGPT выбирает минимальный набор тематических `SplitDoc`, а агент не читает все `SplitDoc` подряд. Подробные Git/REVIEW правила вынесены в новый `Documentation/SplitDoc/GIT_REVIEW_RULES.md`; в `START.md` оставлена короткая ссылка и базовое правило, что Git контролирует пользователь.
- `TASK 107` — выполнена подготовка Obsidian-перехода. База `Documentation_DayZ_Qwest` нормализована: старые упоминания `DayZ_Qwest` исправлены, создан раздел агентских правил и подготовлены зеркала `AGENT_TASK_LOOP` / `START`. Старые рабочие документы `Documentation/AGENT_TASK_LOOP.md` и `Documentation/SplitDoc/START.md` остались источником правды и физически не переносились.
- `TASK 109` — выполнен перенос нужных рабочих документов в Obsidian как полных зеркал. В `Documentation_DayZ_Qwest/20_SplitDoc` скопированы нужные правила из `Documentation/SplitDoc`, в `Documentation_DayZ_Qwest/21_Project_Docs` скопированы важные проектные документы из `Documentation`; старые оригиналы не удалялись и система на Obsidian-путь не переключалась.

## Примечание

История остаётся рабочим журналом фактов и решений. Если какая-то задача требует отдельного повторного подтверждения в игре или review, это нужно явно указывать в формулировке записи.
