# CODEX FAST HANDOFF 2026-04-27

## Для чего этот файл

Это короткая точка входа для новой сессии, когда нельзя тратить много контекста на полное перечитывание всей истории.

## Читать в быстром режиме

1. `P:\Silver_77_Quests\Documentation\CODEX_START_HERE.md`
2. `P:\Silver_77_Quests\Documentation\CODEX_FAST_HANDOFF_2026-04-27.md`

Если после этого чего-то не хватает, уже потом открывать:

3. `P:\Silver_77_Quests\Documentation\CODEX_CONTROL_CONTEXT.md`
4. `P:\Silver_77_Quests\Documentation\CODEX_SESSION_HANDOFF_2026-04-26.md`
5. `P:\Silver_77_Quests\Documentation\CODEX_WORKLOG.md`

## Где реально остановились

- Работа сейчас всё ещё на этапе редактора, а не на этапе сборки PBO.
- Основной активный редактор:
  - `P:\Silver_77_Quests\Support\JSON_Quvest`
- Legacy-путь:
  - `P:\Silver_77_Quests\JSON_Quvest`
  считать только обёрткой.

## Что уже точно сделано

- `NPC Flow` уже переведён на новую рабочую модель:
  - верхний picker `Открыть / добавить NPC`
  - локальный picker в карточке `NPC / trigger этого блока`
  - пустой блок `Новый блок цепочки`
- В `P:\Silver_77_Quests\Support\JSON_Quvest\app.js` уже почищены старые дубли:
  - лишние старые `renderQuestEditor(...)`
  - старые мёртвые секции старого редактора
  - ранний дубль `normalizeData(...)`
- Проверено:
  - `node --check P:\Silver_77_Quests\Support\JSON_Quvest\app.js` проходит
  - `server.ps1` редактора поднимается
  - `http://127.0.0.1:4173/api/health` отвечает `{"ok":true}`
  - живая страница уже показывает новый `NPC Flow`, а не старую глобальную схему

## Что сейчас мешает

- Основная проблема на этот момент уже не в UI-каркасе `NPC Flow`, а в самих данных.
- Текущий `editor-draft.json` восстановился из старого черновика и содержит старую полусхему:
  - у части квестов пустые `offerTriggerIds`
  - у части квестов пустые `rewardTriggerIds`
  - местами `completion` ещё стоит там, где теперь должен быть `reward`
  - `trigger.questIds` и role-поля местами расходятся
- Базовый шаблон редактора тоже ещё не до конца выровнен:
  - `P:\Silver_77_Quests\Support\JSON_Quvest\Silver_77_Quests.json`

## Что делать следующим шагом

1. Не уходить сразу в сборку client/server.
2. Сначала выровнять редакторские данные под новую продуктовую схему:
   - `Offer` обязателен
   - `Reward` обязателен
   - `Completion` опционален
3. Проверить и починить:
   - `P:\Silver_77_Quests\Support\JSON_Quvest\editor-draft.json`
   - `P:\Silver_77_Quests\Support\JSON_Quvest\Silver_77_Quests.json`
4. Убедиться, что базовые дефолтные квесты теперь описаны так:
   - если квест простой `A -> A`, то у него явно есть и `offer`, и `reward`, даже если это один и тот же NPC
   - если квест цепочечный `A -> B -> reward`, то промежуточный NPC идёт через `completion`, а финальное закрытие идёт через `reward`
5. Только после этого переходить к пересборке и игровому тесту.

## Важные продуктовые правила

- `Offer` = где квест берут.
- `Completion` = промежуточная передача / этап, может быть один или несколько.
- `Reward` = финальное закрытие квеста и выдача награды.
- Если есть хотя бы один `Completion`, `Reward` не должен закрывать квест, пока все `Completion` не выполнены.
- У `offer`, `completion`, `reward` у каждого свой dialog через `triggerActions`.
- Старую пользовательскую совместимость можно не тащить, если она мешает новой модели.
- Но дефолтный список старых квестов по смыслу должен сохраниться.

## Чего не делать первым делом

- Не уходить сразу в DayZ build.
- Не тратить время на старый `M:` как на основную мастерскую.
- Не возвращаться к старой глобальной схеме `Trigger Roles / NPC Actions / Give Items / Objectives / Rewards` как к основной модели редактора.

## Update 2026-04-27 - editor data alignment already completed

- `Silver_77_Quests.json` and `editor-draft.json` were already brought in line with the current product rule:
  - `Offer` is mandatory
  - `Reward` is mandatory
  - `Completion` is optional
- Simple quests are now represented as explicit `Offer -> Reward`, even when both roles point to the same NPC.
- The draft keeps the chain example `quest_hunter_2` as `Offer -> Completion -> Reward`.
- `triggerActions` were resynced to the active roles, so role arrays and per-trigger action cards now describe the same flow.
- `trigger.questIds` were also resynced to match the active role usage in each file.
- The broken draft placeholder `quest_new` was removed.

## Next practical step after this

1. Open the current editor in `P:\Silver_77_Quests\Support\JSON_Quvest`.
2. Do not assume the role refactor is finished just because the data was aligned.
3. First verify and finish the strict block model:
   - one block = one role
   - no second `Offer`
   - no second `Reward`
   - no mixed `Offer + Reward` in one card
   - the same NPC may repeat only as another separate block with another role
4. Verify that `Completion` is a true intermediate stage with its own dialog and optional local reward, but without final quest closure.
5. Only after that move to live visibility checks, yellow reward highlight checks, and then build / in-game validation.

## Update 2026-04-29 - newest product override

- The freshest product rules are now captured in:
  - `P:\Silver_77_Quests\Documentation\CODEX_SESSION_HANDOFF_2026-04-29.md`
- Read that handoff before assuming the editor phase is complete.
- Current priority is again editor/schema hardening, not immediate PBO build.
