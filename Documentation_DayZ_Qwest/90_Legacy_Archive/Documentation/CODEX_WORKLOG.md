# Codex Worklog: Silver_77_Quests

## Update 2026-04-26 - editor single-root rule

- Канонический editor root зафиксирован как `P:\Silver_77_Quests\Support\JSON_Quvest`.
- Legacy `P:\Silver_77_Quests\JSON_Quvest` больше не должен восприниматься как отдельная живая копия редактора.
- Старый `JSON_Quvest\server.ps1` превращён в обёртку на `Support\JSON_Quvest\server.ps1`, чтобы случайный запуск не поднимал устаревший root.
- Для отладки любых “призраков старой версии” сначала проверять, какой editor root реально обслуживает `127.0.0.1:4173`.

## Update 2026-04-26 - NPC Flow card trigger picker and backup location

- В каждом блоке `NPC Flow` добавлен собственный выбор `NPC / trigger этого блока`.
- Смена trigger внутри карточки переносит включённые роли `offer / completion / reward` и связанные `triggerActions` на выбранный trigger.
- Редактор может досеять отсутствующие `offer` и `reward` из `trigger.questIds`, чтобы старый дефолтный список не развалился. `completion` больше не досеивается автоматически.
- Backup редактора перенесён в видимый соседний файл `Silver_77_Quests_BackUP.json`; `.local/Silver_77_Quests_BackUP.json` больше не должен быть рабочим дефолтом.

Дата: 18.04.2026
Проект: `D:\Dayz\Silver_77_Quests`

Аварийный краткий контекст для перезапуска: `Documentation/CODEX_EMERGENCY_CONTEXT.md`
Контрольный контекст для продолжения: `Documentation/CODEX_CONTROL_CONTEXT.md`

## Краткий статус

Мод `Silver_77_Quests` - система квестов для DayZ. По описанию пользователя, базовая логика квестов уже в целом работает: конфиг JSON, триггеры, прогресс игроков, награды, повторяемые квесты и cooldown.

### Update 26.04.2026

- Редактор квестов в `Support/JSON_Quvest` переведен на role-driven и trigger-first модель.
- `NPC Flow` теперь работает как набор универсальных NPC-блоков.
- По умолчанию показываются только активные блоки квеста.
- Новый NPC можно открыть прямо внутри `NPC Flow` через picker `Открыть / добавить NPC`.
- После включения роли `offer / completion / reward` NPC автоматически попадает в активные блоки.
- Починен конфликт черновиков редактора:
  - draft теперь сохраняется с `updatedAt`
  - при восстановлении выбирается более свежий черновик
  - legacy draft без `updatedAt` больше не должен автоматически восстанавливаться

Главные проблемы на момент продолжения работы:

1. `QuestHint.layout` вызывал зависание/крэш игры на финальной загрузке.
2. Нажатие `F` в зоне триггера не открывало меню квестов.
3. Меню квестов вызывалось через `EnterScriptedMenu(MENU_QUEST_UI, null)`, но кастомный ID меню не был обработан в `CreateScriptedMenu`.

## Что я понял по устройству проекта

Основные файлы:

- `config.cpp` подключает модули `3_Game`, `4_World`, `5_Mission`.
- `scripts/3_Game/QuestData.c` содержит структуры квестов, целей, наград и триггеров.
- `scripts/3_Game/PlayerQuestData.c` содержит прогресс игрока.
- `scripts/4_World/QuestManager.c` загружает конфиг, сохраняет прогресс, выдает предметы и награды.
- `scripts/5_Mission/QuestTrigger.c` проверяет нахождение игрока в триггерной зоне.
- `scripts/5_Mission/QuestUI.c` описывает меню квестов.
- `scripts/5_Mission/mission/MissionGameplay.c` отвечает за клиентскую проверку триггеров, подсказку и открытие меню.
- `gui/QuestMenu.layout` - layout основного меню.
- `gui/layouts/QuestHint.layout` - layout экранной подсказки.

Важное наблюдение: vanilla DayZ прокидывает нажатия клавиш в `MissionGameplay.OnKeyPress(int key)`, поэтому для ловли `F` надежнее использовать `OnKeyPress`, а не polling через `LocalPress` в `OnUpdate`.

## Что было изменено

### 1. `gui/layouts/QuestHint.layout`

Файл был переписан в более стандартную DayZ layout-структуру:

- корневой `FrameWidgetClass QuestHintRoot`;
- внутри `PanelWidgetClass QuestHintPanel`;
- внутри панели `TextWidgetClass QuestHintAction`;
- дочерние виджеты вынесены в отдельные блоки `{ ... }`;
- возвращены свойства в стиле рабочего `QuestMenu.layout`, включая `"text halign"` и `"text valign"`;
- убраны нестандартные `text_halign` и `text_valign`.

Идея правки: старая версия вкладывала `TextWidgetClass` напрямую в свойства панели без отдельного блока детей, что могло ломать парсер layout и зависать на загрузке.

### 2. `scripts/5_Mission/mission/MissionGameplay.c`

Сделаны изменения:

- добавлен `override UIScriptedMenu CreateScriptedMenu(int id)`;
- для `MENU_QUEST_UI` теперь создается `new QuestUIMenu()` и вызывается `SetID(id)`;
- открытие меню по `F` перенесено в `override void OnKeyPress(int key)`;
- старые проверки `input.LocalPress("UAAction")` и `LocalPress_ID(KeyCode.KC_F)` из `OnUpdate` убраны;
- создание виджета подсказки больше не происходит в `OnMissionStart`;
- подсказка создается лениво, только при входе игрока в триггер;
- добавлен флаг `m_QuestHintWidgetFailed`, чтобы не пытаться бесконечно пересоздавать битый виджет;
- если layout подсказки не загрузился или не найден `QuestHintAction`, включается fallback через системный чат;
- при завершении миссии виджет подсказки удаляется через `DestroyHintWidget()`.

## Почему это должно помочь

### Подсказка

Раньше виджет создавался во время старта миссии. Если layout был некорректным, игра могла зависать еще до нормального входа в игровой мир.

Теперь:

- layout имеет корректную вложенность;
- виджет создается только при реальном входе в зону;
- при ошибке есть fallback в чат;
- ошибочный виджет удаляется, чтобы не оставлять мусор в UI.

### Клавиша F

Раньше код пытался ловить `F` через `LocalPress` в `OnUpdate`. По логам пользователя, это не давало записей.

Теперь используется `OnKeyPress`, куда DayZ уже прокидывает клавиши из `DayZGame`. Если игрок находится в зоне квеста и других меню нет, на `KeyCode.KC_F` вызывается `RequestOpenQuestMenu()`.

### Открытие меню

Раньше был вызов:

```c
EnterScriptedMenu(MENU_QUEST_UI, null)
```

Но проект не говорил DayZ, какой класс надо создать для `MENU_QUEST_UI`.

Теперь `MissionGameplay.CreateScriptedMenu()` возвращает `QuestUIMenu` для этого ID.

## Что проверить в игре

После пересборки PBO:

1. Запустить сервер/клиент с модом.
2. Зайти в мир.
3. Подойти к NPC/триггеру. Справочные координаты:
   - `13092.814453 117.007767 13084.485352`
   - `13091.663086 116.755630 13088.637695`
4. Проверить, что игра больше не зависает на финальной загрузке.
5. Проверить лог на строки:
   - `[Silver_77_Quests] Client loading config`
   - `[Silver_77_Quests] Player IS inside trigger`
   - `[Silver_77_Quests] Showing hint`
   - `[Silver_77_Quests] Widget created successfully`
   - `[Silver_77_Quests] F pressed in quest zone`
   - `[Silver_77_Quests] Quest menu opened`
6. Если виджет не появился, проверить чатовый fallback.
7. Если `F pressed in quest zone` есть, но меню не открылось, смотреть ошибку создания `QuestUIMenu`.
8. Если `F pressed in quest zone` нет, значит `OnKeyPress` не доходит или игрок не считается в зоне.

## Следующие важные задачи

1. Протестировать UI в игре после сборки.
2. Если подсказка все еще ломает клиент, временно отключить создание `QuestHint.layout` и оставить только чатовый fallback.
3. После открытия меню проверить принятие и завершение квеста.
4. Проверить в игре серверную RPC-цепочку взятия/сдачи квеста.
5. Проверить повторяемый квест после cooldown.
6. Подумать, должно ли меню показывать все квесты или только квесты текущего триггера/NPC.

## Текущая гипотеза по главным багам

Проблема зависания была не в иконке `F`, а в синтаксисе/структуре `QuestHint.layout`.

Проблема открытия меню была не только в клавише `F`, а еще и в том, что `MENU_QUEST_UI` не был обработан в `CreateScriptedMenu`.

## Для следующей сессии

Начать с проверки этих двух файлов:

- `D:\Dayz\Silver_77_Quests\scripts\5_Mission\mission\MissionGameplay.c`
- `D:\Dayz\Silver_77_Quests\gui\layouts\QuestHint.layout`

Если пользователь уже протестировал мод, сначала спросить:

- зависает ли загрузка;
- появляется ли подсказка;
- есть ли в логах `F pressed in quest zone`;
- открывается ли `QuestUIMenu`.

## Обновление текущей сессии

Сделана более глубокая проверка цепочки `config -> QuestManager -> RPC -> UI -> trigger`.

Что изменено:

1. `AcceptQuest` и `CompleteQuest` больше не вызываются напрямую из клиентского UI. `QuestUI.c` отправляет запросы на сервер через RPC, а сервер обрабатывает их на `PlayerBase.OnRPC`.
2. Добавлен `scripts/4_World/QuestPlayerRPC.c`:
   - запрос серверного конфига;
   - запрос прогресса игрока;
   - серверная обработка взятия квеста;
   - серверная обработка сдачи квеста;
   - применение синхронизированного конфига/прогресса на клиенте.
3. `QuestManager.c` теперь:
   - хранит ревизии конфига и прогресса для обновления UI;
   - на клиенте держит временный дефолтный конфиг только в памяти до серверной синхронизации;
   - не сохраняет прогресс из клиентского `$profile`;
   - досоздает недостающие записи прогресса при добавлении новых квестов;
   - нормализует JSON-конфиг, если в нем отсутствуют массивы;
   - считает cooldown в UTC Unix seconds, а не в миллисекундах `GetGame().GetTime()`.
4. `MissionGameplay.c` теперь запрашивает серверный конфиг и прогресс после появления игрока, а при изменении ревизии конфига пересоздает клиентские триггеры.
5. `QuestUI.c` теперь:
   - хранит реальные `questId` для строк списка;
   - обновляется при приходе синхронизированного прогресса;
   - блокирует кнопки, пока ждет ответ сервера.
6. Документация обновлена под реальные координаты триггеров:
   - рыбак: `13092.814453 117.007767 13084.485352`;
   - охотник: `13091.663086 116.755630 13088.637695`.
7. `build_and_pack.bat` больше не завязан на старый проектный диск; `-project` берется из текущей папки проекта. Также добавлена проверка наличия `AddonBuilder.exe`.

Проверки:

- Проверен баланс `{}` в `scripts` и `gui` - расхождений нет.
- Старые тестовые координаты в README/Documentation не найдены.
- Старый путь проектного диска в README/Documentation/build script не найден.
- PBO не собран: `AddonBuilder.exe` не найден по пути `C:\Program Files (x86)\Steam\steamapps\common\DayZ Tools\Bin\AddonBuilder\AddonBuilder.exe` и не найден через `where AddonBuilder.exe`.

Что проверить в игре после установки DayZ Tools/сборки PBO:

1. Клиент не зависает на финальной загрузке.
2. В логах есть запрос/отправка синхронизации:
   - `[Silver_77_Quests] Requesting initial quest sync from server`
   - `[Silver_77_Quests] Sent quest config to client`
   - `[Silver_77_Quests] Sent quest progress to client`
3. В зоне триггера появляется подсказка.
4. Нажатие `F` открывает меню.
5. Взятие квеста выдает стартовые предметы сервером.
6. Сдача квеста забирает цели, выдает награду и сохраняет прогресс в серверном `$profile`.

## Обновление координат NPC

Координаты дефолтных триггеров изменены:

- NPC 1 / рыбак:
  - position: `13092.814453 117.007767 13084.485352`
  - orientation: `-167.224930 4.670787 -0.265732`
- NPC 2 / охотник:
  - position: `13091.663086 116.755630 13088.637695`
  - orientation: `-46.336338 -1.064082 2.994759`

Важное: ориентация сейчас только записана в отчет для контекста. Текущий мод не спавнит NPC, а только создает зоны взаимодействия, поэтому orientation должен использовать тот скрипт/мод, который ставит NPC. Радиус триггеров выставлен `1.4`, потому что NPC стоят близко друг к другу и старый радиус `3.0` давал бы пересечение зон.

Если сервер уже запускал мод и файл `profiles/Silver_77_Quests/Silver_77_Quests.json` существует, дефолтные координаты из PBO не применятся автоматически. Нужно изменить JSON вручную или удалить файл, чтобы он пересоздался.

## Обновление после первого игрового теста

Что подтвердилось:

- подсказка возле NPC появляется;
- `F` открывает меню;
- выбор квестов в меню работает;
- кнопка `ЗАКРЫТЬ` работает.

Что поправлено по скриншотам:

1. Рыбак перенесен на новую точку:
   - position: `13092.814453 117.007767 13084.485352`
   - orientation: `-167.224930 4.670787 -0.265732`
2. `QuestHint.layout` переведен на фиксированную панель `420x26` и текст `"exact text size" 18`, чтобы подсказка не резалась и была примерно в два раза меньше.
3. `QuestMenu.layout` переведен на фиксированные размеры панели и текстов. Для `DescriptionText` добавлен `"exact text size" 18` и `wrap 1`, потому что огромные буквы в меню были похожи на растянутый multiline-текст без явного размера шрифта.
4. Для кнопок меню добавлен явный размер текста `"exact text size" 20`.
5. Для проблемы “квест нельзя взять” добавлены логи:
   - `[Silver_77_Quests] Accept button clicked: ...`
   - `[Silver_77_Quests] Sending accept quest RPC: ...`
   - `[Silver_77_Quests] Received quest progress request RPC`
   - `[Silver_77_Quests] Accept quest RPC result for ...`
6. Проверка RPC-отправителя на сервере стала мягче: если `sender` приходит `null`, запрос не отбрасывается автоматически, потому что RPC уже пришел на объект игрока.
7. В UI добавлен таймаут ожидания ответа сервера. Если прогресс не вернулся за 5 секунд, кнопки разблокируются и меню повторно запросит прогресс.
8. Исправлено наложение подсказки `Поговорить с ...` на меню: `MissionGameplay.RequestOpenQuestMenu()` теперь скрывает hint перед открытием меню, а `QuestUIMenu.OnShow/OnHide` сообщает миссии об открытии/закрытии. После закрытия меню подсказка возвращается, если игрок все еще в зоне NPC.
9. Шрифты подсказки и меню переведены на фиксированные DayZ bitmap-шрифты (`metron16`, `metron22`, `metron28`) вместо SDF/масштабируемого варианта. По игровому скриншоту подсказка `[F] Поговорить с рыбаком` отображается корректно: компактно, без обрезания и без гигантских букв.
10. Исправлена проблема, когда список визуально подсвечивал квест, но `QuestUI.c` все еще считал, что квест не выбран: справа оставалось `Выберите квест из списка`, а кнопка `ВЗЯТЬ КВЕСТ` была недоступна. Теперь после `RefreshQuestList()` автоматически выбирается первая строка, `m_SelectedQuestId` выставляется напрямую, а в `Update()` выбранная строка синхронизируется с `TextListboxWidget.GetSelectedRow()`.
11. После успешного теста взятия/сдачи квеста исправлена логика удаления целей. `Carp` имеет внутреннюю quantity как пищевой предмет, поэтому старый код мог уменьшать quantity рыбы, но не удалять сам предмет. В `Silver77_QuestObjective` добавлен `useItemQuantity`:
   - `false` - считать предметы штуками и удалять целые предметы;
   - `true` - считать внутреннюю quantity предмета.
   Для дефолтных карпов и мяса стоит `useItemQuantity = false`. `removeOnComplete` остается настройкой “забирать предмет при сдаче или оставить игроку”.
12. Меню квестов теперь фильтрует список по текущему NPC/триггеру. `MissionGameplay.c` запоминает `questIds` зоны, где стоит игрок, а `QuestUI.c` показывает только эти квесты. Если список `questIds` у триггера пустой, остается fallback: показывать все квесты, чтобы меню не стало пустым из-за ошибки конфига.
13. Добавлен временный журнал активных квестов:
   - новое меню `QuestJournalUIMenu` в `scripts/5_Mission/QuestJournalUI.c`;
   - новый layout `gui/QuestJournal.layout`;
   - открытие по клавише `J` через `MissionGameplay.OnKeyPress`;
   - в журнале показываются только квесты со статусом `active`, их описание, цели, готовность целей по текущему инвентарю, награды и подсказка, что сдавать квест нужно у NPC, который его выдал.

## Обновление 18.04.2026 - журнал и стартовый JSON

1. Поведение `scripts/5_Mission/QuestJournalUI.c` оставлено намеренно: журнал по `J` не забирает game focus, чтобы персонаж мог двигаться, пока игрок смотрит активные квесты.
2. Добавлен пример стартового конфига `Documentation/STARTER_QUEST_CONFIG.json`. Его можно использовать как основу для серверного `profiles/Silver_77_Quests/Silver_77_Quests.json`.
3. Важно: дефолтный JSON создается только если серверного файла еще нет. Если `profiles/Silver_77_Quests/Silver_77_Quests.json` уже существует, нужно заменить/отредактировать именно его или удалить файл перед запуском сервера, чтобы мод создал новый.

## Обновление 18.04.2026 - накопительная сдача предметов

1. В `Silver77_QuestObjective` добавлен флаг `allowPartialTurnIn`.
2. В `PlayerQuestProgress` добавлен массив `objectiveProgress`, где сохраняется количество уже внесенных предметов по целям квеста.
3. `QuestManager.CompleteQuest()` теперь для накопительных целей сначала вносит доступные предметы, удаляет их из инвентаря и сохраняет прогресс. Награды выдаются только после достижения полного количества.
4. Старые квесты остаются в прежнем режиме, если `allowPartialTurnIn = false`.
5. `QuestUI.c` и `QuestJournalUI.c` показывают прогресс накопительных целей в формате `Сдано: X / Y`.
6. Первичный пример `quest_farmer_1` с `Potato x50` был заменен пользовательскими стартовыми квестами в следующем обновлении.

## Обновление 18.04.2026 - пользовательские стартовые квесты

1. `CreateDefaultQuestConfig()` переписан под пользовательские стартовые квесты:
   - `quest_fisherman_1`: `Картошечка с маслицем`;
   - `quest_hunter_1`: `Рыба это вам не картошка!`.
2. В обеих стартовых цепочках все item-цели получили `allowPartialTurnIn = true`.
3. `Documentation/STARTER_QUEST_CONFIG.json` теперь содержит только эти два пользовательских квеста и два текущих триггера.
4. Подсказки триггеров обновлены:
   - `[F] Коля Ворон`;
   - `[F] Рыбак Гаврила перец`.
5. Важно: существующий серверный `profiles/Silver_77_Quests/Silver_77_Quests.json` все равно нужно заменить вручную или удалить перед запуском сервера, иначе новый дефолт из PBO не подхватится.

## Обновление 18.04.2026 - управление журналом

1. В `MissionGameplay.OnKeyPress()` добавлено закрытие журнала активных квестов по `ESC`.
2. Обработка `ESC` стоит до `super.OnKeyPress(key)`, чтобы попытаться закрыть журнал без открытия системного меню.
3. В `QuestJournalUI.c` добавлено переключение выбранного активного квеста колесом мыши:
   - колесо вверх - предыдущий квест;
   - колесо вниз - следующий квест;
   - список зациклен.
4. Добавлен контрольный контекст `Documentation/CODEX_CONTROL_CONTEXT.md` для продолжения работы завтра.
5. Важно проверить в игре, доходит ли `OnMouseWheel` до журнала без захвата game focus.

## Обновление 18.04.2026 - первый split client/server

1. Создана отдельная split-структура:
   - `SplitMods/Silver_77_Quests_Client`;
   - `SplitMods/Silver_77_Quests_Server`.
2. Добавлен сборочный скрипт `build_split_mods.bat`.
3. Добавлена документация `Documentation/SPLIT_CLIENT_SERVER.md`.
4. Обновлен `Documentation/BUILD.md`: рекомендуемый путь теперь client/server split, монолитная сборка оставлена как резервная.
5. Client PBO:
   - публикуется в Workshop;
   - содержит DTO, UI, layouts, `QuestClientManager`, `QuestClientRPC`, client-side trigger/hint logic;
   - получает конфиг и прогресс с сервера через RPC.
6. Server PBO:
   - не публикуется;
   - содержит `QuestServerManager`, `QuestServerRPC`, `MissionServer`;
   - грузит JSON, проверяет квесты, удаляет предметы, выдает награды, сохраняет прогресс.
7. Серверный `config.cpp` зависит от `Silver_77_Quests_Client`, чтобы общие классы не дублировались:

```cpp
requiredAddons[] = {"DZ_Data", "DZ_Scripts", "Silver_77_Quests_Client"};
```

8. Запуск сервера для split:

```bat
-mod=@Silver_77_Quests_Client
-serverMod=@Silver_77_Quests_Server
```

9. Корневой монолитный мод оставлен как резерв. Не запускать его одновременно со split-версией.

## Обновление 18.04.2026 - восстановление русского текста в split

1. В split-копиях часть русских строк была сохранена как mojibake вида `Р’С‹Р±РµСЂРёС‚Рµ`.
2. Восстановлены видимые строки в:
   - `SplitMods/Silver_77_Quests_Client/scripts/5_Mission/QuestUI.c`;
   - `SplitMods/Silver_77_Quests_Client/scripts/5_Mission/QuestJournalUI.c`;
   - `SplitMods/Silver_77_Quests_Client/scripts/5_Mission/QuestTrigger.c`;
   - `SplitMods/Silver_77_Quests_Server/scripts/4_World/QuestServerManager.c`.
3. Серверный лог изменен на:

```text
[Silver_77_Quests] QuestServerManager initialized
```

4. Документация `BUILD.md` и `SPLIT_CLIENT_SERVER.md` обновлена под этот лог.
5. Проверено:
   - в `SplitMods` не осталось монолитных имен `QuestManager`, `g_QuestConfig`, `g_PlayerQuestData`, `g_Silver77_Quest`;
   - `$PBOPREFIX$` у client равен `Silver_77_Quests`;
   - `$PBOPREFIX$` у server равен `Silver_77_Quests_Server`;
   - папки из `config.cpp` существуют.
6. PBO через DayZ Tools после этих правок еще не собирался.

## Заметка 18.04.2026 - почему ломается русский

Русский ломается из-за смешения кодировок Windows. Типичный сценарий: файл был в UTF-8, инструмент прочитал байты как Windows-1251/OEM/ANSI, получил текст вида `Р’С‹`, потом этот текст записали обратно в UTF-8.

Отдельная памятка добавлена в `Documentation/RUSSIAN_ENCODING.md`.

Правила:

1. Держать `.c`, `.layout`, `.json`, `.md`, `.bat` в UTF-8.
2. В VS Code проверять кодировку в нижнем правом углу.
3. В PowerShell читать русские файлы через `Get-Content -Encoding UTF8`.
4. Не переписывать файлы через `>` или `Out-File` без явной кодировки.
5. Для bat с русским выводом использовать `chcp 65001 >nul`.
6. Если в строках в кавычках появились `Р’С‹`, `Рџ`, `СЃ`, `С‚`, это уже не просто проблема отображения консоли, а сломанный текст в файле.

## Обновление 19.04.2026 - утренняя проверка контекста

1. Подтверждено, что контрольный контекст для продолжения лежит в `Documentation/CODEX_CONTROL_CONTEXT.md`.
2. Рабочая папка `D:\Dayz\Silver_77_Quests` не является git-репозиторием, поэтому состояние проверяется по файлам и документации, а не через `git status`.
3. Повторно проверены `SplitMods`, `scripts` и `gui` на типичные признаки mojibake. После правки битые русские строки в коде не найдены.
4. Исправлены оставшиеся битые русские комментарии в split-файлах:
   - `SplitMods/Silver_77_Quests_Client/scripts/5_Mission/mission/MissionGameplay.c`;
   - `SplitMods/Silver_77_Quests_Server/scripts/4_World/QuestServerRPC.c`;
   - `SplitMods/Silver_77_Quests_Server/scripts/5_Mission/mission/MissionServer.c`.
5. Проверено, что в `SplitMods` нет старых монолитных имен `QuestManager`, `g_QuestConfig`, `g_PlayerQuestData`, `g_Silver77_Quest`.
6. `Documentation/STARTER_QUEST_CONFIG.json` проходит `ConvertFrom-Json`.
7. `AddonBuilder.exe` найден по пути `D:\SteamLibrary\steamapps\common\DayZ Tools\Bin\AddonBuilder\AddonBuilder.exe`.
8. В `build_split_mods.bat` обновлен путь `DAYZ_TOOLS` под найденный DayZ Tools.
9. PBO/split-сборка пока не запускалась: скрипт пишет результат в `D:\Dayz\Mods_DONE`, что требует отдельного разрешения на запуск вне sandbox.
10. Пользователь отдельно уточнил: сборку, выкладку и публикацию он делает сам. Codex дальше не должен запускать build/publish без прямой новой просьбы.

## Обновление 19.04.2026 - split-папки вынесены в корень

1. Для понятной ручной сборки созданы корневые папки:
   - `Silver_77_Quests_Client`;
   - `Silver_77_Quests_Server`.
2. Эти папки скопированы из уже подготовленного `SplitMods`.
3. `build_split_mods.bat` переключен на новые корневые source directory:
   - `%ROOT_DIR%Silver_77_Quests_Client`;
   - `%ROOT_DIR%Silver_77_Quests_Server`.
4. Документация обновлена: `README.md`, `Documentation/BUILD.md`, `Documentation/SPLIT_CLIENT_SERVER.md`, `Documentation/CODEX_CONTROL_CONTEXT.md`, `Documentation/CODEX_EMERGENCY_CONTEXT.md`, `Documentation/RUSSIAN_ENCODING.md`.
5. `SplitMods/` не удалялся и оставлен как резервная копия, чтобы не потерять старую split-структуру.
6. Проверено:
   - клиентский `$PBOPREFIX$`: `Silver_77_Quests`;
   - серверный `$PBOPREFIX$`: `Silver_77_Quests_Server`;
   - в корневых client/server папках не найдены типичные mojibake-строки;
   - в корневых client/server папках не найдены старые монолитные имена `QuestManager`, `g_QuestConfig`, `g_PlayerQuestData`, `g_Silver77_Quest`.

## Обновление 19.04.2026 - финальная проверка перед ручной сборкой

1. Сборка и публикация не запускались: пользователь делает build/publish сам.
2. Проверена структура:
   - `Silver_77_Quests_Client` содержит UI, layouts, общие DTO, client manager/RPC и `MissionGameplay`;
   - `Silver_77_Quests_Server` содержит `QuestServerManager`, server RPC и `MissionServer`.
3. Проверены `config.cpp` и `$PBOPREFIX$`:
   - client prefix: `Silver_77_Quests`;
   - server prefix: `Silver_77_Quests_Server`;
   - server `requiredAddons[]` содержит `Silver_77_Quests_Client`.
4. Проверено разделение ответственности:
   - JSON-загрузка, `$profile`, сохранение прогресса, удаление предметов и выдача наград находятся в серверной папке;
   - UI, layouts, клавиши, подсказки и клиентские RPC-запросы находятся в клиентской папке.
5. Проверено, что имена виджетов из кода есть в layout-файлах: `QuestListbox`, `DescriptionText`, `AcceptButton`, `CompleteButton`, `CloseButton`, `QuestHintAction`.
6. `Documentation/STARTER_QUEST_CONFIG.json` успешно проходит `ConvertFrom-Json`.
7. Простая проверка баланса `{}` в `.c`, `.cpp`, `.layout` не нашла расхождений.
8. В корневых split-папках не найдены типичные mojibake-строки.
9. Выровнен текст первого стартового квеста с реальными целями: описание теперь просит 20 картошек и 1 белый гриб, как указано в целях (`PotatoSeed x20`, `BoletusMushroom x1`).
10. Остались известные compile-risk места, которые можно подтвердить только DayZ-компилятором:
    - `QuestJournalUI.OnMouseWheel(Widget w, int x, int y, int wheel)`;
    - `KeyCode.KC_ESCAPE`.

## Обновление 19.04.2026 - правки после ошибок DayZ script compiler

1. При сборке/запуске DayZ Tools выдавал `Parser: quoted string not closed on line 1` на split `.c` файлах, хотя кавычки были парные.
2. Найден повторяющийся фактор риска: UTF-8 BOM и кириллица в комментариях script-файлов.
3. Все `.c`, `.cpp`, `.layout` в `Silver_77_Quests_Client` и `Silver_77_Quests_Server` пересохранены как UTF-8 без BOM.
4. Русские комментарии в split script-файлах заменены на ASCII-комментарии.
5. Русские игровые строки в строковых литералах пока оставлены, потому что серверный `World` после удаления BOM/русских комментариев прошел дальше.
6. Проверено:
   - в split code/layout файлах нет UTF-8 BOM;
   - в split script-файлах нет русской кириллицы в комментариях;
   - баланс `{}` в `.c`, `.cpp`, `.layout` нормальный.
7. После этих правок нужно пересобрать клиентский PBO, потому что ошибка запуска указывала на `Silver_77_Quests/scripts/5_Mission/questtrigger.c`, то есть на клиентский мод.

## Обновление 19.04.2026 - ESC для NPC меню и фикс раннего завершения накопительных целей

1. Игровой тест подтвердил:
   - split запускается;
   - UI открывается;
   - русский текст отображается;
   - скролл в журнале квестов работает;
   - `ESC` закрывает журнал квестов;
   - накопительная сдача предметов работает.
2. Добавлено закрытие NPC quest menu по `ESC` в `Silver_77_Quests_Client/scripts/5_Mission/mission/MissionGameplay.c`.
3. Исправлен серверный баг раннего завершения накопительного квеста:
   - сценарий: рыбный квест `Carp x6`, игрок внес `2/6`, потом принес еще `2`, и квест мог завершиться на `4/6`;
   - причина: после `DepositPartialQuestItems()` сервер сразу вызывал `CanCompleteQuest()`, а только что удаленные предметы могли еще считаться в инвентаре в тот же тик;
   - решение: после частичного внесения сервер завершает квест только через `AreQuestObjectivesReadyForCompletion()`, где partial-цели проверяются строго по сохраненному `depositedQuantity`.
4. Проверено:
   - split `.c/.cpp/.layout` файлы сохранены без UTF-8 BOM;
   - русских комментариев в split script-файлах нет;
   - баланс `{}` в `.c`, `.cpp`, `.layout` нормальный.
5. Для проверки этой правки нужно пересобрать оба PBO:
   - client: из-за `ESC` в `MissionGameplay.c`;
   - server: из-за фикса `QuestServerManager.c`.

## Обновление 19.04.2026 - разделение F pickup и F диалога NPC

1. В игровом тесте найден конфликт: игрок поднимает предмет по `F`, и в тот же момент открывается quest menu NPC, потому что игрок находится в quest trigger zone.
2. Вариант с отдельной клавишей `U` был начат, но отменен по уточнению пользователя.
3. Реализован focus-check при сохранении клавиши `F`:
   - подсказки остаются `[F]`;
   - `F` открывает NPC quest menu только если игрок находится в радиусе триггера и луч камеры проходит через виртуальную область возле центра NPC;
   - если игрок смотрит вниз на предмет, quest menu не открывается, а ванильное действие DayZ может поднять предмет.
4. Изменения:
   - `Silver_77_Quests_Client/scripts/5_Mission/QuestTrigger.c`:
     - добавлены `SILVER77_QUEST_FOCUS_HEIGHT = 1.0`;
     - добавлены `SILVER77_QUEST_FOCUS_RADIUS = 0.5`;
     - добавлен `QuestTrigger.IsPlayerLookingAt()`;
     - добавлен `QuestTriggerManager.GetFocusedTriggerForPlayer()`.
   - `Silver_77_Quests_Client/scripts/5_Mission/mission/MissionGameplay.c`:
     - `F` теперь проверяет focused trigger прямо в момент нажатия;
     - проверка подсказки тоже использует focused trigger.
5. Если наведение окажется слишком строгим или слишком широким, регулировать `SILVER77_QUEST_FOCUS_RADIUS`:
   - меньше значение = строже;
   - больше значение = мягче.
6. Изменены условия первого стартового квеста:
   - `PotatoSeed` теперь `x50`;
   - награда `Ammo_12gaPellets` теперь `x100`;
   - описание квеста обновлено под 50 картошек.
7. Для проверки нужна пересборка обоих PBO:
   - client: из-за focus-check;
   - server: из-за дефолтного квеста `PotatoSeed x50` и награды `x100`.
   Если рабочий серверный JSON уже существует, его нужно обновить вручную или удалить, чтобы сервер создал новый дефолт.

## Обновление 19.04.2026 - настройка высоты focus-check

1. Пользователь заметил, что подсказка/активация диалога появляется слишком низко.
2. В `Silver_77_Quests_Client/scripts/5_Mission/QuestTrigger.c` поднята точка фокуса:
   - было `SILVER77_QUEST_FOCUS_HEIGHT = 1.0`;
   - стало `SILVER77_QUEST_FOCUS_HEIGHT = 1.45`.
3. `SILVER77_QUEST_FOCUS_RADIUS` оставлен `0.5`.
4. Ожидаемое поведение: подсказка и `F`-диалог активируются при наведении примерно на грудь/плечи NPC, а не вниз.
5. Для проверки нужна пересборка клиентского PBO.

## Обновление 19.04.2026 - focus-check настраивается из JSON

1. Пользователь попросил настраивать точку наведения для каждого NPC/триггера в JSON.
2. В `Silver77_QuestTriggerConfig` добавлены поля:
   - `focusHeight` - высота точки наведения над `position`;
   - `focusRadius` - радиус виртуальной точки наведения.
3. Дефолты:
   - `focusHeight = 1.45`;
   - `focusRadius = 0.5`.
4. Старые JSON без этих полей совместимы: `NormalizeQuestConfig()` на сервере подставляет дефолты, если значения отсутствуют или `<= 0`.
5. `QuestTrigger.SetupTrigger()` теперь получает `focusHeight` и `focusRadius` из `Silver77_QuestTriggerConfig`.
6. Обновлены:
   - `Documentation/STARTER_QUEST_CONFIG.json`;
   - `Documentation/README_JSON_CONFIG.md`;
   - `Documentation/README_INSTALLATION.md`;
   - `Documentation/CHANGELOG.md`.
7. Проверено:
   - `STARTER_QUEST_CONFIG.json` валиден;
   - split `.c/.cpp/.layout` без UTF-8 BOM;
   - баланс `{}` нормальный.
8. Для проверки нужна пересборка обоих PBO:
   - client: новые поля DTO и client focus logic;
   - server: дефолтный JSON и normalize логика.
   Если серверный профильный JSON уже существует, добавить `focusHeight`/`focusRadius` вручную или удалить JSON для пересоздания.

## Обновление 19.04.2026 - последовательность квестов через requiredQuestIds

1. Для цепочек квестов используется поле `requiredQuestIds` в JSON.
2. Логика:
   - старое поле `requiresPrevious` оставлено для одного обязательного предыдущего квеста;
   - новое поле `requiredQuestIds` принимает список квестов;
   - все квесты из списка должны быть выполнены хотя бы один раз.
3. Проверка стоит на сервере в `QuestServerManager.CanAcceptQuest()`, поэтому игрок не сможет обойти условие клиентом.
4. Клиентская проверка в `QuestClientManager.CanAcceptQuest()` нужна для корректной кнопки в UI.
5. Для повторяемых квестов проверяется `lastCompletedTime`, поэтому повторное взятие уже закрытого квеста не блокирует следующую ветку.
6. В `QuestUI` добавлен вывод условий: игрок видит, какие квесты нужны и выполнены ли они.
7. Пример:
   ```json
   "requiredQuestIds": [
     "quest_medic_1",
     "quest_medic_2",
     "quest_medic_3",
     "quest_hunter_medic_1"
   ]
   ```
8. Старые JSON совместимы: если `requiredQuestIds` отсутствует, normalize создаёт пустой список.

## Обновление 26.04.2026 - role-driven видимость, reward action и редактор

1. Починен запускатор редактора в `P:\Silver_77_Quests\Support\JSON_Quvest`.
   - исправлены пути в `start-editor.ps1`, `server.ps1`, `editor-config.local.json`;
   - добавлена защита от копирования файла в самого себя;
   - добавана нормальная обработка случая, когда в конфиге указан не JSON-файл, а папка;
   - старые запускаторы в `P:\Silver_77_Quests\JSON_Quvest` теперь работают как обертка на актуальную папку.

2. Для редактора добавлены меры против залипания на старой версии:
   - no-cache headers на сервере редактора;
   - versioned assets в `index.html`.

3. В редакторе уже работает фильтр квестов по trigger / NPC:
   - в левой колонке можно смотреть квесты конкретного NPC;
   - порядок внутри NPC идет из `trigger.questIds`;
   - это теперь считать порядком квестов у NPC, а не отдельной игровой видимостью.

4. Основная модель квестов переведена на role-driven схему:
   - `offerTriggerIds`
   - `completionTriggerIds`
   - `rewardTriggerIds`
   - `hideUntilRequirementsComplete`

5. Новая логика видимости по статусу:
   - `not_started` -> offer
   - `active` -> completion
   - `reward_pending` -> reward
   - `trigger.questIds` больше не считать самостоятельной runtime-видимостью; это список и порядок квестов у конкретного NPC.

6. В `Silver77_Quest` добавлен новый каркас действий по trigger:
   - `triggerActions`
   - один action содержит:
     - `triggerId`
     - `actionType`
     - `dialogText`
     - `rewards`
   - текущие типы действий:
     - `completion`
     - `reward`

7. В редакторе старт квеста оставлен в основном блоке:
   - `description` = стартовое описание / диалог при взятии;
   - `giveItems` = выдача при старте.
   Completion и reward вынесены в отдельные action-карточки по NPC.

8. На клиенте и сервере уже заложена логика отдельного reward trigger:
   - есть статус `reward_pending`;
   - есть переход к выдаче награды у другого NPC;
   - подсветка reward trigger желтым уже была заложена в предыдущей ветке и сохраняется как актуальная модель.

9. Чтобы старый дефолтный список квестов не пропал при переходе на новую схему, добавлено автодосеивание ролей из `trigger.questIds`:
   - на сервере в normalize / migration;
   - на клиенте в normalize;
   - в редакторе в normalize данных.
   Это важно: пользователь не просил сохранять старую пользовательскую совместимость, но просил сохранить по смыслу дефолтный список старых известных квестов.

10. Старые глобальные `turnInDialogText` / `rewardDialogText` выведены из основной модели.
    - активной моделью теперь считать action-диалоги по trigger;
    - в `QuestUI.c` старый fallback убран.

11. Основные файлы этой ветки:
    - `Support/JSON_Quvest/app.js`
    - `Support/JSON_Quvest/index.html`
    - `Support/JSON_Quvest/server.ps1`
    - `Support/JSON_Quvest/start-editor.ps1`
    - `Support/JSON_Quvest/editor-config.local.json`
    - `JSON_Quvest/start-editor.ps1`
    - `JSON_Quvest/start-editor.cmd`
    - `Silver_77_Quests_Client/scripts/3_Game/QuestData.c`
    - `Silver_77_Quests_Client/scripts/4_World/QuestClientManager.c`
    - `Silver_77_Quests_Client/scripts/5_Mission/QuestUI.c`
    - `Silver_77_Quests_Server/scripts/4_World/QuestServerManager.c`
    - `Silver_77_Quests_Server/scripts/4_World/QuestServerRPC.c`

12. Что уже проверено всухую:
    - `node --check Support/JSON_Quvest/app.js`
    - `http://127.0.0.1:4173/api/health` отвечает `{"ok":true}`
    - сервер редактора уже отдает обновленный `app.js`

13. Что еще не проверено:
    - не сделан полный игровой тест после последних role-driven правок;
    - не сделана новая пересборка client/server PBO именно под эту ревизию;
    - логика "именно тот конкретный предмет, который был выдан" как отдельная identity-chain пока не реализована.

14. Следующий практический шаг:
    - пересобрать оба PBO;
    - прогнать в игре один обычный дефолтный квест;
    - прогнать маршрут `A -> B -> reward`;
    - проверить visibility по ролям, completion/reward диалоги и reward_pending.

## Обновление 25.04.2026 - новый минимальный дефолт сервера и синхронизация с редактором

1. Пользователь зафиксировал новый минимальный стартовый набор квестов как основную базу проекта.
2. В этот минимум входят 4 квеста:
   - `quest_hunter_1` — `Картошечка с маслицем`;
   - `quest_fisherman_1` — `Рыба это вам не картошка!`;
   - `quest_Rasputin_1` — `Взаимовыручка прежде всего!`;
   - `quest_fisherman_2` — `Поставка медицины`.
3. Обновлены серверные генераторы дефолта:
   - `Silver_77_Quests_Server/scripts/4_World/QuestServerManager.c`;
   - `SplitMods/Silver_77_Quests_Server/scripts/4_World/QuestServerManager.c`;
   - `scripts/4_World/QuestManager.c` как legacy-источник.
4. Новый дефолтный набор триггеров:
   - `hunter_trigger` -> `quest_hunter_1`;
   - `fisherman_trigger` -> `quest_fisherman_2`, `quest_fisherman_1`;
   - `Rasputin_1_trigger` -> `quest_Rasputin_1`.
5. Обновлены шаблоны/справочные JSON:
   - `Documentation/STARTER_QUEST_CONFIG.json`;
   - `Support/JSON_Quvest/Silver_77_Quests.json`.
6. Важно для реального сервера:
   - мод читает рабочий конфиг из `profiles/Silver_77_Quests/Silver_77_Quests.json`;
   - новый дефолт создастся только если профильный JSON отсутствует;
   - пользователь планирует удалить профильный JSON, чтобы сервер пересоздал его из нового дефолта.
7. Для split-схемы с отдельными `Silver_77_Quests_Client` и `Silver_77_Quests_Server` под этот апдейт достаточно пересобрать/обновить только серверный мод:
   - клиентская логика не менялась;
   - менялся только серверный источник дефолтного конфига.
8. Параллельно синхронизирован редактор JSON на `M:\GITS_VERSE\Neyro_01\Sborka_Json\JSON_Quvest`:
   - там этот же минимальный набор уже лежит как дефолтный `Silver_77_Quests.json`;
   - редактор теперь используется как основная мастерская для правки структуры квестов.
9. В редакторе подготовлена отдельная основа под справочник стеков:
   - слева добавлен отдельный блок `Стаки`;
   - правила хранятся отдельно в `item-stack-rules.json`;
   - сервер редактора получил маршруты `/api/stack-rules`;
   - автоматический пересчет objective из "наборов" в реальное `quantity` пока еще не внедрен, но инфраструктура под ручное наполнение справочника уже заложена.

## Update 2026-04-26 - RPC sync hardening after client crash

1. Crash report from DayZ client showed:
   - reason `String CORRUPTED - FIX OnStoreLoad() !!!`
   - stack root at `Silver_77_Quests/scripts/4_World/questclientrpc.c:29`
   - crash happened while reading `SILVER77_QUEST_RPC_CONFIG_DATA` on the client.
2. Local codebase already had matching `QuestData.c` on disk, so the remaining risk was the binary DayZ RPC serializer itself on the new nested quest schema (`triggerActions -> rewards`) and/or mismatched live builds.
3. Fix applied in runtime sync:
   - server no longer sends `Param1<Silver77_QuestConfig>` over RPC;
   - server no longer sends `Param1<PlayerQuestData>` over RPC;
   - server now serializes both payloads to JSON strings;
   - client now receives JSON string payloads, writes temp sync files under `$profile:Silver_77_Quests`, and loads them through `JsonFileLoader`.
4. Files changed for this hardening:
   - `Silver_77_Quests_Client/scripts/4_World/QuestClientRPC.c`
   - `Silver_77_Quests_Server/scripts/4_World/QuestServerManager.c`
5. Important build note:
   - `build_split_mods.bat` currently prints `[OK]` even when `AddonBuilder` dies earlier with `System.ArgumentNullException` before a real rebuild;
   - do not trust the final `[OK]` line until the build script / AddonBuilder invocation is cleaned up.

## Update 2026-04-26 - quest config sync chunking and template alignment

1. Fresh client log still showed:
   - `QuestTriggerManager initialized with 0 triggers`
   - `ERROR: Failed to read quest config sync RPC`
   - no quest talk prompt because config never reached the client.
2. Player progress sync was arriving, so the remaining problem narrowed to the quest config payload itself.
3. Quest config sync was changed again:
   - server now splits quest config payload into `512` byte chunks;
   - client now receives `Param3<int, int, string>` chunks for `SILVER77_QUEST_RPC_CONFIG_DATA`;
   - client reassembles the payload before loading JSON.
4. Files changed for chunked config sync:
   - `Silver_77_Quests_Client/scripts/4_World/QuestClientRPC.c`
   - `Silver_77_Quests_Server/scripts/4_World/QuestServerManager.c`
5. Default template files were checked and aligned with the current role-driven schema:
   - `Support/JSON_Quvest/Silver_77_Quests.json`
   - `Documentation/STARTER_QUEST_CONFIG.json`
6. Template alignment made explicit:
   - `version` raised to `3`
   - added `hideUntilRequirementsComplete`
   - added explicit `offerTriggerIds`
   - added explicit `completionTriggerIds`
   - added explicit `rewardTriggerIds`
   - added explicit empty `triggerActions`
7. `build_split_mods.bat` was hardened:
   - now captures AddonBuilder output into logs;
   - now fails on `[FATAL]` instead of falsely printing success.

## Update 2026-04-26 - quest editor trigger-first NPC flow UI

1. Quest editor UX in `Support/JSON_Quvest` was reworked around trigger / NPC cards instead of two separate global sections for role checkboxes and action blocks.
2. New editor behavior:
   - each trigger / NPC now has its own card in the quest editor;
   - offer / completion / reward checkboxes live inside that NPC card;
   - when a role is enabled, its detailed settings appear directly below that same NPC card;
   - cards alternate between two accent colors so neighboring NPC blocks are easier to distinguish visually.
3. Offer details are now moving into the same action model:
   - offer dialog is per NPC through `triggerActions`;
   - giveItems and objectives still live as quest-level settings shown inside the Offer block.
4. Completion / reward details stay runtime-compatible:
   - per-trigger dialog and reward settings still use `triggerActions`;
   - the change was presentation / editor UX, not a runtime schema change.
5. Files changed:
   - `Support/JSON_Quvest/app.js`
   - `Support/JSON_Quvest/styles.css`
   - `Support/JSON_Quvest/index.html`

## Update 2026-04-26 - mandatory Reward and Completion chain tracking

1. Product rule fixed:
   - every quest should have `Offer` and `Reward`;
   - `Completion` is optional and can be zero, one, or several intermediate handoff stages.
2. Runtime rule fixed:
   - `Reward` is the final closer;
   - if any Completion exists, Reward cannot close the quest or give final reward until every Completion trigger is done;
   - if no Completion exists, Reward accepts objectives directly and closes the quest.
3. Split client/server updated:
   - `Silver_77_Quests_Client/scripts/3_Game/PlayerQuestData.c` now stores `completedCompletionTriggerIds`;
   - `Silver_77_Quests_Client/scripts/4_World/QuestClientManager.c` predicts visibility/actions with the new Completion chain;
   - `Silver_77_Quests_Client/scripts/5_Mission/QuestUI.c` reads offer/completion/reward dialogs from `triggerActions`;
   - `Silver_77_Quests_Server/scripts/4_World/QuestServerManager.c` marks Completion stages done, sets `reward_pending` only after all Completion stages, and blocks Reward until the chain is complete.
4. Editor updated:
   - `triggerActions` now includes `offer`, `completion`, and `reward`;
   - offer dialog is per NPC, not only global `description`;
   - missing `offer` / `reward` roles can be seeded from `trigger.questIds` so old default quests remain usable as A -> A flows;
   - `completion` is no longer auto-seeded from `trigger.questIds`.
5. Verification done:
   - `node --check P:\Silver_77_Quests\Support\JSON_Quvest\app.js` passes.
6. Not yet verified:
   - no DayZ compile/build was run in this pass;
   - in-game tests are still needed for direct `Offer -> Reward` and chained `Offer -> Completion -> Reward`.

## Update 2026-04-26 - latest session cutoff for universal NPC blocks

1. Product logic was clarified one step further by the user:
   - `Offer` is mandatory;
   - `Reward` is mandatory and is always the final closer;
   - `Completion` is optional and may be one or many intermediate stages.
2. Reward rule was fixed explicitly:
   - if there is at least one `Completion`, `Reward` must not close the quest and must not issue final reward until all `Completion` stages are completed;
   - if there are no `Completion` stages, `Reward` may accept objectives directly.
3. Dialog rule was also fixed explicitly:
   - each `offer`, `completion`, and `reward` action should have its own dialog per NPC.
4. Editor UX target was clarified:
   - `NPC Flow` should move toward universal NPC blocks;
   - every block must have its own local trigger picker;
   - the top picker `Открыть / добавить NPC` is helpful, but not sufficient as the only selector;
   - block 1 is usually the giver (`Offer`), middle blocks are `Completion`, final block is `Reward`.
5. User is OK not preserving old player-history compatibility if it blocks the new model, but still wants the default quest list to remain meaningful and based on known old quests.
6. Separate mechanic still not implemented:
   - tracking the exact issued item instance for courier-style quests remains an idea, not a finished feature.

## Update 2026-04-27 - editor cleanup after NPC Flow transition

1. In `Support/JSON_Quvest/app.js`, old duplicate editor layers were still left in the file after the move to `NPC Flow`.
2. Cleanup done:
   - removed extra old `renderQuestEditor(...)` declarations;
   - removed old dead helper for manual trigger-role checkbox sections;
   - removed old dead completion/reward action section renderers;
   - removed the earlier duplicate `normalizeData(...)`, leaving the final version with `version 3`, role seeding, and `triggerActions` sync.
3. Practical result:
   - `app.js` now has one actual quest editor entry point instead of several competing historical copies;
   - the active editor model is the current `NPC Flow` version with:
     - top picker `Открыть / добавить NPC`;
     - local picker `NPC / trigger этого блока`;
     - empty card `Новый блок цепочки`.
4. Verification:
   - `node --check P:\Silver_77_Quests\Support\JSON_Quvest\app.js` passes;
   - editor server from `P:\Silver_77_Quests\Support\JSON_Quvest\server.ps1` responds on `http://127.0.0.1:4173/api/health` with `{"ok":true}`.
5. Still not fully verified:
   - no interactive browser session was run in this pass;
   - universal-block behavior still needs a live UI pass inside the running editor.

## Update 2026-04-27 - editor JSON aligned to mandatory Offer + Reward

1. The editor data pass was completed before any build/testing work:
   - `P:\Silver_77_Quests\Support\JSON_Quvest\Silver_77_Quests.json`
   - `P:\Silver_77_Quests\Support\JSON_Quvest\editor-draft.json`
2. Product rule now reflected directly in editor data:
   - every quest has at least one `offerTriggerIds`;
   - every quest has at least one `rewardTriggerIds`;
   - `completionTriggerIds` stays empty for direct `Offer -> Reward` quests and is only used for real intermediate stages.
3. Template/default JSON was normalized:
   - returned to the meaningful 4-quest baseline;
   - simple quests now use explicit `Offer + Reward` on the same NPC instead of leaving an old fake `completion`;
   - `trigger.questIds` were resynced with the active role arrays.
4. Draft JSON was normalized without losing the chain scenario:
   - simple quests were moved to `Offer -> Reward`;
   - the chained quest `quest_hunter_2` stays `Offer -> Completion -> Reward`;
   - the broken placeholder `quest_new` was removed from the draft.
5. `triggerActions` now mirror the active roles in both files:
   - simple quests have `offer` + `reward` actions;
   - the chained quest has `offer`, `completion`, and `reward` actions on the expected trigger IDs.
6. Verification done:
   - both JSON files parse successfully;
   - a local structure check confirmed that role arrays, `triggerActions`, and `trigger.questIds` match each other.
7. Still not done in this pass:
   - no interactive browser/editor pass yet after this data alignment;
   - no DayZ build or in-game test yet after this data alignment.

## Update 2026-04-27 - first live UI signal after data alignment

1. During the visual pass in the running editor at `http://127.0.0.1:4173/`, the user confirmed that quest sorting is working again in the quest list.
2. Why this matters:
   - the editor is no longer obviously broken at the list/navigation layer after the JSON role cleanup;
   - it is now reasonable to continue the live pass deeper into per-quest role blocks and dialog fields.
3. Still to verify visually:
   - direct `Offer -> Reward` quest blocks;
   - chained `Offer -> Completion -> Reward` blocks;
   - local trigger picker behavior inside each NPC block;
   - presence/editability of per-role dialog fields.

## Update 2026-04-27 - NPC Flow restrictions for unique role blocks

1. The live UI feedback exposed a real editor-model problem:
   - an already configured NPC block could still be retargeted to another trigger;
   - this made `Offer`, `Completion`, and `Reward` feel too interchangeable inside one universal block;
   - adding a second NPC for chain stages was not clear enough in the current UI.
2. Editor behavior was tightened in `Support/JSON_Quvest/app.js`:
   - if a trigger block already has any active role, its local trigger picker is no longer used to move that block to another NPC;
   - such blocks are now shown as fixed to their current trigger/NPC;
   - trying to move a role-bound block now shows a warning to add a separate chain block instead.
3. Flow focus was also improved:
   - choosing an NPC from the top `Открыть / добавить NPC` picker now opens that block in focused mode;
   - choosing an NPC in the empty `Новый блок цепочки` card now also opens that new block directly for role assignment.
4. Product guidance in the UI was clarified:
   - `Offer` is start-only and does not close the quest;
   - `Reward` is mandatory even when it points to the same NPC as `Offer`;
   - if at least one `Completion` exists, `Reward` must wait until all Completion stages are done.
5. Offer summary text was clarified too:
   - if there are Completion stages, Offer points the reader toward Completion;
   - if there are no Completion stages, Offer now explicitly says that final closure still happens only via Reward.
6. Verification done:
   - `node --check P:\Silver_77_Quests\Support\JSON_Quvest\app.js` passes after this pass.
7. Still to verify live:
   - whether adding a second NPC via the empty chain block now feels obvious enough in the running editor;
   - whether the locked current-block behavior removes the confusion seen in the previous UI pass.

## Update 2026-04-27 - regression fix for adding the second NPC block

1. The first restriction pass introduced a UI regression:
   - after choosing/focusing an NPC, the editor was being pushed too aggressively into a focused view;
   - because of that, the empty `Новый блок цепочки` card effectively disappeared from the practical flow;
   - this made it feel like there was no tooling left to create the second NPC block.
2. The flow behavior was corrected in `Support/JSON_Quvest/app.js`:
   - quest editing actions now fall back to `Активные блоки` instead of hiding the chain builder;
   - the top `Открыть / добавить NPC` picker still opens/focuses the chosen NPC, but no longer removes the visible chain-construction affordance;
   - the empty block for the next NPC should remain available in the normal editing path.
3. Verification done:
   - `node --check P:\Silver_77_Quests\Support\JSON_Quvest\app.js` passes after the regression fix.
4. Next live check:
   - refresh the editor and confirm that a second NPC can again be added from the visible chain UI.

## Update 2026-04-29 - runtime single-trigger normalization and app.js cleanup

1. The next micro-layer after the editor role-model pass is now closed in runtime too:
   - `Silver_77_Quests_Server/scripts/4_World/QuestServerManager.c` now normalizes `offerTriggerIds` and `rewardTriggerIds` down to one trigger during seed/normalize;
   - if the role is empty, server seed now fills it only with the first `assignedTriggerId`;
   - `GetQuestOfferTriggerIds(...)` and `GetQuestRewardTriggerIds(...)` now return only the first valid trigger.
2. The same rule was mirrored on client:
   - `Silver_77_Quests_Client/scripts/4_World/QuestClientManager.c` now applies the same single-trigger normalization for `Offer` and `Reward`;
   - client getters for `offer/reward` were tightened to one trigger as well.
3. `Completion` was intentionally left untouched:
   - `completionTriggerIds` still remains a multi-trigger role.
4. A small editor cleanup pass was also applied in `Support/JSON_Quvest/app.js`:
   - remove/hide/rename trigger paths now keep `offerTriggerIds` and `rewardTriggerIds` single-role instead of using the generic array path;
   - the old validator and old multi-role flow card were physically removed from the live path in `app.js`;
   - one more duplicate helper name was cleaned up so the file no longer keeps hidden `function` override collisions for `buildQuestRoleTextForTrigger(...)`;
   - the active role-driven path stays on:
     - `validateData(...)`
     - `validateTriggerRoleArray(...)`
     - `renderQuestRoleFlowCard(...)`
5. Verification done:
   - `node --check P:\Silver_77_Quests\Support\JSON_Quvest\app.js` passes after this pass.
6. Still not done:
   - live editor checks on real JSON are still needed;
   - in-game build/rebuild/publish and game-side route testing remain user-side.

## Update 2026-04-29 - default save paths and accepted editor baseline

1. User manually walked through all current quests in the editor and confirmed the role blocks look workable:
   - transfer chains and return-for-reward flows are now assembled in the editor;
   - secondary rewards and per-role dialogs are present in the current JSON baseline.
2. The accepted default saved JSON was fixed explicitly:
   - `P:\Silver_77_Quests\JSON_Quvest\Silver_77_Quests.json`
   - `P:\Silver_77_Quests\JSON_Quvest\Silver_77_Quests_BackUP.json`
3. Default config was aligned with the working paths:
   - `Support\JSON_Quvest\editor-config.json` now points to the root `P:\Silver_77_Quests\JSON_Quvest` save/backup files;
   - temporary `editor-config.local.json` override is no longer needed for the normal path.
4. Current accepted baseline:
   - `version = 3`
   - `quests = 5`
   - includes `quest_hunter_2`
5. UI clarity pass:
   - NPC role blocks now show bigger red stage labels:
     - `СТАРТ КВЕСТА`
     - `ПРОМЕЖУТОЧНОЕ ЗАДАНИЕ`
     - `ЗАВЕРШАЮЩИЙ`
6. Verification done:
   - launcher/config path logic now reflects the user-confirmed working save targets;
   - `node --check P:\Silver_77_Quests\Support\JSON_Quvest\app.js` should be rerun after the label pass.


## Update 2026-04-29 - baked accepted quest baseline into the server mod defaults

1. The accepted editor baseline is no longer only external JSON:
   - `P:\Silver_77_Quests\Silver_77_Quests_Server\scripts\4_World\QuestServerManager.c`
   - `CreateDefaultQuestConfig()` now mirrors the accepted `P:\Silver_77_Quests\JSON_Quvest\Silver_77_Quests.json`
   - baseline shape: `version = 3`, `quests = 5`
2. The baked default config now includes the current real content:
   - updated `quest_hunter_1`
   - updated `quest_fisherman_1`
   - updated `quest_Rasputin_1`
   - updated `quest_fisherman_2`
   - new `quest_hunter_2`
3. The chain quest is now part of built-in defaults too:
   - `quest_hunter_2` starts on `hunter_trigger`
   - completes on `Rasputin_1_trigger`
   - returns for reward on `hunter_trigger`
4. Trigger quest ordering was aligned with the accepted JSON baseline:
   - `hunter_trigger` => `quest_hunter_2`, `quest_hunter_1`
   - `fisherman_trigger` => `quest_fisherman_1`, `quest_fisherman_2`
   - `Rasputin_1_trigger` => `quest_Rasputin_1`, `quest_hunter_2`
5. Verification done in this pass:
   - `node --check P:\Silver_77_Quests\Support\JSON_Quvest\app.js` passes after the stage-label UI pass
   - source inspection confirms the built-in server defaults now contain `quest_hunter_2`, the `completion` link, and the updated trigger quest orders
6. Still not done:
   - no rebuild of `Silver_77_Quests_Client` / `Silver_77_Quests_Server`
   - no clean-profile in-game validation after baking these defaults
   - `SplitMods\` was left untouched and remains a reserve copy only
7. Small reference-data sync also done in the same pass:
   - `P:\Silver_77_Quests\JSON_Quvest\item-stack-rules.json` stays the single live source for stack rules
   - current mirrored rule set includes `Ammo_12gaPellets = 10`
   - `Support\JSON_Quvest\server.ps1` now reads the root `JSON_Quvest\item-stack-rules.json`
   - `Support\JSON_Quvest\item-stack-rules.json` was removed so the editor no longer has two competing stack-rule copies
8. After the first rebuild attempt exposed that the default quest JSON was still not being created on a clean profile:
   - `Silver_77_Quests_Server/scripts/4_World/QuestServerManager.c`
   - `Silver77_SaveQuestConfigFile(...)` was switched from manual `JsonSerializer + OpenFile("$profile:...")` writing to `JsonFileLoader<Silver77_QuestConfig>.JsonSaveFile(...)`
   - the save path now matches the already working player-data persistence style more closely

## Update 2026-04-30 - server bootstrap fallback documented, live runtime blocker still open

1. One more server-side fallback was added after the live test still failed:
   - `P:\Silver_77_Quests\Silver_77_Quests_Server\scripts\4_World\QuestServerRPC.c`
   - `modded class PlayerBase`
   - `EEInit()` now logs a bootstrap marker and calls `QuestServerManager.EnsureQuestNpcsSpawned();`
2. Intent:
   - if another mod skips the normal `MissionServer.OnInit()` super chain, the quest system gets another chance to load/create config and spawn NPCs when the first real player initializes on the server
3. Source inspection after that pass showed:
   - default quest trigger NPC classes are valid vanilla survivor classes
   - the more likely blocker is still runtime init / missing fresh deploy / broken hook chain, not obviously bad NPC classnames
4. Live issue still not closed:
   - automatic creation of `profiles\Silver_77_Quests\Silver_77_Quests.json` was still not confirmed
   - quest NPC spawn was still not confirmed
   - manual profile JSON placement still did not restore quest NPCs on the live server
5. Next evidence needed:
   - RPT lines for `MissionServer.OnInit called`
   - `Loading quest config from:`
   - `Saving quest config to:`
   - `PlayerBase.EEInit bootstrap for quest server:`
   - `Quest NPC cache is empty, spawning configured NPCs now`
   - `Spawned quest NPC ...`

## Update 2026-04-30 - server addon registration mismatch found in live RPT

1. Live server inspection confirmed:
   - `Silver_77_Quests_Server.pbo` exists on the live machine in `@Silver_77_Quests_Server\addons`
   - its SHA256 matches the locally built PBO
2. Built PBO content inspection also confirmed the package is not empty:
   - it contains `config.cpp`
   - `QuestServerManager.c`
   - `QuestServerRPC.c`
   - `MissionServer.c`
3. But live RPT still showed that:
   - `CacheSpawner` from `-servermod` is added as package and appears in the script addon list
   - `Silver_77_Quests_Server` does not appear in those same places
   - `Silver_77_Quests_Client` does appear normally
4. Based on that, the working blocker moved from “quest JSON/runtime data” to “server addon registration / script module wiring”.
5. Immediate config mitigation applied:
   - `P:\Silver_77_Quests\Silver_77_Quests_Server\config.cpp`
   - removed `Silver_77_Quests_Client` from `requiredAddons[]`
   - reduced `dependencies[]` to `{"World", "Mission"}`
6. Rationale:
   - the server addon only exposes `worldScriptModule` and `missionScriptModule`
   - the old `Game` dependency did not have a matching module definition
   - the direct patch dependency on the client addon may also have interfered with the clean `-servermod` registration path

## Update 2026-04-30 - live retest still shows no server addon registration

1. Rebuild/redeploy after the `config.cpp` mitigation was tested live.
2. Result:
   - `profiles\Silver_77_Quests\Silver_77_Quests.json` still was not created
   - quest NPCs still were absent
   - there were still no `[Silver_77_Quests] ...` runtime logs
3. Most important fresh RPT evidence:
   - `@CacheSpawner` from the same `-servermod` line still appears in `Adding package ...` and `SCRIPT : CacheSpawner`
   - `@Silver_77_Quests_Server` still does not appear in either place
4. Current working blocker:
   - the issue is still best modeled as “`Silver_77_Quests_Server.pbo` is not being registered/loaded by the engine as a server addon package”
   - not as a quest runtime logic bug

## Update 2026-04-30 - duplicate internal prefix discovered in live client/server PBOs

1. Live header inspection of both deployed PBOs showed:
   - `Silver_77_Quests_Client.pbo` currently carries `prefix = Silver_77_Quests_Server`
   - `Silver_77_Quests_Server.pbo` also carries `prefix = Silver_77_Quests_Server`
2. This is now the strongest concrete root-cause candidate.
3. Working interpretation:
   - the client addon was likely packed with the wrong Addon Builder prefix override
   - client/server now collide on the same internal addon prefix
   - that collision may explain why the server addon never shows up in `Adding package ...` or `SCRIPT : ...`
4. Next external check/fix:
   - rebuild the client PBO with the intended distinct client prefix instead of `Silver_77_Quests_Server`
   - redeploy both PBOs
   - re-check RPT for `Adding package ... Silver_77_Quests_Server.pbo` and `SCRIPT : Silver_77_Quests_Server`
## Update 2026-04-30 - runtime recovered, potato quest data corrected

Closed in this pass:

1. Live quest runtime is back after the client PBO was rebuilt with the correct distinct prefix:
   - `profiles\Silver_77_Quests\Silver_77_Quests.json` now auto-creates again;
   - quest NPCs are visible again;
   - quests can be accepted again.
2. A new gameplay bug was isolated immediately after that recovery:
   - `quest_hunter_2` advanced at Rasputin without actually taking the potatoes;
   - `quest_hunter_1` still expected the wrong potato class for the large cumulative hand-in.
3. Root cause:
   - `quest_hunter_2` had `giveItems = PotatoSeed x12` but `objectives = []`;
   - `quest_hunter_1` still pointed at `PotatoSeed x120`, while the intended accumulative item in project docs/examples is `Potato`.
4. Data/runtime defaults were aligned, then corrected after a later live review:
   - final accepted item is still `PotatoSeed`, not peeled `Potato`;
   - `quest_hunter_1` objective is back on `PotatoSeed`;
   - `quest_hunter_2` give item is back on `PotatoSeed x12`;
   - `quest_hunter_2` still keeps the real `PotatoSeed x12` completion objective with `allowPartialTurnIn = true`.
5. Synced files:
   - `P:\Silver_77_Quests\Silver_77_Quests_Server\scripts\4_World\QuestServerManager.c`
   - `P:\Silver_77_Quests\JSON_Quvest\Silver_77_Quests.json`
   - `P:\Silver_77_Quests\JSON_Quvest\Silver_77_Quests_BackUP.json`
   - `P:\Silver_77_Quests\Support\JSON_Quvest\Silver_77_Quests.json`
   - `P:\Silver_77_Quests\Support\JSON_Quvest\Silver_77_Quests_BackUP.json`
   - `P:\Silver_77_Quests\Support\JSON_Quvest\editor-draft.json`

Still not closed:

1. Rebuild/redeploy and live retest are still needed.
2. Rasputin must now be verified to block `quest_hunter_2` until the potatoes are actually deposited.
3. Voron must be verified to accept the intended `Potato` class for the `x120` quest.

## Update 2026-04-30 - client quest chain UX made explicit in NPC window

Closed in code:

1. Active chain quests no longer rely only on the journal to remain understandable.
2. `P:\Silver_77_Quests\Silver_77_Quests_Client\scripts\4_World\QuestClientManager.c`
   - `active` and `reward_pending` quests now remain visible on assigned chain NPCs.
3. `P:\Silver_77_Quests\Silver_77_Quests_Client\scripts\5_Mission\QuestUI.c`
   - the NPC quest window now builds a visible `Линия квеста` block;
   - the same window also shows `Контекст NPC`, so the player can see the next chain step without leaving the NPC dialog;
   - button captions were made stage-aware for accept / deposit / complete / reward;
   - a first dedicated `DialogText` area was also added to the NPC layout so stage dialog can be separated from the main description.

Still not closed:

1. This UX layer still needs a live client rebuild and in-game readability check.
2. Need to confirm the same behavior at both primary and secondary chain NPCs.

## Update 2026-04-30 - user clarified final chain/dialog UX target

Closed in planning / handoff only:

1. The current inline `Линия квеста` text inside the main NPC description must be treated as a temporary stopgap, not the final UX.
2. Final target requested by the user:
   - a separate dedicated chain panel/window in the NPC interaction UI;
   - the same separate chain panel/window in the quest journal;
   - separate dedicated dialog area/window for the current NPC stage;
   - history of already visited dialogs;
   - unvisited stages show only NPC / trigger names plus the stage goal if it was already told by the first quest-giver, but not the future stage's own dialog text;
   - completed chain stages should be visually marked as completed.
3. Additional JSON/editor requirement captured:
   - raw item `className` values must stop leaking into player-facing UI;
   - add a separate human-readable display-name field for items and render that field in UI/journal/NPC windows.
4. Additional content consistency issue captured:
   - live review suggests the latest potato content mapping may still be wrong;
   - live output showed peeled `Potato` being given;
   - Rasputin dialog also did not match the intended stage owner;
   - this should be treated as a possible editor/JSON remap mistake before the next major content bake.

Still not closed:

1. None of the above separate-panel UX has been implemented yet.
2. The current inline chain/context text should likely be replaced, not just polished.

## Update 2026-04-30 / 2026-05-01 - roadmap rewritten to the final UI/data direction

Execution order is no longer based on “many separate windows”. The user clarified the preferred architecture more precisely:

1. Lock the NPC window to a hard UI contract:
   - left = quest titles + state markers only;
   - center = status / goal / progress / reward only;
   - right = trigger route only;
   - bottom = one shared scrolling dialog journal only.
2. Replace the temporary left-side chain prototype with the final right-side trigger column.
3. Build one unified lower dialog journal instead of popup-first / split-dialog solutions.
   - the current/live stage should be presented first at the top;
   - older history should sink lower in the same scroll area.
4. Add trigger-driven focus:
   - right-side trigger click should focus the matching lower dialog/history block;
   - the matching block should be highlighted in gold.
5. Secondary / optional stages without strict authored order should be stored and displayed by first visit / first activation order.
6. Mirror the same route/history behavior in the quest journal instead of inventing a different journal model.
7. Extend JSON/editor for player-facing fields and then run live validation on content/remap/runtime.

## Update 2026-04-30 - dedicated NPC chain panel extracted from main description

1. The first roadmap step is now implemented in the client NPC window.
   - `Silver_77_Quests_Client/gui/QuestMenu.layout` now includes a separate left-side `ChainPanel` / `ChainText` block.
   - the quest list was shortened to make room for that dedicated route area.
2. `Silver_77_Quests_Client/scripts/5_Mission/QuestUI.c` now feeds chain text into `m_ChainText` instead of appending it into the main description body.
3. The main description block now stays focused on:
   - quest description;
   - status / requirements;
   - NPC context;
   - objectives / rewards;
   - server wait state.
4. The journal still does not have the matching separate chain panel yet.
   - that is now the next active roadmap step.

5. This left-side `ChainPanel` is now considered only a prototype.
   - after the 2026-05-01 UX discussion, the final route must move to the right side of the NPC window;
   - do not treat the current left-side placement as the final intended design.

## Update 2026-04-30 - immediate rebuild / retest checklist recorded

1. Rebuild client.
2. Rebuild server or refresh live quest JSON from root baseline.
3. Check that `quest_hunter_2` gives `PotatoSeed x12`.
4. Check that Rasputin requires the real `PotatoSeed x12` hand-in.
5. Check that Voron accepts `PotatoSeed` for the large potato quest.
6. Check that current NPC dialog is not duplicated in the upper description body and is ready to live in the one shared lower scrolling dialog journal.
7. Check that Rasputin/Voron stage-dialog ownership is correct.
8. Check that chain quests stay visible across all participating NPCs.
9. Check that the temporary route prototype remains readable until the final right-side trigger column replaces it.

## Update 2026-05-01 - accept-flow stabilization and “one shared scroll” decision

1. Live test feedback exposed two separate issues:
   - the NPC window still felt visually mixed;
   - `quest_fisherman_2` / `Поставка медицины` sometimes felt like it accepted only after repeated clicks.
2. The quest data for `quest_fisherman_2` was re-checked first.
   - no obvious invalid requirement chain or malformed JSON structure was found;
   - the problem looked more like accept-flow / feedback / trigger-context drift than a clearly broken quest definition.
3. First code-level stabilizer already added:
   - `MissionGameplay.OnUpdate()` stops re-running trigger focus checks while the quest UI or journal is open;
   - this keeps `m_CurrentTriggerId` / `m_CurrentQuestIds` stable for the active interaction.
4. The NPC upper description body also stopped injecting `quest.description`.
   - that upper area should stay about state / goal / progress / reward;
   - live NPC dialog is intended to live in the lower dialog area only.
5. The user then locked the preferred dialog/history direction:
   - do not build many separate dialog windows;
   - use one shared scrolling lower dialog journal;
   - right-side trigger clicks should highlight the related lower dialog block in gold instead of opening a popup-first flow.
6. Additional presentation rule clarified after that:
   - the current stage should stay at the top of the lower journal;
   - older dialogue/history should move lower as past content.
7. Secondary / optional stages currently do not have strict authored order.
   - for now, their history order should be first visit / first activation order.
8. Current unresolved items going into the next session:
   - repeated-click feeling on quest acceptance still needs rebuild + live retest after the trigger-freeze stabilizer;
   - lower dialog area is not yet a true scrolling history journal widget;
   - the new persistent `stageVisits` model exists, but rich UI rendering is still only partially wired;
   - right-side route focus still needs a true gold-highlight presentation instead of a text marker.

## Update 2026-05-01 - stage activation history added to player progress

1. The project now has a real data model for future dialog-history UI instead of relying only on current status.
   - `PlayerQuestStageVisit` was added to player progress data.
   - `PlayerQuestProgress` now includes `stageVisits`.
2. Server-side progress migration was extended:
   - old player profiles automatically get an empty `stageVisits` array on load.
3. Server runtime now records stage activation order on first entry:
   - `offer` is recorded when a quest is accepted;
   - `completion` is recorded when a completion stage is actually finished;
   - `reward` is recorded when the final reward stage is claimed.
4. This is the basis for the agreed UI direction:
   - one shared scrolling lower dialog journal;
   - secondary / optional stage history shown by first activation order.
5. Important limitation:
   - old already-active profiles cannot reconstruct their earlier dialog history retroactively.
6. Test note:
   - when we start validating dialog-history order live, clear `profiles\\Silver_77_Quests\\players\\*` first for a clean run.

## Update 2026-05-01 - lower NPC dialog block switched to history-first journal builder

1. `QuestUI.c` no longer treats the lower dialog block as only “current NPC line”.
2. `BuildQuestDialogText(...)` now first tries a new journal-builder path:
   - reads `stageVisits` in sorted activation order;
   - renders already visited stage dialogs first;
   - appends the current live stage only if it has not been recorded yet.
3. This is still only the foundation of the final UX:
   - right-side trigger route was the next step after this foundation;
   - gold highlight / focus by trigger click was still pending at this point;
   - journal output currently covers dialog text, not richer per-stage reward/goal payloads yet.

## Update 2026-05-01 - right-side route list wired into NPC window

1. The temporary left-side route prototype is no longer the active direction in code.
   - `QuestMenu.layout` now has a real `TriggerRouteListbox` on the right side of the NPC window.
   - central description width was reduced to make room for the route column.
2. `QuestUI.c` now tracks visible route keys and selected route focus.
   - selecting a quest resets route focus;
   - selecting a route entry updates the lower journal context.
3. The lower journal now has a first working route-focus behavior:
   - current stage still stays on top;
   - older history stays below;
   - a route-selected historical block is marked with a text focus marker (`>>`).
4. This is not the final presentation yet.
   - the right-side route exists;
   - the in-journal focus is not yet true gold-colored text highlighting;
   - richer per-stage goal/reward payload is still not rendered in the lower journal.

## Update 2026-05-01 - NPC route / description / dialog panels separated

1. `QuestMenu.layout` no longer lets the lower dialog area physically overlap the right-side trigger route.
2. The NPC window now has three distinct visual zones:
   - central description panel;
   - right-side route panel;
   - lower dialog panel.
3. This matches the agreed contract better and should make the next live visual pass much less ambiguous.
4. Still pending after this geometry split:
   - real scroll behavior for the shared dialog journal;
   - true gold-highlight focus inside the lower journal;
   - richer per-stage payloads in the journal itself.
5. New live-test clarification after item hand-in:
   - the server appears to consume the required partial-delivery amount correctly;
   - extra items remain once the requirement is already satisfied, which strongly suggests the deposit math itself is alive on the server;
   - the broken layer is now considered client-side progress refresh / player-data sync after partial turn-in, because the UI still shows `0 / N`, keeps the submit button active, and eventually times out with `waiting for server`.
6. Diagnostic instrumentation added immediately after that:
   - client quest RPC now logs received player-progress payload length and decoded quest-entry count;
   - if synced player data arrives with empty `steamId`, the client now applies a local identity fallback before storing it;
   - `ApplySyncedPlayerData(...)` now logs how many non-zero objective-progress entries were actually applied on the client.

## Update 2026-05-01 - static NPC UI text encoding and panel contrast pass

1. Rewrote [QuestMenu.layout](/P:/Silver_77_Quests/Silver_77_Quests_Client/gui/QuestMenu.layout) and resaved it in `windows-1251`.
2. This pass specifically targets the live `mojibake` seen in the NPC window static text:
   - title;
   - panel labels;
   - action buttons.
3. Added a stronger visual split between blocks:
   - darker backgrounds for description / route / dialog panels;
   - brighter description heading;
   - gold-accent headings for route and dialog.
4. This pass does **not** touch server deposit logic.
5. Still open:
   - client-side progress refresh after partial hand-in;
   - true scrolling lower journal;
   - true gold focus inside journal text;
   - reduced action buttons / extra helper textbox on the right.

## Update 2026-06-12 - quest editor reward flow cleanup

1. The editor reward model was clarified around the strict `NPC Flow` contract:
   - `Offer` uses `giveItems` for items issued when the quest starts;
   - `Completion` blocks may use their own local `triggerActions[].rewards`;
   - the final `Reward` block uses its own local `triggerActions[].rewards` and closes the quest.
2. Because of that strict flow, the visible `Root Rewards (общие fallback rewards)` editor section was removed from `JSON_Quvest/app.js`.
   - `quests[].rewards` still exists in the JSON/server contract as a legacy fallback;
   - the editor no longer exposes it as a normal block, because it is not attached to a specific NPC/stage and confused the visual chain.
3. The empty `Новый блок цепочки` card is now hidden once a final `Reward` block already exists.
   - after the final reward, the editor no longer suggests that another chain stage can be added;
   - the toolbar hint now explains that the final reward is already set.
4. Verification:
   - `node --check D:\Dayz\Silver_77_Quests\JSON_Quvest\app.js` passes;
   - live editor reload at `http://127.0.0.1:4173/` showed `Root Rewards` absent, `NPC Flow` present, final `ЗАВЕРШАЮЩИЙ` present, and zero empty chain blocks after it.
5. Data note:
   - existing `JSON_Quvest/Silver_77_Quests.json` and `JSON_Quvest/Silver_77_Quests_BackUP.json` were already modified in the worktree and were not touched by this editor UI cleanup.
