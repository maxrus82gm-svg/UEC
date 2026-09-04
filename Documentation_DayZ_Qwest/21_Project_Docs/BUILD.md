# Инструкция по сборке Silver_77_Quests

## Структура проекта

```
@Silver_77_Quests/
├── mod.cpp                    (метаданные мода)
├── README.md                  (краткое описание)
├── addons/                    (PBO файлы после сборки)
│   └── Silver_77_Quests.pbo
├── key/                       (ключи подписи, опционально)
│   └── [твой_ключ].bikey
├── Documentation/             (документация)
│   ├── README_INSTALLATION.md
│   ├── README_JSON_CONFIG.md
│   └── BUILD.md
└── [исходники для сборки]
    ├── $PBOPREFIX$
    ├── config.cpp
    ├── gui/
    └── scripts/
```

## Рекомендуемая сборка: client/server split

Для приватной серверной логики используй раздельную сборку:

```text
Silver_77_Quests/
├── Silver_77_Quests_Client/      -> исходники @Silver_77_Quests_Client
└── Silver_77_Quests_Server/      -> исходники @Silver_77_Quests_Server
```

Запуск из корня проекта:

```bat
build_split_mods.bat
```

Скрипт собирает готовые моды сюда:

```text
..\Mods_DONE\@Silver_77_Quests_Client
..\Mods_DONE\@Silver_77_Quests_Server
```

Запуск сервера:

```bat
-mod=@Silver_77_Quests_Client
-serverMod=@Silver_77_Quests_Server
```

В Steam Workshop публикуй только `@Silver_77_Quests_Client`. Серверный `@Silver_77_Quests_Server` держи только на сервере. Подробности и чеклист проверки: `Documentation_DayZ_Qwest/21_Project_Docs/SPLIT_CLIENT_SERVER.md`.

Если собираешь вручную через Addon Builder:

```text
Клиент:
Source Directory:      D:\Dayz\Silver_77_Quests\Silver_77_Quests_Client
Destination Folder:    D:\Dayz\Mods_DONE\@Silver_77_Quests_Client\addons
Addon prefix:          Silver_77_Quests

Сервер:
Source Directory:      D:\Dayz\Silver_77_Quests\Silver_77_Quests_Server
Destination Folder:    D:\Dayz\Mods_DONE\@Silver_77_Quests_Server\addons
Addon prefix:          Silver_77_Quests_Server
```

Папка `SplitMods/` оставлена только как старая резервная копия split-исходников. Для новой сборки используй две корневые папки выше.

## Сборка через DayZ Tools

Раздел ниже описывает старую монолитную сборку `@Silver_77_Quests` в один PBO. Она оставлена как резервный вариант.

### Рекомендуемая структура (автоматическая сборка):

```
Mods_DONE/
└── @Silver_77_Quests/          ← Готовый мод
    ├── mod.cpp
    ├── addons/                 ← PBO попадёт сюда автоматически
    ├── key/
    └── Documentation/

Silver_77_Quests/               ← Исходники (эта папка)
├── $PBOPREFIX$
├── config.cpp
├── gui/
└── scripts/
```

### 1. Addon Builder (автоматическая сборка)

**Первоначальная настройка:**
1. Создай папку `Mods_DONE/@Silver_77_Quests/`
2. Скопируй туда: `mod.cpp`, папки `addons/`, `key/`, `Documentation/`

**Настройка Addon Builder:**
1. Открой DayZ Tools → Addon Builder
2. Настрой параметры:
   - **Source Directory**: `D:\Dayz\Silver_77_Quests` (или твой текущий путь к исходникам)
   - **Destination Folder**: `D:\Dayz\Mods_DONE\@Silver_77_Quests\addons` ← ВАЖНО!
   - **PBO Name Prefix**: `Silver_77_Quests`
   - **Exclude Files**: `*.md;*.txt;*.bat;.vscode;Documentation;key;addons;*.pbo;*.bisign;*.depbo*;exclude.lst`
3. Нажми **Pack**
4. Готово! PBO автоматически в `Mods_DONE/@Silver_77_Quests/addons/`

**Альтернатива (если не хочешь создавать Mods_DONE):**
- **Destination Folder**: `D:\Dayz\Silver_77_Quests\addons`
- После сборки проверь, что PBO лежит в папке `addons/`

### 2. Через Mikero's Tools (альтернатива)

```bash
PboProject.exe -P "D:\Dayz\Silver_77_Quests"
```

## Подпись мода (опционально)

### Генерация ключа:

```bash
DSSignFile.exe keygen mykey
```

Это создаст:
- `mykey.biprivatekey` (приватный ключ, НЕ публикуй!)
- `mykey.bikey` (публичный ключ, раздавай с модом)

### Подпись PBO:

```bash
DSSignFile.exe sign mykey.biprivatekey addons\Silver_77_Quests.pbo
```

Скопируй `mykey.bikey` в папку `key/`

## После сборки

**Обязательно переместите файлы:**
```bash
# Переместить PBO и bisign в addons/
Move-Item Silver_77_Quests.pbo addons/
Move-Item Silver_77_Quests.pbo.*.bisign addons/
```

Или вручную перетащи файлы из корня в папку `addons/`

## Публикация

### Структура для раздачи:

```
@Silver_77_Quests/
├── mod.cpp
├── README.md
├── addons/
│   ├── Silver_77_Quests.pbo
│   └── Silver_77_Quests.pbo.mykey.bisign  (если подписан)
├── key/
│   └── mykey.bikey  (если подписан)
└── Documentation/
    ├── README_INSTALLATION.md
    └── README_JSON_CONFIG.md
```

**Перед публикацией удали из корня:**
- Исходники (config.cpp, scripts/, gui/, $PBOPREFIX$)
- .vscode/
- Оставь только: mod.cpp, README.md, addons/, key/, Documentation/

### Для Steam Workshop (монолитный вариант):

1. Создай папку `@Silver_77_Quests` со структурой выше
2. Открой DayZ Tools → Publisher
3. Выбери папку `@Silver_77_Quests`
4. Заполни описание, теги
5. Опубликуй

## Тестирование

Для split-сборки:

1. Скопируй `@Silver_77_Quests_Client` и `@Silver_77_Quests_Server` в папку с модами сервера
2. Запусти сервер с параметрами:
   ```bat
   -mod=@Silver_77_Quests_Client
   -serverMod=@Silver_77_Quests_Server
   ```
3. Клиенту нужен только `@Silver_77_Quests_Client`
4. Проверь логи на наличие:
   ```
   [Silver_77_Quests] MissionServer.OnInit called
   [Silver_77_Quests] QuestServerManager initialized
   [Silver_77_Quests] Loaded 2 quests and 2 triggers
   ```
5. Зайди в игру и подойди к NPC/триггеру
6. Нажми F - должно открыться меню

Для старой монолитной сборки используй `mods = @Silver_77_Quests;`.

## Обновление

При изменении кода:
1. Пересобери PBO через Addon Builder
2. Перезапусти сервер
3. Конфиг `profiles/Silver_77_Quests/Silver_77_Quests.json` НЕ перезаписывается

## Troubleshooting

### PBO не собирается:
- Проверь, что клиентский `$PBOPREFIX$` содержит `Silver_77_Quests`, а серверный содержит `Silver_77_Quests_Server`
- Убедись, что `config.cpp` без ошибок
- Проверь пути в Addon Builder

### Мод не загружается:
- Проверь логи сервера
- Убедись, что `mod.cpp` в корне `@Silver_77_Quests`
- Проверь, что PBO в папке `addons/`

### Меню не открывается:
- Проверь, что триггеры создались (логи)
- Убедись, что игрок в радиусе триггера
- Проверь, что `gui/QuestMenu.layout` в PBO
