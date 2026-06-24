# Юнит-тесты в EME-L

Синонимы: юнит-тесты, unit tests, TestJson, CHECK_EQ, раннер тестов, Console, тестирование объектов

## Что такое юнит-тесты в EME-L

Юнит-тесты — это изолированные проверки отдельных методов объекта. В отличие от ERP-тестов, они не требуют подключения к внешним системам и запускаются внутри одного класса.

Канонический пример юнит-тестов в проекте — семейство `Tests.TestJson.*`, которое проверяет все методы объекта `Json` (45 тестов, по одному на метод).

## Архитектура юнит-тестов

### 1. Раннер — `Tests.TestJson`

Раннер (`TestJson.txt`) отвечает за:
- Обнаружение всех дочерних тест-классов
- Запуск каждого теста
- Подсчёт статистики: `total`, `failed`, `passed`

```eme-l
Tests.TestJson
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************
totalCounter;
passedCounter;
failedCounter;

************************             Методы            ************************
RunTests()
{
    console = Object("Console");
    console.Show();
    console.PutText("\n========================================================");
    console.PutText("BEGIN CIJSON TESTS\n");

    classRef = Object().GetClassRef();

    classesRec = Object("dsDB", "EMELClass");
    classesRec.SetSkipMode();
    classesRec.GetParentClassRefFld().MustBeRefEQ(classRef);

    totalCounter = 0;
    failedCounter = 0;
    passedCounter = 0;

    counters = Object("Array");

    Loop (classesRec)
        console.PutText("\nRunning " + classesRec.GetName() + "...");
        class = Object("IClass", classesRec.GetName(), classesRec.GetName());
        res = class.RunTestAndCountFailedAndPassed(counters);
        is_split(res, ";", counters);

        totalCounter = totalCounter + counters.GetAt(0);
        failedCounter = failedCounter + counters.GetAt(1);
        passedCounter = passedCounter + counters.GetAt(2);
    End Loop

    console.PutText("\nEND CIJSON TESTS");
    console.PutText("TOTAL: " + totalCounter);
    console.PutText("FAILED: " + failedCounter);
    console.PutText("========================================================");
}
```

**Ключевые моменты:**
- Используется `Object("IClass", ...)` для динамического создания экземпляра тест-класса
- Каждый тест возвращает три счётчика через `Array`: `total`, `failed`, `passed`
- Результаты выводятся в консоль через объект `Console`

### 2. Макрос проверки `CHECK_EQ`

```eme-l
CHECK_EQ(left, right)
{
    totalCounter = totalCounter + 1;
    console = Object("Console");
    console.Show();

    If (is_type_equ(left, right) & left == right)
        console.PutText("____PASSED: [" + decorateValue(left) + " == " + decorateValue(right) + "]");
        passedCounter = passedCounter + 1;
    Else
        console.PutText("XXX_FAILED: [" + decorateValue(left) + " != " + decorateValue(right) + "]");
        failedCounter = failedCounter + 1;
    End If
}
```

**Особенности `CHECK_EQ`:**
- Сначала проверяет совпадение типов (`is_type_equ`), затем значений (`==`)
- Выводит понятное сообщение: `PASSED` или `FAILED` с декорированными значениями
- Увеличивает соответствующий счётчик

### 3. Декоратор значений `decorateValue`

```eme-l
decorateValue(value)
{
    If (is_empty(value))
        Return "<EMPTY>";
    End If
    If (is_type_equ(value, ""))
        Return "\"" + value + "\"";
    End If
    If (is_type_equ(value, 0))
        Return value;
    End If
    If (is_type_equ(value, 0.0))
        Return value;
    End If
    If (is_type_equ(value, true))
        If (value == true)
            Return "TRUE";
        Else
            Return "FALSE";
        End If
    End If

    Return value;
}
```

## Пример тест-класса: `TestJson_001__GetType`

Каждый тест-класс наследуется от `Tests.TestJson` и переопределяет метод `RunTest()`.

```eme-l
Tests.TestJson.TestJson_001__GetType
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    CHECK_EQ( Object("Json", "null").GetType()     , JSON_NODE_TYPE_NULL    );
    CHECK_EQ( Object("Json", "123").GetType()      , JSON_NODE_TYPE_INTEGER );
    CHECK_EQ( Object("Json", "12.3").GetType()     , JSON_NODE_TYPE_DOUBLE  );
    CHECK_EQ( Object("Json", "true").GetType()     , JSON_NODE_TYPE_BOOLEAN );
    CHECK_EQ( Object("Json", "\"blah\"").GetType() , JSON_NODE_TYPE_STRING  );
    CHECK_EQ( Object("Json", "{}").GetType()       , JSON_NODE_TYPE_OBJECT  );
    CHECK_EQ( Object("Json", "[]").GetType()       , JSON_NODE_TYPE_ARRAY   );
}
```

**Принцип работы:**
1. Создаётся объект `Json` со строковым литералом
2. Вызывается тестируемый метод (`GetType()`)
3. Результат сравнивается с ожидаемым константой через `CHECK_EQ`
4. Раннер собирает статистику по всем `CHECK_EQ`

## Константы типов JSON

```eme-l
/* null */
JSON_NODE_TYPE_NULL    = 0;
/* целое число */
JSON_NODE_TYPE_INTEGER = 1;
/* число с плавающей точкой */
JSON_NODE_TYPE_DOUBLE  = 2;
/* true/false */
JSON_NODE_TYPE_BOOLEAN = 3;
/* строка */
JSON_NODE_TYPE_STRING  = 4;
/* объект { } */
JSON_NODE_TYPE_OBJECT  = 5;
/* массив [ ] */
JSON_NODE_TYPE_ARRAY   = 6;
```

## Другие юнит-тестовые классы

| Класс | Что тестирует |
|-------|---------------|
| `Tests.TestUnit` | Серверный режим (`is_server_emel()`, `is_server()`) |
| `Tests.Test` | «Песочница»: `PropertyTree`, `Semaphore`, `CubiScan`, `Map`, `BitBuffer`, HTTP/S, SMS |
| `Tests.Test2` | Создание `IClass` по строковому имени |
| `Tests.Test3` | Слой `GoodsItem` (работа с записью и свойствами) |
| `Tests.Test4` | Транзакции и массовое обновление |
| `Tests.TestGoodsItem` | Точность `GoodsItem` |
| `Tests.TestFindLine` | Хэшированный поиск в `dsDB` |
| `Tests.TestPickByLine` | `GetParamsByTransitOrPBL` |
| `Tests.TestPatch` | Сценарии патча (окна, кнопки) |
| `Tests.TestCEMERec` | Базовый класс `CEMERec` |
| `Tests.TestCEMESkip` | `SetSkipMode` + `MustBeEQAbt` |
| `Tests.TestXml` | Парсинг XML (курсы ЦБ) |
| `Tests.TestHttpsReq` | HTTP(S)-запросы и пагинация |
| `Tests.TestRPC` | RPC over file system |
| `Tests.TestDBWatch` | Объект `DBWatch` (дозор БД) |
| `Tests.TestUserError` | Пользовательская ошибка в транзакции |
| `Tests.TestTranslate` | Локализация (`tr()`) |
| `Tests.TestExternalDB` | ODBC к внешней БД |

## Примеры из песочницы `Tests.Test`

Файл `Test.txt` — «песочница» разработчиков с множеством примеров использования объектов.

### CubiScan — измерение ВГХ

```eme-l
Cubi()
{
    CubiScan = Object("CubiScan", "10.24.2.251", 443);
    CubiScan.SetDebug(True);
    Result = CubiScan.Ping();
    is_message("Cubi Ping", Result, 1, 1);

    Result = CubiScan.Measure();
    is_message("Cubi Measure", Result, 1, 1);
    If (Result == 200)
        is_message("Cubi ВГХ",
            "Length = " + CubiScan.GetLength() +
            ", Width = " + CubiScan.GetWidth() +
            ", Height = " + CubiScan.GetHeight() +
            ", Weight = " + CubiScan.GetWeight(), 1, 1);
    End If
}
```

### Фильтр `MustBeInMap`

```eme-l
TestMustBeInMap()
{
    StockStatusMap = Object("Map", 17, 0);
    StockStatusMap.SetAt(is_char("M"), 0);
    StockStatusMap.SetAt(is_char("K"), 0);

    r_Registers = Object("dsDB", "Registers");
    r_Registers.SetSkipMode();
    r_Registers.GetStockStatusFld().MustBeInMap(StockStatusMap);
    Count = 0;
    Loop (r_Registers)
        Count = Count + 1;
    End Loop

    is_message("TestMustBeInMap", Count, "OK", "INFORMATION");
}
```

### Присоединённое поле времени (`AttachTimeFld`)

```eme-l
TestAttachTimeFld()
{
    r_Document = Object("dsDB", "Document");
    r_Document.SetSkipMode();
    'Присоединяем время к дате для фильтрации по datetime'
    r_Document.GetCreateDateFld().AttachTimeFld();
    r_Document.GetCreateDateFld().MustBeGT(is_date(19, 11, 2018, 11, 0));
    r_Document.GetCreateDateFld().MustBeLT(is_date(19, 11, 2018, 12, 0));
    Count = 0;
    Loop (r_Document)
        Count = Count + 1;
    End Loop

    is_message("TestAttachTimeFld", Count, "OK", "INFORMATION");
}
```

### Копирование записей

```eme-l
CopyOrder()
{
    nLineFrom = 291415;
    is_transaction(1, tr("Копирование заказа"));

    dbOrder = Object("dsDB", "Orders");
    dbOrder.AppendAndSetLine();
    dbOrder.CopyLineFrom(nLineFrom);
    dbOrder.PutOrdersBatchRef(NULL_REF);
    dbOrder.PutShipmentRef(NULL_REF);
    dbOrder.ClrIntroduced();
    dbOrder.ClrFinished();
    dbOrder.PutOrderNo(dbOrder.GetOrderNo() + "_" + is_time());

    'Копируем строки'
    dbLines = Object("dsDB", "OrdersLines");
    dbLines.SetChain("@Заказ", nLineFrom);
    arr = Object("Array");
    For (dbLines.SetFirstLine(); dbLines.IsValidLine(); dbLines.SetNextLine())
        arr.Add(dbLines.GetLine());
    End For

    dbLines.SetChain("@Заказ", dbOrder.GetLine());
    For (i = 0; i < arr.GetSize(); i = i + 1)
        dbLines.AppendAndSetLine();
        dbLines.CopyLineFrom(arr.GetAt(i));
        dbLines.PutOrderRef(dbOrder.GetLine());
    End For

    dbOrder.Introduce();
    is_transaction(-1);
}
```

### Семафор

```eme-l
rwer()
{
    Semaphore = Object("Semaphore", "dfsdfsdfsdf", False);
    If (~Semaphore.Check())
        Semaphore.Set();
    End If

    'Проверить состояние семафора: TRUE — установлен'
    SemSet = Semaphore.Check();
}
```

### Итерация по `Map` и `BitBuffer`

```eme-l
MapTest()
{
    Map = Object("Map");
    Map.SetAt("a", 1);
    Map.SetAt("b", 2);
    Loop(Map)
        Key = Map.GetKey();
        Value = Map.GetValue();
    End Loop

    BitBuffer = Object("BitBuffer");
    BitBuffer.HewItemGrow(2);
    Loop(BitBuffer)
        BitBuffer.GetLine();
    End Loop
}
```

---

## Запросы-тесты

В директории `Запросы/` находятся SQL-тесты:

| Файл | Назначение |
|------|------------|
| `Tests.DupKeys.txt` | Дубли ключей |
| `Tests.DupLotID.txt` | Дубли `LotID` |
| `Tests.DupLotNo.txt` | Дубли номера партии |
| `Tests.ExternalDB.txt` | SQL к внешним БД |
| `Tests.SqlServer.txt` | Проверка под SQL Server |
| `Tests.TestDocLinesDups.txt` | Дубли строк документов |
| `Tests.TestRegistersDups.txt` | Дубли регистров |
| `Tests.TestSpLock.txt` | Блокировки хранимых процедур |
| `Tests.TestStoredProcedure.txt` | Хранимые процедуры |
| `Tests.TestWITH.txt` | SQL `WITH` |

## Как смотреть работу объекта

| Объект | Где смотреть |
|--------|--------------|
| `Json` / `PropertyTree` | `Tests.TestJson.TestJson_001..045` |
| `dsDB` (записи) | `testChainRestore`, `testFindCell`, `Tests.Test` |
| `Query` | `ERPTests`, `Tests.Test` |
| `Map` / `Array` / `BitBuffer` | `Tests.Test` |
| `Console` | `TestJson.txt` — `RunTests()` |
| `IClass` | `TestJson.txt` — динамический вызов |
| `Xml` | `ERPTests.TestXmlUpload()`, `TestXmlLoad()` |
