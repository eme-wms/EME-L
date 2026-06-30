Report ReportClass Object("Report") CIReport Класс отчёта отчёт шаблон отчёта Run Create BindBand ExportExcel ExportPDF ExportHtml ExportXlsx Print GetParam PutParam SetCaption GetReportName GetBandLine ShowWindow OpenFile SaveToFile Close UpdateReport Rebuild GetUpdateTag Collapse Uncollapse EnumAllReports SearchCell FixArea VScroll SetScale GetScale Enable Disable IsReportReady IsSelectionMode IsFixedArea IsDisabled IsFrameInternal SetFrameInternal SetSelectedCellColor GetCollapsedBands SetCollapsedBands ShiftToVIndex BindCell GetCaption CloseReportByCaption DisableFixedArea SetUpdateOnResize GetRect GetRebuildTag PutScale Update

# Класс Report — управление шаблоном отчёта EME DB

`Report` в языке EME-L — это класс управления шаблоном отчёта EME DB и его жизненным циклом: построение, отображение в окне, экспорт в форматы Excel/HTML/PDF/XLSX, печать, настройка параметров и связывание бандов с источниками данных (объектами `Query`). Класс наследует `Range` и тем самым через программный интерфейс родительского класса предоставляет доступ к ячейкам отчёта: значения, оформление, бины, навигация по строкам и столбцам, доступ к источникам данных, привязанным к ячейкам (биндинг).

Объект `Report` создаётся из текущего контекста отчёта (например, из скрипта, привязанного к шаблону отчёта) либо явно по имени шаблона. Через методы класса можно перестраивать отчёт, менять шаблон, управлять масштабом и заголовком окна, включать/отключать редактирование ячеек, выполнять поиск по ячейкам и фиксировать области.

В системе EME.WMS класс `Report` применяется для программного запуска отчётов, прикрепления построенных отчётов к задачам, пакетной печати и автоматизированного экспорта. `NULL_REF` в возвращаемых значениях равен `-1` (каноническое значение EME-L, не `0`).

### Создание объекта — конструкторы Report

Класс `Report` имеет пять конструкторов, различающихся числом аргументов.

`Object("Report")` — привязка к текущему отчёту из контекста выполнения (без аргументов).
`Object("Report", reportName)` — привязка к шаблону отчёта по имени (reportName: строка).
`Object("Report", objectName, memberName)` — привязка через пару «объект/мембер» класса (objectName, memberName: строки).
`Object("Report", objectName, memberName, context)` — то же с явным числовым контекстом вызова.
`Object("Report", objectName, memberName, context, softMode)` — то же с флагом softMode (boolean): `TRUE` — игнорировать отсутствие отчёта, `FALSE` (по умолчанию) — генерировать исключение.

### Идентификация и заголовок — методы Report

`GetReportName()` возвращает строку — имя шаблона текущего отчёта; пустая строка, если шаблон недоступен.
`GetCaption()` возвращает строку — заголовок окна отчёта; пустая строка, если шаблон недоступен.
`SetCaption(caption)` (caption: строка) устанавливает новый заголовок окна и возвращает строку — предыдущий заголовок; пустая строка, если caption пустой или шаблон недоступен.
`GetBandLine(bandName)` (bandName: строка — имя банда) возвращает целое — номер текущей строки указанного банда; `NULL_REF` (-1), если банд не найден, отчёт не готов или строку определить невозможно. Для вертикального банда возвращает вертикальную строку текущей ячейки.

### Флаги и режимы — методы Report

`IsReportReady()` возвращает boolean — `TRUE`, если отчёт построен; `FALSE`, если ещё строится или шаблон недоступен.
`IsSelectionMode()` возвращает boolean — `TRUE`, если включён режим выделения ячеек («муравьи») в текущем отчёте.
`IsFixedArea()` возвращает boolean — `TRUE`, если активен режим фиксации областей отчёта.
`IsDisabled()` возвращает boolean — `TRUE`, если редактирование ячеек отключено.
`Enable([flag])` (flag: boolean, необязательный, по умолчанию `TRUE`) включает (`TRUE`) или отключает (`FALSE`) редактирование ячеек; без аргументов — включает. Возвращает целое: 1 при успехе, 0 при неверных аргументах.
`Disable([flag])` (flag: boolean, необязательный, по умолчанию `TRUE`) — обратное действие `Enable`: отключает (`TRUE`) или включает (`FALSE`) редактирование; без аргументов — отключает.
`IsFrameInternal()` возвращает boolean — `TRUE`, если включено внутреннее отображение границ ячеек для всего отчёта.
`SetFrameInternal()` без аргументов включает внутренние границы; возвращает boolean (`TRUE`, если режим включён или уже был включен).
`SetFrameInternal(flag)` (flag: boolean) явно включает (`TRUE`) или выключает (`FALSE`) внутренние границы; возвращает boolean (`TRUE`, если режим установлен в запрошенное состояние).

### Масштабирование — методы Report

`GetScale()` возвращает вещественное — текущий масштаб отображения отчёта; `1.0`, если окно/шаблон недоступны.
`SetScale(scale)` (scale: вещественное; диапазон больше `0.0` и не более `10.0`) устанавливает масштаб и возвращает вещественное — установленное значение; `0.0`, если значение вне диапазона или окно/шаблон недоступны.
`PutScale(scale)` — синоним `SetScale`.

### Перестройка отчёта — методы Report

`UpdateReport()` без аргументов перестраивает текущий отчёт без смены шаблона; возвращает boolean.
`UpdateReport(newReportName)` (newReportName: строка; пусто — текущий шаблон) перестраивает отчёт с возможной сменой шаблона; возвращает boolean.
`UpdateReport(newReportName, tag)` (tag: строка — тег перестройки, доступный затем через `GetUpdateTag`) перестраивает отчёт со сменой шаблона и тегом; возвращает boolean.
`Update([newReportName])` (newReportName: строка, необязательный) — синоним `UpdateReport`; без аргументов перестраивает текущий шаблон.
`Rebuild([newReportName])` — синоним `UpdateReport`.
`GetUpdateTag()` возвращает строку — тег последней перестройки; пустая строка, если тег не задан.
`GetRebuildTag()` — синоним `GetUpdateTag`.

### Параметры отчёта — методы Report

`GetParam(name)` (name: строка — имя параметра) возвращает строку — значение строкового параметра; `Empty`, если параметр не найден.
`GetParam(name, defaultValue)` (defaultValue: variant) возвращает variant — значение параметра или `defaultValue`, если параметр не найден.
`PutParam(name, value)` (name: строка, value: variant) записывает строковый параметр отчёта; возвращает boolean.

### Связывание данных (биндинг) — методы Report

`BindBand(bandName, source)` (bandName: строка — имя банда; source: ссылка `Query`) — простое связывание горизонтального банда с источником данных; возвращает boolean.
`BindBand(bandName, aliasName, source, fieldName[, dynamic[, saveable]])` (aliasName: строка — человекочитаемый псевдоним; source: ссылка `Query`; fieldName: строка — поле-идентификатор строки банда, пусто — порядковый номер; dynamic: boolean, по умолчанию `FALSE` — динамический банд; saveable: boolean, по умолчанию `FALSE` — добавление/удаление строк) — расширенное связывание банда; возвращает boolean.
`BindCell(cellName, fieldName[, saveable[, noDbSave[, check[, saveAlways]]]])` (cellName: строка — имя ячейки; fieldName: строка — поле источника; saveable: boolean, по умолчанию `FALSE` — сохраняемая ячейка; noDbSave: boolean, по умолчанию `FALSE` — без сохранения в БД; check: boolean, по умолчанию `FALSE` — проверяемая ячейка; saveAlways: boolean, по умолчанию `FALSE` — сохранять всегда) — связывание ячейки с полем; возвращает boolean.
`ShiftToVIndex(bandName, vIndex)` (bandName: строка — имя горизонтального банда/ячейки; vIndex: целое — индекс вертикального банда, 0 — нет сдвига) сдвигает шаблонные ячейки вправо на ширину вертикального банда, умноженную на индекс; применимо, когда в отчёте ровно один вертикальный банд; возвращает boolean.
`ShiftToVIndex(bandName, vBandName, vIndex)` (vBandName: строка — имя вертикального банда) — сдвиг с явным указанием вертикального банда; возвращает boolean.

### Печать и экспорт — методы Report

`Print()` — печать на принтере по умолчанию, одна копия; возвращает boolean.
`Print(printerName)` (printerName: строка; пусто — принтер по умолчанию) — печать на указанном принтере, одна копия; возвращает boolean.
`Print(printerName, copies)` (copies: целое, минимум 1) — печать с заданным числом копий; возвращает boolean.
`ExportExcel(fileName)` (fileName: строка — полный путь) — экспорт в Excel с форматированием; возвращает boolean (`FALSE`, если файл уже существует).
`ExportExcel(fileName, formatted)` (formatted: boolean, по умолчанию `TRUE`) — экспорт с управлением форматированием.
`ExportExcel(fileName, formatted, overwrite)` (overwrite: boolean, по умолчанию `FALSE`) — экспорт с управлением форматированием и перезаписью.
`ExportHtml(fileName)` — экспорт в HTML; возвращает boolean (`FALSE`, если файл уже существует).
`ExportHtml(fileName, overwrite)` (overwrite: boolean, по умолчанию `FALSE`) — экспорт в HTML с перезаписью.
`ExportPDF(fileName)` — экспорт в PDF формата A4 с ориентацией из шаблона; возвращает boolean.
`ExportPDF(fileName, overwrite)` (overwrite: boolean, по умолчанию `FALSE`) — экспорт в PDF формата A4.
`ExportPDF(fileName, overwrite, pageSize)` (pageSize: строка — `A4`/`A3`/`A5`, по умолчанию `A4`) — экспорт с заданным форматом страницы.
`ExportPDF(fileName, overwrite, pageSize, orientation)` (orientation: целое — 0 портрет, 1 ландшафт) — экспорт с форматом страницы и ориентацией.
`ExportXlsx(fileName)` — экспорт в XLSX без использования Excel; возвращает boolean.
`ExportXlsx(fileName, openAfter)` (openAfter: boolean, по умолчанию `FALSE` — открыть файл после экспорта) — экспорт в XLSX.

### Управление окном и жизненным циклом — методы Report

`ShowWindow()` — выводит отчёт в отдельное окно; возвращает ссылку `Report` (`NULL`, если окно создать не удалось).
`ShowWindow([maximize])` (maximize: boolean, по умолчанию `FALSE` — развернуть на весь экран) — вывод с управлением разворотом.
`Create(reportName)` (reportName: строка — имя шаблона) — создаёт неглобальный объект отчёта без отображения окна; возвращает ссылку `Report` (`NULL`, если создать не удалось).
`OpenFile(fileName)` (fileName: строка — путь к файлу) — загружает отчёт из файла и отображает окно; возвращает ссылку `Report` (`NULL`, если файл не открыт).
`OpenFile(fileName, visible)` (visible: boolean, по умолчанию `TRUE`) — загрузка из файла с управлением видимостью.
`Close()` — закрывает окно отчёта (для отчёта, не встроенного в диалог); возвращает boolean.
`CloseReportByCaption(caption)` (caption: строка — заголовок окна) — закрывает окно отчёта с указанным заголовком; возвращает boolean.
`DisableFixedArea()` — отключает режим фиксации областей; возвращает boolean.
`SaveToFile(fileName)` (fileName: строка — путь к файлу) — сохраняет отчёт в файл; возвращает boolean.
`SetUpdateOnResize(updateOnResize)` (updateOnResize: boolean) — включает (`TRUE`) или отключает (`FALSE`) автоматическую перестройку при изменении размеров окна; возвращает Empty.

### Запуск отчёта — методы Report

`Run(reportName)` (reportName: строка) — запускает построение отчёта с отображением окна; возвращает ссылку `Report` (`NULL`, если отчёт не создан).
`Run(reportName, visible)` (visible: boolean, по умолчанию `TRUE`) — построение с управлением видимостью.
`Run(reportName, visible, params)` (params: строка параметров или ссылка `Parameters`) — построение с параметрами.
`Run(reportName, visible, params, lParam)` (lParam: ссылка — дополнительный объект, передаваемый в контекст построения) — построение с параметрами и объектом lParam.
`Run(reportName, visible, params, lParam, useDialog)` (useDialog: boolean, по умолчанию `FALSE` — учитывать текущий диалог) — построение с параметрами, lParam и учётом диалога.

### Управление бандами и навигация — методы Report

`Collapse()` — сворачивает все банды, поддерживающие сворачивание; возвращает boolean.
`Uncollapse()` — разворачивает все свёрнутые банды; возвращает boolean.
`GetCollapsedBands(mapResults)` (mapResults: ссылка `Map`; заполняется парами «текст ячейки банда» → `TRUE`/`FALSE`) — заполняет Map состоянием сворачивания бандов; возвращает boolean.
`GetCollapsedBands(mapResults, mapCells)` (mapCells: ссылка `Map` — фильтр по именам ячеек, идентифицирующим строки банда) — то же с фильтром по ячейкам.
`SetCollapsedBands(mapResults)` (mapResults: ссылка `Map`; `TRUE` — свернуть, `FALSE` — развернуть) — сворачивает/разворачивает банды по данным из `GetCollapsedBands`; возвращает boolean.
`SetCollapsedBands(mapResults, mapCells)` (mapCells: ссылка `Map` — фильтр по ячейкам) — то же с фильтром.
`SearchCell(findText[, flags])` (findText: строка; flags: целое, необязательный; 1 `SF_BACK` обратный поиск, 2 `SF_MATCHCASE` учитывать регистр, 4 `SF_MATCHWHOLE` целое слово, 8 `SF_FROMBEGINNING` с начала отчёта; без flags — 0) — находит ячейку с текстом и устанавливает фокус; возвращает boolean.
`VScroll(command)` (command: целое — команда прокрутки, см. константы Windows `SB_*`) — вертикальная прокрутка окна; возвращает boolean.
`FixArea(cellName)` (cellName: строка — имя ячейки) — фиксирует области отчёта по указанной ячейке; для отчётов на диалоге вызывать в `Dialog_OnAfterUpdate`; возвращает boolean.

### Утилиты — методы Report

`EnumAllReports()` — возвращает ссылку `Collection` — коллекцию объектов `Report` для всех открытых отчётов; пустая коллекция, если отчётов нет.
`EnumAllReports(array)` (array: ссылка `Array` для заполнения) — заполняет массив объектами `Report`; возвращает целое — количество добавленных объектов.
`EnableToolTips()` — включает всплывающие подсказки (tooltips) для отчёта; возвращает boolean.
`EnableToolTips(enable)` (enable: boolean) — включает (`TRUE`) или отключает (`FALSE`) подсказки; возвращает boolean.
`SetSelectedCellColor(colorIndex, color)` (colorIndex: целое — 1 рамка фокусной ячейки, 2 рамка выделения; color: целое) — устанавливает цвет рамки; возвращает boolean.
`GetRect()` — возвращает ссылку `Rect` — прямоугольник клиентской области окна отчёта; `NULL`, если создать не удалось.

### Примеры в языке EME-L

Запуск отчёта без отображения окна (реальный паттерн из классов EME.WMS):

```EME-L
'Построить отчёт и получить объект Report для дальнейшей обработки'
rep = Object("Report").Run("MyReport", 0);
```

Установка заголовка окна текущего отчёта из контекста выполнения:

```EME-L
'Установить заголовок отчёта в обработчике построения'
Object("Report").SetCaption(tr("Операции для клиентов"));
```

Получение параметра отчёта со значением по умолчанию:

```EME-L
'Получить строковый параметр; если не задан — значение по умолчанию'
printer = Object("Report").GetParam("Printer", "");
```

Экспорт отчёта в PDF формата A4 с перезаписью:

```EME-L
'Экспортировать отчёт в PDF, перезаписав файл при необходимости'
Object("Report").ExportPDF("C:\\Reports\\MyReport.pdf", TRUE, "A4");
```

Сохранение состояния свёрнутых бандов и восстановление:

```EME-L
'Запомнить состояние бандов в Map'
mapState = Object("Map");
Object("Report").GetCollapsedBands(mapState);
'... позже — восстановить состояние'
Object("Report").SetCollapsedBands(mapState);
```

### См. также

- Класс `Range` — родительский класс `Report` для работы с ячейками шаблона отчёта (значения, оформление, бины, строки/столбцы).
- Класс `Rect` — результат метода `GetRect()` (прямоугольник клиентской области окна).
- Класс `Query` — источник данных, передаваемый в методы `BindBand`.
- Класс `Collection` — тип результата `EnumAllReports()`.
- Класс `Map` — тип аргумента `GetCollapsedBands`/`SetCollapsedBands`.
- Класс `Parameters` — тип аргумента `params` перегрузок `Run`.
