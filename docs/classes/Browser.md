> Synonyms for search: Browser, CIBrowser, класс Browser, объект браузера, Object("Browser"), Object Browser, grid EME-L, браузер EME-L, таблица EME-L, CBrowse, CUniBrowse, выбор строк, множественный выбор, mark/unmark, выделение строк, SetFirstSelectedLine, SetNextSelectedLine, Run, RunWithFilter, GetDBObject, GetColumn, Browser.Column, SetLineColor, LoadLines, UnloadAllLines, SetChain, SetIndex, ConvertToExcel, навигация по браузеру, раскраска строк, колонки браузера

# Класс Browser — интерактивный просмотр табличных данных

в языке EME-L класс `Browser` (внутреннее имя C++ — `CIBrowser`) инкапсулирует интерактивный просмотр табличных данных базы данных — диалоговый элемент «браузер» (grid). Регистрируется макросом `REGISTRY_EMEL_CLASS` в `p_System.cpp` и предоставляет программный доступ к навигации по строкам записи, одиночному и множественному выбору строк, фильтрации, сортировке, управлению колонками (ширина, заголовок, видимость, цвет), загрузке и выгрузке строк по условию, установке цепочек и индексов, а также экспорту в Microsoft Excel и запуску модальных диалогов из контекста браузера.

в системе EME.WMS класс `Browser` применяется в прикладных скриптах для организации интерактивного просмотра и выбора данных из таблиц базы данных — как в обработчиках жизненного цикла браузера (`Browser_OnInit`, `OnTune`, `OnSkipEx`), так и для запуска самостоятельных браузеров по имени шаблона. Объект создаётся с привязкой к шаблону браузера по имени либо к текущему контексту выполнения.

Класс поддерживает два типа браузеров под капотом: обычный (`CBrowse`) и древовидный (`CTreeBrowse`). Ряд методов (колонки, цепочки, индексы, раскраска строк) работает только с обычным и явно выбрасывает исключение для древовидного — это отмечено в описаниях соответствующих методов.

---

## Создание объекта — конструкторы

в языке EME-L объект класса `Browser` создаётся оператором `Object("Browser", …)`.

| Конструктор | Аргументы | Возвращает | Описание |
|-------------|-----------|------------|----------|
| `Browser()` | — | Browser | Текущий активный браузер контекста выполнения. Применяется в обработчиках сообщений браузера (`Browser_OnInit`, `OnTune` и т. п.). |
| `Browser(name)` | name: String | Browser | Браузер по имени шаблона (запись справочника «Шаблоны BROWSE»). |
| `Browser(objectName, memberName)` | objectName: String, memberName: String | Browser | Браузер-член EME-L класса по имени объекта и имени мембера. |
| `Browser(objectName, memberName, context)` | + context: Integer | Browser | То же с явным контекстом вызова. |
| `Browser([name[, memberName[, context[, softMode]]]])` | см. ниже | Browser | Гибкая форма. `softMode`: Boolean — при TRUE (ненулевое значение) отсутствие шаблона не вызывает исключение, объект остаётся пустым. По умолчанию FALSE (0) — выбрасывает исключение. |

При создании без аргументов вне контекста браузера/колонки выбрасывается исключение «Объект браузера нужно создавать в контексте браузера или колонки браузера». Используйте безаргументную форму только внутри сообщений жизненного цикла браузера.

```EME-L
'Текущий браузер контекста (в обработчике Browser_OnInit)'
brw = Object("Browser");

'По имени шаблона'
brw = Object("Browser", "GoodsItems");

'По объекту класса и имени мембера'
brw = Object("Browser", "LoadersClassifier", "SectorSelectionBrowser");
```

---

## Жизненный цикл и запуск

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `Run()` | — | Integer | Открывает браузер с одиночным выделением. Возвращает номер строки БД, на которой завершён просмотр. |
| `Run(multipleSelection)` | multipleSelection: Boolean | Integer | То же; TRUE (-1) — множественное выделение, FALSE (0, по умолчанию) — одиночное. |
| `RunWithFilter(bitBuffer)` | bitBuffer: BitBuffer (по ссылке) | Empty | Открывает браузер с множественным выделением, предварительно загрузив выделение из битового массива; после закрытия сохраняет выделение обратно в `bitBuffer`. |
| `Close()` | — | Empty | Закрывает окно браузера, отправляя сообщение о закрытии родительскому окну. |
| `Delete()` | — | Empty | Принудительно удаляет объект-браузер, если его окно уже закрыто. Решает проблему циклических ссылок между браузером и породившим его EME-L классом (dead lock LOCAL_AREA). |
| `GetDialog()` | — | Dialog | Создаёт и возвращает объект диалога, связанного с текущим контекстом браузера. |
| `GetContext()` | — | Reference | Внутренний контекст параметров браузера (указатель на структуру TS_PARAM). |

```EME-L
'Запустить браузер выбора секторов с множественным выделением'
brw = Object("Browser", "LoadersClassifier", "SectorSelectionBrowser");
brw.Run(TRUE);
```

---

## Навигация по строкам

в языке EME-L методы позиционируют текущую строку браузера. Формы с необязательным `changeFocus` (Boolean, по умолчанию FALSE) только возвращают номер строки; при TRUE — также перемещают фокус.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetFirstLine()` / `SetFirstLine(changeFocus)` | changeFocus: Boolean (опц.) | Integer | Первая строка. |
| `SetLastLine()` / `SetLastLine(changeFocus)` | changeFocus: Boolean (опц.) | Integer | Последняя строка. |
| `SetNextLine()` / `SetNextLine(changeFocus)` | changeFocus: Boolean (опц.) | Integer | Следующая строка. |
| `SetPreviousLine()` / `SetPreviousLine(changeFocus)` | changeFocus: Boolean (опц.) | Integer | Предыдущая строка. |
| `GetCurrentLine()` | — | Integer | Номер текущей строки. |
| `SetCurrentLine(line)` | line: Integer | Integer | Устанавливает текущую строку; возвращает её номер. |
| `IsValidLine()` | — | Boolean | TRUE — текущая строка реальна (не NULL_REF). |
| `SetLine(line)` | line: Integer | Empty | Устанавливает стартовую строку БД для браузера. |
| `GetFocusLine()` | — | Integer | Фокусная строка БД. |
| `SetFocusLine(line)` | line: Integer | Empty | Устанавливает фокусную строку БД. |

---

## Выделение строк (mark)

в системе EME.WMS одиночные проверки и групповые операции над пометками строк браузера.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `Select(line, flag)` | line: Integer, flag: Integer | Empty | Ненулевой `flag` — пометить, 0 — снять пометку. |
| `IsSelected(line)` | line: Integer | Integer | 1 — строка помечена, 0 — нет. |
| `GetSelected(startLine)` | startLine: Integer | Integer | Номер первой помеченной строки начиная с `startLine`; NULL_REF, если таковой нет. |
| `IsAllMarked()` / `IsAllSelected()` | — | Integer | 1 — все помечены или ни одна, 0 — частичное выделение. `IsAllSelected` — синоним `IsAllMarked`. |
| `UnMarkAll()` / `UnSelectAll()` | — | Empty | Снять все пометки. `UnSelectAll` — синоним `UnMarkAll`. |
| `GetMarkedCount()` / `GetSelectedCount()` | — | Integer | Количество помеченных строк. `GetSelectedCount` — синоним `GetMarkedCount`. |
| `LoadSelection(bitBuffer)` | bitBuffer: BitBuffer (по ссылке) | Empty | Загружает выделение из битового массива. |
| `SaveSelection(bitBuffer)` | bitBuffer: BitBuffer (по ссылке) | Empty | Сохраняет текущее выделение в битовый массив. |

---

## Обход помеченных строк

Перед обходом выбирается режим упорядочения, затем — цикл от `SetFirstSelectedLine()` до `NULL_REF` через `SetNextSelectedLine()`. в языке EME-L `NULL_REF` равен -1 (не 0) и означает «нет строки».

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetRealSelectionMode()` | — | Empty | Обход по физическому порядку строк БД. |
| `SetSortSelectionMode()` / `SetSortSelectionMode(field)` | field: String/Integer (опц.) | Empty | Обход по полю сортировки браузера (из настроек или явно: имя/номер поля). |
| `SetUserSelectionMode()` | — | Empty | Обход в порядке, который видит пользователь (с учётом наложенной пользователем сортировки). |
| `SetFirstSelectedLine()` | — | Integer | Первая помеченная строка в текущем режиме; NULL_REF, если помеченных нет. |
| `SetLastSelectedLine()` | — | Integer | Последняя помеченная строка; NULL_REF, если нет. |
| `SetNextSelectedLine()` | — | Integer | Следующая помеченная строка; NULL_REF — конец обхода. |
| `SetPreviousSelectedLine()` | — | Integer | Предыдущая помеченная строка; NULL_REF — начало. |
| `GetSelectedLine()` | — | Integer | Текущая помеченная строка режима обхода. |
| `IsValidSelectedLine()` | — | Boolean | TRUE — текущая помеченная строка реальна и помечена. |

```EME-L
'Обойти помеченные строки в порядке сортировки пользователя'
brw.SetUserSelectionMode();
For (line = brw.SetFirstSelectedLine(); line != NULL_REF; line = brw.SetNextSelectedLine())
    r_GoodsItem.SetLine(line);
    'обработать выбранную строку записи в системе EME.WMS'
    arr.Add(line);
End For
```

---

## Запись и строки БД

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetDBObject()` | — | CEMERec | Объект записи, связанной с браузером, установленный на текущую строку. |
| `GetRecord()` | — | Integer | Номер записи БД браузера. |
| `SetRecord(record)` | record: Integer | Empty | Устанавливает браузер на указанную запись. |
| `GetName()` | — | String | Программное имя браузера. |
| `GetEMELClassName()` | — | String | Имя EME-L класса, связанного с браузером через прокси-функцию `bs_main_proxy`; пустая строка, если класс не задан. Только для недревовидных браузеров. |
| `GetMemberName()` | — | String | Имя члена объекта, извлечённое из программного имени браузера. |

```EME-L
'Получить объект записи текущей строки браузера'
r_Item = brw.GetDBObject();
If (r_Item.IsValidLine())
    Code = r_Item.GetCodeNumber();
End If
```

---

## Загрузка и выгрузка строк

Управляет составом строк, отображаемых браузером (механика skip-режима записи). в системе EME.WMS типовой сценарий в `OnSkipEx`: выгрузить все строки, затем загрузить только нужные по условию.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `LoadAllLines()` | — | Empty | Загрузить все строки записи в буфер. |
| `UnloadAllLines()` | — | Empty | Выгрузить все строки из буфера. |
| `LoadLine(line)` / `UnloadLine(line)` | line: Integer | Empty | Загрузить/выгрузить одну строку по номеру БД. |
| `LoadLines(object)` / `UnloadLines(object)` | object: BitBuffer / Array / CEMERec (по ссылке) | Empty | Загрузить/выгрузить строки, помеченные в объекте-источнике. Для записи со skip-режимом показывает прогресс-бар. |
| `GetFilterStartLine()` | — | Integer | Стартовая физическая строка предварительного фильтра браузера. |
| `GetFilterLineCount()` | — | Integer | Количество строк предварительного фильтра. |

```EME-L
'В OnSkipEx выгрузить все, затем загрузить только разрешённые'
AllowedBrowser = Object("Browser");
AllowedBrowser.UnloadAllLines();
AllowedBrowser.LoadLines(Buffer);
```

---

## Колонки

> Методы колонок не поддерживаются древовидными браузерами (`CTreeBrowse`) — выбрасывают исключение.

в языке EME-L параметр `column` у методов колонок принимает **либо** порядковый номер (Integer), **либо** программное имя (String).

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetNoOfColumns()` | — | Integer | Количество колонок. |
| `GetColumn(index)` | index: Integer (0…N-1) | Browser.Column | Объект колонки. См. [класс Column](#класс-column). |
| `GetColumnNumber(columnName)` | columnName: String | Integer | Порядковый номер колонки по программному имени. |
| `GetCurrentColumn()` | — | Integer | Номер текущей (выделенной) колонки. |
| `GetCurrentColumnName()` | — | String | Программное имя текущей колонки. |
| `GetColumnWidth(column)` | column: Integer/String | Integer | Ширина в пикселях; -1, если колонка не найдена. |
| `SetColumnWidth(column, width)` | column: Integer/String, width: Integer | Integer | Устанавливает ширину; возвращает предыдущую (-1, если не найдена). |
| `SetColumnCaption(column, caption)` | column: Integer/String, caption: String | Empty | Заголовок колонки. |
| `ShowColumn(column, visible)` | column: Integer/String, visible: Boolean | Empty | Показать/скрыть колонку. |
| `RemoveColumn(column)` | column: Integer/String | Empty | Удалить колонку. |
| `GetCellRect(column, row)` | column: Integer/String, row: Integer | Rect | Прямоугольник ячейки в экранных координатах. |

---

## Цвет строк и ячеек

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `ResetColorLines()` | — | Empty | Создаёт внутренние структуры цвета строк и сбрасывает расцветки. Должен вызываться до `SetLineColor`/`SetLineSystemColor`. |
| `SetLineColor(line, textColor[, backColor])` | line: Integer, textColor: Integer, backColor: Integer (опц.) | Empty | Цвет текста (и фона) строки. Код цвета 0…15 (см. палитру ниже). При двух аргументах — один цвет для текста и фона. |
| `SetLineSystemColor(line)` | line: Integer | Empty | Системные цвета строки (сброс к стандарту). |
| `SetPaletteColor(colorNum, rgbValue)` | colorNum: Integer (0…15), rgbValue: Integer (RGB) | Empty | Задаёт RGB значение для номера цвета палитры, используемого `SetLineColor`. |
| `SetCellColor(color)` | color: Integer (COLORREF) | Empty | Цвет фона ячеек. |
| `SetTextColor(color)` | color: Integer (COLORREF) | Empty | Цвет текста ячеек. |

Палитра кодов цвета для `SetLineColor`/`SetLineSystemColor`: 0 — системный, 1 — красный, 2 — зелёный, 3 — синий, 4 — жёлтый, 5 — тёмно-серый, 6 — розовый, 7 — пурпурно-фиолетовый, 8 — коричневый, 9 — светло-синий, 10 — серый, 11 — светло-зелёный, 12 — светло-коричневый, 13 — фиолетовый, 14 — светло-жёлтый, 15 — белый.

```EME-L
'Раскрасить тревожную строку: зелёный текст, красный фон'
brw.ResetColorLines();
brw.SetLineColor(Index, 2, 1);
```

---

## Цепочки и индексы

> Только для недревовидных браузеров. `SetChain` допустим только в обработчике `OnTune` — иначе некорректный доступ к памяти.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetChain(field, ref)` | field: String, ref: Integer | Empty | Устанавливает цепочку по полю и строке-заголовку. Только в OnTune. |
| `SetChainArray(field, refArray)` | field: String, refArray: Array (по ссылке) | Empty | Несколько цепочек по массиву ссылок с общим полем. |
| `SetIndex(fieldName)` | fieldName: String | Empty | Поле индекса для поиска. Для индекса по ускорителю — имя ускоряемого поля. |
| `SetIndexKey([keyPart…])` | keyPart: любые | Empty | Ключ поиска по индексу (составной — несколько аргументов). |
| `SetIndexLowerKey([keyPart…])` | keyPart: любые | Empty | Нижняя граница диапазона поиска по индексу. |
| `SetIndexUpperKey([keyPart…])` | keyPart: любые | Empty | Верхняя граница диапазона поиска по индексу. |

Методы `SetIndex*` появились в ядре версии 24.6.

---

## Прочее

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetCaption(caption)` | caption: String | Empty | Текст заголовка окна браузера. |
| `SetPeriodCaptionField(fieldName)` | fieldName: String | Empty | Поле, по которому формируется заголовок кнопки переключения периодов. |
| `Redraw()` | — | Empty | Перерисовка содержимого (обновление текста элементов). |
| `Update()` | — | Empty | Пересчёт строкового буфера отображения. |
| `DeleteUserFilter()` | — | Empty | Снимает все пользовательские фильтры. Недревовидные браузеры. |
| `ConvertToExcel()` / `ConvertToExcel(fileName)` | fileName: String (опц.) | Empty | Экспорт в Excel (без аргумента — через диалог выбора файла). |
| `RunModalDialog(dialogName, startLine)` | dialogName: String, startLine: Integer | Empty | Запускает модальный диалог из контекста браузера с возможностью возврата. |
| `FillLineNumToDBLineArray(array)` | array: Array (по ссылке) | Integer | Заполняет массив соответствием номеров строк браузера → строки БД. Возвращает количество элементов. |
| `SetExtendedMenuFlag([flag])` | flag: Boolean (опц., по умолч. TRUE) | Empty | Признак наличия расширенного меню браузера. |
| `SetFindTextWithoutLaunch(editControl)` | editControl: Control (по ссылке) | Empty | Передаёт в браузер искомый текст из органа ввода для поиска без запуска. |
| `SetFillBrowseWithoutLaunch([flag])` | flag: Boolean (опц., по умолч. TRUE) | Empty | Формирование буфера браузера без визуального запуска. |
| `SetFilterPeriod([field[, mode[, dateFrom[, dateTo]]]])` | field: Integer, mode: Integer, dateFrom: Date, dateTo: Date | Empty | Фильтрация по полю типа Дата и коду периода. Без аргументов отключает фильтр. Ядро ≥ 32.1. |

`mode` для `SetFilterPeriod`: 0 — Сегодня, 1 — Завтра, 2 — Текущая неделя, 3 — Текущий месяц, 4 — Текущий квартал, 5 — Произвольный период (требует `dateFrom`/`dateTo`), 6 — Последняя неделя, 7 — Последний месяц, 8 — Последний квартал.

---

## Класс Column

Объект вложенного класса `Browser.Column` (C++ — `CIBrowser::Column`) возвращается методом `GetColumn(index)`. Предоставляет метаинформацию о колонке браузера.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetName()` | — | String | Программное имя колонки. |
| `GetCaption()` | — | String | Заголовок (отображаемое название) колонки. |
| `GetMemberName()` | — | String | Имя члена объекта, извлечённое из программного имени колонки. |
| `GetFields(array)` | array: Array (по ссылки) | Empty | Заполняет массив номерами полей привязки колонки. |
| `GetEMELClassName()` | — | String | Имя EME-L класса, связанного с колонкой через прокси-функцию `bs_proxy`; пустая строка, если прокси-функция не найдена. |

---

## См. также

- [`system-functions/browser.md`](../system-functions/browser.md) — `is_*` функции группы Browser (`is_hewed`, `is_run_browse` и др.).
- [`EME_L_Events.md`](../EME_L_Events.md) — события жизненного цикла браузера (`Browser_OnInit`, `OnTune`, `OnSkipEx`).
- [`EME_L_Records_and_Fields.md`](../EME_L_Records_and_Fields.md) — класс `CEMERec`, обход строк записи, skip-режим.
- [`EME_L_Classes.md`](../EME_L_Classes.md) — объектная модель EME-L, оператор `Object()`, создание объектов.
