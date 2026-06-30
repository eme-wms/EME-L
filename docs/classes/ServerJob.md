ServerJob CIServerJob SendJob GetJobStatus GetJobResult GetJobReturnValue GetJobNo GetJobId GetJobName GetJobQueue GetJobLength IsJobsComplete CancelJob FindJob GetJobReturnType EnumJobReturnValues GetLastError Close GetStatusAsString SetTimeout SetJobLimit SetConcurrentTransactions GetServerLog GetJobLog GetJobArgValue GetJobArgType GetJobPropertyValue GetJobParameters TRANSACTION REPORT TRANSACTION:SHARE TRANSACTION:BATCH LIFELENGTH SILENT

### Класс ServerJob — клиент очереди заданий HTTP-сервера

Класс `ServerJob` (C++ `CIServerJob`) в языке EME-L — клиент для отправки и управления заданиями на HTTP-сервере транзакций и обычном HTTP-сервере. Предоставляет механизм удалённого выполнения методов EME-L-классов на сервере через очередь заданий.

В системе EME.WMS класс `ServerJob` используется для выноса длительных вычислений и отчётов на сервер, разгрузки рабочей станции и организации фоновой обработки данных. Поддерживает два типа заданий: **REPORT** (отчёты без открытия транзакции) и **TRANSACTION** (транзакционные операции с базой данных). Задания ставятся в очередь на сервере, выполняются последовательно или параллельно, а результаты возвращаются на рабочую станцию по запросу.

Класс позволяет отправлять задания с произвольным набором аргументов (включая сериализуемые объекты), получать статус, номер в очереди, результат и значения возвращаемых переменных. В языке EME-L поддерживаются расширенные режимы: `TRANSACTION:SHARE` — общая транзакция для нескольких заданий; `TRANSACTION:BATCH` — пакетное выполнение с массивом параметров.

Объект класса `ServerJob` создаётся через `Object("ServerJob", …)`. Класс клиентский — он отправляет HTTP-запросы на сервер заданий, где исполнительный поток опрашивает очередь один раз в секунду. Таймаут запросов по умолчанию — **5000 мс**.

### Создание объекта

В языке EME-L класс `ServerJob` имеет четыре перегрузки конструктора (по числу аргументов от 0 до 3).

```EME-L
'Без аргументов: автоопределение адреса и портов сервера в сети'
objJob = Object("ServerJob");
```

Конструктор без аргументов определяет адрес и порты автоматически поиском HTTP-сервера транзакций в сети. Если сервер не обнаружен, генерируется ошибка. В монопольном режиме отправка запросов на сервер заданий запрещена.

```EME-L
'Только IP-адрес: порт запросов по умолчанию 80'
objJob = Object("ServerJob", "192.168.1.100");
```

Порт запросов по умолчанию равен **80**. Порт транзакций принимается равным порту запросов. Задание отправляется на обычный порт — на указанном сервере обязательно должен быть запущен HTTP-сервер транзакций.

```EME-L
'IP-адрес и порт запросов'
objJob = Object("ServerJob", "192.168.1.100", 8080);
```

Порт транзакций принимается равным порту запросов (раздельные серверы не поддерживаются этой перегрузкой).

```EME-L
'IP-адрес, порт запросов и порт транзакций — для раздельных серверов'
objJob = Object("ServerJob", "192.168.1.100", 8080, 8081);
```

При раздельных серверах (когда `reportPort != transactionPort`) используется схема чётных/нечётных идентификаторов: задания REPORT получают чётные идентификаторы, TRANSACTION — нечётные. Внутренний номер задания получается делением идентификатора на 2 с округлением.

### Отправка задания — SendJob

`SendJob` — основной метод класса `ServerJob` в языке EME-L. Отправляет задание на HTTP-сервер для выполнения в очереди. Задание не выполняется немедленно, а ставится в очередь, продвижение которой происходит один раз в секунду.

`SendJob(jobType, jobName[, jobParams], className, methodName[, arg1[, arg2[, ...]]])` → Integer

| Параметр | Тип | Обязательный | Описание |
|----------|-----|-------------|----------|
| jobType | String | Да | Тип задания. `"TRANSACTION"` — транзакционное; `"REPORT"` — отчёт; `"TRANSACTION:SHARE"` — общая транзакция; `"TRANSACTION:BATCH"` — пакетное. |
| jobName | String | Да | Имя задания. Уникальное для транзакций, уникальное в рамках компьютера для отчётов. |
| jobParams | Parameters | Нет | Объект `Parameters` с настройками задания. Для совместимости может быть целым числом — тогда трактуется как `LIFELENGTH`. |
| className | String | Да | Имя EME-L-класса, метод которого будет выполнен. |
| methodName | String | Да | Имя метода для выполнения. |
| argN | произвольный | Нет | Аргументы вызова метода. Должны быть сериализуемы. |

**Возвращает**: `Integer` — идентификатор задания. Чётный для REPORT, нечётный для TRANSACTION. -1 при ошибке отправки. Минимальное число аргументов вызова — 5 (тип, имя, класс, метод и хотя бы ноль аргументов); при `argn < 4` ядро генерирует «слишком мало аргументов» (проверка `too_rare()` в теле `CIServerJob::SendJob`).

В языке EME-L при использовании `"TRANSACTION:SHARE"` само задание не должно открывать и закрывать транзакцию. При использовании `"TRANSACTION:BATCH"` задание может иметь только один параметр.

Параметры задания (объект `Parameters` в языке EME-L):

| Параметр | Тип | По умолчанию | Описание |
|----------|-----|-------------|----------|
| LIFELENGTH | Integer | 60 | Время хранения задания после выполнения, в минутах. |
| SILENT | Boolean | FALSE | Подавить уведомления о выполнении. |

### Настройка таймаута — SetTimeout

`SetTimeout(timeout)` → Empty

| Параметр | Тип | Обязательный | Описание |
|----------|-----|-------------|----------|
| timeout | Integer | Да | Таймаут запросов к серверу в миллисекундах. По умолчанию 5000 мс. |

### Статус и очередь

Методы класса `ServerJob` в языке EME-L для контроля состояния задания. Статус — целое число от 0 до 5.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetJobStatus(jobId)` | jobId: Integer/String | Integer/Empty | 0 — Создано, 1 — В очереди, 2 — Выполняется, 3 — Выполнено, 4 — Закрыто, 5 — Ошибка. Empty — не найдено. |
| `GetStatusAsString(status)` | status: Integer | String | «Создано», «В очереди», «Выполняется», «Выполнено», «Закрыто», «Ошибка», «Неизвестно». |
| `GetJobQueue(jobId)` | jobId: Integer/String | Integer/Empty | Позиция в очереди (с 0). Отрицательное — выполнено. Empty — не найдено. |
| `GetJobLength(jobId)` | jobId: Integer/String | Integer/Empty | Чистая продолжительность в мс. 0 — не выполнено. Empty — не найдено. |
| `IsJobsComplete(jobId)` | jobId: Integer/String/Array | Boolean | TRUE — задание или все задания массива выполнены. При неверном номере — исключение. |
| `CancelJob(jobId)` | jobId: Integer/String | Boolean/Empty | TRUE — отменено. Empty — не найдено или уже выполнено. Поведение зависит от состояния: при статусе моложе «Выполняется» задание переводится в «Ошибка» с результатом «Выполнение задания отменено»; если поток уже начал выполнение, ядро пытается остановить его через `SetStopExecution(TRUE)` (версия ядра ≥ 46.0) и возвращает TRUE, если поток найден. |

### Идентификация и поиск задания

В языке EME-L идентификатор задания — целое число, номер — строка в формате `dd-hh:mm-NNNNNN`. Метод `GetJobId` статический и не обращается к серверу.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetJobId(jobNo)` | jobNo: String | Integer | Числовой идентификатор по номеру-строке. |
| `GetJobNo(jobId)` | jobId: Integer | String/Empty | Номер-строка по идентификатору. |
| `GetJobName(jobId)` | jobId: Integer/String | String/Empty | Имя задания. |
| `FindJob(jobType, jobName)` | jobType: String, jobName: String | Integer | Идентификатор по типу и имени. -1 — не найдено. |

### Результат выполнения

В языке EME-L результат задания — значение, возвращённое исполнительным методом. Переменные результата должны быть определены в конструкторе вызываемого класса как `AsExternal`.

`GetJobResult(jobId)` → Number/String/Empty — основной результат (простой тип). Объекты не возвращаются. Для нескольких значений используйте `GetJobReturnValue` и `EnumJobReturnValues`.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetJobReturnValue(jobId, variableName)` | jobId: Integer/String, variableName: String | Number/String/Empty | Значение переменной по имени. Только простые типы. Если объект задания временно недоступен (другой поток читает результат), сервер вместо возврата данных записывает в `m_csResult` строку «Задание временно недоступно. Попробуйте позже» — её возвращает `GetLastError`, а `GetJobReturnValue` вернёт Empty. |
| `GetJobReturnValue(jobId, variableName, returnObjects)` | + returnObjects: Boolean (по умолч. FALSE) | произвольный/Empty | То же с возвратом объектов (Array, Map, Query, сериализуемый класс). Вызывать до `Close`. |
| `GetJobReturnType(jobId, variableName)` | jobId: Integer/String, variableName: String | String/Empty | Тип значения переменной. |
| `EnumJobReturnValues(jobId)` | jobId: Integer/String | Array/Empty | Массив имён переменных-результатов. Вызывать до `Close`. |
| `GetLastError(jobId)` | jobId: Integer/String | String/Empty | «OK» при успехе, текст ошибки при ошибке, Empty — не найдено. |
| `Close(jobId)` | jobId: Integer/String | Boolean | Переводит в «Закрыто» и удаляет задание с сервера. TRUE — успешно. |

### Аргументы и свойства задания

Методы класса `ServerJob` для доступа к аргументам вызова и свойствам задания.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetJobArgValue(jobId, argNo)` | jobId: Integer/String, argNo: Integer | Number/String/Empty | Значение аргумента вызова по индексу (с 0). Простые типы. |
| `GetJobArgValue(jobId, argNo, returnObjects)` | + returnObjects: Boolean | произвольный/Empty | То же с возвратом объектов. |
| `GetJobArgType(jobId, argNo)` | jobId: Integer/String, argNo: Integer | String/Empty | Тип аргумента. Реестр объявляет 2 параметра, но реализация клиента и сервера читают третий аргумент `returnObjects` (Boolean, по умолчанию FALSE) — фактически метод поддерживает необязательный третий параметр, влияющий только на формат ответа сервера. |
| `GetJobPropertyValue(jobId, propertyName)` | jobId: Integer/String, propertyName: String | произвольный/Empty | Значение свойства задания. |
| `GetJobParameters(jobId)` | jobId: Integer/String | String/Empty | Параметры задания строкой. |

Свойства `GetJobPropertyValue`: `ARGN` (Integer, число аргументов), `CLASS_NAME` (String), `METHOD_NAME` (String), `TYPE` (String, тип задания), `COMPUTER` (String), `USER` (String), `EXE_DATE_TIME` (DateTime, вычисляется приблизительно по `GetTickCount()` на сервере — не хранится как абсолютное время; может расходиться с реальным временем старта на время работы сервера после перезагрузки системы; если задание не выполнено, возвращает пустую дату), `LIFE_LENGTH` (Integer, минуты).

### Журнал сервера заданий

В языке EME-L методы `GetServerLog` и `GetJobLog` возвращают объект `Query` с записями журнала. Журнал `GetServerLog` объединяет записи с обычного сервера и сервера транзакций.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetServerLog()` | — | Query | Полный журнал сервера. |
| `GetServerLog(seconds)` | seconds: Integer | Query | Журнал за последние N секунд. |
| `GetJobLog(jobId)` | jobId: Integer/String | Query/Empty | Журнал одного задания. |

Поля объекта `Query` из журнала сервера заданий: `JobNo` (TEXT), `Name` (TEXT), `Date` (DATE), `Time` (TIME), `Computer` (TEXT), `User` (TEXT), `Status` (INT), `LastError` (TEXT).

### Лимиты и параллельность

В языке EME-L методы управления лимитами класса `ServerJob`.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetJobLimit(jobName, limit)` | jobName: String, limit: Integer | Integer | Лимит одновременно выполняемых заданий с именем. Только для REPORT. 0 — без ограничения. Возвращает предыдущее значение. |
| `SetConcurrentTransactions(allow)` | allow: Boolean | Boolean | Разрешить/запретить параллельные транзакционные задания. По умолчанию FALSE. Возвращает предыдущее значение. |

### Примеры

Отправка транзакционного задания с параметрами и ожидание результата в языке EME-L:

```EME-L
objJob     = Object("ServerJob");
jobParams  = Object("Parameters");
jobParams.PutParam("LIFELENGTH", 30);
jobParams.PutParam("SILENT", True);

JobId = objJob.SendJob("TRANSACTION:SHARE", "Расчёт заказов", jobParams, "OrdersBatch", "ProcessOrders", OrderId);

If (objJob.IsJobsComplete(JobId))
    strResult = objJob.GetJobResult(JobId);
    objJob.Close(JobId);
End If
```

Поиск задания по имени и чтение журнала в системе EME.WMS:

```EME-L
objJob = Object("ServerJob");
JobId = objJob.FindJob("REPORT", "Движение товаров");

If (JobId != -1)
    QueuePos = objJob.GetJobQueue(JobId);
    JobLog   = objJob.GetServerLog(3600);
End If
```

### См. также

- [Parameters](./Parameters.md) — объект параметров задания.
- [Array](./Array.md) — массив идентификаторов для `IsJobsComplete`.
- [Query](./Query.md) — результат `GetServerLog` и `GetJobLog`.
