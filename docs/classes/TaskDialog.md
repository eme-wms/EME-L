# Класс TaskDialog

> Синонимы для поиска: CITaskDialog TaskDialog класс EME-L, Task Dialog EME-L, диалоговое окно EME-L, DoModal EME-L, SetMainInstruction SetContent SetWindowTitle SetFooterText SetExpansionArea, SetMainIcon SetFooterIcon SetDialogWidth SetOptions, SetCommonButtons SetCommonButtonOptions, AddCommandControl SetCommandControlOptions GetSelectedCommandControlID IsCommandControlEnabled SetDefaultCommandControl RemoveAllCommandControls, AddRadioButton SetRadioButtonOptions GetSelectedRadioButtonID IsRadioButtonEnabled SetDefaultRadioButton RemoveAllRadioButtons, SetVerificationCheckboxText SetVerificationCheckbox GetVerificationCheckboxState, SetProgressBarRange SetProgressBarPosition SetProgressBarState SetProgressBarMarquee, командные кнопки EME-L, радиокнопки EME-L, флажок подтверждения EME-L, индикатор прогресса EME-L, CommonButtons OK YES NO CANCEL RETRY CLOSE, опции диалога ENABLE_HYPERLINKS USE_COMMAND_LINKS SHOW_PROGRESS_BAR SHOW_MARQUEE_PROGRESS_BAR ALLOW_DIALOG_CANCELLATION

## Класс TaskDialog — описание на русском

Класс `TaskDialog` в языке EME-L — класс для создания и отображения диалоговых окон Task Dialog с расширенными возможностями взаимодействия с пользователем. Класс `TaskDialog` предоставляет программный интерфейс к современным диалоговым окнам операционной системы, которые отображают основное сообщение, выделенную инструкцию, заголовок, стандартные и командные кнопки, радиокнопки, флажок подтверждения, область подвала и индикатор прогресса. По сравнению с обычным `MessageBox` класс `TaskDialog` в системе EME.WMS предоставляет значительно более широкие возможности настройки внешнего вида и анализа выбора пользователя.

В языке EME-L объект `TaskDialog` создаётся с обязательным указанием текста сообщения, инструкции и заголовка. В расширенных формах конструктора можно сразу задать стандартные кнопки, опции отображения, текст подвала и список команд. После создания объект настраивается методами (текст, иконки, кнопки, команды, радиокнопки, прогресс) и отображается модальным вызовом `DoModal`, после которого доступны идентификаторы выбранной команды, радиокнопки и состояние флажка подтверждения.

### Основные возможности TaskDialog в EME-L

- Настройка текста заголовка, основной инструкции, сообщения, подвала и разворачиваемой области с дополнительной информацией.
- Установка основной иконки и иконки подвала из ресурсов модуля (`имя_модуля#ID_ресурса`) или псевдонимов встроенных иконок EME: `SERVER`, `MONO`, `WS`, `EME`.
- Формирование набора стандартных кнопок `OK`, `YES`, `NO`, `CANCEL`, `RETRY`, `CLOSE` с возможностью отключения отдельных кнопок и отображения иконки повышения прав.
- Добавление командных кнопок с произвольными идентификаторами, текстом, состоянием активности и иконкой повышения прав.
- Добавление радиокнопок с выбором одного варианта и флажка подтверждения с произвольным текстом.
- Управление поведением диалога в EME-L через флаги: гиперссылки, командные кнопки, разворачиваемая область, индикатор прогресса, бегущий индикатор, позиционирование относительно окна, правосторонняя разметка, возможность сворачивания и другие.
- Настройка индикатора прогресса: диапазон, текущая позиция, состояние (`NORMAL`, `ERROR`, `PAUSED`), бегущий режим.
- Модальное отображение диалога методом `DoModal` и получение результата взаимодействия с пользователем.

Результат работы метода `DoModal` в системе EME.WMS зависит от выбора пользователя: при нажатии стандартной кнопки возвращается её имя (`OK`, `YES`, `NO`, `CANCEL`, `RETRY` или `CLOSE`), при выборе командной кнопки — строка с идентификатором команды. Идентификаторы выбранной команды, радиокнопки и состояние флажка подтверждения доступны через соответствующие методы после закрытия диалога.

Класс `TaskDialog` в языке EME-L используется в прикладных скриптах для организации информативных диалогов с пользователем, запроса подтверждений, выбора варианта действия из нескольких команд, отображения хода длительных операций и других сценариев, где возможностей `MessageBox` недостаточно. Класс `TaskDialog` доступен только при включённой поддержке Task Dialog в ядре (C++ символ препроцессора `EME_TASK_DIALOG`).

## Создание объекта TaskDialog — описание на русском

Класс `TaskDialog` в языке EME-L имеет три конструктора — один на каждое поддерживаемое число аргументов. Все три конструктора требуют обязательных первых трёх параметров: `content` (текст основного сообщения), `mainInstruction` (выделенная инструкция) и `title` (заголовок окна).

```EME-L
'Минимальная форма — три обязательных строковых аргумента'
dlg = Object("TaskDialog", "Операция завершена", "Подтверждение", "EME.WMS");
```

```EME-L
'Расширенная форма — добавлены стандартные кнопки, опции, подвал'
dlg = Object("TaskDialog", "Удалить выбранные записи?", "Подтверждение удаления", "YES NO", "USE_COMMAND_LINKS", "Действие необратимо");
```

```EME-L
'Полная форма — со списком команд (массив строк)'
arrCommands = Array("Удалить", "Перенести в архив", "Отмена");
dlg = Object("TaskDialog", "Выберите действие с записями", "Обработка записей", "EME.WMS", "YES NO", "USE_COMMAND_LINKS", "", arrCommands);
```

### Конструкторы класса TaskDialog

| Конструктор | Аргументы | Описание |
|-------------|-----------|----------|
| `TaskDialog(content, mainInstruction, title)` | content: String, mainInstruction: String, title: String | Создаёт объект диалога с заданным текстом сообщения, инструкции и заголовка. Стандартные кнопки, опции, подвал и команды не заданы. |
| `TaskDialog(content, mainInstruction, title, commonButtons, options, footer)` | + commonButtons: String, options: String, footer: String | То же с набором стандартных кнопок, опциями отображения и текстом подвала, заданными при создании. |
| `TaskDialog(content, mainInstruction, title, commonButtons, options, footer, commands)` | + commands: Array (по ссылке) | То же со списком команд, заданным массивом строк. Каждый элемент массива добавляется как команда; первая команда получает идентификатор `0`, последующие — `1`, `2` и т. д. |

Параметры `commonButtons` и `options` в языке EME-L принимают строки, в которых несколько значений перечисляются через любой разделитель (запятая, пробел и т. п.). Допустимые значения `commonButtons` — `OK`, `YES`, `NO`, `CANCEL`, `RETRY`, `CLOSE`. Допустимые значения `options` см. в разделе «Опции диалога».

## Текстовые элементы — описание на русском

Методы настройки текстовых элементов диалога в языке EME-L. Все методы возвращают пустое значение (`Empty`).

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetWindowTitle(title)` | title: String | Empty | Устанавливает текст заголовка окна. |
| `SetMainInstruction(instruction)` | instruction: String | Empty | Устанавливает текст основной инструкции, выделенной крупным шрифтом в верхней части диалога. |
| `SetContent(content)` | content: String | Empty | Устанавливает текст основного сообщения. |
| `SetFooterText(footer)` | footer: String | Empty | Устанавливает текст подвала диалога. |
| `SetExpansionArea(expandedInformation)` | expandedInformation: String | Empty | Устанавливает текст разворачиваемой области с дополнительной информацией. |
| `SetExpansionArea(expandedInformation, collapsedLabel, expandedLabel)` | + collapsedLabel: String, expandedLabel: String | Empty | То же с явным текстом кнопки сворачивания (`collapsedLabel`) и разворачивания (`expandedLabel`). |

## Иконки — описание на русском

Иконки в языке EME-L загружаются из ресурсов модуля по строке описания. Формат: `имя_модуля#ID_ресурса`, например `db.dll#101`. Также поддерживаются псевдонимы встроенных иконок EME: `SERVER`, `MONO`, `WS`, `EME`. Метод `SetMainIcon` и `SetFooterIcon` возвращают пустое значение.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetMainIcon(icon)` | icon: String | Empty | Загружает и устанавливает основную иконку диалога. |
| `SetFooterIcon(icon)` | icon: String | Empty | Загружает и устанавливает иконку подвала диалога. |

## Размер и опции — описание на русском

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetDialogWidth(width)` | width: Integer | Empty | Устанавливает ширину диалога в пикселях. `0` — использовать ширину по умолчанию. Значение должно быть больше или равно нулю. |

### Опции диалога TaskDialog

`SetOptions(options)` в языке EME-L принимает строку с одним или несколькими флагами, перечисленными через любой разделитель. Флаги, заданные при вызове `SetOptions`, полностью заменяют набор опций, установленный ранее (включая заданные в конструкторе). Метод возвращает пустое значение.

| Флаг | Описание |
|------|----------|
| `ENABLE_HYPERLINKS` | Разрешить гиперссылки в тексте. |
| `USE_HICON_MAIN` | Использовать `HICON` для основной иконки. |
| `USE_HICON_FOOTER` | Использовать `HICON` для иконки подвала. |
| `ALLOW_DIALOG_CANCELLATION` | Разрешить закрытие диалога клавишей `Escape` или кнопкой закрытия в заголовке. |
| `USE_COMMAND_LINKS` | Использовать командные кнопки (со стрелками). |
| `USE_COMMAND_LINKS_NO_ICON` | Использовать командные кнопки без иконок стрелок. |
| `EXPAND_FOOTER_AREA` | Размещать разворачиваемую область в области подвала. |
| `EXPANDED_BY_DEFAULT` | Разворачиваемая область открыта по умолчанию. |
| `VERIFICATION_FLAG_CHECKED` | Флажок подтверждения установлен по умолчанию. |
| `SHOW_PROGRESS_BAR` | Отображать индикатор прогресса. |
| `SHOW_MARQUEE_PROGRESS_BAR` | Отображать бегущий индикатор прогресса. |
| `CALLBACK_TIMER` | Включить таймер обратного вызова с интервалом примерно 200 мс. |
| `POSITION_RELATIVE_TO_WINDOW` | Центрировать диалог относительно родительского окна. |
| `RTL_LAYOUT` | Использовать правостороннее расположение элементов. |
| `NO_DEFAULT_RADIO_BUTTON` | Не выбирать радиокнопку по умолчанию. |
| `CAN_BE_MINIMIZED` | Разрешить сворачивание диалога. |
| `NO_SET_FOREGROUND` | Не вызывать `SetForegroundWindow` при активации. |
| `SIZE_TO_CONTENT` | Подгонять размер диалога под содержимое. |

## Стандартные кнопки — описание на русском

Допустимые значения для всех параметров, принимающих список кнопок в языке EME-L: `OK`, `YES`, `NO`, `CANCEL`, `RETRY`, `CLOSE`. Несколько значений перечисляются через любой разделитель (запятая, пробел и т. п.). Все методы этого раздела возвращают пустое значение.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetCommonButtons(buttons)` | buttons: String | Empty | Устанавливает активные стандартные кнопки. Отключённые кнопки и иконки щита сбрасываются. |
| `SetCommonButtons(buttons, disabledButtons)` | + disabledButtons: String | Empty | То же; `disabledButtons` — кнопки, которые отображаются, но неактивны. |
| `SetCommonButtons(buttons, disabledButtons, elevationButtons)` | + elevationButtons: String | Empty | То же; `elevationButtons` — кнопки, отображаемые с иконкой щита повышения прав. |
| `SetCommonButtonOptions(disabledButtons)` | disabledButtons: String | Empty | Отключает указанные кнопки, не меняя набор активных кнопок. Иконка щита не устанавливается. |
| `SetCommonButtonOptions(disabledButtons, elevationButtons)` | + elevationButtons: String | Empty | То же с явным указанием кнопок для иконки щита повышения прав. |

## Командные кнопки — описание на русском

Каждая командная кнопка в языке EME-L имеет целочисленный идентификатор (больше или равен нулю), уникальный в пределах диалога. Заголовок команды может содержать символ `\n`: текст до него отображается крупным шрифтом, текст после — поясняющим комментарием мелким шрифтом.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `AddCommandControl(id, caption)` | id: Integer, caption: String | Empty | Добавляет активную команду без иконки щита. |
| `AddCommandControl(id, caption, enabled)` | + enabled: Boolean | Empty | То же с явным флагом активности (`TRUE` по умолчанию). |
| `AddCommandControl(id, caption, enabled, requiresElevation)` | + requiresElevation: Boolean | Empty | То же; `requiresElevation=TRUE` отображает иконку щита (`FALSE` по умолчанию). |
| `SetCommandControlOptions(id, enabled)` | id: Integer, enabled: Boolean | Empty | Изменяет только активность команды. |
| `SetCommandControlOptions(id, enabled, requiresElevation)` | + requiresElevation: Boolean | Empty | То же с явным флагом иконки щита (`FALSE` по умолчанию). |
| `GetSelectedCommandControlID()` | — | Integer | Идентификатор команды, выбранной пользователем. Вызывать после `DoModal`. `-1` — была нажата стандартная кнопка, а не команда. |
| `IsCommandControlEnabled(id)` | id: Integer | Boolean | `TRUE` — команда активна, `FALSE` — неактивна. |
| `SetDefaultCommandControl(id)` | id: Integer | Empty | Устанавливает команду, выделенную по умолчанию. |
| `RemoveAllCommandControls()` | — | Empty | Удаляет все командные кнопки. |

## Радиокнопки — описание на русском

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `AddRadioButton(id, caption)` | id: Integer, caption: String | Empty | Добавляет активную радиокнопку. |
| `AddRadioButton(id, caption, enabled)` | + enabled: Boolean | Empty | То же с явным флагом активности (`TRUE` по умолчанию). |
| `SetRadioButtonOptions(id, enabled)` | id: Integer, enabled: Boolean | Empty | Изменяет активность радиокнопки. |
| `GetSelectedRadioButtonID()` | — | Integer | Идентификатор радиокнопки, выбранной пользователем. Вызывать после `DoModal`. |
| `IsRadioButtonEnabled(id)` | id: Integer | Boolean | `TRUE` — радиокнопка активна, `FALSE` — неактивна. |
| `SetDefaultRadioButton(id)` | id: Integer | Empty | Устанавливает радиокнопку, выбранную по умолчанию. |
| `RemoveAllRadioButtons()` | — | Empty | Удаляет все радиокнопки. |

## Флажок подтверждения — описание на русском

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetVerificationCheckboxText(text)` | text: String | Empty | Устанавливает текст флажка подтверждения. |
| `SetVerificationCheckbox(checked)` | checked: Boolean | Empty | Устанавливает начальное состояние флажка. |
| `GetVerificationCheckboxState()` | — | Boolean | Состояние флажка, установленное пользователем. Вызывать после `DoModal`. `TRUE` — установлен, `FALSE` — сброшен. |

## Индикатор прогресса — описание на русском

Для отображения индикатора прогресса в языке EME-L необходимо включить флаг `SHOW_PROGRESS_BAR` в опциях и не включать `SHOW_MARQUEE_PROGRESS_BAR`. Бегущий режим включается флагом `SHOW_MARQUEE_PROGRESS_BAR`. Методы этого раздела возвращают пустое значение.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetProgressBarRange(min, max)` | min: Integer, max: Integer | Empty | Устанавливает диапазон. `min` должен быть больше или равен нулю, `max` должен быть строго больше `min`. |
| `SetProgressBarPosition(position)` | position: Integer | Empty | Устанавливает текущую позицию. Должна находиться в диапазоне, заданном `SetProgressBarRange`. |
| `SetProgressBarState(state)` | state: String | Empty | Устанавливает цветовое состояние. Допустимые значения: `NORMAL` (зелёный), `ERROR` (красный), `PAUSED` (жёлтый). |
| `SetProgressBarMarquee()` | — | Empty | Включает бегущий индикатор со скоростью по умолчанию. |
| `SetProgressBarMarquee(enabled)` | enabled: Boolean | Empty | Включает или выключает бегущий индикатор. |
| `SetProgressBarMarquee(enabled, speed)` | + speed: Integer | Empty | То же с заданной скоростью. Большее значение — меньшая скорость; `0` — скорость по умолчанию. |

## DoModal — отображение и результат

Метод `DoModal` в языке EME-L отображает диалог в модальном режиме и возвращает управление после выбора пользователя. Возвращаемое значение — строка:

- При нажатии стандартной кнопки возвращается её имя: `"OK"`, `"YES"`, `"NO"`, `"CANCEL"`, `"RETRY"`, `"CLOSE"`.
- При выборе командной кнопки возвращается строка с идентификатором команды (целое число в виде текста, например `"0"`, `"1"`).

Идентификатор выбранной команды (как число через `GetSelectedCommandControlID`), идентификатор выбранной радиокнопки (через `GetSelectedRadioButtonID`) и состояние флажка подтверждения (через `GetVerificationCheckboxState`) доступны через соответствующие методы после закрытия диалога.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `DoModal()` | — | String | Отображает диалог и возвращает результат. |

## Примеры использования класса TaskDialog в языке EME-L

Простой диалог подтверждения с двумя командами:

```EME-L
'Создать диалог с двумя командами и кнопкой отмены'
dlg = Object("TaskDialog", "Выберите действие", "Подтверждение", "EME.WMS", "CANCEL", "USE_COMMAND_LINKS", "", Array("Сохранить и закрыть", "Сохранить и продолжить"));

Result = dlg.DoModal();
If (Result = "0")
    'Выбрана первая команда — сохранить и закрыть'
ElseIf (Result = "1")
    'Выбрана вторая команда — сохранить и продолжить'
ElseIf (Result = "CANCEL")
    'Нажата кнопка отмены'
End If
```

Диалог с радиокнопками и флажком подтверждения:

```EME-L
dlg = Object("TaskDialog", "Параметры экспорта", "Экспорт данных", "EME.WMS", "OK CANCEL", "ALLOW_DIALOG_CANCELLATION", "");

'Добавить радиокнопки выбора формата'
dlg.AddRadioButton(10, "CSV");
dlg.AddRadioButton(20, "JSON");

'Флажок подтверждения'
dlg.SetVerificationCheckboxText("Открыть папку после экспорта");
dlg.SetVerificationCheckbox(FALSE);

Result = dlg.DoModal();
If (Result = "OK")
    FormatID = dlg.GetSelectedRadioButtonID();
    If (FormatID = 10)
        'Экспорт в CSV'
    ElseIf (FormatID = 20)
        'Экспорт в JSON'
    End If

    If (dlg.GetVerificationCheckboxState())
        'Открыть папку после экспорта'
    End If
End If
```

## См. также

- [Класс Dialog](./dialog.md) — класс для работы с диалоговыми окнами системы EME, привязанными к записям базы данных.
- [Класс MessageBox и системные функции сообщений](/language/basics/system-functions/messages/) — функции `is_message` и родственные для простых диалогов.
