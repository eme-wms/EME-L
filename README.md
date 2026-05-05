# EME-L Documentation for Context7

Документация по языку программирования EME-L и системе управления складом EME.WMS. Оптимизирована для RAG-поиска через Context7.

## Структура документации (44 файла)

### Язык EME-L — Основы

- **docs/EME_L_Syntax.md** — переменные, 9 типов данных, операторы, управляющие конструкции
- **docs/EME_L_Classes.md** — объектная модель, Object(), IClass, интерфейсы CEME
- **docs/EME_L_Events.md** — все обработчики событий (24 диалог + таблицы + браузеры + интегратор + отчёты)
- **docs/EME_L_Constants.md** — 13 групп констант (DOCUMENT_TYPE_*, os*, MO_TYPE_*, ZONE_*)
- **docs/EME_L_IS_Functions.md** — is-функции групп Bit, Browser, Context

### Язык EME-L — Данные и память

- **docs/EME_L_Database.md** — классы Query, BitBuffer, Array, MultiArray, Map
- **docs/EME_L_Database_Advanced.md** — DBWatch, расширенная фильтрация MustBe*, транзакции
- **docs/EME_L_Memory.md** — правила памяти, Const, утечки, производительность
- **docs/EME_L_Records_and_Fields.md** — записи, поля, браузеры, цепочки, режимы обхода
- **docs/EME_SQL_Advanced.md** — полный EME-SQL (SELECT, WITH, JOIN, UNION, TOTAL)

### Язык EME-L — Продвинутые

- **docs/EME_L_Advanced_Features.md** — компоненты, ByRef/ByVal, tr()
- **docs/EME_L_Advanced_API_and_Patterns.md** — String (40+ методов), PropertyTree, боты
- **docs/EME_L_HTTP_Integration.md** — HTTP-архитектура, ТСД-коммуникация, Batch
- **docs/EME_L_Query_Reports.md** — отчёты, печатные формы, бэнды
- **docs/EME_L_Misc_and_Localization.md** — UI-тестирование, робот-тестеры, локализация

### Архитектура и администрирование

- **docs/EME_WMS_Theory_and_Architecture.md** — архитектура WMS, транзакции, выравнивание
- **docs/Business_Analytics.md** — история платформы, состав базовой версии
- **docs/Server_and_Services_Setup.md** — сервер, ServerJob, Монитор Заданий
- **docs/Performance_and_Logging.md** — логирование, диагностика, профилирование
- **docs/System_Archiving_and_Priorities.md** — архивация, приоритеты, алгоритмы

### Интеграция

- **docs/ERP_1C_Integration.md** — интеграция с 1С через промежуточную SQL-БД
- **docs/Web_Services_and_API.md** — JSON API, SOAP, PropertyTree
- **docs/PostgreSQL_ODBC_Integration.md** — ExternalDB для PostgreSQL через ODBC

### GUI, ТСД, сборка

- **docs/GUI_DB_Structure.md**, **GUI_Dialog_Editor.md**, **GUI_Project_Deployment.md**, **GUI_Data_Slices_and_Lessons.md**, **GUI_Activist_and_Configurator.md**
- **docs/TSD_Operations_and_Settings.md**, **TSD_WDC_Measurement.md**, **TSD_WDC_Cell_Routing.md**
- **docs/Build_Metaproject.md**, **Build_Android_TSD.md**
- **docs/AI_Programming_Assistant.md**, **EME_L_Examples.md**, **EME_WMS_Linux.md**

## Использование

Документация оптимизирована для RAG-поиска в Context7. Запросы на русском языке.

## Лицензия

© 2026 EME
