# Справочник обработчиков сообщений ядра EME.WMS — события, обработчики, хуки

> Синонимы для поиска: EME.WMS события, обработчики сообщений ядра, Dialog_OnInit, Dialog_OnClose, Dialog_OnLoad, Dialog_OnCheck, Dialog_OnClick, OnInit, OnClick, OnLoad, OnUpdate, OnSave, OnDelete, OnBrowse, OnEdit, OnEndEdit, OnInput, OnCheck, OnTimer, OnContextMenu, OnCommand, Table_OnClick, Table_OnSetNextCell, Browser_OnInit, Browser_OnSelect, Browser_OnCommand, Browser_OnSkipEx, Integrator_OnSelect, Integrator_OnEdit, Report_OnBeginReport, Report_OnEndReport, Band_OnSkip, Band_OnNewItem, обработчики диалогов EME, обработчики таблиц EME, обработчики браузеров EME, обработчики интегратора EME, обработчики отчетов EME, обработчики бандов EME, события ядра EME БД, OnNewItem, OnAfterUpdate, OnBeforeUpdate, OnBeforeSave, OnAfterSave, OnChangeTableLine, OnDeleteTableLine, OnReport, OnHyperTextRef, OnGetProperty, OnFillPrintForms, OnRunPrintForm, OnMarkTableLine, OnSkip, OnRClick, OnAfterSelect, OnMarkTreeItem, OnTune, OnStartAddDialog, OnPutColumn, OnGetChild, OnGetSibling, OnGetParent, OnGetItemText, OnGetItemIcon, OnAttach, OnGetMenuCount, OnGetMenuItem, OnGetContextMenu, OnSelection, OnMark, OnFormatItemText, OnInsertItem, OnDropItem, OnSetFocus, OnKillFocus, OnExpand, OnFillGraphWindow, OnPut

---

## Обработчики сообщений ядра для диалогов в ЕМЕ БД

в системе EME.WMS для диалогов имеется 24 обработчика сообщений ядра.

### Dialog_OnInit

Метод вызывается только один раз для каждого диалога в момент его создания (в отличие от функций обновления диалога). Во время инициализации можно задать значения переменных, которые будут использоваться в дальнейшем, или задать параметры органов диалога.

в языке EME-L во время инициализации не имеет смысла заполнять значениями органы диалога — после инициализации будет вызвана функция обновления, которая очистит все органы. Значения органов следует заполнять при вызове `OnAfterUpdate`.

**Входные параметры:** отсутствуют.

**Выходные параметры:** по умолчанию ничего не возвращает. Если метод вернёт `True` — открытие диалога запрещено и он закроется.

```EME-L
Dialog_OnInit()
{
    Clients = Object("dsDB", "Clients");
    Clients.SetFirstLine();
    Object("Dialog").SetLine(Clients.GetLine());
    If (~is_center() & ~is_system_filial())
        Object("Dialog").AddFlags(1, True);
    End If
}
```

### Dialog_OnClose

в системе EME.WMS метод вызывается каждый раз при попытке закрытия диалога. Вернув `True`, можно отменить закрытие. Обычно используется для очистки или удаления объектов, созданных ранее для текущего диалога.

**Входные параметры:** отсутствуют.

**Выходные параметры:** если метод вернёт `True` — диалог не будет закрыт.

```EME-L
Dialog_OnClose()
{
    If (~is_NULL(Object("Dialog").GetParent()))
        Object("Dialog").GetParent().Update();
    End If
}
```

### Dialog_OnLoad

в языке EME-L метод вызывается только один раз в момент открытия диалога (после `OnInit()`). Подходит для заполнения выпадающих списков (ComboBox). Во время загрузки можно работать с органами диалога, но при обновлении диалога этот метод не вызывается.

**Входные параметры:** отсутствуют.

**Выходные параметры:** отсутствуют.

```EME-L
Dialog_OnLoad()
{
    BBD = Object("Control", , "BBDPresentationMethod");
    BBD.Add("День, месяц, год производства", 0);
    BBD.Add("Неделя и год производства", 1);
    BBD.Add("День, месяц, год срока годности", 2);
    BBD.Add("Неделя и год срока годности", 3);
}
```

### Dialog_OnNewItem

в EME БД метод вызывается до создания новой строки диалога. Заполнять органы диалога значениями не имеет смысла — после него будет производиться обновление, и значения будут потеряны.

**Входные параметры:** отсутствуют.

**Выходные параметры:** если метод вернёт `True` — создание новой строки отменяется.

```EME-L
Dialog_OnNewItem()
{
    Object("Control", , "SourceButton").Disable(True);
    Object("Control", , "TestDialog").Disable(True);
    Object("Control", , "TestButton").Disable(True);
}
```

### Dialog_OnAfterNewItem

в системе EME.WMS метод вызывается после создания новой строки диалога. Можно заполнить все необходимые органы значениями по умолчанию.

**Входные параметры:** отсутствуют.

**Выходные параметры:** отсутствуют.

```EME-L
Dialog_OnAfterNewItem()
{
    Object("Control", ,"CreationDate").Put(is_dos_date());
    Object("Control", ,"CreationTime").Put(is_dos_time());
    Object("Control", , "RegNo").Put(Object("dsDIALOG").regno_Get(0));
}
```

### Dialog_OnBeforeUpdate

в языке EME-L метод вызывается каждый раз перед обновлением диалога. Можно задавать свойства органов или определять переменные. Не стоит класть новые значения в органы — после данного метода ядро заполнит органы и все значения будут потеряны. Ручное заполнение — в `OnAfterUpdate()`.

**Входные параметры:** отсутствуют.

**Выходные параметры:** отсутствуют.

```EME-L
Dialog_OnBeforeUpdate()
{
    If (Object("Dialog").GetLine() != NEW_LINE_MARK)
        UpdateIntegrator = True;
    End If
}
```

### Dialog_OnAfterUpdate

в EME БД метод вызывается каждый раз при обновлении данных диалога, после того как органы уже заполнены из ядра. Если необходимо вручную заполнить органы — это нужно делать именно здесь.

**Входные параметры:** отсутствуют.

**Выходные параметры:** отсутствуют.

```EME-L
Dialog_OnAfterUpdate()
{
    Object("Control", , "StartTime").Put(is_time(0, 0));
    Object("Control", , "EndTime").Put(is_time(23, 59));
}
```

### Dialog_OnBeforeSave

в системе EME.WMS метод вызывается перед сохранением данных диалога в БД. Транзакция уже открыта, новая строка добавлена, но данные еще не записаны. Можно фиксировать изменения, сравнивая значения органов с полями БД. Проверку на новую строку — методом `CIDialog::IsNewLine()`.

Важно: нельзя записывать данные в органы (они прошли `OnCheck`) и в поля БД (сохранение может перезаписать).

**Входные параметры:** отсутствуют.

**Выходные параметры:** отсутствуют.

```EME-L
Dialog_OnBeforeSave()
{
    If (bModified)
        SaveChanges();
    End If
}
```

### Dialog_OnAfterSave

в языке EME-L метод вызывается после сохранения данных диалога (во время записи в БД). Транзакция еще не закрыта — можно записать данные, не сохраняемые автоматически. Генерацию номера нового документа выполнять здесь. Проверку на новую строку — методом `CIDialog::IsNewLine()`.

**Входные параметры:** отсутствуют.

**Выходные параметры:** отсутствуют.

```EME-L
Dialog_OnAfterSave()
{
    Line = Object("Dialog").GetLine();
    dsDB = Object("dsDB");
    dsDB.SetLine(Line);
    dsDB.PutOperatorRef(is_system(0));
    dsDB.PutDate(is_dos_date());
    dsDB.PutTime(is_dos_time());
}
```

### Dialog_OnCheck(Transaction)

в EME БД метод предназначен для проверки данных диалога перед сохранением. Вызывается два раза (если все проверки пройдены) — до открытия транзакции и при открытой транзакции. Параметр `Transaction` указывает состояние.

Важно: метод предназначен только для проверки. Запись в органы или БД запрещена.

**Входные параметры:**
- `Transaction` — флаг: `True` = транзакция открыта, `False` = не открыта.

**Выходные параметры:** если метод вернёт `True` — сохранение отменяется.

```EME-L
Dialog_OnCheck(Transaction)
{
    If (Transaction)
        Return False;
    End If
    ShortName = Object("Control", , "ShortName");
    If (ShortName.IsEmpty())
        is_message("Внимание!", "Не заполнено поле 'Запрос'", 1);
        ShortName.SetFocus();
        Return True;
    End If
}
```

Пример сбора информации об удаляемых строках в `OnCheck` и проверки возможности удаления:

```EME-L
r_MoveOrder = Object("dsDB", "MoveOrder");
If (~IsTransaction)
    arrDeleteMoveOrders.RemoveAll();
End If

For (line = tbl.GetFirstDeletedLine(); line != NULL_REF; line = tbl.GetNextDeletedLine())
    dbLines.SetLine(line);
    If(dbLines.IsValidLine())
        If (dbLines.IsGoodsLoaded() | dbLines.IsGoodsInBuffer())
            is_transaction(0);
            is_message(tr("Сохранение запрещено!"), tr("Некоторые строки в буфере или отгружены!"), 1, 3);
            dlg.Update();
            Return 1;
        End If
        If (~IsTransaction)
            r_MoveOrder.SetChain("@Строка документа", dbLines.GetLine());
            Loop (r_MoveOrder)
                arrDeleteMoveOrders.Add(r_MoveOrder.GetLine());
            End Loop
        End If
    End If
End For
```

### Dialog_OnCheckDelete

в языке EME-L метод вызывается при попытке удаления строки диалога или строки таблицы. Транзакция еще не открыта. Чтобы различать, для какого объекта вызван метод:

```EME-L
if (is_rec() == Object("Dialog").GetRecord())
...
end if
```

**Входные параметры:** отсутствуют.

**Выходные параметры:** если метод вернёт `True` — удаление отменяется.

```EME-L
Dialog_OnCheckDelete()
{
    dsOrder = Object("dsDB", "Orders");
    dsOrder.SetLine(Object("Dialog").GetLine());
    If (dsOrder.IsValidLine())
        If (dsOrder.GetOrdersBatchRef() != NULL_REF)
            is_message("Внимание!",
                is_format("Данный заказ находится в пачке \"%s\". Удаление отменено.",
                dsOrder.GetOrdersBatch().GetRegNo()), 1, 3);
            Return True;
        End If
    End If
    Return False;
}
```

### Dialog_OnCheckReferences

в EME БД метод производит проверку на наличие ссылок при удалении строки. Если метод вернёт `True`, удаление произойдет независимо от ссылок. Для проверки используются `is_ref_to()`, `is_exist_ref_to()`.

**Входные параметры:** отсутствуют.

**Выходные параметры:** если метод вернёт `True` — ссылки игнорируются, строка удаляется.

```EME-L
Dialog_OnCheckReferences()
{
    RecOne = is_try_find_record("Документы переупаковки");
    FieldOne = is_try_find_field(RecOne, "@Документ");
    RefToRec = RecOne + "," + FieldOne;
    res = is_ref_to(RefToRec, 0, is_rec(), is_dlg_line());
    If (res == 0)
        Return True;
    End If
    Return False;
}
```

### Dialog_OnDelete

в системе EME.WMS метод вызывается при удалении строки диалога. Транзакция уже открыта, строка еще не удалена. Можно удалить связанные строки в той же транзакции.

**Входные параметры:** отсутствуют.

**Выходные параметры:** если метод вернёт `True` — строка не удаляется.

```EME-L
Dialog_OnDelete()
{
    r_Order = Object("dsDB", "Orders");
    r_Order.SetLine(Object("Dialog").GetLine());
    r_DocLines = r_Order.GetFirstOrderLine();
    For (r_DocLines.SetFirstLine(); r_DocLines.IsValidLine(); r_DocLines.SetNextLine())
        r_DocLines.DeleteAndSetLine();
    End For
}
```

### Dialog_OnAfterDelete

в языке EME-L метод вызывается после удаления строки диалога или строки из таблицы.

**Входные параметры:** отсутствуют.

**Выходные параметры:** отсутствуют.

```EME-L
Dialog_OnAfterDelete()
{
    is_transaction(1, "Применение роли к пользователям");
    UpdateRoleUsersMenu(lUserRefForRemovedMenu);
    is_transaction(-1);
}
```

### Dialog_OnChangeTableLine

в EME БД метод вызывается при изменении текущей строки любой таблицы на диалоге. Вызывается для диалога, а не для таблицы. Для различения таблиц используется `is_table()`.

**Входные параметры:** отсутствуют.

**Выходные параметры:** отсутствуют.

```EME-L
Dialog_OnChangeTableLine()
{
    If (is_table() != "Inventory.SectorTable")
        Return;
    End If
    Object("Table", , "CompareTable").Update();
}
```

### Dialog_OnChangeTableColumn

в системе EME.WMS метод вызывается при изменении текущего столбца таблицы. Для различения таблиц — `is_table()`. Объект таблицы — `Object("Table")`.

**Входные параметры:** отсутствуют.

**Выходные параметры:** отсутствуют.

```EME-L
Dialog_OnChangeTableColumn()
{
    Table = Object("Table", , "MainTable");
    For (col = 0; col < Table.GetNoOfColumns(); col = col + 1)
        Table.SetColumnColor(col, is_RGB(255, 255, 255));
    End For
    Table.SetColumnColor(Table.GetCurrentColumn(), is_RGB(255, 255, 192));
}
```

### Dialog_OnDeleteTableLine

в языке EME-L метод вызывается при удалении строк таблиц на диалоге. Для различения таблиц — `is_table()`.

**Входные параметры:** отсутствуют.

**Выходные параметры:** отсутствуют.

```EME-L
Dialog_OnDeleteTableLine()
{
    ValuesTable = Object("Table", , "ValuesTable");
    If (is_table() == ValuesTable.GetName())
        colValue = Object("Control", , "colValue");
        Sum = 0;
        For (i = 0; i < ValuesTable.GetNoOfLines(); i = i + 1)
            Sum = Sum + colValue.GetFromRow(i);
        End For
        Object("Control", , "edSum").Put(Sum);
    End If
}
```

### Dialog_OnReport(ReportName)

в EME БД метод вызывается при нажатии на кнопку печати. Параметр `ReportName` содержит название выбранного отчета. Если метод вернёт `True` — стандартная обработка не производится.

**Входные параметры:**
- `ReportName` — строка с названием печатной формы.

**Выходные параметры:** если метод вернёт `True` — стандартная обработка не производится.

```EME-L
Dialog_OnReport(ReportName)
{
    If (ReportName == "Унифицированная форма МХ-14")
        Object("System").InitQueryCache();
        Query = Object("Query", "Инвентаризация.МХ-14.Результат");
        Query.Execute();
        If (Query.GetNoOfLines() == 0)
            is_message("Печать документа невозможна!",
                "В заказе нет ни одной строки", 1, 3);
            Object("System").ClearQueryCache();
            Return True;
        End If
        Query.PutDialogVariable("Результат запроса");
        Object("Dialog").RunReport("Унифицированная форма МХ-14");
        Object("System").ClearQueryCache();
        Return True;
    End If
}
```

### Dialog_OnCommandNew

в системе EME.WMS метод вызывается при нажатии кнопки "Новый". Если вернёт `True` — создание отменяется.

**Входные параметры:** отсутствуют.

**Выходные параметры:** если метод вернёт `True` — обработка прерывается.

```EME-L
Dialog_OnCommandNew()
{
    If (addMode == -1)
        Return True;
    End If
}
```

### Dialog_OnCommandDelete

в языке EME-L метод вызывается при нажатии кнопки "Удалить". Если вернёт `True` — удаление отменяется.

**Входные параметры:** отсутствуют.

**Выходные параметры:** если метод вернёт `True` — обработка прерывается.

```EME-L
Dialog_OnCommandDelete()
{
    If (is_message("Внимание!", "Удалить класс?", 4, 1, 1) == False)
        Return 1;
    End If
}
```

### Dialog_OnTabItemChanged

в EME БД метод вызывается при смене закладки. Для определения активной закладки — `Dialog.IsTabItemActive("Название")`.

**Входные параметры:** отсутствуют.

**Выходные параметры:** отсутствуют.

```EME-L
Dialog_OnTabItemChanged()
{
    Dialog = Object("Dialog");
    If (Dialog.IsTabItemActive("Результат просчета пачки"))
        Object("Table", , "TableOK").Update();
    End If
}
```

### Dialog_OnFillPrintForms(Info As "FILL_PRINT_FORMS")

в системе EME.WMS метод вызывается при составлении списка печатных форм. Новая форма добавляется через `Info.AddPrintForm(ID, "Название")`.

**Входные параметры:**
- `Info` — структура со списком печатных форм.

**Выходные параметры:** отсутствуют.

```EME-L
Dialog_OnFillPrintForms(Info As "FILL_PRINT_FORMS")
{
    DName = Object("Dialog").GetName();
    Code = is_system(3);
    r_Forms = Object("dsDB", "PrintsForms");
    For (r_Forms.SetFirstLine(); r_Forms.IsValidLine(); r_Forms.SetNextLine())
        If (r_Forms.GetDialogName() == DName & r_Forms.GetRunCode() == Code)
            Info.AddPrintForm(r_Forms.GetLine(), r_Forms.GetName());
        End If
    End For
}
```

### Dialog_OnRunPrintForm(RunPrintForm As "RUN_PRINT_FORM")

в языке EME-L метод вызывается для отображения выбранной печатной формы. Идентификатор формы — через `Info.GetData()`.

**Входные параметры:**
- `RunPrintForm` — структура с параметрами формы.

**Выходные параметры:** отсутствуют.

```EME-L
Dialog_OnRunPrintForm(RunPrintForm As "RUN_PRINT_FORM")
{
    r_Forms = Object("dsDB", "PrintsForms");
    r_Forms.SetLine(Info.GetData());
    If (~r_Forms.IsValidLine())
        Return;
    End If
    Switch (r_Forms.GetTypesRef())
    Case 0:
        If (is_MSWord_installed())
            Doc = r_Forms.OpenMSWordDoc(False);
            If (~is_NULL(Doc))
                OLEDoc = Doc.GetOLEDoc();
                If (~is_NULL(OLEDoc))
                    FillMsWordTemplate(OLEDoc);
                End If
            End If
        Else
            is_message("Печатные формы", "Microsoft Word не установлен.", 0);
        End If
    Case 1:
        Object("System").RunReport(r_Forms.GetReportName());
    End Switch
}
```

### Dialog_OnHyperTextRef(Ref)

в EME БД метод вызывается при щелчке по гиперссылке в гипертекстовом органе. Параметр `Ref` — полный текст ссылки.

**Входные параметры:**
- `Ref` — строка с гиперссылкой.

**Выходные параметры:** отсутствуют.

```EME-L
Dialog_OnHyperTextRef(Ref)
{
    protocol = is_left(Ref, is_find(Ref, "://"));
    data1 = is_right(Ref, is_length(Ref) - is_find(Ref, "://", 1) - 3);
    data2 = is_right(data1, is_length(data1) - is_find(data1, "/", 1) - 1);
    data1 = is_left(data1, is_find(data1, "/", 1));
    If (protocol == "emeref")
        If (data2 != "")
            r_SchoolBook = Object("dsDB", "SchoolBook");
            r_SchoolBook.SetSkipMode();
            r_SchoolBook.GetCodeFld().MustBeEQ(data2);
            r_SchoolBook.MustBeValid();
            r_SchoolBook.SetFirstLine();
            line = r_SchoolBook.GetLine();
            Object("Dialog").SetLine(line);
        End If
    Else
        If (protocol == "emerobot")
            Help_OnRobot(data1 + "/" + data2);
        End If
    End If
}
```

---

## Обработчики сообщений ядра для органов диалога в ЕМЕ БД

в системе EME.WMS для органов диалога имеется 16 обработчиков сообщений ядра. Органы диалога бывают 14 видов: кнопка, галочка, радио-кнопка, статический текст, текстовый редактор, рисунок, список, междиалоговые закладки, закладки, группа элементов, дерево, комбинированный список, таблица, браузер.

### OnInit для органа диалога

в языке EME-L метод вызывается для всех органов на диалоге в момент его создания (после `OnInit()` и `OnLoad()` для диалога).

**Входные/выходные параметры:** отсутствуют.

```EME-L
SelType_OnInit()
{
    SelType = Object("Control", , "SelType");
    SelType.Clear();
    SelType.Add("Stock Transfer Order");
    SelType.Add("Instruction To Collect");
    SelType.Add("Return To Factory");
    SelType.Add("Purchase Order");
}
```

### OnNewItem для органа диалога

в EME БД метод вызывается при создании новой строки диалога. Можно заполнить орган значением по умолчанию.

**Входные/выходные параметры:** отсутствуют.

```EME-L
RegNo_OnNewItem()
{
    dsDB = Object("dsDB");
    dsDB.SetLine(Object("Dialog").GetLine());
    Object("Control").Put(dsDB.regno_Get(0));
    Return True;
}
```

### OnBeforeUpdate для органа диалога

в системе EME.WMS метод вызывается перед обновлением данных в органе. Можно отменить ядерное заполнение, вернув `True`.

**Входные/выходные параметры:** отсутствуют.

```EME-L
DefaultStock_OnBeforeUpdate()
{
    If (is_empty(bStatusListFilled))
        objCombo = Object("Control");
        objCombo.Add(tr("<Пусто>"));
        AddUserStatuses(objCombo);
        bStatusListFilled = True;
    End If
}
```

### OnAfterUpdate для органа диалога

в языке EME-L метод вызывается после обновления данных в органе диалога. Вызывается каждый раз при обновлении.

**Входные/выходные параметры:** отсутствуют.

```EME-L
FileName_OnAfterUpdate()
{
    FileNameControl = Object("Control");
    FileRecord = Object("dsDB", "FileSystem");
    FileRecord.SetLine(Object("Dialog").GetLine());
    If (FileRecord.IsValidLine())
        FilePath = FileRecord.GetFullPath();
        FileNameControl.Put(FilePath + FileRecord.GetName());
    Else
        FileNameControl.Put("");
    End If
}
```

### OnInput для органа диалога

в EME БД метод вызывается каждый раз при вводе пользователем нового значения (не при программном задании).

**Входные/выходные параметры:** отсутствуют.

```EME-L
DialogRef_OnInput()
{
    ShowButton = Object("Control", , "ShowButton");
    DialogRef = Object("Control", , "DialogRef");
    ShowButton.Disable(DialogRef.Get() == NULL_REF);
}
```

### OnBeforeSave для органа диалога

в системе EME.WMS метод вызывается перед сохранением данных органа. Только для органов, привязанных к БД. Транзакция открыта, данные еще не записаны.

**Входные параметры:** отсутствуют.

**Выходные параметры:** если метод вернёт `True` — данные не сохраняются.

```EME-L
Table_OnBeforeSave()
{
    m_arrNewLines.RemoveAll();
    For (i = table.GetNoOfLines() - 1; i >= 0; i = i - 1)
        db.SetLine(table.GetDBLine(i));
        If(~db.IsValidLine())
            m_arrNewLines.Add(i);
        End If;
    End For;
}
```

### OnAfterSave для органа диалога

в языке EME-L метод вызывается во время записи данных с органа в БД. Данные уже сохранены, транзакция еще не закрыта. Только для органов, привязанных к БД.

**Входные/выходные параметры:** отсутствуют.

```EME-L
RegNo_OnAfterSave()
{
    RegNo = Object("Control");
    dsDB = Object("dsDB");
    dsDB.SetLine(is_dlg_line());
    If (RegNo.Get() == "")
        regno = Object("RegNo", dsDB);
        regno.Init(True, dsDB.GetNoOfLines());
        strRegNo = regno.Get();
        dsDB.PutRegNo(strRegNo);
    End If
}
```

### OnBrowse для органа диалога

в EME БД метод вызывается для органов с галочкой "Использовать браузер". При нажатии на "..." или двойном щелчке.

**Входные/выходные параметры:** отсутствуют.

```EME-L
Stockman_OnBrowse()
{
    Browser = Object("Browser", "StockStaffBrowser.Browser");
    Line = Browser.Run(0);
    If (Line != NULL_REF)
        Staff = Browser.GetDBObject();
        Staff.SetLine(Line);
        Stockman = Object("Control", , "Stockman");
        Stockman.PutText(Staff.Get("Фамилия"));
        Stockman.Put(Line);
    End If
}
```

### OnEdit для органа диалога

в системе EME.WMS метод вызывается при начале редактирования органа. Для ячейки таблицы — при попытке редактирования. Если вернёт `True` — редактирование ячейки таблицы запрещается.

**Входные параметры:** отсутствуют.

**Выходные параметры:** для ячейки таблицы — `True` запрещает редактирование.

```EME-L
FIO_OnEdit()
{
    GlobalFIOValue = Object("Control").Get();
}
```

### OnEndEdit для органа диалога

в языке EME-L метод вызывается при завершении редактирования. Для ячейки таблицы — если она открывалась для редактирования.

**Входные/выходные параметры:** отсутствуют.

```EME-L
FIO_OnEndEdit()
{
    If (Object("Control").Get() == "")
        Object("Control").Put(GlobalFIOValue);
    End If
}
```

### OnCheck(Transaction) для органа диалога

в EME БД метод предназначен для проверки данных органа перед сохранением. Вызывается два раза — до транзакции и при открытой транзакции. Нельзя использовать для записи данных — только для проверки.

**Входные параметры:**
- `Transaction` — флаг транзакции.

**Выходные параметры:** если метод вернёт `True` — сохранение отменяется.

```EME-L
Qty_OnCheck(Transaction)
{
    Qty = Object("Control").Get();
    If (Qty < 0)
        is_transaction(0);
        is_message("Ввод заказов", "Количество не может быть отрицательным.", 1);
        Return True;
    End If;
}
```

### OnCheckDelete для органа диалога

в системе EME.WMS метод вызывается при попытке удаления строки в таблице. Только для таблиц. Если вернёт `True` — удаление запрещается.

**Входные параметры:** отсутствуют.

**Выходные параметры:** если метод вернёт `True` — удаление запрещается.

```EME-L
PONoTable_OnCheckDelete()
{
    PONoTable = Object("Table", "PONoTable");
    colPO = Object("Control", , "PO");
    r_PO = Object("dsDB", "PurchaseOrder");
    row = PONoTable.GetCurrentRow();
    r_PO.SetLine(colPO.GetFromRow(row));
    If (~r_PO.IsValidLine())
        Return True;
    End If
    Return False;
}
```

### OnTimer для органа диалога

в языке EME-L метод вызывается для органов, обновляемых по таймеру. Таймеры устанавливаются через `SetControlTimer` объекта диалога.

**Входные/выходные параметры:** отсутствуют.

```EME-L
Timer_OnTimer()
{
    is_ws_align_data();
    r_MailFolder = Object("dsDB","MailFolder");
    r_MailFolder.SetLine(ActiveFolder);
    If (r_MailFolder.IsValidLine() & r_MailFolder.IsFolderInbox())
        LoadMailData();
        OnSelect(r_MailFolder);
    End If
}
```

### OnContextMenu для органа диалога

в EME БД метод вызывается при показе контекстного меню. Можно заменить стандартное меню на свое.

**Входные параметры:** отсутствуют.

**Выходные параметры:** если метод вернёт `True` — стандартное меню не показывается.

```EME-L
GoodsCode_OnContextMenu()
{
    Menu = Object("Menu", True);
    Menu.AppendMenu("STRING", 1, tr("Cписок движений ERP-WMS"));
    Menu.AppendMenu("SEPARATOR");
    Menu.AppendMenu("STRING", 4, tr("Системное контекстное меню"));
    Menu.SetDefaultItem(1);
    Switch (Menu.TrackPopupMenuWithMousePos())
    Case 1:
        RunOperationsReport(Object("Control").Get());
    Case 4:
        Return;
    End Switch
    Return 1;
}
```

### OnCommand для органа диалога

в системе EME.WMS метод предназначен для обработки нажатия кнопки. Вызывается только для кнопок.

**Входные/выходные параметры:** отсутствуют.

```EME-L
CloseBtn_OnCommand()
{
    Object("Dialog").Close();
}
```

### OnGetProperty(Param As "GET_PROPERTY_NOTIFY") для органа диалога

в языке EME-L метод вызывается при получении контекстного меню для органа. Позволяет добавить свои пункты. Вызывается два раза: для заполнения меню и для обработки выбора.

**Входные параметры:**
- `Param` — структура с настройками контекстного меню.

**Выходные параметры:** отсутствуют.

```EME-L
GoodsCode_OnGetProperty(Param As "GET_PROPERTY_NOTIFY")
{
    If (Info.GetMenuItemId() == "")
        Info.AppendMenuItem("Наличие на складах", "PROP_REM");
        Info.AppendMenuItem("Свойства", "PROP_ID");
        Return True;
    Else
        dbGoodsProp = Object("dsDB", "GoodsItem");
        Info.SetRecord(dbGoodsProp.GetRecord());
        Info.SetField(0);
        Info.SetLine(GoodsCode.Get());
        Info.SetReferenceFlag(True);
        dbGoodsProp.SetLine(Info.GetLine());
        If (~dbGoodsProp.IsValidLine())
            Return True;
        End If
        If(Info.GetMenuItemId() == "PROP_REM")
            Object("Dialog", "RemaindersGoods.Dialog").Show(0, 0, is_format("@Продукт=%d", dbGoodsProp.GetLine()));
            Return True;
        End If
        If (Info.GetMenuItemId() == "PROP_ID")
            is_property_dialog("GoodsItem.Dialog", dbGoodsProp.GetLine(), 0);
            Return True;
        End If
    End If
}
```

---

## Обработчики сообщений ядра для таблиц в ЕМЕ БД

в EME БД для таблиц имеется 7 обработчиков сообщений ядра.

### Table_OnSetNextCell

в системе EME.WMS метод вызывается при вводе данных в ячейку и нажатии Enter. Если вернёт `True` — перенос фокуса на следующую ячейку не производится.

**Входные параметры:** отсутствуют.

**Выходные параметры:** если `True` — фокус не переносится.

```EME-L
Table_OnSetNextCell()
{
    Table = Object("Table", , "Table");
    If (Table.GetCurrentRow() < Table.GetNoOfLines())
        If (Table.GetCurrentColumn() == 0)
            Table.SetFocus(2, );
            Return True;
        End If
    End If
    Return False;
}
```

### Table_OnClickColumnHeader(Info As "TABLE_XCLICK_ON_COLHDR_NOTIFY")

в языке EME-L метод вызывается при нажатии левой кнопки мыши по заголовку колонки.

**Входные параметры:**
- `Info` — структура с информацией о столбце.

**Выходные параметры:** отсутствуют.

### Table_OnMarkTableLine(Info As "HEADER_MARK_TABLE_LINE")

в EME БД метод вызывается при попытке выделения строки в таблице с поддержкой выделения. Если вернёт `True` — строка не помечается.

**Входные параметры:**
- `Info` — структура с информацией.

**Выходные параметры:** если `True` — строка не помечается.

```EME-L
Table_OnMarkTableLine(Info As "HEADER_MARK_TABLE_LINE")
{
    Row = Info.GetCurrentRow();
    FlagsInTable = Object("Control", , "FlagsInTable");
    If (is_and(FlagsInTable.GetFromRow(Row), 0x1))
        is_message("Внимание", "Документ уже сохранен ранее.", 1, 1);
        Return True;
    End If
}
```

### Table_OnSkip

в системе EME.WMS метод вызывается при заполнении привязанной к БД таблицы. Позволяет отфильтровать строки. Если вернёт `True` — строка не добавляется.

**Входные параметры:** отсутствуют.

**Выходные параметры:** если `True` — строка пропускается.

```EME-L
Table_OnSkip()
{
    VendorFilter = Object("Control", , "VendorFilter");
    MatrixRec.SetLine(is_line());
    Return (MatrixRec.GetVendorRef() != VendorFilter.Get());
}
```

### Table_OnCommandDelete

в языке EME-L метод вызывается при нажатии "Удалить" в таблице. Если вернёт `True` — обработка прерывается.

**Входные параметры:** отсутствуют.

**Выходные параметры:** если `True` — удаление отменяется.

```EME-L
Table_OnCommandDelete()
{
    Table = Object("Table", , "Table");
    Row = Table.GetCurrentRow();
    If (Row < Table.GetNoOfLines())
        FlagAuthorized = Object("Control", , "FlagAuthorized");
        If (FlagAuthorized.GetFromRow(Row) != 0)
            is_message("Внимание", "Утвержденную строку удалять нельзя.", 1, 1);
            Return True;
        End If
    End If
}
```

### Table_OnClick

в EME БД метод вызывается при двойном щелчке по ячейке таблицы. Если вернёт `True` — обработка не производится.

**Входные параметры:** отсутствуют.

**Выходные параметры:** если `True` — обработка не производится.

```EME-L
Table_OnClick()
{
    Table = Object("Table", , "Table");
    Col = Table.GetCurrentColumn();
    Row = Table.GetCurrentRow();
    If (Col > 0 & Row < Table.GetNoOfLines())
        Value = arValues.GetAt(Col - 1).GetAt(Row);
        Value = If (Value == 0x02, 0, Value + 1);
        arValues.GetAt(Col - 1).SetAt(Row, Value);
        Table.SetCellColor(Col, Row, arColors.GetAt(Value));
    End If
    Return True;
}
```

### Table_OnRClick(Info As "TABLE_RCLICK_ON_CELL_NOTIFY")

в системе EME.WMS метод вызывается при нажатии правой кнопки мыши по ячейке таблицы. Если вернёт `True` — стандартная обработка не производится.

**Входные параметры:**
- `Info` — структура с информацией о контекстном меню.

**Выходные параметры:** если `True` — обработка не производится.

```EME-L
Table_OnRClick()
{
    Table = Object("Table", , "Table");
    If (~Table.IsValidRow())
        Return True;
    End If

    Menu = Object("Menu");
    Menu.CreatePopupMenu();
    Menu.AppendMenu("STRING", 0, "Выделить строку");
    Menu.AppendMenu("SEPARATOR");
    Menu.AppendMenu("STRING", 1, "Выделить всё");
    Menu.AppendMenu("STRING", 2, "Очистить выделение");

    Selected = Menu.TrackPopupMenuWithMousePos();
    NoOfRows = Table.GetNoOfLines();
    Switch (Selected)
        Case 0:
            Row = Table.GetCurrentRow();
            If (Table.IsValidRow())
                Table.MarkRow(Row);
            End If
        Case 1:
            For (Row = 0; Row < NoOfRows; Row = Row + 1)
                Table.MarkRow(Row);
            End For
        Case 2:
            For (Row = 0; Row < NoOfRows; Row = Row + 1)
                Table.MarkRow(Row, False);
            End For
    End Switch
    Return True;
}
```

---

## Обработчики сообщений ядра для браузеров в ЕМЕ БД

в языке EME-L для браузеров имеется 11 обработчиков сообщений ядра.

### Browser_OnInit

в EME БД метод вызывается в момент создания браузера. Обычно используется для запоминания объекта браузера в глобальную переменную. Каждый раз при открытии создается новый объект браузера.

**Входные/выходные параметры:** отсутствуют.

```EME-L
Browser_OnInit()
{
    Browser = Object("Browser");
}
```

### Browser_OnLoad

в системе EME.WMS метод вызывается при запуске браузера, после полного создания объекта.

**Входные/выходные параметры:** отсутствуют.

```EME-L
Browser_OnLoad()
{
    Object("Browser").UnMarkAll();
}
```

### Browser_OnDestroy

в языке EME-L метод вызывается при удалении объекта браузера (обычно при закрытии). Если объект записан в глобальную переменную, он не будет уничтожен при закрытии.

**Входные/выходные параметры:** отсутствуют.

```EME-L
Browser_OnDestroy()
{
    bbFilter.LoadHewVariableSelection(Object("Dialog"), "Товарная единица");
}
```

### Browser_OnSelect(Selected)

в EME БД метод вызывается при попытке пометить строку браузера (пробел или правая кнопка мыши). Вызывается до изменения состояния. Если вернёт `True` — состояние не изменится.

**Входные параметры:**
- `Selected` — текущее состояние строки.

**Выходные параметры:** если `True` — состояние не меняется.

```EME-L
SectorsList_OnSelect(Selected)
{
    If (~Selected)
        dbSect = Object("dsDB", "Sectors");
        dbSect.SetLine(is_line());
        Switch (dbSect.GetInventoryStatus())
            Case 1:
                is_message("Внимание!", "Сектор уже инвентаризуется", 1, 3);
                Return 1;
            Case 2:
                If (~is_message("Внимание!", "Сектор уже был инвентаризован. Повторно?", 4, 2, 1))
                    Return 1;
                End If
        End Switch
    End If
}
```

### Browser_OnAfterSelect

в системе EME.WMS метод вызывается после пометки строки браузера (аналог `OnSelect`, но после изменения состояния).

**Входные/выходные параметры:** отсутствуют.

```EME-L
Browser_OnAfterSelect()
{
    For (Line = Browser.GetSelected(0); Line != NULL_REF; Line = Browser.GetSelected(Line + 1))
        PartsRecord.SetLine(Line);
        If(~PartsRecord.IsOwner())
            Object("Browser").Select(Line, 0);
        End If;
    End For;
    Update();
}
```

### Browser_OnCommand

в языке EME-L метод вызывается при выборе строки браузера (Enter или двойной щелчок).

**Входные/выходные параметры:** отсутствуют.

```EME-L
Browser_OnCommand()
{
    ClientsDialog = Object("Dialog", "Clients.Main.Dialog");
    ClientsDialog.AddFlags(0x0004, False);
    ClientsDialog.AddFlags(0x0008, True);
    ClientsDialog.ShowLine(False, Object("Browser").GetFocusLine());
}
```

### Browser_OnMarkTreeItem(Info As "ITEM_MARK_NOTIFY")

в EME БД метод вызывается при пометке строки в древовидном браузере. Позволяет управлять пометкой родительских и дочерних строк через `Info.SetMarkParent()` и `Info.SetMarkChildren()`.

**Входные параметры:**
- `Info` — структура с дополнительной информацией.

**Выходные параметры:** если `True` — стандартная обработка не производится.

```EME-L
BrowserGroups_OnMarkTreeItem(Info As "ITEM_MARK_NOTIFY")
{
    If (is_key_press(0x10))
        Info.SetMarkParent(True);
        Info.SetMarkChildren(True);
    Else
        Info.SetMarkParent(False);
        Info.SetMarkChildren(False);
    End If
    Return True;
}
```

### Browser_OnTune

в системе EME.WMS метод вызывается перед заполнением браузера. Позволяет настроить браузер (например, на цепочку).

**Входные/выходные параметры:** отсутствуют.

```EME-L
Browser_OnTune()
{
    Browser = Object("Browser");
    Browser.SetChain("@Склад", Object("Dialog").GetLine());
}
```

### Browser_OnSkip(Record)

в языке EME-L метод предназначен для фильтрации строк браузера. Если вернёт `True` — строка не добавляется.

### Browser_OnSkipEx

в EME БД метод вызывается один раз при заполнении браузера и позволяет заполнить его целиком. Используются методы `UnloadAllLines`, `LoadLine`, `LoadLines`.

**Входные/выходные параметры:** отсутствуют.

```EME-L
ZoneBrowser_OnSkipEx()
{
    Browser = Object("Browser");
    Browser.UnloadAllLines();
    r_Zone = Object("dsDB", "Zone");
    r_Zone.SetChain("@Склад", Object("Dialog").GetLine());
    Browser.LoadLines(r_Zone);
}
```

### Browser_OnStartAddDialog

в системе EME.WMS метод вызывается при открытии привязанного к браузеру диалога. Если вернёт `True` — диалог не откроется.

**Входные параметры:** отсутствуют.

**Выходные параметры:** если `True` — диалог не открывается.

```EME-L
ClientBrowser_OnStartAddDialog()
{
    Object("Dialog", "Clients.Dialog").ShowLine(0, NEW_LINE_MARK, 1);
    Return True;
}
```

---

## Обработчики сообщений ядра для ячеек браузера в ЕМЕ БД

в языке EME-L для ячеек браузера имеется 3 обработчика.

### OnPutColumn(objColumn As "Control")

в EME БД метод вызывается во время заполнения ячейки браузера. Позволяет заполнять колонки вручную. Нельзя создавать объекты внутри метода (оператор `Object` запрещён) — объекты нужно создавать в конструкторе.

**Входные параметры:**
- `objColumn` — объект колонки браузера, тип `"Control"`.

**Выходные параметры:** отсутствуют.

```EME-L
OrdersBrwPOD_OnPutColumn(objColumn As "Control")
{
    m_r_Orders.SetLine(is_line());
    if (m_r_Orders.IsPOD())
        objColumn.Put("+");
    end if
}
```

### OnEdit для ячейки браузера

в системе EME.WMS метод вызывается при редактировании ячейки (флажок "Редактируется"). Если вернёт `True` — редактирование запрещено.

**Входные параметры:** отсутствуют.

**Выходные параметры:** если `True` — редактирование запрещается.

### OnInput для ячейки браузера

в языке EME-L метод вызывается при задании значения в ячейке браузера. Требуется флажок "Редактируется".

**Входные/выходные параметры:** отсутствуют.

---

## Обработчики сообщений ядра для интегратора в ЕМЕ БД

в EME БД для интегратора имеется 20 обработчиков. Если на диалоге несколько интеграторов, методы называются `ИмяИнтегратора_ИмяМетода()`. Методы заполнения должны содержать имя записи: `OnGetChild{ИмяЗаписи}`, `OnGetSibling{ИмяЗаписи}`, `OnGetParent{ИмяЗаписи}`, `OnGetItemText{ИмяЗаписи}`, `OnGetItemIcon{ИмяЗаписи}`.

### Integrator_OnGetChild{ИмяЗаписи}(Rec As "CEMERec")

в системе EME.WMS метод вызывается для видимых элементов интегратора для поиска дочерних элементов. Возвращает объект записи дочернего элемента или `NULL`. Рекомендуется использовать `Duplicate()`.

**Входные параметры:**
- `Rec` — объект записи текущего элемента.

**Выходные параметры:** объект записи дочернего элемента или `NULL`.

```EME-L
Integrator_OnGetChildDetails(Rec As "CEMERec")
{
    r = Object("dsDB", "DetailsRepository").Duplicate();
    r.SetChain("@Категория", Rec.GetLine());
    r.SetFirstLine();
    If (r.IsValidLine())
        Return r;
    End If
    Return NULL;
}
```

### Integrator_OnGetSibling{ИмяЗаписи}(Rec As "CEMERec")

в языке EME-L метод вызывается для определения соседних элементов. Возвращает объект записи или `NULL`.

```EME-L
Integrator_OnGetSiblingDetailsRepository(Rec As "CEMERec")
{
    r = Rec.Duplicate();
    r.SetChain("@Категория", Rec.GetCategoryRef());
    r.SetLine(Rec.GetLine());
    r.SetNextLine();
    If (r.IsValidLine())
        Return r;
    End If
    Return NULL;
}
```

### Integrator_OnGetParent{ИмяЗаписи}(Rec As "CEMERec")

в EME БД метод предназначен для поиска родительского элемента.

```EME-L
Integrator_OnGetParentDetails(Rec)
{
    r = Object("dsDB", "Groups").Duplicate();
    r.SetLine(Rec.GetGroupRef());
    If (r.IsValidLine())
        Return r;
    End If
    Return NULL;
}
```

### Integrator_OnGetItemText{ИмяЗаписи}(Rec As "CEMERec")

в системе EME.WMS метод вызывается при добавлении нового элемента. Возвращает текст элемента.

```EME-L
Integrator_OnGetItemTextDetails(Rec As "CEMERec")
{
    Return Rec.GetName();
}
```

### Integrator_OnGetItemIcon{ИмяЗаписи}(Rec As "CEMERec")

в языке EME-L метод вызывается для задания иконки элемента. Возвращает строку вида `"Модуль#Ресурс"`.

```EME-L
Integrator_OnGetItemIconSchoolBook(Rec)
{
    r_SchoolBook = Object("dsDB", "SchoolBook");
    r_SchoolBook.SetChain("@Родитель", Rec.GetLine());
    r_SchoolBook.SetFirstLine();
    If (r_SchoolBook.IsValidLine())
        Return "MetC_Sys.dll#249";
    End For
    Return "MetC_Sys.dll#251";
}
```

### Integrator_OnAttach(IntegratorObject As "CEMEDBIntegrator::Interface")

в EME БД метод вызывается в момент создания интегратора.

```EME-L
IntegratorRepos_OnAttach(IntegratorObject As "CEMEDBIntegrator::Interface")
{
    IntegratorRepos = IntegratorObject;
    IntegratorRepos.EnableSelection(True);
}
```

### Integrator_OnGetMenuCount(Rec)

в системе EME.WMS метод возвращает количество пунктов контекстного меню.

```EME-L
Integrator_OnGetMenuCount(Rec)
{
    Return 3;
}
```

### Integrator_OnGetMenuItem(MenuItem As "DBContextMenu", Index)

в языке EME-L метод формирует параметры контекстного меню.

```EME-L
Integrator_OnGetMenuItem(MenuItem As "DBContextMenu", Index)
{
    Switch (Index)
        Case 0: MenuItem.SetText("Переименовать...");
        Case 1: MenuItem.SetText("Свойства...");
        Case 2: MenuItem.SetText("Удалить");
    End Switch
}
```

### Integrator_OnCommand(Rec As "CEMERec", ID)

в EME БД метод обрабатывает выбор пункта контекстного меню.

```EME-L
Integrator_OnCommand(Rec As "CEMERec", ID)
{
    Switch (ID)
        Case 0: DoRename();
        Case 1: DoProperties();
        Case 2: DoDelete();
    End Switch
}
```

### Integrator_OnGetContextMenu(Rec As "CEMERec")

в системе EME.WMS метод создает и отображает контекстное меню с возможностью вложенных пунктов (в отличие от `OnGetMenuCount`/`OnGetMenuItem`/`OnCommand`).

```EME-L
Integrator_OnGetContextMenu(Rec As "CEMERec")
{
    Menu = Object("Menu");
    Menu.CreatePopupMenu();
    Menu.AppendMenu("STRING", 1, "Переименовать...", "ENABLED");
    Menu.AppendMenu("STRING", 2, "Свойства...", "ENABLED");
    Menu.AppendMenu("STRING", 3, "Удалить", "ENABLED");
    Res = Menu.TrackPopupMenuWithMousePos();
    Switch (Res)
        Case 1: DoRename();
        Case 2: DoProperties();
        Case 3: DoDelete();
    End Switch
}
```

### Integrator_OnSelect(Rec As "CEMERec")

в языке EME-L метод вызывается при выделении элемента интегратора.

```EME-L
Integrator_OnSelect(Rec As "CEMERec")
{
    Integrator.Expand(Integrator.GetSelectedItem());
}
```

### Integrator_OnSelection(Rec As "CEMERec", Item)

в EME БД метод вызывается при нажатии на флажок выделения элемента.

**Входные параметры:**
- `Rec` — объект записи элемента.
- `Item` — ссылка на элемент интегратора.

### Integrator_OnEdit(Rec As "CEMERec")

в системе EME.WMS метод вызывается по нажатию левой кнопки мыши или Enter.

```EME-L
Integrator_OnEdit(Rec As "CEMERec")
{
    Line = Rec.GetLine();
    Object("Dialog").SetLine(Line);
}
```

### Integrator_OnMark(Rec As "CEMERec")

в языке EME-L метод вызывается при пометке элемента пробелом. Вызывается даже если интегратор не поддерживает выделение.

### Integrator_OnFormatItemText(Item, Text)

в EME БД метод вызывается после `OnGetItemText` и предоставляет ссылку на конкретный элемент для форматирования текста.

```EME-L
Integrator_OnFormatItemText(Item, Text)
{
    ParentItem = Integrator.GetParentItem(Item);
    count = 0;
    strText = Object("String");
    strText.SetString(Text);
    Text = strText.ExtractUpto("("));
    If (ParentItem != 0)
        ParentObj = Integrator.GetItemObject(ParentItem);
        Switch (ParentObj.GetText())
            Case "Входящие":     count = InboxCount(Text);
            Case "Отправленные": count = OutboxCount(Text);
        End Switch
    End If
    If (count > 0)
        Integrator.SetItemBold(Item, True);
        Text = Text + is_format("(%d)", count);
    Else
        Integrator.SetItemBold(Item, False);
    End If
    Return Text;
}
```

### Integrator_OnInsertItem(Item)

в системе EME.WMS метод используется при добавлении нового элемента.

```EME-L
Integrator_OnInsertItem(Item)
{
    ParentItem = Integrator.GetParentItem(Item);
    If (ParentItem > 0)
        ParentObj = Integrator.GetItemObject(ParentItem);
        Obj = Integrator.GetItemObject(Item);
        If (Obj.GetParentRef() != ParentObj.GetLine())
            Integrator.SetItemCut(Item, True);
        End If
    End If
}
```

### Integrator_OnDropItem(TargetItem, SourceItem, TargetIntegrator As "CEMEDBIntegrator::Interface", SourceIntegrator As "CEMEDBIntegrator::Interface")

в языке EME-L метод используется при перетаскивании элементов мышью.

```EME-L
Integrator_OnDropItem(TargetItem, SourceItem, TargetIntegrator As "CEMEDBIntegrator::Interface", SourceIntegrator As "CEMEDBIntegrator::Interface")
{
    TargetObj = Integrator.GetItemObject(TargetItem);
    SourceObj = Integrator.GetItemObject(SourceItem);
    is_transaction(1, "Запись ссылки на родителя");
    SourceObj.Put("@Родитель", TargetObj.GetLine());
    is_transaction(-1);
    Object("Dialog").Update();
}
```

### Integrator_OnSetFocus

в EME БД метод вызывается при получении интегратором фокуса.

```EME-L
Integrator_OnSetFocus()
{
    Object("Control", , "TextInfo").Invisible(False);
}
```

### Integrator_OnKillFocus

в системе EME.WMS метод вызывается при потере интегратором фокуса.

```EME-L
Integrator_OnKillFocus()
{
    Object("Control", , "TextInfo").Invisible(True);
}
```

### Integrator_OnExpand(Item, Action)

в языке EME-L метод вызывается при сворачивании/разворачивании элемента. `Action == 1` — сворачивание, `Action == 2` — разворачивание.

```EME-L
Integrator_OnExpand(Item, Action)
{
    If (Action == 2)
        Integrator.SetItemIcon(Item, "MetC_Sys.dll#250");
    Else
        Integrator.SetItemIcon(Item, "MetC_Sys.dll#249");
    End If
}
```

---

## Обработчики сообщений ядра для отчетов в ЕМЕ БД

в EME БД для отчетов имеется 3 обработчика.

### Report_OnBeginReport

в системе EME.WMS метод вызывается перед формированием отчета. Если вернёт `True` — отчет не формируется. Позволяет создать отчет на данных из запросов, заполненных из EME-L.

**Входные параметры:** отсутствуют.

**Выходные параметры:** если `True` — отчет не формируется.

```EME-L
Report_OnBeginReport()
{
    is_do_wait_cursor(1);
    Object("System").InitQueryCache();
    Query = Object("Query");
    LineQuery = Object("Query");
    QueryProblems = Object("Query");

    CreateLineQuery();

    If (LineQuery.GetNoOfLines() == 0)
        is_message("Внимание", "Матрица не заполнена.", 1, 1);
        Return True;
    End If

    CreateQuery();
}
```

### Report_OnEndReport

в языке EME-L метод вызывается после формирования отчета. Можно завершить действия, начатые в `Report_OnBeginReport()`.

**Входные/выходные параметры:** отсутствуют.

```EME-L
Report_OnEndReport()
{
    Query.Delete();
    LineQuery.Delete();
    QueryProblems.Delete();
    Object("System").ClearQueryCache();
    is_do_wait_cursor(-1);
}
```

### Report_OnCloseReport

в EME БД метод вызывается при закрытии окна отчета. Позволяет обеспечить взаимосвязь между отчетом и диалогом.

**Входные/выходные параметры:** отсутствуют.

```EME-L
Report_OnCloseReport()
{
    ReportShow = False;
}
```

---

## Обработчики сообщений ядра для элементов отчета в ЕМЕ БД

в системе EME.WMS для элементов отчета имеется 5 обработчиков.

### OnInit для элемента отчета

в языке EME-L метод вызывается во время создания элемента отчета.

**Входные/выходные параметры:** отсутствуют.

### OnPut для элемента отчета

в EME БД метод вызывается при заполнении ячеек отчета. Позволяет положить в ячейку произвольное значение.

**Входные/выходные параметры:** отсутствуют.

```EME-L
Client_OnPut()
{
    MatrixRec = Object("dsDB", "OrderMatrix");
    MatrixRec.SetLine(is_report_line());
    If (MatrixRec.IsValidLine())
        Object("Control").Put(MatrixRec.GetClient().GetName());
    End If
}
```

### OnCommand для элемента отчета

в системе EME.WMS метод вызывается при двойном щелчке по элементу. Если у элемента стоит "Разрешить редактирование" — метод не вызывается.

**Входные/выходные параметры:** отсутствуют.

```EME-L
MU_OnCommand()
{
    GoodsItemRef = is_cell_data("Report.GoodsItemRef");
    MUArray = Object("Array");
    r_MU = Object("dsDB", "GoodsItemMU");
    r_MU.SetChain("@Товарная единица", GoodsItemRef);
    For (r_MU.SetFirstLine(); r_MU.IsValidLine(); r_MU.SetNextLine())
        MUArray.Add(r_MU.GetName());
    End For

    Menu = Object("Menu");
    Menu.CreatePopupMenu();
    For (i = 0; i < MUArray.GetSize(); i = i + 1)
        Menu.AppendMenu("STRING", i, MUArray.GetAt(i));
    End For

    Rect = Object("Control", , "MU").GetRect();
    SelItem = Menu.TrackPopupMenu(Rect.GetLeft(), Rect.GetBottom());

    If (SelItem >= 0)
        NewMU = MUArray.GetAt(SelItem);
        is_put_cell_data("Report.MU", NewMU);
    End If
}
```

### OnInput для элемента отчета

в языке EME-L метод вызывается при ручном вводе значения в элемент (флажок "Разрешить редактирование").

**Входные/выходные параметры:** отсутствуют.

```EME-L
ReportQty_OnInput()
{
    If (~Dialog.IsWindow())
        is_message("Внимание", "Диалог закрыт.", 1, 1);
        Return;
    End If

    Column = is_band_column();
    Line = is_cell_data_ex("Строка БД", Column);
    Row = Table.GetRow(Line);
    Value = Object("String", is_cell_data_ex("Document.Inventory.ReportQty", Column));
    If (Table.IsValidRow(Row))
        Value.TrimLeft();
        Value.TrimRight();
        Value.Replace(",", ".");
        Qty = 0.0;
        If (Value.AsString() != "")
            Qty = is_abs(Value.AsReal());
        End If
        Quantity.PutToRow(Qty, Row);
        Quantity.SetModified(True, Row);
        Dialog.SetModified(True);
    End If
}
```

### OnFillGraphWindow для элемента отчета

в EME БД метод вызывается только для объектов "Графический слайд". Предназначен для заполнения содержимого через методы объекта `Object("GraphCell")`.

**Входные/выходные параметры:** отсутствуют.

```EME-L
GraphCell_OnFillGraphWindow()
{
    GC = Object("GraphCell");

    Months = Object("Array");
    Months.Add("янв");
    Months.Add("фев");
    Months.Add("мар");
    Months.Add("апр");
    Months.Add("май");
    Months.Add("июн");
    Months.Add("июл");
    Months.Add("авг");
    Months.Add("сен");
    Months.Add("окт");
    Months.Add("ноя");
    Months.Add("дек");

    CurrMonth = is_month(is_dos_date());
    For (y = 0; y < 2; y = y + 1)
        For (x = 0; x < 6; x = x + 1)
            Rect = GC.CreateRectangle(x * 30, y * 20, 30, 20);
            Rect.SetBackgroundColor(-1);
            Text = Rect.CreateText(Months.GetAt(y * 6 + x));
            Text.SetTextHeight(7);
            Text.SetVAlign(0);
            Text.SetHAlign(2);
            If (CurrMonth == 6 * y + x)
                Rect.SetBackgroundColor(224, 224, 224);
                GC.CreateCross(x * 30, y * 20, 30, 20);
            End If
        End For
    End For

    GC.SetAnisotropicMode();
}
```

---

## Обработчики сообщений ядра для бандов отчета в ЕМЕ БД

в системе EME.WMS для бандов отчета имеется 2 обработчика.

### OnNewItem для банда отчета

в языке EME-L метод вызывается в момент создания ячеек банда. Позволяет контролировать создание строк извне. Добавление строк — через `is_create_band_line(номер_строки)`.

**Входные/выходные параметры:** отсутствуют.

```EME-L
BandLines_OnNewItem()
{
    For (Line = 0; Line < 30; Line = Line + 1)
        If (CurrentQueryLine >= Query.GetNoOfLines())
            break;
        End If
        is_create_band_line(CurrentQueryLine);
        CurrentQueryLine = CurrentQueryLine + 1;
    End For
}
```

### OnSkip для банда отчета

в EME БД метод вызывается при генерации строк банда. Предназначен для фильтрации строк и настройки внешнего вида.

**Входные/выходные параметры:** отсутствуют.

```EME-L
Band_OnSkip()
{
    Control = Object("Control");
    ClientLinesRecord = Object("dsDB", "ClientLines");
    If (ClientLinesRecord.GetRecord() == PDRecordColumn.GetFromRow(PersFileLine))
        ClientLinesRecord.SetLine(PDLineColumn.GetFromRow(PersFileLine));
        If (ClientLinesRecord.IsAccepted())
            Control.SetColorBk(is_RGB(127,127,255));
        Else If (ClientLinesRecord.IsRejected())
            Control.SetColorBk(is_RGB(255, 127, 127));
        Else
            Control.SetColorBk(is_RGB(255, 255, 255));
        End If
        End If
    Else
        Control.SetColorBk(is_RGB(255, 255, 255));
    End If
    Return False;
}
```
