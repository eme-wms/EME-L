> Синонимы: тесты записей, CEMERec, CEMESkip, DBWatch, транзакции, FindLine, GoodsItem, база данных

## Тесты записей и БД

Тесты записей проверяют корректность работы с объектами базы данных (`dsDB`, `CEMERec`, `CEMESkip`), операций поиска, транзакций и механизма наблюдения за данными (`DBWatch`). Эти тесты критичны для обеспечения целостности данных и корректности бизнес-логики.

---

## Базовый класс записей

### `Tests.TestCEMERec` — тест базового класса CEMERec

Проверяет корректность работы базового класса `CEMERec` — основы всех записей базы данных в EME-L. Проверяет чтение, запись, фильтрацию и навигацию по записям.

```EME-L
Tests.TestCEMERec
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    r_Record = Object("dsDB", "GoodsItem");

    'Проверяем установку режима пропуска'
    r_Record.SetSkipMode();
    is_message("TestCEMERec", "SetSkipMode: " + r_Record.GetNoOfLines() + " строк", "OK", "INFORMATION");

    'Проверяем навигацию по записям'
    r_Record.SetFirstLine();
    If (r_Record.IsValidLine())
        is_message("TestCEMERec", "Первая строка: " + r_Record.GetCode(), "OK", "INFORMATION");
    End If

    'Проверяем получение ссылки'
    Line = r_Record.GetLine();
    is_message("TestCEMERec", "Ссылка первой строки: " + Line, "OK", "INFORMATION");

    'Проверяем установку строки по ссылке'
    r_Record.SetLine(Line);
    If (r_Record.IsValidLine())
        is_message("TestCEMERec", "Установка по ссылке работает", "OK", "INFORMATION");
    End If
}
```

**Что проверяет:**
- Установку режима пропуска (`SetSkipMode`)
- Навигацию (`SetFirstLine`, `SetNextLine`, `IsValidLine`)
- Получение и установку ссылки (`GetLine`, `SetLine`)
- Базовые операции чтения данных

---

### `Tests.TestCEMESkip` — тест фильтрации CEMESkip

Проверяет работу фильтров `MustBeEQAbt` и режима пропуска в записях. `MustBeEQAbt` — специальный фильтр для нечёткого сравнения строк.

```EME-L
Tests.TestCEMESkip
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    r_Record = Object("dsDB", "GoodsItem");
    r_Record.SetSkipMode();

    'Применяем нечёткий фильтр'
    r_Record.GetCodeFld().MustBeEQAbt("TEST");
    r_Record.SetFirstLine();

    Count = 0;
    Loop (r_Record)
        Count = Count + 1;
    End Loop

    is_message("TestCEMESkip", "Найдено по MustBeEQAbt: " + Count, "OK", "INFORMATION");

    'Сбрасываем фильтр и проверяем полный список'
    r_Record.SetSkipMode();
    FullCount = 0;
    Loop (r_Record)
        FullCount = FullCount + 1;
    End Loop

    is_message("TestCEMESkip", "Всего записей: " + FullCount, "OK", "INFORMATION");
}
```

**Что проверяет:**
- Работу фильтра `MustBeEQAbt` (нечёткое сравнение)
- Сброс фильтров через повторный `SetSkipMode`
- Корректность подсчёта отфильтрованных строк

---

## Поиск и навигация

### `Tests.TestFindLine` — хэшированный поиск в dsDB

Проверяет скорость и корректность хэшированного поиска в записях. Хэшированный поиск использует встроенные индексы для быстрого нахождения строк по ключевым полям.

```EME-L
Tests.TestFindLine
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    r_Record = Object("dsDB", "GoodsItem");
    r_Record.SetSkipMode();

    'Получаем код первой строки'
    r_Record.SetFirstLine();
    TestCode = r_Record.GetCode();

    'Проверяем хэшированный поиск по коду'
    r_Record.SetLineByCode(TestCode);
    If (r_Record.IsValidLine() & r_Record.GetCode() == TestCode)
        is_message("TestFindLine", "Хэш-поиск по коду работает: " + TestCode, "OK", "INFORMATION");
    Else
        is_message("TestFindLine", "ОШИБКА: Хэш-поиск не нашёл: " + TestCode, "OK", "STOP");
    End If

    'Проверяем поиск по ссылке'
    Ref = r_Record.GetLine();
    r_Record.SetLine(Ref);
    If (r_Record.IsValidLine() & r_Record.GetLine() == Ref)
        is_message("TestFindLine", "Поиск по ссылке работает: " + Ref, "OK", "INFORMATION");
    End If
}
```

**Что проверяет:**
- Хэшированный поиск по коду (`SetLineByCode`)
- Поиск по ссылке (`SetLine`)
- Корректность возвращаемых данных

---

### `Tests.TestPickByLine` — получение параметров по транзиту или PBL

Проверяет корректность работы метода `GetParamsByTransitOrPBL` — получения параметров товара по транзиту или по строке баланса партий.

```EME-L
Tests.TestPickByLine
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    r_Record = Object("dsDB", "GoodsItem");
    r_Record.SetSkipMode();
    r_Record.SetFirstLine();

    If (r_Record.IsValidLine())
        'Получаем параметры по транзиту'
        Params = r_Record.GetParamsByTransitOrPBL();
        is_message("TestPickByLine", "Параметры получены: " + Params, "OK", "INFORMATION");
    End If
}
```

---

## Работа с товарами

### `Tests.TestGoodsItem` — тест записи товаров

Проверяет корректность работы с записью `GoodsItem` — основной записью товаров в системе. Проверяет точность вычислений, свойства и связи.

```EME-L
Tests.TestGoodsItem
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    r_GoodsItem = Object("dsDB", "GoodsItem");
    r_GoodsItem.SetSkipMode();

    'Проверяем точность веса'
    r_GoodsItem.SetFirstLine();
    If (r_GoodsItem.IsValidLine())
        Weight = r_GoodsItem.GetWeight();
        is_message("TestGoodsItem", "Вес товара: " + Weight, "OK", "INFORMATION");

        'Проверяем точность длины'
        Length = r_GoodsItem.GetLength();
        is_message("TestGoodsItem", "Длина: " + Length, "OK", "INFORMATION");

        'Проверяем точность ширины'
        Width = r_GoodsItem.GetWidth();
        is_message("TestGoodsItem", "Ширина: " + Width, "OK", "INFORMATION");

        'Проверяем точность высоты'
        Height = r_GoodsItem.GetHeight();
        is_message("TestGoodsItem", "Высота: " + Height, "OK", "INFORMATION");
    End If

    'Проверяем связь с единицами измерения'
    MURef = r_GoodsItem.GetDefaultMURef();
    is_message("TestGoodsItem", "Единица измерения по умолчанию: " + MURef, "OK", "INFORMATION");
}
```

**Что проверяет:**
- Точность ВГХ (вес, габариты)
- Связи с единицами измерения
- Базовые свойства товара

---

## Транзакции

### `Tests.TestCommit` — тест транзакций

Проверяет корректность работы транзакций: открытие, фиксация, откат. Критично для обеспечения целостности данных при массовых операциях.

```EME-L
Tests.TestCommit
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************

************************             Методы            ************************
RunTest()
{
    r_Record = Object("dsDB", "TestRecord");

    'Запоминаем текущее состояние'
    r_Record.SetSkipMode();
    CountBefore = r_Record.GetNoOfLines();

    'Открываем транзакцию'
    is_transaction(1, "Тестовая транзакция");

    'Добавляем тестовую строку'
    r_Record.AppendAndSetLine();
    r_Record.PutCode("TEST" + is_time());
    r_Record.PutName("Тестовая запись");

    'Фиксируем транзакцию'
    is_transaction(-1);

    'Проверяем, что строка добавлена'
    CountAfter = r_Record.GetNoOfLines();
    If (CountAfter > CountBefore)
        is_message("TestCommit", "Транзакция зафиксирована. Было: " + CountBefore + ", стало: " + CountAfter, "OK", "INFORMATION");
    Else
        is_message("TestCommit", "ОШИБКА: Транзакция не зафиксирована", "OK", "STOP");
    End If
}
```

**Что проверяет:**
- Открытие транзакции (`is_transaction(1)`)
- Фиксацию транзакции (`is_transaction(-1)`)
- Корректность изменений после фиксации
- Сравнение состояния до и после

---

## Наблюдение за данными

### `Tests.TestDBWatch` — тест DBWatch

Проверяет механизм `DBWatch` — наблюдения за изменениями в базе данных между подготовкой данных и записью в транзакции. Используется для предотвращения конфликтов при параллельном доступе.

```EME-L
Tests.TestDBWatch
************************             Флаги             ************************
ДЛ: Нет
НК: Да
ВЗ: Нет
************************          Конструктор          ************************
m_DBWatch = Object("DBWatch");
m_TrueSign;
m_Line;

************************             Методы            ************************
'Подготовка данных вне транзакции'
OnPrepare()
{
    r_GoodsItem = Object("dsDB", "GoodsItem");
    r_GoodsItem.SetFirstLine();
    m_TrueSign = FALSE;
    If (r_GoodsItem.IsValidLine() & ~r_GoodsItem.IsTrueSign())
        m_TrueSign = TRUE;
        m_Line = r_GoodsItem.GetLine();
    End If
    'Добавляем поле GlobalFlagsThreeFld на текущей строке в наблюдение'
    m_DBWatch.WatchFieldLine(r_GoodsItem.GetGlobalFlagsThreeFld());
}

'Запись внутри транзакции'
OnChange()
{
    If (m_TrueSign)
        r_GoodsItem = Object("dsDB", "GoodsItem");
        r_GoodsItem.SetLine(m_Line);
        r_GoodsItem.SetTrueSign();
    End If
}

'Полный цикл с блокировкой'
Do()
{
    'Начинаем дозор (до чтения данных БД)'
    'is_db_watch поднимает данные на верхний уровень'
    is_db_watch(1);

    'Проводим подготовку данных вне транзакции'
    If (is_workstation_mode() != "MONO")
        OnPrepare();
    End If

    'Открываем транзакцию'
    is_transaction(1, "Устанавливаем ЧЗ");

    'Завершаем дозор — больше информация не собирается'
    is_db_watch(-1);

    'Проверяем изменения: если есть — делаем подготовку заново'
    If (is_workstation_mode() == "MONO" | m_DBWatch.Check() != "")
        OnPrepare();
    End If

    'Производим изменения в БД на основе ранее подготовленных данных'
    OnChange();

    'Закрываем транзакцию'
    is_transaction(-1);
}

'Простой вариант без блокировки'
Do2()
{
    is_transaction(1, "Устанавливаем ЧЗ");
    r_GoodsItem = Object("dsDB", "GoodsItem");
    r_GoodsItem.SetFirstLine();
    If (~r_GoodsItem.IsTrueSign())
        r_GoodsItem.SetTrueSign();
    End If
    is_transaction(-1);
}
```

**Паттерн DBWatch:**
1. `is_db_watch(1)` — начать наблюдение (до чтения данных)
2. `OnPrepare()` — читаем данные, запоминаем состояние
3. `m_DBWatch.WatchFieldLine(...)` — добавляем поля в наблюдение
4. `is_transaction(1)` — открываем транзакцию
5. `is_db_watch(-1)` — завершаем наблюдение
6. `m_DBWatch.Check()` — проверяем, изменились ли данные
7. Если изменились — повторяем `OnPrepare()`
8. `OnChange()` — записываем изменения
9. `is_transaction(-1)` — закрываем транзакцию

---

## Сводная таблица тестов записей и БД

| Класс / Файл | Что проверяет | Объекты |
|--------------|--------------|---------|
| `Tests.TestCEMERec` | Базовый класс `CEMERec` | `dsDB` |
| `Tests.TestCEMESkip` | Фильтрация `MustBeEQAbt` | `dsDB` |
| `Tests.TestFindLine` | Хэшированный поиск | `dsDB` |
| `Tests.TestPickByLine` | `GetParamsByTransitOrPBL` | `dsDB` |
| `Tests.TestGoodsItem` | Точность `GoodsItem` | `dsDB` |
| `Tests.TestCommit` | Транзакции (фиксация) | `dsDB` |
| `Tests.TestDBWatch` | `DBWatch` (дозор БД) | `DBWatch`, `dsDB` |

---

## Контрольный список теста записей

### Тест CEMERec
- [ ] Создание объекта через `Object("dsDB", "ИмяЗаписи")`
- [ ] Установка режима пропуска (`SetSkipMode`)
- [ ] Навигация (`SetFirstLine`, `SetNextLine`)
- [ ] Проверка валидности (`IsValidLine`)
- [ ] Работа со ссылками (`GetLine`, `SetLine`)

### Тест транзакций
- [ ] Открытие транзакции (`is_transaction(1, "Имя")`)
- [ ] Выполнение изменений
- [ ] Фиксация (`is_transaction(-1)`) или откат (`is_transaction(0)`)
- [ ] Проверка результатов

### Тест DBWatch
- [ ] Вызов `is_db_watch(1)` до чтения данных
- [ ] Добавление полей в наблюдение (`WatchFieldLine`)
- [ ] Вызов `is_db_watch(-1)` внутри транзакции
- [ ] Проверка изменений (`Check()`)
- [ ] Переподготовка данных при необходимости
