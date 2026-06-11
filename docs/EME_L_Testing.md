# Тестирование в EME-L

## Обзор системы тестов

В проекте EME.WMS существует несколько семейств тестов, каждое со своей целью, архитектурой и способом запуска. Все тестовые классы располагаются в директории `Классы/` базовой выгрузки проекта.

### Семейства тестов

| Семейство | Пространство имён | Назначение |
|-----------|-------------------|------------|
| **Юнит-тесты** | `Tests.TestJson.*` | Изолированная проверка методов объекта `Json` / `PropertyTree` |
| **ERP-тесты** | `ERPSolution.ERPTests` | Интеграционные тесты обмена данными между WMS и ERP-системой |
| **Самодиагностика** | `RobotHealer.healClasses.*` | Проверка здоровья БД, скорости операций, исправление проблем |
| **Общие / ad-hoc** | `Tests.Test`, `Tests.TestUnit` | Разработческие «песочницы», эксперименты, ручные проверки |

### Сравнение типов тестов

| Характеристика | Юнит-тесты | ERP-тесты | Самодиагностика |
|----------------|-----------|-----------|-----------------|
| **Что проверяет** | Один объект/метод | Интеграция с внешней ERP | Состояние БД и настроек |
| **Запуск** | Через `RunTests()` | Вручную по кнопке | Автоматически / по расписанию |
| **Результат** | passed / failed / total | Сообщение + лог | Оценка 1–10 + отчёт |
| **Классический assert** | Да (`CHECK_EQ`) | Нет | Нет |
| **Внешние зависимости** | Нет | Да (ERP, ODBC, БД) | Нет |

---

## Сводная таблица: класс → тест → что проверяет

### A. Юнит-тесты объекта `Json` (`Tests.TestJson.*`)

Основной раннер: `Tests.TestJson` (`TestJson.txt`). Каждый подтест — отдельный класс, проверяющий один метод `Json`.

| Файл | Класс | Что проверяет | Объект |
|------|-------|---------------|--------|
| `TestJson.txt` | `Tests.TestJson` | Раннер: `RunTests()`, `CHECK_EQ()`, счётчики | `Json` |
| `TestJson_001__GetType.txt` | `TestJson_001__GetType` | `GetType()` — распознавание типов узлов | `Json` |
| `TestJson_002__GetTypeAt.txt` | `TestJson_002__GetTypeAt` | `GetTypeAt()` — тип узла по пути | `Json` |
| `TestJson_003__GetCurrentType.txt` | `TestJson_003__GetCurrentType` | `GetCurrentType()` — тип текущего узла при итерации | `Json` |
| `TestJson_004__IsType_typed.txt` | `TestJson_004__IsType_typed` | `IsType()` — проверка типа с константами | `Json` |
| `TestJson_005__IsTypeAt_typed.txt` | `TestJson_005__IsTypeAt_typed` | `IsTypeAt()` — проверка типа по пути | `Json` |
| `TestJson_006__IsCurrentType_typed.txt` | `TestJson_006__IsCurrentType_typed` | `IsCurrentType()` — проверка типа текущего узла | `Json` |
| `TestJson_007__IsKeyExists.txt` | `TestJson_007__IsKeyExists` | `IsKeyExists()` — наличие ключа в объекте | `Json` |
| `TestJson_008__GetValue.txt` | `TestJson_008__GetValue` | `GetValue()` — чтение значения по ключу | `Json` |
| `TestJson_009__GetValueAt.txt` | `TestJson_009__GetValueAt` | `GetValueAt()` — чтение по пути | `Json` |
| `TestJson_010__GetValueAtOrDefault.txt` | `TestJson_010__GetValueAtOrDefault` | `GetValueAtOrDefault()` — значение или дефолт | `Json` |
| `TestJson_011__GetCurrentValue.txt` | `TestJson_011__GetCurrentValue` | `GetCurrentValue()` — значение при итерации | `Json` |
| `TestJson_012__GetNodeAt.txt` | `TestJson_012__GetNodeAt` | `GetNodeAt()` — получение под-узла по пути | `Json` |
| `TestJson_013__GetCurrentNode.txt` | `TestJson_013__GetCurrentNode` | `GetCurrentNode()` — текущий узел итерации | `Json` |
| `TestJson_014__PutValue.txt` | `TestJson_014__PutValue` | `PutValue()` — запись значения по ключу | `Json` |
| `TestJson_015__PutValueAt.txt` | `TestJson_015__PutValueAt` | `PutValueAt()` — запись по пути | `Json` |
| `TestJson_016__IsEmpty.txt` | `TestJson_016__IsEmpty` | `IsEmpty()` — пустой ли узел | `Json` |
| `TestJson_017__GetSize.txt` | `TestJson_017__GetSize` | `GetSize()` — размер массива/объекта | `Json` |
| `TestJson_018__AddNodeAt.txt` | `TestJson_018__AddNodeAt` | `AddNodeAt()` — добавление узла по пути | `Json` |
| `TestJson_019__ResizeArray.txt` | `TestJson_019__ResizeArray` | `ResizeArray()` — изменение размера массива | `Json` |
| `TestJson_020__PushBackValue.txt` | `TestJson_020__PushBackValue` | `PushBackValue()` — добавление значения в массив | `Json` |
| `TestJson_021__PushBackNode.txt` | `TestJson_021__PushBackNode` | `PushBackNode()` — добавление узла в массив | `Json` |
| `TestJson_022__PutArray.txt` | `TestJson_022__PutArray` | `PutArray()` — запись массива | `Json` |
| `TestJson_023__PutArrayAt.txt` | `TestJson_023__PutArrayAt` | `PutArrayAt()` — запись массива по пути | `Json` |
| `TestJson_024__GetArray.txt` | `TestJson_024__GetArray` | `GetArray()` — чтение массива | `Json` |
| `TestJson_025__GetArrayAt.txt` | `TestJson_025__GetArrayAt` | `GetArrayAt()` — чтение массива по пути | `Json` |
| `TestJson_026__PutQuery.txt` | `TestJson_026__PutQuery` | `PutQuery()` — загрузка `Query` в JSON | `Json` |
| `TestJson_027__PutQueryAt.txt` | `TestJson_027__PutQueryAt` | `PutQueryAt()` — загрузка `Query` по пути | `Json` |
| `TestJson_028__GetQuery.txt` | `TestJson_028__GetQuery` | `GetQuery()` — выгрузка JSON в `Query` | `Json` |
| `TestJson_029__GetQueryAt.txt` | `TestJson_029__GetQueryAt` | `GetQueryAt()` — выгрузка по пути | `Json` |
| `TestJson_030__EraseAt.txt` | `TestJson_030__EraseAt` | `EraseAt()` — удаление по пути | `Json` |
| `TestJson_031__EraseCurrent.txt` | `TestJson_031__EraseCurrent` | `EraseCurrent()` — удаление текущего узла | `Json` |
| `TestJson_032__EraseAll.txt` | `TestJson_032__EraseAll` | `EraseAll()` — очистка узла | `Json` |
| `TestJson_033__SetFirstLine.txt` | `TestJson_033__SetFirstLine` | `SetFirstLine()` — итерация по узлам | `Json` |
| `TestJson_034__IsValidLine.txt` | `TestJson_034__IsValidLine` | `IsValidLine()` — валидность текущей строки | `Json` |
| `TestJson_035__SetNextLine.txt` | `TestJson_035__SetNextLine` | `SetNextLine()` — переход к следующему узлу | `Json` |
| `TestJson_036__ResetIteration.txt` | `TestJson_036__ResetIteration` | `ResetIteration()` — сброс итератора | `Json` |
| `TestJson_037__GetCurrentKey.txt` | `TestJson_037__GetCurrentKey` | `GetCurrentKey()` — ключ текущего узла | `Json` |
| `TestJson_038__GetCurrentIndex.txt` | `TestJson_038__GetCurrentIndex` | `GetCurrentIndex()` — индекс текущего узла | `Json` |
| `TestJson_039__iteration.txt` | `TestJson_039__iteration` | Комплексная проверка итерации | `Json` |
| `TestJson_040__ParseNoExcept.txt` | `TestJson_040__ParseNoExcept` | `ParseNoExcept()` — парсинг без исключений | `Json` |
| `TestJson_041__Parse.txt` | `TestJson_041__Parse` | `Parse()` — парсинг строки | `Json` |
| `TestJson_042__DumpToStr.txt` | `TestJson_042__DumpToStr` | `DumpToStr()` — сериализация в строку | `Json` |
| `TestJson_043__DumpToStrFormatted.txt` | `TestJson_043__DumpToStrFormatted` | `DumpToStrFormatted()` — форматированная сериализация | `Json` |
| `TestJson_044__GetContent.txt` | `TestJson_044__GetContent` | `GetContent()` — получение «сырого» содержимого | `Json` |
| `TestJson_045__codepage.txt` | `TestJson_045__codepage` | Работа с кодировками (`ANSI`/`UTF-8`) | `Json` |

### B. ERP-тесты (`ERPSolution.ERPTests`)

Класс: `ERPSolution.ERPTests` (`ERPTests.txt`)

| Метод | Что тестирует | Объекты |
|-------|---------------|---------|
| `Test()` | Создание таблиц во внешней БД, экспорт товаров (`ERPGoodsComponent`) | `ExternalDB`, `Query` |
| `Connect()` | Подключение к внешней БД по ODBC (PostgreSQL / SQL Server / EME DB) | `ExternalDB` |
| `Execute()` | Выполнение SQL-запроса во внешней БД | `ExternalDB`, `Query` |
| `CreateTable()` / `UpdateTable()` | DDL/DML операции через `ExternalDB` | `ExternalDB`, `Query` |
| `TestNewMessagesSpeed1/2/3()` | Скорость обнаружения новых сообщений во внешней БД | `ExternalDB` |
| `TestERPEngineSharp()` | Тест C#-движка `EME.ERPEngine` (экспорт товаров) | `Automation` |
| `CreateQualityChangeMessage()` | Формирование сообщения `quality_change` | `Automation` |
| `CreateChangeMessage()` | Формирование сообщения `change` | `Automation` |
| `CreateClientsMessage()` | Формирование сообщения `clients` | `Automation` |
| `CreateOrdersMessage()` | Формирование сообщения `orders` | `Automation` |
| `TestSqlServer()` | Прямой SQL-запрос к EME SQL Server | `EMEQuerySQLExternalDB` |
| `TestEnumTables()` | Перечисление таблиц внешней БД | `ExternalDB` |
| `TestImportReceipts()` | Импорт уведомлений о приёмке (`receipt`) | `Automation` |
| `TestOnCloseDocumentInThread()` | Экспорт документа в фоновом потоке | `System` |
| `TestJsonUpload()` / `TestJsonLoad()` | Выгрузка / загрузка сообщения в JSON | `PropertyTree`, `IDoc` |
| `TestXmlUpload()` / `TestXmlLoad()` | Выгрузка / загрузка сообщения в XML | `Xml` |
| `CreateWordDocument()` | Автоматизация Word (`Word.Application`) | `Automation` |
| `ExtractFields()` / `TestExtractFields()` | Извлечение полей, используемых в ERP | `IClass` |
| `ImportGoodsFlatFile()` | Импорт номенклатуры из CSV/TXT | `EMEQueryTextFile` |

### C. Самодиагностика (`RobotHealer.healClasses.*`)

| Файл | Класс | Что тестирует / что лечит |
|------|-------|---------------------------|
| `testExample.txt` | `testExample` | Шаблон теста производительности (балл 1–10) |
| `testDebug.txt` | `testDebug` | Режим отладки (`d_emel`) и флаг дампа |
| `testChainRestore.txt` | `testChainRestore` | Разрывы цепочек в `DocLines` (`Prev`/`Next`) |
| `testAccessibility.txt` | `testAccessibility` | Корректность флагов доступности ячеек |
| `testFindNonAddrCell.txt` | `testFindNonAddrCell` | Состояние безадресных ячеек |
| `testFindCell.txt` | `testFindCell` | Скорость автопоиска ячеек для размещения |
| `testGood.txt` | `testGood` | Заглушка для тестов проблем товаров |
| `testGood_openDialog.txt` | `testGood_openDialog` | Скорость открытия диалога «Продукт» |
| `testStructure.txt` | `testStructure` | Корректность сортировочных полей-цепочек |
| `testSelfTest.txt` | `testSelfTest` | Работоспособность модуля самодиагностики |
| `testRegisters.txt` | `testRegisters` | Фрагментация записи регистров |
| `testKernelSettings.txt` | `testKernelSettings` | Параметры ядра: архивы, HTTP, DLL, кэш |

---

## Что такое ERP-тесты

**ERP-тесты** — это интеграционные тесты подсистемы обмена данными между WMS и внешней ERP-системой (1С, SAP, учётные системы заказчика). Они собраны в классе `ERPSolution.ERPTests` (`ERPTests.txt`).

В отличие от юнит-тестов, ERP-тесты:
- Требуют подключения к внешней базе данных через ODBC
- Проверяют канал связи, форматы сообщений и движок обмена
- Запускаются вручную (кнопкой) разработчиком
- Не используют классические `assert` — результат проверяется визуально или через логи

### Цели ERP-тестов

1. **Проверить канал связи** — подключение к внешней БД через ODBC (`ExternalDB`), скорость запросов
2. **Проверить движок обмена** — C#-компонент `EME.ERPEngine` (создание сообщений `goods`, `orders`, `clients`, `asn`, `receipt`, `change`, `quality_change`)
3. **Проверить форматы** — корректность выгрузки/загрузки в JSON и XML через `PropertyTree` + `IDoc`
4. **Проверить маппинг полей** — что все поля, используемые в обработчиках `ERP*Component`, зарегистрированы и не противоречат друг другу (`TestOnErrors`)
5. **Проверить импорт плоских файлов** — загрузка номенклатуры из CSV/TXT через `EMEQueryTextFile`

---

## Юнит-тесты: подробно

### Архитектура

#### 1. Раннер — `Tests.TestJson`

Раннер (`TestJson.txt`) отвечает за обнаружение всех дочерних тест-классов, запуск каждого теста и подсчёт статистики.

```eme-l
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

Ключевые моменты:
- Используется `Object("IClass", ...)` для динамического создания экземпляра тест-класса
- Каждый тест возвращает три счётчика через `Array`: `total`, `failed`, `passed`
- Результаты выводятся в консоль через объект `Console`

#### 2. Макрос проверки `CHECK_EQ`

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

Особенности `CHECK_EQ`:
- Сначала проверяет совпадение типов (`is_type_equ`), затем значений (`==`)
- Выводит понятное сообщение: `PASSED` или `FAILED` с декорированными значениями
- Увеличивает соответствующий счётчик

#### 3. Пример тест-класса

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

Принцип работы:
1. Создаётся объект `Json` со строковым литералом
2. Вызывается тестируемый метод (`GetType()`)
3. Результат сравнивается с ожидаемой константой через `CHECK_EQ`

### Дополнительные примеры из `Tests.Test`

#### CubiScan — измерение ВГХ

```eme-l
Cubi()
{
    CubiScan = Object("CubiScan", "10.24.2.251", 443);
    CubiScan.SetDebug(True);
    Result = CubiScan.Ping();
    Result = CubiScan.Measure();
    If (Result == 200)
        Length = CubiScan.GetLength();
        Width = CubiScan.GetWidth();
        Height = CubiScan.GetHeight();
        Weight = CubiScan.GetWeight();
    End If
}
```

#### Фильтр `MustBeInMap`

```eme-l
TestMustBeInMap()
{
    StockStatusMap = Object("Map", 17, 0);
    StockStatusMap.SetAt(is_char("M"), 0);
    StockStatusMap.SetAt(is_char("K"), 0);

    r_Registers = Object("dsDB", "Registers");
    r_Registers.SetSkipMode();
    r_Registers.GetStockStatusFld().MustBeInMap(StockStatusMap);
    Loop (r_Registers)
        ' Обработка '
    End Loop
}
```

#### Присоединённое поле времени

```eme-l
TestAttachTimeFld()
{
    r_Document = Object("dsDB", "Document");
    r_Document.SetSkipMode();
    r_Document.GetCreateDateFld().AttachTimeFld();
    r_Document.GetCreateDateFld().MustBeGT(is_date(19, 11, 2018, 11, 0));
    r_Document.GetCreateDateFld().MustBeLT(is_date(19, 11, 2018, 12, 0));
    Loop (r_Document)
        ' Фильтрация по дате+времени '
    End Loop
}
```

#### Копирование записей

```eme-l
CopyOrder()
{
    nLineFrom = 291415;
    is_transaction(1, tr("Копирование заказа"));
    dbOrder = Object("dsDB", "Orders");
    dbOrder.AppendAndSetLine();
    dbOrder.CopyLineFrom(nLineFrom);
    dbOrder.PutOrdersBatchRef(NULL_REF);
    dbOrder.ClrIntroduced();
    dbOrder.PutOrderNo(dbOrder.GetOrderNo() + "_" + is_time());

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

#### Семафор

```eme-l
Semaphore = Object("Semaphore", "my_semaphore", False);
If (~Semaphore.Check())
    Semaphore.Set();
End If
If (Semaphore.Check())
    ' Семафор установлен '
End If
```

#### Итерация по Map и BitBuffer

```eme-l
Map = Object("Map");
Map.SetAt("a", 1);
Loop(Map)
    Key = Map.GetKey();
    Value = Map.GetValue();
End Loop

BitBuffer = Object("BitBuffer");
BitBuffer.HewItemGrow(2);
Loop(BitBuffer)
    Line = BitBuffer.GetLine();
End Loop
```
4. Раннер собирает статистику по всем `CHECK_EQ`

### Константы типов JSON

```eme-l
JSON_NODE_TYPE_NULL    = 0   ' null
JSON_NODE_TYPE_INTEGER = 1   ' целое число
JSON_NODE_TYPE_DOUBLE  = 2   ' число с плавающей точкой
JSON_NODE_TYPE_BOOLEAN = 3   ' true/false
JSON_NODE_TYPE_STRING  = 4   ' строка
JSON_NODE_TYPE_OBJECT  = 5   ' объект { }
JSON_NODE_TYPE_ARRAY   = 6   ' массив [ ]
```

---

## ERP-тесты: подробно

### Основной класс

Файл: `ERPTests.txt`

```eme-l
ERPSolution.ERPTests
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
```

### Подключение к внешней БД

```eme-l
Connect(ExternalDB As "ExternalDB")
{
    'Error = ExternalDB.Connect(
        "ODBC:Driver={PostgreSQL ODBC Driver(ANSI)};Server=localhost;Port=5432;Database=WMS_ERP;Uid=postgres;Pwd=123;");'
    'Error = ExternalDB.Connect(
        "ODBC:Driver={SQL Server};Server=AMP-NOTEBOOK\\SQL2005;Database=WMS_KINEL;Uid=sa;Pwd=123;");'
    Error = ExternalDB.Connect(
        "ODBC:driver={EME DB};server=amp-syktyvkar;port=80;uid=amp;pwd=bnfl;");
    If (Error != "")
        is_message(
            "ExternalDBTest",
            tr("Не удалось подключить клиента к внешней базе данных:\n\n") + Error, "OK", "STOP");
        Return FALSE;
    End If

    Return TRUE;
}
```

### Тест движка `EME.ERPEngine`

```eme-l
TestERPEngineSharp()
{
    erp = Object("Automation", "EME.ERPEngine");
    Timeout = erp.GetQueryTimeout();
    erp.SetQueryTimeout(60);

    Error = erp.Connect();
    If (Error != "")
        is_message("TestERPEngineSharp", tr("Ошибка при запуске движка:\n") + Error, "OK", "STOP");
        Return;
    End If

    erp.BeginExport("erp", "wms", "goods");

    erp.AppendHeaderLine();
    erp.PutHeaderData("id", "12222258");
    erp.PutHeaderData("name", "РОССИЙСКИЙ Шоколад Горький70% 27(5х90г)");

    erp.SelectChild("packs");

    erp.AppendChildLine();
    erp.PutChildData("id", "Штука");
    erp.PutChildData("mu_code", "EA");
    erp.PutChildData("factor", 1);

    Error = erp.CommitExport();
    If (Error != "")
        erp.Disconnect();
        is_message("TestERPEngineSharp", tr("Ошибка при экспорте сообщения:\n") + Error, "OK", "STOP");
        Return;
    End If

    erp.Disconnect();
    is_message("TestERPEngineSharp", tr("Тест завершен!"), "OK", "INFORMATION");
}
```

Паттерн работы с `ERPEngine`:
1. `Connect()` — подключение к движку
2. `BeginExport(источник, назначение, тип_сообщения)` — начало экспорта
3. `AppendHeaderLine()` + `PutHeaderData(...)` — заполнение заголовка
4. `SelectChild("имя_дочерней_таблицы")` + `AppendChildLine()` + `PutChildData(...)` — заполнение строк
5. `CommitExport()` — фиксация сообщения
6. `Disconnect()` — отключение

---

## Самодиагностика: подробно

### Контракт healClasses

Каждый класс самодиагностики должен реализовывать три метода:

| Метод | Назначение | Возвращает |
|-------|------------|------------|
| `describe()` | Название тестируемой функции (для пользователя) | `String` |
| `test()` | Выполняет тест и ставит оценку | `Integer` (1–10) |
| `heal()` | Выполняет исправление проблем | `String` (отчёт) |

### Шаблон: `testExample`

Файл: `testExample.txt`

```eme-l
RobotHealer.healClasses.testExample
************************             Флаги             ************************
ДЛ: Нет
НК: Нет
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************

/* #KPI 13.09.2013 Пример класса для тестирования производительности одной функции программы */

/* метод возвращает название тестируемой функции */
describe()
{
    return tr("Вписать здесь название тестируемой функции");
}

/* метод выполняет тест и возвращает оценку */
test()
{
    rating = 1;
    ' TO DO тест должен поставить оценку от одного до десяти '
    rating = rating + SameTest1();
    rating = rating + SameTest2();
    Return rating;
}

/* метод выполняет необходимые действия для исправления проблем */
heal()
{
    strResult = "";

    is_transaction(1); ' тут нужно дать транзакции нормальное имя '
    ' TO DO вызов исправляющей функции '
    strResult = strResult + CureSameTest1();
    strResult = strResult + CureSameTest2();
    is_transaction(-1);

    Return strResult;
}
```

---

## Дозор базы данных (DBWatch)

`DBWatch` — механизм отслеживания изменений в БД между подготовкой данных и записью в транзакции.

```eme-l
Tests.TestDBWatch
************************          Конструктор          ************************
m_DBWatch = Object("DBWatch");
m_TrueSign;
m_Line;

************************             Методы            ************************
OnPrepare()
{
    r_GoodsItem = Object("dsDB", "GoodsItem");
    r_GoodsItem.SetFirstLine();
    m_TrueSign = FALSE;
    If (r_GoodsItem.IsValidLine() & ~r_GoodsItem.IsTrueSign())
        m_TrueSign = TRUE;
        m_Line = r_GoodsItem.GetLine();
    End If
    m_DBWatch.WatchFieldLine(r_GoodsItem.GetGlobalFlagsThreeFld());
}

OnChange()
{
    If (m_TrueSign)
        r_GoodsItem = Object("dsDB", "GoodsItem");
        r_GoodsItem.SetLine(m_Line);
        r_GoodsItem.SetTrueSign();
    End If
}

Do()
{
    is_db_watch(1);
    If (is_workstation_mode() != "MONO")
        OnPrepare();
    End If
    is_transaction(1, "Устанавливаем ЧЗ");
    is_db_watch(-1);
    If (is_workstation_mode() == "MONO" | m_DBWatch.Check() != "")
        OnPrepare();
    End If
    OnChange();
    is_transaction(-1);
}
```

**Паттерн DBWatch:**
1. `is_db_watch(1)` — начать наблюдение
2. `OnPrepare()` — читаем данные
3. `m_DBWatch.WatchFieldLine(...)` — добавляем поля в наблюдение
4. `is_transaction(1)` — открываем транзакцию
5. `is_db_watch(-1)` — завершаем наблюдение
6. `m_DBWatch.Check()` — проверяем изменения
7. Если изменились — повторяем `OnPrepare()`
8. `OnChange()` — записываем
9. `is_transaction(-1)` — закрываем транзакцию

---

## ERP-стресс-тесты и симулятор

Класс `ERPSolution.ERPStressTest` — массовая выгрузка и симуляция.

### Пачковый экспорт с BitBuffer

```eme-l
ExportAll(Source, Target, Name, RecordObject As "CEMERec", BitBufferFilter As "BitBuffer")
{
    m_ERPEngine.Launch(m_ERPEngine.GetConnStr());
    m_ERPEngine.StartPercentage("Читаем из записи", RecordObject.GetNoOfLines());
    RecordObject.SetFirstLine();
    Refs = Object("Array");
    While (GetNextExportBatch(RecordObject, BitBufferFilter, Refs))
        Error = m_ERPEngine.BeginExport(Source, Target, Name);
        If (Error == "")
            Loop (Refs)
                RecordObject.SetLine(Refs.Get());
                m_ERPEngine.NextPercentage(RecordObject.GetLine());
                Error = m_ERPEngine.DoExport(RecordObject);
                is_peek_all_messages();
                If (Error != "")
                    Break
                End If
            End Loop
        End If
        If (Error == "")
            Error = m_ERPEngine.EndExport();
        End If
    End While
}
```

### Верификаторы сообщений

| Верификатор | Тип |
|-------------|-----|
| `ERPAsnVerifier` | ASN |
| `ERPChangeVerifier` | CHANGE |
| `ERPGoodsVerifier` | GOODS |
| `ERPOrdersVerifier` | ORDERS |
| `ERPShipmentsVerifier` | SHIPMENTS |
| `ERPStaffVerifier` | STAFF |

### Симулятор EME.WMS

```eme-l
SimulateEmeWms()
{
    Menu = Object("IClass", "ActiveMenuObject_" + Object().GetObjectName(), "ActiveMenu");
    Menu.AddMenuItem("Тестовый склад", "CreateTestWarehouse");
    Menu.AddMenuItem("Приход#Случайный приход", "CreateReceipt", "");
    Menu.AddMenuItem("Отгрузка#Подобрать заказ", "CreateDespatch", "Assembly");
    Menu.AddMenuItem("Перемещение#Приход под заказ", "CreateReceiptPerPO", "Transfer");
    Menu.Run();
}
```

---

## Шаблоны для создания новых тестов

### Шаблон юнит-теста

```eme-l
Tests.TestJson.TestJson_XXX__MethodName
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    ' Создаём тестовый объект '
    json = Object("Json", "{ \"key\": \"value\", \"num\": 42 }");

    ' Проверяем метод '
    CHECK_EQ( json.MethodName()        , expectedValue1 );
    CHECK_EQ( json.MethodName("arg")   , expectedValue2 );

    ' Проверяем крайние случаи '
    emptyJson = Object("Json", "{}");
    CHECK_EQ( emptyJson.MethodName()   , defaultValue );
}
```

### Шаблон ERP-теста

```eme-l
TestMyIntegration()
{
    ExternalDB = Object("ExternalDB");
    If (~Connect(ExternalDB))
        Return;
    End If

    Query = Object("Query");
    Query.Create();

    Error = ExternalDB.Execute("SELECT * FROM my_table", Query);
    If (Error != "")
        is_message("TestMyIntegration", "Error: " + Error, "OK", "STOP");
        Return;
    End If

    If (Query.GetNoOfLines() > 0)
        is_message("TestMyIntegration", "OK: " + Query.GetNoOfLines() + " rows", "OK", "INFORMATION");
    Else
        is_message("TestMyIntegration", "No data found", "OK", "EXCLAMATION");
    End If

    ExternalDB.Disconnect();
}
```

### Шаблон самодиагностики

```eme-l
RobotHealer.healClasses.testMyFeature
************************             Флаги             ************************
ДЛ: Нет
НК: Нет
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************

describe()
{
    return tr("Проверка моей фичи");
}

test()
{
    rating = 10;
    r_Record = Object("dsDB", "MyRecord");
    r_Record.SetSkipMode();
    If (r_Record.GetNoOfLines() == 0)
        rating = 1;
    End If
    Return rating;
}

heal()
{
    strResult = "";
    is_transaction(1, tr("Исправление моей фичи"));
    strResult = strResult + tr("Исправление выполнено");
    is_transaction(-1);
    Return strResult;
}
```

### Шаблон теста с Browser

```eme-l
TestWithBrowser()
{
    Select (Query)
        SELECT [@], [Рег.No] FROM [Заказы]
        WHERE { r_Orders = Const(Object("dsDB", "Orders"));
                r_Orders.SetLine(is_query(, "@"));
                Return r_Orders.GetStatusAsNumber() >= osIntroduced; }
    End Select

    Browser = Object("FreeBrowser", Query, "Заказы");
    Browser.Run(TRUE);

    OrdersRefs = Object("Array");
    For (QueryLine = Browser.GetSelected(0);
         QueryLine != NULL_REF;
         QueryLine = Browser.GetSelected(QueryLine + 1))
        OrdersRefs.Add(Query.GetData("@", QueryLine));
    End For

    If (OrdersRefs.GetSize() > 0)
        is_transaction(1, "Обработка заказов");
        Loop (OrdersRefs)
            r_Orders = Object("dsDB", "Orders");
            r_Orders.SetLine(OrdersRefs.Get());
        End Loop
        is_transaction(-1);
    End If
}
```

### Шаблон серверного теста

```eme-l
Tests.TestUnit
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************
AAA = 0;

************************             Методы            ************************
Run()
{
    Params = Object("Parameters");
    Params.PutParam("Action", "Run");
    Params.PutParam("Class", "TestUnit");
    Params.PutParam("Method", "TestRun");
    Result = is_server_emel(1009, Params);
    Params.SetString(Result);
    ssSA = is_frombase64(Params.GetParam("Result", ""));
}

TestRun()
{
    is_catastrophic_log("Server test started");
    Return is_server();
}
```

---

## UI-тест интерфейса диалогов

**Запуск:** `Система → Ресурсы → Тестирование → Тест интерфейса`

Последовательно открывает все диалоги системы (~3 минуты). Не нажимает кнопки и не закрывает окна сообщений.

### `is_ui_test_mode()` — ключевая функция

```eme-l
Dialog_OnLoad()
{
    If (ContextFindParamsRec.GetNoOfLines() == 0 & ~is_ui_test_mode())
        If (is_message(tr("Внимание"), tr("Создать настройки поиска?"), "YESNO", "QUESTION") == "YES")
            CreateDefault();
        End If
    End If
}
```

### `is_lost_fields(Missing, TypeErrors, Dups)` — проверка потерянных полей

```eme-l
Missing = Object("Array")
TypeErrors = Object("Array")
Dups = Object("Array")
is_lost_fields(Missing, TypeErrors, Dups)

If (Missing.GetSize() > 0)
    is_error(1, "Обнаружены несоответствия метапроекта")
End If
```

| Параметр | Формат |
|----------|--------|
| `Missing` | `имя_класса\|имя_записи\|имя_метода\|имя_поля` |
| `TypeErrors` | `имя_класса\|имя_записи\|тип_метода\|тип_поля` |
| `Dups` | `имя_класса\|имя_метода` |

---

## Роботы-тестировщики (CIRobot)

**Запуск:** `Система → Ресурсы → Тестирование → Робот` (требуется монопольный режим)

| Тип | Что тестирует | Базовый класс |
|-----|---------------|---------------|
| **Классификаторы** | Склады, сектора, товары, транзакции | `GuideRobots` |
| **Операции** | Приход, отгрузка, перемещение, списание | `OperationRobots` |
| **Отчёты** | Печатные формы | `ReportsRobots` |

### Запуск робота через код

```eme-l
is_run_robot("RobotOpenDialogs", 1000, 600, 0)
```

| Параметр | Описание |
|----------|----------|
| `args[0]` | Имя робота: `"RobotOpenDialogs"`, `"RobotStaffPosition"`, `"RobotStaff"` |
| `args[1]` | Пошаговая задержка в мс (по умолчанию 500) |
| `args[2]` | Таймаут в секундах (по умолчанию 500) |
| `args[3]` | Признак скрытия диалога (по умолчанию 0) |

### Добавление нового робота

1. Создать класс
2. Реализовать метод `RobotBatchGo_RST(Robot AS "CIRobot")`
3. Прописать в конструкторе класса `AllRobots`

---

## Тесты целостности базы данных

### `is_run_dbtest` — полный тест БД

```eme-l
' Полный тест (физический + логический)
Result = is_run_dbtest()

' Только физический тест для записи 10
Result = is_run_dbtest(1, 10)
```

| Параметр | Описание |
|----------|----------|
| `args[0]` | `0` — физический + логический, `1` — только физический, `2` — только логический |
| `args[1]` | Номер записи (по умолчанию все) |
| `args[2]` | Номер поля (по умолчанию все) |

**Возвращает:** `Integer` — `0` = ошибок не обнаружено.

### `is_test_references_counter` — счётчики ссылок

```eme-l
' Проверить все записи
is_test_references_counter()

' Проверить диапазон 10-50
is_test_references_counter(10, 50)
```

---

## Тест виртуальной файловой системы (VFS)

```eme-l
' Тест целостности таблицы размещения
is_vfs_test()

' Запустить конкретный тест №5
is_vfs_test(5)
```

| Параметр | Описание |
|----------|----------|
| `args[0]` | Номер теста (по умолчанию — тест целостности таблицы размещения) |
| `args[1]` | Дополнительный параметр (по умолчанию 0) |
| `args[2]` | Предыдущее значение для проверки (по умолчанию 0) |

---

## Тест блочной записи

```eme-l
is_test_block_write(1, 1, 100, 10, headersArray, calcResults)
```

Тестирует режим пакетной (блочной) записи в БД — оптимизацию для массовых операций вставки.

---

## Тестирование запросов

```eme-l
If (is_trace_query())
    ' Запрос прошёл валидацию
End If
```

**Возвращает:** `Boolean` — `TRUE`, если запрос оттестирован.

---

## Отладочные is-функции для тестирования

| Функция | Назначение |
|---------|------------|
| `is_memory_leak(1024)` | Создать утечку памяти указанного размера для тестирования |
| `is_OLEAUT32_error()` | Генерация принудительного исключения OLEAUT32 |
| `is_test_email_message(...)` | Отправка email с выбором режима (тест почты) |
| `is_lost_fields(M, T, D)` | Проверка совместимости метапроекта с данными |

### Пример `is_test_email_message`

```eme-l
result = is_test_email_message(
    "test@local", "", "admin@local", "",
    "Тест", "Проверка связи", FALSE, "", FALSE
)
' result = 1 при успешной отправке
```

---

## Требования к коду (из EME_L_Examples.md)

> **11.** В коде должна быть встроена самодиагностика: проверка промежуточных результатов, читаемых из БД значений.
>
> **12.** Должны быть предусмотрены средства автотестирования (юнит-тесты).

---

## Полезные объекты для тестов

| Объект | Создание | Назначение |
|--------|----------|------------|
| `Console` | `Object("Console")` | Вывод текста в консоль |
| `Json` | `Object("Json", строка)` | Работа с JSON |
| `PropertyTree` | `Object("PropertyTree")` | Древовидная структура данных |
| `Query` | `Object("Query")` | SQL-запросы |
| `dsDB` | `Object("dsDB", "ИмяЗаписи")` | Записи базы данных |
| `ExternalDB` | `Object("ExternalDB")` | Подключение к внешней БД |
| `Array` | `Object("Array")` | Динамический массив |
| `Map` | `Object("Map")` | Ассоциативный массив |
| `System` | `Object("System")` | Системные функции |
| `File` | `Object("File", путь)` | Работа с файлами |
| `Automation` | `Object("Automation", ProgID)` | COM-автоматизация |
| `IClass` | `Object("IClass", имя, имя)` | Динамический вызов класса |

---

## Навигация по документации тестирования

- [Обзор тестирования](testing/EME_L_Testing_Index.md) — сводная таблица всех тестов
- [Юнит-тесты](testing/EME_L_Testing_Unit.md) — изолированные тесты объектов Json
- [ERP-тесты](testing/EME_L_Testing_ERP.md) — интеграционные тесты обмена с ERP
- [Самодиагностика](testing/EME_L_Testing_SelfDiagnostics.md) — система KPI, роботы, UI-тесты
- [Шаблоны тестов](testing/EME_L_Testing_Templates.md) — готовые заготовки
- [SQL-тесты](testing/sql-tests.md) — проверка целостности данных через SQL
- [Тесты записей и БД](testing/record-tests.md) — CEMERec, CEMESkip, DBWatch, транзакции
- [Интеграционные тесты](testing/integration-tests.md) — HTTP, RPC, XML, внешние БД
- [Роботы-тестировщики](testing/robot-tests.md) — автоматические тесты классификаторов и операций
- [Тесты браузеров](testing/browser-tests.md) — браузеры терминалов TSD
- [Прочие тесты](testing/misc-tests.md) — изображения, ошибки, ВЕТИС, EANCOM, Диадок
