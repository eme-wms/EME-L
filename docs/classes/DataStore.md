# Класс DataStore

> Синонимы для поиска: CIDataStore DataStore класс EME-L, хранилище данных EME-L, DataStore PutDirectoryArray GetDirectoryArray SetCurrentDirectory GetCurrentDirectory Stabilize IntegrityTest Save Delete GetNoOfDocuments GetNoOfVersions GetVersions PutData GetData GetDataVersion PutAsString GetAsString GetVersionAsString GetVersionDate GetVersionTime AddQuery GetQuery CDataStorage EME-L, сериализация Query EME-L, версии документов EME-L, стабилизация хранилища EME-L

Класс `DataStore` в языке EME-L предоставляет доступ к хранилищу данных — внешнему по отношению к основной БД постоянному хранилищу, в котором сохраняются версии документов, бинарные данные и сериализованные объекты `Query`. Класс DataStore в системе EME.WMS позволяет открывать и инициализировать хранилища по списку директорий, читать и записывать версии документов по диапазону дат и номеру строки, выполнять стабилизацию и проверку целостности, а также сохранять и восстанавливать объекты `Query` (в том числе построчно).

Объект класса `DataStore` в языке EME-L создаётся через `Object()` и не привязан к контексту диалога или браузера. До выполнения операций чтения/записи необходимо выбрать текущее хранилище класса DataStore — через конструктор с аргументом или методом `SetCurrentDirectory`.

> **Примечание:** хранилище данных (`DataStore`) в системе EME.WMS — это не таблица БД и не файловый архив `Archive`. Это отдельная подсистема версионированного хранения: каждая запись идентифицируется диапазоном дат (`ДатаС`/`ДатаПо`), номером документа (`Документ`) и номером версии. Нестабильные версии становятся стабильными после вызова `Stabilize`.

## Создание объекта класса DataStore

```EME-L
objStore = Object("DataStore");
objStore = Object("DataStore", "WarehouseList");
```

Конструктор без аргументов создаёт объект DataStore без текущей директории — её нужно задать методом `SetCurrentDirectory` перед любыми операциями. Конструктор с аргументом сразу открывает (или инициализирует) указанное хранилище и делает его текущим.

## Управление хранилищами

Методы класса DataStore в языке EME-L для управления подключенными хранилищами данных.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `PutDirectoryArray(arg0)` | arg0: Array (по ссылке) | Empty | Открывает/активирует хранилища по списку директорий из массива; первая становится текущей. Режим исправления ошибок — по умолчанию (true). |
| `PutDirectoryArray(arg0, arg1)` | arg0: Array (по ссылке), arg1: Boolean | Empty | То же с явным флагом `Лечить`: true — исправлять ошибки при открытии файлов, false — не исправлять. |
| `GetDirectoryArray(arg0)` | arg0: Array (по ссылке) | Empty | Заполняет массив именами всех активных хранилищ; содержимое массива очищается. |
| `SetCurrentDirectory(arg0)` | arg0: String | Empty | Устанавливает текущую директорию хранилища. |
| `GetCurrentDirectory()` | — | String | Имя директории текущего хранилища. |

## Стабилизация и проверка

Методы стабилизации и проверки целостности хранилища данных класса DataStore в языке EME-L.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `Stabilize()` | — | Empty | Фиксирует все нестабильные версии документов текущего хранилища, делая их стабильными. При серьёзных ошибках генерирует исключение. |
| `IntegrityTest()` | — | Empty | Проверяет целостность внутренней структуры файлов хранилища. При серьёзных ошибках генерирует исключение. |
| `Save()` | — | Integer | Сохраняет на диск все несохранённые данные (фиксирует pending-изменения). Возвращает код ошибки; 0 — успех. |
| `Delete()` | — | Empty | Удаляет текущее хранилище данных. При серьёзных ошибках генерирует исключение. |

После записи новых данных через `PutData`/`PutAsString`/`AddQuery` вызов `Stabilize` обязателен в языке EME-L — без него новые версии остаются нестабильными и не возвращаются методами `GetData`/`GetAsString` (читающими только последнюю стабильную версию) класса DataStore.

## Статистика хранилища

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetNoOfDocuments()` | — | Integer | Общее количество документов (строк) во всех группах текущего хранилища. |
| `GetNoOfVersions()` | — | Integer | Общее количество версий документов во всех группах текущего хранилища. |
| `GetVersions(arg0, arg1, arg2, arg3)` | arg0: Boolean (нестабильные), arg1: Date (ДатаС), arg2: Date (ДатаПо), arg3: Integer (Документ) | Integer | Количество версий указанного документа в диапазоне дат. 0 — ошибка или отсутствие версий. |

## Запись и чтение текстовых данных

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `PutAsString(arg0, arg1, arg2, arg3)` | arg0: Date, arg1: Date, arg2: Integer (Документ), arg3: String (Текст) | Integer | Записывает новую версию документа в виде строки. 0 — успех. |
| `GetAsString(arg0, arg1, arg2)` | arg0: Date, arg1: Date, arg2: Integer (Документ) | String | Текстовое содержимое последней стабильной версии документа. |

## Запись и чтение бинарных данных

Для работы с бинарными данными класса DataStore в языке EME-L используется объект `CDataStorage` — бинарное хранилище, возвращаемое, например, методом `File.GetData()`.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `PutData(arg0, arg1, arg2, arg3)` | arg0: Date, arg1: Date, arg2: Integer (Документ), arg3: CDataStorage (по ссылке) | Integer | Записывает новую версию бинарных данных. 0 — успех. |
| `GetData(arg0, arg1, arg2)` | arg0: Date, arg1: Date, arg2: Integer (Документ) | CDataStorage | Бинарные данные последней стабильной версии. |
| `GetDataVersion(arg0, arg1, arg2, arg3)` | arg0: Date, arg1: Date, arg2: Integer (Документ), arg3: Integer (Версия) | CDataStorage | Бинарные данные указанной версии. |

## Метаданные версии

Эти методы класса DataStore в языке EME-L возвращают атрибуты конкретной версии документа в заданном диапазоне дат.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetVersionAsString(arg0, arg1, arg2, arg3)` | arg0: Date, arg1: Date, arg2: Integer (Документ), arg3: Integer (Версия) | String | Текстовое содержимое указанной версии. |
| `GetVersionDate(arg0, arg1, arg2, arg3)` | arg0: Date, arg1: Date, arg2: Integer (Документ), arg3: Integer (Версия) | Date | Дата создания указанной версии. |
| `GetVersionTime(arg0, arg1, arg2, arg3)` | arg0: Date, arg1: Date, arg2: Integer (Документ), arg3: Integer (Версия) | Time | Время создания указанной версии. |

## Сериализация объектов Query

Методы `AddQuery`/`GetQuery` класса DataStore в языке EME-L сериализуют объект `Query` (через внутренний архив) и сохраняют его в хранилище как версию документа. Опциональный флаг `ПоСтрокам` включает построчную сериализацию.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `AddQuery(arg0, arg1, arg2, arg3)` | arg0: Date, arg1: Date, arg2: Integer (Номер), arg3: Query (по ссылке) | Integer | Сериализует и сохраняет объект Query. 0 — успех. |
| `AddQuery(arg0, arg1, arg2, arg3, arg4)` | + arg4: Boolean (ПоСтрокам) | Integer | То же с явным флагом построчной сериализации (по умолчанию false). |
| `GetQuery(arg0, arg1, arg2, arg3)` | arg0: Date, arg1: Date, arg2: Integer (Номер), arg3: Query (по ссылке) | Empty | Извлекает сериализованный объект Query в переданный объект. |
| `GetQuery(arg0, arg1, arg2, arg3, arg4)` | + arg4: Boolean (ПоСтрокам) | Empty | То же с явным флагом построчной десериализации (по умолчанию false). |

## Примеры

Сохранение объекта Query в хранилище класса DataStore в языке EME-L и последующая стабилизация:

```EME-L
DataStore = Object("DataStore");
DataStore.SetCurrentDirectory("WarehouseList");
'Сохраняем исходную версию запроса'
DataStore.AddQuery(is_date(), is_date(), VersionNo, Query);
'Фиксирует хранилище. Вызов обязателен!'
DataStore.Stabilize();
```

Чтение сохранённого запроса по номеру версии методом GetQuery класса DataStore в языке EME-L:

```EME-L
DataStore = Object("DataStore");
DataStore.SetCurrentDirectory("WarehouseList");
OldQuery = Object("Query");
OldQuery.Create();
DataStore.GetQuery(is_date(), is_date(), VersionNo, OldQuery);
OldQuery.Dump("Старый запрос");
```

Открытие хранилища через конструктор с директорией:

```EME-L
DataStore = Object("DataStore", "WarehouseList");
NoOfDocs = DataStore.GetNoOfDocuments();
NoOfVersions = DataStore.GetNoOfVersions();
```

Удаление хранилища и очистка связанных записей в БД:

```EME-L
DataStore = Object("DataStore");
DataStore.SetCurrentDirectory(Dir);
is_transaction(1, "Удаляем хранилище " + Dir);
DataStore.Delete();
'...удаление связанных записей в БД...'
is_transaction(-1);
```

## См. также

- [Класс File](./File.md) — метод `File.GetData()` возвращает `CDataStorage`, совместимый с `DataStore.PutData`.
- [Класс Archive](./Archive.md) — архивация файлов (не путать с хранилищем данных).
- [База данных: транзакции](../database/Database.md) — `is_transaction(1, ...)` / `is_transaction(-1)`.
