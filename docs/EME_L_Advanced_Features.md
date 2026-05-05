# Программирование компонентов и малоизвестные возможности EME-L

> Синонимы для поиска: компоненты EME БД, IClass программирование компонентов, Selections класс, броузер EME-L, типизация объектов, компилятор EME-L, ServerJob серверные задания, скрытые возможности EME-L, tr() перевод локализация, ENG.txt файл перевода, ByRef ByVal RefOf ValOf, ссылки EME-L, память EME-L, утечка памяти оптимизация, GUI классы серверные классы, разделение классов EME

## Программирование компонентов в EME БД

В данной главе рассматривается создание группы элементов диалога с привязкой не к классу диалога, а к своему собственному классу. в языке EME-L такой подход называется программированием компонентов. Удобство состоит в том, что группа элементов может быть просто скопирована на другой диалог и сразу начать функционировать.

### Пример компонента Selections

Создадим компонент выбора параметров для отчёта с собственным EME-L классом `Selections`.

**Конструктор класса:**

```EME-L
ctrlCaption = Object("Control", , "Caption");
ctrlEdit    = Object("Control", , "Edit");
btnBrowse   = Object("Control", , "btnBrowse");
btnClear    = Object("Control", , "btnClear");

SelectedLines = Object("BitBuffer");

Record AS "CEMERec";
Field;
RecordNumber;
```

**Методы класса:**

```EME-L
SetRecord(RecordName)
{
    Record = Object("dsDB", RecordName);
    RecordNumber = is_try_find_record(Record.GetRecordName());
}

SetField(FieldName)
{
    Field = FieldName;
}

SetCaption(NewCaption)
{
    ctrlCaption.PutText(NewCaption);
}

IsNeedLine(Line)
{
    If (SelectedLines.IsEmpty())
        Return TRUE;
    End If
    Return SelectedLines.IsHewed(Line);
}

Edit_OnInput()
{
    btnBrowse_OnCommand();
}

btnBrowse_OnCommand()
{
    Browser = Object("Browser", , "Browser");
    Browser.SetRecord(RecordNumber);
    Browser.SetColumnCaption(0, Field);
    Browser.SetCaption(Record.GetRecordName());
    If (Browser.Run(TRUE) == NULL_REF)
        Return;
    End If
    Count = Browser.GetMarkedCount();
    If (Count > 1)
        ctrlEdit.PutText(Count + "/" + (is_No_of_lines(Browser.GetRecord()) - 1));
    Else
        RecLine = Browser.GetSelected(0);
        Record.SetLine(RecLine);
        ctrlEdit.PutText(Record.Get(Field));
    End If

    SelectedLines.LoadBrowserSelection(Browser);
}

Browser_OnSkipEx()
{
    Browser = Object("Browser");
    EnteredSurname = is_upper(ctrlEdit.GetText());
    If (EnteredSurname == "ВСЕ")
        Browser.LoadAllLines();
    Else
        For (Record.SetFirstLine(); Record.IsValidLine(); Record.SetNextLine())
            ExistSurname = is_upper(Record.Get(Field));
            If (is_find(ExistSurname, EnteredSurname) > -1)
                Browser.LoadLine(Record.GetLine());
            End If
        End For
    End If
}

btnClear_OnCommand()
{
    ctrlEdit.PutText("Все");
    SelectedLines.Clear();
}

brwColumn_OnPut(brwRecord AS "CEMERec")
{
    Object("Control").Put(brwRecord.Get(Field));
}
```

### Использование компонента в другом классе

```EME-L
SelectionBlock = Object("IClass", "Selections", "Selections");

Dialog_OnAfterUpdate()
{
    SelectionBlock.SetRecord("Покупатели");
    SelectionBlock.SetField("Фамилия");
    SelectionBlock.SetCaption("Покупатели");
}

IsNeedLine(Line)
{
    return SelectionBlock.IsNeedLine(Line);
}
```

### Передача параметров в SQL-запрос (оператор WHERE)

```EME-SQL
WHERE
{
    NeedData = Const(is_edit_data("BuyersCallReport.Date"));
    DialogClassObject = Const(Object("Dialog").GetClassObject());
    Flag = DialogClassObject.IsNeedLine(is_query(,"@Покупатель"));
    Return (is_query(, "Дата") == NeedData & Flag);
}
```

## Малоизвестные возможности программирования в ЕМЕ БД

### 1. Создание SQL-запроса без обращения к записи

Конструкция `FROM NEW{}` формирует запрос без обращения к какой-либо записи БД:

```EME-SQL
SELECT
    { Return is_date() - 1;}:DATE AS [Дата начала],
    { Return is_time(0,0,0);}:TIME AS [Время начала],
    { Return is_date();}:DATE AS [Дата завершения],
    { Return is_time();}:TIME AS [Время завершения]
FROM NEW{1}
```

### 2. Привязка внешних устройств к диалогу

в языке EME-L имеется возможность использовать в диалогах устройства с последовательным интерфейсом (весы, проводной сканер, считыватель карт, считыватель отпечатков).

```EME-L
Dialog_OnLoad()
{
    Dialog = Object("Dialog");
    Dialog.LoadScannerDriver(0);
}

Dialog_OnCipherInput(CCD)
{
}

Dialog_OnClose()
{
    Dialog = Object("Dialog");
    Dialog.UnloadScannerDriver(0);
}
```

Название метода должно содержать название драйвера из настроек рабочей станции.

### 3. Создание календаря для ввода диапазона дат

```EME-L
Dialog_OnLoad()
{
    Dialog = Object("Dialog");
    ctrlStartDate = Object("Control",,"StartDate");
    ctrlEndDate = Object("Control",,"EndDate");
    Dialog.SetControlDateRange(ctrlStartDate, ctrlEndDate);
}
```

### 4. Создание списка отчётов без привязки к шаблону

```EME-L
Dialog_OnAfterUpdate()
{
    arrReports = Object("Array");
    arrReports.Add("Лист вложения");
    Dialog.ShowReports(arrReports, True);
}

GetReport(r_Order)
{
    menu = Object("Menu", True);
    arrReports = Object("Array");

    submenu = Object("Menu", True);
        submenu.AppendMenu("STRING", arrReports.Add("ТТН"), "ТТН");
        submenu.AppendMenu("STRING", arrReports.Add("Накладная"), "Накладная");
    menu.AppendMenu("POPUP", submenu, "НАКЛАДНЫЕ");

    action = menu.TrackPopupMenuWithMousePos();
    If (action<0 | action>=arrReports.GetSize())
        Return "";
    End If
    Return arrReports.GetAt(action);
}

Dialog_OnReport(Name)
{
    repotrName = GetReport(dsDB);
    If (repotrName != "")
        PrintReport(repotrName, dsDB);
    End If
    Return True;
}
```

### 6. Работа с памятью в EME-L

в языке EME-L нет операторов `new` и `delete`, память выделяется и освобождается автоматически. Правила работы:

1. Никогда не использовать `Object("ClassName")` внутри цикла.
2. Всегда добавлять префикс `m_` к именам переменных-членов класса.
3. Не инициализировать переменные-члены класса типа `Object` вне конструктора.
4. Использовать оператор `Const` для создания объектов-членов класса.

### 7. Передача аргументов по ссылке (ByRef, ByVal, RefOf, ValOf)

в языке EME-L добавлены операторы для передачи аргументов по ссылке:

```EME-L
MyMethod(MyReference ByRef)

Result = MyMethod(RefOf MyValue)
Check = ValOf MyReference > 0
MyReference ByVal = 5
```

### 8. Функция tr() для перевода рабочей станции

Функция `tr()` обёртывает строки для перевода с русского на иностранный язык:

```EME-L
is_format(tr("На складе %s включена инвентаризация"), m_r_Warehouse.GetCode());
```

**Правильное использование:** `tr("строка с %s и %d")` внутри `is_format()`.

**Неправильное использование:**
- `tr("текст" + переменная)` — не будет найден перевод.
- `tr("часть1") + tr("часть2")` — подстроки переводятся независимо.

### 9. Утилиты локализации (класс LocalizationUtilities)

- `TranslateContextFindParams()` — выгружает строки записи «Параметры контекстного поиска».
- `GetTextFromUserMenu()` — тексты из меню.
- `GetTextFromTREMEL()` — тексты из EME-L кода, обёрнутые в `tr()`.
- `TranslateDialogs()` — тексты из шаблонов диалогов.
- `TranslateBrowsers()` — тексты из шаблонов браузеров.
- `TranslateSettings()` — тексты из настроек.

### 10. Разделение классов на серверные и GUI

в языке EME-L программист должен указывать, серверный класс или GUI. GUI-классы можно наследовать от серверных, но не наоборот. Компилятор автоматически проверяет это.

Макросы C++ для указания типа класса:
- `END_EMEL_GUI_CLASS` — GUI-класс
- `END_EMEL_CONTEXT_GUI_CLASS` — GUI-класс в C++
- `REGISTRY_GUI_IS_FUNC_GROUP` — GUI is-функция

Если необходимо реализовать класс для сервера и рабочей станции, используйте наследование: в базовом классе реализуйте методы без GUI, а в наследуемом — методы интерфейса.

## См. также

- [EME_L_Query_Reports.md](./EME_L_Query_Reports.md) — формирование отчётов
- [EME_L_Advanced_API_and_Patterns.md](./EME_L_Advanced_API_and_Patterns.md) — классы String, PropertyTree, боты
- [EME_L_Database.md](./EME_L_Database.md) — структуры данных Query, BitBuffer, Array, Map
