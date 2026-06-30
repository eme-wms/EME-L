> Синонимы для поиска: CIQuery Query класс EME-L, EME запрос, Object("Query"), CEMEQuery, AddTEXT AddINT AddFLOAT AddDATE AddTIME AddDATETIME AddMONEY AddINT64 AddREF AddVVR AddField AddConnectedField, Execute Create Delete, GetNoOfLines GetNoOfFields FindField TryFindField GetFieldNarrative GetFieldAttribute GetFieldLength GetFieldReference GetFieldGroup, GetData PutData PutTextData Get Put, Sort Sort2 Group Group2 FixOrder GetCurrentOrder SetCurrentOrder FastFind FastFind2, ConnectFields ConnectAllFields PutConnectData PutAllConnectData DisconnectFields, CreateXML CreateCSV Dump RunViewer, CompareQueries CreatePatch ApplyPatch, GetDBObject

# Класс Query

Класс `Query` в языке EME-L — обёртка над внутренним объектом `CEMEQuery`. В системе EME.WMS и языке EME-L он предоставляет скриптам возможность формировать, выполнять, модифицировать и анализировать двумерные наборы данных, состоящие из полей (колонок) и строк. Это основной инструмент для получения выборок из базы данных, временного хранения промежуточных результатов, подготовки данных для отчётов и обмена информацией между подсистемами.

В языке EME-L класс `Query` поддерживает создание пустого запроса программно или выполнение именованного запроса проекта с параметрами, классом, индикатором прогресса, кэшированием, блочным чтением и оптимизацией EME-L. Он позволяет программно строить структуру запроса, заполнять и изменять данные, копировать, объединять, сравнивать запросы и строить патчи, сортировать и группировать, получать метаданные полей и навигацию по строкам, подключать поля к БД для блочной записи, выгружать в XML/CSV/дамп и просматривать результаты в браузере запросов.

## Создание объекта класса Query

В языке EME-L класс `Query` имеет четыре конструктора — по числу аргументов. Все аргументы опциональны: пустой вызов создаёт локальный запрос EME-L, не кэшируемый и не привязанный к именованному запросу проекта.

```EME-L
'Без аргументов — пустой запрос EME-L, не кэшируется'
q = Object("Query");
```

```EME-L
'По имени запроса проекта — контейнер и контекст наследуются из параметров вызова'
q = Object("Query", "MyProjectQuery");
```

```EME-L
'С явным контейнером (родительским объектом)'
q = Object("Query", "MyProjectQuery", container);
```

```EME-L
'С контейнером и контекстом вызова'
q = Object("Query", "MyProjectQuery", container, context);
```

Если имя запроса не задано (`Object("Query")`), запрос должен быть построен программно через `Create` и `Add*`-методы, после чего он не помещается в кэш автоматически. В языке EME-L контейнер, если он невалиден, обнуляется.

## Выполнение и жизненный цикл

В языке EME-L методы класса `Query` управляют выполнением и жизненным циклом запроса.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `Execute()` | — | Empty | Выполняет именованный запрос проекта с параметрами по умолчанию. |
| `Execute(singleThread)` | arg0: Boolean | Empty | Запрос выполняется в одном потоке. |
| `Execute(params, className)` | arg0: String (параметры), arg1: String (класс) | Empty | Явные параметры и класс, например `"Входной фильтр"`. |
| `Execute(params, className, withoutIndicator)` | + arg2: Boolean | Empty | Отключает индикатор прогресса. |
| `Execute(params, className, withoutIndicator, cache)` | + arg3: Boolean | Empty | Управляет кэшированием (FALSE отключает). |
| `Execute(params, className, withoutIndicator, cache, blockRead)` | + arg4: Boolean | Empty | Включает блочное чтение данных БД. |
| `Execute(params, className, withoutIndicator, cache, blockRead, optimizeEMEL)` | + arg5: Boolean | Empty | Включает оптимизатор EME-L. |
| `Create()` | — | Empty | Создаёт структуру запроса без выполнения, с кэшированием по умолчанию. |
| `Create(enableCache)` | arg0: Boolean | Empty | То же; FALSE отключает кэширование при создании. |
| `Delete()` | — | Empty | Удаляет данные запроса. Запрос в кэше не затрагивается. |
| `Delete(clearCache)` | arg0: Boolean | Empty | При TRUE дополнительно удаляет запрос из кэша. |
| `AddToCache()` | — | Empty | Помещает запрос в кэш под своим именем. Не применим к безымянным. |
| `AddToCache(cacheName)` | arg0: String | Empty | Помещает запрос в кэш под указанным именем. |
| `SetWithoutIndicator()` | — | Empty | Эквивалент `Execute(..., TRUE)` для индикатора; действует до выполнения. |
| `SetMemorySize(bytes)` | arg0: Integer | Empty | Устанавливает размер буфера памяти запроса в байтах. |
| `Dump(fileName)` | arg0: String | Empty | Выводит данные запроса в дамп-файл. |
| `RunViewer()` | — | Empty | Открывает браузер запросов. |
| `RunViewer(templateName)` | arg0: String | Empty | Открывает браузер запросов по шаблону. |
| `IsEmpty()` | — | Boolean | TRUE — запрос не создан или не содержит строк. |
| `Backup(targetQuery)` | arg0: Query (по ссылке) | Empty | Выводит данные текущего запроса в целевой запрос. |

В языке EME-L метод `Execute` без параметров и метод `Create` различаются: `Create` строит структуру без обращения к БД, `Execute` обращается к БД и заполняет строки. Установка размера памяти через `SetMemorySize` учитывается как при `Create`, так и при `Execute`.

## Структура и поля

В языке EME-L методы класса `Query` предоставляют метаданные структуры запроса.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetNoOfLines()` | — | Integer | Количество строк. |
| `SetNoOfLines(count)` | arg0: Integer | Empty | Устанавливает количество строк; усекает запрос. |
| `GetNoOfFields()` | — | Integer или Empty | Количество полей в структуре. Empty, если запрос не создан. |
| `FindField(field)` | arg0: String/Integer | Integer | Номер поля по имени или номеру. -1, если не найдено. |
| `TryFindField(field)` | arg0: String/Integer | Integer | Номер поля. `NULL_FIELD` (-100), если не найдено. |
| `GetFieldNarrative(field)` | arg0: Integer | String или Empty | Имя поля по его номеру. |
| `GetFieldAttribute(field)` | arg0: String/Integer | Integer или Empty | Код атрибута (типа) поля. |
| `GetFieldLength(field)` | arg0: String/Integer | Integer | Длина поля в байтах. 0, если поле не найдено. |
| `GetFieldLengthInChars(field)` | arg0: String/Integer | Integer | Длина текстового поля в символах; для остальных — в байтах. 0, если поле не найдено. |
| `GetFieldReference(field)` | arg0: String/Integer | Integer | Номер ссылочной записи. `NULL_RECORD`, если поле не ссылочное. |
| `GetFieldGroup(field)` | arg0: String/Integer | Integer | Индекс поля в текущей группировке. -1, если поле не группировочное. |
| `GetFieldSum(field)` | arg0: String/Integer | Integer/Real/Money | Сумма значений поля по всем строкам. -1, если тип поля не поддерживает суммирование. |
| `GetFields()` | — | Byte array | Сериализованная структура полей. |
| `AddFields(fields)` | arg0: Byte array | Empty | Загружает структуру полей из сериализованного массива. |
| `GetName()` | — | String или Empty | Имя запроса. |
| `SetName(name)` | arg0: String | Empty | Устанавливает имя запроса. |
| `GetContainer()` | — | Object | Контейнер (родительский объект), связанный с запросом. |
| `SetContainer(container)` | arg0: Object (по ссылке) | Empty | Устанавливает контейнер. |
| `SetContext(context)` | arg0: Context (по ссылке) | Empty | Устанавливает новый контекст вызова. |
| `PutDialogVariable(varName)` | arg0: String | Boolean | Сохраняет указатель на запрос в переменную диалога. FALSE, если диалог недоступен. |
| `PutReportVariable(varName)` | arg0: String | Empty | Сохраняет указатель на запрос в переменную отчёта. |

Константы, используемые в языке EME-L при работе с классом `Query`:

- `NULL_FIELD` = **-100** — возвращается `TryFindField`, если поле не найдено.
- `NULL_RECORD` — возвращается `GetFieldReference`, если поле не ссылочное; используется как заглушка записи в `AddREF`/`AddVVR`.
- `NULL_REF` — возвращается `AddLine`, если запрос не создан.

## Добавление полей

В языке EME-L методы `Add*` класса `Query` предназначены для запросов EME-L (`Object("Query")`). При работе с именованным запросом проекта они возвращают `NULL_FIELD` и оставляют структуру нетронутой. Все методы возвращают номер добавленного поля (`Integer`) или Empty, если запрос не создан.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `AddTEXT(fieldName)` | arg0: String | Integer или Empty | Текстовое поле переменной длины. |
| `AddTEXT(fieldName, size)` | + arg1: Integer | Integer или Empty | Текстовое поле заданной длины. Размер 0 = переменная длина. Отрицательный размер вызывает ошибку. |
| `AddINT(fieldName)` | arg0: String | Integer или Empty | Целочисленное поле. |
| `AddINT(fieldName, statFunction)` | + arg1: String | Integer или Empty | Целочисленное поле со статистической функцией. |
| `AddFLOAT(fieldName)` | arg0: String | Integer или Empty | Вещественное поле. |
| `AddFLOAT(fieldName, statFunction)` | + arg1: String | Integer или Empty | Вещественное поле со статистической функцией. |
| `AddDATE(fieldName)` | arg0: String | Integer или Empty | Поле типа дата. |
| `AddTIME(fieldName)` | arg0: String | Integer или Empty | Поле типа время. |
| `AddDATETIME(fieldName)` | arg0: String | Integer или Empty | Поле типа дата-время. |
| `AddMONEY(fieldName)` | arg0: String | Integer или Empty | Поле типа деньги. |
| `AddMONEY(fieldName, statFunction)` | + arg1: String | Integer или Empty | Поле деньги со статистической функцией. |
| `AddINT64(fieldName)` | arg0: String | Integer или Empty | Целое 64 бита. |
| `AddINT64(fieldName, statFunction)` | + arg1: String | Integer или Empty | Целое 64 бита со статистической функцией. |
| `AddREF(fieldName, record)` | arg0: String, arg1: String/Integer | Integer или Empty | Ссылочное поле, привязанное к записи. |
| `AddVVR(fieldName)` | arg0: String | Integer или Empty | Ссылочное поле типа VVR без записи по умолчанию. |
| `AddVVR(fieldName, record)` | arg0: String, arg1: String/Integer | Integer или Empty | Ссылочное поле типа VVR с записью. |
| `AddField(dbField)` | arg0: Field БД (по ссылке) | Integer или Empty | Поле, эквивалентное указанному полю БД. |
| `AddField(dbField, statFunction)` | + arg1: String | Integer или Empty | То же со статистической функцией. |
| `AddConnectedField(dbField)` | arg0: Field БД (по ссылке) | Integer или Empty | Поле по образцу поля БД с немедленным подключением для блочной записи. |

`statFunction` — имя статистической функции, получаемое внутри системы EME. По умолчанию используется значение, соответствующее отсутствию функции.

При добавлении поля в запрос, уже содержащий строки, данные переносятся в новый запрос с пустыми значениями в новом поле. Добавление «на-лету» работает только для EME-L запросов — для именованных вызывается ошибка.

## Данные и строки

В языке EME-L методы класса `Query` предоставляют доступ к данным запроса по полям и строкам.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetData()` | — | CEMEQuery (по ссылке) | Внутренний объект данных запроса. |
| `GetData(field, line)` | arg0: String/Integer, arg1: Integer | Variant | Значение поля в строке. |
| `PutData(field, line, value)` | arg0: String/Integer, arg1: Integer, arg2: Variant | Empty | Записывает значение в поле и строку. |
| `PutTextData(field, line, value)` | arg0: String/Integer, arg1: Integer, arg2: Variant | Empty | Преобразует значение в строку и записывает в текстовое поле. Поле обязано быть текстовым. |
| `Get(field)` | arg0: String/Integer | Variant | Значение поля текущей строки. |
| `Put(field, value)` | arg0: String/Integer, arg1: Variant | Empty | Записывает значение в текущую строку. |
| `GetDataAsSqlValue(field, line, syntax)` | arg0: String/Integer, arg1: Integer, arg2: String | String | Значение поля в SQL-формате. `syntax`: `"ODBC_SYNTAX"` или `"ANSI_SQL_92_SYNTAX"`. |
| `AddLine()` | — | Integer или Empty | Добавляет пустую строку; возвращает её номер. `NULL_REF`, если запрос не создан. |
| `CopyLine(sourceLine, targetLine)` | arg0: Integer, arg1: Integer | Empty | Копирует данные между строками текущего запроса. |
| `CopyLine(sourceQuery, sourceLine, targetLine)` | arg0: Query (по ссылке), arg1: Integer, arg2: Integer | Empty | Копирует строку из другого запроса по совпадению имён и типов полей. |
| `Copy(sourceQuery, startLine, endLine, createFields)` | arg0: Query, arg1..arg3 опциональны | Empty | Копирует диапазон строк из исходного запроса. По умолчанию: `startLine=0`, `endLine=-1` (до конца), `createFields=TRUE`. |
| `Merge(sourceQuery)` | arg0: Query (по ссылке) | Empty | Объединяет данные исходного запроса с текущим. Структура копируется при пустом приёмнике. |
| `DeleteAllLines()` | — | Empty | Удаляет все строки, сохраняя структуру полей. |
| `GetLineData(line)` | arg0: Integer | Byte array | Сериализованные данные строки. |
| `PutLineData(line, data)` | arg0: Integer, arg1: Byte array | Empty | Загружает данные строки из сериализованного массива. |
| `CompareQueries(oldQuery, idField, resultQuery)` | 3 аргумента Query/Field/Query | Empty | Сравнивает текущий запрос со старым по полю-идентификатору; результат — в третий запрос. |
| `CreatePatch(oldQuery, idField, patchQuery)` | 3 аргумента | Empty | Формирует запрос-патч изменений. |
| `ApplyPatch(oldQuery, idField, patchQuery)` | 3 аргумента | Empty | Применяет патч к старому запросу, восстанавливая новые данные в текущем. |

## Навигация по строкам

В языке EME-L класс `Query` поддерживает навигацию по строкам результата.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetLine()` | — | Integer | Номер текущей строки. |
| `SetLine(line)` | arg0: Integer | Empty | Устанавливает текущую строку. |
| `SetFirstLine()` | — | Empty | Переходит к первой строке. |
| `SetNextLine()` | — | Empty | Переходит к следующей строке. |
| `IsValidLine()` | — | Boolean | Допустима ли текущая строка. |
| `IsValidLine(line)` | arg0: Integer | Boolean | Существует ли строка с указанным номером. |
| `NextBand(groupFields)` | arg0+: String (список) | Boolean | Переход к следующей строке группы по полям группировки. |
| `GetDBObject(field)` | arg0: String/Integer | CEMERec | Объект записи БД для ссылочного поля текущей строки. |
| `GetDBObject(field, line)` | arg0: String/Integer, arg1: Integer | CEMERec | То же для указанной строки. |

`Get` и `Put` в языке EME-L работают с текущей строкой. До первого вызова `SetLine`/`SetFirstLine`/`AddLine` номер строки равен -1 — вызов `Get`/`Put` в этом состоянии вызывает ошибку.

`NextBand` в языке EME-L требует, чтобы переданные поля были полями группировки, причём в правильном порядке. Первый вызов для поля возвращает первую строку группы, последующие переходят к следующей. Возврат FALSE означает конец группы.

## Сортировка и группировка

В языке EME-L класс `Query` предоставляет методы сортировки, группировки и быстрого поиска.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `Sort(fields)` | arg0+: String (список) или Array | Empty | Сортировка по полям. `DESC:` перед именем — по убыванию. |
| `Sort2(fields)` | arg0: String (поля через `;`) | Empty | То же, поля в одной строке через `;`. |
| `Group(fields)` | arg0+: String (список) или Array | Empty | Группировка по полям. `DESC:` поддерживается. |
| `Group2(fields)` | arg0: String (поля через `;`) | Empty | То же, поля в одной строке через `;`. |
| `FixOrder()` | — | Integer | Фиксирует текущую сортировку и возвращает её индекс. -1, если запрос не создан. |
| `GetCurrentOrder()` | — | Integer | Индекс текущей сортировки. -1, если не создан. |
| `SetCurrentOrder(orderIndex)` | arg0: Integer | Empty | Устанавливает сортировку по индексу. |
| `TrySetCurrentOrder(orderIndex)` | arg0: Integer | Boolean | То же; FALSE при неудаче. |
| `GetCurrentOrderField(index)` | arg0: Integer | Integer | Номер поля по индексу в текущей сортировке. -1 при ошибке. |
| `GetCurrentOrderFieldType(index)` | arg0: Integer | Integer | 0 — возрастание, 1 — убывание, -1 — ошибка. |
| `GetCurrentSort2()` | — | String | Текущая сортировка в формате `Sort2` (поля через `;`, с префиксом `DESC:`). |
| `GetNoOfOrderFields()` | — | Integer | Количество полей в текущей сортировке. -1, если не создан. |
| `FastFind(orderIndex, values)` | arg0: Integer (-1/Empty = текущая), arg1+ значения | Integer | Номер найденной строки. -1, если не найдена. |
| `FastFind2(result, orderIndex, values)` | arg0: ByRef Integer, arg1: Integer, arg2+ значения | Integer | То же; `result`: 0 — совпадение, 1 — строка меньше или равна, -1 — больше или равна. |
| `MarkUp(mode)` | arg0: Integer | Empty | Разметка запроса в заданном режиме. |
| `SetTransformQuery(crossQuery)` | arg0: Query (по ссылке) | Empty | Устанавливает перекрёстный запрос. |

`Sort` и `Group` в языке EME-L принимают либо список аргументов-строк, либо один аргумент типа `Array`. В обоих случаях `DESC:` (с двоеточием, без пробела) перед именем поля инвертирует порядок. `Sort` сбрасывает временную сортировку до и после выполнения.

## Подключение к БД и экспорт

В языке EME-L класс `Query` позволяет подключать поля запроса к полям записи БД для блочной записи и экспортировать данные.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `ConnectFields(dbField)` | arg0: Field БД (по ссылке) | Empty | Подключает источник по имени поля БД. |
| `ConnectFields(dbField, sourceField)` | + arg1: String/Integer | Empty | Подключает указанный источник к полю БД. |
| `ConnectAllFields(dbRecord)` | arg0: Record БД (по ссылке) | Empty | Подключает все поля-источники к записи-приёмнику. |
| `PutConnectData(line)` | arg0: Integer | Empty | Копирует подключённые данные для строки. |
| `PutAllConnectData()` | — | Empty | Копирует подключённые данные по всем строкам. |
| `DisconnectFields()` | — | Empty | Отключает источники и сбрасывает остаток данных. |
| `CreateXML(target, parentElement)` | arg0: String или HTMLDocument, arg1 опционально | Integer | XML-представление запроса. Код результата, 0 при ошибке. |
| `CreateCSV(fileName)` | arg0: String | Integer | Сохранение в CSV. |
| `CreateCSV(fileName, useLocale)` | + arg1: Boolean (по умолч. FALSE) | Integer | То же с локальными настройками. |
| `CreateCSV(fileName, useLocale, wrapText)` | + arg2: Boolean (по умолч. FALSE) | Integer | То же; `wrapText` заключает текстовые поля в кавычки. |
| `Load(bitBuffer, recordName)` | arg0: BitBuffer (по ссылке), arg1: String | Empty | Загружает данные из буфера, добавляя служебное поле со ссылками. |

## Примеры

Программное создание запроса и заполнение строк в языке EME-L:

```EME-L
'Построение временного запроса в памяти'
q = Object("Query");
q.Create();
q.AddTEXT("Code", 32);
q.AddINT("Quantity");
q.AddMONEY("Amount");

line = q.AddLine();
q.Put("Code", "A-001");
q.Put("Quantity", 5);
q.Put("Amount", 1250.50);

Loop (q)
    Code = q.Get("Code");
    Qty = q.Get("Quantity");
End Loop
```

Выполнение именованного запроса проекта и чтение результатов в языке EME-L:

```EME-L
'Запрос проекта ABCAnalysis.Result'
q = Object("Query", "ABCAnalysis.Result");
Object("System").InitQueryCache();
q.Execute();
Object("System").ClearQueryCache();

If (q.GetNoOfLines() == 0)
    Return;
End If

q.SetFirstLine();
While (q.IsValidLine())
    Code = q.GetData("Code", q.GetLine());
    q.SetNextLine();
End While
```

## См. также

- [Системные функции](../system-functions/) — функции `is_query*` для работы с запросами без класса в языке EME-L.
- [Класс BitBuffer](./BitBuffer.md) — использование `Load` для загрузки отмеченных строк в языке EME-L.
- [Класс File](./File.md) — файловый ввод-вывод в языке EME-L.
- [Класс Array](./Array.md) — передача массива имён полей в `Sort`/`Group` в языке EME-L.
