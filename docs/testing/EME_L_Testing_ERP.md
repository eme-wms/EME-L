# ERP-тесты в EME-L

Синонимы: ERP-тесты, интеграционные тесты, ERPSolution, ERPTests, ERPStressTest, ERPEngine, внешняя БД, ODBC, JSON, XML, PropertyTree, IDoc

## Что такое ERP-тесты

**ERP-тесты** — это интеграционные тесты подсистемы обмена данными между WMS и внешней ERP-системой (1С, SAP, учётные системы заказчика). Они собраны в классе `ERPSolution.ERPTests` (`ERPTests.txt`).

В отличие от юнит-тестов, ERP-тесты:
- Требуют подключения к внешней базе данных через ODBC
- Проверяют канал связи, форматы сообщений и движок обмена
- Запускаются вручную (кнопкой) разработчиком
- Не используют классические `assert` — результат проверяется визуально или через логи

## Цели ERP-тестов

1. **Проверить канал связи** — подключение к внешней БД через ODBC (`ExternalDB`), скорость запросов
2. **Проверить движок обмена** — C#-компонент `EME.ERPEngine` (создание сообщений `goods`, `orders`, `clients`, `asn`, `receipt`, `change`, `quality_change`)
3. **Проверить форматы** — корректность выгрузки/загрузки в JSON и XML через `PropertyTree` + `IDoc`
4. **Проверить маппинг полей** — что все поля, используемые в обработчиках `ERP*Component`, зарегистрированы и не противоречат друг другу (`TestOnErrors`)
5. **Проверить импорт плоских файлов** — загрузка номенклатуры из CSV/TXT через `EMEQueryTextFile`

## Структура ERP-тестов

### Основной класс: `ERPSolution.ERPTests`

Файл: `ERPTests.txt`

```eme-l
ERPSolution.ERPTests
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
```

### Подключение к внешней БД

```eme-l
Connect(ExternalDB As "ExternalDB")
{
    'Error = ExternalDB.Connect(
        "ODBC:Driver={PostgreSQL ODBC Driver(ANSI)};Server=localhost;Port=5432;Database=WMS_ERP;Uid=postgres;Pwd=123;");'
    'Error = ExternalDB.Connect(
        "ODBC:Driver={SQL Server};Server=AMP-NOTEBOOK\\SQL2005;Database=WMS_KINEL;Uid=sa;Pwd=123;");'
    'Error = ExternalDB.Connect(
        "ODBC:DSN=WMS_YONA;Uid=system;Pwd=123;");'
    Error = ExternalDB.Connect(
        "ODBC:driver={EME DB};server=amp-syktyvkar;port=80;uid=amp;pwd=bnfl;");
    If (Error != "")
        is_message(
            "ExternalDBTest",
            tr("Не удалось подключить клиента к внешней базе данных:\n\n") + Error, "OK", "STOP");
        Return FALSE;
    End If

    Return TRUE;
}
```

**Поддерживаемые СУБД:** PostgreSQL, SQL Server, EME DB.

### Создание таблиц во внешней БД

```eme-l
Test()
{
    System = Object("System");
    ExternalDB = Object("ExternalDB");

    If (~Connect(ExternalDB))
        Return;
    End If

    LinesPrimaryKeys = Object("Array");
    LinesPrimaryKeys.Add("id");
    LinesPrimaryKeys.Add("header_id");

    i_ERPGoodsComponent = Object("IClass", "ERPGoodsComponentObject", "ERPGoodsComponent");
    HeaderQuery = Object("Query");
    MuLinesQuery = Object("Query");
    BarCodesQuery = Object("Query");
    HeaderQuery.Create();
    MuLinesQuery.Create();
    BarCodesQuery.Create();
    i_ERPGoodsComponent.ERPMarkup(HeaderQuery, MuLinesQuery, BarCodesQuery);

    Execute(ExternalDB, "DROP TABLE \"erp_goods_header\"", );
    Execute(ExternalDB, "DROP TABLE \"erp_goods_mu_lines\"", );
    Execute(ExternalDB, "DROP TABLE \"erp_goods_bar_codes\"", );
    CreateTable(ExternalDB, "erp_goods_header", HeaderQuery, "id");
    CreateTable(ExternalDB, "erp_goods_mu_lines", MuLinesQuery, LinesPrimaryKeys);
    CreateTable(ExternalDB, "erp_goods_bar_codes", BarCodesQuery, LinesPrimaryKeys);

    ' ... экспорт данных ... '

    UpdateTable(ExternalDB, "erp_goods_header", HeaderQuery);
    UpdateTable(ExternalDB, "erp_goods_mu_lines", MuLinesQuery);
    UpdateTable(ExternalDB, "erp_goods_bar_codes", BarCodesQuery);

    ExternalDB.Disconnect();

    is_message("ERPInterface::Test", "Ok", "OK", "INFORMATION");
}
```

### Тест движка `EME.ERPEngine` (C#)

```eme-l
TestERPEngineSharp()
{
    erp = Object("Automation", "EME.ERPEngine");
    Timeout = erp.GetQueryTimeout();
    erp.SetQueryTimeout(60);

    Error = erp.Connect();
    If (Error != "")
        is_message("TestERPEngineSharp", tr("Ошибка при запуске движка:\n") + Error, "OK", "STOP");
        Return;
    End If

    erp.BeginExport("erp", "wms", "goods");

    erp.AppendHeaderLine();
    erp.PutHeaderData("id", "12222258");
    erp.PutHeaderData("article", "12222258");
    erp.PutHeaderData("name", "РОССИЙСКИЙ Шоколад Горький70% 27(5х90г)");
    ' ... ещё поля ... '

    erp.SelectChild("packs");

    erp.AppendChildLine();
    erp.PutChildData("id", "Штука");
    erp.PutChildData("mu_code", "EA");
    erp.PutChildData("factor", 1);
    ' ... ещё строки ... '

    Error = erp.CommitExport();
    If (Error != "")
        erp.Disconnect();
        is_message("TestERPEngineSharp", tr("Ошибка при экспорте сообщения:\n") + Error, "OK", "STOP");
        Return;
    End If

    erp.Disconnect();

    is_message("TestERPEngineSharp", tr("Тест завершен!"), "OK", "INFORMATION");
}
```

**Паттерн работы с `ERPEngine`:**
1. `Connect()` — подключение к движку
2. `BeginExport(источник, назначение, тип_сообщения)` — начало экспорта
3. `AppendHeaderLine()` + `PutHeaderData(...)` — заполнение заголовка
4. `SelectChild("имя_дочерней_таблицы")` + `AppendChildLine()` + `PutChildData(...)` — заполнение строк
5. `CommitExport()` — фиксация сообщения
6. `Disconnect()` — отключение

### Формирование сообщений разных типов

| Метод | Тип сообщения | Назначение |
|-------|---------------|------------|
| `CreateQualityChangeMessage()` | `quality_change` | Изменение качества товара |
| `CreateChangeMessage()` | `change` | Смена статуса/аналитики |
| `CreateClientsMessage()` | `clients` | Клиенты и адреса доставки |
| `CreateOrdersMessage()` | `orders` | Заказы |

### Импорт сообщений

```eme-l
TestImportReceipts()
{
    ERPEngine = Object("Automation", "EME.ERPEngine");

    Error = ERPEngine.Connect();
    If (Error != "")
        is_message("", Error);
        Return;
    End If

    ERPEngine.BeginImport("wms", "erp", "receipt");
    Try (Error)
        is_transaction(1, tr("Тест импорта уведомлений о приемке"));
        Try (Error)
            While (ERPEngine.NextHeaderLine())
                Id = ERPEngine.GetHeaderData("id");

                'Проверки на первом проходе'
                Switch (is_random(4))
                Case 0:
                    ERPEngine.ErrorHeader("TSTBAD");
                Case 1:
                    ERPEngine.WarningHeader("DCMNTBAD");
                Case 2:
                    ERPEngine.Success();
                Default:
                End Switch

                ERPEngine.SelectChild("lines");
                While (ERPEngine.NextChildLine())
                    GoodsCode = ERPEngine.GetChildData("goods_code");
                End While
            End While

            is_transaction(-1);
        Catch
        If (~is_empty(Error))
            is_transaction(0);
            is_error(1, Error);
        End If
        ERPEngine.CommitImport();
    Catch
    If (~is_empty(Error))
        ERPEngine.RollbackImport();
        is_message("", Error);
    End If

    ERPEngine.Disconnect();
}
```

### Работа с JSON и XML

```eme-l
' Выгрузка в JSON через PropertyTree + IDoc
TestJsonUpload()
{
    Json = Object("PropertyTree");
    IDoc = Object("IDoc", "TestJSON", Json);

    Message = IDoc.AddERPMessage();
    Header = Message.AddHeaderEdit();

    Prices = Header.AddPrices();
    Prices.Put(3.14);

    Header.PutDocDate(is_dos_date());
    Header.PutDocTime(is_dos_time());

    Lines = Header.AddLines();
    Lines.PutId(1);
    Lines.PutHeaderId(2);

    File = Object("File", "E:\\Json\\Test.json");
    File.CreateFile();
    File.PutEndFileData(Json.GetJSON());
}

' Загрузка из JSON
TestJsonLoad()
{
    File = Object("File", "E:\\Json\\Test.json");
    Json = Object("PropertyTree");
    Json.PutJSON(File.GetData().AsString());
    IDoc = Object("IDoc", "TestJSON", Json);
    IsERPMessage = IDoc.TuneERPMessage();
    Message = IDoc.GetERPMessage();
    Header = Message.GetHeaderEdit();

    IsDocDate = Header.TuneDocDate();
    DocDate = Header.GetDocDate();

    Lines = Header.GetLines();
    Loop (Lines)
        Id = Lines.GetId();
        HeaderId = Lines.GetHeaderId();
    End Loop
}
```

## ERP-стресс-тесты

Класс `ERPSolution.ERPStressTest` (`ERPStressTest.txt`) — расширение ERP-тестов для массовой выгрузки и симуляции.

### Массовая выгрузка сообщений

```eme-l
UploadErpWmsMessages()
{
    m_ERPEngine = Object("IClass", "ERPEngineObject_" + Object().GetObjectName(), "ERPEngine");

    Menu = Object("IClass", "ActiveMenuObject_" + Object().GetObjectName(), "ActiveMenu");
    Menu.AddMenuItem("Выгрузка всех продуктов#Из карточки", "UploadGoodsItems", "All");
    Menu.AddMenuItem("Выгрузка всех клиентов", "UploadClients");
    Menu.AddMenuItem("Выгрузка всех ASN", "UploadASNs");
    Menu.AddMenuItem("Выгрузка всех заказов", "UploadOrders");
    Menu.AddMenuItem("Выгрузка всех перевозок", "UploadShipments");
    ' ... ещё 11 пунктов ... '
    Menu.Run();
}
```

### Верификаторы сообщений

```eme-l
VerifyMessages(ClassName)
{
    Object("IClass", ClassName, ClassName).RunVerify();
}
```

| Верификатор | Тип сообщения |
|-------------|---------------|
| `ERPAsnVerifier` | ASN |
| `ERPChangeVerifier` | CHANGE |
| `ERPChangePoVerifier` | CHANGE_PO |
| `ERPClassifierVerifier` | CLASSIFIER |
| `ERPGoodsVerifier` | GOODS |
| `ERPOffwritePoVerifier` | OFFWRITE_PO |
| `ERPOrdersVerifier` | ORDERS |
| `ERPPalletsVerifier` | PALLETS |
| `ERPPoVerifier` | PO |
| `ERPShipmentsVerifier` | SHIPMENTS |
| `ERPStaffVerifier` | STAFF |
| `ERPTransferPoVerifier` | TRANSFER_PO |

### Симулятор EME.WMS

```eme-l
SimulateEmeWms()
{
    Menu = Object("IClass", "ActiveMenuObject_" + Object().GetObjectName(), "ActiveMenu");
    Menu.AddMenuItem("Тестовый склад", "CreateTestWarehouse");
    Menu.AddMenuItem("Приход#Случайный приход", "CreateReceipt", "");
    Menu.AddMenuItem("Приход#Сторно-приход", "CreateReceipt", "Storno");
    Menu.AddMenuItem("Приход по ASN#Полный приход", "CreateReceipt", "ASN");
    Menu.AddMenuItem("Приход по ASN#Приход с проблемами", "CreateReceipt",
        "ASN,Shortage,Overhead,Defect,Mixup");
    Menu.AddMenuItem("Отгрузка#Заказ по ASN", "CreateOrderPerASN", "");
    Menu.AddMenuItem("Отгрузка#Подобрать заказ", "CreateDespatch", "Assembly");
    Menu.AddMenuItem("Отгрузка#Отгрузить заказ", "CreateDespatch", "Finish");
    Menu.AddMenuItem("Перемещение#Приход под заказ", "CreateReceiptPerPO", "Transfer");
    Menu.AddMenuItem("Списание#Приход под заказ", "CreateReceiptPerPO", "Offwrite");
    Menu.AddMenuItem("Изм. характеристик#Приход под заказ", "CreateReceiptPerPO", "Change");
    Menu.Run();
}
```

### Пачковый экспорт с BitBuffer

```eme-l
ExportAll(Source, Target, Name, RecordObject As "CEMERec", BitBufferFilter As "BitBuffer")
{
    Console = Object("Console");
    Console.Show(1);
    Console.PutText(tr("Соединяем с внешней базой данных..."));

    m_ERPEngine.SetConsole(Console);
    Error = m_ERPEngine.Launch(m_ERPEngine.GetConnStr());

    is_do_wait_cursor(1);

    m_ERPEngine.StartPercentage("Читаем из записи \"" +
        RecordObject.GetRecordName() + "\"", RecordObject.GetNoOfLines());
    RecordObject.SetFirstLine();
    Refs = Object("Array");
    While (GetNextExportBatch(RecordObject, BitBufferFilter, Refs))

        Error = m_ERPEngine.BeginExport(Source, Target, Name);
        If (Error == "")
            OldRef = RecordObject.GetLine();
            Loop (Refs)
                RecordObject.SetLine(Refs.Get());
                m_ERPEngine.NextPercentage(RecordObject.GetLine());
                Error = m_ERPEngine.DoExport(RecordObject);
                is_peek_all_messages();
                If (Error != "")
                    Break
                End If
            End Loop
            RecordObject.SetLine(OldRef);
        End If

        If (Error == "")
            Error = m_ERPEngine.EndExport();
        End If
        If (Error != "")
            Break
        End If
    End While

    is_do_wait_cursor(-1);
}

GetNextExportBatch(r_RecordObject As "CEMERec", BitBufferFilter As "BitBuffer", Refs As "Array")
{
    Refs.RemoveAll();
    Count = 0;
    While (r_RecordObject.IsValidLine())
        'Проверим фильтр'
        If (~is_empty(BitBufferFilter))
            If (~BitBufferFilter.IsHewed(r_RecordObject.GetLine()))
                r_RecordObject.SetNextLine();
                Continue;
            End If
        End If

        Refs.Add(r_RecordObject.GetLine());
        r_RecordObject.SetNextLine();
        Count = Count + 1;
        If (Count == BATCH_SIZE)
            Break
        End If
    End While
    Return Refs.GetSize() != 0;
}
```

**Ключевые особенности пачкового экспорта:**
- Используется `BitBuffer` для фильтрации записей (например, только товары из ASN)
- `BATCH_SIZE` — размер пачки (константа класса)
- `GetNextExportBatch()` — формирует пачку ссылок с учётом фильтра
- `is_peek_all_messages()` — обработка окон сообщений внутри цикла
- `Console` — визуальный прогресс-бар для пользователя

### Создание тестового склада

```eme-l
CreateTestWarehouse()
{
    r_Warehouse = Object("dsDB", "Warehouse");
    r_Warehouse.SetSkipMode();
    r_Warehouse.GetBranchRefFld().MustBeRefEQ(is_system(5));
    r_Warehouse.GetCodeFld().MustBeEQ("TST");
    r_Warehouse.SetFirstLine();

    If (r_Warehouse.IsValidLine())
        is_message("CreateTestWarehouse", "Тестовый склад уже есть в БД", "OK", "EXCLAMATION");
        Return;
    End If

    r_FindParams = Object("dsDB", "FindParams");
    r_FindParams.SetSkipMode();
    r_FindParams.GetCodeFld().MustBeEQ("WH1");

    is_transaction(1, "Создать тестовый склад");
    r_Warehouse.AppendAndSetLine();
    r_Warehouse.PutBranchRef(is_system(5));
    r_Warehouse.PutWarehouseType(1);
    r_Warehouse.PutCode("TST");
    r_Warehouse.PutName("Тестовый склад");
    r_Warehouse.PutKeeperRef(Object("dsDB", "Clients").GetDefaultVendorRefForDocuments());
    r_Warehouse.PutSSCCPrefix("14600000000");
    r_Warehouse.SetDefaultInDoc();
    r_FindParams.SetFirstLine();
    r_Warehouse.PutFindParamsRef(r_FindParams.GetLine());
    is_transaction(-1);

    is_message("CreateTestWarehouse", "Создан тестовый склад", "OK", "EXCLAMATION");
}
```

## Связанные классы

| Класс | Назначение |
|-------|------------|
| `XsdSchemas.XsdTestCDATA` | Тестовый XSD-обработчик формата `CDATA` |
| `XsdSchemas.XsdTestDateTime` | Тестовый формат даты-времени |
| `PortalsAPI.DIADOC.DIADOCTestRequests` | Запросы к API Диадок |
| `EANCOM.EANCOMTest` | Тест экспортного EANCOM-сообщения |
| `PortalsAPI.VET.VETDialog.VETTest` | Тест кнопки ВЕТИС |
| `Utilities.ExportToTHQ.TestExportToTHQ` | Выгрузка в ЦУП |
| `AAATrash.OneC.TestOrderFrom1C` | Заказ из 1С через COM |
