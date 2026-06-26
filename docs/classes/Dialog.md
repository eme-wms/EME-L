Dialog CIDialog Object("Dialog") Show ShowLine ShowNewLine Close Activate Hide SetLine GetLine GetTemplateLine GetRecord CreateLine DeleteLine IsNewLine GetControl GetFirstControl GetTable GetFocusedTable GetMainBrowser GetDBObject IsModified SetModified GetFlags SetFlags AddFlags IsWindow GetCaption SetCaption GetName SetWndFlags GetRect MoveWindow Update Disable DisableTabItem DisableAllTabItems IsTabItemActive SetTabItemActive InvisibleTabItem SetTabItemTextColor SetTabItemText GetTabItemText RunReport RunReportIndirect RunReportIndirectInPrinter SendReportByEmail SkipReports ShowReports SetSemaphore CheckSemaphore DeleteSemaphore GetParameters GetClassObject PutDialogVariable GetContext GetParent BreakExecute Save GetHelpID PutHelpID IsEnterSemaphore ShowPopupMessage SetControlTimer KillControlTimer SetControlDateRange LoadScannerDriver SendDriverRequest UnloadScannerDriver

### Dialog — класс диалоговых окон системы EME

Класс `Dialog` (C++ `CIDialog`) в языке EME-L — это класс для работы с диалоговыми окнами системы EME. Предоставляет программный доступ к созданию, открытию, управлению и закрытию диалоговых окон, привязанных к записям базы данных или работающих в независимом режиме. Скриптовое имя для конструирования: `Object("Dialog", ...)`.

Класс поддерживает открытие диалогов в модальном и немодальном режимах, навигацию по строкам записи, создание и удаление строк, управление флагами поведения диалога и внешним видом окна. Объект создаётся с привязкой к текущему контексту, по имени диалога, либо по имени объекта и члена класса.

В обработчиках диалога (`Dialog_OnInit`, `Dialog_OnAfterUpdate`, `Dialog_OnAfterSave` и т. п.) объект уже доступен как неявная переменная `Dialog` — повторная конструкция `Object("Dialog")` в таких случаях используется в полях класса для получения ссылки. Класс `Dialog` в системе EME.WMS — это центральный элемент интерактивного взаимодействия с пользователем.

#### Конструкторы

| Конструктор | Аргументы | Описание |
|-------------|-----------|----------|
| `Dialog()` | — | Создаёт объект диалога, подключённый к текущему контексту выполнения в языке EME-L. |
| `Dialog(name)` | name: String — имя шаблона диалога | Создаёт объект диалога по имени шаблона. |
| `Dialog(object_name, member_name)` | object_name: String, member_name: String | Создаёт объект диалога по имени объекта класса и имени члена класса. |
| `Dialog(object_name, member_name, context)` | + context: Integer — контекст вызова | То же с явным контекстом. |
| `Dialog(object_name, member_name, context, soft_mode)` | + soft_mode: Boolean | То же; soft_mode=TRUE — игнорировать отсутствие диалога и вернуть пустой объект, FALSE (по умолчанию) — генерировать исключение. |

#### Открытие и закрытие диалога

Методы открытия в языке EME-L используют параметр `mode`: 0 — модальный диалог (блокирует родительский интерфейс до закрытия), 1 — новый немодальный (открывается как самостоятельное окно). При модальном режиме возвращается результат выполнения (Integer), при немодальном — пустое значение.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `Show(mode)` | mode: Integer (0 — модальный, 1 — новый немодальный) | Integer / Empty | Открывает диалог на текущей строке записи. |
| `Show(mode, run_code)` | + run_code: Integer — код запуска модуля | Integer / Empty | То же с кодом запуска. |
| `Show(mode, run_code, params)` | + params: String — параметры запуска | Integer / Empty | То же с параметрами. |
| `ShowLine(mode, line)` | + line: Integer — номер строки записи | Integer / Empty | Открывает диалог на указанной строке. |
| `ShowLine(mode, line, run_code)` | + run_code: Integer | Integer / Empty | То же с кодом запуска. |
| `ShowLine(mode, line, run_code, params)` | + params: String | Integer / Empty | То же с параметрами. |
| `ShowNewLine(mode)` | mode: Integer | Integer / Empty | Открывает диалог и создаёт новую строку записи. |
| `ShowNewLine(mode, run_code)` | + run_code: Integer | Integer / Empty | То же с кодом запуска. |
| `ShowNewLine(mode, run_code, params)` | + params: String | Integer / Empty | То же с параметрами. |
| `Close()` | — | Empty | Закрывает диалоговое окно. |
| `Activate()` | — | Integer | Находит открытое окно по имени, обновляет данные и активизирует его. |
| `Activate(line)` | line: Integer | Integer | То же с указанием строки записи. |
| `Hide(hide)` | hide: Boolean | Empty | TRUE — скрыть окно, FALSE — показать. |

Флаги внешнего вида окна (`SetWndFlags`) нужно выставлять **до** вызова `Show`/`ShowLine`.

#### Строки и навигация

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetLine(line)` | line: Integer | Empty | Переключает диалог на указанную строку записи. Если в диалоге есть дерево, выделяет соответствующую ветвь. При передаче `NULL_REF` переключение не выполняется. |
| `GetLine()` | — | Integer | Номер текущей строки записи, на которой открыт диалог. |
| `GetTemplateLine()` | — | Integer | Номер строки шаблона диалога в записи DialogTemplates. |
| `GetRecord()` | — | Integer | Номер записи базы данных, к которой привязан диалог. |
| `CreateLine()` | — | Empty | Создаёт новую строку записи. Эквивалентно нажатию F6. |
| `DeleteLine()` | — | Empty | Удаляет текущую строку. Эквивалентно нажатию F8. |
| `IsNewLine()` | — | Boolean | TRUE, если диалог открыт на новой строке и это первое сохранение данных на диск. |

#### Органы управления, таблицы, браузер

В языке EME-L доступ к органам управления диалога (`Control`), таблицам (`Table`) и главному браузеру (`Browser`) производится через следующие методы. Подробности по классу органа — см. `./Control.md`.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetControl(name)` | name: String | Control | Объект органа управления по программному имени. |
| `GetControl(object_name, control_name)` | object_name: String, control_name: String | Control | По имени объекта и имени органа. |
| `GetControl(object_name, control_name, soft_mode)` | + soft_mode: Boolean | Control / Empty | То же; soft_mode=TRUE — вернуть пустой объект при отсутствии органа, FALSE — генерировать исключение. |
| `GetFirstControl()` | — | Control | Первый доступный орган в порядке табуляции. |
| `GetTable(object_name, table_name)` | object_name: String, table_name: String | Table | Объект таблицы по имени объекта и имени таблицы. |
| `GetFocusedTable()` | — | Table / Empty | Таблица с фокусом ввода либо пустое значение. |
| `GetMainBrowser()` | — | Browser / Empty | Главный браузер диалога (см. `./Browser.md`) либо пустой объект. |
| `GetDBObject()` | — | CEMERec (dsDB) | Объект записи базы данных, к которой привязан диалог. |

#### Состояние и флаги поведения

Поле флагов диалога в системе EME.WMS определяет поведение диалога: разрешение добавления/удаления строк, видимость браузера и тулбара, использование отчётов, семафоров и почтовой рассылки.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `IsModified()` | — | Boolean | TRUE — пользователь внёс изменения. |
| `SetModified(modified)` | modified: Boolean | Empty | Устанавливает/сбрасывает флаг модификации. |
| `GetFlags()` | — | Integer | Текущее значение поля флагов. |
| `SetFlags(flags)` | flags: Integer | Empty | Заменяет поле флагов поведения. |
| `AddFlags(flags, set)` | flags: Integer, set: Boolean | Empty | Устанавливает (set=TRUE) или сбрасывает (set=FALSE) указанные биты. |
| `IsWindow()` | — | Integer | Ненулевое, если реальное окно Windows существует; 0 — не существует. |

Биты флагов (`SetFlags`, `AddFlags`): `0x00000001` DF_DONT_ADD_NEWLINE (запретить новые строки), `0x00000002` DF_DONT_DELETE_LINE (запретить удаление), `0x00000004` DF_OUT_BROWSE (выводить браузер перед стартом), `0x00000008` DF_DONT_LISTING (запретить листание), `0x00000010` DF_USE_REPORT (разрешить отчёты), `0x00000020` DF_SHOW_HELP_BUTTON (кнопка справки), `0x00000040` DF_CLOSE_ALL_WINDOWS (закрывать все отчёты при закрытии), `0x00000080` DF_NO_TOOLBAR (без панели кнопок), `0x00000100` DF_NO_BROWSE (скрыть браузер), `0x00000200` DF_USE_PACKET_PRINT (пакетная печать), `0x00000400` DF_SHOW_REPORT_WITHOUT_CAPTION (отчёт без заголовка), `0x00000800` DF_ADD_LINE_AFTER_START (новая строка сразу после старта), `0x00001000` DF_NO_SND_UPDT_AFTER_CLOSE (не рассылать обновление после закрытия), `0x00002000` DF_CHECK_USER_FINGER (проверка отпечатка), `0x00004000` DF_NO_OWNER_CHANGE (запрет смены владельца), `0x00008000` DF_AUTO_CONVERT_TO_EMEL (автоконвертация в EME-L), `0x00010000` DF_DONT_SAVE (запрет save_to_DB), `0x00020000` DF_USE_SEMAPHORE (серверные семафоры), `0x00040000` DF_USER_READ_ONLY (полный запрет изменения), `0x00080000` DF_RUN_ONLY (только один запуск), `0x00100000` DF_USE_MAIL_BUTTON (почтовая рассылка), `0x00200000` DF_DIALOG_HAS_HOT_FUNCTIONS (горячие функции), `0x00400000` DF_NO_SCREENSHOT (запрет снимков экрана).

#### Внешний вид окна

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetCaption()` | — | String | Текущий заголовок окна. |
| `SetCaption(caption)` | caption: String | Empty | Устанавливает заголовок окна. |
| `GetName()` | — | String | Программное имя диалога (soft name). |
| `SetWndFlags(flags)` | flags: Integer | Empty | Флаги внешнего вида (побитовое ИЛИ): `0x1` — без заголовка, `0x2` — маленький заголовок, `0x4` — фиксированный размер. Вызывать до `Show`/`ShowLine`. |
| `GetRect()` | — | Rect | Экранные координаты окна. |
| `MoveWindow(parent, corner)` | parent: Dialog/Control/Table/Integrator, corner: Integer (0–3) | Empty | Располагает окно в указанном углу родителя. Для окон-инструментов. |
| `Update()` | — | Empty | Обновляет данные органов из БД (таблицы — по умолчанию). |
| `Update(update_tables)` | update_tables: Boolean | Empty | TRUE — обновлять и таблицы, FALSE — только органы. |
| `Disable(disable_fields)` | disable_fields: Boolean | Empty | TRUE — поля только для чтения. |
| `Disable(disable_fields, disable_buttons)` | + disable_buttons: Boolean | Empty | То же; дополнительно блокирует/разблокирует кнопки функций. |

#### Закладки диалога

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `DisableTabItem(name, disable)` | name: String, disable: Boolean | Empty | TRUE — закладка недоступна. |
| `DisableAllTabItems(disable)` | disable: Boolean | Empty | Блокирует/восстанавливает все закладки. |
| `IsTabItemActive(name)` | name: String | Boolean | TRUE, если указанная закладка — текущая активная. |
| `SetTabItemActive(name)` | name: String | Empty | Активизирует указанную закладку. |
| `InvisibleTabItem(name, invisible)` | name: String, invisible: Boolean | Empty | TRUE — скрыть закладку. |
| `SetTabItemTextColor(name, color)` | name: String, color: Integer (RGB) | Empty | Цвет текста заголовка закладки. |
| `SetTabItemText(name, caption)` | name: String, caption: String | Empty | Изменяет заголовок закладки. |
| `GetTabItemText(name)` | name: String | String / Empty | Текущий заголовок закладки. |

#### Отчёты и печать

`RunReport` запускает отчёт на экран из текущего диалога в системе EME.WMS. `RunReportIndirect` выводит отчёт напрямую на принтер без предварительного просмотра и возвращает Boolean. `RunReportIndirectInPrinter` выводит отчёт на указанный принтер.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `RunReport(report_name)` | report_name: String | Empty | Запускает отчёт на экран. |
| `RunReport(report_name, base_line)` | + base_line: Integer | Empty | То же с базовой строкой. |
| `RunReport(report_name, base_line, params)` | + params: String или Parameters | Empty | То же с параметрами. |
| `RunReportIndirect(report_name)` | report_name: String | Boolean | Печать напрямую на принтер. |
| `RunReportIndirect(report_name, base_line)` | + base_line: Integer | Boolean | То же с базовой строкой. |
| `RunReportIndirect(report_name, base_line, copies)` | + copies: Integer | Boolean | То же с числом копий. |
| `RunReportIndirect(report_name, base_line, copies, params)` | + params: String | Boolean | То же с параметрами. |
| `RunReportIndirectInPrinter(report_name, printer_name)` | + printer_name: String | Boolean | Печать на указанный принтер. |
| `RunReportIndirectInPrinter(..., base_line)` | + base_line: Integer | Boolean | То же с базовой строкой. |
| `RunReportIndirectInPrinter(..., base_line, copies)` | + copies: Integer | Boolean | То же с числом копий. |
| `RunReportIndirectInPrinter(..., base_line, copies, params)` | + params: String | Boolean | То же с параметрами. |
| `RunReportIndirectInPrinter(..., orientation)` | + orientation: Integer (1 — портрет, 2 — ландшафт) | Boolean | То же с ориентацией (при поддержке ядром). |
| `RunReportIndirectInPrinter(..., orientation, duplex)` | + duplex: Boolean | Boolean | То же с двухсторонней печатью. |
| `SendReportByEmail(report_name, base_line, recipients)` | recipients: String (через `;`) | Integer (1 — успех, 0 — ошибка) | Формирует HTML и отправляет по почте. |
| `SendReportByEmail(report_name, base_line, recipients, subject)` | + subject: String | Integer | То же с темой письма. |
| `SkipReports(report_names...)` | переменное число: String | Empty | Скрывает указанные отчёты из списка печатных форм. |
| `ShowReports(reports, show)` | reports: Array (по ссылке), show: Boolean | Empty | Показывает/скрывает отчёты из массива. |

#### Семафоры диалога

Защита от одновременного редактирования разными пользователями. Используется совместно с флагом `DF_USE_SEMAPHORE`.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetSemaphore()` | — | Empty / String | Устанавливает серверный семафор. Пустое значение при успехе; описание владельца, если блокировка уже есть. |
| `CheckSemaphore()` | — | Empty / String | Проверяет семафор. Пустое значение — свободно; описание владельца — занято. |
| `DeleteSemaphore()` | — | Empty | Удаляет семафор текущего диалога. |

#### Параметры, переменные и контекст

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetParameters()` | — | Parameters | Параметры запуска диалога. Пробелы удаляются по умолчанию. |
| `GetParameters(keep_spaces)` | keep_spaces: Boolean | Parameters | То же; TRUE — сохранить пробелы, FALSE — удалить. |
| `GetClassObject()` | — | Object (Dynamic) | Контейнер данных текущего диалога. |
| `GetClassObject(scope)` | scope: String (`'PARENT'` / `'DIALOG'` / `'GLOBAL'`) | Object (Dynamic) | Контейнер в указанной области видимости. |
| `PutDialogVariable(variable, object)` | variable: String, object: ссылка | Empty | Сохраняет ссылку на объект в переменную диалога. |
| `GetContext()` | — | ссылка | Внутренний контекст выполнения диалога. |
| `GetParent()` | — | Dialog / Empty | Объект родительского диалога; пустой объект, если родителя нет. |
| `BreakExecute()` | — | Empty | Прерывает выполнение главных функций диалога (`OnAfterUpdate` и аналогичных). |

#### Прочее

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `Save(confirm)` | confirm: Boolean | Integer | Низкоуровневое сохранение в БД без авто-`Update`. 0 — отмена операции, 1 — сохранено, 2 — не состоялось, 3 — отмена с продолжением, 4 — не потребовалось. |
| `GetHelpID()` | — | Integer | Идентификатор раздела справки. |
| `PutHelpID(help_id)` | help_id: String или Integer | Integer (1 — успех, 0 — нет диалога) | Устанавливает раздел справки. Строка — идентификатор учебника; число — строка «Учебник» или раздел файла справки. |
| `IsEnterSemaphore()` | — | Empty | Метод-заглушка для совместимости. Всегда пустое значение. |
| `ShowPopupMessage(text)` | text: String | Empty | Всплывающее окно в левом верхнем углу диалога. |
| `ShowPopupMessage(text, x, y)` | + x: Integer, y: Integer | Empty | В заданных координатах. |
| `ShowPopupMessage(text, x, y, bg_color)` | + bg_color: Integer (RGB) | Empty | С цветом фона. |
| `ShowPopupMessage(text, x, y, bg_color, text_color)` | + text_color: Integer (RGB) | Empty | С цветом текста. |
| `ShowPopupMessage(text, x, y, bg_color, text_color, shadow_x, shadow_y)` | + shadow_x: Integer, shadow_y: Integer | Empty | С размерами тени. |
| `SetControlTimer(control, interval)` | control: String/Control, interval: Integer (мс) | Boolean | Таймер для органа управления. |
| `KillControlTimer(control)` | control: String/Control | Boolean | Уничтожает таймер органа. |
| `SetControlDateRange(start_control, end_control)` | имена/объекты Control | Boolean | Один двойной календарь для двух полей даты периода. |
| `LoadScannerDriver(driver)` | driver: Integer или String | Empty | Загружает драйвер сканера. |
| `SendDriverRequest(request, param)` | request: String, param: Integer | Empty | Запрос к подключённому устройству (сканеру). |
| `UnloadScannerDriver(driver)` | driver: Integer или String | Empty | Выгружает драйвер сканера. |

#### Примеры

Синхронизация строки диалога с записью в обработчике `Dialog_OnInit` (реальный паттерн из live-кодовой базы):

```EME-L
'#DEV_INITIALS 27.06.2025 EME company
Класс-обработчик диалога: синхронизация строки диалога с записью'
Dialog_OnInit()
{
    m_r_ActivistTasks.SetLastLine();

    If (~m_r_ActivistTasks.IsValidLine())
        'm_r_ActivistTasks.AppendAndSetLine();'
    End If

    'Переключить диалог на строку записи'
    m_Dialog.SetLine(m_r_ActivistTasks.GetLine());
}
```

Использование родителя, параметров запуска и метода `Close` в диалоге запроса комментария перед печатью (live-паттерн):

```EME-L
'#DEV_INITIALS 27.06.2025 EME company
Диалог запроса комментария: использовать родителя и параметры запуска'
Parent = Dialog.GetParent();
Param = Dialog.GetParameters();

'Установить заголовок из параметров (значение по умолчанию)'
Dialog.SetCaption(Param.GetParam("Caption", "Введите комментарий"));

btnOpenForm_OnCommand()
{
    ReturnQuestion(0);
    Dialog.Close();
}
```

Модальный запуск диалога с кодом запуска и параметрами:

```EME-L
'Открыть диалог выбора склада модально на новой строке'
dlg = Object("Dialog", "WarehouseSelect");
result = dlg.ShowNewLine(0, 100, "mode=pick");
If (result = 1)
    'Пользователь подтвердил выбор'
End If
```

#### См. также

- `./Control.md` — программный доступ к органам управления диалога.
- `./Browser.md` — главный браузер диалога.
- `./File.md` — ссылочный пример формата страницы класса EME-L.
