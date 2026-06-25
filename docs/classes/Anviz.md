# Класс Anviz

> Синонимы для поиска: CIAnviz Anviz класс EME-L, биометрический терминал Anviz EME-L, сканер отпечатков Anviz EME-L, TCP/IP терминал учёта рабочего времени, Connect Disconnect SetTimeout GetUsers AddUsers AddOneUser DeleteUser DeleteUserInfo InitUserDataArea GetUsersCapacity GetFingerprintTemplate SetFingerprintTemplate AddFingerprintOnline AddFingerPrintForExistingEmployee EnrollFPAndCreateEmployee GetFingerprintsCapacity GetRecords ClearRecords ClearNewRecords GetRecordsCapacity GetDeviceDateTime SetDeviceDateTime AddMessage GetMessage GetMessageList DeleteMessage GetStatistics, ACK_SUCCESS, DataStorage EME-L, отпечатки пальцев Anviz, журнал сканирований EME-L, карты доступа Anviz

Класс `Anviz` в языке EME-L управляет биометрическими терминалами Anviz (сканерами отпечатков пальцев и карт доступа) через TCP/IP-соединение. Класс Anviz в системе EME.WMS предоставляет методы для подключения к устройству, управления пользователями (добавление, удаление, загрузка списка), получения журналов событий (сканирований о входе и выходе), работы с отпечатками пальцев, управления временем устройства и персональными сообщениями пользователей.

Объект класса `Anviz` в языке EME-L создаётся с указанием сетевого адреса устройства (host), порта (port) и идентификатора устройства (deviceId), а также опциональных параметров таймаута соединения (timeout, по умолчанию 2000 мс) и паузы после отправки команд (receiveWait, по умолчанию 400 мс). Anviz — это аппаратный бренд биометрических терминалов; класс EME-L инкапсулирует их сетевой протокол поверх TCP/IP.

## Создание объекта класса Anviz

Конструкторы класса Anviz в языке EME-L имеют перегрузки — от 3 обязательных аргументов до 5.

```EME-L
'Подключение к терминалу по адресу 192.168.1.150, порт 5010, deviceId = 1'
objAnviz = Object("Anviz", "192.168.1.150", 5010, 1);

'Та же конфигурация с таймаутом ответа 3000 мс'
objAnviz = Object("Anviz", "192.168.1.150", 5010, 1, 3000);

'Таймаут 3000 мс и пауза после отправки команды 500 мс'
objAnviz = Object("Anviz", "192.168.1.150", 5010, 1, 3000, 500);
```

Все параметры после `deviceId` опциональны: `timeout` по умолчанию 2000 мс, `receiveWait` по умолчанию 400 мс. Конструктор класса Anviz возвращает ссылку на объект CIAnviz; объект автоматически отключается от устройства в деструкторе при выходе из области видимости.

## Подключение и сеанс класса Anviz

Методы подключения класса Anviz в языке EME-L управляют TCP/IP-соединением с биометрическим терминалом.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `Connect()` | — | Boolean | Устанавливает TCP/IP-соединение с таймаутом по умолчанию 5000 мс. TRUE — успешно. |
| `Connect(arg0)` | arg0: Integer (мс) | Boolean | То же с заданным таймаутом подключения. |
| `Disconnect()` | — | Boolean | Разрывает соединение с таймаутом по умолчанию 400 мс. |
| `Disconnect(arg0)` | arg0: Integer (мс) | Boolean | То же с заданным таймаутом отключения. |
| `SetTimeout(arg0)` | arg0: Integer (сек) | Boolean | Устанавливает время ожидания ответа от устройства при последующих командах. Всегда TRUE. |

Типичный цикл работы с классом Anviz в языке EME-L: создать объект, вызвать `Connect`, выполнить операции, затем `Disconnect`. Соединение автоматически разрывается в деструкторе объекта при выходе из области видимости.

## Пользователи класса Anviz

Методы класса Anviz в языке EME-L для управления пользователями биометрического терминала. Методы загрузки списка пользователей возвращают данные через объект `Query` (класс EME-L Query), поля которого перечислены в описании каждого метода.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetUsers(arg0)` | arg0: Query (по ссылке) | Boolean | Загружает статистику по количеству записей, пользователей, отпечатков. Поля: `AllRecordAmount`, `CardAmount`, `FingerPrintAmount`, `NewRecordAmount`, `PasswordAmount`, `UserAmount` (Integer). |
| `GetUsers(arg0, arg1)` | arg0: Query (по ссылке), arg1: Integer | Boolean | Загружает список сотрудников. Поля: `Id`, `Name` (String, 64), `Group`, `Department`, `Password`, `PasswordLength`, `CardId`, `CardIdLength`, `UserType`, `FPEnroll`, `Keep`, `Keep2`, `IdentificationType`, `NoPassword`, `NoCardId` (Integer). |
| `AddUsers(arg0)` | arg0: Query (по ссылке) | Integer | Загружает сотрудников из запроса на устройство. Возвращает битовую маску подтверждённых загрузок; 0 — ошибка. Обязательные поля запроса: `Id`, `Name`, `Password`, `Department`, `Group`, `CardId`, `IdentificationType`, `NoPassword`, `NoCardId`. |
| `AddOneUser(arg0, arg1, ..., arg12)` | 13 аргументов (см. ниже) | Boolean | Добавляет одного пользователя с полным набором параметров. |
| `DeleteUser(arg0)` | arg0: Integer | Boolean | Полностью удаляет пользователя с устройства по идентификатору. |
| `DeleteUserInfo(arg0, arg1, arg2, arg3, arg4)` | arg0: Integer, arg1–arg4: Boolean | Boolean | Удаляет указанные данные пользователя (отпечатки, пароль, карту), не удаляя самого пользователя. |
| `InitUserDataArea()` | — | Boolean | Удаляет данные ВСЕХ пользователей, включая отпечатки, пароли и карты. |
| `GetUsersCapacity()` | — | Integer | Максимальное количество пользователей, которое устройство может хранить. |

### Параметры метода AddOneUser класса Anviz

Метод `AddOneUser` класса Anviz в языке EME-L принимает 13 аргументов, описывающих пользователя терминала.

| № | Параметр | Тип | Описание |
|---|----------|-----|----------|
| arg0 | `Id` | Integer | Идентификатор пользователя на устройстве. |
| arg1 | `Name` | String | Имя в кодировке UTF-8, максимум 64 байта. |
| arg2 | `UserType` | Integer | Тип пользователя. По умолчанию 1 (обычный пользователь). |
| arg3 | `Password` | Integer | Пароль пользователя. Если не задан — пароль отключён. |
| arg4 | `PasswordLen` | Integer | Длина пароля в символах. Если не задана — вычисляется автоматически. |
| arg5 | `CardId` | Integer | Идентификатор карты доступа. Если не задан — карта отключена. |
| arg6 | `CardIdLen` | Integer | Длина идентификатора карты в битах: 24 или 32. По умолчанию 0. |
| arg7 | `Group` | Integer | Идентификатор группы. По умолчанию 0. |
| arg8 | `Department` | Integer | Идентификатор отдела. По умолчанию 0. |
| arg9 | `FPEnroll` | Integer | Состояние регистрации отпечатков. По умолчанию 0. |
| arg10 | `Keep` | Integer | Резервное поле. По умолчанию 36. |
| arg11 | `Keep2` | Integer | Резервное поле. По умолчанию 13. |
| arg12 | `IdentificationType` | Integer | Тип идентификации. По умолчанию 0 (не задан). |

### Параметры метода DeleteUserInfo класса Anviz

Метод `DeleteUserInfo` класса Anviz в языке EME-L удаляет указанные данные пользователя, оставляя самого пользователя в памяти устройства.

| № | Параметр | Тип | Описание |
|---|----------|-----|----------|
| arg0 | `userId` | Integer | Идентификатор пользователя на устройстве. |
| arg1 | `deleteFP1` | Boolean | TRUE — удалить отпечаток пальца N1. |
| arg2 | `deleteFP2` | Boolean | TRUE — удалить отпечаток пальца N2. |
| arg3 | `deletePassword` | Boolean | TRUE — удалить пароль. |
| arg4 | `deleteCard` | Boolean | TRUE — удалить карту доступа. |

## Отпечатки пальцев класса Anviz

Методы класса Anviz в языке EME-L для работы с шаблонами отпечатков пальцев на биометрическом терминале. Онлайн-регистрация отпечатка на старых моделях Anviz C может не работать.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetFingerprintTemplate(arg0, arg1)` | arg0: Integer (userId), arg1: Integer (1 или 2) | CDataStorage или Boolean | Загружает бинарный шаблон отпечатка. При ошибке — FALSE. |
| `SetFingerprintTemplate(arg0, arg1, arg2)` | arg0: Integer, arg1: Integer (1 или 2), arg2: CDataStorage | Boolean | Загружает бинарный шаблон на устройство. |
| `AddFingerprintOnline(arg0, arg1, arg2)` | arg0: Integer (userId), arg1: Integer (палец), arg2: Integer (1 или 2 — стадия) | Integer | Запускает онлайн-регистрацию отпечатка или лица. Для полного завершения вызывать дважды со стадией 1 и 2. При успехе — ACK_SUCCESS. |
| `AddFingerPrintForExistingEmployee(arg0, [arg1])` | arg0: Integer (userId), arg1: Integer (палец, 1–10, по умолч. 1) | Integer | Регистрирует отпечаток для существующего пользователя, автоматически выполняя обе стадии. При успехе — код успеха, при ошибке стадии — отрицательный код, если пользователь не найден — -1. |
| `EnrollFPAndCreateEmployee(arg0, arg1, ..., arg7)` | 6–8 аргументов (см. ниже) | Integer | Создаёт нового пользователя и запускает регистрацию отпечатка с обеими стадиями автоматически. |
| `GetFingerprintsCapacity()` | — | Integer | Максимальное количество отпечатков, которое устройство может хранить. |

### Параметры метода EnrollFPAndCreateEmployee класса Anviz

Метод `EnrollFPAndCreateEmployee` класса Anviz в языке EME-L создаёт нового пользователя и запускает процесс онлайн-регистрации отпечатка пальца, автоматически выполняя обе стадии сканирования.

| № | Параметр | Тип | Описание |
|---|----------|-----|----------|
| arg0 | `Id` | Integer | Идентификатор пользователя. |
| arg1 | `Name` | String | Имя в кодировке UTF-8, максимум 64 байта. |
| arg2 | `Password` | Integer | Пароль пользователя. По умолчанию не задан. |
| arg3 | `DepartmentId` | Integer | Идентификатор отдела. По умолчанию 0. |
| arg4 | `GroupId` | Integer | Идентификатор группы. По умолчанию 0. |
| arg5 | `CardId` | Integer | Идентификатор карты доступа. По умолчанию не задан. |
| arg6 | `IdentificationType` | Integer | Тип идентификации. По умолчанию 248 (палец и карта). Возможные значения: 196 (только палец), 248 (палец и карта). |
| arg7 | `fingerNumber` | Integer | Номер пальца для регистрации (1–10). По умолчанию 1. |

Возвращает: при успехе — `ACK_SUCCESS`; если пользователь уже существует — -1; при ошибке загрузки данных пользователя — -2; при ошибке на стадии регистрации — отрицательный код ошибки устройства.

## Записи сканирований (журнал событий) класса Anviz

Методы класса Anviz в языке EME-L для получения и очистки журнала событий биометрического терминала — записей о сканированиях при входе и выходе сотрудников. Записи загружаются через объект `Query`.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetRecords(arg0, arg1, [arg2])` | arg0: Query (по ссылке), arg1: Integer (кол-во), arg2: Boolean (только новые, по умолч. FALSE) | Boolean | Загружает записи о входе/выходе. Поля: `BackupCode`, `DateTime` (DateTime), `RecordType`, `UserCode`, `WorkType`, `RawDateTime` (Integer). |
| `ClearRecords()` | — | Boolean | Удаляет все записи сканирований с устройства. |
| `ClearRecords(arg0)` | arg0: Integer | Boolean | Удаляет указанное количество записей. Если 0 — удаляются все. |
| `ClearNewRecords()` | — | Boolean | Сбрасывает метку «новая» со всех записей. |
| `ClearNewRecords(arg0)` | arg0: Integer | Boolean | Сбрасывает метку с указанного количества записей. Если 0 — со всех. |
| `GetRecordsCapacity()` | — | Integer | Максимальное количество записей сканирований, которое устройство может хранить. |

## Время устройства класса Anviz

Методы класса Anviz в языке EME-L для управления встроенными часами биометрического терминала.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetDeviceDateTime()` | — | DateTime или Empty | Текущие дата и время устройства. При ошибке возвращает пустое значение. |
| `SetDeviceDateTime(arg0)` | arg0: DateTime | Boolean | Устанавливает дату и время на устройстве. |

## Персональные сообщения класса Anviz

Методы класса Anviz в языке EME-L для управления персональными сообщениями, отображаемыми на экране биометрического терминала в заданный период для конкретного пользователя. Текст сообщения — максимум 48 символов в UTF-8.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `AddMessage(arg0, arg1, arg2, arg3)` | arg0: Integer (userId), arg1: Date (начало), arg2: Date (конец), arg3: String (текст, ≤ 48 символов UTF-8) | Boolean | Добавляет персональное сообщение, отображаемое в заданный период. |
| `GetMessage(arg0, arg1)` | arg0: Integer (индекс), arg1: Query (по ссылке) | Boolean | Получает сообщение по индексу. Поля: `UserId`, `StartDate` (Date), `EndDate` (Date), `Text` (String, 48). |
| `GetMessageList(arg0)` | arg0: Query (по ссылке) | Boolean | Получает список заголовков сообщений (без текста). Поля: `UserId`, `StartDate`, `EndDate`. |
| `DeleteMessage(arg0)` | arg0: Integer (индекс) | Boolean | Удаляет сообщение по порядковому номеру на устройстве. |

## Статистика класса Anviz

Метод класса Anviz в языке EME-L для получения сводной информации о содержимом биометрического терминала.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetStatistics(arg0)` | arg0: Query (по ссылке) | Boolean | Помещает статистику устройства в запрос одной строкой. Поля: `AllRecordAmount`, `CardAmount`, `FingerPrintAmount`, `NewRecordAmount`, `PasswordAmount`, `UserAmount` (Integer). |

## Примеры работы с классом Anviz

Пример подключения к терминалу Anviz и получения журнала сканирований в языке EME-L:

```EME-L
'Создать объект и подключиться к терминалу Anviz'
objAnviz = Object("Anviz", "192.168.1.150", 5010, 1);
If (objAnviz.Connect())
    'Загрузить 100 последних записей сканирований'
    objQuery = Object("Query");
    If (objAnviz.GetRecords(objQuery, 100, TRUE))
        'Обработать записи в objQuery...'
        nRecords = objQuery.GetNoOfLines();
    End If
    objAnviz.Disconnect();
End If
```

Синхронизация времени устройства Anviz с сервером в языке EME-L:

```EME-L
objAnviz = Object("Anviz", "192.168.1.150", 5010, 1);
If (objAnviz.Connect())
    'Установить на устройстве текущее серверное время'
    objAnviz.SetDeviceDateTime(is_now());
    objAnviz.Disconnect();
End If
```

Создание пользователя класса Anviz с регистрацией отпечатка пальца в одной операции в языке EME-L:

```EME-L
objAnviz = Object("Anviz", "192.168.1.150", 5010, 1);
If (objAnviz.Connect())
    'Создать пользователя 42 с именем "Иванов" и зарегистрировать палец 1'
    Result = objAnviz.EnrollFPAndCreateEmployee(42, "Иванов", , , , , 248, 1);
    'Result = ACK_SUCCESS при успехе, -1 если пользователь уже существует'
    objAnviz.Disconnect();
End If
```

Загрузка списка пользователей с устройства Anviz в языке EME-L:

```EME-L
objAnviz = Object("Anviz", "192.168.1.150", 5010, 1);
If (objAnviz.Connect())
    objQuery = Object("Query");
    'Загрузить до 200 пользователей'
    If (objAnviz.GetUsers(objQuery, 200))
        nUsers = objQuery.GetNoOfLines();
    End If
    objAnviz.Disconnect();
End If
```

## См. также

- [Класс File](./File.md) — работа с файловой системой в языке EME-L.
- Класс Query в языке EME-L — объект Query, через который возвращаются данные пользователей и записей сканирований.
