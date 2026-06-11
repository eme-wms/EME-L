> Синонимы: роботы-тестировщики, CIRobot, GuideRobots, GoodsItemRobot, WarehouseRobot, INCOMERobot, OrdersBatchRobot

## Роботы-тестировщики (CIRobot)

Роботы-тестировщики — это автоматизированные сценарии тестирования, выполняющие полные циклы работы с классификаторами, операциями склада и отчётами. Роботы имитируют действия пользователя: открывают диалоги, заполняют поля, нажимают кнопки, проверяют результаты.

**Запуск:** `Система → Ресурсы → Тестирование → Робот`  
**Требуется монопольный режим.**

В отличие от юнит-тестов, роботы:
- Выполняют полные бизнес-сценарии от начала до конца
- Работают через UI (имитация действий пользователя)
- Требуют монопольного доступа к базе данных
- Запускаются через систему `CIRobot`

---

## Архитектура системы роботов

### Базовый класс: `CIRobot`

Все роботы реализуют метод `RobotBatchGo_RST(Robot AS "CIRobot")`. Система `CIRobot` предоставляет методы для имитации действий пользователя:

| Метод CIRobot | Назначение |
|---------------|-----------|
| `SetDelay(ms)` | Установка задержки между шагами |
| `ClickButton("OK")` | Клик по кнопке |
| `SetText("Field", "Value")` | Ввод текста в поле |
| `SelectItem("Combo", "Item")` | Выбор элемента из списка |
| `OpenDialog("DialogName")` | Открытие диалога |
| `CloseDialog()` | Закрытие диалога |
| `Wait(ms)` | Ожидание |

### Регистрация робота

Чтобы робот появился в списке тестирования, необходимо:
1. Создать класс с методом `RobotBatchGo_RST(Robot AS "CIRobot")`
2. Прописать имя класса в конструкторе `AllRobots`

```EME-L
AllRobots
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
Constructor()
{
    ' Регистрируем всех роботов '
    Robots = Object("Array")
    Robots.Add("GuideRobots")
    Robots.Add("GoodsItemRobot")
    Robots.Add("ClientRobot")
    Robots.Add("WarehouseRobot")
    Robots.Add("INCOMERobot")
    Robots.Add("OrdersBatchRobot")
    ' ... и т.д. ... '
}
```

---

## Роботы классификаторов

### `GuideRobots` — базовый класс роботов классификаторов

`GuideRobots` — базовый класс для всех роботов-тестировщиков классификаторов. Реализует общую логику создания, редактирования и удаления элементов классификаторов.

```EME-L
GuideRobots
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RobotBatchGo_RST(Robot AS "CIRobot")
{
    ' Открываем справочник '
    Robot.OpenDialog("GoodsItem")
    Robot.Wait(500)

    ' Создаём новый элемент '
    Robot.ClickButton("New")
    Robot.Wait(300)

    ' Заполняем поля '
    Robot.SetText("Code", "TEST_" + is_time())
    Robot.SetText("Name", "Тестовый товар")
    Robot.SelectItem("MU", "Штука")

    ' Сохраняем '
    Robot.ClickButton("Save")
    Robot.Wait(300)

    ' Закрываем диалог '
    Robot.CloseDialog()
}
```

**Что проверяет:**
- Открытие справочника
- Создание нового элемента
- Заполнение полей
- Сохранение
- Закрытие диалога

---

### `GoodsItemRobot` — тестирование справочника товаров

Проверяет полный цикл работы со справочником товаров: создание, редактирование, установку ВГХ, единиц измерения, штрих-кодов.

```EME-L
GoodsItemRobot
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RobotBatchGo_RST(Robot AS "CIRobot")
{
    ' Открываем справочник товаров '
    Robot.OpenDialog("GoodsItem")
    Robot.Wait(500)

    ' Создаём тестовый товар '
    Robot.ClickButton("New")
    Robot.Wait(300)

    Code = "ROBOT_" + is_time()
    Robot.SetText("Code", Code)
    Robot.SetText("Name", "Товар создан роботом")
    Robot.SetText("Weight", "1.5")
    Robot.SetText("Length", "20")
    Robot.SetText("Width", "15")
    Robot.SetText("Height", "10")

    ' Выбираем единицу измерения '
    Robot.SelectItem("DefaultMU", "Штука")

    ' Сохраняем товар '
    Robot.ClickButton("Save")
    Robot.Wait(300)

    ' Проверяем, что товар создан '
    Robot.SetText("SearchCode", Code)
    Robot.ClickButton("Find")
    Robot.Wait(200)

    ' Закрываем диалог '
    Robot.CloseDialog()
}
```

---

### `ClientRobot` — тестирование справочника клиентов

Проверяет работу с клиентами: создание, группы клиентов, адреса доставки.

```EME-L
ClientRobot
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RobotBatchGo_RST(Robot AS "CIRobot")
{
    ' Открываем справочник клиентов '
    Robot.OpenDialog("Clients")
    Robot.Wait(500)

    ' Создаём тестового клиента '
    Robot.ClickButton("New")
    Robot.Wait(300)

    Robot.SetText("Code", "CLT_" + is_time())
    Robot.SetText("Name", "Тестовый клиент")
    Robot.SetText("INN", "1234567890")
    Robot.SetText("KPP", "123456789")

    ' Сохраняем '
    Robot.ClickButton("Save")
    Robot.Wait(300)

    Robot.CloseDialog()
}
```

---

### `WarehouseRobot` — тестирование справочника складов

Проверяет создание и настройку складов, зон, секторов, ячеек.

```EME-L
WarehouseRobot
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RobotBatchGo_RST(Robot AS "CIRobot")
{
    ' Открываем справочник складов '
    Robot.OpenDialog("Warehouse")
    Robot.Wait(500)

    ' Создаём тестовый склад '
    Robot.ClickButton("New")
    Robot.Wait(300)

    Robot.SetText("Code", "WH_" + is_time())
    Robot.SetText("Name", "Тестовый склад")
    Robot.SelectItem("WarehouseType", "Обычный")

    ' Сохраняем '
    Robot.ClickButton("Save")
    Robot.Wait(300)

    Robot.CloseDialog()
}
```

---

## Полный список роботов классификаторов

| Робот | Справочник | Что тестирует |
|-------|-----------|--------------|
| `GuideRobots` | Базовый | Общая логика классификаторов |
| `ClientRobot` | Клиенты | Создание, ИНН, КПП |
| `ClientGroupClassifierRobot` | Группы клиентов | Иерархия групп |
| `ClientClassifierRobot` | Классификатор клиентов | Типы клиентов |
| `CountryRobot` | Страны | Справочник стран |
| `CurrencyRobot` | Валюты | Курсы, коды |
| `GeographyClassifierRobot` | География | Регионы, города |
| `GoodsItemClassifierRobot` | Классификатор товаров | Категории |
| `GoodsItemRobot` | Товары | ВГХ, единицы, штрих-коды |
| `LoaderClassifierRobot` | Грузчики | Сотрудники-грузчики |
| `MeasUnitRobot` | Единицы измерения | Факторы, коэффициенты |
| `PriceGroupRobot` | Ценовые группы | Группы цен |
| `ProducerRobot` | Производители | Бренды |
| `QualityCertificatesRobot` | Сертификаты качества | Сроки, номера |
| `ReasonClassifierRobot` | Причины | Причины операций |
| `StaffPositionRobot` | Должности | Штатные единицы |
| `StaffRobot` | Сотрудники | Персонал |
| `TerminalTypeRobot` | Типы терминалов | TSD, принтеры |
| `TransactionRobot` | Типы транзакций | Операции склада |
| `TransportRobot` | Транспорт | ТС, водители |
| `TransportTypeRobot` | Типы транспорта | Классы ТС |
| `TrayRobot` | Контейнеры | Поддоны, ящики |
| `VendorRobot` | Поставщики | Контрагенты |
| `WarehouseRobot` | Склады | Зоны, типы |
| `WarehouseRobotMain` | Основные склады | Главные склады |
| `WarehouseRobotMarking` | Маркировка складов | Коды маркировки |
| `WarehouseRobotNew` | Новые склады | Создание складов |
| `WarehouseZoneRobot` | Зоны склада | Зоны хранения |
| `WriteOffClassifierRobot` | Причины списания | Категории списания |
| `ZoneRobot` | Зоны | Зоны размещения |
| `SectorsRobot` | Сектора | Сектора склада |
| `CellsRobot` | Ячейки | Адресное хранение |
| `GatesRobot` | Ворота | Ворота склада |
| `FindParamsRobot` | Параметры поиска | Шаблоны поиска |
| `SluiceRobot` | Шлюзы | Шлюзовые зоны |
| `SCDRobot` | SCD | SCD-обработка |

---

## Роботы операций прихода

### `INCOMERobot` — базовый робот прихода

Проверяет полный цикл операции прихода товаров: создание документа прихода, приёмка по ASN, размещение.

```EME-L
INCOMERobot
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RobotBatchGo_RST(Robot AS "CIRobot")
{
    ' Открываем документ прихода '
    Robot.OpenDialog("Receipt")
    Robot.Wait(500)

    ' Создаём новый приход '
    Robot.ClickButton("New")
    Robot.Wait(300)

    ' Заполняем шапку '
    Robot.SelectItem("Warehouse", "Тестовый склад")
    Robot.SelectItem("Vendor", "Тестовый поставщик")
    Robot.SetText("DocDate", is_dos_date())

    ' Добавляем строку '
    Robot.ClickButton("AddLine")
    Robot.Wait(200)
    Robot.SelectItem("GoodsItem", "Тестовый товар")
    Robot.SetText("Quantity", "100")
    Robot.SetText("LotNo", "LOT_" + is_time())

    ' Сохраняем документ '
    Robot.ClickButton("Save")
    Robot.Wait(300)

    ' Проводим документ '
    Robot.ClickButton("Introduce")
    Robot.Wait(500)

    Robot.CloseDialog()
}
```

---

### `IncomingASNRobot` — приход по ASN

Проверяет приёмку товаров по предварительному уведомлению о поставке (ASN — Advance Shipping Notice).

```EME-L
IncomingASNRobot
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RobotBatchGo_RST(Robot AS "CIRobot")
{
    ' Открываем приёмку по ASN '
    Robot.OpenDialog("ReceiptPerASN")
    Robot.Wait(500)

    ' Выбираем ASN '
    Robot.SelectItem("ASN", "ASN_TEST_001")
    Robot.Wait(300)

    ' Подтверждаем приёмку '
    Robot.ClickButton("Confirm")
    Robot.Wait(500)

    ' Проверяем соответствие количеств '
    Robot.ClickButton("CheckQuantities")
    Robot.Wait(300)

    Robot.CloseDialog()
}
```

---

## Роботы операций отгрузки

### `OrdersBatchRobot` — отгрузка по заказам

Проверяет полный цикл отгрузки: формирование заказа, сборка, упаковка, отгрузка.

```EME-L
OrdersBatchRobot
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RobotBatchGo_RST(Robot AS "CIRobot")
{
    ' Открываем заказ '
    Robot.OpenDialog("Orders")
    Robot.Wait(500)

    ' Создаём новый заказ '
    Robot.ClickButton("New")
    Robot.Wait(300)

    ' Заполняем шапку '
    Robot.SelectItem("Client", "Тестовый клиент")
    Robot.SelectItem("Warehouse", "Тестовый склад")
    Robot.SetText("OrderNo", "ORD_" + is_time())

    ' Добавляем строки заказа '
    Robot.ClickButton("AddLine")
    Robot.Wait(200)
    Robot.SelectItem("GoodsItem", "Тестовый товар")
    Robot.SetText("Quantity", "50")

    ' Сохраняем '
    Robot.ClickButton("Save")
    Robot.Wait(300)

    ' Переводим в статус "Собирается" '
    Robot.ClickButton("StartAssembly")
    Robot.Wait(500)

    ' Завершаем отгрузку '
    Robot.ClickButton("Finish")
    Robot.Wait(500)

    Robot.CloseDialog()
}
```

---

## Роботы отчётов

### `ReportsRobots` — тестирование отчётов

Проверяет формирование печатных форм и отчётов: настройку параметров, выборку данных, экспорт в PDF.

```EME-L
ReportsRobots
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RobotBatchGo_RST(Robot AS "CIRobot")
{
    ' Открываем отчёт по остаткам '
    Robot.OpenDialog("StockReport")
    Robot.Wait(500)

    ' Устанавливаем параметры '
    Robot.SelectItem("Warehouse", "Тестовый склад")
    Robot.SetText("Date", is_dos_date())

    ' Формируем отчёт '
    Robot.ClickButton("Generate")
    Robot.Wait(1000)

    ' Экспортируем в PDF '
    Robot.ClickButton("ExportPDF")
    Robot.Wait(500)

    Robot.CloseDialog()
}
```

---

## Специальные роботы

### `RobotOpenDialogs` — тест открытия всех диалогов

Последовательно открывает все диалоги системы (~3 минуты). Не нажимает кнопки и не закрывает окна сообщений — тест должен пройти без ручного вмешательства.

```EME-L
RobotOpenDialogs
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RobotBatchGo_RST(Robot AS "CIRobot")
{
    ' Список всех диалогов для тестирования '
    Dialogs = Object("Array")
    Dialogs.Add("GoodsItem")
    Dialogs.Add("Clients")
    Dialogs.Add("Orders")
    Dialogs.Add("Receipt")
    Dialogs.Add("Despatch")
    ' ... все остальные диалоги ... '

    Loop (Dialogs)
        DialogName = Dialogs.Get()
        Robot.OpenDialog(DialogName)
        Robot.Wait(300)
        Robot.CloseDialog()
        Robot.Wait(200)
    End Loop
}
```

---

### `RobotInter` — интерфейсный робот

Проверяет работу интерфейсных элементов: браузеры, таблицы, фильтры, сортировку.

```EME-L
RobotInter
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RobotBatchGo_RST(Robot AS "CIRobot")
{
    ' Открываем браузер товаров '
    Robot.OpenDialog("GoodsItemBrowser")
    Robot.Wait(500)

    ' Устанавливаем фильтр '
    Robot.SetText("FilterCode", "TEST")
    Robot.ClickButton("ApplyFilter")
    Robot.Wait(300)

    ' Сортируем по наименованию '
    Robot.ClickButton("SortName")
    Robot.Wait(200)

    ' Выбираем первую строку '
    Robot.ClickRow(0)
    Robot.Wait(200)

    ' Открываем выбранный товар '
    Robot.ClickButton("Open")
    Robot.Wait(300)

    ' Закрываем диалог товара '
    Robot.CloseDialog()
    Robot.Wait(200)

    ' Закрываем браузер '
    Robot.CloseDialog()
}
```

---

### `CheckSpeedRobot` — робот проверки скорости

Измеряет время выполнения ключевых операций: открытие диалогов, поиск, формирование отчётов.

```EME-L
CheckSpeedRobot
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RobotBatchGo_RST(Robot AS "CIRobot")
{
    ' Измеряем время открытия диалога товаров '
    StartTime = is_tick_count()
    Robot.OpenDialog("GoodsItem")
    EndTime = is_tick_count()
    OpenTime = EndTime - StartTime
    Robot.CloseDialog()

    is_message("CheckSpeed", "Открытие GoodsItem: " + OpenTime + " мс", "OK", "INFORMATION")

    ' Измеряем время поиска '
    Robot.OpenDialog("GoodsItem")
    StartTime = is_tick_count()
    Robot.SetText("Search", "TEST")
    Robot.ClickButton("Find")
    EndTime = is_tick_count()
    SearchTime = EndTime - StartTime
    Robot.CloseDialog()

    is_message("CheckSpeed", "Поиск: " + SearchTime + " мс", "OK", "INFORMATION")
}
```

---

## Сводная таблица роботов

### Роботы классификаторов (35+)

| Робот | Справочник | Тип |
|-------|-----------|-----|
| `GoodsItemRobot` | Товары | Создание, ВГХ |
| `ClientRobot` | Клиенты | Создание, реквизиты |
| `WarehouseRobot` | Склады | Создание, зоны |
| `VendorRobot` | Поставщики | Контрагенты |
| `StaffRobot` | Сотрудники | Персонал |
| `MeasUnitRobot` | Единицы измерения | Факторы |
| `ProducerRobot` | Производители | Бренды |
| `CurrencyRobot` | Валюты | Курсы |
| `CellsRobot` | Ячейки | Адресация |
| `ZoneRobot` | Зоны | Хранение |

### Роботы операций (20+)

| Робот | Операция |
|-------|----------|
| `INCOMERobot` | Приход |
| `IncomingASNRobot` | Приход по ASN |
| `IncomingFastRobot` | Быстрый приход |
| `OrdersBatchRobot` | Отгрузка |
| `OrderNewRobot` | Новый заказ |
| `MoveInCellRobot` | Перемещение |
| `OffwriteRobot` | Списание |
| `InventoryNewRobot` | Инвентаризация |

### Специальные роботы

| Робот | Назначение |
|-------|----------|
| `RobotOpenDialogs` | Открытие всех диалогов |
| `RobotInter` | Интерфейсные тесты |
| `CheckSpeedRobot` | Проверка скорости |
| `RobotStatistics` | Статистика роботов |
| `DemoRobot` | Демонстрационный |

---

## Запуск робота через код

```EME-L
' Запустить робота для тестирования диалогов
is_run_robot("RobotOpenDialogs", 1000, 600, 0)
```

| Параметр | Описание |
|----------|----------|
| `args[0]` | Имя робота: `"RobotOpenDialogs"`, `"RobotStaffPosition"`, `"RobotStaff"` |
| `args[1]` | Пошаговая задержка в мс (по умолчанию 500) |
| `args[2]` | Таймаут в секундах (по умолчанию 500) |
| `args[3]` | Признак скрытия диалога (по умолчанию 0) |

---

## Контрольный список создания робота

- [ ] Создан класс с методом `RobotBatchGo_RST(Robot AS "CIRobot")`
- [ ] Прописан в конструкторе `AllRobots`
- [ ] Установлены разумные задержки между шагами
- [ ] Обработаны модальные окна сообщений
- [ ] Тест проходит без ручного вмешательства
- [ ] Проверена работа в монопольном режиме
