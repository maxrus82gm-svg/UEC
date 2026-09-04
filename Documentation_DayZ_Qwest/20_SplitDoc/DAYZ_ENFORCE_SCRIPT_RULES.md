# DAYZ ENFORCE SCRIPT RULES

Постоянные правила написания и проверки DayZ Enforce Script в проекте `Silver_77_Quests`.

## 1. Ограничение сложности formula

Enforce Script ограничивает сложность одного expression / formula. Код, который выглядит допустимым в других языках, может завершиться compile error:

`Formula too complex`

Не создавать одно длинное выражение с большим количеством:

- string concatenation через `+`;
- member access;
- вызовов `.ToString()`;
- других вызовов функций;
- арифметических операций;
- длинных комбинаций `&&` и `||`;
- вложенных вычислений и сложных аргументов функций.

Правило относится ко всему DayZ-коду проекта в файлах `.c`, а не только к одному моду.

## 2. Пошаговое построение строк

Не передавать длинную string formula непосредственно в logging helper.

Нежелательно:

```csharp
LogInfo(prefix + " scenario=" + scenarioId + " group=" + runtimeGroupId + " value=" + value.ToString());
```

Правильно:

```csharp
string line = prefix;
line = line + " scenario=";
line = line + scenarioId;
line = line + " group=";
line = line + runtimeGroupId;
line = line + " value=";
line = line + value.ToString();
LogInfo(line);
```

Особенно внимательно проверять:

- `LogInfo()`;
- `LogError()`;
- `LogStuckDebug()`;
- другие функции, которым передаётся составная строка.

Один assignment должен содержать простое и очевидное действие. Не собирать новую длинную цепочку внутри последнего вызова.

## 3. Разделение условий и вычислений

Большое условие делить на промежуточные значения:

```csharp
bool conditionA = target != null;
bool conditionB = mindState != DayZInfectedConstants.MINDSTATE_CALM;

if (conditionA || conditionB)
{
    // Действие.
}
```

Сложное значение сначала вычислять в локальную переменную, затем передавать в функцию или добавлять в строку:

```csharp
float elapsedSeconds = elapsedMs / 1000.0;
string elapsedText = elapsedSeconds.ToString();
line = line + elapsedText;
```

Это же правило применять к:

- длинным аргументам функций;
- большим `if`;
- сложным `return` expressions;
- арифметике вместе с member access и вызовами функций;
- вложенным вычислениям.

## 4. Диагностика ошибки

Компилятор Enforce Script может показать `Formula too complex` на строке, следующей за реально проблемным expression.

Если указанная строка сама не содержит сложной formula:

1. проверить несколько предыдущих выражений;
2. найти длинную string concatenation, условие, return или аргумент функции;
3. разбить весь подозрительный участок пошагово;
4. просмотреть остальные аналогичные выражения в изменённом файле, а не ждать следующую ошибку компилятора по одному месту.

## 5. Обязательная проверка перед завершением

Для любой задачи, которая создаёт или меняет DayZ `.c`, агент обязан отдельно проверить новые и изменённые участки на Enforce formula complexity.

Минимальная проверка:

- нет длинных цепочек `+` внутри `LogInfo`, `LogError`, `LogStuckDebug` и других вызовов;
- сложные значения вынесены в локальные переменные;
- большие условия разделены на промежуточные `bool`, если expression становится перегруженным;
- сложные return expressions упрощены;
- после декомпозиции сохранены порядок вычислений, marker names и runtime-семантика;
- проверены фигурные, круглые и квадратные скобки;
- выполнен `git diff --check`.

Статическая проверка не заменяет сборку PBO и runtime compile, если они входят в отдельный этап проверки пользователя.
