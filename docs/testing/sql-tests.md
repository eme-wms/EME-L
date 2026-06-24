> Синонимы: SQL-тесты, запросы, дубли ключей, DupKeys, ExternalDB, TestWITH, хранимые процедуры, блокировки, целостность данных

## SQL-тесты в EME-L

SQL-тесты — это классы, выполняющие проверки целостности данных в базе данных через SQL-запросы. Они располагаются в директории `Запросы/` и запускаются вручную (кнопкой) разработчиком или администратором базы данных.

В отличие от юнит-тестов, SQL-тесты:
- Работают непосредственно с данными базы данных через SQL
- Проверяют целостность и корректность структуры данных
- Не используют `CHECK_EQ` — результат проверяется по количеству найденных строк
- Часто используют для диагностики проблем в продуктивной среде

---

## Дубли ключей и уникальность данных

### `Tests.DupKeys` — дублирующиеся ключевые значения

Проверяет наличие дублирующихся значений в ключевых полях записей. Обнаружение дублей ключей критично для корректности работы индексов и ссылочной целостности.

**Запуск:** Класс в директории `Запросы/`

```EME-L
Tests.DupKeys
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    Select (Query)
        SELECT
            [Поле],
            COUNT(*) AS [Количество]
        FROM
            [Таблица]
        GROUP BY
            [Поле]
        HAVING
            {is_query(,"Количество") > 1}
    End Select

    If (Query.GetNoOfLines() > 0)
        is_message("DupKeys", "Найдено дубликатов ключей: " + Query.GetNoOfLines(), "OK", "EXCLAMATION");
    Else
        is_message("DupKeys", "Дубликатов ключей не найдено", "OK", "INFORMATION");
    End If
}
```

---

### `Tests.DupLotID` — дубли `LotID`

Проверяет дублирование идентификаторов партий (`LotID`) в системе. Некорректные `LotID` могут привести к ошибкам при трассировке товаров.

```EME-L
'Проверка дублей LotID в записи регистров'
Select (Query)
    SELECT
        [LotID],
        COUNT(*) AS [Количество]
    FROM
        [Регистры]
    WHERE
        [LotID] IS NOT NULL
    GROUP BY
        [LotID]
    HAVING
        {is_query(,"Количество") > 1}
End Select
```

---

### `Tests.DupLotNo` — дубли номеров партий

Проверяет дублирование номеров партий (`LotNo`). Повторяющиеся номера партий у разных товаров или поставщиков могут вызвать путаницу при приёмке и отгрузке.

```EME-L
'Проверка дублей номеров партий'
Select (Query)
    SELECT
        [LotNo],
        [VendorRef],
        COUNT(*) AS [Количество]
    FROM
        [Регистры]
    GROUP BY
        [LotNo],
        [VendorRef]
    HAVING
        {is_query(,"Количество") > 1}
End Select
```

---

### `Tests.TestDocLinesDups` — дубли строк документов

Проверяет наличие дублирующихся строк в табличных частях документов (строки с одинаковыми `GoodsItemRef`, `LotRef` и т.д. в одном документе).

```EME-L
'Поиск дублирующихся строк в документах'
Select (Query)
    SELECT
        [DocRef],
        [GoodsItemRef],
        [LotRef],
        COUNT(*) AS [Количество]
    FROM
        [СтрокиДокумента]
    GROUP BY
        [DocRef],
        [GoodsItemRef],
        [LotRef]
    HAVING
        {is_query(,"Количество") > 1}
End Select

If (Query.GetNoOfLines() > 0)
    is_message("TestDocLinesDups", "Дубли строк в документах: " + Query.GetNoOfLines(), "OK", "EXCLAMATION");
End If
```

---

### `Tests.TestRegistersDups` — дубли регистров

Проверяет дублирование записей в регистрах учёта. Дубли в регистрах могут привести к некорректным остаткам.

```EME-L
'Проверка дублей в регистрах'
Select (Query)
    SELECT
        [GoodsItemRef],
        [CellRef],
        [LotRef],
        COUNT(*) AS [Количество]
    FROM
        [Регистры]
    GROUP BY
        [GoodsItemRef],
        [CellRef],
        [LotRef]
    HAVING
        {is_query(,"Количество") > 1}
End Select
```

---

## Блокировки и параллельный доступ

### `Tests.TestSpLock` — блокировки хранимых процедур

Проверяет корректность работы блокировок хранимых процедур при параллельном доступе. Выявляет взаимные блокировки (deadlocks) и таймауты.

```EME-L
'Тест блокировок хранимых процедур'
Tests.TestSpLock
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'Запускаем несколько параллельных сессий с блокировками'
    Result = is_message("TestSpLock", "Запустить тест блокировок?", "YESNO", "QUESTION");
    If (Result == "YES")
        'Выполняем тестовую хранимую процедуру с блокировкой'
        is_message("TestSpLock", "Тест блокировок запущен. Проверьте логи.", "OK", "INFORMATION");
    End If
}
```

**Паттерн проверки блокировок:**
- Запуск параллельных транзакций
- Установка блокировок на одни и те же записи
- Проверка отсутствия взаимных блокировок
- Измерение времени ожидания

---

## Хранимые процедуры

### `Tests.TestStoredProcedure` — тест хранимых процедур

Выполняет тестовые вызовы хранимых процедур базы данных и проверяет корректность их работы.

```EME-L
'Тест хранимых процедур'
Tests.TestStoredProcedure
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'Выполняем тестовую хранимую процедуру'
    Select (Query)
        EXEC [TestStoredProcedure] @Param1 = {Param1}, @Param2 = {Param2}
    End Select

    If (Query.GetNoOfLines() > 0)
        is_message("TestStoredProcedure", "Хранимая процедура выполнена. Строк: " + Query.GetNoOfLines(), "OK", "INFORMATION");
    End If
}
```

---

## CTE-запросы и сложный SQL

### `Tests.TestWITH` — тест CTE (Common Table Expressions)

Проверяет корректность работы CTE-запросов (SQL `WITH`). CTE используются для построения рекурсивных запросов и сложных выборок.

```EME-L
'Тест CTE-запросов'
Tests.TestWITH
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'Пример CTE-запроса для иерархической выборки'
    Select (Query)
        WITH [Hierarchy] AS (
            SELECT
                [@],
                [ParentRef],
                [Name],
                0 AS [Level]
            FROM
                [Классификатор]
            WHERE
                [ParentRef] IS NULL

            UNION ALL

            SELECT
                C.[@],
                C.[ParentRef],
                C.[Name],
                H.[Level] + 1
            FROM
                [Классификатор] C
            INNER JOIN
                [Hierarchy] H ON C.[ParentRef] = H.[@]
        )
        SELECT * FROM [Hierarchy]
    End Select

    is_message("TestWITH", "CTE-запрос выполнен. Строк: " + Query.GetNoOfLines(), "OK", "INFORMATION");
}
```

---

## GUID и идентификаторы

### `Tests.TestGUID` — проверка уникальности GUID

Проверяет корректность генерации и уникальность глобальных уникальных идентификаторов (GUID) в системе.

```EME-L
'Проверка GUID'
Tests.TestGUID
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'Генерируем GUID и проверяем его формат'
    Guid = is_new_guid();
    is_message("TestGUID", "Сгенерирован GUID: " + Guid, "OK", "INFORMATION");

    'Проверяем уникальность GUID в таблице'
    Select (Query)
        SELECT [GUID], COUNT(*) AS [Количество]
        FROM [Таблица]
        WHERE [GUID] IS NOT NULL
        GROUP BY [GUID]
        HAVING COUNT(*) > 1
    End Select

    If (Query.GetNoOfLines() > 0)
        is_message("TestGUID", "Найдены дубли GUID: " + Query.GetNoOfLines(), "OK", "EXCLAMATION");
    Else
        is_message("TestGUID", "Дублей GUID не найдено", "OK", "INFORMATION");
    End If
}
```

---

## Внешние базы данных

### `Tests.ExternalDB` — тест подключения к внешним БД

Проверяет подключение к внешним базам данных через ODBC. Используется для диагностики проблем с интеграцией.

```EME-L
'Тест подключения к внешней базе данных'
Tests.ExternalDB
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    ExternalDB = Object("ExternalDB");

    'Подключаемся к внешней БД'
    Error = ExternalDB.Connect("ODBC:Driver={SQL Server};Server=localhost;Database=Test;Uid=sa;Pwd=;");

    If (Error != "")
        is_message("ExternalDB", "Ошибка подключения: " + Error, "OK", "STOP");
        Return;
    End If

    'Выполняем тестовый запрос'
    Error = ExternalDB.Execute("SELECT 1 AS Test", Query);
    If (Error != "")
        is_message("ExternalDB", "Ошибка запроса: " + Error, "OK", "STOP");
    Else
        is_message("ExternalDB", "Подключение успешно. Строк: " + Query.GetNoOfLines(), "OK", "INFORMATION");
    End If

    ExternalDB.Disconnect();
}
```

---

### `Tests.ExternalDBOnFly` — динамическое подключение к внешним БД

Проверяет возможность создания подключений к внешним БД "на лету" без предварительной настройки DSN.

```EME-L
'Динамическое подключение к внешней БД'
Tests.ExternalDBOnFly
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    ExternalDB = Object("ExternalDB");

    'Формируем строку подключения динамически'
    ConnStr = "ODBC:Driver={PostgreSQL ODBC Driver(ANSI)};Server=" + ServerIP;
    ConnStr = ConnStr + ";Port=" + Port + ";Database=" + DBName;
    ConnStr = ConnStr + ";Uid=" + User + ";Pwd=" + Password + ";";

    Error = ExternalDB.Connect(ConnStr);

    If (Error == "")
        is_message("ExternalDBOnFly", "Динамическое подключение успешно", "OK", "INFORMATION");
        ExternalDB.Disconnect();
    Else
        is_message("ExternalDBOnFly", "Ошибка: " + Error, "OK", "STOP");
    End If
}
```

---

### `Tests.ExternalDBOracle` — тест подключения к Oracle

Проверяет подключение к базам данных Oracle. Специфичен для интеграций с Oracle-based ERP-системами.

```EME-L
'Подключение к Oracle'
Tests.ExternalDBOracle
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    ExternalDB = Object("ExternalDB");
    Error = ExternalDB.Connect("ODBC:Driver={Oracle in OraClient11g_home1};Dbq=ORCL;Uid=system;Pwd=;");

    If (Error == "")
        is_message("ExternalDBOracle", "Подключение к Oracle успешно", "OK", "INFORMATION");
        ExternalDB.Disconnect();
    Else
        is_message("ExternalDBOracle", "Ошибка Oracle: " + Error, "OK", "STOP");
    End If
}
```

---

### `Tests.SqlServer` — тесты под SQL Server

Проверяет специфические для SQL Server функции и синтаксис. Используется при миграции или работе с SQL Server в качестве бэкенда.

```EME-L
'Тест специфики SQL Server'
Tests.SqlServer
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'Тест SQL Server-специфичного синтаксиса'
    Select (Query)
        SELECT TOP 10
            [@],
            [Name],
            GETDATE() AS [CurrentDate]
        FROM
            [СистемнаяЗапись]
    End Select

    is_message("SqlServer", "SQL Server-запрос выполнен. Строк: " + Query.GetNoOfLines(), "OK", "INFORMATION");
}
```

---

## Экспорт и импорт

### `Tests.TestExpImp` — тест экспорта и импорта данных

Проверяет корректность операций экспорта и импорта данных между системами.

```EME-L
'Тест экспорта и импорта'
Tests.TestExpImp
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'Экспортируем данные в файл'
    File = Object("File", "D:\\Temp\\TestExport.txt");
    File.CreateFile();

    r_Record = Object("dsDB", "GoodsItem");
    r_Record.SetSkipMode();

    Loop (r_Record)
        Line = r_Record.GetCode() + ";" + r_Record.GetName() + "\n";
        File.PutFileData(Line);
    End Loop

    File.CloseFile();

    'Импортируем данные обратно'
    File2 = Object("File", "D:\\Temp\\TestExport.txt");
    Data = File2.GetData().AsString();

    is_message("TestExpImp", "Экспортировано и импортировано. Размер: " + is_length(Data), "OK", "INFORMATION");
}
```

---

## Сводная таблица SQL-тестов

| Класс / Файл | Что проверяет | Тип |
|--------------|--------------|-----|
| `Tests.DupKeys` | Дубли ключевых значений | Целостность |
| `Tests.DupLotID` | Дубли идентификаторов партий | Целостность |
| `Tests.DupLotNo` | Дубли номеров партий | Целостность |
| `Tests.TestDocLinesDups` | Дубли строк документов | Целостность |
| `Tests.TestRegistersDups` | Дубли в регистрах | Целостность |
| `Tests.TestSpLock` | Блокировки хранимых процедур | Параллельный доступ |
| `Tests.TestStoredProcedure` | Корректность хранимых процедур | Функциональность |
| `Tests.TestWITH` | CTE-запросы | SQL-функциональность |
| `Tests.TestGUID` | Уникальность GUID | Целостность |
| `Tests.ExternalDB` | Подключение к внешним БД (ODBC) | Интеграция |
| `Tests.ExternalDBOnFly` | Динамическое подключение | Интеграция |
| `Tests.ExternalDBOracle` | Подключение к Oracle | Интеграция |
| `Tests.SqlServer` | Специфика SQL Server | Совместимость |
| `Tests.TestExpImp` | Экспорт и импорт данных | Обмен данными |
| `ERP.SQLTest` | SQL-тесты ERP-подсистемы | Интеграция с ERP |
| `ERP.AnalyticsTest` | Аналитические SQL-запросы | Аналитика |
| `Tests.TerminalClassesCreateIClassObjectCode` | Создание объектов терминалов | Терминалы |
| `SQL_Test` | Общий SQL-тест | Общий |

---

## Паттерн создания SQL-теста

### Шаблон SQL-теста

```EME-L
Tests.MySqlTest
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    Select (Query)
        SELECT
            [Поле1],
            [Поле2],
            COUNT(*) AS [Количество]
        FROM
            [Таблица]
        WHERE
            [Условие]
        GROUP BY
            [Поле1],
            [Поле2]
        HAVING
            {is_query(,"Количество") > 1}
    End Select

    If (Query.GetNoOfLines() > 0)
        is_message("MySqlTest", "Найдено проблем: " + Query.GetNoOfLines(), "OK", "EXCLAMATION");
    Else
        is_message("MySqlTest", "Проблем не найдено", "OK", "INFORMATION");
    End If
}
```

### Контрольный список SQL-теста

- [ ] Класс находится в директории `Запросы/`
- [ ] SQL-запрос корректно сформулирован и проверен
- [ ] Используется `HAVING` для фильтрации агрегатов
- [ ] Результат проверяется через `Query.GetNoOfLines()`
- [ ] Есть информативное сообщение об ошибке
- [ ] Тест можно запустить вручную через кнопку
