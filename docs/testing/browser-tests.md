> Синонимы: тесты браузеров, TerminalTester, браузеры терминалов, TSD, BrowserASN, BrowserGood

## Тесты браузеров

Браузеры в EME.WMS — это табличные представления данных с возможностью фильтрации, сортировки, выбора строк и навигации. Тесты браузеров проверяют корректность отображения данных, работу фильтров, выбор строк и взаимодействие с другими диалогами.

Тестовые браузеры находятся в директории `Браузеры/` и используются:
- Для проверки корректности отображения данных в табличном виде
- Для тестирования работы терминалов сбора данных (TSD)
- Для проверки интерфейсных сценариев выбора данных

---

## Браузеры терминалов

### `TerminalTester` — тестовый набор браузеров терминалов

`TerminalTester` — это комплексный набор из 21 браузера для тестирования различных сценариев работы терминалов сбора данных. Каждый браузер проверяет конкретную операцию или тип данных.

### `TerminalTester.BrowserASN` — браузер ASN

Проверяет отображение и выбор предварительных уведомлений о поставке (ASN) на терминале.

```EME-L
'Открываем браузер ASN на терминале'
Browser = Object("Browser", "TerminalTester.BrowserASN");
Browser.Run();

'Применяем фильтр по дате'
Browser.SetFilter("DocDate", is_dos_date());

'Выбираем первую строку'
If (Browser.GetNoOfLines() > 0)
    Browser.SetLine(0);
    ASNRef = Browser.GetSelectedLine();
    is_message("BrowserASN", "Выбран ASN: " + ASNRef, "OK", "INFORMATION");
End If
```

---

### `TerminalTester.BrowserBatch` — браузер партий

Проверяет отображение партий (batch/lot) на терминале. Критично для операций приёмки и отгрузки, где требуется выбор конкретной партии товара.

```EME-L
'Браузер партий для терминала'
Browser = Object("Browser", "TerminalTester.BrowserBatch");
Browser.SetFilter("GoodsItemRef", GoodsItemRef);
Browser.Run();

If (Browser.GetNoOfLines() > 0)
    'Выбираем партию с наибольшим остатком'
    Browser.SetLine(0);
    BatchRef = Browser.GetSelectedLine();
    is_message("BrowserBatch", "Выбрана партия: " + BatchRef, "OK", "INFORMATION");
End If
```

---

### `TerminalTester.BrowserBox` — браузер коробов

Проверяет работу с коробами (упаковками) на терминале. Используется для операций, связанных с мультиканальностью и упаковкой товаров.

```EME-L
'Браузер коробов'
Browser = Object("Browser", "TerminalTester.BrowserBox");
Browser.SetFilter("Status", "Available");
Browser.Run();

BoxCount = Browser.GetNoOfLines();
is_message("BrowserBox", "Доступно коробов: " + BoxCount, "OK", "INFORMATION");
```

---

### `TerminalTester.BrowserCell` — браузер ячеек

Проверяет отображение ячеек хранения на терминале. Критично для операций размещения и подбора.

```EME-L
'Браузер ячеек для размещения'
Browser = Object("Browser", "TerminalTester.BrowserCell");
Browser.SetFilter("WarehouseRef", WarehouseRef);
Browser.SetFilter("ZoneRef", ZoneRef);
Browser.Run();

If (Browser.GetNoOfLines() > 0)
    'Выбираем ячейку для размещения'
    Browser.SetLine(0);
    CellRef = Browser.GetSelectedLine();
    is_message("BrowserCell", "Выбрана ячейка: " + CellRef, "OK", "INFORMATION");
End If
```

---

### `TerminalTester.BrowserGood` — браузер товаров

Проверяет отображение списка товаров на терминале. Самый часто используемый браузер в операциях склада.

```EME-L
'Браузер товаров для терминала'
Browser = Object("Browser", "TerminalTester.BrowserGood");
Browser.SetFilter("WarehouseRef", WarehouseRef);
Browser.Run();

'Выбираем товар по штрих-коду'
Barcode = is_input("Отсканируйте штрих-код:", "", "Товар");
Browser.SetFilter("BarCode", Barcode);

If (Browser.GetNoOfLines() > 0)
    Browser.SetLine(0);
    GoodsRef = Browser.GetSelectedLine();
    is_message("BrowserGood", "Товар: " + GoodsRef, "OK", "INFORMATION");
Else
    is_message("BrowserGood", "Товар не найден", "OK", "EXCLAMATION");
End If
```

---

### `TerminalTester.BrowserInventory` — браузер инвентаризации

Проверяет отображение заданий на инвентаризацию на терминале.

```EME-L
'Браузер заданий инвентаризации'
Browser = Object("Browser", "TerminalTester.BrowserInventory");
Browser.SetFilter("Status", "Open");
Browser.Run();

InvCount = Browser.GetNoOfLines();
is_message("BrowserInventory", "Открытых заданий: " + InvCount, "OK", "INFORMATION");
```

---

### `TerminalTester.BrowserSSCC` — браузер SSCC

Проверяет работу с кодами SSCC (Serial Shipping Container Code) на терминале.

```EME-L
'Браузер SSCC'
Browser = Object("Browser", "TerminalTester.BrowserSSCC");
Browser.SetFilter("WarehouseRef", WarehouseRef);
Browser.Run();

If (Browser.GetNoOfLines() > 0)
    Browser.SetLine(0);
    SSCC = Browser.GetValue("SSCCCode");
    is_message("BrowserSSCC", "SSCC: " + SSCC, "OK", "INFORMATION");
End If
```

---

### `TerminalTester.BrowserShipments` — браузер отгрузок

Проверяет отображение документов отгрузки на терминале.

```EME-L
'Браузер отгрузок'
Browser = Object("Browser", "TerminalTester.BrowserShipments");
Browser.SetFilter("DocDate", is_dos_date());
Browser.SetFilter("Status", "Ready");
Browser.Run();

ShipCount = Browser.GetNoOfLines();
is_message("BrowserShipments", "Готовых к отгрузке: " + ShipCount, "OK", "INFORMATION");
```

---

### `TerminalTester.BrowserStaff` — браузер сотрудников

Проверяет отображение списка сотрудников на терминале. Используется для привязки операций к исполнителям.

```EME-L
'Браузер сотрудников'
Browser = Object("Browser", "TerminalTester.BrowserStaff");
Browser.SetFilter("IsActive", TRUE);
Browser.Run();

StaffCount = Browser.GetNoOfLines();
is_message("BrowserStaff", "Активных сотрудников: " + StaffCount, "OK", "INFORMATION");
```

---

### `TerminalTester.BrowserMove` — браузер перемещений

Проверяет отображение заданий на перемещение товаров.

```EME-L
'Браузер перемещений'
Browser = Object("Browser", "TerminalTester.BrowserMove");
Browser.SetFilter("Status", "Open");
Browser.Run();

MoveCount = Browser.GetNoOfLines();
is_message("BrowserMove", "Открытых перемещений: " + MoveCount, "OK", "INFORMATION");
```

---

## Полный список браузеров терминалов

| Браузер | Назначение | Операция |
|---------|-----------|----------|
| `TerminalTester.BrowserASN` | Предварительные уведомления | Приёмка |
| `TerminalTester.BrowserBatch` | Партии товаров | Приёмка, отгрузка |
| `TerminalTester.BrowserBox` | Короба/упаковки | Упаковка |
| `TerminalTester.BrowserCart` | Тележки | Сборка |
| `TerminalTester.BrowserCell` | Ячейки хранения | Размещение, подбор |
| `TerminalTester.BrowserGates` | Ворота | Отгрузка |
| `TerminalTester.BrowserGood` | Товары | Все операции |
| `TerminalTester.BrowserInventory` | Инвентаризация | Инвентаризация |
| `TerminalTester.BrowserLoaders` | Грузчики | Распределение |
| `TerminalTester.BrowserMove` | Перемещения | Перемещение |
| `TerminalTester.BrowserPPU` | Приказы на подбор | Подбор |
| `TerminalTester.BrowserPrinters` | Принтеры | Печать |
| `TerminalTester.BrowserSSCC` | SSCC-коды | Отгрузка |
| `TerminalTester.BrowserSerialNumberCell` | Серийные номера | Учёт |
| `TerminalTester.BrowserShipments` | Отгрузки | Отгрузка |
| `TerminalTester.BrowserStaff` | Сотрудники | Привязка |
| `TerminalTester.BrowserStaffPosition` | Должности | Управление |
| `TerminalTester.BrowserStampsSSCC` | Марки SSCC | Маркировка |
| `TerminalTester.BrowserTTN` | ТТН | Транспорт |
| `TerminalTester.BrowserWarehouseTerritory` | Территория склада | Навигация |
| `TerminalTester.Warehouse` | Склады | Выбор склада |

---

## Браузеры роботов

### `RobotInter.brwFullTest` — полный тест интерфейса

Проверяет все аспекты работы браузеров: открытие, фильтрацию, сортировку, выбор, пагинацию.

```EME-L
'Полный тест браузера'
RobotInter.brwFullTest
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'Открываем браузер'
    Browser = Object("Browser", "GoodsItemBrowser");
    Browser.Run();

    'Проверяем количество строк'
    TotalLines = Browser.GetNoOfLines();
    is_message("brwFullTest", "Всего строк: " + TotalLines, "OK", "INFORMATION");

    'Применяем фильтр'
    Browser.SetFilter("Code", "TEST");
    FilteredLines = Browser.GetNoOfLines();
    is_message("brwFullTest", "После фильтра: " + FilteredLines, "OK", "INFORMATION");

    'Сбрасываем фильтр'
    Browser.ResetFilter();
    ResetLines = Browser.GetNoOfLines();
    is_message("brwFullTest", "После сброса: " + ResetLines, "OK", "INFORMATION");

    'Проверяем сортировку'
    Browser.Sort("Name", TRUE);
    is_message("brwFullTest", "Сортировка по имени (ASC)", "OK", "INFORMATION");

    Browser.Sort("Name", FALSE);
    is_message("brwFullTest", "Сортировка по имени (DESC)", "OK", "INFORMATION");

    'Выбираем строку'
    If (Browser.GetNoOfLines() > 0)
        Browser.SetLine(0);
        SelectedRef = Browser.GetSelectedLine();
        is_message("brwFullTest", "Выбрана строка: " + SelectedRef, "OK", "INFORMATION");
    End If
}
```

---

### `TestAbsent.Browser` — тест отсутствующих данных

Проверяет корректность поведения браузера при отсутствии данных (пустой результат).

```EME-L
'Тест пустого браузера'
TestAbsent.Browser
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'Устанавливаем фильтр, который точно ничего не найдёт'
    Browser = Object("Browser", "GoodsItemBrowser");
    Browser.SetFilter("Code", "NONEXISTENT_CODE_99999");
    Browser.Run();

    If (Browser.GetNoOfLines() == 0)
        is_message("TestAbsent", "Корректно: пустой результат при отсутствии данных", "OK", "INFORMATION");
    Else
        is_message("TestAbsent", "ОШИБКА: Браузер вернул данные при невозможном фильтре", "OK", "STOP");
    End If
}
```

---

### `ProcessTaskTemplates.BrowserRobot` — браузер шаблонов заданий

Проверяет отображение и выбор шаблонов заданий для терминалов.

```EME-L
'Браузер шаблонов заданий'
Browser = Object("Browser", "ProcessTaskTemplates.BrowserRobot");
Browser.SetFilter("IsActive", TRUE);
Browser.Run();

TemplateCount = Browser.GetNoOfLines();
is_message("BrowserRobot", "Доступно шаблонов: " + TemplateCount, "OK", "INFORMATION");
```

---

### `Zone.RobotCells` — браузер ячеек зоны

Проверяет отображение ячеек внутри конкретной зоны склада.

```EME-L
'Браузер ячеек зоны'
Browser = Object("Browser", "Zone.RobotCells");
Browser.SetFilter("ZoneRef", ZoneRef);
Browser.Run();

CellCount = Browser.GetNoOfLines();
is_message("RobotCells", "Ячеек в зоне: " + CellCount, "OK", "INFORMATION");
```

---

## Паттерн создания тестового браузера

### Шаблон теста браузера

```EME-L
Tests.TestMyBrowser
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'Создаём браузер'
    Browser = Object("Browser", "MyBrowser");

    'Устанавливаем фильтры'
    Browser.SetFilter("Field1", Value1);
    Browser.SetFilter("Field2", Value2);

    'Запускаем браузер'
    Browser.Run();

    'Проверяем результаты'
    LineCount = Browser.GetNoOfLines();
    is_message("TestMyBrowser", "Найдено строк: " + LineCount, "OK", "INFORMATION");

    'Выбираем строку'
    If (LineCount > 0)
        Browser.SetLine(0);
        SelectedRef = Browser.GetSelectedLine();
        is_message("TestMyBrowser", "Выбрана строка: " + SelectedRef, "OK", "INFORMATION");
    End If
}
```

---

## Контрольный список теста браузера

- [ ] Браузер открывается без ошибок
- [ ] Фильтрация работает корректно
- [ ] Сброс фильтра возвращает полный список
- [ ] Сортировка работает в обоих направлениях
- [ ] Выбор строки возвращает корректную ссылку
- [ ] Пустой результат обрабатывается корректно
- [ ] Пагинация работает (если применима)
- [ ] Браузер закрывается без утечек памяти
