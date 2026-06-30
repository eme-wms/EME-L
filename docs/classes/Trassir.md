Trassir CITrassir Object("Trassir") SendEvent SendEventMoveOrder SendEventOrderControl SendEventAcceptingDocLines SendEventAcceptingIncomingLines SendEventInventoryChecked SendEventShipment SendEventOrderControlPacker GetChannelsFromLocation GetAndSaveVideo GetAndShowVideo SetFileCreationTimeoutMs SetFileDownloadTimeoutPerMinuteMs POSNG_STOREHOUSE_FROMLOCATION POSNG_STOREHOUSE_TOLOCATION POSNG_STOREHOUSE_SELECTION POSNG_STOREHOUSE_CHECK_SELECTED POSNG_STOREHOUSE_ACCEPTING POSNG_STOREHOUSE_SSCC POSNG_STOREHOUSE_SHIPMENT POSNG_STOREHOUSE_NVENTORY trassir_locations.txt

# Класс Trassir

Класс `Trassir` в языке EME-L — класс для интеграции с системой видеонаблюдения Trassir. В системе EME.WMS он предназначен для отправки событий складских операций на сервер видеонаблюдения Trassir через TCP-соединение и для работы с видеоархивом сервера.

В языке EME-L объект класса `Trassir` (CI-имя `CITrassir`) создаётся через оператор `Object("Trassir")`. При создании из настроек модулей считываются параметры `Видеонаблюдение.TRASSIR.Отправлять события на сервер`, `Видеонаблюдение.TRASSIR.IP Адрес` и `Видеонаблюдение.TRASSIR.Порт`. Если отправка событий отключена или IP-адрес не задан, объект переходит в режим отказа от отправки, и все вызовы методов отправки событий в языке EME-L завершаются без действия.

Класс в языке EME-L предоставляет методы для передачи данных о приёмке, перемещении, инвентаризации, отгрузке и контроле заказов. Каждое событие формируется в виде XML-сообщения и отправляется на сервер Trassir в отдельном потоке. Кроме того, в языке EME-L доступно получение видеоархива с сервера Trassir по временному интервалу и каналу, а также поиск каналов по локации через файл `trassir_locations.txt` в общей директории с сервером.

## Создание объекта

В языке EME-L класс `Trassir` имеет единственный конструктор без аргументов.

```EME-L
'Создать объект для интеграции с Trassir в языке EME-L'
objTrassir = Object("Trassir");
```

При создании объекта `Trassir` в системе EME.WMS выполняется чтение настроек модулей. Если параметр `Видеонаблюдение.TRASSIR.Отправлять события на сервер` не равен `Да` или `Видеонаблюдение.TRASSIR.IP Адрес` пуст, дальнейшая отправка событий блокируется.

## Отправка событий

В языке EME-L методы этой группы формируют XML-сообщения о складских операциях и отправляют их на сервер Trassir. Если в настройках модулей отправка событий запрещена, методы завершаются без действия.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SendEvent(arg0, arg1, arg2, arg3, arg4, arg5, arg6, arg7, arg8, arg9, arg10, arg11, arg12)` | arg0: String (тип события), arg1: String (идентификатор операции), arg2: String (кассир/оператор), arg3: Date (дата), arg4: Time (время), arg5: String (локация), arg6: String (текст), arg7: String (позиция), arg8: String (количество), arg9: String (вес), arg10: String (цена), arg11: String (штрихкод), arg12: String (артикул) | Empty | В языке EME-L отправляет произвольное событие на сервер Trassir в виде XML-сообщения. Все 13 параметров обязательны на уровне арности; для необязательных полей передаётся пустая строка. Если дата или время не заданы, в XML подставляется текущая дата и время. |
| `SendEventMoveOrder(arg0, arg1)` | arg0: Integer (ссылка на приказ на перемещение), arg1: Boolean (True — начало, False — завершение) | Empty | В языке EME-L отправляет событие о начале или завершении приказа на перемещение. |
| `SendEventOrderControl(arg0)` | arg0: Integer (ссылка на строку распределения количества по заказам) | Empty | В языке EME-L отправляет событие контроля заказа на столе. Локация берётся из модуля `Видеонаблюдение.TRASSIR.Локация рабочей станции`. |
| `SendEventAcceptingDocLines(arg0)` | arg0: Integer (ссылка на строку документа) | Empty | В языке EME-L отправляет событие приёмки товара по строкам документа. |
| `SendEventAcceptingIncomingLines(arg0)` | arg0: Integer (ссылка на строку прихода) | Empty | В языке EME-L отправляет событие приёмки товара по строкам прихода. |
| `SendEventInventoryChecked(arg0, arg1)` | arg0: Integer (ссылка на приказ на перемещение), arg1: Integer (ссылка на ячейку) | Empty | В языке EME-L отправляет событие инвентаризации с ТСД по ячейке. |
| `SendEventShipment(arg0, arg1)` | arg0: Integer (ссылка на приказ на перемещение), arg1: String (SSCC) | Empty | В языке EME-L отправляет событие отгрузки заказа с ТСД. В реализации C++ формируются два XML-сообщения: с типом `POSNG_STOREHOUSE_SSCC` и с типом `POSNG_STOREHOUSE_SHIPMENT`. |
| `SendEventOrderControlPacker(arg0)` | arg0: Integer (ссылка на приказ на перемещение) | Empty | В языке EME-L отправляет событие контроля заказа упаковщиком. Локация берётся из зоны назначения заказа. |

## Работа с видеоархивом

В языке EME-L методы этой группы загружают видеофрагменты с сервера Trassir через HTTPS-протокол и работают с файлом `trassir_locations.txt` в общей директории, настроенной в модуле `Видеонаблюдение.TRASSIR.Папка с общим доступом с сервером`. Для загрузки видео в системе EME.WMS дополнительно требуются параметры `Видеонаблюдение.TRASSIR.Порт SDK` и `Видеонаблюдение.TRASSIR.Пароль SDK`.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetChannelsFromLocation(arg0, arg1)` | arg0: String (название локации), arg1: Boolean (True — выводить сообщения об ошибках) | String | В языке EME-L возвращает строку с идентификаторами каналов, разделёнными символом `;`. Первый элемент — GUID интерфейса оператора, остальные — каналы локации. Если локация не найдена, возвращает пустую строку. |
| `GetAndSaveVideo(arg0, arg1, arg2, arg3, arg4, arg5)` | arg0: Date (дата начала), arg1: Time (время начала), arg2: Date (дата окончания), arg3: Time (время окончания), arg4: String (GUID интерфейса оператора), arg5: String (GUID или название канала) | Empty | В языке EME-L загружает видеофрагмент и сохраняет его в файл AVI в общей директории. Если файл с таким именем уже существует, к имени добавляется порядковый номер в скобках. После завершения выводит сообщение с именем файла. |
| `GetAndShowVideo(arg0, arg1, arg2, arg3, arg4, arg5)` | те же, что `GetAndSaveVideo` | Empty | В языке EME-L загружает видеофрагмент во временный файл `Video.avi` и автоматически воспроизводит его стандартным проигрывателем. Временный файл удаляется перед следующим вызовом. |
| `SetFileCreationTimeoutMs(arg0)` | arg0: Integer (таймаут в миллисекундах) | Empty | В языке EME-L устанавливает таймаут ожидания создания временного файла видео. `0` — не ожидать. По умолчанию `500` мс. |
| `SetFileDownloadTimeoutPerMinuteMs(arg0)` | arg0: Integer (таймаут в миллисекундах на каждую минуту видео) | Empty | В языке EME-L устанавливает таймаут загрузки одной минуты видео после появления временного файла. `0` — не ожидать и не воспроизводить. По умолчанию `3000` мс. |

## Константы и параметры

В языке EME-L и в системе EME.WMS класс `Trassir` использует следующие значения и настройки (взяты из реализации C++):

| Константа / параметр | Значение | Назначение |
|----------------------|----------|------------|
| `POSNG_STOREHOUSE_FROMLOCATION` | строка | Тип события для начала перемещения (включая PPU) в языке EME-L. |
| `POSNG_STOREHOUSE_TOLOCATION` | строка | Тип события для завершения перемещения (не PPU) в языке EME-L. |
| `POSNG_STOREHOUSE_SELECTION` | строка | Тип события для завершения PPU-перемещения в языке EME-L. |
| `POSNG_STOREHOUSE_CHECK_SELECTED` | строка | Тип события для контроля заказа на столе и для контроля упаковщиком в языке EME-L. |
| `POSNG_STOREHOUSE_ACCEPTING` | строка | Тип события для приёмки по строкам документа и прихода в языке EME-L. |
| `POSNG_STOREHOUSE_SSCC` | строка | Тип события для SSCC при отгрузке в языке EME-L. |
| `POSNG_STOREHOUSE_SHIPMENT` | строка | Тип события для заказа при отгрузке в языке EME-L. |
| `POSNG_STOREHOUSE_NVENTORY` | строка | Тип события для инвентаризации в C++-исходнике (опечатка — пропущена буква `I`). |
| `m_lFileCreationTimeoutMs` | `500` мс | Таймаут создания временного файла видео по умолчанию в системе EME.WMS. |
| `m_lFileDownloadTimeoutPerMinuteMs` | `3000` мс | Таймаут загрузки одной минуты видео по умолчанию в системе EME.WMS. |
| `Видеонаблюдение.TRASSIR.Отправлять события на сервер` | `Да` / `Нет` | Главный выключатель отправки событий в системе EME.WMS. |
| `Видеонаблюдение.TRASSIR.IP Адрес` | строка | IP-адрес или имя хоста сервера Trassir в системе EME.WMS. |
| `Видеонаблюдение.TRASSIR.Порт` | целое число | Порт TCP-сервера событий Trassir в системе EME.WMS. |
| `Видеонаблюдение.TRASSIR.Порт SDK` | целое число | Порт HTTPS SDK сервера Trassir (в коде по умолчанию `8080`) в системе EME.WMS. |
| `Видеонаблюдение.TRASSIR.Пароль SDK` | строка | Пароль для HTTPS SDK в системе EME.WMS. |
| `Видеонаблюдение.TRASSIR.Папка с общим доступом с сервером` | строка | Путь к общей директории с файлом `trassir_locations.txt` и сохранением видео в системе EME.WMS. |
| `Видеонаблюдение.TRASSIR.Локация рабочей станции` | строка | Локация, используемая методом `SendEventOrderControl` в системе EME.WMS. |
| Подстановка текущей даты/времени | текущие дата и время | В методе `SendEvent` при передаче пустой даты подставляются текущие дата и время в системе EME.WMS. |

## Примеры

В языке EME-L отправка события инвентаризации с ТСД:

```EME-L
'Создать объект Trassir в языке EME-L'
objTrassir = Object("Trassir");

'Отправить событие инвентаризации по ячейке в языке EME-L'
objTrassir.SendEventInventoryChecked(r_MoveOrder.GetLine(), r_Cell.GetLine());
```

В языке EME-L отправка событий отгрузки по заказам:

```EME-L
'Создать объект Trassir в языке EME-L'
objTrassir = Object("Trassir");

'Пройти по карте заказов и отправить событие для каждого SSCC в языке EME-L'
Loop(mapOrders)
    r_Order.SetLine(mapOrders.GetKey());
    strSSCC = mapOrders.GetValue();
    objTrassir.SendEventShipment(r_Order.GetMoveOrderRef(), strSSCC);
End Loop
```

В языке EME-L получение списка каналов и воспроизведение видео:

```EME-L
'Создать объект Trassir в языке EME-L'
objTrassir = Object("Trassir");

'Получить каналы локации без вывода ошибок в языке EME-L'
strChannels = objTrassir.GetChannelsFromLocation("PackingZone", False);

'Первый элемент — GUID интерфейса оператора в языке EME-L'
objChannels = Object("String", strChannels);
strOperatorGUI = objChannels.ExtractUpto(";");
strChannel = objChannels.ExtractUpto(";");

'Установить таймауты для загрузки видео в языке EME-L'
objTrassir.SetFileCreationTimeoutMs(5000);
objTrassir.SetFileDownloadTimeoutPerMinuteMs(20000);

'Воспроизвести видеофрагмент в языке EME-L'
objTrassir.GetAndShowVideo(startDate, startTime, finishDate, finishTime, strOperatorGUI, strChannel);
```

## Расхождения с реализацией C++

В файле `i_Trassir.cpp` обнаружены следующие расхождения между регистрацией методов и их реализацией:

- **`SendEventShipment` формирует два XML-события, а не одно.** В реестре метод описан как «Отправляет событие ... при отгрузке заказа с ТСД» (единственное число). Тело метода последовательно вызывает `SendEvent` с типом `POSNG_STOREHOUSE_SSCC` (передаёт SSCC) и с типом `POSNG_STOREHOUSE_SHIPMENT` (передаёт пустой текст). Таким образом, на сервер уходит два XML-сообщения.
- **Опечатка в типе события инвентаризации.** В теле `SendEventInventoryChecked` тип события задаётся строкой `"POSNG_STOREHOUSE_NVENTORY"` (пропущена буква `I` в слове `INVENTORY`). В документации это сохранено как фактическое значение из C++-исходника.
- **Ошибка обрезки завершающего слэша в `GetVideo`.** В строке 1197 метода `TrassirSDKHttpsCommunication::GetVideo` используется `csDirectory = m_csDirectory.Left(csDirectory.GetLength() - 1);` вместо `csDirectory = csDirectory.Left(csDirectory.GetLength() - 1);`. На момент выполнения член `m_csDirectory` ещё не инициализирован, поэтому при конфигурации папки с завершающим слэшем путь обрезается неверно, и последующие операции с файлом могут завершиться ошибкой.

## См. также

- [Класс String](./String.md) — работа с разбором строки каналов через `ExtractUpto(";")` в языке EME-L.
