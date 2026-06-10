# Тестирование в EME-L

Синонимы: тесты, тестирование, юнит-тесты, ERP-тесты, самодиагностика, RobotHealer, healClasses, CIRobot, TestJson, DBWatch, шаблоны тестов

## Обзор системы тестов

В проекте EME.WMS существует несколько семейств тестов, каждое со своей целью, архитектурой и способом запуска. Все тестовые классы располагаются в директории `Классы/` базовой выгрузки проекта.

### Семейства тестов

| Семейство | Пространство имён | Назначение |
|-----------|-------------------|------------|
| **Юнит-тесты** | `Tests.TestJson.*` | Изолированная проверка методов объекта `Json` / `PropertyTree` |
| **ERP-тесты** | `ERPSolution.ERPTests` | Интеграционные тесты обмена данными между WMS и ERP-системой |
| **Самодиагностика** | `RobotHealer.healClasses.*` | Проверка здоровья БД, скорости операций, исправление проблем |
| **Общие / ad-hoc** | `Tests.Test`, `Tests.TestUnit` | Разработческие «песочницы», эксперименты, ручные проверки |

### Сравнение типов тестов

| Характеристика | Юнит-тесты | ERP-тесты | Самодиагностика |
|----------------|-----------|-----------|-----------------|
| **Что проверяет** | Один объект/метод | Интеграция с внешней ERP | Состояние БД и настроек |
| **Запуск** | Через `RunTests()` | Вручную по кнопке | Автоматически / по расписанию |
| **Результат** | passed / failed / total | Сообщение + лог | Оценка 1–10 + отчёт |
| **Классический assert** | Да (`CHECK_EQ`) | Нет | Нет |
| **Внешние зависимости** | Нет | Да (ERP, ODBC, БД) | Нет |

---

## Сводная таблица: класс → тест → что проверяет

### A. Юнит-тесты объекта `Json` (`Tests.TestJson.*`)

Основной раннер: `Tests.TestJson` (`TestJson.txt`). Каждый подтест — отдельный класс, проверяющий один метод `Json`.

| Файл | Класс | Что проверяет | Объект |
|------|-------|---------------|--------|
| `TestJson.txt` | `Tests.TestJson` | Раннер: `RunTests()`, `CHECK_EQ()`, счётчики | `Json` |
| `TestJson_001__GetType.txt` | `TestJson_001__GetType` | `GetType()` — распознавание типов узлов | `Json` |
| `TestJson_002__GetTypeAt.txt` | `TestJson_002__GetTypeAt` | `GetTypeAt()` — тип узла по пути | `Json` |
| `TestJson_003__GetCurrentType.txt` | `TestJson_003__GetCurrentType` | `GetCurrentType()` — тип текущего узла при итерации | `Json` |
| `TestJson_004__IsType_typed.txt` | `TestJson_004__IsType_typed` | `IsType()` — проверка типа с константами | `Json` |
| `TestJson_005__IsTypeAt_typed.txt` | `TestJson_005__IsTypeAt_typed` | `IsTypeAt()` — проверка типа по пути | `Json` |
| `TestJson_006__IsCurrentType_typed.txt` | `TestJson_006__IsCurrentType_typed` | `IsCurrentType()` — проверка типа текущего узла | `Json` |
| `TestJson_007__IsKeyExists.txt` | `TestJson_007__IsKeyExists` | `IsKeyExists()` — наличие ключа в объекте | `Json` |
| `TestJson_008__GetValue.txt` | `TestJson_008__GetValue` | `GetValue()` — чтение значения по ключу | `Json` |
| `TestJson_009__GetValueAt.txt` | `TestJson_009__GetValueAt` | `GetValueAt()` — чтение по пути | `Json` |
| `TestJson_010__GetValueAtOrDefault.txt` | `TestJson_010__GetValueAtOrDefault` | `GetValueAtOrDefault()` — значение или дефолт | `Json` |
| `TestJson_011__GetCurrentValue.txt` | `TestJson_011__GetCurrentValue` | `GetCurrentValue()` — значение при итерации | `Json` |
| `TestJson_012__GetNodeAt.txt` | `TestJson_012__GetNodeAt` | `GetNodeAt()` — получение под-узла по пути | `Json` |
| `TestJson_013__GetCurrentNode.txt` | `TestJson_013__GetCurrentNode` | `GetCurrentNode()` — текущий узел итерации | `Json` |
| `TestJson_014__PutValue.txt` | `TestJson_014__PutValue` | `PutValue()` — запись значения по ключу | `Json` |
| `TestJson_015__PutValueAt.txt` | `TestJson_015__PutValueAt` | `PutValueAt()` — запись по пути | `Json` |
| `TestJson_016__IsEmpty.txt` | `TestJson_016__IsEmpty` | `IsEmpty()` — пустой ли узел | `Json` |
| `TestJson_017__GetSize.txt` | `TestJson_017__GetSize` | `GetSize()` — размер массива/объекта | `Json` |
| `TestJson_018__AddNodeAt.txt` | `TestJson_018__AddNodeAt` | `AddNodeAt()` — добавление узла по пути | `Json` |
| `TestJson_019__ResizeArray.txt` | `TestJson_019__ResizeArray` | `ResizeArray()` — изменение размера массива | `Json` |
| `TestJson_020__PushBackValue.txt` | `TestJson_020__PushBackValue` | `PushBackValue()` — добавление значения в массив | `Json` |
| `TestJson_021__PushBackNode.txt` | `TestJson_021__PushBackNode` | `PushBackNode()` — добавление узла в массив | `Json` |
| `TestJson_022__PutArray.txt` | `TestJson_022__PutArray` | `PutArray()` — запись массива | `Json` |
| `TestJson_023__PutArrayAt.txt` | `TestJson_023__PutArrayAt` | `PutArrayAt()` — запись массива по пути | `Json` |
| `TestJson_024__GetArray.txt` | `TestJson_024__GetArray` | `GetArray()` — чтение массива | `Json` |
| `TestJson_025__GetArrayAt.txt` | `TestJson_025__GetArrayAt` | `GetArrayAt()` — чтение массива по пути | `Json` |
| `TestJson_026__PutQuery.txt` | `TestJson_026__PutQuery` | `PutQuery()` — загрузка `Query` в JSON | `Json` |
| `TestJson_027__PutQueryAt.txt` | `TestJson_027__PutQueryAt` | `PutQueryAt()` — загрузка `Query` по пути | `Json` |
| `TestJson_028__GetQuery.txt` | `TestJson_028__GetQuery` | `GetQuery()` — выгрузка JSON в `Query` | `Json` |
| `TestJson_029__GetQueryAt.txt` | `TestJson_029__GetQueryAt` | `GetQueryAt()` — выгрузка по пути | `Json` |
| `TestJson_030__EraseAt.txt` | `TestJson_030__EraseAt` | `EraseAt()` — удаление по пути | `Json` |
| `TestJson_031__EraseCurrent.txt` | `TestJson_031__EraseCurrent` | `EraseCurrent()` — удаление текущего узла | `Json` |
| `TestJson_032__EraseAll.txt` | `TestJson_032__EraseAll` | `EraseAll()` — очистка узла | `Json` |
| `TestJson_033__SetFirstLine.txt` | `TestJson_033__SetFirstLine` | `SetFirstLine()` — итерация по узлам | `Json` |
| `TestJson_034__IsValidLine.txt` | `TestJson_034__IsValidLine` | `IsValidLine()` — валидность текущей строки | `Json` |
| `TestJson_035__SetNextLine.txt` | `TestJson_035__SetNextLine` | `SetNextLine()` — переход к следующему узлу | `Json` |
| `TestJson_036__ResetIteration.txt` | `TestJson_036__ResetIteration` | `ResetIteration()` — сброс итератора | `Json` |
| `TestJson_037__GetCurrentKey.txt` | `TestJson_037__GetCurrentKey` | `GetCurrentKey()` — ключ текущего узла | `Json` |
| `TestJson_038__GetCurrentIndex.txt` | `TestJson_038__GetCurrentIndex` | `GetCurrentIndex()` — индекс текущего узла | `Json` |
| `TestJson_039__iteration.txt` | `TestJson_039__iteration` | Комплексная проверка итерации | `Json` |
| `TestJson_040__ParseNoExcept.txt` | `TestJson_040__ParseNoExcept` | `ParseNoExcept()` — парсинг без исключений | `Json` |
| `TestJson_041__Parse.txt` | `TestJson_041__Parse` | `Parse()` — парсинг строки | `Json` |
| `TestJson_042__DumpToStr.txt` | `TestJson_042__DumpToStr` | `DumpToStr()` — сериализация в строку | `Json` |
| `TestJson_043__DumpToStrFormatted.txt` | `TestJson_043__DumpToStrFormatted` | `DumpToStrFormatted()` — форматированная сериализация | `Json` |
| `TestJson_044__GetContent.txt` | `TestJson_044__GetContent` | `GetContent()` — получение «сырого» содержимого | `Json` |
| `TestJson_045__codepage.txt` | `TestJson_045__codepage` | Работа с кодировками (`ANSI`/`UTF-8`) | `Json` |

### B. ERP-тесты (`ERPSolution.ERPTests`)

Класс: `ERPSolution.ERPTests` (`ERPTests.txt`)

| Метод | Что тестирует | Объекты |
|-------|---------------|---------|
| `Test()` | Создание таблиц во внешней БД, экспорт товаров (`ERPGoodsComponent`) | `ExternalDB`, `Query` |
| `Connect()` | Подключение к внешней БД по ODBC (PostgreSQL / SQL Server / EME DB) | `ExternalDB` |
| `Execute()` | Выполнение SQL-запроса во внешней БД | `ExternalDB`, `Query` |
| `CreateTable()` / `UpdateTable()` | DDL/DML операции через `ExternalDB` | `ExternalDB`, `Query` |
| `TestNewMessagesSpeed1/2/3()` | Скорость обнаружения новых сообщений во внешней БД | `ExternalDB` |
| `TestERPEngineSharp()` | Тест C#-движка `EME.ERPEngine` (экспорт товаров) | `Automation` |
| `CreateQualityChangeMessage()` | Формирование сообщения `quality_change` | `Automation` |
| `CreateChangeMessage()` | Формирование сообщения `change` | `Automation` |
| `CreateClientsMessage()` | Формирование сообщения `clients` | `Automation` |
| `CreateOrdersMessage()` | Формирование сообщения `orders` | `Automation` |
| `TestSqlServer()` | Прямой SQL-запрос к EME SQL Server | `EMEQuerySQLExternalDB` |
| `TestEnumTables()` | Перечисление таблиц внешней БД | `ExternalDB` |
| `TestImportReceipts()` | Импорт уведомлений о приёмке (`receipt`) | `Automation` |
| `TestOnCloseDocumentInThread()` | Экспорт документа в фоновом потоке | `System` |
| `TestJsonUpload()` / `TestJsonLoad()` | Выгрузка / загрузка сообщения в JSON | `PropertyTree`, `IDoc` |
| `TestXmlUpload()` / `TestXmlLoad()` | Выгрузка / загрузка сообщения в XML | `Xml` |
| `CreateWordDocument()` | Автоматизация Word (`Word.Application`) | `Automation` |
| `ExtractFields()` / `TestExtractFields()` | Извлечение полей, используемых в ERP | `IClass` |
| `ImportGoodsFlatFile()` | Импорт номенклатуры из CSV/TXT | `EMEQueryTextFile` |

### C. Самодиагностика (`RobotHealer.healClasses.*`)

| Файл | Класс | Что тестирует / что лечит |
|------|-------|---------------------------|
| `testExample.txt` | `testExample` | Шаблон теста производительности (балл 1–10) |
| `testDebug.txt` | `testDebug` | Режим отладки (`d_emel`) и флаг дампа |
| `testChainRestore.txt` | `testChainRestore` | Разрывы цепочек в `DocLines` (`Prev`/`Next`) |
| `testAccessibility.txt` | `testAccessibility` | Корректность флагов доступности ячеек |
| `testFindNonAddrCell.txt` | `testFindNonAddrCell` | Состояние безадресных ячеек |
| `testFindCell.txt` | `testFindCell` | Скорость автопоиска ячеек для размещения |
| `testGood.txt` | `testGood` | Заглушка для тестов проблем товаров |
| `testGood_openDialog.txt` | `testGood_openDialog` | Скорость открытия диалога «Продукт» |
| `testStructure.txt` | `testStructure` | Корректность сортировочных полей-цепочек |
| `testSelfTest.txt` | `testSelfTest` | Работоспособность модуля самодиагностики |
| `testRegisters.txt` | `testRegisters` | Фрагментация записи регистров |
| `testKernelSettings.txt` | `testKernelSettings` | Параметры ядра: архивы, HTTP, DLL, кэш |

---

## Навигация по документации тестирования

- [Юнит-тесты](EME_L_Testing_Unit.md) — как писать и запускать изолированные тесты объектов
- [ERP-тесты](EME_L_Testing_ERP.md) — интеграционные тесты обмена с ERP
- [Самодиагностика](EME_L_Testing_SelfDiagnostics.md) — система KPI, роботы-тестировщики, UI-тесты, тесты БД
- [Шаблоны тестов](EME_L_Testing_Templates.md) — готовые заготовки для новых тестов
