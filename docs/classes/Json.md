# Класс Json

> Синонимы для поиска: CIJson Json класс EME-L, JSON EME-L, парсинг JSON EME-L, GetType GetTypeAt GetCurrentType IsNullType IsNullTypeAt IsCurrentNullType IsStringType IsStringTypeAt IsCurrentStringType IsIntegerType IsIntegerTypeAt IsCurrentIntegerType IsDoubleType IsDoubleTypeAt IsCurrentDoubleType IsBoolType IsBoolTypeAt IsCurrentBoolType IsSimpleType IsSimpleTypeAt IsCurrentSimpleType IsObjectType IsObjectTypeAt IsCurrentObjectType IsArrayType IsArrayTypeAt IsCurrentArrayType IsStructuredType IsStructuredTypeAt IsCurrentStructuredType, GetValue GetValueAt GetValueAtOrDefault GetCurrentValue GetNodeAt GetCurrentNode IsKeyExists IsEmpty GetSize, PutValue PutValueAt AddNodeAt ResizeArray PushBackValue PushBackNode, SetFirstLine IsValidLine SetNextLine ResetIteration GetCurrentKey GetCurrentIndex, EraseAt EraseCurrent EraseAll, PutArray PutArrayAt GetArray GetArrayAt, PutQuery PutQueryAt GetQuery GetQueryAt, Parse ParseNoExcept DumpToStr DumpToStrFormatted GetContent, SetConst IsConst, JSON_NODE_TYPE_NULL JSON_NODE_TYPE_BOOLEAN JSON_NODE_TYPE_STRING JSON_NODE_TYPE_INTEGER JSON_NODE_TYPE_DOUBLE JSON_NODE_TYPE_ARRAY JSON_NODE_TYPE_OBJECT JSON_NODE_TYPE_UNKNOWN, EmeQuery сериализация Query EME-L, Array CIArray EME-L

Класс `Json` в языке EME-L — класс для работы с JSON-данными в системе EME. В языке EME-L класс Json предоставляет программный доступ к созданию, чтению, модификации и сериализации структур данных в формате JSON. Объект `Json` внутренне представляет собой дерево узлов, каждый из которых может иметь один из следующих типов: null, boolean, string, integer, double, array или object. Корневой узел создаётся при создании объекта и может быть инициализирован пустым значением null или путём парсинга переданной JSON-строки.

Основные возможности класса Json в языке EME-L:

- Парсинг JSON-строк и преобразование узла в строку компактного или отформатированного вида с выбором кодировки результата (ANSI или UTF-8).
- Определение типа текущего, дочернего или текущего в итерации узла, а также проверка принадлежности к группам типов (простой, структурный, null, string, integer, double, boolean, object, array).
- Чтение и запись простых значений узлов, доступ к дочерним узлам по строковому ключу (для object) или целочисленному индексу (для array), получение значений по умолчанию.
- Создание, изменение размера и наполнение массивов, добавление дочерних узлов объектов.
- Обход дочерних элементов object и array с помощью итератора: установка на первый элемент, переход к следующему, получение ключа или индекса текущего элемента.
- Удаление дочерних узлов по ключу, индексу или текущей позиции итератора, а также полная очистка структурных узлов.
- Обмен данными с объектами `Array` и `Query`: запись содержимого массива в JSON-узел, получение JSON-массива в виде `Array`, сериализация и десериализация объекта `Query`.
- Установка признака константности узла, запрещающего модифицирующие операции.

Используется в прикладных скриптах EME-L для хранения, обработки и передачи структурированных данных, конфигураций и параметров в формате JSON.

Объект класса `Json` в языке EME-L создаётся через `Object()`. Все операции выполняются над корневым узлом или его дочерними элементами. Методы, возвращающие ссылку на дочерний узел (`GetNodeAt`, `GetCurrentNode`, `AddNodeAt`, `PushBackNode`), возвращают новый объект `Json`, разделяющий внутреннее дерево с исходным объектом — изменения дочернего узла отражаются в исходном дереве.

## Создание объекта класса Json

Класс `Json` в языке EME-L имеет два конструктора — на ноль и на один аргумент.

```EME-L
'Конструктор без аргументов — корневой узел типа null'
objJson = Object("Json");
```

```EME-L
'Конструктор с парсингом JSON-строки (ошибки парсинга подавляются)'
objJson = Object("Json", "{\"name\":\"EME\",\"version\":1}");
```

Конструктор с одним аргументом в языке EME-L вызывает `ParseNoExcept` — при синтаксической ошибке узел остаётся типа null, исключение не генерируется. Чтобы получить текст ошибки парсинга, используйте метод `ParseNoExcept` после создания объекта.

## Типы узлов JSON в языке EME-L

Каждый узел JSON в классе Json языка EME-L имеет один из восьми типов, возвращаемых методом `GetType` в виде числовой константы.

| Константа | Значение | Описание |
|-----------|----------|----------|
| `JSON_NODE_TYPE_UNKNOWN` | 0 | Неизвестный тип. |
| `JSON_NODE_TYPE_NULL` | 1 | null. |
| `JSON_NODE_TYPE_BOOLEAN` | 2 | Логическое значение. |
| `JSON_NODE_TYPE_STRING` | 3 | Строка. |
| `JSON_NODE_TYPE_INTEGER` | 4 | Целое число. |
| `JSON_NODE_TYPE_DOUBLE` | 5 | Вещественное число. |
| `JSON_NODE_TYPE_ARRAY` | 6 | Массив. |
| `JSON_NODE_TYPE_OBJECT` | 7 | Объект. |

Простые типы в системе EME.WMS и языке EME-L: null, boolean, string, integer, double. Структурные типы: array, object.

## Константность узла Json

Методы управления признаком константности в классе Json языка EME-L. При установленной константности любые модифицирующие операции с узлом генерируют исключение.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetConst(arg0)` | arg0: Boolean | Empty | Устанавливает (true) или снимает (false) признак константности текущего JSON-узла. |
| `IsConst()` | — | Boolean | true — константность установлена, false — нет. |

## Проверка типа узла Json

Методы проверки типа в классе Json языка EME-L существуют в трёх вариантах: для текущего узла (без суффикса), для дочернего узла по ключу/индексу (суффикс `At`), для текущего узла в итерации (префикс `IsCurrent`). Проверки для дочернего узла генерируют ошибку, если родительский узел не является object для строкового ключа или array для целочисленного индекса, дочерний узел не найден или индекс выходит за пределы массива.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetType()` | — | Integer | Тип текущего узла (константа JSON_NODE_TYPE_*). |
| `GetTypeAt(arg0)` | arg0: String/Integer | Integer | Тип дочернего узла по ключу (String) или индексу (Integer). |
| `GetCurrentType()` | — | Integer | Тип текущего узла в итерации. Ошибка, если итератор не установлен или узел удалён. |
| `IsNullType()` / `IsNullTypeAt(arg0)` / `IsCurrentNullType()` | — / arg0 / — | Boolean | Проверка типа null в языке EME-L. |
| `IsBoolType()` / `IsBoolTypeAt(arg0)` / `IsCurrentBoolType()` | — / arg0 / — | Boolean | Проверка типа boolean. |
| `IsStringType()` / `IsStringTypeAt(arg0)` / `IsCurrentStringType()` | — / arg0 / — | Boolean | Проверка типа string. |
| `IsIntegerType()` / `IsIntegerTypeAt(arg0)` / `IsCurrentIntegerType()` | — / arg0 / — | Boolean | Проверка типа integer. |
| `IsDoubleType()` / `IsDoubleTypeAt(arg0)` / `IsCurrentDoubleType()` | — / arg0 / — | Boolean | Проверка типа double. |
| `IsSimpleType()` / `IsSimpleTypeAt(arg0)` / `IsCurrentSimpleType()` | — / arg0 / — | Boolean | Проверка простого типа (null/boolean/string/integer/double). |
| `IsObjectType()` / `IsObjectTypeAt(arg0)` / `IsCurrentObjectType()` | — / arg0 / — | Boolean | Проверка типа object. |
| `IsArrayType()` / `IsArrayTypeAt(arg0)` / `IsCurrentArrayType()` | — / arg0 / — | Boolean | Проверка типа array. |
| `IsStructuredType()` / `IsStructuredTypeAt(arg0)` / `IsCurrentStructuredType()` | — / arg0 / — | Boolean | Проверка структурного типа (object/array) в языке EME-L. |

## Чтение значений из узлов Json

Методы чтения значений класса Json в языке EME-L. Значения простых типов (String, Integer, Real, Boolean) возвращаются напрямую; для null — Empty.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `IsKeyExists(arg0)` | arg0: String | Boolean | Проверка существования ключа в object. Ошибка, если родительский узел не object или ключ не строка. |
| `GetValue()` | — | String/Integer/Real/Boolean/Empty | Значение текущего узла простого типа. Empty для null. Ошибка, если тип не простой. |
| `GetValueAt(arg0)` | arg0: String/Integer | String/Integer/Real/Boolean/Empty | Значение дочернего узла по ключу или индексу. |
| `GetValueAtOrDefault(arg0, arg1)` | arg0: String/Integer, arg1: значение по умолчанию | String/Integer/Real/Boolean/Empty | Значение дочернего узла или arg1 при ошибке доступа. Генерирует ошибку только при неверном типе arg0. |
| `GetCurrentValue()` | — | String/Integer/Real/Boolean/Empty | Значение текущего узла в итерации простого типа. |
| `GetNodeAt(arg0)` | arg0: String/Integer | Json (ссылка) | Объект дочернего узла по ключу или индексу в языке EME-L. |
| `GetCurrentNode()` | — | Json (ссылка) | Объект текущего узла в итерации. |
| `IsEmpty()` | — | Boolean | true — object/array не содержит дочерних элементов. Ошибка, если узел не object/array. |
| `GetSize()` | — | Integer | Количество дочерних элементов object/array. Ошибка, если узел не object/array. |

## Запись значений в узлы Json

Модифицирующие методы класса Json в языке EME-L генерируют ошибку, если установлена константность узла. При записи по ключу в узел типа null тип автоматически меняется на object; при работе с массивом в узле типа null — на array.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `PutValue(arg0)` | arg0: String/Integer/Real/Boolean/Empty | Empty | Записывает простое значение в текущий узел. Сбрасывает активную итерацию. |
| `PutValueAt(arg0, arg1)` | arg0: String/Integer, arg1: простое значение | Empty | Записывает простое значение в дочерний узел по ключу или индексу. При записи по ключу в узел типа null тип меняется на object. |
| `AddNodeAt(arg0)` | arg0: String | Json (ссылка) | Создаёт новый дочерний узел с заданным ключом в object. Если родитель типа null, его тип меняется на object. Ошибка, если родитель не null/object или ключ уже существует. |
| `ResizeArray(arg0)` | arg0: Integer | Empty | Изменяет размер массива. Если узел типа null — становится array. |
| `PushBackValue(arg0)` | arg0: простое значение | Empty | Добавляет простое значение в конец массива. Если узел типа null — становится array. |
| `PushBackNode()` | — | Json (ссылка) | Добавляет пустой узел в конец массива в языке EME-L. Если узел типа null — становится array. |

## Итерация по дочерним элементам Json

Для обхода дочерних элементов object или array класса Json в языке EME-L используйте `SetFirstLine` / `IsValidLine` / `SetNextLine` или оператор `Loop`, который вызывает эти методы автоматически. `GetCurrentKey` применим только при обходе object, `GetCurrentIndex` — только при обходе array.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetFirstLine()` | — | Empty | Устанавливает итератор на первый дочерний элемент. Ошибка, если узел не object/array. |
| `IsValidLine()` | — | Boolean | true — итератор указывает на валидный элемент. |
| `SetNextLine()` | — | Empty | Переход к следующему элементу. Ошибка, если узел не object/array. |
| `ResetIteration()` | — | Empty | Сбрасывает итератор. |
| `GetCurrentKey()` | — | String | Ключ текущего узла при обходе object. |
| `GetCurrentIndex()` | — | Integer | Индекс текущего узла при обходе array в языке EME-L. |

## Удаление узлов Json

Методы удаления класса Json в языке EME-L. Все методы генерируют ошибку при установленной константности.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `EraseAt(arg0)` | arg0: String/Integer | Empty | Удаляет дочерний узел по ключу или индексу. Ошибка, если активна итерация. |
| `EraseCurrent()` | — | Empty | Удаляет текущий узел в итерации. Родительский узел должен быть object. |
| `EraseAll()` | — | Empty | Удаляет все дочерние элементы object/array. Ошибка, если активна итерация. |

## Обмен с CIArray (класс Array)

Методы класса Json в языке EME-L преобразуют содержимое JSON-массива простых значений в объект `Array` и обратно. Элементы массива должны быть простого типа (String, Integer, Real, Bool).

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `PutArray(arg0)` | arg0: Array (по ссылке) | Empty | Записывает содержимое CIArray в текущий узел, превращая его в массив простых значений. Сбрасывает активную итерацию. |
| `PutArrayAt(arg0, arg1)` | arg0: String/Integer, arg1: Array (по ссылке) | Empty | Записывает CIArray в дочерний узел по ключу или индексу. |
| `GetArray()` | — | Array | Возвращает содержимое текущего узла типа array в виде CIArray. Ошибка, если узел не array или есть элемент непростого типа. |
| `GetArrayAt(arg0)` | arg0: String/Integer | Array | Возвращает содержимое дочернего узла типа array в виде CIArray. |

## Обмен с CIQuery (класс Query)

Методы класса Json в языке EME-L сериализуют и десериализуют объект `Query` в формате, совместимом с C# классом EmeQuery. Режим сериализации `"zip"` архивирует данные запроса.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `PutQuery(arg0)` | arg0: Query (по ссылке) | Empty | Сериализует Query в текущий узел. Сбрасывает активную итерацию. |
| `PutQuery(arg0, arg1)` | arg0: Query (по ссылке), arg1: String ("json"/"zip") | Empty | То же с режимом сериализации. `"json"` — по умолчанию, `"zip"` — архивирование. |
| `PutQueryAt(arg0, arg1)` | arg0: String/Integer, arg1: Query (по ссылке) | Empty | Сериализует Query в дочерний узел по ключу или индексу. |
| `PutQueryAt(arg0, arg1, arg2)` | arg0: String/Integer, arg1: Query (по ссылке), arg2: String ("json"/"zip") | Empty | То же с режимом сериализации. |
| `GetQuery(arg0)` | arg0: Query (по ссылке) | Empty | Десериализует Query из текущего узла. |
| `GetQueryAt(arg0, arg1)` | arg0: String/Integer, arg1: Query (по ссылке) | Empty | Десериализует Query из дочернего узла по ключу или индексу. |

## Парсинг и сериализация JSON

Методы парсинга и сериализации класса Json в языке EME-L. Кодировка результата: `"UTF-8"` или `"ANSI"` (по умолчанию). При передаче недопустимой кодировки генерируется ошибка.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `Parse(arg0)` | arg0: String | Empty | Парсит JSON-строку, заменяя содержимое текущего узла. Генерирует исключение при ошибке парсинга. |
| `ParseNoExcept(arg0)` | arg0: String | String/Empty | Парсит JSON-строку без генерации исключения. Возвращает текст ошибки при неудаче, иначе Empty. |
| `DumpToStr()` | — | String | Сериализует узел в компактную строку. Кодировка ANSI по умолчанию. |
| `DumpToStr(arg0)` | arg0: String ("UTF-8"/"ANSI") | String | То же с указанием кодировки результата. |
| `DumpToStrFormatted()` | — | String | Сериализует узел в отформатированную строку с отступами. Кодировка ANSI по умолчанию. |
| `DumpToStrFormatted(arg0)` | arg0: String ("UTF-8"/"ANSI") | String | То же с указанием кодировки результата. |
| `GetContent()` | — | String | Отформатированная JSON-строка в кодировке UTF-8 в языке EME-L. |

## Примеры использования класса Json

Парсинг JSON и чтение значения по ключу со значением по умолчанию в языке EME-L (шаблон из ERPFirmPolyus):

```EME-L
'Создать объект и распарсить строку параметров'
objJson = Object("Json");
objJson.Parse(ParamsStr);
If (objJson.GetValueAt("iblnr") == iblnrId)
    bFound = True;
End If
```

Построение JSON-запроса с нуля в языке EME-L (шаблон из HardwareOrdersDikomCrane):

```EME-L
'Создать объект и заполнить поля команды'
JSON = Object("Json");
JSON.PutValueAt("operation", Command);
JSON.PutValueAt("cellId", StorageId);
JSON.PutValueAt("barcode", ContainerBarcode);
RequestText = JSON.DumpToStr();
```

Чтение ответа от внешнего API со значениями по умолчанию в языке EME-L (шаблон из HardwareOrdersDikomCrane):

```EME-L
'JSONAnswer получен от HTTP-запроса'
If (JSONAnswer.IsNullType())
    ErrorString = "Ошибка обработки запроса";
Else
    JobStatus = JSONAnswer.GetValueAtOrDefault("status", 2);
    ErrorCode = JSONAnswer.GetValueAtOrDefault("errorCode", 0);
End If
```

Обход объекта с помощью итератора в языке EME-L:

```EME-L
objJson = Object("Json", "{\"a\":1,\"b\":2,\"c\":3}");
objJson.SetFirstLine();
Loop (objJson)
    Key = objJson.GetCurrentKey();
    Value = objJson.GetCurrentValue();
End Loop;
```

## См. также

- [Класс Array](./Array.md) — обмен данными через `PutArray`/`GetArray` в языке EME-L.
- [Класс Query](./Query.md) — сериализация запросов через `PutQuery`/`GetQuery` в языке EME-L.
