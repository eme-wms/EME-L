# Класс DeliveryDaysMap

> Синонимы для поиска: CIDeliveryDaysMap DeliveryDaysMap класс EME-L, карта сроков доставки EME-L, сроки доставки EME.WMS, GetCount DeliveryDaysMap, Object("DeliveryDaysMap"), mapDeliveryDays, IsStockStatusExpired срок доставки, GetDeliveryDays регистр, ассоциативный массив сроков доставки, справочник сроков доставки Guide.DeliveryDays

Класс `DeliveryDaysMap` в языке EME-L — это класс-оболочка над ассоциативным массивом (картой) сроков доставки. Предоставляет доступ к данным сроков доставки в EME-L-скриптах и используется для работы с картой сроков доставки, в частности для получения количества записей.

Объект класса `DeliveryDaysMap` в языке EME-L создаётся через `Object("DeliveryDaysMap")` и выступает как вспомогательный контекстный объект в системе EME.WMS: его создание необходимо для корректной работы ряда методов записей (например, `IsStockStatusExpired`, `GetDeliveryDays` регистра склада), которые обращаются к карте сроков доставки внутри. Карта сроков доставки — структура данных, связывающая пары географических точек (откуда → куда) и типов транспорта со сроками доставки в днях.

> Класс `DeliveryDaysMap` в языке EME-L не предназначен для самостоятельного редактирования карты сроков доставки — это объект только для чтения, который предоставляет информацию о загруженной карте. Изменение сроков доставки в системе EME.WMS выполняется через отдельный справочник сроков доставки (класс `Guide.DeliveryDays`), а не через `DeliveryDaysMap`. Для редактирования используется отдельный диалоговый класс, тогда как `DeliveryDaysMap` — вспомогательный объект контекста.

## Создание объекта класса DeliveryDaysMap

```EME-L
'Создание карты сроков доставки для работы методов записей'
mapDeliveryDays = Object("DeliveryDaysMap");
```

Конструктор класса `DeliveryDaysMap` в языке EME-L без аргументов создаёт объект, связанный с текущей картой сроков доставки в системе EME.WMS. Конструктор с аргументами также определён, но аргументы не обрабатываются — оба конструктора создают эквивалентный объект карты. Объект не привязан к конкретному диалогу или браузеру; это самостоятельный вспомогательный объект, который живёт до конца метода, в котором был создан (память EME-L освобождается автоматически — операторов `new` и `delete` нет).

## Методы класса DeliveryDaysMap

Методы класса `DeliveryDaysMap` в языке EME-L предоставляют информацию о загруженной карте сроков доставки в системе EME.WMS.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetCount()` | — | Integer | Возвращает количество записей в карте сроков доставки. Каждая запись соответствует одному маршруту доставки (пара географических точек + тип транспорта) с заданным сроком. |

## Примеры использования класса DeliveryDaysMap

Создание объекта `DeliveryDaysMap` перед обходом строк в языке EME-L — реальный паттерн из системы EME.WMS (классы `OrdersBatch`, `OrderNew`, `OrdersLoader`, `ARXOrderInput`), где карта сроков доставки необходима для работы метода `IsStockStatusExpired` записи ячейки. Без создания `mapDeliveryDays` метод `IsStockStatusExpired` не сможет корректно определить просроченность статуса складского учёта:

```EME-L
'Создание карты сроков доставки перед обработкой строк'
mapDeliveryDays = Object("DeliveryDaysMap");
'необходим для работы r_Cell.IsStockStatusExpired'

For (i_line = 0; i_line < Table.GetNoOfLines(); i_line = i_line + 1)
    dsLine.SetLine(Table.GetDBLine(i_line));
    If (dsLine.IsValidLine())
        'Метод IsStockStatusExpired обращается к mapDeliveryDays внутренне'
        If (dsLine.IsStockStatusExpired())
            'Строка просрочена по статусу складского учёта'
        End If
    End If
End For
```

Получение количества записей в карте сроков доставки в языке EME-L — единственный публичный метод класса, возвращающий целое число:

```EME-L
mapDeliveryDays = Object("DeliveryDaysMap");
Count = mapDeliveryDays.GetCount();
'Count — количество записей в карте сроков доставки'
```

Карта сроков доставки в расчёте сроков доставки клиента в системе EME.WMS — объект создаётся в методе `FillTables` перед вычислением сроков, чтобы метод `GetDeliveryDays` регистра склада мог обращаться к карте (реальный паттерн из класса `OrdersBatch`). Регистр склада вычисляет срок доставки для клиента по ссылке на адрес доставки и тип транспорта:

```EME-L
'Регистр нужен, ибо из его функции просчитывается срок доставки для клиента'
dbReg = Document.GetWarehouse().GetRegisters();
mapDeliveryDays = Object("DeliveryDaysMap");
'необходим для работы r_Reg.GetDeliveryDays'

For (i_line = 0; i_line < Table.GetNoOfLines(); i_line = i_line + 1)
    dsOrderLines.SetLine(Table.GetDBLine(i_line));
    If (dsOrderLines.IsValidLine())
        'GetDeliveryDays возвращает срок в днях или -1 при отсутствии данных'
        DeliveryDays = dbReg.GetDeliveryDays(lAddressRef, lVehicleRef, NULL_REF);
    End If
End For
```

## См. также

- [Класс CalcStockHistory](./CalcStockHistory.md) — другой класс группы WMS в языке EME-L, вычисление остатков по складским регистрам.
- [Класс DataStore](./DataStore.md) — хранилище данных для срезов в системе EME.WMS.
