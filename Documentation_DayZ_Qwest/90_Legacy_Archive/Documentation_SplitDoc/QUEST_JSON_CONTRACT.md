# QUEST JSON CONTRACT

Короткий агентский контракт между модом, JSON-квестами, UI и редактором.

## 1. Источник правды

Мод является источником правды для JSON-контракта.

Редактор, UI и вспомогательные инструменты должны подстраиваться под то, что реально читает мод.

Расширенный логический документ проекта:

- `Documentation/QUEST_LOGIC_SPEC.md`

Этот `SplitDoc` нужен как короткая operational-версия для агентов.

## 2. Главная модель квеста

Каждый квест может иметь:

- `requiredQuestIds[]`
- `Offer`
- `Completion[]`
- `Reward`

Рабочая модель:

```text
один Offer -> несколько Completion -> один Reward
```

Текущая runtime-модель:

- `Offer` и `Reward` фактически single-trigger роли;
- `Completion` может быть несколько.

## 3. Реальные поля, которые надо учитывать

Ключевые поля:

- `quest.description`
- `requiredQuestIds[]`
- `objectives[]`
- `rewards[]`
- `offerTriggerIds[]`
- `completionTriggerIds[]`
- `rewardTriggerIds[]`
- `triggerActions[]`
- `triggerActions[].dialogText`

## 4. Чего в контракте нет

Не вводить ложные поля.

В контракте нет отдельного поля:

- `requiredItems`
- `dialogue`
- `dialogueText` как отдельной сущности вне `triggerActions[]`
- `rewardBlock`
- `completionBlock`
- `offerBlock`

Роль `requiredItems` выполняют:

- `objectives[]`
- особенно `objectives[]` с `type == "item"`

NPC-текст хранится в:

- `triggerActions[].dialogText`

## 5. Семантика ролей

`Offer`:

- место / NPC, где игрок берёт квест;
- стартовый диалог;
- может выдать стартовые предметы через `giveItems`;
- не является финальным закрытием квеста.

`Completion`:

- промежуточный этап;
- может быть `0`, `1` или несколько;
- может иметь свой `triggerActions[].dialogText`;
- может принимать objectives;
- может выдавать промежуточную награду через `triggerActions[].rewards`.

`Reward`:

- финальное закрытие квеста;
- выдаёт итоговую награду;
- переводит квест в `completed`.

## 6. Выдаваемые предметы

Выдаваемый item используется в:

- `giveItems[]`;
- `rewards[]`;
- `triggerActions[].rewards[]`.

Поля выдаваемого item:

- `className` — какой предмет создать;
- `quantity` — сколько физических предметов / стаков создать;
- `spawnOnGround` — создавать на земле или сначала пробовать инвентарь;
- `setItemQuantity` — нужно ли выставить внутреннее количество созданного предмета;
- `itemQuantity` — какое внутреннее количество поставить каждому созданному предмету.

Пример: `quantity = 1`, `setItemQuantity = 1`, `itemQuantity = 5` для `Ammo_12gaPellets` означает один физический стак с количеством 5.

`itemQuantity` относится только к выдаваемым предметам и не заменяет objective-настройку `useItemQuantity`.

## 7. Objective-правила

Для предметных целей ориентироваться на:

- `objectives[]`
- `quantity`
- `useItemQuantity`
- `allowPartialTurnIn`
- `removeOnComplete`

Не выдумывать отдельную сущность "список обязательных предметов", если речь идёт о runtime JSON.

`useItemQuantity` в objective используется для проверки и сдачи предметов игроком. `itemQuantity` в выдаваемом item используется только при создании награды / стартового предмета.

## 8. Связь с UI

Смысловые данные делятся так:

- описание, цели, прогресс, статус, награды -> `DescriptionPanel`
- NPC-диалог текущего действия -> `DialogPanel`

`triggerActions[].dialogText` не должен подменять основное описание квеста.

Подробные UI-правила см. в:

- `Documentation/SplitDoc/QUEST_UI_RULES.md`

## 9. Риск дублей

В проекте уже есть более широкий документ:

- `Documentation/QUEST_LOGIC_SPEC.md`

Если между ним и этим `SplitDoc` появится расхождение, это нужно считать риском двойного канона и выносить в отдельную синхронизационную задачу.
