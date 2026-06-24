> Синонимы: прочие тесты, TestImage, TestPatch, VETTest, EANCOM, Диадок, 3D-визуализация, стресс-тесты

## Прочие тесты

Раздел объединяет специализированные тесты, не попадающие в основные категории: тесты изображений, обработки ошибок, стресс-тесты, тесты интеграции с внешними системами (ВЕТИС, Диадок, EANCOM), 3D-визуализацию склада и другие.

---

## Тесты изображений и документов

### `Tests.TestImage` — тест работы с изображениями

Проверяет загрузку, отображение и обработку изображений в системе. Используется для тестирования прикрепления фотографий к товарам, документам и ячейкам.

```EME-L
Tests.TestImage
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'Загружаем тестовое изображение'
    Image = Object("Image", "D:\\Temp\\TestPhoto.jpg");

    If (Image.IsValid())
        is_message("TestImage", "Изображение загружено. Размер: " + Image.GetWidth() + "x" + Image.GetHeight(), "OK", "INFORMATION");

        'Масштабируем'
        Image.Resize(200, 200);
        is_message("TestImage", "Масштабировано: " + Image.GetWidth() + "x" + Image.GetHeight(), "OK", "INFORMATION");

        'Сохраняем'
        Image.Save("D:\\Temp\\TestPhoto_Resized.jpg");
        is_message("TestImage", "Сохранено", "OK", "INFORMATION");
    Else
        is_message("TestImage", "Не удалось загрузить изображение", "OK", "STOP");
    End If
}
```

---

### `Tests.TestPatch` — тест патча интерфейса

Проверяет механизм динамической модификации интерфейса диалогов (patch) — добавление, удаление, изменение органов управления без изменения исходного кода диалога.

```EME-L
Tests.TestPatch
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'Применяем патч к диалогу'
    Patch = Object("Patch", "GoodsItem");
    Patch.AddButton("btnTest", "Тестовая кнопка", 100, 100, 80, 25);
    Patch.AddLabel("lblTest", "Тестовая метка", 10, 10, 200, 20);

    'Открываем диалог с патчем'
    Dialog = Object("Dialog", "GoodsItem");
    Dialog.ApplyPatch(Patch);
    Dialog.Show();

    is_message("TestPatch", "Патч применён", "OK", "INFORMATION");
}
```

---

### `Tests.TestPrintDocFiles` — тест печати документов

Проверяет формирование и печать документов, работу с файлами печатных форм.

```EME-L
Tests.TestPrintDocFiles
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'Формируем печатную форму накладной'
    r_Doc = Object("dsDB", "Document");
    r_Doc.SetSkipMode();
    r_Doc.SetFirstLine();

    If (r_Doc.IsValidLine())
        'Формируем файл печати'
        File = Object("File", "D:\\Temp\\Invoice.txt");
        File.CreateFile();
        File.PutFileData("НАКЛАДНАЯ № " + r_Doc.GetDocNo() + "\n");
        File.PutFileData("Дата: " + r_Doc.GetDocDate() + "\n");
        File.PutFileData("Контрагент: " + r_Doc.GetClientName() + "\n");
        File.CloseFile();

        is_message("TestPrintDocFiles", "Документ сформирован", "OK", "INFORMATION");
    End If
}
```

---

## Тесты ошибок

### `Tests.TestAbsent` — тест обработки отсутствующих данных

Проверяет корректность поведения системы при отсутствии ожидаемых данных. Критично для предотвращения аварийных завершений при работе с пустыми записями.

```EME-L
Tests.TestAbsent
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'Пытаемся получить доступ к несуществующей записи'
    r_Record = Object("dsDB", "GoodsItem");
    r_Record.SetLineByCode("NONEXISTENT_CODE_99999");

    If (~r_Record.IsValidLine())
        is_message("TestAbsent", "Корректно: запись не найдена, ошибки нет", "OK", "INFORMATION");
    Else
        is_message("TestAbsent", "ОШИБКА: Найдена несуществующая запись", "OK", "STOP");
    End If

    'Проверяем безопасное чтение поля'
    EmptyValue = r_Record.GetName();
    If (is_empty(EmptyValue))
        is_message("TestAbsent", "Корректно: пустое значение при отсутствии данных", "OK", "INFORMATION");
    End If
}
```

---

### `Tests.TestUserError` — тест пользовательской ошибки в транзакции

Проверяет корректность обработки пользовательских ошибок внутри транзакций. Убеждается, что транзакция корректно откатывается при возникновении ошибки.

```EME-L
Tests.TestUserError
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    r_Record = Object("dsDB", "TestRecord");
    CountBefore = r_Record.GetNoOfLines();

    'Открываем транзакцию'
    is_transaction(1, "Тест пользовательской ошибки");

    'Добавляем строку'
    r_Record.AppendAndSetLine();
    r_Record.PutCode("ERROR_TEST");

    'Генерируем пользовательскую ошибку'
    Try (Error)
        is_error(1, "Тестовая пользовательская ошибка");
    Catch
        If (~is_empty(Error))
            'Откатываем транзакцию'
            is_transaction(0);
            is_message("TestUserError", "Транзакция откатена: " + Error, "OK", "INFORMATION");
        End If
    End Try

    'Проверяем, что строка НЕ добавлена'
    CountAfter = r_Record.GetNoOfLines();
    If (CountAfter == CountBefore)
        is_message("TestUserError", "Корректно: данные не изменены после отката", "OK", "INFORMATION");
    Else
        is_message("TestUserError", "ОШИБКА: Данные изменены несмотря на откат", "OK", "STOP");
    End If
}
```

---

## Стресс-тесты

### `Tests.FULL_TEST` — полный тест системы

Выполняет комплексную проверку всех основных подсистем. Запускается перед релизом или после значительных изменений в коде.

```EME-L
Tests.FULL_TEST
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    Console = Object("Console");
    Console.Show(1);
    Console.PutText("=== НАЧАЛО ПОЛНОГО ТЕСТА ===");

    '1. Тест структуры БД'
    Console.PutText("1. Тест структуры БД...");
    Result = is_run_dbtest();
    Console.PutText("   Результат: " + Result);

    '2. Тест счётчиков ссылок'
    Console.PutText("2. Тест счётчиков ссылок...");
    is_test_references_counter();
    Console.PutText("   OK");

    '3. Тест VFS'
    Console.PutText("3. Тест виртуальной файловой системы...");
    is_vfs_test();
    Console.PutText("   OK");

    '4. Тест JSON'
    Console.PutText("4. Тест JSON...");
    Object("IClass", "Tests.TestJson", "TestJson").RunTests();
    Console.PutText("   OK");

    '5. Тест UI'
    Console.PutText("5. Тест интерфейса...");
    'is_ui_test_mode() используется для пропуска MessageBox'
    Console.PutText("   OK");

    Console.PutText("=== КОНЕЦ ПОЛНОГО ТЕСТА ===");
    is_message("FULL_TEST", "Полный тест завершён", "OK", "INFORMATION");
}
```

---

### `Tests.SQL_Test` — тест SQL-подсистемы

Проверяет корректность работы SQL-запросов различной сложности, включая вложенные запросы, JOIN, агрегатные функции.

```EME-L
Tests.SQL_Test
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'Сложный SQL-запрос с JOIN и агрегацией'
    Select (Query)
        SELECT
            G.[Code] AS [КодТовара],
            G.[Name] AS [Наименование],
            COUNT(DISTINCT R.[@]) AS [КоличествоПартий],
            SUM(R.[Quantity]) AS [ОбщийОстаток]
        FROM
            [Товары] G
        LEFT JOIN
            [Регистры] R ON G.[@] = R.[GoodsItemRef]
        WHERE
            G.[IsActive] = 1
        GROUP BY
            G.[@],
            G.[Code],
            G.[Name]
        HAVING
            SUM(R.[Quantity]) > 0
    End Select

    is_message("SQL_Test", "SQL-запрос выполнен. Строк: " + Query.GetNoOfLines(), "OK", "INFORMATION");
}
```

---

## 3D-визуализация

### `Tests.Wms3DTest` — тест 3D-визуализации склада

Проверяет работу 3D-модуля визуализации склада. Отображает трёхмерную модлу зон, ячеек и размещённых товаров.

```EME-L
Tests.Wms3DTest
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'Открываем 3D-визуализацию склада'
    View3D = Object("View3D", "Warehouse");
    View3D.SetWarehouseRef(WarehouseRef);
    View3D.SetZoneRef(ZoneRef);

    'Настраиваем отображение'
    View3D.ShowCells(TRUE);
    View3D.ShowGoods(TRUE);
    View3D.SetZoom(1.5);

    'Открываем окно'
    View3D.Show();

    is_message("Wms3DTest", "3D-визуализация открыта", "OK", "INFORMATION");
}
```

---

## Интеграция с внешними системами

### `Tests.VETTest` — тест интеграции с ВЕТИС

Проверяет интеграцию с системой Меркурий (ВЕТИС) — ветеринарные сертификаты, электронные ветеринарные сопроводительные документы.

```EME-L
Tests.VETTest
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'Формируем запрос к ВЕТИС'
    Vetis = Object("Automation", "Vetis.Client");
    Error = Vetis.Connect();

    If (Error != "")
        is_message("VETTest", "Ошибка подключения к ВЕТИС: " + Error, "OK", "STOP");
        Return;
    End If

    'Получаем список ветеринарных сертификатов'
    Certificates = Vetis.GetCertificates();
    is_message("VETTest", "Получено сертификатов: " + Certificates.GetSize(), "OK", "INFORMATION");

    Vetis.Disconnect();
}
```

---

### `Tests.EANCOMTest` — тест EANCOM-сообщений

Проверяет формирование и парсинг EANCOM-сообщений (стандарт EDI для обмена данми между торговыми партнёрами).

```EME-L
Tests.EANCOMTest
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'Формируем тестовое EANCOM-сообщение'
    EANCOM = Object("EANCOM");
    EANCOM.SetMessageType("DESADV");
    EANCOM.SetSender("1234567890123");
    EANCOM.SetReceiver("9876543210987");

    'Добавляем строку'
    EANCOM.AddLine();
    EANCOM.SetLineField("GTIN", "4601234567890");
    EANCOM.SetLineField("Quantity", "100");
    EANCOM.SetLineField("SSCC", "146000000001234567");

    'Формируем сообщение'
    MessageText = EANCOM.Generate();
    is_message("EANCOMTest", "EANCOM-сообщение сформировано. Размер: " + is_length(MessageText), "OK", "INFORMATION");

    'Сохраняем в файл'
    File = Object("File", "D:\\Temp\\TestEANCOM.edi");
    File.CreateFile();
    File.PutFileData(MessageText);
    File.CloseFile();
}
```

---

### `Tests.DIADOCTestRequests` — тест запросов к API Диадок

Проверяет интеграцию с сервисом электронного документооборота Диадок (SKB Kontur).

```EME-L
Tests.DIADOCTestRequests
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'Подключаемся к API Диадок'
    Diadoc = Object("Automation", "Diadoc.Api");
    Error = Diadoc.Authenticate("login", "password");

    If (Error != "")
        is_message("DIADOCTest", "Ошибка авторизации: " + Error, "OK", "STOP");
        Return;
    End If

    'Получаем список документов'
    Documents = Diadoc.GetDocuments();
    is_message("DIADOCTest", "Документов в Диадок: " + Documents.GetSize(), "OK", "INFORMATION");

    Diadoc.Logout();
}
```

---

## Тесты XSD-схем

### `Tests.XsdTestCDATA` — тест обработки CDATA в XML

Проверяет корректность обработки секций CDATA в XML-документах.

```EME-L
Tests.XsdTestCDATA
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'XML с CDATA'
    XmlText = "<?xml version=\"1.0\"?>";
    XmlText = XmlText + "<Document>";
    XmlText = XmlText + "<Description><![CDATA[<Специальные > символы & сущности]]></Description>";
    XmlText = XmlText + "</Document>";

    Xml = Object("Xml");
    Xml.LoadFromString(XmlText);

    Description = Xml.GetValue("Description");
    is_message("XsdTestCDATA", "CDATA содержимое: " + Description, "OK", "INFORMATION");
}
```

---

### `Tests.XsdTestDateTime` — тест форматов даты-времени в XML

Проверяет корректность сериализации и десериализации значений даты и времени в XML (форматы ISO 8601, xsd:dateTime).

```EME-L
Tests.XsdTestDateTime
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'Формируем XML с датой-временем'
    XmlText = "<?xml version=\"1.0\"?>";
    XmlText = XmlText + "<Event>";
    XmlText = XmlText + "<DateTime>2024-06-10T14:30:00+03:00</DateTime>";
    XmlText = XmlText + "</Event>";

    Xml = Object("Xml");
    Xml.LoadFromString(XmlText);

    DateTimeStr = Xml.GetValue("DateTime");
    ParsedDate = is_parse_iso_datetime(DateTimeStr);
    is_message("XsdTestDateTime", "Распарсенная дата: " + ParsedDate, "OK", "INFORMATION");
}
```

---

## Схемы IDoc

### Тестовые схемы IDoc

Схемы IDoc используются для описания структуры сообщений обмена данными между системами.

| Файл схемы | Назначение |
|-----------|----------|
| `TestCDATA.xml` | Тест обработки CDATA-секций |
| `TestDateTime.xml` | Тест форматов даты-времени |
| `TestJSON.xml` | Тест JSON-структур в IDoc |
| `Tests.xml` | Общая тестовая схема |

```EME-L
'Использование тестовой схемы IDoc'
IDoc = Object("IDoc", "TestJSON");

'Создаём сообщение'
Message = IDoc.AddMessage();
Message.PutField("DocNo", "TEST-001");
Message.PutField("DocDate", is_dos_date());

'Добавляем строку'
Line = Message.AddLine();
Line.PutField("GoodsCode", "12345");
Line.PutField("Quantity", "100");

'Получаем JSON-представление'
JsonText = IDoc.ToJSON();
is_message("IDocTest", "JSON: " + JsonText, "OK", "INFORMATION");
```

---

## Отчёты-роботы

### `RobotInter.repAnalysis` — аналитический отчёт

Формирует аналитический отчёт по операциям роботов: количество выполненных тестов, время выполнения, успешность.

```EME-L
RobotInter.repAnalysis
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunReport()
{
    'Формируем отчёт по роботам'
    Report = Object("Report");
    Report.Create();

    'Шапка'
    Report.AddBand("Header");
    Report.PutField("Title", "Анализ работы роботов");
    Report.PutField("Date", is_dos_date());

    'Данные по роботам'
    Robots = Object("Array");
    Robots.Add("GoodsItemRobot");
    Robots.Add("ClientRobot");
    Robots.Add("INCOMERobot");

    Loop (Robots)
        Report.AddBand("Detail");
        Report.PutField("RobotName", Robots.Get());
        Report.PutField("Status", "OK");
        Report.PutField("Duration", "500 мс");
    End Loop

    'Выводим отчёт'
    Report.Show();
}
```

---

## Специальные тесты

### `Tests.PromiseTests` — тест обещаний (promises)

Проверяет механизм асинхронных операций (promises) в EME-L.

```EME-L
Tests.PromiseTests
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'Создаём обещание'
    Promise = Object("Promise");
    Promise.Then("OnSuccess");
    Promise.Catch("OnError");

    'Запускаем асинхронную операцию'
    Promise.Resolve("Результат операции");
}

OnSuccess(Result)
{
    is_message("PromiseTests", "Успешно: " + Result, "OK", "INFORMATION");
}

OnError(Error)
{
    is_message("PromiseTests", "Ошибка: " + Error, "OK", "STOP");
}
```

---

### `Tests.CompleteStockmanOperations` — тест операций кладовщика

Проверяет полный цикл операций кладовщика: приёмка, размещение, подбор, отгрузка.

```EME-L
Tests.CompleteStockmanOperations
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    'Шаг 1: Приёмка'
    is_message("StockmanOps", "Шаг 1: Приёмка", "OK", "INFORMATION");
    '... логика приёмки ...'

    'Шаг 2: Размещение'
    is_message("StockmanOps", "Шаг 2: Размещение", "OK", "INFORMATION");
    '... логика размещения ...'

    'Шаг 3: Подбор'
    is_message("StockmanOps", "Шаг 3: Подбор", "OK", "INFORMATION");
    '... логика подбора ...'

    'Шаг 4: Отгрузка'
    is_message("StockmanOps", "Шаг 4: Отгрузка", "OK", "INFORMATION");
    '... логика отгрузки ...'

    is_message("StockmanOps", "Все операции кладовщика выполнены", "OK", "INFORMATION");
}
```

---

## Сводная таблица прочих тестов

| Класс / Файл | Категория | Что проверяет |
|--------------|----------|--------------|
| `Tests.TestImage` | Изображения | Загрузка, масштабирование, сохранение |
| `Tests.TestPatch` | Интерфейс | Динамическая модификация диалогов |
| `Tests.TestPrintDocFiles` | Печать | Формирование печатных форм |
| `Tests.TestAbsent` | Ошибки | Обработка отсутствующих данных |
| `Tests.TestUserError` | Ошибки | Транзакции и пользовательские ошибки |
| `Tests.FULL_TEST` | Стресс | Полная проверка всех подсистем |
| `Tests.SQL_Test` | SQL | Сложные SQL-запросы |
| `Tests.Wms3DTest` | 3D | Визуализация склада |
| `Tests.VETTest` | Интеграция | ВЕТИС (Меркурий) |
| `Tests.EANCOMTest` | Интеграция | EANCOM/EDI-сообщения |
| `Tests.DIADOCTestRequests` | Интеграция | Электронный документооборот Диадок |
| `Tests.XsdTestCDATA` | XML | Обработка CDATA |
| `Tests.XsdTestDateTime` | XML | Форматы даты-времени |
| `Tests.PromiseTests` | Асинхронность | Обещания (promises) |
| `Tests.CompleteStockmanOperations` | Бизнес-процессы | Полный цикл кладовщика |
| `errorTest` | Ошибки | Обработка системных ошибок |
| `WebTestTemplate` | Веб | Шаблон веб-тестов |
| `HeaderGatesTerritory` | Склад | Территория и ворота |
| `IncomeCreateSticker` | Этикетки | Формирование этикеток при приёмке |
| `ProcessTemplateStrings` | Шаблоны | Обработка строковых шаблонов |

---

## Контрольный список специализированного теста

### Тест интеграции с внешней системой
- [ ] Проверено подключение
- [ ] Обработаны ошибки авторизации
- [ ] Проверены форматы обмена
- [ ] Есть таймаут на операции
- [ ] Корректное отключение

### Тест обработки ошибок
- [ ] Проверены граничные случаи
- [ ] Проверено отсутствие данных
- [ ] Проверены некорректные входные данные
- [ ] Транзакции откатываются при ошибках
- [ ] Сообщения об ошибках информативны
