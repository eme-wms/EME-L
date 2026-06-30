PropertyTree CIPropertyTree property_tree BoostPropertyTree JSON GetJSON PutJSON AddChild PushBack Put Get GetArray GetMap GetKey GetParams GetQuery GetContent AddQuery PutArray PutMap Empty Delete SetFirstLine SetNextLine IsValidLine IsEmpty IsConst SetConst

### PropertyTree — класс работы с древовидными структурами данных JSON в языке EME-L

Класс `PropertyTree` (C++ `CIPropertyTree`) в языке EME-L предназначен для работы с древовидными структурами данных (property tree) в формате JSON на базе `boost::property_tree`. Объект `PropertyTree` создаётся в скриптах EME-L для программного создания, модификации, обхода и сериализации иерархических данных. Каждый узел дерева в системе EME.WMS может содержать скалярное значение (строка, целое число, вещественное число, boolean, дата/время) и/или набор именованных дочерних узлов и безымянных элементов массива.

В языке EME-L класс `PropertyTree` является основным инструментом для работы с JSON-данными и формирования ответов HTTP-сервера в системе EME.WMS. Альтернативный класс `Json` предоставляет программный доступ к тем же структурам через иную модель узлов (null, boolean, string, integer, double, array, object). Взаимодействие с классом `Query` поддерживает два режима сериализации: `"json"` (узлы дерева) и `"zip"` (сериализация + сжатие + Base64).

#### Создание объекта PropertyTree в языке EME-L

Класс `PropertyTree` имеет один конструктор — без аргументов. Создаёт пустое дерево свойств на базе `boost::property_tree`. Данные заполняются методами `Put`, `AddChild`, `PushBack`, `PutJSON`, `PutArray`, `PutMap` и `AddQuery`.

```EME-L
'Создает пустое дерево свойств (PropertyTree) на базе boost::property_tree'
objTree = Object("PropertyTree");
```

#### Константы и внутренние параметры PropertyTree в языке EME-L

Внутренние хеш-таблицы дочерних узлов (`m_Children`) и кэшированных массивов (`m_Array`) инициализируются размером **101** (простое число). Значение `DevMode` в дереве (метод `GetJSON`) включает режим форматирования `boost::property_tree::write_json` с отступами; по умолчанию выключен.

### Методы редактирования дерева PropertyTree в языке EME-L

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `Put(key, value)` | key: String, value: String / Integer / Real / Boolean / DateTime | Empty | Добавляет или обновляет элемент в узле дерева по заданному ключу. Если узел отсутствует, он создаётся. Не поддерживает передачу объектов (`Object`) — при передаче объекта генерируется внутренняя ошибка. Если дерево помечено как неизменяемое, выводится сообщение об ошибке. |
| `AddChild(key)` | key: String | PropertyTree | Добавляет новый дочерний узел с указанным именем и возвращает ссылку на объект `PropertyTree`, представляющий этот узел. Если узел с таким именем уже существует, создаётся дублирующийся узел. Если дерево неизменяемое, выводится сообщение об ошибке. |
| `PushBack()` | — | PropertyTree | Добавляет пустой дочерний узел без имени (элемент массива) в текущий узел дерева и возвращает ссылку на объект `PropertyTree`, представляющий добавленный элемент. Если дерево неизменяемое, выводится сообщение об ошибке. |
| `PutJSON(jsonString)` | jsonString: String | Empty или String | Полностью заменяет содержимое дерева данными из строки в формате JSON. Исходный объект перезаписывается. Если дерево неизменяемое, выводится сообщение об ошибке. При ошибке парсинга JSON возвращается строка с описанием ошибки; при успехе — пустое значение. |
| `PutArray(nodeName, array)` | nodeName: String, array: Array | Empty | Добавляет массив из объекта `Array` в узел дерева с указанным именем. Если узел отсутствует, он создаётся; если существует, значение перезаписывается. Вещественные значения сохраняются как `float` (сужение из `double`). Если передан пустой массив, в узел добавляется один пустой элемент. Если дерево неизменяемое или передан не объект `Array`, выводится сообщение об ошибке. |
| `PutMap(nodeName, map)` | nodeName: String, map: Map | Empty | Добавляет объект `Map` в дочернюю ветку с именем `nodeName`. Служебные узлы метаданных (`Type`, `Count`, `HashSize`) записываются в **текущий (корневой) узел дерева**, а не внутрь `nodeName`; дочерние элементы `Key`/`Value`/`Type` — внутри `nodeName`. Если узел отсутствует, он создаётся; если существует, значение перезаписывается. Если дерево неизменяемое или передан не объект `Map`, выводится сообщение об ошибке. |
| `AddQuery(query)` | query: Query | Empty | Добавляет в дерево узел с именем `"query"`, содержащий данные объекта `Query`, в режиме `"json"`. Если узел уже существует, добавляется дублирующийся узел. Если дерево неизменяемое или передан не объект `Query`, выводится сообщение об ошибке. |
| `AddQuery(query, mode)` | query: Query, mode: String | Empty | Добавляет в дерево узел с именем `"query"`, содержащий данные объекта `Query`, в заданном режиме. Режим `"json"` сохраняет поля и строки запроса в виде узлов дерева; значения строк ключуются по имени поля (`FieldNarrative`), а разделитель пути `\t` запрещён в именах полей. Режим `"zip"` сериализует запрос, сжимает его и сохраняет в виде Base64-строки (`ArchiveData`/`ArchiveSize`/`ArchiveAlgo`). Если дерево неизменяемое или передан не объект `Query`, выводится сообщение об ошибке. |
| `Empty()` | — | Empty | Полностью очищает содержимое дерева: удаляет все дочерние узлы, массивы и значения. Если дерево неизменяемое, выводится сообщение об ошибке. |
| `Delete()` | — | Empty | Удаляет текущий дочерний узел дерева, установленный вызовами `SetFirstLine` или `SetNextLine`. После удаления позиция итерации сбрасывается (`CurrentKey`/`CurrentTree`/`CurrentPosition`). Если дерево неизменяемое, выводится сообщение об ошибке. |
| `Delete(key)` | key: String | Empty | Удаляет узел дерева с указанным именем. Если узел не существует, дерево не изменяется. Если имеется несколько узлов с одинаковым именем, удаляются все. |
| `SetConst(isConst)` | isConst: Boolean | Empty | Устанавливает или снимает признак неизменяемости дерева. Если значение `true`, дерево становится неизменяемым; последующие модифицирующие операции будут вызывать ошибку. |

### Методы получения данных из дерева PropertyTree в языке EME-L

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `Get()` | — | PropertyTree | Возвращает ссылку на объект `PropertyTree`, соответствующий текущему дочернему узлу, заданному последними вызовами `SetFirstLine` или `SetNextLine`. Если текущий узел не задан, генерируется внутренняя ошибка. |
| `Get(key)` | key: String | PropertyTree | Возвращает ссылку на дочерний объект `PropertyTree` по имени узла `key`. Если узел с таким именем не существует, возвращается пустая ссылка. |
| `Get(key, defaultValue)` | key: String, defaultValue: String / Integer / Real / Boolean | соответствует defaultValue | Возвращает значение элемента узла дерева по ключу `key`, приведённое к типу переменной `defaultValue`. Поддерживаются строки, вещественные числа, целые числа и boolean. Если узел отсутствует, возвращается значение `defaultValue`. |
| `GetArray(nodeName, sample)` | nodeName: String, sample: Real / Integer / String | Array | Возвращает ссылку на объект `Array`, содержащий элементы дочернего массива узла дерева с указанным именем. Тип элементов массива определяется типом переменной `sample`: вещественное число, целое число или строка (по умолчанию). Если массив не найден, узел пуст или тип не соответствует, возвращается пустая ссылка. |
| `GetJSON()` | — | String | Возвращает строковое представление дерева в формате JSON. Кодировка возвращаемой строки — ANSI. |
| `GetJSON(encoding)` | encoding: String | String | Возвращает строковое представление дерева в формате JSON в указанной кодировке. Допустимые значения: `"UTF-8"` или `"ANSI"` (регистр не учитывается). При недопустимой кодировке выводится сообщение об ошибке. |
| `GetKey()` | — | String | Возвращает имя текущего узла дерева, установленного вызовами `SetFirstLine` или `SetNextLine`. Если текущий узел не установлен, возвращается пустая строка. |
| `GetMap(nodeName)` | nodeName: String | Map | Создаёт объект `Map` и заполняет его данными из дочерней ветки с именем `nodeName`. Метаданные `Type`/`Count`/`HashSize` читаются из **текущего (корневого) узла**, а не из `nodeName`; размер хеш-таблицы итоговой карты увеличивается до `count*1.3`, если исходный `HashSize` меньше `Count`. Если узел не существует или `Count == 0`, возвращается пустой объект `Map`. |
| `GetParams(params)` | params: Params | Empty | Читает параметры из узла `"parameters"` дерева и заполняет ими переданный объект `Params`. Если узел `"parameters"` отсутствует, объект `Params` не изменяется. |
| `GetQuery(query)` | query: Query | Empty | Заполняет объект `Query` данными из узла `"query"` дерева, ранее добавленного методом `AddQuery`. Если узел отсутствует, объект `Query` не изменяется. |
| `GetContent()` | — | String | Возвращает строковое представление дерева в формате JSON в кодировке ANSI. Предназначен для отладочного вывода. |

### Методы итерации и обхода дочерних узлов PropertyTree в языке EME-L

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetFirstLine()` | — | Empty | Устанавливает первый дочерний узел дерева в качестве текущего для итерации. Может использоваться в цикле `Loop`. Если дочерние узлы отсутствуют, текущий узел становится недействительным. |
| `SetNextLine()` | — | Empty | Переходит к следующему дочернему узлу дерева. Может использоваться в цикле `Loop`. Если достигнут конец списка, текущий узел становится недействительным. |
| `IsValidLine()` | — | Boolean | Проверяет, установлен ли текущий дочерний узел дерева (корректна ли текущая позиция итерации). Возвращает `true`, если текущий узел установлен; `false` — в противном случае. |

### Методы проверки состояния дерева PropertyTree в языке EME-L

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `IsEmpty()` | — | Boolean | Проверяет, пусто ли дерево (не содержит ли оно элементов). Возвращает `true`, если дерево пустое; `false` — в противном случае. |
| `IsConst()` | — | Boolean | Проверяет, помечено ли дерево как неизменяемое. Возвращает `true`, если дерево неизменяемое; `false` — в противном случае. |

### Примеры использования PropertyTree в языке EME-L

Формирование JSON-ответа HTTP-сервера в системе EME.WMS — паттерн из `AIApi.txt`:

```EME-L
'#DEV_INITIALS 30.06.2026 EME company
Формирование JSON-ответа с результатами операции'
treeAnswer = Object("PropertyTree");
treeAnswer.Put("Result", "Success Connection");

answerNode = treeAnswer.AddChild("Answer");
answerNode.Put("Err", "");
answerNode.Put("Count", 42);

treeAnswerString.Add(treeAnswer.GetJSON("UTF-8"));
```

Построение массива объектов через `AddChild` и `PushBack` в языке EME-L — паттерн из `AIApi.txt`:

```EME-L
'#DEV_INITIALS 30.06.2026 EME company
Построение массива параметров в ответе'
treeAnswer = Object("PropertyTree");
arrayJson = treeAnswer.AddChild("Answer");

arr = Object("Array");
arr.Add("value1");
arr.Add("value2");

elem = arrayJson.PushBack();
elem.Put("ParamName", "MyParam");
elem.Put("Type", 1);
elem.PutArray("ValueRange", arr);

Return treeAnswer.GetJSON("UTF-8");
```

Чтение массива из JSON-запроса в языке EME-L — паттерн из `AIApi.txt`:

```EME-L
'#DEV_INITIALS 30.06.2026 EME company
Чтение массива параметров из входящего JSON-запроса'
treeRequest = Object("PropertyTree");
treeRequest.PutJSON(requestBody);

arrParams = treeRequest.GetArray("params", Object("Array"));
If (~is_null(arrParams))
    For (arrParams.SetFirstLine(); arrParams.IsValidLine(); arrParams.SetNextLine())
        name = arrParams.Get();
        '... обработка name ...'
    End For
End If
```

Парсинг JSON-строки и обход дочерних узлов в скрипте EME-L:

```EME-L
'#DEV_INITIALS 30.06.2026 EME company
Парсинг JSON-строки и обход дочерних узлов'
objTree = Object("PropertyTree");
objTree.PutJSON('{"users": [{"name": "Иван", "age": 30}, {"name": "Петр", "age": 25}]}');

usersNode = objTree.Get("users");
If (is_NULL(usersNode) == False)
    usersNode.SetFirstLine();
    While (usersNode.IsValidLine())
        userNode = usersNode.Get();
        userName = userNode.Get("name", "");
        userAge = userNode.Get("age", 0);
        usersNode.SetNextLine();
    End While
End If
```

Работа с `Query` через `PropertyTree` в языке EME-L:

```EME-L
'#DEV_INITIALS 30.06.2026 EME company
Сохранение запроса в дерево и обратное восстановление'
objTree = Object("PropertyTree");
objQuery = Object("Query");
'... заполнение запроса ...'
objTree.AddQuery(objQuery, "json");
jsonString = objTree.GetJSON();

'Восстановление запроса из JSON'
objTree2 = Object("PropertyTree");
objTree2.PutJSON(jsonString);
objQuery2 = Object("Query");
objTree2.GetQuery(objQuery2);
```

### См. также

- [Json](./Json.md) — альтернативный класс работы с JSON-данными в языке EME-L
- [Array](./Array.md) — класс работы с массивами
- [Map](./Map.md) — класс работы с картами (хэш-таблицами)
- [Query](./Query.md) — класс работы с запросами
- [Parameters](./Parameters.md) — класс работы с параметрами
