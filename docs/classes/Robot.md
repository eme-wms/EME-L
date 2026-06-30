Robot CIRobot CRobotPopupStyle тестирование робот popup всплывающее окно подсказка Log LogError Stop Sleep Wait WaitForWindow WaitForLastAction EnterStepMode SetTimeOut IsComplete RemoveAllFalses GetErrorQty EmptyLog Popup StartPopup EndPopup GetPopupStyle SetFocusPopupStyle FocusPopup GetWindowText GetWindowCaption CloseReportByCaption SelectMenuDialog SelectMenuModule MenuModuleFocus MenuDialogFocus GetLastOpenedMenu SetDialog SetModuleDialog SetBrowser DialogClose CentreDialog ActivateTabItem BrowserColumnFocus EditItemInput EditItemInputToRow EditItemKeydown EditItemKeydownToRow EditItemFocus EditItemFocusToRow CheckBoxClick CheckBoxClickToRow ButtonClick Input CheckBox Keydown SetRow GetRow DialogKeydown BrowserKeydown FocusWindowKeydown FocusWindowInput AppKeydown ActivateKeyboardLayout InputRefFromBrowser InputRefFromBrowserToRow GetFromControl GetDialogRecord GetDialogRecordObj GetDialogLine GetTableRecord GetTableLine GetBrowserLine GetBrowserRecord GetNoOfRows CheckValidLine IntegratorSelect IsHasContainer CreateContainer GetContainer GetOrCreateContainer GetDataFromContainer DeleteContainer DeleteAllContainers DeleteAllContainersWithout IsLogContainerOperations RobotGo GetDelay SetDelay IsSchoolBookMode GetDialogReportsQty GetDialogReportName SetWidth SetHeight SetTextColor SetBkColor SetBorderColor SetBorderWidth SetPos SetFontName SetFontHeight SetFontBold SetFontItalic SetFontUnderline SetLeaderPos dsDIALOG dsDB NULL_RECORD NULL_REF

### Класс Robot — описание на русском

Класс `Robot` в языке EME-L — класс для автоматизации имитации действий пользователя при функциональном тестировании приложения. Предоставляет программный интерфейс для управления диалогами, браузерами, меню, вводом данных, нажатием кнопок и клавиш, работы с таблицами и интеграторами. CI-имя класса в системе EME.WMS — `CIRobot`; всплывающие окна стилизуются через вспомогательный класс `CRobotPopupStyle`, доступный из робота.

В языке EME-L класс `Robot` поддерживает создание всплывающих окон с подсказками, ведение журнала тестирования, контроль выполнения шагов сценария, а также передачу данных между роботами через контейнеры. Все действия ставятся в очередь и выполняются в интерфейсном потоке приложения. В системе EME.WMS робот может запускать другие роботы, управлять задержками и пошаговым режимом, а также проверять корректность записей базы данных. Используется для создания автоматизированных тестовых сценариев и демонстрационных режимов.

### Создание объекта Robot в языке EME-L

Самостоятельное создание объектов класса `Robot` через `Object("Robot")` **не допускается**. В языке EME-L объект `Robot` предоставляется системой автоматически при выполнении тестовых сценариев — он доступен в теле методов EME-L-класса робота как предопределённая переменная.

```EME-L
'Переменная Robot уже доступна системой — Object() не вызывается'
Robot.Log("Начало теста");
```

Конструктор без аргументов (`IMPLEMENT_CTOR(CIRobot, 0)`) существует в реализации, но вызывается системой при запуске тестового сценария, а не разработчиком.

### Всплывающие окна и подсказки — методы Robot в языке EME-L

Методы этой группы управляют всплывающими информационными окнами (popup) в демонстрационном режиме в системе EME.WMS. Стиль окна настраивается через вспомогательный объект `CRobotPopupStyle`, получаемый методом `GetPopupStyle`.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetPopupStyle(name)` | name: String | CRobotPopupStyle | Возвращает объект стиля всплывающего окна по имени. Если стиль не создан — создаётся автоматически. |
| `Popup(style, text)` | style: String, text: String | Empty | Выводит всплывающее окно с текстом. Продолжительность рассчитывается автоматически (количество слов × задержка / 3 + задержка). |
| `Popup(style, text, duration)` | style: String, text: String, duration: Integer (секунды) | Empty | То же с явной продолжительностью в секундах. |
| `StartPopup(style, text)` | style: String, text: String | Reference | Начинает отображение, возвращает указатель на объект окна для `EndPopup`. |
| `EndPopup(window_ptr)` | window_ptr: Reference | Empty | Закрывает окно по указателю от `StartPopup`. Выполняет `Wait` перед закрытием. |
| `SetFocusPopupStyle(style)` | style: String | CRobotPopupStyle | Устанавливает имя стиля для подсказок `FocusPopup`. |
| `FocusPopup(place, text)` | place: String, text: String | Empty | Выводит подсказку рядом с элементом интерфейса. Продолжительность рассчитывается так же, как у `Popup(style, text)` — `задержка + количество слов × задержка / 3`. |
| `FocusPopup(place, text, row)` | place: String, text: String, row: Integer | Empty | То же для элемента в указанной строке таблицы. |

В языке EME-L параметр `place` в `FocusPopup` задаёт место вывода: имя органа управления (например, `GoodsItem.Name`) или имя колонки таблицы с номером бита через `#` (например, `Признаки#3`).

### Класс CRobotPopupStyle — стилизация всплывающих окон в языке EME-L

Вспомогательный класс `CRobotPopupStyle` для настройки визуального оформления всплывающих окон в системе EME.WMS. Объект получается из `Robot` методом `GetPopupStyle` или `SetFocusPopupStyle`. Позволяет настраивать размеры окна, цвета текста и фона, параметры рамки, позицию на экране и шрифт.

#### Размеры и позиция

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetWidth(width)` | width: Integer | Empty | Ширина окна в пикселях. |
| `SetHeight(height)` | height: Integer | Empty | Высота окна в пикселях. |
| `SetPos(position)` | position: String | Empty | Позиция окна на экране. Недопустимое значение вызывает `Robot.Stop`. |

#### Цвета и рамка

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetTextColor(color)` | color: Integer (RGB) | Empty | Цвет текста. |
| `SetBkColor(color)` | color: Integer (RGB) | Empty | Цвет фона. |
| `SetBorderColor(color)` | color: Integer (RGB) | Empty | Цвет рамки. |
| `SetBorderWidth(width)` | width: Integer | Empty | Толщина рамки в пикселях. |

#### Шрифт

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetFontName(name)` | name: String | Empty | Имя шрифта. |
| `SetFontHeight(height)` | height: Integer | Empty | Высота шрифта в пикселях. |
| `SetFontBold()` | — | Empty | Жирное начертание. |
| `SetFontItalic()` | — | Empty | Курсивное начертание. |
| `SetFontUnderline()` | — | Empty | Подчёркнутое начертание. |

#### Указатель (хвост) окна

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetLeaderPos(position)` | position: String | Empty | Позиция хвоста для фокусных элементов. Значения: `LB` (слева снизу), `RB` (справа снизу), `LLT` (лево лево верх). |

#### Константы позиций SetPos

Метод `SetPos` в языке EME-L принимает строковый код позиции из следующей таблицы. В реализации недопустимое значение вызывает ошибку через `Robot.Stop`.

| Код | Значение |
|-----|---------|
| `F` | Фокус на текущем элементе |
| `FD` | Фокус на диалоге |
| `FM` | Фокус на пункте меню |
| `C` | Центр экрана |
| `TL` | Верхний левый угол (top left) |
| `TR` | Верхний правый угол (top right) |
| `TC` | Верхний центр (top center) |
| `BL` | Нижний левый угол (bottom left) |
| `BR` | Нижний правый угол (bottom right) |
| `BC` | Нижний центр (bottom center) |
| `CL` | Центр левый (center left) |
| `CR` | Центр правый (center right) |
| `RB` | Справа снизу |
| `LB` | Слева снизу |
| `LLT` | Лево лево верх |

### Журнал тестирования и контроль выполнения в языке EME-L

В языке EME-L класс `Robot` предоставляет методы для ведения журнала тестирования и контроля выполнения шагов. Ошибки, записанные через `LogError`, сохраняются во внутреннем списке и доступны через `GetErrorQty`.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `Log(format_string[, arg1[, ...]])` | format_string: String, argN: любой | Empty | Запись в журнал с датой/временем. Форматная строка с подстановкой через `is_format`. |
| `LogError(format_string[, arg1[, ...]])` | format_string: String, argN: любой | Empty | Запись об ошибке в журнал. Сохраняется во внутреннем списке ошибок. |
| `EmptyLog()` | — | Empty | Очищает журнал и удаляет все действия из очереди. |
| `Stop(format_string[, arg1[, ...]])` | format_string: String, argN: любой | Empty | Останавливает сценарий с сообщением. При включённом `StopIfNotComplete` — безвозвратно. |
| `GetErrorQty()` | — | Integer | Количество накопленных ошибок. |
| `IsComplete()` | — | Boolean | TRUE — все действия в очереди выполнены. |
| `IsComplete(index)` | index: Integer | Boolean | TRUE — указанное действие по номеру выполнено. |
| `RemoveAllFalses()` | — | Empty | Помечает все незавершённые действия как выполненные. Для продолжения цепочки роботов после ошибок. |

### Ожидание и синхронизация — методы Robot в системе EME.WMS

В системе EME.WMS методы ожидания обеспечивают синхронизацию между потоком робота и интерфейсным потоком. `Wait` — основная точка синхронизации, ожидающая завершения всех действий всех роботов.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `Sleep(milliseconds)` | milliseconds: Integer | Empty | Приостанавливает выполнение на указанное время в миллисекундах. |
| `Sleep(milliseconds, message)` | milliseconds: Integer, message: String | Empty | То же с записью сообщения в журнал. |
| `Wait()` | — | Empty | Точка синхронизации: ожидает все действия всех роботов. При невыполненных действиях поведение зависит от `StopIfNotComplete`. |
| `Wait(comment)` | comment: String | Empty | То же с записью в журнал информации о проверяемом объекте. |
| `WaitForWindow()` | — | Integer | Ожидает появления модального диалога. |
| `WaitForWindow(info)` | info: String | Integer | То же с описанием ожидаемого окна для журнала. |
| `WaitForLastAction()` | — | Empty | Ожидает завершения только последнего действия текущего робота. Для работы с модальными диалогами. |
| `EnterStepMode(enabled)` | enabled: Boolean | Empty | Включает/отключает пошаговый режим (пауза после каждого действия). |
| `SetTimeOut(seconds)` | seconds: Integer | Boolean | Таймаут ожидания команды в секундах. Всегда возвращает FALSE. |

### Информация об окнах в языке EME-L

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetWindowText()` | — | String | Текст активного модального окна (MessageBox), зафиксированный при `Wait`/`WaitForWindow`. |
| `GetWindowCaption()` | — | String | Заголовок активного окна. |
| `CloseReportByCaption(caption)` | caption: String | Boolean | Закрывает окно отчёта по заголовку. |

### Меню и навигация в системе EME.WMS

В системе EME.WMS методы навигации воспроизводят выбор пунктов меню и установку фокуса на них. Метод `SelectMenuDialog` открывает диалог через меню, а `MenuDialogFocus`/`MenuModuleFocus` лишь устанавливают фокус без активации.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SelectMenuDialog(dialog_name)` | dialog_name: String | Integer | Выбор пункта меню для открытия диалога. |
| `SelectMenuDialog(dialog_name, launch_code)` | dialog_name: String, launch_code: Integer | Integer | То же с кодом запуска. |
| `SelectMenuModule(module_name)` | module_name: String | Integer | Выбор пункта меню для открытия модуля (имя из «Конфигурации системы»). |
| `MenuModuleFocus(module_name)` | module_name: String | Integer | Фокус на пункт меню модуля без активации. |
| `MenuDialogFocus(dialog_name)` | dialog_name: String | Integer | Фокус на пункт меню диалога без активации. |
| `MenuDialogFocus(dialog_name, launch_code)` | dialog_name: String, launch_code: Integer | Integer | То же с кодом запуска. |
| `GetLastOpenedMenu()` | — | CIMenu | Объект последнего открытого меню. |

### Диалоги и браузеры — контекстные методы в языке EME-L

В языке EME-L методы `SetDialog`/`SetBrowser` задают контекст для последующих операций ввода и чтения. Без вызова `SetDialog` методы `GetDialogRecord`, `GetDialogLine` не смогут найти целевой диалог и вернут `NULL_RECORD`/`NULL_REF`.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetDialog(dialog_name)` | dialog_name: String | Empty | Задаёт имя текущего диалога. |
| `SetModuleDialog(module_dialog_name)` | module_dialog_name: String | Empty | Задаёт имя диалога модуля (имя модуля преобразуется к имени диалога). |
| `SetBrowser(browser_name)` | browser_name: String | Empty | Задаёт имя текущего браузера. |
| `DialogClose()` | — | Boolean | Закрывает текущий диалог. |
| `CentreDialog()` | — | Empty | Центрирует текущий диалог с размером по умолчанию. |
| `ActivateTabItem(tab_name)` | tab_name: String | Boolean | Открывает закладку на диалоге. |
| `BrowserColumnFocus(column_name)` | column_name: String | Integer | Фокус на колонку в текущем браузере. |

### Ввод данных и нажатие клавиш — методы Robot в языке EME-L

#### Прямые методы (с явным указанием строки таблицы)

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `EditItemInput(control, value)` | control: String, value: любой | Integer | Ввод значения в орган управления. Тип определяется по типу поля диалога. |
| `EditItemInputToRow(control, value, row)` | control: String, value: любой, row: Integer | Integer | То же в указанной строке таблицы. |
| `EditItemKeydown(control, key_code)` | control: String, key_code: String | Integer | Нажатие клавиши в органе управления (с установкой фокуса). |
| `EditItemKeydownToRow(control, key_code, row)` | control: String, key_code: String, row: Integer | Integer | То же в указанной строке таблицы. |
| `EditItemFocus(control)` | control: String | Integer | Устанавливает фокус в орган управления. |
| `EditItemFocusToRow(control, row)` | control: String, row: Integer | Integer | То же в указанной строке таблицы. |
| `CheckBoxClick(control, bit)` | control: String, bit: Integer | Integer | Щелчок в чекбокс на текущей строке. |
| `CheckBoxClickToRow(control, bit, row)` | control: String, bit: Integer, row: Integer | Integer | Щелчок в чекбокс в указанной строке. |
| `ButtonClick(control)` | control: String | Integer | Нажатие кнопки. |

#### Контекстные методы (через SetRow) в языке EME-L

В языке EME-L методы `Input`, `CheckBox`, `Keydown` используют текущую строку, заданную `SetRow`. Если строка ≥ 0, вызывается вариант `*ToRow`, иначе — обычный вариант.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `Input(control, value)` | control: String, value: любой | Integer | Ввод значения (с учётом текущей строки). |
| `CheckBox(control, bit)` | control: String, bit: Integer | Integer | Щелчок в чекбокс (с учётом текущей строки). |
| `Keydown(control, key_code)` | control: String, key_code: String | Integer | Нажатие клавиши (с учётом текущей строки). |
| `SetRow(row)` | row: Integer | Empty | Устанавливает текущую строку. -1 — работа с диалогом. |
| `GetRow()` | — | Integer | Текущая строка. -1 — работа с диалогом. |

#### Нажатие клавиш на уровнях диалога, браузера, окна и приложения

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `DialogKeydown(key_code)` | key_code: String | Integer | Нажатие комбинации в текущем диалоге. |
| `BrowserKeydown(key_code)` | key_code: String | Integer | Нажатие клавиши в текущем браузере. |
| `FocusWindowKeydown(key_code)` | key_code: String | Integer | Нажатие клавиши в текущем модальном окне. |
| `FocusWindowKeydown(key_code, caption)` | key_code: String, caption: String | Integer | То же в окне с указанным заголовком. |
| `FocusWindowInput(text)` | text: String | Integer | Ввод текста в текущее модальное окно. |
| `FocusWindowInput(text, caption)` | text: String, caption: String | Integer | То же в окне с указанным заголовком. |
| `AppKeydown(key_code)` | key_code: String | Integer | Нажатие клавиши на уровне приложения. |
| `AppKeydown()` | — | Integer | Сбрасывает состояние клавиатуры (отпускает все клавиши). |

#### Коды клавиш

Параметр `key_code` — строка. Поддерживаются модификаторы `Ctrl+`, `Shift+`, `Alt+`. В реализации `ParseCodeString` обрабатывает строку: ищет модификаторы, затем сопоставляет оставшуюся часть с кодами клавиш.

| Код | Клавиша |
|-----|---------|
| `ESC` | Escape |
| `ENTER` | Enter |
| `TAB` | Tab |
| `PGDN` | Page Down |
| `PGUP` | Page Up |
| `DOWN`, `UP`, `LEFT`, `RIGHT` | Стрелки |
| `DEL` | Delete |
| `SPACE` | Пробел |
| `HOME` | Home |
| `END` | End |
| `F1`–`F12` | Функциональные клавиши |
| `ADD` | Плюс (numpad +, выделение всех строк в браузере) |

Если ни один код не распознан, берётся первый символ строки как код клавиши.

#### Раскладка клавиатуры

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `ActivateKeyboardLayout(language)` | language: String | Integer | Устанавливает раскладку. Значения: `Russian`, `English (United States)`. |

### Ввод ссылок из браузера в языке EME-L

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `InputRefFromBrowser(control, browser, reference)` | control: String, browser: String, reference: Reference | Empty | Вписывает ссылку в поле через браузер добавления. |
| `InputRefFromBrowserToRow(control, row, browser, reference)` | control: String, row: Integer, browser: String, reference: Reference | Empty | То же в указанной строке таблицы. |

### Чтение данных из диалога и браузера в системе EME.WMS

В системе EME.WMS методы чтения возвращают информацию о состоянии диалога, браузера и их строк. Методы `GetTableLine` и `GetBrowserLine` возвращают номер строки БД (не индекс в таблице). Если объект не найден, возвращается `NULL_RECORD` или `NULL_REF`.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetFromControl(control)` | control: String | любой | Значение из органа управления (для колонки таблицы — текущая строка). |
| `GetFromControl(control, row)` | control: String, row: Integer | любой | Значение из ячейки таблицы по строке. |
| `GetDialogRecord()` | — | Integer | Номер записи (RECORD) диалога. Не найден — `NULL_RECORD`. |
| `GetDialogRecordObj(source_type)` | source_type: String | CEMERec | Объект записи диалога. `source_type`: `dsDIALOG` (только чтение) или `dsDB`. Не найден — NULL. |
| `GetDialogLine()` | — | Integer | Номер строки (LINE) диалога. Не найден — `NULL_REF`. |
| `GetTableRecord(table_name)` | table_name: String | Integer | Номер записи (D_RECORD) таблицы. Не найдена — `NULL_RECORD`. |
| `GetTableLine(table_name)` | table_name: String | Integer | Номер строки БД для текущей строки таблицы. Не найдена — `NULL_REF`. |
| `GetTableLine(table_name, row)` | table_name: String, row: Integer | Integer | Номер строки БД для указанной строки таблицы. |
| `GetBrowserLine()` | — | Integer | Номер строки БД для текущей строки браузера. |
| `GetBrowserRecord()` | — | Integer | Номер записи (BR_RECORD) браузера. |
| `GetNoOfRows(table_name)` | table_name: String | Integer | Количество строк в таблице. Не найдена — -1. |
| `CheckValidLine(record, error_message)` | record: CEMERec, error_message: String | Boolean | Проверяет валидность строки записи. При невалидности — ошибка в журнал. |

### Интегратор — метод Robot в языке EME-L

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `IntegratorSelect(control, record)` | control: String, record: CEMERec | Integer | Находит и выделяет элемент в интеграторе по полному адресу записи БД. |

### Контейнеры данных — передача данных между роботами в языке EME-L

В языке EME-L контейнеры позволяют передавать данные между роботами в цепочке через строковые ключи. Контейнер — это именованный объект `CRobotDataContainer`, создаваемый через `CreateContainer`, `GetContainer` или `GetOrCreateContainer`.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `IsHasContainer(name)` | name: String | Boolean | TRUE — контейнер существует. |
| `CreateContainer(name)` | name: String | CRobotDataContainer | Создаёт контейнер. Если существует — очищает. |
| `GetContainer(name)` | name: String | CRobotDataContainer | Возвращает контейнер; создаёт при отсутствии. |
| `GetOrCreateContainer(name)` | name: String | CRobotDataContainer | Возвращает или создаёт контейнер. |
| `GetDataFromContainer(container, key, default_value)` | container: String, key: String, default_value: любой | любой | Чтение из контейнера по ключу без переменной контейнера. |
| `DeleteContainer(name)` | name: String | Boolean | Удаляет контейнер. |
| `DeleteAllContainers()` | — | Boolean | Удаляет все контейнеры. |
| `DeleteAllContainersWithout(name1[, ...])` | nameN: String | Boolean | Удаляет все, кроме перечисленных. |
| `IsLogContainerOperations(enabled)` | enabled: Boolean | Boolean | Включает/отключает логирование операций Put. Всегда FALSE. |

### Управление роботом — выполнение, задержки, режимы в системе EME.WMS

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `RobotGo(class_name[, container[, method_name]])` | class_name: String, container: Reference, method_name: String | любой | Выполняет другой робот как подпрограмму. Первый аргумент — объект Robot. Метод по умолчанию `RobotGo`. |
| `GetDelay()` | — | Integer | Задержка между шагами (мс). |
| `SetDelay(milliseconds)` | milliseconds: Integer | Integer | Устанавливает задержку. Возвращает предыдущее значение. |
| `IsSchoolBookMode()` | — | Boolean | TRUE — активен режим учебника. |

### Отчёты диалога в языке EME-L

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetDialogReportsQty()` | — | Integer | Количество отчётов текущего диалога (инициализирует список). |
| `GetDialogReportName(index)` | index: Integer | String | Имя отчёта по позиции. |

### Примеры использования класса Robot в языке EME-L

Ввод данных в диалог и сохранение:

```EME-L
'Установить контекст диалога'
Robot.SetDialog("GoodsItem");

'Заполнить поле наименования товара'
Robot.EditItemInput("Name", "Тестовый товар");

'Перейти на первую строку таблицы и заполнить ячейку'
Robot.SetRow(0);
Robot.Input("Qty", 10);

'Нажать кнопку сохранения'
Robot.ButtonClick("SaveButton");
Robot.Wait("Проверка сохранения товара");
```

Проверка значения и работа с модальным окном в системе EME.WMS:

```EME-L
'Открыть браузер выбора и вписать ссылку'
Robot.InputRefFromBrowser("GoodsItem", "GoodsBrowser", nGoodsRef);
Robot.Wait();

'Проверить значение в поле после выбора'
sValue = Robot.GetFromControl("Name");
If (sValue != "Ожидаемое наименование")
    Robot.LogError("Ожидалось: %s, получено: %s", "Ожидаемое", sValue);
End If

'Обработать модальное окно подтверждения'
Robot.WaitForWindow("Диалог подтверждения");
Robot.FocusWindowKeydown("Enter");
Robot.Wait();
```

### См. также

- [RobotStockWMS](./RobotStockWMS.md) — вспомогательный класс для тестирования складских операций
