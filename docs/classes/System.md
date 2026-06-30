# Класс System

> Синонимы для поиска: CISystem System класс EME-L, системный класс EME-L, глобальные сервисы EME-L, InitQueryCache ClearQueryCache IsQueryCaching GetQuery, EnableComb IsEnabledComb GetOvlCode SetOvlCode LoadBreakpoints, BrowseForFolder BrowseForFile BrowseForFilesGroup GetFileNameDialog InputColor, StartSection NextStep EndSection ExecuteInThread SuspendThread ResumeThread StopThread GetThreadInfo, RunReport RunReportIndirect RunReportIndirectInPrinter SaveReportToBuffer SendReportByEmail SendReportAsExcelByEmail ExportXlsx, HttpRequest HttpsRequest WEBRequest HttpPostFile HttpsPostFile WebService GetConnectedUsers, GetCurrentDir SetCurrentDir CreateDir GetMetaprojDir GetKernelDir GetDataDir GetDbDir GetArchiveDir DeleteOldFiles GetEnvironmentVariable System, GetSystemPrintersAsArray GetNameDefaultPrinter GetPrinterUserName GetOrganizationName SetOrganizationName UpdateWindowHeader NewMailMessage, SetSemaphore CheckSemaphore DeleteSemaphore, MakeEvent CallResident Sleep Dump AddSmartDate RemoveSmartDate RemoveAllSmartDates CloseAllDialogs GetAllDialogsList ShowCloud ShowServerConsole Help HelpInNewProcess InspectObjects InspectQueries, центральная точка доступа EME-L, инфраструктурные сервисы платформы EME

Класс `System` в языке EME-L — это системный класс, предоставляющий доступ к глобальным возможностям среды выполнения EME. Класс System в языке EME-L является центральной точкой доступа к инфраструктурным сервисам платформы EME: управлению выполнением скриптов EME-L, взаимодействию с пользователем через диалоги и сообщения, формированию и печати отчетов, сетевому обмену, работе с файловой системой и получению системной информации.

Объект класса System в языке EME-L создаётся без параметров или с явным контекстом выполнения. Используется в прикладных скриптах EME-L для доступа к системным сервисам и управления глобальным состоянием приложения. Класс зарегистрирован под именем `System`, соответствующий C++-класс — `CISystem` (исходный файл `i_System.cpp`).

Класс System в системе EME.WMS охватывает следующие функциональные категории: управление контекстом выполнения (кэш запросов, OVL-код, «гребёнка» БД), прогресс-индикаторы и рабочие потоки, диалоги выбора каталога/файла/цвета и взаимодействие с интерфейсом, отчёты и печать (экран, принтер, буфер, e-mail, XLSX-экспорт), сетевое взаимодействие (HTTP/HTTPS, web-сервисы ЕМЕ), файловая система и переменные окружения, системная информация (принтеры, название проекта, заголовок окна, почта), серверные семафоры для защиты записей от одновременного выполнения, а также события, вызовы резидента, умный календарь и вспомогательные сервисы.

## Создание объекта класса System

```EME-L
'Создать объект System с использованием текущего контекста выполнения'
sys = Object("System");

'Создать объект System с явным контекстом вызова'
sys = Object("System", context);
```

Конструктор без аргументов в классе System в языке EME-L использует текущий контекст, если он доступен. При передаче контекста (`context` — ссылка) некоторые методы выполняются относительно этого контекста. Вызов без аргументов эквивалентен конструктору с одним аргументом, когда контекст уже задан средой.

В большинстве прикладных скриптов EME-L объект System используется через inline-выражение `Object("System")` без сохранения в переменную — например, для однократного вызова `InitQueryCache()`. При многократных обращениях в пределах метода предпочтительно сохранить объект в локальной переменной класса EME-L.

## Управление контекстом выполнения в языке EME-L

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `InitQueryCache()` | — | Empty | Инициализирует кэш запросов в языке EME-L. |
| `InitQueryCache(keepManual)` | keepManual: Boolean | Empty | Инициализирует кэш запросов. При `keepManual = TRUE` запрос хранится в кэше до явного удаления; иначе удаляется автоматически. |
| `ClearQueryCache()` | — | Empty | Очищает кэш запросов EME-L. |
| `IsQueryCaching()` | — | Boolean | Возвращает `TRUE` (в EME-L `TRUE = -1`), если кэш запросов активен, иначе `FALSE` (`0`). |
| `GetQuery([name])` | name: String (необязательный) | ссылка | Возвращает объект запроса из кэша по имени. Без аргумента — текущий запрос из контекста. Если запрос не найден, генерируется ошибка. |
| `GetOvlCode()` | — | Integer | Возвращает код запуска текущего модуля (OVL-код) в системе EME. |
| `SetOvlCode(ovlCode)` | ovlCode: Integer | Integer | Устанавливает OVL-код текущего модуля. Возвращает предыдущее значение. |
| `EnableComb(mode)` | mode: Boolean/Integer | Integer | Управляет «гребенчатой» схемой работы базы данных EME-L. Возвращает результат установки режима. |
| `IsEnabledComb()` | — | Integer | Возвращает состояние «гребенчатой» схемы БД. |
| `LoadBreakpoints(classRecord)` | classRecord: String/Integer | Empty | Загружает из базы данных точки останова заданного класса EME-L. `classRecord` — строка в записи Классы или номер записи. |

Методы `EnterTransactionSection()` и `LeaveTransactionSection()` класса System в языке EME-L зарезервированы, но в текущей реализации не выполняют действий — оставлены для совместимости.

## Прогресс-индикаторы и рабочие потоки в системе EME

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `StartSection(steps, name)` | steps: Integer, name: String | Empty | Запускает прогресс-индикатор с заданным числом шагов. |
| `NextStep()` | — | Empty | Выполняет очередной шаг прогресс-индикатора. |
| `EndSection()` | — | Empty | Завершает секцию и выключает прогресс-индикатор. |
| `ExecuteInThread(method, [className], [objectName], [sections], [async], [reserved5], [reserved6], [reserved7], [param...])` | method: String/Integer; остальные — необязательные | Integer (дескриптор потока) или произвольный | Выполняет метод объекта класса в рабочем потоке EME-L. `method` — имя или номер метода; `className` — имя класса для создания объекта; `sections` — число секций потока (`-1` — не вести учёт); `async = TRUE` — запустить независимый поток и не ждать завершения. Параметры `reserved5`–`reserved7` должны быть пустыми. Остальные аргументы передаются вызываемому методу. |
| `SuspendThread(descriptor)` | descriptor: Integer | Empty | Приостанавливает поток, запущенный асинхронно через `ExecuteInThread`. |
| `ResumeThread(descriptor)` | descriptor: Integer | Empty | Продолжает выполнение приостановленного потока. |
| `StopThread(descriptor)` | descriptor: Integer | Empty | Останавливает поток, запущенный асинхронно. |
| `GetThreadInfo(descriptor)` | descriptor: Integer | String | Возвращает текстовую информацию о запущенном потоке. |

## Диалоги выбора и взаимодействие с интерфейсом в языке EME-L

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `BrowseForFolder()` | — | String | Открывает диалог выбора каталога с путём по умолчанию. Возвращает путь; пустая строка при отмене. |
| `BrowseForFolder(startPath)` | startPath: String | String | Диалог выбора каталога с начальным путём. |
| `BrowseForFolder(startPath, title)` | startPath: String, title: String | String | Диалог выбора каталога с заголовком окна. |
| `BrowseForFile(filter, startPath)` | filter: String, startPath: String | String | Диалог выбора файла. `filter` — например, `"Текстовые файлы (*.txt)\|*.txt\|Все файлы (*.*)\|*.*\|\|"`. |
| `BrowseForFile(filter, startPath, title)` | filter, startPath: String, title: String | String | Диалог выбора файла с заголовком. |
| `BrowseForFile(filter, startPath, title, openMode)` | openMode: Integer (1 — Открыть, 0 — Сохранить) | String | Диалог выбора или сохранения файла. |
| `BrowseForFile(filter, startPath, title, openMode, flags)` | flags: Integer | String | Диалог выбора/сохранения файла с дополнительными флагами OpenFileName. |
| `BrowseForFilesGroup(filter, startPath)` | filter: String, startPath: String | Array | Возвращает массив полных путей выбранных файлов; пустое значение при отмене. |
| `GetFileNameDialog(filter, startPath, openMode, title)` | openMode: Boolean (TRUE — Открыть, FALSE — Сохранить как); title: String (пустая — заголовок по умолчанию) | String | Диалог выбора имени файла. |
| `GetFileNameDialog(filter, startPath, openMode, title, defaultFileName)` | defaultFileName: String | String | Диалог выбора имени файла с подставленным именем по умолчанию. |
| `InputColor(defaultColor)` | defaultColor: Integer (RGB) | Integer | Диалог выбора цвета MFC `CColorDialog`. При отмене возвращает `defaultColor`. |
| `CloseAllDialogs([exceptDialog])` | exceptDialog: String (необязательный) | Empty | Закрывает все открытые диалоги EME-L, кроме указанного по внутреннему имени. |
| `GetAllDialogsList([dialogsArray])` | dialogsArray: Array (необязательный) | Integer | Возвращает количество открытых диалогов. Если передан массив, заполняет его внутренними именами диалогов. |
| `ShowCloud(message, [delay])` | message: String; delay: Integer (по умолчанию 5 секунд) | Empty | Показывает всплывающее сообщение-облако в статус-баре приложения EME. |
| `ShowServerConsole()` | — | Empty | Выводит консоль сервера EME. |
| `Help(topic)` | topic: String | Empty | Выводит справку о классах EME-L в модальном окне. |
| `HelpInNewProcess(topic)` | topic: String | Empty | Выводит справку о классах EME-L в новом процессе через браузер. |
| `InspectObjects()` | — | Empty | Выводит окно инспектора объектов EME-L. |
| `InspectQueries()` | — | Empty | Выводит окно инспектора объектов запросов из кэша. |

## Отчёты и печать в системе EME.WMS

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `RunReport(reportName)` | reportName: String | Empty | Выводит отчёт на экран. |
| `RunReport(reportName, dbLine)` | dbLine: Integer | Empty | Выводит отчёт для указанной базовой строки БД. |
| `RunReport(reportName, dbLine, params)` | params: String или CIParameters | Empty | Выводит отчёт с дополнительными параметрами. |
| `RunReportIndirect(reportName)` | reportName: String | Boolean | Печатает отчёт напрямую на принтер. |
| `RunReportIndirect(reportName, dbLine)` | dbLine: Integer | Boolean | Печать на принтер для указанной базовой строки. |
| `RunReportIndirect(reportName, dbLine, copies)` | copies: Integer | Boolean | Печать с указанным числом копий. |
| `RunReportIndirect(reportName, dbLine, copies, params)` | params: String | Boolean | Печать с дополнительными параметрами. |
| `RunReportIndirect(reportName, dbLine, copies, params, lParam)` | lParam: ссылка | Boolean | Печать с дополнительным параметром `lParam`. |
| `RunReportIndirect(reportName, dbLine, copies, params, lParam, printInThread)` | printInThread: Boolean | Boolean | Печать, возможно, в отдельном потоке. |
| `RunReportIndirectInPrinter(reportName, printerName)` | printerName: String | Boolean | Печать напрямую на указанный принтер. |
| `RunReportIndirectInPrinter(reportName, printerName, dbLine)` | dbLine: Integer | Boolean | Печать на указанный принтер для базовой строки. |
| `RunReportIndirectInPrinter(reportName, printerName, dbLine, copies)` | copies: Integer | Boolean | Печать с числом копий. |
| `RunReportIndirectInPrinter(reportName, printerName, dbLine, copies, params)` | params: String | Boolean | Печать с параметрами. |
| `RunReportIndirectInPrinter(reportName, printerName, dbLine, copies, params, port)` | port: String | Boolean | Печать с указанием порта принтера. |
| `RunReportIndirectInPrinter(reportName, printerName, dbLine, copies, params, port, lParam)` | lParam: ссылка | Boolean | Печать с параметром `lParam`. |
| `RunReportIndirectInPrinter(reportName, printerName, dbLine, copies, params, port, lParam, printInThread)` | printInThread: Boolean | Boolean | Печать, возможно, в отдельном потоке. |
| `RunReportIndirectInPrinter(reportName, printerName, dbLine, copies, params, port, lParam, printInThread, orientation, duplex)` | orientation: Integer (1 — вертикальная, 2 — горизонтальная); duplex: Boolean | Boolean | Полная перегрузка с ориентацией страницы и двусторонней печатью. |
| `SaveReportToBuffer(reportName, dbLine, storage)` | storage: CIDataStorage | Integer | Сохраняет внутреннее представление отчёта в буфер. Возвращает `1` при успехе, `0` при ошибке. |
| `SendReportByEmail(reportName, recipients)` | recipients: String (адреса через `;`) | Integer | Формирует отчёт, конвертирует в HTML и отправляет по электронной почте. Возвращает `1` при успехе. |
| `SendReportByEmail(reportName, recipients, dbLine)` | dbLine: Integer | Integer | Отправка отчёта для указанной базовой строки. |
| `SendReportByEmail(reportName, recipients, dbLine, subject)` | subject: String | Integer | Отправка с указанной темой письма. |
| `SendReportByEmail(reportName, recipients, dbLine, subject, params)` | params: String или CIParameters | Integer | Отправка отчёта с параметрами. |
| `SendReportByEmail(reportName, recipients, dbLine, subject, params, fromName, fromEmail)` | fromName, fromEmail: String | Integer | Отправка с явным отправителем. |
| `SendReportAsExcelByEmail(fromEmail, fromName, recipients, subject, body, reportName, dbLine, fileName, invisible)` | fromEmail/fromName/recipients/subject/body/reportName/fileName: String; dbLine: Integer; invisible: Integer (1 — без интерфейса, 0 — с интерфейсом) | Integer | Формирует отчёт, сохраняет в XLSX и отправляет по почте. |
| `ExportXlsx(fileName, openAfter, reportOrSheet...)` | fileName: String; openAfter: Boolean; далее чередуются ссылки на отчёты и имена листов (необязательные — по умолчанию `"List1"`, `"List2"`, …) | Boolean | Экспортирует несколько отчётов на разных листах одного файла XLSX. |

## Сетевое взаимодействие класса System в языке EME-L

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `HttpRequest(serverIP, port, url, timeout, buffer, tag, [param...])` | serverIP: String; port: Integer; url: String; timeout: Integer (секунды, `0` — по умолчанию); buffer: CIString или CIDataStorage; tag: String (пустая — вернуть всю страницу); `param...`: пары `имя=значение`, пустое имя — тело запроса | String | Отправляет HTTP-запрос. Возвращает `"Ok"` при успехе, новый URL при перенаправлении или текст ошибки. |
| `HttpsRequest(serverIP, port, url, timeout, buffer, tag, [param...])` | аналогично `HttpRequest` | String | Отправляет HTTPS-запрос. |
| `WEBRequest(serverIP, port, timeout, params, [buffer])` | serverIP: String; port: Integer; timeout: Integer; params: CIParameters; buffer: CIString/CIDataStorage (необязательный) | String | HTTP/HTTPS-запрос в новом формате. Параметры `CIParameters`: без префикса — в запрос; `Header.<имя>` — в заголовок; `Request.SSL` (`0` — HTTP, `1` — HTTPS, `2` — HTTPS без проверки сертификатов); `Request.URL`; `Request.Tag`; `Request.Answer` (`1` — ждать ответ). |
| `HttpPostFile(serverIP, port, url, timeout, fileData, [fileName], [buffer])` | fileData: String; fileName: String; buffer: CIString | String | Отправляет файл методом POST на HTTP-сервер. |
| `HttpsPostFile(serverIP, port, url, timeout, fileData, [fileName], [buffer])` | аналогично `HttpPostFile` | String | Отправляет файл методом POST на HTTPS-сервер. |
| `WebService(serverIP, port, serviceName, methodName, answerTree, timeout, [param...])` | serverIP/serviceName/methodName: String; port/timeout: Integer; answerTree: CIPropertyTree (ответ в JSON); `param...`: пары `имя=значение` | String | Вызывает web-сервис через сервис ЕМЕ. Возвращает `"Ok"` или текст ошибки. |
| `GetConnectedUsers(userRefs, [computerNames], [connectionIDs])` | userRefs: CIArray; computerNames/connectionIDs: CIArray (необязательные) | Integer | Возвращает количество подключённых пользователей сервера EME; заполняет переданные массивы. |

## Файловая система и окружение в языке EME-L

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetCurrentDir()` | — | String | Возвращает текущий рабочий каталог приложения EME. |
| `SetCurrentDir(directory)` | directory: String | Boolean | Устанавливает текущий рабочий каталог. |
| `CreateDir(directory)` | directory: String | Empty | Создаёт каталог, включая промежуточные подкаталоги. |
| `GetMetaprojDir()` | — | String | Директория метапроекта EME. |
| `GetMetaprojExeDir()` | — | String | Директория исполняемых файлов метапроекта. |
| `GetKernelDir()` | — | String | Директория динамических библиотек ядра EME. |
| `GetKernelExeDir()` | — | String | Директория исполняемых файлов ядра. |
| `GetKernelResourceDir()` | — | String | Директория ресурсов ядра. |
| `GetDataDir()` | — | String | Директория данных проекта EME. |
| `GetDbDir()` | — | String | Корневой каталог базы данных EME. |
| `GetArchiveDir()` | — | String | Директория архива базы данных. |
| `DeleteOldFiles(directory, beforeDate, mask)` | directory: String; beforeDate: Date; mask: String (`""` — `*.*`) | Integer | Удаляет файлы, созданные раньше `beforeDate`. Возвращает количество удалённых файлов. |
| `GetEnvironmentVariable(name)` | name: String | String | Возвращает значение переменной окружения; пустое значение, если не найдена. |
| `System(command)` | command: String | Integer | Выполняет системную команду через стандартную функцию `system`. |

## Системная информация, принтеры, почта в системе EME

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetSystemPrintersAsArray()` | — | Array | Возвращает массив имён установленных принтеров. |
| `GetNameDefaultPrinter()` | — | String | Имя принтера по умолчанию; пустая строка, если не обнаружен. |
| `GetPrinterUserName(printerName)` | printerName: String | String | Имя принтера в формате `"Имя_принтера на Сервер_принтера"`. |
| `GetOrganizationName()` | — | String | Название проекта (организации) EME. |
| `SetOrganizationName(name)` | name: String | Empty | Устанавливает название проекта и обновляет заголовок главного окна. |
| `UpdateWindowHeader()` | — | Empty | Обновляет заголовок главного окна приложения EME. |
| `NewMailMessage(recipientName, recipientAddress, subject, body, attachments)` | recipientName/recipientAddress/subject/body: String; attachments: CIArray | Boolean | Открывает в почтовом клиенте новое письмо с заполненными полями. |

## Серверные семафоры класса System в языке EME-L

Серверные семафоры в языке EME-L защищают особо важные процедуры от одновременного выполнения несколькими пользователями. Аргумент `record` — номер записи базы данных EME или её наименование (строка или целое число).

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetSemaphore(record)` | record: String/Integer | Empty или String | Устанавливает семафор. Возвращает пустое значение при успехе; если запись уже заблокирована — текст с информацией о пользователе, занявшем семафор. |
| `CheckSemaphore(record)` | record: String/Integer | Empty или String | Проверяет семафор. Пустое значение — свободен; иначе — текст о владельце. |
| `DeleteSemaphore(record)` | record: String/Integer | Empty | Удаляет семафор. |

## События, резидент, умный календарь и прочие сервисы EME-L

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `MakeEvent(eventType, [param...])` | eventType: String; param...: произвольные | Empty | Активирует событие указанного типа в системе EME. |
| `CallResident(methodName, [param...])` | methodName: String; param...: произвольные | произвольный | Вызывает метод резидента проекта EME. |
| `Sleep(milliseconds)` | milliseconds: Integer | Empty | Приостанавливает выполнение текущего потока. |
| `Dump(text)` | text: String | Empty | Выводит строку в файл дампа. |
| `AddSmartDate(date, hint)` | date: Date; hint: String | Empty | Добавляет общую «умную дату» с текстом подсказки для диалога календаря EME-L. |
| `AddSmartDate(date, hint, backColor, textColor)` | backColor, textColor: Integer (RGB) | Empty | Добавляет умную дату с заданными цветами фона и текста. |
| `RemoveSmartDate(date)` | date: Date | Boolean | Удаляет умную дату. `TRUE`, если дата была и удалена. |
| `RemoveAllSmartDates()` | — | Empty | Удаляет все общие умные даты. |

## Примеры использования класса System в языке EME-L

### Кэш запросов и «гребенчатая» схема базы данных EME

```EME-L
'Инициализировать кэш запросов перед тяжёлым расчётом'
Object("System").InitQueryCache();

'Выполнить запрос, использующий кэш'
query = Object("Query", "MyReport.Data");
query.Execute();

'Отключить "гребёнку" внутри транзакции для сплошной записи'
is_transaction(1, tr("Пересчёт коэффициентов"));
Object("System").EnableComb(FALSE);
'...цикл записи в БД...'
Object("System").EnableComb(TRUE);
is_transaction(-1);

'Очистить кэш после использования'
Object("System").ClearQueryCache();
```

### Диалог выбора файла в EME-L

```EME-L
'Получить путь к Excel-файлу от пользователя'
FileName = Object("System").BrowseForFile(
    tr("Файлы") + " Excel (*.xls;*.xlsx)|*.xls;*.xlsx||",
    "");

If (FileName == "")
    Return;
End If
```

### Диалог выбора имени файла с параметрами

```EME-L
'Диалог "Сохранить как" с подставленным именем файла'
path = Object("System").GetFileNameDialog(
    "Все файлы (*.*)|*.*||",
    "",
    FALSE,
    "",
    r_File.GetName() + ".txt");
```

### Печать отчёта на заданный принтер

```EME-L
'Напечатать отчёт напрямую на указанный принтер в отдельном потоке'
ok = Object("System").RunReportIndirectInPrinter(
    "MyReport.Layout",
    Printer,
    dbLine,
    1,
    "",
    "",
    0,
    TRUE);
```

### HTTP-запрос в новом формате через WEBRequest

```EME-L
'Вызвать внешний HTTP-сервис через WEBRequest'
Params = Object("Parameters");
Params.Request.SSL = 0;
Params.Request.URL = "/api/data";
Params.Request.Answer = 1;
Params.Header."Content-Type" = "application/json";
Params."field" = "value";

Result = Object("System").WEBRequest(IP, Port, TimeOut, Params, objResult);
```

### Семафор для защиты записи документа EME

```EME-L
'Попытаться занять семафор для записи документа'
Holder = Object("System").SetSemaphore(nDocRecord);
If (Holder != "")
    is_message("!", tr("Документ заблокирован пользователем %s", Holder), 1, 3);
    Return;
End If

'...критическая секция работы с записью...'

Object("System").DeleteSemaphore(nDocRecord);
```

### Всплывающее сообщение в статус-баре EME

```EME-L
'Показать "облако" на 3 секунды'
Object("System").ShowCloud(tr("Операция завершена"), 3);
```

## Замечания по реализации класса System в языке EME-L

- Методы `EnterTransactionSection()` и `LeaveTransactionSection()` объявлены в классе System в языке EME-L, но в текущей реализации не выполняют действий — оставлены для совместимости.
- Метод `SetExecuteInMainThread(flag)` класса System также зарезервирован и не влияет на выполнение в текущей версии EME.
- Метод `SetBlockReadModeGlobalState` присутствует в исходном коде C++ закомментированным (отключён разработчиком #ANK 16.12.2015) и недоступен из скриптов EME-L.
- Диагностический метод `TestWorkerDataStructure()` возвращает `0` и используется только для внутренних проверок структуры данных раздела EME-L.
- Метод `BrowseForFile` при ошибке или отмене возвращает пустое значение (Empty), а не пустую строку — в отличие от `BrowseForFolder`, который возвращает пустую строку.
- Перегрузка `RunReportIndirectInPrinter(...)` с 9 аргументами (только `orientation`) доступна только при включённой compile-time опции `PRINT_REPORT_INDIRECT_PARAMS_ORIENTATION`; перегрузка с 10 аргументами (`orientation` + `duplex`) доступна безусловно.
