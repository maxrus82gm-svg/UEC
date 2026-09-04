# Codex Control Context

## UPDATE 2026-04-26 - editor single-root rule

- Канонический editor root: `P:\Silver_77_Quests\Support\JSON_Quvest`
- Legacy `P:\Silver_77_Quests\JSON_Quvest` = только совместимая обёртка
- Относительные editor paths (`savePath`, `backupPath`, draft, stack rules) всегда трактуются от папки реально запущенного `server.ps1`
- Основной JSON редактора и backup теперь ожидаются рядом: `Silver_77_Quests.json` и `Silver_77_Quests_BackUP.json`
- Если редактор ведёт себя как старая версия, сначала проверять active editor root, а не квестовую runtime-логику

## UPDATE 2026-04-26 - NPC Flow picker behavior

- У каждого блока `NPC Flow` теперь должен быть собственный selector `NPC / trigger этого блока`.
- Этот selector отвечает за то, какой trigger попадёт в генерацию JSON для ролей блока.
- При смене trigger у карточки редактор переносит роли и action-данные на новый trigger.
- Редактор может досеять отсутствующие `offer` и `reward` из `trigger.questIds`, чтобы старый дефолтный список не развалился; `completion` больше не досеивается автоматически.

## UPDATE 2026-04-26 - mandatory offer/reward chain

- Новая базовая схема: `Offer + Reward` обязательны, `Completion` опционален.
- `Offer` = где квест берётся.
- `Completion` = промежуточная передача / сдача; таких этапов может не быть или может быть несколько.
- `Reward` = финальное закрытие квеста и выдача награды.
- Если в квесте есть Completion-trigger, Reward обязан проверить, что все они выполнены, иначе квест не закрывается и награда не выдаётся.
- Если Completion-trigger нет, Reward принимает цели напрямую и закрывает квест.
- Все три роли теперь имеют action-диалог через `triggerActions`: `offer`, `completion`, `reward`.
- В split runtime прогресс игрока хранит `completedCompletionTriggerIds`, чтобы Reward знал, какие Completion-этапы уже закрыты.

Дата: 20.04.2026
Проект: `D:\Dayz\Silver_77_Quests`

## Зачем этот файл

Перед этим файлом сначала открыть:

- `Documentation/CODEX_START_HERE.md`

Это короткий контрольный контекст для продолжения работы после перезапуска или на следующий день. После `CODEX_START_HERE.md` читать его первым, потом при необходимости открывать:

- `Documentation/CODEX_CONTEXT_2026-04-20.md`
- `Documentation/CODEX_EMERGENCY_CONTEXT.md`
- `Documentation/CODEX_WORKLOG.md`
- `Documentation/REPORT_2026-04-18.md`
- `Documentation/RUSSIAN_ENCODING.md`

Актуальный короткий срез после последних продуктовых уточнений лежит в `Documentation/CODEX_SESSION_HANDOFF_2026-04-29.md`.

## UPDATE 2026-04-29 - жесткая уникальность role-блоков

- `NPC Flow` теперь нужно считать не просто role-driven, а role-strict.
- Один блок = одна роль.
- `Offer` должен быть уникальным.
- `Reward` должен быть уникальным и обязательным.
- `Completion` может быть несколько.
- Один и тот же NPC может повторяться, но только как отдельный блок с другой ролью.
- `Completion` может иметь собственный диалог и локальную награду, но не может закрывать квест.
- `Reward` остается единственной финальной точкой закрытия квеста.
- Пользователь разрешил при необходимости ломать старый JSON / draft ради чистой новой схемы.
- `P:\Silver_77_Quests` не считать git-репозиторием: git-команды здесь не являются обязательной частью старта.
- Пользователь сам делает build / rebuild / publish; Codex не должен брать на себя сборку без новой прямой просьбы.
- Русский текст и UTF-8 кодировка обязательны.
  - Любая mojibake-строка (`Р...`, `С...`, `Ð...`) считается ошибкой проекта.
  - Битый русский текст нельзя оставлять "на потом", даже если логика фичи уже работает.

`Documentation/CODEX_CONTEXT_2026-04-20.md` и нижние разделы этого файла читать как более старый архитектурный фон.

## UPDATE 26.04.2026 - читать первым

- Запускатор редактора в `Support\JSON_Quvest` уже починен.
- Старые запускаторы в `P:\Silver_77_Quests\JSON_Quvest` теперь работают как обертка на актуальную папку.
- Основная модель квестов больше не строится вокруг ручной `видимости`.
  - `trigger.questIds` = список и порядок квестов у NPC
  - игровая видимость считается по ролям `offer / completion / reward` и по статусу квеста
- В редакторе уже есть action-карточки для `offer`, `completion` и `reward`.
- `NPC Flow` в редакторе теперь:
  - по умолчанию показывает только активные NPC-блоки
  - умеет открывать новый NPC через встроенный picker `Открыть / добавить NPC`
  - использует универсальные карточки с ролями `offer / completion / reward`
  - имеет режимы `Активные блоки / Выбранный NPC / Все NPC`
- Восстановление черновиков в редакторе обновлено:
  - драфты сравниваются по `updatedAt`
  - старый файловый draft больше не должен перетирать более свежий local draft
  - legacy-драфты без `updatedAt` можно больше не считать надежным источником
- На клиенте и сервере уже заложен `reward_pending` и отдельный reward trigger.
- Старый дефолтный набор квестов должен сохраниться по смыслу; если role-полей еще нет, они досеиваются из `trigger.questIds`.
- Пользователь не требует поддерживать старую пользовательскую историю/совместимость, если это мешает новой модели, но дефолтный список старых квестов должен остаться.

## Текущее состояние

Мод `Silver_77_Quests` - система квестов для DayZ.

Сейчас реализовано:

- меню NPC по `F`;
- журнал активных квестов по `J`;
- журнал по `J` намеренно не блокирует движение персонажа;
- серверная обработка принятия/сдачи через RPC;
- сохранение прогресса игроков в `$profile`;
- накопительная сдача предметов через `allowPartialTurnIn`;
- пользовательские стартовые квесты в дефолтном конфиге.

## Стартовые квесты

Дефолтный генератор:

`scripts/4_World/QuestManager.c` -> `CreateDefaultQuestConfig()`

Стартовый JSON-шаблон:

`Documentation/STARTER_QUEST_CONFIG.json`

Текущие стартовые квесты:

1. `quest_fisherman_1`
   - название: `Картошечка с маслицем`
   - цели:
     - `PotatoSeed` x20
     - `PleurotusMushroom` x1
     - `MacrolepiotaMushroom` x2
     - `BoletusMushroom` x1
   - все цели с `allowPartialTurnIn = true`
   - награда: `Ammo_12gaPellets` x7
   - NPC hint: `[F] Коля Ворон`

2. `quest_hunter_1`
   - название: `Рыба это вам не картошка!`
   - цель:
     - `Carp` x6
   - цель с `allowPartialTurnIn = true`
   - награда: `Ammo_12gaPellets` x7
   - NPC hint: `[F] Рыбак Гаврила перец`

## Важный серверный момент

Рабочий JSON сервера лежит в:

`profiles/Silver_77_Quests/Silver_77_Quests.json`

Если этот файл уже существует, новый дефолт из PBO не применится. Нужно:

- заменить серверный JSON содержимым `Documentation/STARTER_QUEST_CONFIG.json`;
- или удалить серверный JSON перед стартом, чтобы мод создал новый.

## Последние правки

### ESC для журнала

Файл:

`scripts/5_Mission/mission/MissionGameplay.c`

Добавлено:

- если открыт `MENU_QUEST_JOURNAL_UI`, нажатие `ESC` закрывает журнал;
- обработка стоит до `super.OnKeyPress(key)`, чтобы не открывать системное меню поверх журнала.

### Колесо мыши для журнала

Файл:

`scripts/5_Mission/QuestJournalUI.c`

Добавлено:

- `SelectQuestOffset(int offset)`;
- `OnMouseWheel(...)`;
- колесо вверх выбирает предыдущий активный квест;
- колесо вниз выбирает следующий активный квест;
- выбор зациклен: после последнего идет первый, перед первым идет последний.

## Первое, что проверить завтра

1. Собирается ли PBO без ошибок компиляции.
2. Если есть ошибка по `OnMouseWheel`, проверить сигнатуру override в DayZ scripts. Возможная точка риска:

```c
override bool OnMouseWheel(Widget w, int x, int y, int wheel)
```

3. Если есть ошибка по `KeyCode.KC_ESCAPE`, проверить имя константы ESC в DayZ. Возможная точка риска:

```c
KeyCode.KC_ESCAPE
```

4. В игре:
   - открыть журнал по `J`;
   - нажать `ESC`;
   - журнал должен закрыться;
   - персонаж по-прежнему может двигаться при открытом журнале;
   - колесо мыши должно переключать активные квесты.

## Если колесо не работает

Вероятная причина: журнал не забирает game focus, поэтому mouse wheel может не попадать в `QuestJournalUI`.

Варианты решения:

1. Оставить `ESC`, а перелистывание сделать клавишами `PageUp/PageDown` или стрелками.
2. Временно забирать UI focus только для mouse wheel, но это может сломать желаемое движение персонажа.
3. Ловить альтернативные input events на уровне `MissionGameplay`, если в DayZ есть удобный способ для mouse wheel без focus.

## Не ломать

- Не блокировать движение при открытом `J` без прямой просьбы пользователя.
- Не убирать серверную RPC-логику принятия/сдачи.
- Не рассчитывать, что дефолтный JSON перезапишет существующий серверный JSON.
- `allowPartialTurnIn` должен быть опциональным: старые квесты без него работают по старой логике.

## Следующие идеи пользователя

Пользователь хочет двигаться к защите мода через разделение на клиентский Workshop-мод и приватный серверный `-serverMod`.

Выбранный путь в 2 этапа:

1. Быстрое разделение на client/server:
   - клиентский мод публикуется в Workshop;
   - серверный мод остается приватно на сервере;
   - настоящая логика квестов, наград, проверок, прогресса и JSON уходит в серверный PBO;
   - клиентский PBO оставляет UI, layouts, клавиши и RPC-запросы.

2. Более закрытая серверная архитектура:
   - клиент перестает получать полный конфиг квестов/триггеров;
   - сервер сам проверяет зоны NPC;
   - сервер отправляет клиенту только текущую подсказку, доступные квесты и прогресс, нужный для UI;
   - координаты, полный список квестов, награды и логика остаются на сервере.

## Текущее направление: split client/server

Созданы отдельные исходники:

- `Silver_77_Quests_Client`
- `Silver_77_Quests_Server`

Добавлен сборочный скрипт:

- `build_split_mods.bat`

Добавлена инструкция:

- `Documentation/SPLIT_CLIENT_SERVER.md`

Текущая архитектура первого этапа:

- client PBO содержит общие DTO, UI, layouts, client-side manager `QuestClientManager`, client RPC sync handlers;
- server PBO содержит приватный `QuestServerManager`, server RPC request handlers и `MissionServer` init;
- серверный `config.cpp` зависит от `Silver_77_Quests_Client`, чтобы не дублировать общие классы.

Монолитные `scripts/`, `gui/` и `config.cpp` в корне проекта пока оставлены как резервная рабочая версия. Для split-теста не запускать старый `@Silver_77_Quests` одновременно с `@Silver_77_Quests_Client` и `@Silver_77_Quests_Server`.

Старая папка `SplitMods/` оставлена как резервная копия. Основные split-исходники теперь лежат прямо в корне проекта.

## Что было сделано после обрыва связи

После восстановления связи был проверен свежий split и исправлены заметные проблемы:

1. В `SplitMods/Silver_77_Quests_Client/scripts/5_Mission/QuestUI.c` восстановлены русские UI-строки:
   - `[Активен]`;
   - `[Выполнен]`;
   - `Выберите квест из списка`;
   - `Можно взять/сдать`;
   - `Цели`, `Сдано`, `Награда`.

2. В `SplitMods/Silver_77_Quests_Client/scripts/5_Mission/QuestJournalUI.c` восстановлены русские строки журнала:
   - `Активных квестов нет.`;
   - `Статус: взят`;
   - `Можно внести часть предметов`;
   - `[готово]`, `[не готово]`;
   - `Сдать квест можно у NPC, который его выдал.`

3. В `SplitMods/Silver_77_Quests_Client/scripts/5_Mission/QuestTrigger.c` восстановлена подсказка по умолчанию:
   - `[F] Открыть квесты`.

4. В `SplitMods/Silver_77_Quests_Server/scripts/4_World/QuestServerManager.c` восстановлены русские дефолтные квесты и подсказки:
   - `Картошечка с маслицем`;
   - `Рыба это вам не картошка!`;
   - `[F] Коля Ворон`;
   - `[F] Рыбак Гаврила перец`.

5. Серверный лог и документация согласованы:

```text
[Silver_77_Quests] QuestServerManager initialized
```

6. `Documentation/BUILD.md` обновлен: первым указан рекомендуемый split-вариант, монолитная сборка оставлена как резервная.

## Проверки, которые уже сделаны

- В `SplitMods` не найдено старых ссылок на `QuestManager`, `g_QuestConfig`, `g_PlayerQuestData`, `g_Silver77_Quest`.
- В исправленных split-файлах не найдено типичных битых UTF-8 строк вида `Р’С‹...`.
- `$PBOPREFIX$` проверены:
  - client: `Silver_77_Quests`;
  - server: `Silver_77_Quests_Server`.
- Папки, на которые ссылаются client/server `config.cpp`, существуют.

PBO через DayZ Tools еще не собирался после этих правок.

## Что проверить следующим запуском

1. Запустить `build_split_mods.bat`.
2. Если сборка падает, первым смотреть:
   - `QuestJournalUI.OnMouseWheel` сигнатуру;
   - `KeyCode.KC_ESCAPE`;
   - наличие `@Silver_77_Quests_Client` в `-mod` при запуске сервера.
3. Сервер запускать так:

```bat
-mod=@Silver_77_Quests_Client
-serverMod=@Silver_77_Quests_Server
```

4. В игре проверить:
   - сервер стартует с `@Silver_77_Quests_Client` в `-mod` и `@Silver_77_Quests_Server` в `-serverMod`;
   - клиенту нужен только `@Silver_77_Quests_Client`;
   - в логах есть `MissionServer.OnInit called` и `QuestServerManager initialized`;
   - подсказка NPC отображается по-русски;
   - `F` открывает меню NPC;
   - `J` открывает журнал;
   - `ESC` закрывает журнал;
   - накопительная сдача предметов работает.

## Утренняя проверка 19.04.2026

Проверено после включения диска и восстановления рабочего пути:

- `D:\Dayz\Silver_77_Quests` сейчас не является git-репозиторием, поэтому `git status` недоступен.
- `Documentation/STARTER_QUEST_CONFIG.json` успешно проходит `ConvertFrom-Json`.
- В `Silver_77_Quests_Client` и `Silver_77_Quests_Server` не найдены старые монолитные имена `QuestManager`, `g_QuestConfig`, `g_PlayerQuestData`, `g_Silver77_Quest`.
- В `Silver_77_Quests_Client`, `Silver_77_Quests_Server`, `scripts` и `gui` после правки не найдены типичные mojibake-строки.
- Оставшиеся битые русские комментарии исправлены в:
  - `Silver_77_Quests_Client/scripts/5_Mission/mission/MissionGameplay.c`;
  - `Silver_77_Quests_Server/scripts/4_World/QuestServerRPC.c`;
  - `Silver_77_Quests_Server/scripts/5_Mission/mission/MissionServer.c`.
- `AddonBuilder.exe` найден по пути `D:\SteamLibrary\steamapps\common\DayZ Tools\Bin\AddonBuilder\AddonBuilder.exe`.
- В `build_split_mods.bat` обновлен путь `DAYZ_TOOLS` под найденный DayZ Tools.
- PBO/split-сборка пока не запускалась: скрипт пишет результат в `D:\Dayz\Mods_DONE`, что требует отдельного разрешения на запуск вне sandbox.
- Пользователь уточнил: сборку, выкладку и публикацию он делает сам. Codex не должен запускать build/publish без прямой новой просьбы.

## Корневое разделение 19.04.2026

Для понятной ручной сборки split-исходники вынесены прямо в корень проекта:

- `Silver_77_Quests_Client` - source directory для клиентского Addon Builder;
- `Silver_77_Quests_Server` - source directory для серверного Addon Builder.

`build_split_mods.bat` тоже переключен на эти корневые папки. `SplitMods/` оставлен как старая резервная копия и не должен быть основным рабочим путем.

## Финальная проверка 19.04.2026 перед ручной сборкой

Сборка и публикация не запускались. Пользователь делает build/publish сам.

Проверено:

- структура client/server папок;
- client/server `config.cpp`;
- `$PBOPREFIX$`;
- server `requiredAddons[] = {"DZ_Data", "DZ_Scripts", "Silver_77_Quests_Client"}`;
- отсутствие старых монолитных имен в корневых split-папках;
- отсутствие типичных mojibake-строк в корневых split-папках;
- соответствие layout-путей prefix `Silver_77_Quests`;
- наличие виджетов `QuestListbox`, `DescriptionText`, `AcceptButton`, `CompleteButton`, `CloseButton`, `QuestHintAction`;
- валидность `Documentation/STARTER_QUEST_CONFIG.json` через `ConvertFrom-Json`;
- простой баланс `{}` в `.c`, `.cpp`, `.layout`.

Дополнительно выровнен текст первого стартового квеста с реальными целями: описание теперь просит 20 картошек и 1 белый гриб, как указано в целях.

Остались только известные compile-risk места, которые нужно подтвердить при сборке DayZ Tools:

- `QuestJournalUI.OnMouseWheel(Widget w, int x, int y, int wheel)`;
- `KeyCode.KC_ESCAPE`.

## Русский язык и кодировки

Проблема, которую видели в split-файлах:

```text
Р’С‹Р±РµСЂРёС‚Рµ РєРІРµСЃС‚
```

Это не DayZ "переводит" русский. Это mojibake: русская строка была в UTF-8, потом какой-то инструмент прочитал эти байты как Windows-1251/ANSI или OEM-кодировку, после чего этот уже сломанный текст был снова сохранен в файл.

Почему это у нас повторяется:

1. Windows PowerShell 5.1, `cmd`, батники, старые Windows-инструменты и некоторые редакторы могут использовать разные кодировки по умолчанию.
2. Вывод в консоли может выглядеть сломанным даже если файл нормальный, потому что консоль показывает байты не той кодировкой.
3. Хуже, когда сломанный вывод не просто показали, а записали обратно в `.c`, `.layout`, `.json` или `.md`.
4. Русские комментарии безопасны для логики, но если ломаются строки в кавычках, игрок увидит кракозябры в UI или JSON.

Правила для проекта:

- Все `.c`, `.layout`, `.json`, `.md`, `.bat` держать в UTF-8.
- В VS Code проверять нижний правый угол: должно быть `UTF-8`.
- Не сохранять файл как `Windows 1251`, `ANSI`, `OEM 866`.
- В PowerShell читать русские файлы так:

```powershell
Get-Content -Encoding UTF8 path\to\file
```

- Если нужно писать через PowerShell, явно указывать UTF-8. Лучше не переписывать большие файлы через `>` или `Out-File` без нужной кодировки.
- В bat-файлах, где есть русский вывод, держать:

```bat
chcp 65001 >nul
```

- Если видишь `Р’С‹`, `Рџ`, `СЃ`, `С‚`, `Рќ` внутри строки в кавычках, это почти наверняка уже сломанная строка в файле, ее нужно заменить нормальным русским текстом из резервной версии или документации.

## Для следующего Codex

1. Сначала открыть этот файл.
2. Если задача про split, открыть:
   - `Documentation/SPLIT_CLIENT_SERVER.md`;
   - `Documentation/BUILD.md`;
   - `build_split_mods.bat`;
   - `Silver_77_Quests_Client/config.cpp`;
   - `Silver_77_Quests_Server/config.cpp`.
3. Если задача про русский текст, сначала проверить реальное содержимое файла через `Get-Content -Encoding UTF8`, а не доверять обычному выводу PowerShell.
   Подробная памятка: `Documentation/RUSSIAN_ENCODING.md`.
4. Не трогать монолитный корневой мод без явной причины: он сейчас резерв.
5. Не публиковать серверный split-мод в Workshop.

## Актуализация 25.04.2026

### Что считать актуальной базой

Новый минимальный стартовый набор теперь состоит из 4 квестов:

- `quest_hunter_1` — `Картошечка с маслицем`;
- `quest_fisherman_1` — `Рыба это вам не картошка!`;
- `quest_Rasputin_1` — `Взаимовыручка прежде всего!`;
- `quest_fisherman_2` — `Поставка медицины`.

И 3 триггеров:

- `hunter_trigger` -> `quest_hunter_1`;
- `fisherman_trigger` -> `quest_fisherman_2`, `quest_fisherman_1`;
- `Rasputin_1_trigger` -> `quest_Rasputin_1`.

### Где это уже зафиксировано

Новый минимум уже внесен в:

- `Silver_77_Quests_Server/scripts/4_World/QuestServerManager.c`;
- `SplitMods/Silver_77_Quests_Server/scripts/4_World/QuestServerManager.c`;
- `scripts/4_World/QuestManager.c` как legacy-источник;
- `Documentation/STARTER_QUEST_CONFIG.json`;
- `Support/JSON_Quvest/Silver_77_Quests.json`.

### Что это значит для сервера

- Реальный runtime-конфиг по-прежнему лежит в `profiles/Silver_77_Quests/Silver_77_Quests.json`.
- Новый дефолт подхватится только если профильный JSON отсутствует.
- Текущий согласованный сценарий пользователя: удалить профильный JSON и дать серверу пересоздать его из нового дефолта.

### Что пересобирать

Для текущей split-схемы с отдельными `Silver_77_Quests_Client` и `Silver_77_Quests_Server` под этот апдейт нужен только server rebuild/update:

- клиентский мод не менялся;
- менялся только серверный источник дефолтного JSON.

### Какой редактор считать основной мастерской

Основная точка для парсинга и ручной правки JSON:

```text
M:\GITS_VERSE\Neyro_01\Sborka_Json\JSON_Quvest
```

Именно там:

- лежит актуальный дефолтный `Silver_77_Quests.json`;
- хранятся последние правки редактора;
- ведется дальнейшая работа по авторскому JSON-пайплайну.

### Что уже подготовлено под механику стеков

В редакторе уже добавлена отдельная инфраструктура под ручной справочник стеков:

- боковой блок `Стаки`;
- отдельный файл `item-stack-rules.json`;
- отдельные локальные маршруты `/api/stack-rules`;
- раздельное хранение `className` и `stackSize` вне основного quest JSON.

Важно:

- это пока только основа под ручное заполнение справочника;
- автоматический пересчет objective из "наборов" в итоговое `quantity` еще не подключен.

### Что открыть следующему Codex в первую очередь

Если задача касается актуальной конфигурации проекта, сначала открыть:

1. `Documentation/CODEX_START_HERE.md`
2. `Documentation/CODEX_CONTEXT_2026-04-20.md`
3. `Documentation/CODEX_WORKLOG.md`
4. `Documentation/STARTER_QUEST_CONFIG.json`
5. `M:\GITS_VERSE\Neyro_01\Sborka_Json\JSON_Quvest\PROJECT_CONTEXT.md`

Если задача касается именно редактора / парсера, дополнительно открыть:

1. `M:\GITS_VERSE\Neyro_01\Sborka_Json\JSON_Quvest\README.md`
2. `M:\GITS_VERSE\Neyro_01\Sborka_Json\JSON_Quvest\PROJECT_CONTEXT.md`
3. `M:\GITS_VERSE\Neyro_01\Sborka_Json\JSON_Quvest\Silver_77_Quests.json`

Latest runtime note:

- quest config / player progress sync is now JSON-over-RPC instead of raw object-over-RPC because the client crashed on `String CORRUPTED - FIX OnStoreLoad() !!!` while reading the new nested quest config payload.
- if a future session sees quest sync crashes again, inspect:
  - `Silver_77_Quests_Client/scripts/4_World/QuestClientRPC.c`
  - `Silver_77_Quests_Server/scripts/4_World/QuestServerManager.c`
  - actual deployed client/server PBO timestamps, not just the build script final output.

Additional current state:

- raw object-over-RPC is no longer used for synced quest config;
- single large string-over-RPC also proved unreliable for quest config;
- current active transport for quest config is chunked config payload reassembly on the client;
- starter template JSON files are now expected to be `version 3` with explicit role fields.
- quest editor quest-flow UX is now trigger-first:
  - each NPC / trigger has its own card;
  - offer / completion / reward toggles live inside that card;
  - detail blocks open directly below the same card instead of in a separate global action section.
