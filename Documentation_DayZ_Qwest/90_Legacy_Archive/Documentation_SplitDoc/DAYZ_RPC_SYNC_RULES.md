# DAYZ RPC SYNC RULES

Правила для задач, связанных с DayZ RPC, client/server sync и UI, зависящим от серверных данных.

## 1. Когда читать

Читать для задач, связанных с:

- `PlayerBase.OnRPC`
- `ParamsReadContext`
- `RPCSingleParam`
- `Param1 / Param2 / Param3 / Param4`
- `config sync`
- `player data sync`
- `quest progress sync`
- `JSON payload`
- UI, который зависит от `server data`

## 2. Главное правило

Не отправлять длинный `JSON`, `progress`, `config`, `dialog` или другой большой `string payload` одной строкой через RPC.

Для DayZ/Enforce это ненадёжно.

Если в логах появляется:

```text
Reason: !!! String CORRUPTED - FIX OnStoreLoad() !!!
```

то сначала проверять транспорт RPC и формат передачи, а не UI.

## 3. Chunked sync

Любые крупные или потенциально сложные данные передавать чанками.

Рекомендуемый формат:

```text
Param3<int, int, string>
```

Где:

- `param1` = `chunkIndex`
- `param2` = `totalChunks`
- `param3` = `chunkPayload`

Рекомендуемый размер чанка:

- `512` байт;
- либо уже существующая константа проекта.

## 4. Отдельные buffer по типам payload

Для каждого типа данных нужен отдельный buffer.

Правильно:

- `config chunks` отдельно;
- `player data chunks` отдельно;
- `dialog data chunks` отдельно;
- будущие snapshot/inventory payload отдельно.

Неправильно:

- складывать разные payload в один общий buffer;
- переиспользовать один `expectedChunkCount` для разных RPC;
- собирать разные payload-типы в одной общей функции без явного разделения.

## 5. Правило ParamsReadContext

`ctx.Read(...)` выполнять прямо внутри соответствующего `case` в `OnRPC`.

Не передавать `ParamsReadContext` в helper-функции.

Helper-функции должны получать уже прочитанные значения:

- `string payload`
- `int chunkIndex`
- `int totalChunks`
- `PlayerQuestData`
- `Silver77_QuestConfig`

## 6. Диагностика по цепочке

При проблемах client/server sync проверять строго по этапам:

1. Клиент отправил request?
2. Сервер получил request?
3. Сервер загрузил реальные данные?
4. Сервер подготовил payload?
5. Сервер отправил response?
6. Клиент получил chunks?
7. Клиент собрал payload?
8. Клиент десериализовал payload?
9. Клиент применил данные?
10. UI обновился после apply?

Не перепрыгивать сразу к UI, если не подтверждён транспорт и чтение данных.

## 7. Если клиент падает в ctx.Read

Проверить:

1. совпадает ли `RPC ID` на сервере и клиенте;
2. совпадает ли `Param`-тип на сервере и клиенте;
3. не читает ли `super.OnRPC` или другой обработчик контекст раньше нашего `case`;
4. не передаётся ли `ParamsReadContext` в helper;
5. не отправляется ли длинная строка одной строкой;
6. нет ли признака `String CORRUPTED`.

Если есть `String CORRUPTED`, сначала переводить payload на `chunked sync`.

## 8. Server is source of truth

Сервер:

- хранит progress;
- загружает progress из profile JSON;
- меняет quest status;
- выдаёт rewards;
- сохраняет player data;
- отправляет sync клиенту.

Клиент:

- запрашивает config и player data;
- получает sync;
- отображает UI;
- не должен считать default `available` реальным progress;
- не должен принимать server-side решения самостоятельно.

## 9. UI не должен маскировать проблемы sync

Если `HasSyncedPlayerData == false`:

- UI может показывать `loading`;
- UI не должен показывать окончательные статусы;
- UI не должен активировать кнопки на основе default `available`;
- UI не должен создавать впечатление, что квест реально доступен, если sync ещё не получен.

Функции вроде `EnsurePlayerProgress` или `GetQuestStatus` не должны подменять отсутствие sync настоящим прогрессом.

## 10. Минимальные задачи

При RPC/client-server проблемах не менять сразу одновременно:

- UI;
- JSON;
- server progress;
- client manager;
- RPC format;
- layout.

Идти маленькими шагами:

1. подтвердить request;
2. подтвердить server data;
3. подтвердить server send;
4. подтвердить client receive;
5. подтвердить `ctx.Read`;
6. подтвердить deserialize;
7. подтвердить apply;
8. подтвердить UI update.

## 11. Подтверждённый опыт

Опыт `TASK 064`:

- проблема была в передаче `player data JSON` одной строкой через RPC;
- клиент падал с `String CORRUPTED`;
- переход на `chunked sync` исправил транспорт;
- после этого UI начал получать реальные статусы квестов.

Правило на будущее:

Если через RPC передаётся `JSON/string payload`, сразу проектировать `chunked sync`.
