# Шаблоны тестов в EME-L

Синонимы: шаблоны тестов, заготовки тестов, TestJson_XXX, шаблон юнит-теста, шаблон ERP-теста, healClass, шаблон самодиагностики, шаблон серверного теста, FreeBrowser, шаблон SQL-теста, шаблон бота

## Шаблон юнит-теста (для объекта Json)

Создайте новый класс в пространстве имён `Tests.TestJson`:

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
    'Создаём тестовый объект'
    json = Object("Json", "{ \"key\": \"value\", \"num\": 42 }");

    'Проверяем метод'
    CHECK_EQ( json.MethodName()        , expectedValue1 );
    CHECK_EQ( json.MethodName("arg")   , expectedValue2 );

    'Проверяем крайние случаи'
    emptyJson = Object("Json", "{}");
    CHECK_EQ( emptyJson.MethodName()   , defaultValue );
}
```

**Правила именования:**
- Формат: `TestJson_XXX__MethodName`
- `XXX` — трёхзначный номер (001, 002, ...)
- `MethodName` — имя тестируемого метода
- Двойное подчёркивание `__` разделяет номер и имя

## Шаблон ERP-теста

Создайте класс в пространстве имён `ERPSolution.ERPTests` или отдельный класс:

```eme-l
ERPSolution.ERPTests
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
TestMyIntegration()
{
    '1. Подключаемся к внешней БД'
    ExternalDB = Object("ExternalDB");
    If (~Connect(ExternalDB))
        Return;
    End If

    '2. Подготавливаем данные'
    Query = Object("Query");
    Query.Create();

    '3. Выполняем тестируемую операцию'
    Error = ExternalDB.Execute("SELECT * FROM my_table", Query);
    If (Error != "")
        is_message("TestMyIntegration", "Error: " + Error, "OK", "STOP");
        Return;
    End If

    '4. Проверяем результаты'
    If (Query.GetNoOfLines() > 0)
        is_message("TestMyIntegration", "OK: " + Query.GetNoOfLines() + " rows", "OK", "INFORMATION");
    Else
        is_message("TestMyIntegration", "No data found", "OK", "EXCLAMATION");
    End If

    '5. Отключаемся'
    ExternalDB.Disconnect();
}
```

## Шаблон самодиагностики (healClass)

Создайте класс в пространстве имён `RobotHealer.healClasses`:

```eme-l
RobotHealer.healClasses.testMyFeature
************************             Флаги             ************************
ДЛ: Нет
НК: Нет
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************

/* Возвращает название тестируемой функции */
describe()
{
    return tr("Проверка моей фичи");
}

/* Выполняет тест и возвращает оценку 1–10 */
test()
{
    rating = 10;

    'Проверяем условие'
    r_Record = Object("dsDB", "MyRecord");
    r_Record.SetSkipMode();
    If (r_Record.GetNoOfLines() == 0)
        'Проблема: нет данных'
        rating = 1;
    End If

    Return rating;
}

/* Выполняет исправление проблем */
heal()
{
    strResult = "";

    is_transaction(1, tr("Исправление моей фичи"));

    'Выполняем исправляющий код'
    strResult = strResult + tr("Исправление выполнено");

    is_transaction(-1);

    Return strResult;
}
```

## Шаблон SQL-теста (запрос)

Создайте файл в директории `Запросы/`:

```eme-l
Tests.MySqlTest
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    Select (Query)
        SELECT
            [Поле],
            COUNT(*) AS [Количество]
        FROM
            [Таблица]
        GROUP BY
            [Поле]
        HAVING
            {is_query(,"Количество") > 1}
    End Select

    If (Query.GetNoOfLines() > 0)
        is_message("MySqlTest", "Найдено дубликатов: " + Query.GetNoOfLines(), "OK", "EXCLAMATION");
    Else
        is_message("MySqlTest", "Дубликатов не найдено", "OK", "INFORMATION");
    End If
}
```

## Шаблон теста с использованием `dsDB`

```eme-l
Tests.TestMyRecord
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************
r_Record = Object("dsDB", "MyRecord");

************************             Методы            ************************
TestFindLine()
{
    'Настраиваем фильтр'
    r_Record.SetSkipMode();
    r_Record.GetCodeFld().MustBeEQ("TEST001");
    r_Record.SetFirstLine();

    'Проверяем результат'
    If (r_Record.IsValidLine())
        is_message("TestFindLine", "Найдена строка: " + r_Record.GetCode(), "OK", "INFORMATION");
    Else
        is_message("TestFindLine", "Строка не найдена", "OK", "STOP");
    End If
}

TestUpdate()
{
    r_Record.SetLineByCode("TEST001");
    If (r_Record.IsValidLine())
        is_transaction(1);
        r_Record.PutName("Новое название");
        is_transaction(-1);
        is_message("TestUpdate", "Обновлено", "OK", "INFORMATION");
    End If
}
```

## Шаблон теста бота (для Монитора Заданий)

```eme-l
AutoCreateTasksBot
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
Test()
{
    'Тест инициализации стартовых заданий'
    'Запускается вручную (F5) при работе в моно-режиме'
    CreateStartingTasks();
    is_message("AutoCreateTasksBot", "Стартовые задания созданы", "OK", "INFORMATION");
}

CreateStartingTasks()
{
    'Создание начальных заданий для планировщика'
    r_Schedule = Object("dsDB", "Schedule");
    '... логика создания заданий ...'
}
```

**Особенности теста бота:**
- В моно-режиме бот не запускается автоматически — `Test()` вызывается вручную (F5)
- Используется для первоначальной настройки системы
- Результат проверяется визуально или через `is_message`

---

## Шаблон теста с выбором через Browser

```eme-l
TestWithBrowser()
{
    Select (Query)
        SELECT
            [@],
            [Рег.No],
            [Дата],
            [Время]
        FROM
            [Заказы]
        WHERE
            {
                r_Orders = Const(Object("dsDB", "Orders"));
                r_Orders.SetLine(is_query(, "@"));
                Return r_Orders.GetStatusAsNumber() >= osIntroduced;
            }
    End Select

    'FreeBrowser — выбор нескольких строк'
    Browser = Object("FreeBrowser", Query, "Заказы");
    Browser.Run(TRUE);

    'Собираем выбранные строки'
    OrdersRefs = Object("Array");
    For (QueryLine = Browser.GetSelected(0);
         QueryLine != NULL_REF;
         QueryLine = Browser.GetSelected(QueryLine + 1))
        Line = Query.GetData("@", QueryLine);
        OrdersRefs.Add(Line);
    End For

    If (OrdersRefs.GetSize() > 0)
        is_transaction(1, "Обработка выбранных заказов");
        Loop (OrdersRefs)
            r_Orders = Object("dsDB", "Orders");
            r_Orders.SetLine(OrdersRefs.Get());
            '... обработка ...'
        End Loop
        is_transaction(-1);
    End If
}
```

**Ключевые моменты:**
- `FreeBrowser` позволяет выбрать несколько строк (галочками)
- `Browser.Run(TRUE)` — модальный вызов
- `Browser.GetSelected(0)` — первая выбранная строка
- `Browser.GetSelected(prev + 1)` — следующая выбранная строка

---

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

    'Отправляем задание на сервер'
    Result = is_server_emel(1009, Params);
    Params.SetString(Result);

    'Декодируем результат из base64'
    ssSA = is_frombase64(Params.GetParam("Result", ""));
}

TestRun()
{
    is_catastrophic_log("!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!");
    Return is_server();
}
```

**Как это работает:**
1. `Run()` вызывается на клиенте — формирует параметры и отправляет на сервер
2. `is_server_emel(1009, Params)` — отправляет задание на сервер EME-L
3. Сервер выполняет метод `TestRun()`
4. Результат возвращается в `Params`
5. `is_frombase64()` — декодирует ответ

---

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

## Полезные объекты для тестов

| Объект | Создание | Назначение |
|--------|----------|------------|
| `Console` | `Object("Console")` | Вывод текста в консоль |
| `Json` | `Object("Json", строка)` | Работа с JSON |
| `PropertyTree` | `Object("PropertyTree")` | Древовидная структура данных |
| `Query` | `Object("Query")` | SQL-запросы |
| `dsDB` | `Object("dsDB", "ИмяЗаписи")` | Записи базы данных |
| `ExternalDB` | `Object("ExternalDB")` | Подключение к внешней БД |
| `Array` | `Object("Array")` | Динамический массив |
| `Map` | `Object("Map")` | Ассоциативный массив |
| `System` | `Object("System")` | Системные функции |
| `File` | `Object("File", путь)` | Работа с файлами |
| `Automation` | `Object("Automation", ProgID)` | COM-автоматизация |
| `IClass` | `Object("IClass", имя, имя)` | Динамический вызов класса |
