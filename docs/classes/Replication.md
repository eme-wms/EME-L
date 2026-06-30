# Класс Replication

> Синонимы для поиска: CIReplication Replication класс EME-L, репликация данных EME в SQL EME-L, выгрузка записей EME в SQL-базу EME-L, экспорт данных EME.WMS во внешнюю базу EME-L, SetDBMSType GetSqlObjectName GetTableName GetConstraintName GetColumnName MarkupTableQuery GetInsertLinesScript BulkCopy ExpImpBulkCopy QueryCopy EnumTables EnumColumns EnumFlags GetViewName GetViewScript GetDupKeys ResolveDupKeys, PostgreSQL репликация EME-L, GET_BIT EME-L, varchar(max) EME-L, LineKey EME-L, первичный ключ _pkey EME-L, дубликаты INT64 ключей EME-L, BulkCopy ExternalDB EME-L, CREATE VIEW EME-L, INSERT INTO EME-L, диалект СУБД репликация EME-L

## Класс Replication — описание и назначение

Класс `Replication` (C++ `CIReplication`) в языке EME-L — класс для репликации данных записей EME в сторонние SQL-базы. В языке EME-L класс Replication предоставляет полный набор операций для формирования имён SQL-объектов (таблиц, колонок, ограничений, представлений) по записям и полям EME, генерации SQL-скриптов создания представлений и вставки строк, прямого массового копирования данных через `ExternalDB`, перечисления метаданных схемы, поиска и устранения дубликатов первичных ключей.

Класс Replication в системе EME.WMS учитывает диалект целевой СУБД (Microsoft SQL Server, PostgreSQL, Oracle, MySQL): для PostgreSQL имена обрезаются по числу байт в кодировке UTF-8, а битовые поля представляются через `GET_BIT`; для прочих СУБД используется побитовая маска через `CAST(... AS BIT)`. Ссылочные поля экспортируются как 64-битные целочисленные ключи (`bigint`); поля переменной длины — как `varchar(max)`.

Используется в прикладных скриптах EME-L для организации выгрузки данных во внешние хранилища, интеграции с ERP-системами, построения зеркал данных EME в SQL-базах и диагностики состояния первичных ключей. Класс Replication в языке EME-L работает в паре с классом [ExternalDB](./ExternalDB.md) — ExternalDB устанавливает соединение с SQL-базой и выполняет запросы, а Replication готовит имена, скрипты и данные для выгрузки.

Объект класса `Replication` в языке EME-L создаётся через `Object()` без аргументов и не привязан к контексту диалога или браузера — это самостоятельный инструмент репликации. Перед вызовом методов формирования имён и скриптов рекомендуется задать тип целевой СУБД методом `SetDBMSType`, чтобы обеспечить корректную транслитерацию и обрезку имён. Класс Replication кэширует метаданные записи: при первом обращении к записи разбирается её структура (поля, типы, соответствие столбцам представления), при последующих вызовах для той же записи кэш используется повторно.

### Создание объекта класса Replication

Класс `Replication` в языке EME-L имеет один конструктор — без аргументов.

```EME-L
'Создать объект репликации данных EME в SQL'
objReplication = Object("Replication");
```

Конструктор `Replication()` возвращает объект с типом СУБД по умолчанию `dbmsUnknown`. Пока тип не задан явно через `SetDBMSType`, методы формирования имён работают по общим правилам — без UTF-8 обрезки и без `GET_BIT`. Член класса, хранящий объект Replication, рекомендуется называть с префиксом `m_` (`m_Replication`) и инициализировать в конструкторе содержащего класса; как локальную переменную — `objReplication`.

### Настройка целевой СУБД — SetDBMSType

Метод класса Replication в языке EME-L для настройки диалекта целевой базы данных. Влияет на все последующие операции формирования имён, скриптов и представления битовых полей.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetDBMSType(arg0)` | arg0: String (имя СУБД) | Empty | Устанавливает тип СУБД для репликации. Допустимые значения: `"SQLServer"` (Microsoft SQL Server), `"PostgreSQL"` (PostgreSQL), `"Oracle"` (Oracle Database), `"MySQL"` (MySQL). Для PostgreSQL имена обрезаются по числу байт UTF-8 и используются функции `GET_BIT` для битовых полей; для остальных — обрезка по числу символов и маска через `CAST(... & 0xMASK AS BIT)`. |

### Формирование имён SQL-объектов — GetTableName, GetConstraintName, GetColumnName, GetViewName, GetSqlObjectName

Методы формирования имён SQL-объектов класса Replication в языке EME-L. Имя строится в формате `"NNN.Name"`, где `NNN` — порядковый номер объекта, а `Name` — описание записи/поля EME. Символы `"`, `[`, `]` удаляются из описания перед подстановкой. Для PostgreSQL длина ограничивается по числу байт в UTF-8 (в многобайтных символах учитывается по 2 байта), для прочих СУБД — по числу символов (`MAX_SQL_NAME`).

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetSqlObjectName(arg0, arg1)` | arg0: Integer (номер объекта), arg1: String (описание) | String | Базовый метод формирования имени SQL-объекта в формате `"NNN.Name"`. Обрезает до `MAX_SQL_NAME` символов (или байт UTF-8 для PostgreSQL). Удаляет символы `"`, `[`, `]` из описания. |
| `GetTableName(arg0)` | arg0: Integer (номер записи) | String | Имя SQL-таблицы для записи EME. Делегирует в `GetSqlObjectName` с описанием записи из `get_record_narrative`. |
| `GetConstraintName(arg0)` | arg0: Integer (номер записи) | String | Имя ограничения первичного ключа SQL-таблицы. К имени применяется сокращение на 5 символов (`MAX_SQL_NAME - 5`) и добавляется суффикс `"_pkey"`. |
| `GetColumnName(arg0, arg1)` | arg0: Integer или String (номер/имя записи), arg1: Integer или String (номер/имя поля) | String | Имя колонки SQL-таблицы для поля записи EME. Параметры могут быть заданы числом (номер) или строкой (имя записи/поля — разрешается через `smart_find_record`/`smart_find_field`). Если имя поля не сопоставлено — возвращает пустое значение. |
| `GetViewName(arg0)` | arg0: Integer (номер записи) | String | Имя SQL-представления (VIEW), соответствующего записи EME. Берётся из метаданных `CEMERec` (поле `EmeClassName`). Если у записи нет связанного представления — пустая строка. При отсутствии записи в `CEMERec` генерируется ошибка `smart_error`. |
| `GetViewScript(arg0)` | arg0: Integer (номер записи) | String | SQL-скрипт `CREATE VIEW` для записи. Включает колонку `LineKey` (первичный ключ, тип `bigint`), ссылочные поля с суффиксом `"Ref"`, битовые поля с префиксом `"Is"`. Для PostgreSQL битовые поля строятся через `GET_BIT(CAST(... AS BIT(N)), K) <> 0`, для прочих СУБД — через `CAST(... & 0xMASK AS BIT)`. Поля с флагами `FIELD_NOT_IN_USE_BIT` или `DONT_COPY_TO_SERVER_BIT` исключаются. Пустая строка, если представление не определено или не сопоставлено с полями записи. |

### Подготовка и заполнение Query — MarkupTableQuery, QueryCopy, GetInsertLinesScript

Методы класса Replication в языке EME-L для подготовки структуры запроса и переноса строк данных из записи EME. Объект [Query](./ExternalDB.md) — это структура полей и строк, используемая классом ExternalDB для приёма и передачи данных в SQL-базу; класс Replication заполняет Query либо структурой колонок, либо строками данных.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `MarkupTableQuery(arg0, arg1)` | arg0: Integer (номер записи), arg1: Query (объект Query) | Boolean | Заполняет объект Query структурой колонок SQL-таблицы для записи EME: для каждого поля записи вызывает `AddField` с именем колонки, типом и длиной. Пропускает поля с флагами `RECORD_NOT_IN_USE_BIT`, `DONT_COPY_TO_SERVER_BIT` или поля, не попавшие в схему репликации. Если запись помечена как неиспользуемая — возвращает `FALSE` без заполнения. `TRUE` — структура подготовлена успешно. |
| `QueryCopy(arg0, arg1, arg2, arg3)` | arg0: Query, arg1: Integer (номер записи), arg2: Integer (начальная строка), arg3: Integer (конечная строка) | Empty | Копирует строки данных из записи EME в объект Query. Структура Query (имена полей) должна соответствовать именам колонок SQL-таблицы — сопоставление идёт по именам полей Query и колонкам записи. Поле Query с описанием «Флаги» автоматически сопоставляется с системным полем флагов записи (`FLAG_FIELD_ID`). В копию включаются удалённые строки (поле флагов сохраняет признак удаления); это позволяет реплицировать полную картину данных. |
| `GetInsertLinesScript(arg0, arg1, arg2)` | arg0: Integer (номер записи), arg1: Integer (начальная строка), arg2: Integer (конечная строка) | String | Формирует SQL-скрипт `INSERT INTO` для вставки строк данных записи в SQL-таблицу. Первая строка задаёт секцию `INSERT INTO "таблица" (список колонок) VALUES (...)`, последующие строки добавляются через `, (...)`. Значения ссылочных полей выводятся как `INT64`-ключи (`NULL` для пустых ссылок); значения прочих полей — в формате ODBC (`AsSqlValue`). Ключевые поля выводятся как `INT64`. Удалённые строки пропускаются. Скрипт завершается `;`. Пустая строка, если в заданном диапазоне нет пригодных строк данных. |

### Массовое копирование (BulkCopy) — BulkCopy, ExpImpBulkCopy

Методы массового копирования класса Replication в языке EME-L напрямую передают данные записи EME во внешнюю SQL-базу через объект [ExternalDB](./ExternalDB.md), используя механизм BulkCopy провайдера. Это самый быстрый способ выгрузки — BulkCopy минует построчные INSERT и передаёт данные пакетами.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `BulkCopy(arg0, arg1, arg2, arg3)` | arg0: ExternalDB, arg1: Integer (номер записи), arg2: Integer (начальная строка), arg3: Integer (конечная строка) | String | Прямое массовое копирование строк записи EME в SQL-таблицу через `ExternalDB.BulkCopy`. Имя целевой таблицы формируется методом `GetTableName`. Возвращает пустую строку при успехе, иначе текст ошибки. |
| `ExpImpBulkCopy(arg0, arg1, arg2, arg3, arg4)` | arg0: ExternalDB, arg1: String (имя записи), arg2: String (имя таблицы), arg3: Query, arg4: Array (индексы строк) | String | Массовое копирование данных из объекта ExpImp в SQL-таблицу. Строки для копирования задаются массивом индексов (0-based) в Query. Имена полей Query сопоставляются с колонками SQL-таблицы по структуре записи EME. Поля, не сопоставленные с записью, пропускаются; при расхождении типов атрибута поля и атрибута репликации выводится `catastrophic_log` и тип переопределяется на атрибут репликации. Возвращает пустую строку при успехе, иначе текст ошибки. |
| `ExpImpBulkCopy(arg0, arg1, arg2, arg3, arg4, arg5)` | + arg5: Array (имена полей) | String | То же с явным списком имён колонок SQL-таблицы. Позволяет ограничить или переопределить набор колонок для массовой вставки. Используется, когда целевая таблица имеет подмножество колонок записи EME. |

### Перечисление метаданных схемы — EnumTables, EnumColumns, EnumFlags

Методы перечисления метаданных класса Replication в языке EME-L. Перед использованием вызывают внутреннюю инициализацию кэша метаданных схемы (один раз за сессию объекта Replication): кэш строится из проекта EME (`CEMEPrj::GetProject()->GetInf`) и связывает записи EME с описаниями таблиц, колонок и битовых флагов.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `EnumTables(arg0)` | arg0: Query | Empty | Перечисляет все записи EME, доступные для репликации. Поля Query: `RECORD_NO` (Integer, номер записи), `RECORD_NAME` (String, имя записи), `TABLE_NAME` (String, имя SQL-таблицы), `VIEW_NAME` (String, имя представления), `HELP` (String, описание записи). Пропускает записи с флагами `RECORD_NOT_IN_USE_BIT` или `DONT_COPY_TO_SERVER_BIT`. |
| `EnumColumns(arg0, arg1)` | arg0: Integer (номер записи), arg1: Query | Empty | Перечисляет колонки SQL-таблицы для записи. Поля Query: `FIELD_NO` (Integer), `FIELD_NAME` (String), `COLUMN_NAME` (String), `VIEW_NAME` (String, имя в представлении — для ссылок дополняется суффиксом `"Ref"`), `HELP` (String), `TYPE_NAME` (String, SQL-тип: `bigint`, `integer`, `varchar(N)`, `varchar{max}`, `float`, `date`, `time`), `KIND_NAME` (String, 4 символа: `KEY`, `REF`, `BIT`, `FLD`). Поле первичного ключа перечисляется с `VIEW_NAME="LineKey"` и `TYPE_NAME="bigint"`. Поля-начала цепочек (`CHAIN_BEGIN_ATTRIBUTE`) пропускаются. Битовое поле помечается `KIND_NAME="BIT"`, если у него зарегистрированы флаги; иначе `KIND_NAME="FLD"`. |
| `EnumFlags(arg0, arg1, arg2)` | arg0: Integer (номер записи), arg1: Integer (номер поля), arg2: Query | Empty | Перечисляет флаги (битовые под-поля) битового поля записи. Поля Query: `VIEW_NAME` (String, имя с префиксом `"Is"`), `HELP` (String, описание), `BIT` (Integer, номер бита), `MASK` (Integer, маска бита), `BITS_TEXT` (String, список битов в виде текста). Если у поля нет зарегистрированных флагов — Query остаётся пустым. |

### Диагностика дубликатов ключей — GetDupKeys, ResolveDupKeys

Методы класса Replication в языке EME-L для поиска и устранения дубликатов первичных ключей в записях EME. Дубликаты первичных ключей недопустимы в SQL-таблицах с ограничением `PRIMARY KEY` — методы позволяют обнаружить и устранить их перед репликацией.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetDupKeys(arg0, arg1)` | arg0: Integer (номер записи), arg1: Query | Empty | Заполняет Query списком продублированных первичных ключей в записи. Поля Query: `KEY` (`INT64`, значение ключа) и `COUNT` (Integer, число повторений). Ключи со значением 0 игнорируются. Читает все ключи записи единым блоком через `get_elementary_data_block` и сортирует перед подсчётом. |
| `ResolveDupKeys(arg0)` | arg0: Integer (номер записи) | Integer | Устраняет дубликаты первичных ключей в записи: для каждого дубликата (обход с конца диапазона) генерируется новый ключ через `put_new_line_key`. Возвращает количество переназначенных ключей. Подробности каждого изменения (номер, строка, ключ) записываются в catastrophic-лог. |

### Примеры использования класса Replication

Подготовка SQL-скрипта создания представления и вставки строк для записи в системе EME.WMS:

```EME-L
objReplication = Object("Replication");
objReplication.SetDBMSType("PostgreSQL");
'Получить SQL-скрипт CREATE VIEW для записи 150'
ViewScript = objReplication.GetViewScript(150);
'Сформировать INSERT для строк 0..99'
InsertScript = objReplication.GetInsertLinesScript(150, 0, 100);
```

Массовое копирование строк записи во внешнюю базу данных в языке EME-L:

```EME-L
objReplication = Object("Replication");
objExternalDB = Object("ExternalDB");
objExternalDB.Connect("ODBC:DSN=replica;");
objReplication.BulkCopy(objExternalDB, 150, 0, 1000);
```

Перечисление схемы таблицы и поиск дубликатов ключей:

```EME-L
objReplication = Object("Replication");
objQuery = Object("Query");
objReplication.EnumColumns(150, objQuery);
objDupQuery = Object("Query");
objReplication.GetDupKeys(150, objDupQuery);
FixedCount = objReplication.ResolveDupKeys(150);
```

### См. также

- [Класс ExternalDB](./ExternalDB.md) — подключение к SQL-базе и выполнение запросов; объект ExternalDB принимается методами `BulkCopy` и `ExpImpBulkCopy` класса Replication.
- [Класс Query](./DB.md) — объект Query используется для приёма результатов `EnumTables`/`EnumColumns`/`EnumFlags` и передачи строк данных в `BulkCopy`/`ExpImpBulkCopy`.
- [Класс ExpImp](./ExpImp.md) — источник данных для перегрузки `ExpImpBulkCopy` с объектом Query и массивом строк `Lines`.
