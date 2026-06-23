# Тестирование в EME-L

Синонимы: тесты, тестирование, юнит-тесты, ERP-тесты, самодиагностика, RobotHealer, healClasses, CIRobot, TestJson, DBWatch, шаблоны тестов, роботы-тестировщики, UI-тест, тест интерфейса, тест целостности БД, is_run_dbtest, is_ui_test_mode

## Обзор системы тестов

В проекте EME.WMS существует несколько семейств тестов, каждое со своей целью, архитектурой и способом запуска. Все тестовые классы располагаются в директории `Классы/` базовой выгрузки проекта.

### Семейства тестов

| Семейство | Пространство имён / Запуск | Назначение |
|-----------|---------------------------|------------|
| **Юнит-тесты** | `Tests.TestJson.*` | Изолированная проверка методов объекта `Json` / `PropertyTree` |
| **ERP-тесты** | `ERPSolution.ERPTests` | Интеграционные тесты обмена данными между WMS и ERP-системой |
| **Самодиагностика** | `RobotHealer.healClasses.*` | Проверка здоровья БД, скорости операций, исправление проблем |
| **Роботы-тестировщики** | `Система → Ресурсы → Тестирование → Робот` | Автоматические тесты классификаторов и операций (CIRobot) |
| **UI-тест диалогов** | `Система → Ресурсы → Тестирование → Тест интерфейса` | Последовательное открытие всех диалогов (~3 мин) |
| **Тест целостности БД** | `is_run_dbtest()` | Физический и логический тест структуры базы данных |
| **SQL-тесты** | `Запросы/Tests.*` | Проверка целостности данных через SQL-запросы |
| **Тесты браузеров** | `Браузеры/TerminalTester.*` | Табличные представления данных для терминалов |
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

| Файл | Класс | Что проверяет |
|------|-------|---------------|
| `TestJson.txt` | `Tests.TestJson` | Раннер: `RunTests()`, `CHECK_EQ()`, счётчики |
| `TestJson_001__GetType.txt` | `TestJson_001__GetType` | `GetType()` — распознавание типов узлов |
| `TestJson_002__GetTypeAt.txt` | `TestJson_002__GetTypeAt` | `GetTypeAt()` — тип узла по пути |
| `TestJson_003__GetCurrentType.txt` | `TestJson_003__GetCurrentType` | `GetCurrentType()` — тип текущего узла при итерации |
| `TestJson_004__IsType_typed.txt` | `TestJson_004__IsType_typed` | `IsType()` — проверка типа с константами |
| `TestJson_005__IsTypeAt_typed.txt` | `TestJson_005__IsTypeAt_typed` | `IsTypeAt()` — проверка типа по пути |
| `TestJson_006__IsCurrentType_typed.txt` | `TestJson_006__IsCurrentType_typed` | `IsCurrentType()` — проверка типа текущего узла |
| `TestJson_007__IsKeyExists.txt` | `TestJson_007__IsKeyExists` | `IsKeyExists()` — наличие ключа в объекте |
| `TestJson_008__GetValue.txt` | `TestJson_008__GetValue` | `GetValue()` — чтение значения по ключу |
| `TestJson_009__GetValueAt.txt` | `TestJson_009__GetValueAt` | `GetValueAt()` — чтение по пути |
| `TestJson_010__GetValueAtOrDefault.txt` | `TestJson_010__GetValueAtOrDefault` | `GetValueAtOrDefault()` — значение или дефолт |
| `TestJson_011__GetCurrentValue.txt` | `TestJson_011__GetCurrentValue` | `GetCurrentValue()` — значение при итерации |
| `TestJson_012__GetNodeAt.txt` | `TestJson_012__GetNodeAt` | `GetNodeAt()` — получение под-узла по пути |
| `TestJson_013__GetCurrentNode.txt` | `TestJson_013__GetCurrentNode` | `GetCurrentNode()` — текущий узел итерации |
| `TestJson_014__PutValue.txt` | `TestJson_014__PutValue` | `PutValue()` — запись значения по ключу |
| `TestJson_015__PutValueAt.txt` | `TestJson_015__PutValueAt` | `PutValueAt()` — запись по пути |
| `TestJson_016__IsEmpty.txt` | `TestJson_016__IsEmpty` | `IsEmpty()` — пустой ли узел |
| `TestJson_017__GetSize.txt` | `TestJson_017__GetSize` | `GetSize()` — размер массива/объекта |
| `TestJson_018__AddNodeAt.txt` | `TestJson_018__AddNodeAt` | `AddNodeAt()` — добавление узла по пути |
| `TestJson_019__ResizeArray.txt` | `TestJson_019__ResizeArray` | `ResizeArray()` — изменение размера массива |
| `TestJson_020__PushBackValue.txt` | `TestJson_020__PushBackValue` | `PushBackValue()` — добавление значения в массив |
| `TestJson_021__PushBackNode.txt` | `TestJson_021__PushBackNode` | `PushBackNode()` — добавление узла в массив |
| `TestJson_022__PutArray.txt` | `TestJson_022__PutArray` | `PutArray()` — запись массива |
| `TestJson_023__PutArrayAt.txt` | `TestJson_023__PutArrayAt` | `PutArrayAt()` — запись массива по пути |
| `TestJson_024__GetArray.txt` | `TestJson_024__GetArray` | `GetArray()` — чтение массива |
| `TestJson_025__GetArrayAt.txt` | `TestJson_025__GetArrayAt` | `GetArrayAt()` — чтение массива по пути |
| `TestJson_026__PutQuery.txt` | `TestJson_026__PutQuery` | `PutQuery()` — загрузка `Query` в JSON |
| `TestJson_027__PutQueryAt.txt` | `TestJson_027__PutQueryAt` | `PutQueryAt()` — загрузка `Query` по пути |
| `TestJson_028__GetQuery.txt` | `TestJson_028__GetQuery` | `GetQuery()` — выгрузка JSON в `Query` |
| `TestJson_029__GetQueryAt.txt` | `TestJson_029__GetQueryAt` | `GetQueryAt()` — выгрузка по пути |
| `TestJson_030__EraseAt.txt` | `TestJson_030__EraseAt` | `EraseAt()` — удаление по пути |
| `TestJson_031__EraseCurrent.txt` | `TestJson_031__EraseCurrent` | `EraseCurrent()` — удаление текущего узла |
| `TestJson_032__EraseAll.txt` | `TestJson_032__EraseAll` | `EraseAll()` — очистка узла |
| `TestJson_033__SetFirstLine.txt` | `TestJson_033__SetFirstLine` | `SetFirstLine()` — итерация по узлам |
| `TestJson_034__IsValidLine.txt` | `TestJson_034__IsValidLine` | `IsValidLine()` — валидность текущей строки |
| `TestJson_035__SetNextLine.txt` | `TestJson_035__SetNextLine` | `SetNextLine()` — переход к следующему узлу |
| `TestJson_036__ResetIteration.txt` | `TestJson_036__ResetIteration` | `ResetIteration()` — сброс итератора |
| `TestJson_037__GetCurrentKey.txt` | `TestJson_037__GetCurrentKey` | `GetCurrentKey()` — ключ текущего узла |
| `TestJson_038__GetCurrentIndex.txt` | `TestJson_038__GetCurrentIndex` | `GetCurrentIndex()` — индекс текущего узла |
| `TestJson_039__iteration.txt` | `TestJson_039__iteration` | Комплексная проверка итерации |
| `TestJson_040__ParseNoExcept.txt` | `TestJson_040__ParseNoExcept` | `ParseNoExcept()` — парсинг без исключений |
| `TestJson_041__Parse.txt` | `TestJson_041__Parse` | `Parse()` — парсинг строки |
| `TestJson_042__DumpToStr.txt` | `TestJson_042__DumpToStr` | `DumpToStr()` — сериализация в строку |
| `TestJson_043__DumpToStrFormatted.txt` | `TestJson_043__DumpToStrFormatted` | `DumpToStrFormatted()` — форматированная сериализация |
| `TestJson_044__GetContent.txt` | `TestJson_044__GetContent` | `GetContent()` — получение «сырого» содержимого |
| `TestJson_045__codepage.txt` | `TestJson_045__codepage` | Работа с кодировками (`ANSI`/`UTF-8`) |

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

### D. SQL-тесты (`Запросы/Tests.*`)

| Файл | Класс | Что проверяет | Тип |
|------|-------|--------------|-----|
| `Tests.DupKeys.txt` | `Tests.DupKeys` | Дубли ключевых значений | Целостность |
| `Tests.DupLotID.txt` | `Tests.DupLotID` | Дубли идентификаторов партий | Целостность |
| `Tests.DupLotNo.txt` | `Tests.DupLotNo` | Дубли номеров партий | Целостность |
| `Tests.TestDocLinesDups.txt` | `Tests.TestDocLinesDups` | Дубли строк документов | Целостность |
| `Tests.TestRegistersDups.txt` | `Tests.TestRegistersDups` | Дубли в регистрах | Целостность |
| `Tests.TestSpLock.txt` | `Tests.TestSpLock` | Блокировки хранимых процедур | Параллельный доступ |
| `Tests.TestStoredProcedure.txt` | `Tests.TestStoredProcedure` | Корректность хранимых процедур | Функциональность |
| `Tests.TestWITH.txt` | `Tests.TestWITH` | CTE-запросы | SQL-функциональность |
| `Tests.TestGUID.txt` | `Tests.TestGUID` | Уникальность GUID | Целостность |
| `Tests.ExternalDB.txt` | `Tests.ExternalDB` | Подключение к внешним БД (ODBC) | Интеграция |
| `Tests.SqlServer.txt` | `Tests.SqlServer` | Специфика SQL Server | Совместимость |

### E. Тесты записей и БД

| Файл | Класс | Что проверяет | Объекты |
|------|-------|--------------|---------|
| `TestCEMERec.txt` | `Tests.TestCEMERec` | Базовый класс `CEMERec` | `dsDB` |
| `TestCEMESkip.txt` | `Tests.TestCEMESkip` | Фильтрация `MustBeEQAbt` | `dsDB` |
| `TestFindLine.txt` | `Tests.TestFindLine` | Хэшированный поиск | `dsDB` |
| `TestPickByLine.txt` | `Tests.TestPickByLine` | `GetParamsByTransitOrPBL` | `dsDB` |
| `TestGoodsItem.txt` | `Tests.TestGoodsItem` | Точность `GoodsItem` | `dsDB` |
| `TestCommit.txt` | `Tests.TestCommit` | Транзакции (фиксация) | `dsDB` |
| `TestDBWatch.txt` | `Tests.TestDBWatch` | `DBWatch` (дозор БД) | `DBWatch`, `dsDB` |

### F. Интеграционные тесты

| Файл | Класс | Что проверяет | Внешняя система |
|------|-------|--------------|-----------------|
| `TestExternalDB.txt` | `Tests.TestExternalDB` | ODBC-подключение | PostgreSQL, SQL Server, Oracle |
| `TestHttpsReq.txt` | `Tests.TestHttpsReq` | HTTP(S)-запросы, пагинация | Веб-сервисы, REST API |
| `TestRPC.txt` | `Tests.TestRPC` | Удалённые вызовы через ФС | Внутренние компоненты |
| `TestXml.txt` | `Tests.TestXml` | Парсинг XML | ЦБ РФ, XML-сервисы |
| `TestTranslate.txt` | `Tests.TestTranslate` | Локализация (`tr()`) | Файлы переводов |
| `TestOrderFrom1C.txt` | `Tests.TestOrderFrom1C` | Импорт заказов | 1С |
| `TestExportToTHQ.txt` | `Tests.TestExportToTHQ` | Выгрузка в ЦУП | ЦУП |

### G. Роботы-тестировщики

| Файл | Класс | Что проверяет | Тип |
|------|-------|--------------|-----|
| `GuideRobots.txt` | `GuideRobots` | Базовая логика классификаторов | Классификаторы |
| `GoodsItemRobot.txt` | `GoodsItemRobot` | Справочник товаров | Классификаторы |
| `ClientRobot.txt` | `ClientRobot` | Справочник клиентов | Классификаторы |
| `WarehouseRobot.txt` | `WarehouseRobot` | Справочник складов | Классификаторы |
| `INCOMERobot.txt` | `INCOMERobot` | Операция прихода | Операции |
| `IncomingASNRobot.txt` | `IncomingASNRobot` | Приход по ASN | Операции |
| `OrdersBatchRobot.txt` | `OrdersBatchRobot` | Отгрузка по заказам | Операции |
| `ReportsRobots.txt` | `ReportsRobots` | Печатные формы | Отчёты |
| `RobotOpenDialogs.txt` | `RobotOpenDialogs` | Открытие всех диалогов | UI-тест |
| `CheckSpeedRobot.txt` | `CheckSpeedRobot` | Скорость операций | Производительность |

---

## Навигация по документации тестирования

- [Юнит-тесты](EME_L_Testing_Unit.md) — как писать и запускать изолированные тесты объектов
- [ERP-тесты](EME_L_Testing_ERP.md) — интеграционные тесты обмена с ERP
- [Самодиагностика](EME_L_Testing_SelfDiagnostics.md) — система KPI, роботы-тестировщики, UI-тесты, тесты целостности БД, DBWatch
- [Шаблоны тестов](EME_L_Testing_Templates.md) — готовые заготовки для новых тестов
- [SQL-тесты](sql-tests.md) — проверка целостности данных через SQL-запросы
- [Тесты записей и БД](record-tests.md) — тестирование CEMERec, CEMESkip, DBWatch, транзакций
- [Интеграционные тесты](integration-tests.md) — HTTP, RPC, XML, внешние БД, 1С
- [Роботы-тестировщики](robot-tests.md) — автоматические тесты классификаторов и операций
- [Тесты браузеров](browser-tests.md) — браузеры терминалов и интерфейсные тесты
- [Прочие тесты](misc-tests.md) — изображения, ошибки, стресс-тесты, ВЕТИС, EANCOM, Диадок
