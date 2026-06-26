DBTruncator CIDBTruncator MarkLineToDelete MarkLinesToDelete GetMarkedLinesToDelete SetTimeQuant SetTimeSlot SetFirstTimeSlot IsValidTimeSlot SetNextTimeSlot GetTimeSlot SetFirstRecord IsValidRecord SetNextRecord GetRecordName SetFirstLine IsValidLine SetNextLine GetLine CheckTimeSlot DeleteTimeSlot усекатель базы данных truncator таймслот

# Класс DBTruncator — усекатель базы данных

Класс `DBTruncator` в языке EME-L управляет поэтапным удалением (усечением) устаревших строк записей базы данных по временным слотам. C++-класс — `CIDBTruncator`; регистрируется макросом `REGISTRY_EMEL_CLASS` в `p_System.cpp`.

Предоставляет механизм пометки строк записей на удаление в рамках временных интервалов (таймслотов), проверки целостности данных перед удалением и физического удаления помеченных строк с предварительным архивированием. Поддерживает два режима пометки строк в системе EME.WMS: по одной строке в режиме RealMode (`MarkLineToDelete`) и массовую пометку в соответствии с правилами обхода записи или битовым буфером (`MarkLinesToDelete`).

Объект создаётся через `Object("DBTruncator")` без аргументов. Конструктор инициализирует внутренний квант времени значением 24 часа и устанавливает итератор таймслотов в неактивное состояние.

## Основной сценарий использования

1. Создать объект: `DBTruncator = Object("DBTruncator")`.
2. Задать квант времени через `SetTimeQuant` (часы; по умолчанию 24).
3. Для каждой удаляемой строки установить таймслот через `SetTimeSlot` (нормализуется к началу слота по кванту; создаётся автоматически при отсутствии) и пометить строку методом `MarkLineToDelete` (RealMode) или `MarkLinesToDelete` (обход/битовый буфер).
4. Перебрать таймслоты методами `SetFirstTimeSlot` / `IsValidTimeSlot` / `SetNextTimeSlot`.
5. Для каждого таймслота проверить отсутствие изменений методом `CheckTimeSlot` (пустая строка — можно продолжать; непустая — усекание прервано).
6. Удалить таймслот методом `DeleteTimeSlot` внутри транзакции — строки архивируются и физически удаляются.

Внутри таймслота записи и строки перебираются отдельными итераторами: `SetFirstRecord` / `IsValidRecord` / `SetNextRecord` / `GetRecordName` и `SetFirstLine` / `IsValidLine` / `SetNextLine` / `GetLine`.

## Создание объекта

```EME-L
'Создать усекатель базы данных'
DBTruncator = Object("DBTruncator");
```

Конструктор без аргументов инициализирует внутренний квант времени значением 24 часа и устанавливает итератор таймслотов в неактивное состояние.

## Пометка строк на удаление

Для пометки строк в текущем таймслоте в языке EME-L используются два метода с разными ограничениями по режиму обхода записи. `MarkLineToDelete` работает только в RealMode и помечает одну строку; `MarkLinesToDelete` работает только вне RealMode и помечает строки массово — по правилам обхода записи или по битовому буферу. Попытка вызвать метод в неподходящем режиме генерирует ошибку с предложением использовать парный метод.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `MarkLineToDelete(arg0)` | arg0: CEMERec (объект записи в RealMode) | Empty | Помечает текущую строку объекта записи на удаление. Допустим только при обходе записи в режиме RealMode; иначе ошибка с предложением использовать `MarkLinesToDelete`. Строка должна быть валидной (`IsValidLine`). |
| `MarkLineToDelete(arg0, arg1)` | arg0: Integer или String (номер/имя записи), arg1: Integer (номер строки) | Empty | Помечает указанную строку заданной записи на удаление. |
| `MarkLinesToDelete(arg0)` | arg0: CEMERec (объект записи НЕ в RealMode) | Empty | Помечает все строки, полученные в процессе обхода записи. Применяется только при обходе НЕ в режиме RealMode; иначе ошибка с предложением использовать `MarkLineToDelete`. |
| `MarkLinesToDelete(arg0, arg1)` | arg0: CEMERec, arg1: BitBuffer | Empty | Помечает строки записи в соответствии с битовым буфером — для каждого установленного бита соответствующая строка помечается на удаление. |
| `GetMarkedLinesToDelete(arg0, arg1)` | arg0: Integer или String (номер/имя записи), arg1: BitBuffer (по ссылке) | Empty | Возвращает строки, помеченные на удаление, для заданной записи в текущем таймслоте — устанавливает соответствующие биты в переданном битовом буфере. |

## Управление временными слотами

Таймслот — временной интервал, к которому привязываются помеченные на удаление строки. Размер интервала задаётся квантом времени в часах (по умолчанию 24). Дата-время нормализуется к началу слота согласно кванту.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetTimeQuant(arg0)` | arg0: Integer (часы) | Empty | Задаёт квант времени в часах для разбиения шкалы на таймслоты. По умолчанию 24. |
| `SetTimeSlot(arg0)` | arg0: DateTime (Real) | Empty | Устанавливает текущий таймслот по дате-времени. Значение нормализуется к началу слота согласно кванту. Если таймслот отсутствует во внутреннем хранилище, он создаётся автоматически. |
| `SetFirstTimeSlot()` | — | Empty | Переставляет итератор на первый таймслот. |
| `IsValidTimeSlot()` | — | Boolean | TRUE — итератор указывает на существующий таймслот; FALSE — итератор вышел за пределы множества. |
| `SetNextTimeSlot()` | — | Empty | Переставляет итератор на следующий таймслот. |
| `GetTimeSlot()` | — | DateTime | Дата и время текущего таймслота. |

## Навигация по записям и строкам

Внутри текущего таймслота в системе EME.WMS записи и строки перебираются отдельными итераторами. Перед чтением итератор всегда проверяется соответствующим методом `IsValid...`.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetFirstRecord()` | — | Empty | Переставляет итератор на первую запись в текущем таймслоте. |
| `IsValidRecord()` | — | Boolean | TRUE — итератор указывает на существующую запись; FALSE — вышел за пределы. |
| `SetNextRecord()` | — | Empty | Переставляет итератор на следующую запись. |
| `GetRecordName()` | — | String | Наименование текущей записи. |
| `SetFirstLine()` | — | Empty | Переставляет итератор строк на первую строку текущей записи. |
| `IsValidLine()` | — | Boolean | TRUE — итератор указывает на существующую строку; FALSE — вышел за пределы. |
| `SetNextLine()` | — | Empty | Переставляет итератор на следующую строку. |
| `GetLine()` | — | Integer | Номер текущей строки. |

## Проверка и удаление таймслота

Перед удалением таймслот проверяется на отсутствие изменений в помеченных строках. Проверка (`CheckTimeSlot`) и удаление (`DeleteTimeSlot`) — раздельные вызовы: если `CheckTimeSlot` вернул непустую строку, усекание прерывается, и `DeleteTimeSlot` не вызывается.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `CheckTimeSlot()` | — | String | Проверяет текущий таймслот на наличие изменений. Пустая строка — изменений нет, усекание можно продолжать. Непустая строка — обнаружены изменения, содержит описание записи и строки; усекание следует прервать. |
| `DeleteTimeSlot()` | — | Empty | Удаляет текущий таймслот: архивирует и физически удаляет все помеченные на удаление строки. Пустые записи пропускаются. Выполняется протоколирование архивируемых записей. |

## Примеры

Пометка строк документов на удаление по дате загрузки с последующим перебором таймслотов (реальный сценарий из системы EME.WMS):

```EME-L
DBTruncator = Object("DBTruncator");

'Помечаем документы на удаление по дате загрузки'
r_Document = Object("dsDB", "Document");
r_Document.SetSkipMode();
r_Document.GetLoadDateFld().MustBeLE(Date);
Loop (r_Document)
    DBTruncator.SetTimeSlot(r_Document.GetLoadDate() + r_Document.GetLoadTime());
    DBTruncator.MarkLineToDelete(r_Document.GetRecord(), r_Document.GetLine());
End Loop

'Перебираем таймслоты, проверяем и удаляем'
For (DBTruncator.SetFirstTimeSlot(); DBTruncator.IsValidTimeSlot(); DBTruncator.SetNextTimeSlot())
    is_transaction(1, "Удаляем таймслот");
    Result = DBTruncator.CheckTimeSlot();
    If (Result != "")
        is_catastrophic_log("Усечение прекращено:\n" + Result);
        is_transaction(-1);
        Return;
    End If
    DBTruncator.DeleteTimeSlot();
    is_transaction(-1);
End For
```

Массовая пометка строк подчинённой записи (не RealMode) с привязкой к таймслоту родителя:

```EME-L
DBTruncator.SetTimeSlot(r_OrdersBatch.GetCloseDate() + r_OrdersBatch.GetCloseTime());
r_OrdersLines = r_OrdersBatch.GetOrdersLines();
DBTruncator.MarkLinesToDelete(r_OrdersLines);
DBTruncator.MarkLineToDelete(r_OrdersBatch.GetRecord(), r_OrdersBatch.GetLine());
```

Получение помеченных строк записи в битовый буфер для инспекции:

```EME-L
bbMarked = Object("BitBuffer", r_Document.GetNoOfLines());
DBTruncator.GetMarkedLinesToDelete(r_Document.GetRecord(), bbMarked);
'bbMarked содержит биты для строк, помеченных на удаление'
```

## См. также

- [База данных](../database.md) — транзакции, режимы обхода записей, `CEMERec`.
- [Класс DBImpExp](./DBImpExp.md) — экспорт/импорт среза базы данных.
