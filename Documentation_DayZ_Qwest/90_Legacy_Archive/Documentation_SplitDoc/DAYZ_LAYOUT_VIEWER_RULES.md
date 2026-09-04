# DAYZ LAYOUT VIEWER RULES

Правила для `DayZ_layout` и read-only viewer-а `.layout` файлов.

## 1. Роль viewer-а

`DayZ_layout` — отдельный инструмент диагностики.

Viewer должен быть `read-only`.

Он может:

- читать выбранный `.layout`;
- декодировать файл в нужной кодировке;
- парсить его;
- строить дерево узлов;
- считать итоговые прямоугольники;
- показывать `visual render`;
- показывать `inspector`;
- запускать `sanity checks`;
- копировать диагностическую информацию.

Он не должен:

- сохранять `.layout`;
- форматировать `.layout`;
- менять кодировку исходного файла;
- генерировать layout-код;
- писать в файлы мода;
- менять JSON.

## 2. Формат DayZ .layout

`.layout` в проекте не является XML.

Нельзя:

- использовать `DOMParser`;
- считать формат XML-деревом;
- ожидать `position[] = {...}` или `size[] = {...}`.

Реальный формат:

- начало виджета выглядит как `<WidgetClassType> <ElementName> {`;
- свойства идут отдельными строками;
- отдельная строка `{` открывает child-group;
- отдельная строка `}` закрывает текущий block.

## 3. Parser и blockStack

Для корректного парсинга нужен `blockStack`, а не простой `nodeStack`.

Стек должен различать:

- `node block`
- `group block`

При поиске текущего `parent` брать последний `blockStack item` с `kind === "node"`.

## 4. Final rect и exact flags

Layout нельзя считать только как:

```text
absX = parent.absX + position.x
absY = parent.absY + position.y
```

Правильная модель:

```text
parentRect + local position + local size + align + exact flags = finalRect
```

Принятое правило:

- `exact = 1` -> значение в пикселях
- `exact = 0` -> значение как доля `parentRect`

`parseLayout()` должен только читать и строить дерево.

`computeFinalRects()` должен считать итоговые прямоугольники.

`renderLayout()` должен только рисовать уже готовый `finalRect`.

## 5. Viewport и renderScale

`center_ref` зависит от размера viewport.

Viewer должен поддерживать presets, например:

- `1000x600`
- `1280x720`
- `1366x768`
- `1600x900`
- `1920x1080`
- `2560x1440`

`renderScale` влияет только на отображение в браузере и не должен менять логику `finalRect`.

## 6. Text widgets и z-order

Нужно различать:

- debug label `node.name`
- игровой текст `node.text`

Для `TextWidgetClass` и `MultilineTextWidgetClass` по возможности использовать:

- `node.text` как основной текст;
- `text halign`;
- `text valign`;
- размер шрифта из `metronXX`, если он явно читается.

Нужен стабильный `z-order`:

- parent ниже child;
- text обычно поверх панели/кнопки;
- без хаотичного перекрытия.

## 7. Inspector и sanity checks

Viewer должен помогать диагностике, а не гаданию по картинке.

Полезно показывать:

- `name`
- `type`
- `parentName`
- `position`
- `size`
- `finalRect`
- `halign / valign`
- `hexactpos / vexactpos`
- `hexactsize / vexactsize`
- `text`
- `font`
- `props`

## 8. Encoding внутри viewer-а

Для `.layout` файлов проекта нельзя считать одну кодировку универсальной.

Viewer должен позволять вручную выбирать хотя бы:

- `UTF-8`
- `Windows-1251`

При смене encoding нужно заново декодировать исходные bytes, а не пытаться чинить уже испорченную строку.

Подробные общие правила кодировок см. в:

- `Documentation/SplitDoc/ENCODING_RULES.md`

## 9. Реальные файлы для read-only проверки

Допустимые входы для диагностики viewer-а:

- `P:\Silver_77_Quests\Silver_77_Quests_Client\gui\QuestMenu.layout`
- `P:\Silver_77_Quests\Silver_77_Quests_Client\gui\QuestJournal.layout`

Использовать их можно как input для viewer-а, но не менять без отдельной задачи.

## 10. Актуальность viewer при UI/layout-изменениях

`DayZ_layout viewer` должен оставаться рабочим диагностическим инструментом после UI- и layout-правок, а не только после отдельных viewer-задач.

Если в задаче меняются:

- `QuestMenu`
- `QuestJournal`
- `.layout`
- `UIScriptedMenu`
- `widget classes`
- `styles`
- `scroll`-паттерны
- размеры, позиционирование или структура UI-блоков

агент обязан проверить, сможет ли viewer корректно разобрать и показать такие изменения.

Если viewer не поддерживает новый элемент, новый паттерн или новую структуру:

- не менять viewer вне разрешённого scope;
- указать ограничение в `PROBLEMS`;
- предложить отдельную задачу на обновление `DayZ_layout`.

Если текущая задача прямо разрешает менять viewer вместе с UI, тогда viewer нужно обновлять в той же задаче, а не оставлять заведомо устаревшим.
