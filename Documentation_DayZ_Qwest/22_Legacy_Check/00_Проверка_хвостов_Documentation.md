# Проверка хвостов старой Documentation

## Цель

Проверить, остались ли активные ссылки на старую папку `Documentation`, и отделить рабочие хвосты от legacy bridge, истории и архивного контекста.

## Проверенные паттерны

- `P:\Silver_77_Quests\Documentation`
- `P:\\Silver_77_Quests\\Documentation`
- `Documentation/AGENT_TASK_LOOP.md`
- `Documentation/SplitDoc/START.md`
- `Documentation/SplitDoc`
- `Documentation/QUEST_LOGIC_SPEC.md`
- `Documentation/README_JSON_CONFIG.md`
- `Documentation/RUSSIAN_ENCODING.md`
- `Documentation/SPLIT_CLIENT_SERVER.md`
- `Documentation/BUILD.md`
- `Documentation/README_INSTALLATION.md`
- `Documentation/README.md`
- `Documentation/CHANGELOG.md`
- `Documentation/`
- `Documentation\`

## Найденные активные хвосты

- `Documentation_DayZ_Qwest/20_SplitDoc/00_INDEX.md` — старый `Documentation/SplitDoc` больше не указан как источник правды; документ переименован по смыслу в активные правила.
- `Documentation_DayZ_Qwest/21_Project_Docs/00_INDEX.md` — старый `Documentation` больше не указан как источник правды; документ переименован по смыслу в активные проектные документы.
- `Documentation_DayZ_Qwest/12_Старт_агента.md` — действующий стартовый документ агента.
- `Documentation_DayZ_Qwest/01_Текущее_состояние.md` — активные правила, START и loop переведены на Obsidian-пути.
- `Documentation_DayZ_Qwest/04_Архитектура.md` — раздел документации и правил переведён на Obsidian-пути.
- `Documentation_DayZ_Qwest/20_SplitDoc/*.md` — внутренние ссылки на правила и проектные документы переведены на `Documentation_DayZ_Qwest/20_SplitDoc` и `Documentation_DayZ_Qwest/21_Project_Docs`.
- `Documentation_DayZ_Qwest/21_Project_Docs/README.md` — первая точка входа для агента заменена на Obsidian-loop и Obsidian-START.
- `Documentation_DayZ_Qwest/21_Project_Docs/BUILD.md` — ссылка на чеклист split-сборки заменена на `Documentation_DayZ_Qwest/21_Project_Docs/SPLIT_CLIENT_SERVER.md`.

## Состояние legacy bridge

- `Documentation/AGENT_TASK_LOOP.md` — удалён после окончательного переключения на `Documentation_DayZ_Qwest/11_Задача_агенту.md` и `Documentation_DayZ_Qwest/12_Старт_агента.md`.
- `Documentation/SplitDoc/START.md` — в текущей структуре отсутствует и не является действующим bridge; единственный действующий START находится в `Documentation_DayZ_Qwest/12_Старт_агента.md`.
- Старая корневая папка `Documentation` удалена и больше не используется.
- `Documentation_DayZ_Qwest/10_Правила_агента.md` фиксирует финальное состояние без существующего bridge.

## Оставленные исторические упоминания

- `Documentation_DayZ_Qwest/20_SplitDoc/TASK_HISTORY.md` — старые пути в TASK 104-110 оставлены как история миграции.
- Архивные копии старых документов в `Documentation_DayZ_Qwest/90_Legacy_Archive` не переписывались: это исторические отчёты и старые рабочие документы.
- `Documentation_DayZ_Qwest/21_Project_Docs/RUSSIAN_ENCODING.md` — ссылки на старые CODEX-документы заменены реальными путями к их копиям в `90_Legacy_Archive`.
- `Documentation_DayZ_Qwest/21_Project_Docs/BUILD.md` и `Documentation_DayZ_Qwest/21_Project_Docs/SPLIT_CLIENT_SERVER.md` — `Documentation/` в деревьях сборки оставлен как описание структуры старого/публикуемого мода, не как активный источник правил агента.

## Непонятные хвосты

- Активных зависимостей от старой корневой папки `Documentation` не осталось.
- Совпадения в `TASK_HISTORY`, разделах TASK 112/124 ниже и `90_Legacy_Archive` являются историческими записями.
- `Documentation/` в деревьях `BUILD.md` и `SPLIT_CLIENT_SERVER.md` относится к структуре готовой сборки мода и не является ссылкой на удалённую рабочую папку.

## Итог

Уникальный `STARTER_QUEST_CONFIG.json` перенесён в `Documentation_DayZ_Qwest/21_Project_Docs`, legacy bridge удалён, старая корневая папка `Documentation` удалена. Активных зависимостей от неё не осталось; исторические записи сохранены.

## TASK 112 — Финальная миграция хвостов

Исправлены README и пользовательские точки входа:

- `README.md`
- `JSON_Quvest/README.md`
- `Support/JSON_Quvest/README.md`
- `Support/Arh_29042026/JSON_Quvest/README.md`

Перенесены архивные документы:

- старые top-level Markdown-документы из `Documentation` скопированы в `Documentation_DayZ_Qwest/90_Legacy_Archive/Documentation`;
- старые Markdown-документы из `Documentation/SplitDoc` скопированы в `Documentation_DayZ_Qwest/90_Legacy_Archive/Documentation_SplitDoc`;
- перед удалением старых дублей выполнена SHA256-сверка архивных копий.

Удалены старые `.md`-дубли после переноса:

- из `Documentation` удалены все top-level `.md`, кроме `AGENT_TASK_LOOP.md` bridge;
- из `Documentation/SplitDoc` удалены все `.md`, кроме `START.md` bridge.

Оставлены допустимые legacy-упоминания:

- `Documentation/AGENT_TASK_LOOP.md` — bridge;
- `Documentation/SplitDoc/START.md` — bridge;
- старые пути внутри `90_Legacy_Archive` — историческое содержимое архивных копий;
- старые пути в `TASK_HISTORY` и отчётах — история миграции.

Итог TASK 112:

- активные README больше не ведут на старую `Documentation`;
- старые Markdown-документы перенесены в Obsidian-архив;
- `Documentation` больше не является рабочим каноном.

## TASK 124 — Ревизия документации и рабочего графа Obsidian

### Рабочий root

- канонический логический путь: `P:\Silver_77_Quests`;
- `P:` является `SUBST` для `D:\Dayz`;
- физический Git root: `D:\Dayz\Silver_77_Quests`;
- оба пути используют `D:\Dayz\Silver_77_Quests\.git` и один HEAD `d72dd0ece725e9962453880b04f96fe362e4e07b`.

### Классификация

ACTIVE:

- 36 сохранённых рабочих `.md/.canvas` в `Documentation_DayZ_Qwest` вне `.trash`, `22_Legacy_Check` и `90_Legacy_Archive`;
- центральная точка: `00_Главная.md`;
- агентская система: `10_Правила_агента.md`, `11_Задача_агенту.md`, `12_Старт_агента.md`;
- тематические правила: `20_SplitDoc`;
- проектные документы: `21_Project_Docs`;
- сюжетные материалы: `13_ЛОР.md` и `Квесты, идеи описания классов.canvas`.

MODULE:

- корневые `README.md` и `возможности_Silver_77_Quests.md`;
- документация `JSON_Quvest`, `Support/JSON_Quvest`, Client, Server и `SplitMods`;
- файлы сохранены на местах и скрыты только из рабочего графа документации.

BRIDGE:

- `Documentation/AGENT_TASK_LOOP.md` ведёт к действующим `11_Задача_агенту.md` и `12_Старт_агента.md`;
- `Documentation/SplitDoc/START.md` на момент ревизии отсутствует и не считается вторым START.

LEGACY / ARCHIVE:

- 30 файлов в `90_Legacy_Archive` сохранены без удаления;
- `Support/Arh_29042026/JSON_Quvest` сохранён как архив самостоятельного инструмента.

EXACT DUPLICATE:

- подтверждены совпадающие архивные копии `README_JSON_CONFIG.md`, `CHANGELOG.md`, `SPLIT_CLIENT_SERVER.md`, `README_INSTALLATION.md`, `QUEST_LOGIC_SPEC.md` и `DAYZ_RPC_SYNC_RULES.md`;
- README Client/Server совпадают с README соответствующих деревьев в `SplitMods`;
- `JSON_Quvest/PROJECT_CONTEXT.md` совпадает с архивной копией в `Support/Arh_29042026`;
- эти копии сохранены как архивные снимки или документация самостоятельных деревьев модулей.

FOREIGN:

- `Support/Mashin/BP_Mashin.md` — уникальная инструкция UE5 Blueprint, не относящаяся к рабочей DayZ-документации; файл сохранён и исключён только фильтром графа.

CONFLICT:

- `JSON_Quvest` и `Support/JSON_Quvest` по-разному называют канонический editor root;
- оба набора содержат уникальную информацию, поэтому автоматическое удаление или объединение не выполнялось.

### Удалено

Как `OBSIDIAN JUNK` удалены только файлы без уникальной информации и входящих ссылок:

- `Documentation_DayZ_Qwest/Добро пожаловать.md` — стандартная приветственная заметка Obsidian;
- `Documentation_DayZ_Qwest/001/Добро пожаловать.md` — вторая стандартная приветственная заметка;
- `Documentation_DayZ_Qwest/Без названия.canvas` — пустой объект `{}`;
- два пустых Markdown-файла и пустой canvas из `Documentation_DayZ_Qwest/.trash`.

Все удалённые файлы были tracked и остаются восстановимыми из Git.

### Переименовано и обновлено

- единственный действующий `12_START_зеркало.md` переименован в `12_Старт_агента.md`;
- заголовок изменён на `# СТАРТ АГЕНТА`;
- активные ссылки, bridge, README, маршрутизация и жёлтая группа графа обновлены;
- `00_Главная.md` связывает разделы, индексы и сюжетные материалы;
- индексы `20_SplitDoc` и `21_Project_Docs` используют полные Obsidian-пути с alias; в индекс правил добавлены ранее пропущенные `DOCUMENTATION_RULES`, `IMAGE_GENERATION_PRESET` и `IMAGE_SCENE_CONTINUITY`.

### Рабочий граф

Фильтр:

`path:"Documentation_DayZ_Qwest/" -path:"Documentation_DayZ_Qwest/90_Legacy_Archive/" -path:"Documentation_DayZ_Qwest/22_Legacy_Check/" -path:"Documentation_DayZ_Qwest/.trash/"`

Граф показывает рабочую базу `Documentation_DayZ_Qwest`, но скрывает архив, legacy-аудит, Obsidian trash и все документы модулей/подпроектов за пределами активной базы. Файлы при этом не удаляются.

## Финальная миграция старой Documentation

- `Documentation/STARTER_QUEST_CONFIG.json` перенесён в `Documentation_DayZ_Qwest/21_Project_Docs/STARTER_QUEST_CONFIG.json` без изменения содержимого.
- SHA-256 до и после переноса совпал: `DFBC14A26D583D09D23205451165466E181BF38D5685B5C5D6331FB8EB779167`.
- JSON остался валидным: версия `3`, четыре квеста и три триггера.
- Legacy bridge `Documentation/AGENT_TASK_LOOP.md` удалён.
- Пустые `Documentation/KnowledgeBase`, `Documentation/SplitDoc` и старая корневая папка `Documentation` удалены.
- Активные ссылки переведены на актуальные или реальные архивные пути; зависимостей от старой корневой папки больше нет.
- Исторические упоминания в `TASK_HISTORY`, `90_Legacy_Archive` и отчётах прошлых задач сохранены.
