> Синонимы: интеграционные тесты, HTTP, RPC, XML, ExternalDB, переводы, 1С, ЦУП, локализация

## Интеграционные тесты

Интеграционные тесты проверяют взаимодействие EME.WMS с внешними системами и сервисами: HTTP-запросы, XML-обмен, удалённые вызовы процедур (RPC), подключение к внешним базам данных, локализацию и интеграцию с учётными системами.

В отличие от юнит-тестов, интеграционные тесты:
- Требуют доступа к внешним сервисам и системам
- Проверяют форматы обмена данными (JSON, XML)
- Тестируют сетевое взаимодействие
- Запускаются вручную разработчиком

---

## HTTP-запросы

### `Tests.TestHttpsReq` — тест HTTP(S)-запросов

Проверяет корректность выполнения HTTP и HTTPS-запросов, обработку ответов, пагинацию и работу с REST API. Используется для интеграции с веб-сервисами.

```EME-L
Tests.TestHttpsReq
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'Выполняем GET-запрос'
    Result = is_http_get("https://api.example.com/data");
    is_message("TestHttpsReq", "GET-ответ: " + is_left(Result, 100), "OK", "INFORMATION");

    'Выполняем POST-запрос с JSON'
    JsonBody = "{\"key\":\"value\",\"count\":42}";
    Result = is_http_post("https://api.example.com/submit", JsonBody, "application/json");
    is_message("TestHttpsReq", "POST-ответ: " + is_left(Result, 100), "OK", "INFORMATION");

    'Проверяем пагинацию'
    Page = 1;
    While (Page <= 3)
        Url = "https://api.example.com/data?page=" + Page;
        Result = is_http_get(Url);
        is_message("TestHttpsReq", "Страница " + Page + ": " + is_length(Result) + " байт", "OK", "INFORMATION");
        Page = Page + 1;
    End While
}
```

**Что проверяет:**
- GET-запросы по HTTP и HTTPS
- POST-запросы с JSON-данными
- Работу с пагинацией API
- Обработку ответов сервера

---

## RPC через файловую систему

### `Tests.TestRPC` — тест удалённых вызовов процедур

Проверяет механизм RPC (Remote Procedure Call) через файловую систему. Используется для межпроцессного взаимодействия между компонентами EME.WMS.

```EME-L
Tests.TestRPC
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'Формируем параметры для RPC-вызова'
    Params = Object("Parameters");
    Params.PutParam("Action", "Test");
    Params.PutParam("Data", "Hello from RPC test");

    'Выполняем RPC-вызов через файловую систему'
    Result = is_rpc_call("TestRPC", Params);

    If (Result == "OK")
        is_message("TestRPC", "RPC-вызов выполнен успешно", "OK", "INFORMATION");
    Else
        is_message("TestRPC", "Ошибка RPC: " + Result, "OK", "STOP");
    End If
}
```

**Что проверяет:**
- Формирование параметров RPC
- Вызов удалённой процедуры
- Обработку результата
- Корректность обмена через ФС

---

## XML-парсинг

### `Tests.TestXml` — тест парсинга XML

Проверяет корректность парсинга XML-документов, включая курсы валют Центрального Банка. Используется для интеграции с системами, использующими XML-формат.

```EME-L
Tests.TestXml
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'Загружаем XML с курсами ЦБ'
    XmlData = is_http_get("https://www.cbr.ru/scripts/XML_daily.asp");

    'Парсим XML'
    Xml = Object("Xml");
    Xml.LoadFromString(XmlData);

    'Получаем список валют'
    Currencies = Xml.GetNodes("Valute");
    is_message("TestXml", "Найдено валют: " + Currencies.GetSize(), "OK", "INFORMATION");

    'Читаем данные первой валюты'
    If (Currencies.GetSize() > 0)
        FirstCurrency = Currencies.GetAt(0);
        Code = FirstCurrency.GetValue("CharCode");
        Rate = FirstCurrency.GetValue("Value");
        is_message("TestXml", "Валюта: " + Code + " = " + Rate, "OK", "INFORMATION");
    End If
}
```

**Что проверяет:**
- Загрузку XML из строки
- Навигацию по XML-дереву
- Чтение значений узлов
- Работу со списками узлов

---

## Подключение к внешним БД

### `Tests.TestExternalDB` — тест подключения к внешним БД через ODBC

Проверяет подключение к внешним базам данных через ODBC. Поддерживает PostgreSQL, SQL Server, Oracle и EME DB.

```EME-L
Tests.TestExternalDB
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    ExternalDB = Object("ExternalDB");

    'Тест подключения к PostgreSQL'
    Error = ExternalDB.Connect(
        "ODBC:Driver={PostgreSQL ODBC Driver(ANSI)};Server=localhost;Port=5432;Database=Test;Uid=postgres;Pwd=123;");
    If (Error == "")
        is_message("TestExternalDB", "PostgreSQL: Подключение успешно", "OK", "INFORMATION");
        TestExternalDB(ExternalDB);
        ExternalDB.Disconnect();
    Else
        is_message("TestExternalDB", "PostgreSQL: " + Error, "OK", "EXCLAMATION");
    End If

    'Тест подключения к SQL Server'
    Error = ExternalDB.Connect(
        "ODBC:Driver={SQL Server};Server=localhost;Database=Test;Uid=sa;Pwd=123;");
    If (Error == "")
        is_message("TestExternalDB", "SQL Server: Подключение успешно", "OK", "INFORMATION");
        TestExternalDB(ExternalDB);
        ExternalDB.Disconnect();
    Else
        is_message("TestExternalDB", "SQL Server: " + Error, "OK", "EXCLAMATION");
    End If
}

TestExternalDB(ExternalDB)
{
    'Выполняем тестовый запрос'
    Query = Object("Query");
    Query.Create();
    Error = ExternalDB.Execute("SELECT 1 AS Test", Query);

    If (Error == "")
        is_message("TestExternalDB", "Запрос выполнен. Строк: " + Query.GetNoOfLines(), "OK", "INFORMATION");
    Else
        is_message("TestExternalDB", "Ошибка запроса: " + Error, "OK", "STOP");
    End If
}
```

**Что проверяет:**
- Подключение к различным СУБД через ODBC
- Выполнение SQL-запросов
- Корректное отключение
- Обработку ошибок

---

## Локализация

### `Tests.TestTranslate` — тест системы переводов

Проверяет корректность работы системы локализации (`tr()`). Убеждается, что все строковые ресурсы корректно переведены для текущего языка.

```EME-L
Tests.TestTranslate
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'Проверяем перевод строки'
    Original = "Hello World";
    Translated = tr(Original);
    is_message("TestTranslate", "Оригинал: " + Original + "\nПеревод: " + Translated, "OK", "INFORMATION");

    'Проверяем перевод системной строки'
    SysTranslated = tr("Сохранить изменения?");
    is_message("TestTranslate", "Системная строка: " + SysTranslated, "OK", "INFORMATION");

    'Проверяем текущий язык'
    CurrentLang = is_get_lang();
    is_message("TestTranslate", "Текущий язык: " + CurrentLang, "OK", "INFORMATION");
}
```

**Что проверяет:**
- Перевод строк через `tr()`
- Работу системы локализации
- Текущий язык интерфейса

---

### `Tests.TestTranslatePura` — тест чистых переводов

Проверяет работу чистых (не связанных с кодом) переводов — текстовых ресурсов, загружаемых из внешних файлов.

```EME-L
Tests.TestTranslatePura
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'Загружаем переводы из файла'
    File = Object("File", "D:\\EME\\Lang\\Custom.lang");
    If (File.Exists())
        LangData = File.GetData().AsString();
        is_message("TestTranslatePura", "Загружено переводов: " + is_length(LangData), "OK", "INFORMATION");
    Else
        is_message("TestTranslatePura", "Файл переводов не найден", "OK", "EXCLAMATION");
    End If
}
```

---

## Интеграция с 1С

### `Tests.TestOrderFrom1C` — тест заказов из 1С

Проверяет импорт заказов из учётной системы 1С через COM-автоматизацию. Критично для интеграции WMS с ERP-системами на базе 1С.

```EME-L
Tests.TestOrderFrom1C
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'Подключаемся к 1С через COM'
    OneC = Object("Automation", "V83.COMConnector");
    Connection = OneC.Connect("Srvr='localhost';Ref='MyBase';Usr='Admin';Pwd='';");

    If (Connection == "")
        is_message("TestOrderFrom1C", "Не удалось подключиться к 1С", "OK", "STOP");
        Return;
    End If

    'Получаем список заказов'
    Query = Connection.NewObject("Query");
    Query.Text = "ВЫБРАТЬ ПЕРВЫЕ 10 Номер, Дата, Контрагент ИЗ Документ.ЗаказКлиента";
    Result = Query.Execute().Выбрать();

    Count = 0;
    While (Result.Следующий())
        Count = Count + 1;
    End While

    is_message("TestOrderFrom1C", "Заказов из 1С: " + Count, "OK", "INFORMATION");
}
```

**Что проверяет:**
- Подключение к 1С через COM
- Выполнение запросов к 1С
- Чтение данных документов
- Обработку ошибок подключения

---

## Экспорт в ЦУП

### `Tests.TestExportToTHQ` — тест выгрузки в ЦУП

Проверяет выгрузку данных в Центр Управления Перевозками (ЦУП). Используется для логистических интеграций.

```EME-L
Tests.TestExportToTHQ
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'Формируем данные для выгрузки'
    r_Orders = Object("dsDB", "Orders");
    r_Orders.SetSkipMode();

    'Экспортируем заказы в формат ЦУП'
    File = Object("File", "D:\\Temp\\THQ_Export.xml");
    File.CreateFile();
    File.PutFileData("<?xml version=\"1.0\" encoding=\"UTF-8\"?>");
    File.PutFileData("<Orders>");

    Loop (r_Orders)
        Line = "<Order Number=\"" + r_Orders.GetOrderNo() + "\" Date=\"" + r_Orders.GetDocDate() + "\"/>";
        File.PutFileData(Line);
    End Loop

    File.PutFileData("</Orders>");
    File.CloseFile();

    is_message("TestExportToTHQ", "Выгружено заказов: " + r_Orders.GetNoOfLines(), "OK", "INFORMATION");
}
```

---

## Сводная таблица интеграционных тестов

| Класс / Файл | Что проверяет | Внешняя система |
|--------------|--------------|-----------------|
| `Tests.TestHttpsReq` | HTTP(S)-запросы, пагинация | Веб-сервисы, REST API |
| `Tests.TestRPC` | Удалённые вызовы через ФС | Внутренние компоненты |
| `Tests.TestXml` | Парсинг XML | ЦБ РФ, XML-сервисы |
| `Tests.TestExternalDB` | ODBC-подключение | PostgreSQL, SQL Server, Oracle |
| `Tests.TestTranslate` | Локализация (`tr()`) | Файлы переводов |
| `Tests.TestTranslatePura` | Чистые переводы | Файлы .lang |
| `Tests.TestOrderFrom1C` | Импорт заказов | 1С |
| `Tests.TestExportToTHQ` | Выгрузка в ЦУП | ЦУП |

---

## Контрольный список интеграционного теста

### HTTP-тест
- [ ] URL корректен и доступен
- [ ] Обработка таймаутов
- [ ] Проверка формата ответа
- [ ] Обработка ошибок HTTP

### Тест внешней БД
- [ ] Строка подключения корректна
- [ ] Нет захардкоженных паролей в коммите
- [ ] Корректное отключение (`Disconnect()`)
- [ ] Обработка ошибок подключения

### Тест локализации
- [ ] Проверены все поддерживаемые языки
- [ ] Отсутствуют непереведённые строки
- [ ] Файлы переводов доступны
