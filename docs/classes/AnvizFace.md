# Класс AnvizFace

> Синонимы для поиска: CIAnvizFace AnvizFace класс EME-L, биометрический терминал Anviz Face7 EME-L, биометрия TCP/IP EME-L, Connect Disconnect RebootDevice GetSerialNumber SetTimeout SetDeviceDateTime GetDeviceDateTime GetStatistics, UploadEmployes AddUsers GetUsers GetAllEmployee DeleteEmployee DeleteUser InitUserDataArea GetUsersCapacity, AddFingerprintOnline AddFingerPrintForExistingEmployee EnrollFPAndCreateEmployee UploadFingerPrint SetFingerprintTemplate DownloadFingerPrint GetFingerprintTemplate GetFingerprintsCapacity, GetRecords GetAllRecords ClearRecords ClearNewRecords DeleteAllRecords DeleteAllNewRecords GetRecordsCapacity, DataStorage EME-L, tc-b_new_sdk.dll

Класс `AnvizFace` в языке EME-L управляет биометрическими терминалами Anviz Face7 (распознавание лица и отпечатков пальцев) через TCP/IP-соединение. Класс AnvizFace в системе EME.WMS предоставляет методы для подключения к устройству, управления пользователями (добавление, удаление, загрузка и получение списка), получения журналов регистрации (сканирований), работы с биометрическими шаблонами (отпечатки и лица), управления временем устройства, получения статистики и серийного номера, а также перезагрузки устройства.

Объект класса `AnvizFace` в языке EME-L создаётся через `Object()` с обязательным указанием сетевого адреса устройства, порта и идентификатора устройства, а также опциональных параметров таймаута соединения и режима отладки библиотеки `tc-b_new_sdk.dll`. Соединение устанавливается явным вызовом метода `Connect` и разрывается методом `Disconnect`. При уничтожении объекта активное соединение автоматически закрывается — после `Disconnect` для повторной работы необходимо создать новый объект `AnvizFace`.

## Создание объекта класса AnvizFace

```EME-L
'Подключение к устройству Anviz Face7 по адресу 192.168.1.100:5010, deviceId=1'
objAnviz = Object("AnvizFace", "192.168.1.100", 5010, 1);
If (objAnviz.Connect())
    'Подключение установлено'
    SN = objAnviz.GetSerialNumber();
End If
objAnviz.Disconnect();
```

Все три конструктора в языке EME-L принимают IP-адрес, порт и идентификатор устройства; необязательные параметры задают таймаут ответа и уровень отладочного вывода библиотеки `tc-b_new_sdk.dll`.

## Подключение и сессия

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `Connect()` | — | Boolean | Устанавливает TCP/IP-соединение с временем ожидания по умолчанию. TRUE — подключение установлено, FALSE — ошибка. |
| `Connect(waitTime)` | arg0: Integer | Boolean | То же с указанным временем ожидания ответа в миллисекундах. |
| `Disconnect()` | — | Boolean | Разрывает TCP/IP-соединение. После отключения для повторной работы необходимо создать новый объект. |
| `SetTimeout(waitTime)` | arg0: Integer | Boolean | Устанавливает время ожидания ответа для последующих операций, в секундах. |
| `RebootDevice()` | — | Integer | Отправляет команду перезагрузки. 1 — успех, -1 — ошибка. |

## Управление пользователями

Методы управления пользователями класса AnvizFace в языке EME-L загружают и получают списки сотрудников, создавая и перезаписывая строки выходного запроса. Поля входного запроса для загрузки: `Id`, `Name`, `Password`, `Department`, `Group`, `CardId`, `IdentificationType`, `NoPassword`, `NoCardId`.

Поля выходного запроса при получении списка пользователей: `Id`, `Name`, `Group`, `Department`, `PasswordLength`, `CardId`, `CardIdLength`, `UserType`, `FPEnroll`, `Keep`, `Keep2`, `IdentificationType`, `NoPassword`, `NoCardId`.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `UploadEmployes(query)` | arg0: Query (по ссылке) | Integer | Загружает список сотрудников из запроса. Возвращает количество успешно загруженных сотрудников или -1 при ошибке. |
| `AddUsers(query)` | arg0: Query (по ссылке) | Integer | Псевдоним `UploadEmployes`. |
| `GetUsers(query)` | arg0: Query (по ссылке) | Integer | Получает список всех пользователей в запрос. Возвращает количество полученных пользователей или -1 при ошибке. |
| `GetUsers(query, dummy)` | arg0: Query (по ссылке), arg1: Integer | Integer | Версия с двумя аргументами для обратной совместимости; второй аргумент игнорируется. |
| `GetAllEmployee(query)` | arg0: Query (по ссылке) | Integer | То же, что `GetUsers`. |
| `DeleteEmployee(userId)` | arg0: Integer | Integer | Удаляет запись пользователя с устройства. >=0 — успех, <0 — ошибка. |
| `DeleteUser(userId)` | arg0: Integer | Integer | Псевдоним `DeleteEmployee`. |
| `InitUserDataArea()` | — | Boolean | Полностью удаляет всех пользователей и их данные с устройства. |
| `GetUsersCapacity()` | — | Integer | Максимальное количество пользователей устройства. -1 для неподдерживаемой модели. |

## Биометрические шаблоны

Методы работы с биометрией класса AnvizFace в языке EME-L регистрируют, загружают и скачивают биометрические шаблоны (отпечатки пальцев или лица). Поддерживаемые размеры шаблона при загрузке: 338, 1200, 2048, 6144, 10240, 15360 байт. Регистрационные методы (`AddFingerprintOnline`, `AddFingerPrintForExistingEmployee`, `EnrollFPAndCreateEmployee`) запускают интерактивный процесс на устройстве и возвращают результат через внутреннее хранилище данных объекта (`DataStorage`).

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `AddFingerprintOnline(userId, fingerNumber)` | arg0: Integer, arg1: Integer | DataStorage | Запускает процесс добавления шаблона для указанного пользователя. Результат сохраняется во внутреннем хранилище. Диапазон `fingerNumber`: 1..10. |
| `AddFingerPrintForExistingEmployee(userId, fingerNumber)` | arg0: Integer, arg1: Integer | Integer | Регистрирует шаблон под временным идентификатором и привязывает к существующему сотруднику. >0 — успех, -3 — ошибка регистрации, -5 — `userId >= 0xFFFF`. |
| `EnrollFPAndCreateEmployee(id, name, password, departmentId, groupId, cardId, identificationType, fingerNumber)` | arg0..arg6 + arg7: Integer/String | Integer | Регистрирует нового сотрудника с биометрией. Сначала создаётся пользователь, затем запускается запись шаблона. `identificationType`: 196 — только отпечаток, 248 — отпечаток и карта. >0 — успех, -3 — ошибка шаблона, -4 — ошибка создания пользователя, -5 — `id >= 0xFFFF`. |
| `UploadFingerPrint(userId, fingerNumber, templateData)` | arg0: Integer, arg1: Integer, arg2: DataStorage | Integer | Загружает шаблон на устройство. >0 — успех, -1 — ошибка или некорректный размер шаблона. |
| `SetFingerprintTemplate(userId, fingerNumber, templateData)` | arg0: Integer, arg1: Integer, arg2: DataStorage | Boolean | Псевдоним `UploadFingerPrint` с булевым результатом. TRUE — шаблон загружен. |
| `DownloadFingerPrint(userId, fingerNumber)` | arg0: Integer, arg1: Integer | DataStorage или FALSE | Скачивает шаблон пользователя через внутреннее хранилище. FALSE — ошибка. |
| `GetFingerprintTemplate(userId, fingerNumber)` | arg0: Integer, arg1: Integer | DataStorage или FALSE | Псевдоним `DownloadFingerPrint`. |
| `GetFingerprintsCapacity()` | — | Integer | Максимальное количество биометрических шаблонов. -1 для неподдерживаемой модели. |

## Журналы регистрации (записи)

Записи о регистрациях (сканированиях) класса AnvizFace в языке EME-L — это журнал проходов через биометрический терминал. Поля выходного запроса: `NewRecordFlag`, `UserCode`, `DateTime`, `BackId`, `RecordType`, `WorkType`, `Rsv`.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetRecords(query, count, onlyNew)` | arg0: Query (по ссылке), arg1: Integer, arg2: Boolean | Boolean | Получает записи в запрос. `count <= 0` — все записи. `onlyNew=TRUE` — только новые. TRUE — успех. |
| `GetAllRecords(query)` | arg0: Query (по ссылке) | Integer | Получает все записи в запрос. Возвращает количество полученных записей или -1 при ошибке. |
| `ClearRecords()` | — | Boolean | Удаляет все записи. Псевдоним `DeleteAllRecords`. |
| `ClearRecords(count)` | arg0: Integer | Boolean | Удаляет указанное количество записей. |
| `ClearNewRecords()` | — | Boolean | Очищает метку новой записи со всех записей. |
| `ClearNewRecords(count)` | arg0: Integer | Boolean | Очищает метку новой записи с указанного количества записей. |
| `DeleteAllRecords()` | — | Integer | Удаляет все записи. Возвращает количество удалённых или -1 при ошибке. |
| `DeleteAllNewRecords()` | — | Integer | Очищает метку новой записи со всех записей. Возвращает количество обработанных или -1 при ошибке. |
| `GetRecordsCapacity()` | — | Integer | Максимальное количество записей устройства. -1 для неподдерживаемой модели. |

## Информация об устройстве

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetSerialNumber()` | — | String | Серийный номер устройства. Пустая строка при ошибке. |
| `GetDeviceDateTime()` | — | DateTime или Empty | Текущие дата и время устройства. Пустое значение при ошибке. |
| `SetDeviceDateTime(dateTime)` | arg0: DateTime | Boolean | Устанавливает дату и время устройства. |
| `GetStatistics(query)` | arg0: Query (по ссылке) | Integer | Запрашивает статистику в запрос одной строкой. Поля: `AllRecordAmount`, `CardAmount`, `FingerPrintAmount`, `NewRecordAmount`, `PasswordAmount`, `UserAmount`. 1 — успех, -1 — ошибка. |

## Примеры использования класса AnvizFace

Подключение, получение серийного номера и статистики в языке EME-L:

```EME-L
'Создать объект и подключиться к терминалу Anviz Face7'
objAnviz = Object("AnvizFace", "192.168.1.100", 5010, 1, 10);
If (objAnviz.Connect())
    'Подключение установлено — запросить серийный номер'
    SN = objAnviz.GetSerialNumber();
    'SN содержит серийный номер устройства'

    'Получить статистику по сотрудникам и записям в запрос'
    objStat = Object("Query");
    objAnviz.GetStatistics(objStat);
    'objStat заполнен одной строкой с полями AllRecordAmount, UserAmount и др.'
End If
objAnviz.Disconnect();
```

Получение журнала регистраций за день в языке EME-L:

```EME-L
'Прочитать последние 100 новых записей о сканированиях'
objAnviz = Object("AnvizFace", "192.168.1.100", 5010, 1);
objAnviz.Connect();
objRecords = Object("Query");
'onlyNew=TRUE, count=100 — только новые записи, не более 100'
objAnviz.GetRecords(objRecords, 100, TRUE);
Loop (objRecords)
    'UserCode — идентификатор сотрудника, DateTime — время сканирования'
    UserCode = objRecords.UserCode;
    ScanTime = objRecords.DateTime;
End Loop
'Снять метку "новая" с обработанных записей'
objAnviz.ClearNewRecords();
objAnviz.Disconnect();
```

Регистрация нового сотрудника с отпечатком пальца в системе EME.WMS:

```EME-L
'Создать сотрудника id=42 и запустить запись отпечатка (номер 1)'
objAnviz = Object("AnvizFace", "192.168.1.100", 5010, 1);
objAnviz.Connect();
'identificationType=196 — только отпечаток; 248 — отпечаток и карта'
res = objAnviz.EnrollFPAndCreateEmployee(42, "Иванов И.И.", 1234, 1, 1, 0, 196, 1);
'>0 — успех, -3 — ошибка записи шаблона, -4 — ошибка создания пользователя, -5 — id >= 0xFFFF'
If (res > 0)
    'Сотрудник создан и зарегистрирован на устройстве'
End If
objAnviz.Disconnect();
```

## См. также

- [Класс File](File.md) — другой emel-класс EME-L, работа с файловой системой.
- [EME-L Advanced API and Patterns](../EME_L_Advanced_API_and_Patterns.md) — методы класса `Query`, используемого как приёмник данных методов `AnvizFace`.
