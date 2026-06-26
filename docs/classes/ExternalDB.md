# Класс ExternalDB

> Синонимы для поиска: CIExternalDB ExternalDB класс EME-L, внешняя база данных EME-L, external database EME-L, ODBC EME-L, .NET провайдер EME-L, PSQL PostgreSQL EME-L, MetC_CLR.dll EME-L, Connect Disconnect IsConnected Execute ExecuteWithTopLimit CreateTable PrepareCreateTableScript AddColumns AlterColumns InsertLine InsertLines PrepareInsertLineScript PrepareInsertLinesScript UpdateLine PrepareUpdateLineScript BulkCopy EnumTables EnumColumns BeginTransaction CommitTransaction RollbackTransaction IsAutoCommit GetIsolationLevel SetIsolationLevel GetInsertBatchSize SetInsertBatchSize GetQueryTimeout SetQueryTimeout GetRowCount GetDBMSName GetDBMSVersion, SQL-запрос внешняя БД EME-L, массовое копирование EME-L, транзакции внешняя БД EME-L, уровни изоляции EME-L, репликация EME-L, интеграция SQL EME-L

Класс `ExternalDB` (C++ `CIExternalDB`) в языке EME-L обеспечивает подключение к внешним базам данных и выполнение SQL-запросов через драйверы ODBC и .NET (а также PSQL/PostgreSQL при условной компиляции с включённым `POSTGRESQL_IS_AVAILABLE`). Класс ExternalDB в языке EME-L предоставляет методы для выполнения запросов, создания и изменения структуры таблиц, вставки и обновления строк, массового копирования данных, управления транзакциями, перечисления таблиц и колонок, а также получения метаданных СУБД.

Класс ExternalDB в системе EME.WMS используется для интеграции с внешними СУБД — импорта товаров и документов из сторонних SQL-баз, выгрузки документов во внешние учётные системы, синхронизации остатков и номенклатуры, репликации данных между EME.WMS и ERP-системами.

Объект класса `ExternalDB` в языке EME-L создаётся через `Object()` и не привязан к контексту диалога или браузера — это самостоятельный клиентский интерфейс для работы с внешней БД. Соединение с внешней БД **не** устанавливается автоматически при конструировании — для подключения необходимо вызвать метод `Connect`. После завершения работы соединение следует разорвать методом `Disconnect`.

## Создание объекта класса ExternalDB

```EME-L
'Создать клиент внешней БД — подключение ещё не установлено'
objExternalDB = Object("ExternalDB");
```

Конструктор без аргументов: `ExternalDB()`. Возвращает ссылку на созданный объект ExternalDB в языке EME-L. Член класса, хранящий клиент ExternalDB, рекомендуется называть с префиксом `m_` (`m_ExternalDB`) и инициализировать в конструкторе содержащего класса; как локальную переменную — `objExternalDB`.

## Подключение и проверка состояния

Методы класса ExternalDB в языке EME-L для управления подключением к внешней базе данных. Строка подключения задаётся конечным пользователем, поэтому ошибки в ней возвращаются как текст, а не генерируют исключение как программная ошибка.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `Connect(arg0)` | arg0: String (строка подключения) | String | Устанавливает соединение по строке подключения. Формат: `"<ТИП>:<параметры>"`. Поддерживаемые типы: `"ODBC"`, `".NET"` (требуется MetC_CLR.dll), `"PSQL"` (PostgreSQL, доступен при включённой условной компиляции `POSTGRESQL_IS_AVAILABLE`). Типы `"OLEDB"` и `"QT"` не поддерживаются и возвращают ошибку. Пустая строка — успех, иначе текст ошибки. Если клиент уже подключен — ошибка. При ошибке подключения освобождает созданный драйвер. |
| `Disconnect()` | — | String | Разрывает соединение и освобождает драйвер. Если клиент не подключён, генерирует ошибку. Пустая строка — успех, иначе текст ошибки. |
| `IsConnected()` | — | Boolean | TRUE — активное соединение установлено, иначе FALSE. |

## Выполнение SQL-запросов

Методы класса ExternalDB в языке EME-L для выполнения SQL-запросов к внешней базе данных. Результаты SELECT помещаются в объект Query, переданный по ссылке. Запрос можно передать как строкой, так и объектом String — извлекается внутреннее значение. Методы возвращают пустую строку при успехе, иначе текст ошибки.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `Execute(arg0, arg1)` | arg0: String или String-объект (SQL), arg1: Query (по ссылке) | String | Выполняет SQL-запрос и помещает результат в объект Query. |
| `Execute(arg0, arg1, arg2)` | arg0: SQL, arg1: Query, arg2: String (алиасы) | String | То же с переименованием колонок SQL в поля Query. Формат `arg2`: `"<ИмяКолонки1>=<ИмяПоля1>\n<ИмяКолонки2>=<ИмяПоля2>\n..."`. Позволяет заполнить существующий Query (с уже созданными полями) данными с сервера, сопоставляя SQL-колонки с полями Query. |
| `ExecuteWithTopLimit(arg0, arg1, arg2)` | arg0: SQL, arg1: Query, arg2: Integer (лимит строк) | String | Выполняет SQL-запрос с ограничением максимального числа возвращаемых строк. |
| `GetRowCount()` | — | Integer | Количество строк, затронутых последней инструкцией UPDATE, INSERT или DELETE. |

## Создание и изменение структуры таблиц

Методы класса ExternalDB в языке EME-L для управления структурой таблиц во внешней базе данных. Метод `CreateTable` создаёт новую таблицу на основе структуры полей объекта Query — поля Query становятся колонками таблицы с соответствующими типами. Методы `AddColumns` и `AlterColumns` соответственно добавляют новые и изменяют существующие колонки. Методы `PrepareCreateTableScript` формируют текст SQL-команды `CREATE TABLE` без выполнения — для отложенного применения в пакетной транзакции или после модификации текста.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `CreateTable(arg0, arg1)` | arg0: String (имя таблицы), arg1: Query | String | Создаёт таблицу по шаблону полей Query. Пустая строка — успех, иначе текст ошибки. Если Query не задан — ошибка. |
| `CreateTable(arg0, arg1, arg2)` | arg0: String, arg1: Query, arg2: String или Array (первичный ключ) | String | То же с указанием первичного ключа. arg2 — имя одной колонки (строка) или массив имён колонок для составного ключа. |
| `PrepareCreateTableScript(arg0, arg1, arg2)` | arg0: String (по ссылке), arg1: String (таблица), arg2: Query | String | Формирует SQL-скрипт CREATE TABLE и добавляет его в объект String. Скрипт не выполняется автоматически. |
| `PrepareCreateTableScript(arg0, arg1, arg2, arg3)` | arg0: String, arg1: String, arg2: Query, arg3: String или Array (первичный ключ) | String | То же с указанием первичного ключа (строка или массив имён колонок). |
| `AddColumns(arg0, arg1)` | arg0: String (таблица), arg1: Query | String | Добавляет новые колонки в существующую таблицу по шаблону полей Query. Если Query не задан — ошибка. |
| `AlterColumns(arg0, arg1)` | arg0: String (таблица), arg1: Query | String | Изменяет структуру существующих колонок по шаблону полей Query. Если Query не задан — ошибка. |

## Вставка строк

Методы класса ExternalDB в языке EME-L для вставки данных из объекта Query в таблицы внешней базы данных. Метод `InsertLine` вставляет одну строку по индексу (0-based), `InsertLines` — набор строк по массиву индексов или все строки сразу. Методы `Prepare*Script` формируют SQL-скрипты INSERT без выполнения — для пакетного применения или логирования. Для массовой вставки предпочтительнее `InsertLines` (без массива индексов вставляет все строки) с предварительно настроенным `SetInsertBatchSize` — это объединяет INSERT-команды в пакеты и существенно ускоряет загрузку по сравнению с построчной `InsertLine`.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `InsertLine(arg0, arg1, arg2)` | arg0: String (таблица), arg1: Query, arg2: Integer (индекс строки, 0-based) | String | Вставляет одну строку Query в таблицу. Если индекс выходит за пределы Query — ошибка. |
| `InsertLines(arg0, arg1, arg2)` | arg0: String, arg1: Query, arg2: Array (индексы строк) | String | Вставляет указанные строки Query в таблицу. Если любой индекс массива выходит за пределы Query — ошибка. |
| `InsertLines(arg0, arg1)` | arg0: String, arg1: Query | String | Вставляет все строки Query в таблицу. |
| `PrepareInsertLineScript(arg0, arg1, arg2, arg3)` | arg0: String (по ссылке), arg1: String (таблица), arg2: Query, arg3: Integer (индекс строки) | String | Формирует SQL-скрипт INSERT для одной строки и добавляет его в объект String. Если индекс выходит за пределы Query — ошибка. |
| `PrepareInsertLinesScript(arg0, arg1, arg2, arg3)` | arg0: String, arg1: String, arg2: Query, arg3: Array (индексы строк) | String | Формирует SQL-скрипт INSERT для указанных строк и добавляет его в объект String. |
| `PrepareInsertLinesScript(arg0, arg1, arg2)` | arg0: String, arg1: String, arg2: Query | String | Формирует SQL-скрипт INSERT для всех строк Query и добавляет его в объект String. |

## Обновление строк

Метод класса ExternalDB в языке EME-L для обновления строк в таблице внешней базы данных. Обновление выполняется по ключевым колонкам, по которым ищется обновляемая строка.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `UpdateLine(arg0, arg1, arg2, arg3)` | arg0: String (таблица), arg1: Query, arg2: Integer (индекс строки), arg3: String или Array (ключевые колонки) | String | Обновляет одну строку в таблице данными из Query. Поиск строки выполняется по ключевым колонкам arg3 (имя одной колонки или массив имён). Если индекс выходит за пределы Query или ключевые колонки не указаны — ошибка. |
| `PrepareUpdateLineScript(arg0, arg1, arg2, arg3, arg4)` | arg0: String (по ссылке), arg1: String (таблица), arg2: Query, arg3: Integer (индекс строки), arg4: String или Array (ключевые колонки) | String | Формирует SQL-скрипт UPDATE для одной строки и добавляет его в объект String. |

## Массовое копирование

Метод `BulkCopy` класса ExternalDB в языке EME-L выполняет серверное массовое копирование данных из объекта Query в таблицу внешней базы данных. Для ODBC-драйвера требуется настройка директории для CSV-файлов (модули — Предпочтения — Репликация — Директория csv-файлов); указанная директория должна быть доступна и клиенту, и серверу СУБД. Если директория не задана, используется стандартная папка Windows `C:\Users\Public`.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `BulkCopy(arg0, arg1)` | arg0: String (таблица), arg1: Query | String | Копирует все данные из Query в таблицу внешней БД. |
| `BulkCopy(arg0, arg1, arg2)` | arg0: String, arg1: Query, arg2: Array (имена полей) | String | Копирует указанные колонки из Query в таблицу внешней БД. |

## Перечисление таблиц и колонок

Методы класса ExternalDB в языке EME-L для получения метаданных о структуре внешней базы данных — списка таблиц и колонок.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `EnumTables(arg0)` | arg0: Query (по ссылке) | String | Заполняет Query списком таблиц внешней БД. Структура результата: `TABLE_NAME : TEXT[64]` — имя таблицы; `TABLE_TYPE : TEXT[16]` — тип (`"TABLE"`, `"VIEW"`, `"SYSTEM TABLE"`). |
| `EnumColumns(arg0, arg1)` | arg0: String (таблица), arg1: Query (по ссылке) | String | Заполняет Query списком колонок указанной таблицы. Структура результата: `COLUMN_NAME : TEXT[64]` — имя колонки; `COLUMN_TYPE : TEXT[32]` — тип (`"varchar(N)"`, `"integer"`, `"float"`, `"date"`, `"time"`, `"datetime"`). |

## Транзакции и уровни изоляции

Все методы транзакций класса ExternalDB в языке EME-L требуют активного подключения к внешней базе данных. Уровни изоляции передаются строкой: `"READ_UNCOMMITTED"` — чтение незафиксированных данных, `"READ_COMMITTED"` — чтение зафиксированных данных, `"REPEATABLE_READ"` — повторяемое чтение, `"SERIALIZABLE"` — сериализуемость.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `BeginTransaction(arg0)` | arg0: String (уровень изоляции) | String | Начинает новую транзакцию с указанным уровнем изоляции. |
| `CommitTransaction()` | — | String | Фиксирует текущую транзакцию, сохраняя все изменения. |
| `RollbackTransaction()` | — | String | Отменяет текущую транзакцию, откатывая все изменения. |
| `IsAutoCommit()` | — | Boolean | TRUE — внешняя БД работает в режиме auto-commit (каждая команда фиксируется автоматически без явного вызова `CommitTransaction`), иначе FALSE. |
| `GetIsolationLevel()` | — | String | Текущий уровень изоляции транзакций для соединения. Возвращает одно из значений `"READ_UNCOMMITTED"`, `"READ_COMMITTED"`, `"REPEATABLE_READ"`, `"SERIALIZABLE"`, либо пустое значение, если уровень неизвестен. |
| `SetIsolationLevel(arg0)` | arg0: String (уровень изоляции) | String | Устанавливает уровень изоляции транзакций для соединения. |

## Параметры выполнения запросов

Методы класса ExternalDB в языке EME-L для настройки параметров выполнения запросов и массовой вставки.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetInsertBatchSize()` | — | Integer | Размер пакета команд INSERT, используемый при массовой вставке строк методом `InsertLines`. Размер пакета определяет, сколько строк INSERT объединяется в одну пакетную команду. |
| `SetInsertBatchSize(arg0)` | arg0: Integer | Empty | Устанавливает размер пакета команд INSERT для массовой вставки методом `InsertLines`. |
| `GetQueryTimeout()` | — | Integer | Текущий таймаут выполнения запроса к внешней БД в секундах. Таймаут определяет максимальное время ожидания выполнения команды перед прерыванием. |
| `SetQueryTimeout(arg0)` | arg0: Integer (секунды) | Empty | Устанавливает таймаут выполнения запроса к внешней БД. |

## Метаданные СУБД

Методы класса ExternalDB в языке EME-L для получения информации о системе управления базой данных внешнего источника.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetDBMSName()` | — | String | Наименование системы управления базами данных (СУБД) внешней базы данных. |
| `GetDBMSVersion()` | — | String | Версия системы управления базами данных (СУБД) внешней базы данных. |

## Примеры

Подключение к внешней БД через ODBC и выполнение SELECT-запроса в объект Query — метод Execute класса ExternalDB в языке EME-L помещает результаты SQL в Query, который затем перебирается циклом Loop:

```EME-L
objExternalDB = Object("ExternalDB");
Error = objExternalDB.Connect("ODBC:Dsn=MyDsn;Uid=user;Pwd=pass;");
If (Error != "")
    is_message(tr("Ошибка"), Error, "OK", "STOP");
    Return;
End If

'Выполнить SELECT и поместить результат в Query'
Query = Object("Query");
Query.Create();
Error = objExternalDB.Execute("select * from Customers", Query);
If (Error == "")
    'перебор строк результата через Loop'
    Loop (Query)
        Name = Query.GetName();
    End Loop
End If

objExternalDB.Disconnect();
```

Создание таблицы по структуре Query с составным первичным ключом и вставка всех строк в транзакции — типовой паттерн миграции данных в системе EME.WMS:

```EME-L
'Открыть транзакцию с уровнем изоляции READ_COMMITTED'
Error = objExternalDB.BeginTransaction("READ_COMMITTED");
If (Error != "")
    Return Error;
End If

'Создать таблицу по структуре Query с составным первичным ключом'
arrKeys = Object("Array");
arrKeys.Add("WH_ID");
arrKeys.Add("OWNER_ID");
Error = objExternalDB.CreateTable("ext_warehouse", SrcQuery, arrKeys);

'Вставить все строки Query'
If (Error == "")
    Error = objExternalDB.InsertLines("ext_warehouse", SrcQuery);
End If

If (Error == "")
    objExternalDB.CommitTransaction();
Else
    objExternalDB.RollbackTransaction();
End If
```

Передача объекта `ExternalDB` во вспомогательные функции через `AsExternal` — в системе EME.WMS типовой паттерн интеграции: объект подключения создаётся один раз и передаётся в функции-обёртки `Connect` и `Execute`, чтобы не открывать подключение повторно:

```EME-L
'передать ExternalDB как внешний объект в функцию Connect и Execute'
Connect(ExternalDB AS "ExternalDB")
{
    Error = ExternalDB.Connect(
        "ODBC:Dsn=" + dsDIALOG.GetName() + ";Uid=" + dsDIALOG.GetLogin() + ";Pwd=" + dsDIALOG.GetPassword() + ";");
    If (Error != "")
        is_message(tr("ВНИМАНИЕ!!!"), tr("Не удалось подключиться:\n\n") + Error, "OK", "STOP");
        Return FALSE;
    End If
    Return TRUE;
}
```

Выполнение SQL-запроса UPDATE без возврата данных — метод Execute класса ExternalDB в языке EME-L выполняет инструкции модификации данных и возвращает пустую строку при успехе. Пример из реального импорта системы Incity:

```EME-L
objExternalDB = Object("ExternalDB");
objExternalDB.Connect("ODBC:Dsn=incity;Uid=ExTest;Pwd=ExTest99;");
resultQuery = Object("Query");
objExternalDB.Execute("update IFC_ExchangeQueueImport set eqi_TypeID = 1 where eqi_ID = 845428", resultQuery);
objExternalDB.Disconnect();
```

Перечисление таблиц внешней БД — метод EnumTables класса ExternalDB в языке EME-L заполняет Query списком таблиц с их типами:

```EME-L
objExternalDB = Object("ExternalDB");
objExternalDB.Connect("ODBC:Dsn=MyDsn;Uid=user;Pwd=pass;");
TablesQuery = Object("Query");
TablesQuery.Create();
objExternalDB.EnumTables(TablesQuery);
Loop (TablesQuery)
    TableName = TablesQuery.GetData("TABLE_NAME");
    TableType = TablesQuery.GetData("TABLE_TYPE");
End Loop
objExternalDB.Disconnect();
```

## См. также

- [Класс Query и шаблоны API](./EME_L_Advanced_API_and_Patterns.md) — объект Query, в который метод `Execute` класса ExternalDB помещает результаты SQL-запроса; также используется как шаблон колонок для `CreateTable` и источник данных для `InsertLine`/`InsertLines`/`BulkCopy`.
- [База данных](./EME_L_Database.md) — работа с внутренней БД EME.L через `CEMERec`, `is_*`-функции и транзакции `is_transaction`; внешние подключения ExternalDB дополняют эти средства интеграцией со сторонними СУБД.
