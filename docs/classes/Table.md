Table CITable Object("Table") GetNoOfLines GetCurrentRow SetCurrentRow AddRow DeleteRow Clear Reset LoadLine LoadLines Update UpdateRow Reskip IsValidRow GetDBObject GetRecord GetDBLine SetDBLine GetRow HasFieldsToWriteToDB Disable SetReadOnly IsReadOnly DisableRow IsDisableRow DisableColumn DisableAddDelRows SetModified IsModified GetNoOfColumns GetCurrentColumn GetColumnName GetColumnCaption GetColumnNumber GetControl SortColumn EnableSorting UniteColumns DeleteColumnsUnion SetMarkColor MarkRow HaveMarkedLine IsLineMarked SetRowTextColor GetRowTextColor ResetRowTextColor SetRowColor GetRowColor IsRowColorExists ResetRowColor SetCellColor SetCellTextColor ResetCellColor ResetCellTextColor SetColumnColor SetFocus IsFocused Invisible GetName SetRedraw GetNoOfInfoLines SetNoOfInfoLines GetCellRect SetOriginalRect GetOriginalRect ExportExcel GetFirstDeletedLine GetNextDeletedLine

### Table — класс таблицы диалога в языке EME-L

Класс `Table` (C++ `CITable`) в языке EME-L — это программный доступ к таблице диалога системы EME.WMS. Через объект `Table` скрипты читают и изменяют строки и ячейки, управляют доступностью, цветом и видимостью, помечают строки, перезагружают данные из БД, сортируют и экспортируют содержимое. Скриптовое имя для конструирования: `Object("Table", ...)`.

Объект создаётся функцией `Object("Table", ...)` и привязан к контексту: либо к текущей таблице диалога (в обработчиках диалога), либо к таблице по имени, либо по имени объекта и члена класса. Конструктор поддерживает пять перегрузок с разным числом аргументов.

Таблица в системе EME.WMS может быть **привязана к БД** (имеет базовую запись `CEMERec` и строковую нумерацию БД) или **не привязана** (хранит данные только в диалоге). Ряд методов (`GetDBObject`, `GetRecord`, `GetDBLine`, `DeleteRow` с подтверждением) ведут себя по-разному в этих режимах — это важно учитывать в прикладных скриптах EME-L.

Создание `Object("Table", ...)` вне конструктора класса (например, в цикле) противоречит правилам управления памятью языка EME-L. В конструкторе класса такие объекты следует инициализировать сразу через `Const`.

#### Конструкторы

| Конструктор | Аргументы | Описание |
|-------------|-----------|----------|
| `Table()` | — | Создаёт объект таблицы, привязанный к текущей таблице диалога по контексту выполнения в языке EME-L. |
| `Table(name)` | name: String — имя таблицы диалога | Создаёт объект таблицы, привязанный к таблице диалога с указанным именем. |
| `Table(objectName, memberName)` | objectName: String, memberName: String | Создаёт объект таблицы по имени объекта класса и имени мембера класса. |
| `Table(objectName, memberName, context)` | + context: Integer — контекст вызова | То же с явным указанием контекста вызова. |
| `Table(objectName, memberName, context, softMode)` | + softMode: Boolean | То же с режимом обработки отсутствия таблицы: `softMode=TRUE` — не бросать исключение, если таблица не найдена (объект с пустой ссылкой), `FALSE` — генерировать исключение. |

#### Строки и навигация в языке EME-L

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetNoOfLines()` | — | Integer | Возвращает количество строк в таблице. |
| `GetCurrentRow()` | — | Integer | Возвращает номер текущей строки таблицы. |
| `SetCurrentRow(row)` | row: Integer | Empty | Устанавливает текущую строку таблицы. |
| `AddRow()` | — | Integer | Добавляет пустую строку (не привязанную к записи БД). Возвращает номер добавленной строки. |
| `AddRow(line)` | line: Integer — номер строки базовой записи | Integer | Добавляет в таблицу уже существующую строку записи БД. Возвращает номер загруженной строки. |
| `DeleteRow(row, confirm)` | row: Integer, confirm: Boolean/Integer | Integer/Boolean | Удаляет строку из таблицы. `confirm=TRUE\|1` — вывести диалог подтверждения перед удалением; `FALSE\|0` — удалить без подтверждения. **Внимание:** вызов с подтверждением в открытой транзакции недопустим и приводит к ошибке. Возвращает `FALSE`, если удаление отменено пользователем, иначе строка удалена. |
| `Clear()` | — | Empty | Удаляет все строки таблицы. Для таблицы, привязанной к БД — через механизм сохранения изменений в БД; для таблицы без привязки — без рассылки сообщений. |
| `Reset()` | — | Empty | Очищает таблицу без удаления строк из базы данных. |
| `LoadLine(line)` | line: Integer — номер строки базовой записи | Integer | Загружает строку в таблицу. Возвращает номер загруженной строки. |
| `LoadLines(lines)` | lines: CIBitBuffer / CIArray / CEMERec (ссылка) | Empty | Загружает группу строк в таблицу: для `CIBitBuffer` — строки, биты которых установлены; для `CIArray` — массив номеров строк; для `CEMERec` — записи текущего набора строк. |
| `Update()` | — | Empty | Перезагружает таблицу из базы данных. |
| `UpdateRow([row[, param]])` | row: Integer (необязательный), param: Integer (необязательный) | Empty | Перезагружает строку таблицы из базы данных. Без аргументов или с одним аргументом перезагружает указанную строку; с двумя — расширенный режим перезагрузки. |
| `Reskip()` | — | Empty | Перезагружает буфер строк для таблицы. |
| `IsValidRow()` | — | Boolean | Проверяет, является ли текущая строка допустимой. TRUE, если текущая строка в диапазоне `0 .. GetNoOfLines()-1`. |
| `IsValidRow(row)` | row: Integer | Boolean | TRUE, если указанная строка находится в диапазоне `0 .. GetNoOfLines()-1`. |

#### Связь с записью БД в системе EME.WMS

Методы этого раздела работают только для таблиц, привязанных к БД. Для таблиц без привязки `GetDBObject` возвращает `NULL`.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetDBObject()` | — | CEMERec / NULL | Возвращает объект записи БД (`CEMERec`) для таблицы, привязанной к БД. Для таблицы без привязки возвращает `NULL`. |
| `GetDBObject(mode)` | mode: String — `"dsDB"` (по умолчанию) или `"dsDIALOG"` | CEMERec / NULL | То же с указанием источника данных: `dsDB` — объект записи строится по данным БД; `dsDIALOG` — по данным диалога. |
| `GetRecord()` | — | Integer | Возвращает номер записи БД, к которой привязана таблица. |
| `GetDBLine()` | — | Integer | Возвращает номер строки БД для текущей строки таблицы. |
| `GetDBLine(row)` | row: Integer — номер строки таблицы | Integer | Возвращает номер строки БД для указанной строки таблицы. |
| `SetDBLine(line)` | line: Integer — номер строки БД | Integer | Устанавливает текущую строку таблицы в соответствии с номером строки БД. Возвращает номер найденной строки таблицы. Если строка не найдена, возвращается количество строк таблицы (`GetNoOfLines()`). |
| `GetRow(line)` | line: Integer — номер строки базовой записи БД | Integer | Возвращает номер строки таблицы по номеру строки базовой записи БД. |
| `HasFieldsToWriteToDB(row)` | row: Integer | Integer | Проверяет, будет ли указанная строка сохраняться в БД ядром. `1`, если строка содержит изменённые поля, подлежащие записи, иначе `0`. |

#### Доступность и флаги строк

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `Disable(disable)` | disable: Boolean | Empty | Делает таблицу доступной только для чтения, устанавливая флаг `READ_ONLY` для всех строк. `TRUE` — только для чтения; `FALSE` — доступна для редактирования. |
| `SetReadOnly(readOnly)` | readOnly: Boolean | Empty | Делает таблицу доступной только для чтения для пользователя. `TRUE` — запретить редактирование; `FALSE` — разрешить. |
| `IsReadOnly()` | — | Boolean | Проверяет, доступна ли таблица пользователю только для чтения. |
| `DisableRow(row, disable)` | row: Integer, disable: Boolean | Empty | Делает указанную строку таблицы доступной только для чтения. `TRUE` — строка задизейблена; `FALSE` — доступна для редактирования. |
| `IsDisableRow(row)` | row: Integer | Boolean | TRUE, если строка задизейблена (только для чтения). |
| `DisableColumn(column, priority)` | column: CIControl (ссылка), priority: Boolean | Empty | Устанавливает доступность ячеек столбца. `priority=FALSE` — доступность ячеек определяется доступностью столбца; `TRUE` — доступностью соответствующих строк, если столбец доступен к редактированию. |
| `DisableAddDelRows(disable)` | disable: Boolean | Empty | Запрещает или разрешает добавление и удаление строк в таблице. `TRUE` — запретить; `FALSE` — разрешить. |
| `SetModified(row, modified)` | row: Integer, modified: Boolean | Empty | Устанавливает или снимает флаг модификации строки пользователем. |
| `IsModified()` | — | Boolean | Проверяет флаг модификации всех строк. TRUE, если хотя бы одна строка была модифицирована пользователем. |
| `IsModified(row)` | row: Integer | Boolean | TRUE, если указанная строка была модифицирована пользователем. |

#### Колонки таблицы

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetNoOfColumns()` | — | Integer | Возвращает количество столбцов в таблице. |
| `GetCurrentColumn()` | — | Integer | Возвращает номер текущего столбца таблицы. |
| `GetColumnName()` | — | String | Возвращает имя текущей колонки таблицы. |
| `GetColumnName(number)` | number: Integer | String | Возвращает имя колонки таблицы по её номеру. |
| `GetColumnCaption()` | — | String | Возвращает заголовок текущей колонки таблицы. |
| `GetColumnCaption(number)` | number: Integer | String | Возвращает заголовок колонки таблицы по её номеру. |
| `GetColumnNumber(name)` | name: String — имя колонки | Integer | Возвращает номер колонки по её имени. |
| `GetControl(number)` | number: Integer — номер колонки | CIControl | Возвращает объект `CIControl` колонки таблицы по номеру столбца. |
| `GetControl([objectName], memberName)` | objectName: String (необязательный), memberName: String | CIControl | Возвращает объект `CIControl` колонки по имени объекта и мембера. Если `objectName` не задан, используется имя таблицы. |
| `SortColumn(name)` | name: String — имя колонки | Empty | Сортирует таблицу по указанной колонке в прямом направлении. |
| `SortColumn(name, direction)` | name: String, direction: Integer (`1` — прямая сортировка, `0` — обратная) | Empty | Сортирует таблицу по указанной колонке с заданным направлением. Требует версию ядра ≥ 21.4. |
| `EnableSorting(enable)` | enable: Boolean | Boolean/Integer | Разрешает или запрещает сортировку таблицы. Возвращает предыдущее значение признака разрешения сортировки. |
| `UniteColumns(firstColumn, lastColumn, caption)` | firstColumn: CIControl/String, lastColumn: CIControl/String, caption: String | Empty | Объединяет колонки таблицы. Колонки должны существовать и находиться на одном этаже заголовка, иначе генерируется исключение. Пустой `caption` — отображается текст из первой колонки. |
| `DeleteColumnsUnion(column)` | column: CIControl/String | Empty | Разрушает объединение колонок, в которое попадает переданная колонка. |

#### Цвет и внешний вид строк и ячеек

Методы цвета в языке EME-L используют формат RGB в виде целого числа: компоненты цвета задаются младшими тремя байтами `0xBBGGRR`.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetMarkColor(color)` | color: Integer (RGB, `0xBBGGRR`) | Empty | Устанавливает цвет помеченных строк таблицы. |
| `MarkRow(row)` | row: Integer | Empty | Помечает указанную строку таблицы. |
| `MarkRow(row, mark)` | row: Integer, mark: Boolean | Empty | Устанавливает (`TRUE`) или снимает (`FALSE`) пометку с указанной строки. |
| `HaveMarkedLine()` | — | Boolean | TRUE, если в таблице есть хотя бы одна отмеченная строка. |
| `IsLineMarked(row)` | row: Integer | Boolean | TRUE, если указанная строка отмечена. |
| `SetRowTextColor(row, color)` | row: Integer, color: Integer (RGB) | Empty | Устанавливает цвет текста указанной строки. |
| `GetRowTextColor(row)` | row: Integer | Integer | Возвращает цвет текста строки (RGB). Если цвет не задан, возвращается `0`. |
| `ResetRowTextColor(row)` | row: Integer | Empty | Сбрасывает цвет текста строки в значение по умолчанию. |
| `SetRowColor(row, color)` | row: Integer, color: Integer (RGB) | Empty | Устанавливает цвет фона указанной строки. |
| `GetRowColor(row)` | row: Integer | Integer | Возвращает цвет фона строки (RGB). Если цвет не задан, возвращается `-1`. |
| `IsRowColorExists(row)` | row: Integer | Integer | `1`, если цвет фона строки был установлен, иначе `0`. Это правильный метод проверки наличия цвета фона. |
| `GetRowColor(row, ignored)` | row: Integer, ignored: любой тип | Integer | **Устаревший метод.** Аналог `IsRowColorExists`; второй параметр игнорируется. Сохранён для обратной совместимости. Не использовать. |
| `ResetRowColor(row)` | row: Integer | Empty | Сбрасывает цвет фона строки в значение по умолчанию. |
| `SetCellColor(column, row, color)` | column: String/Integer, row: Integer, color: Integer (RGB) | Empty | Устанавливает цвет фона ячейки таблицы. |
| `SetCellTextColor(column, row, color)` | column: String/Integer, row: Integer, color: Integer (RGB) | Empty | Устанавливает цвет текста ячейки таблицы. |
| `ResetCellColor(column, row)` | column: String/Integer, row: Integer | Empty | Сбрасывает цвет фона ячейки в значение по умолчанию. |
| `ResetCellTextColor(column, row)` | column: String/Integer, row: Integer | Empty | Сбрасывает цвет текста ячейки в значение по умолчанию. |
| `SetColumnColor(column, color)` | column: String/Integer, color: Integer (RGB) | Empty | Устанавливает цвет фона колонки таблицы. |

#### Окно таблицы и фокус в языке EME-L

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetFocus()` | — | Empty | Устанавливает фокус ввода на таблицу. |
| `SetFocus([column[, row]])` | column: String/Integer (необязательный), row: Integer (необязательный) | Empty | Устанавливает фокус ввода на ячейку таблицы. Без аргументов — на таблицу в целом. |
| `IsFocused()` | — | Boolean | TRUE, если фокус находится на таблице. |
| `Invisible(invisible)` | invisible: Boolean | Empty | `TRUE` — сделать таблицу невидимой; `FALSE` — видимой. |
| `GetName()` | — | String | Возвращает имя таблицы. |
| `SetRedraw(redraw)` | redraw: Boolean | Empty | Разрешает (`TRUE`) или запрещает (`FALSE`) перерисовку таблицы. При `TRUE` выполняется коррекция прокрутки и принудительная перерисовка. |
| `GetNoOfInfoLines()` | — | Integer | Возвращает текущее число нескроллируемых (информационных) строк в таблице. |
| `SetNoOfInfoLines(count)` | count: Integer | Integer | Задаёт число информационных строк. Возвращает предыдущее значение. |
| `GetCellRect(column, row)` | column: String/Integer, row: Integer | CIRect | Возвращает объект `CIRect`, описывающий положение и размер ячейки в экранных координатах. |
| `SetOriginalRect(rect)` | rect: CIRect (ссылка) | Boolean | Устанавливает размеры и положение окна таблицы на диалоге (в единицах редактора диалогов, без учёта растягивания). TRUE при успехе. |
| `GetOriginalRect()` | — | CIRect | Возвращает объект `CIRect` с размерами окна таблицы (в единицах редактора диалогов, без учёта растягивания). |

#### Экспорт и перебор удаляемых строк

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `ExportExcel(fileName, openAfter)` | fileName: String, openAfter: Boolean | Boolean | **Ядро ≥ 50.0.** Экспортирует содержимое таблицы в файл Excel формата XLSX. `openAfter=TRUE` — открыть файл после экспорта. TRUE при успехе. |
| `ExportExcel([fileName[, formatted[, forPrint]]])` | fileName: String, formatted: Boolean (по умолч. TRUE), forPrint: Boolean (по умолч. TRUE) | Boolean | **Ядро < 50.0.** Экспортирует содержимое таблицы в Excel. TRUE при успехе. |
| `GetFirstDeletedLine()` | — | Integer / NULL_REF | Возвращает первую удаляемую таблицей строку БД. Используется в обработчиках `OnCheck` и `OnBeforeSave`. После вызова следующие вызовы `GetNextDeletedLine` возвращают последующие удаляемые строки. `NULL_REF`, если удаляемых строк нет. |
| `GetNextDeletedLine()` | — | Integer / NULL_REF | Возвращает следующую удаляемую таблицей строку БД. `NULL_REF`, если удаляемых строк больше нет. |

#### Примеры использования в языке EME-L

**Получение объекта таблицы и доступ к органу колонки** — типичный паттерн в обработчиках диалога системы EME.WMS:

```EME-L
'Подключение к текущей таблице диалога'
history = Object("Table");

'Получить объект записи БД, привязанной к таблице'
line = history.GetDBObject();
If (line.GetLine() == NULL_REF)
    'Добавить новую строку и заполнить ячейки через GetControl'
    HistoryEquipment.AddRow();
    HistoryEquipment.GetControl(0).Put(is_dos_date());
    HistoryEquipment.GetControl(1).Disable(False);
End If
```

**Пометка строк и проверка наличия помеченных** в прикладном скрипте EME-L:

```EME-L
tbl = Object("Table", "GoodsTable");

'Пометить все строки таблицы'
For (i = 0; i < tbl.GetNoOfLines(); i = i + 1)
    tbl.MarkRow(i, True);
End For

If (tbl.HaveMarkedLine())
    'Есть хотя бы одна помеченная строка'
End If
```

**Перебор удаляемых строк в OnCheck** — проверка ограничений перед сохранением таблицы в БД:

```EME-L
tbl = Object("Table");

'Перебрать строки БД, которые будут удалены при сохранении'
line = tbl.GetFirstDeletedLine();
While (line != NULL_REF)
    'Проверить ограничения для удаляемой строки line'
    line = tbl.GetNextDeletedLine();
End While
```
