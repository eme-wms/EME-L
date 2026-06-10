# Шаблоны тестов в EME-L

Синонимы: шаблоны тестов, заготовки тестов, TestJson_XXX, шаблон юнит-теста, шаблон ERP-теста, healClass, шаблон самодиагностики, шаблон серверного теста, FreeBrowser

## Шаблон юнит-теста

```eme-l
Tests.TestJson.TestJson_XXX__MethodName
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    ' Создаём тестовый объект '
    json = Object("Json", "{ \"key\": \"value\", \"num\": 42 }");

    ' Проверяем метод '
    CHECK_EQ( json.MethodName()        , expectedValue1 );
    CHECK_EQ( json.MethodName("arg")   , expectedValue2 );

    ' Проверяем крайние случаи '
    emptyJson = Object("Json", "{}");
    CHECK_EQ( emptyJson.MethodName()   , defaultValue );
}
```

## Шаблон ERP-теста

```eme-l
TestMyIntegration()
{
    ExternalDB = Object("ExternalDB");
    If (~Connect(ExternalDB))
        Return;
    End If

    Query = Object("Query");
    Query.Create();

    Error = ExternalDB.Execute("SELECT * FROM my_table", Query);
    If (Error != "")
        is_message("TestMyIntegration", "Error: " + Error, "OK", "STOP");
        Return;
    End If

    If (Query.GetNoOfLines() > 0)
        is_message("TestMyIntegration", "OK: " + Query.GetNoOfLines() + " rows", "OK", "INFORMATION");
    Else
        is_message("TestMyIntegration", "No data found", "OK", "EXCLAMATION");
    End If

    ExternalDB.Disconnect();
}
```

## Шаблон самодиагностики

```eme-l
RobotHealer.healClasses.testMyFeature
************************             Флаги             ************************
ДЛ: Нет
НК: Нет
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************

describe()
{
    return tr("Проверка моей фичи");
}

test()
{
    rating = 10;
    r_Record = Object("dsDB", "MyRecord");
    r_Record.SetSkipMode();
    If (r_Record.GetNoOfLines() == 0)
        rating = 1;
    End If
    Return rating;
}

heal()
{
    strResult = "";
    is_transaction(1, tr("Исправление моей фичи"));
    strResult = strResult + tr("Исправление выполнено");
    is_transaction(-1);
    Return strResult;
}
```

## Шаблон теста с Browser

```eme-l
TestWithBrowser()
{
    Select (Query)
        SELECT [@], [Рег.No] FROM [Заказы]
        WHERE { r_Orders = Const(Object("dsDB", "Orders"));
                r_Orders.SetLine(is_query(, "@"));
                Return r_Orders.GetStatusAsNumber() >= osIntroduced; }
    End Select

    Browser = Object("FreeBrowser", Query, "Заказы");
    Browser.Run(TRUE);

    OrdersRefs = Object("Array");
    For (QueryLine = Browser.GetSelected(0);
         QueryLine != NULL_REF;
         QueryLine = Browser.GetSelected(QueryLine + 1))
        OrdersRefs.Add(Query.GetData("@", QueryLine));
    End For

    If (OrdersRefs.GetSize() > 0)
        is_transaction(1, "Обработка заказов");
        Loop (OrdersRefs)
            r_Orders = Object("dsDB", "Orders");
            r_Orders.SetLine(OrdersRefs.Get());
        End Loop
        is_transaction(-1);
    End If
}
```

## Шаблон серверного теста

```eme-l
Tests.TestUnit
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************
AAA = 0;

************************             Методы            ************************
Run()
{
    Params = Object("Parameters");
    Params.PutParam("Action", "Run");
    Params.PutParam("Class", "TestUnit");
    Params.PutParam("Method", "TestRun");
    Result = is_server_emel(1009, Params);
    Params.SetString(Result);
    ssSA = is_frombase64(Params.GetParam("Result", ""));
}

TestRun()
{
    is_catastrophic_log("Server test started");
    Return is_server();
}
```

## Контрольный список при создании теста

### Юнит-тест
- [ ] Класс в правильном пространстве имён (`Tests.TestJson.*`)
- [ ] Метод `RunTest()` переопределён
- [ ] Используется `CHECK_EQ` для проверок
- [ ] Проверены крайние случаи (пустые значения, границы)
- [ ] Тест добавлен в раннер (если нужно)

### ERP-тест
- [ ] Проверка подключения перед операциями
- [ ] Корректное отключение (`Disconnect()`)
- [ ] Обработка ошибок через `is_message`
- [ ] Транзакции оформлены правильно
- [ ] Нет захардкоженных паролей в коммите

### Самодиагностика
- [ ] Реализованы три метода: `describe()`, `test()`, `heal()`
- [ ] `test()` возвращает оценку 1–10
- [ ] `heal()` использует транзакцию
- [ ] Есть описание проблемы в `tr()`
