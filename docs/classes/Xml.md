# Класс Xml

> Синонимы для поиска: CIXml CEMEXml Xml класс EME-L, XML-документ EME-L, Msxml2.DOMDocument RapidXML, LoadFile SaveFile LoadXml SaveXml GetNode GetNodes FindNodes GetRootNode GetParentNode GetChildNodes SetFirstNode SetNextNode IsValidNode GetName GetBaseName GetText GetXml GetAttrib SetAttrib EnumAttribs AppendNode PutText AppendProlog PreserveWhiteSpace DefineNamespace ValidateFile SetCRLF, XPath EME-L, XML namespace EME-L

Класс `Xml` в языке EME-L — класс-оболочка для работы с XML-документами на основе `Msxml2.DOMDocument` и альтернативного парсера `RapidXML`. В языке EME-L класс Xml предоставляет методы для загрузки XML из файлов, строк, объектов `DataStorage` и автоматизации, а также сохранения документа в файлы и буферы данных. Поддерживает красивую печать XML с отступами.

В языке EME-L класс Xml реализует навигацию по дереву узлов: поиск одиночного узла или списка узлов по выражению XPath, поиск узлов по имени с поддержкой масок, получение корневого, родительского и дочерних узлов. Для списков узлов поддерживает последовательный перебор (`SetFirstNode`/`SetFirstLine`, `SetNextNode`/`SetNextLine`), проверку валидности текущей позиции (`IsValidNode`/`IsValidLine`) и выбор узла по индексу (`SetNode`).

В языке EME-L класс Xml позволяет читать имя узла (`GetName`, `GetBaseName` без пространства имен), текстовое содержимое (`GetText`), XML-представление узла (`GetXml`) и значения атрибутов (`GetAttrib` с необязательным значением по умолчанию). Поддерживает перечисление имен атрибутов узла в объект `Array` (`EnumAttribs`).

В языке EME-L класс Xml включает средства модификации документа: добавление новых узлов (`AppendNode`) с указанием пространства имен и типа узла, задание текстового содержимого (`PutText`), установка атрибутов (`SetAttrib`) и добавление инструкций обработки в пролог (`AppendProlog`).

В языке EME-L класс Xml поддерживает управление незначащими пробелами (`PreserveWhiteSpace`), определение пространств имен с псевдонимами (`DefineNamespace`), проверку файла на валидность (`ValidateFile`) и сохранение символов перевода строки CRLF без нормализации (`SetCRLF`).

Объект создаётся без параметров. При загрузке документа выполняется автоматический разбор XML с обработкой ошибок. Все операции с узлами возвращают новый объект `Xml`, что позволяет строить цепочки вызовов в языке EME-L.

## Создание объекта класса Xml

В языке EME-L класс Xml имеет один конструктор — без параметров.

```EME-L
'Создаёт пустой XML-документ'
Xml = Object("Xml");
```

Для загрузки содержимого используются методы `Load`, `LoadFile` или `LoadXml`.

## Загрузка и сохранение XML в языке EME-L

В языке EME-L класс Xml предоставляет следующие методы загрузки и сохранения XML.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `LoadFile(путь)` | arg0: String | String | Загружает XML из файла или URL. Пустая строка при успехе, иначе текст ошибки. |
| `SaveFile(путь)` | arg0: String | String | Сохраняет XML в файл. Пустая строка при успехе, иначе текст ошибки. |
| `SavePrettyFile(путь)` | arg0: String | String | Сохраняет XML в файл с отступами и переносами строк. |
| `Load(источник)` | arg0: String / Object (DataStorage, CIDataStorage, CIString, автоматизация) | String | Загружает XML из источника с учётом пролога. |
| `Save(приёмник)` | arg0: Object (DataStorage, CIDataStorage, автоматизация) | String | Сохраняет XML в приёмник данных с прологом. |
| `LoadXml(xml_строка)` | arg0: String / CIString | String | Загружает XML из строки. |
| `SaveXml(строка_объект)` | arg0: CIString (по ссылке) | Empty | Сохраняет XML-представление документа в объект `String`. |
| `Reset()` | — | Empty | Полностью переинициализирует объект, удаляя загруженный документ. |

При загрузке методы возвращают пустую строку, если разбор прошёл успешно, и текстовое описание ошибки в противном случае. В языке EME-L класс Xml при работе с RapidXML не реализованы методы `Save`, `Load(COleVariant)`, `Save(COleVariant)`, `LoadXml`, `SaveXml`, `ValidateFile` и `PreserveWhiteSpace`; в MSXML-режиме все перечисленные методы работают.

## Навигация по дереву узлов в языке EME-L

В языке EME-L методы поиска класса Xml возвращают новый объект `Xml`. Если узел не найден, возвращается пустой узел, для которого `IsEmptyNode()` равен `TRUE`.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetNode(xpath)` | arg0: String | Xml | Одиночный узел по XPath. Поддерживает пространства имен, зарегистрированные методом `DefineNamespace`. |
| `GetNodes(xpath)` | arg0: String | Xml | Список узлов по XPath. Для перебора используются `SetFirstNode`/`SetNextNode`. |
| `FindNodes(имя_узла)` | arg0: String | Xml | Список всех узлов с заданным именем тега. Поддерживает маски `*` и `?`. |
| `GetRootNode()` | — | Xml | Корневой элемент документа. |
| `GetParentNode()` | — | Xml | Родительский узел текущего узла. Неприменим для корневого документа. |
| `GetChildNodes()` | — | Xml | Список всех дочерних узлов текущего узла. |

## Перебор списка узлов в языке EME-L

В языке EME-L результат `GetNodes`, `FindNodes` и `GetChildNodes` класса Xml — объект `Xml` типа «список узлов». Для перебора используется тот же паттерн, что и для `Record`/`Array`.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `IsEmptyNode()` | — | Boolean | `TRUE`, если текущий узел пустой. |
| `GetCount()` | — | Integer | Количество узлов в списке. Для пустого узла — 0, для одиночного — 1. |
| `SetNode(индекс)` | arg0: Integer | Xml | Устанавливает указатель на узел с указанным индексом (нумерация с 0). Применим только для списка. |
| `SetFirstNode()` | — | Empty | Устанавливает указатель на первый узел списка. |
| `SetNextNode()` | — | Empty | Перемещает указатель на следующий узел списка. |
| `IsValidNode()` | — | Boolean | `TRUE`, если указатель указывает на валидный узел. |
| `SetFirstLine()` | — | Empty | Синоним `SetFirstNode()` в языке EME-L. |
| `SetNextLine()` | — | Empty | Синоним `SetNextNode()` в языке EME-L. |
| `IsValidLine()` | — | Boolean | Синоним `IsValidNode()` в языке EME-L. |

```EME-L
'Перебор всех дочерних узлов корневого элемента в языке EME-L'
Xml = Object("Xml");
Xml.LoadXml("<root><item>1</item><item>2</item></root>");
Nodes = Xml.GetRootNode().GetChildNodes();
Nodes.SetFirstNode();
While (Nodes.IsValidNode())
    Name = Nodes.GetName();
    Text = Nodes.GetText();
    Nodes.SetNextNode();
End While
```

## Чтение данных узла в языке EME-L

В языке EME-L класс Xml предоставляет следующие методы чтения данных узла.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetName()` | — | String | Полное имя узла, включая префикс пространства имен. Неприменим для корневого документа. |
| `GetBaseName()` | — | String | Имя узла без пространства имен. Неприменим для корневого документа. |
| `GetText()` | — | String | Текстовое содержимое узла. Для пустого узла возвращает пустую строку. |
| `GetXml()` | — | String | XML-представление текущего узла или всего документа. |
| `GetAttrib(имя_атрибута)` | arg0: String | String | Значение атрибута. Генерирует ошибку, если атрибут не найден. |
| `GetAttrib(имя_атрибута, значение_по_умолчанию)` | arg0: String, arg1: String | String | Значение атрибута либо значение по умолчанию при отсутствии. |
| `EnumAttribs(массив)` | arg0: Array (по ссылке) | Empty | Добавляет имена всех атрибутов текущего узла в переданный массив. |

## Модификация XML-документа в языке EME-L

В языке EME-L класс Xml предоставляет следующие методы модификации документа.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `AppendNode(имя_или_узел)` | arg0: String или Xml (по ссылке) | Xml | Добавляет новый узел с заданным именем или переносит существующий узел. Внимание: при сборке с RapidXML перенос существующего узла не реализован и вызывает ошибку. |
| `AppendNode(имя_узла, пространство_имен)` | arg0: String, arg1: String | Xml | Добавляет новый узел с указанным пространством имен. |
| `AppendNode(имя_узла, пространство_имен, тип_узла)` | arg0: String, arg1: String, arg2: String | Xml | Добавляет новый узел с пространством имен и типом DOM-узла. По умолчанию тип `ELEMENT`. |
| `PutText(текст)` | arg0: String | Xml | Устанавливает текстовое содержимое текущего узла. |
| `SetAttrib(имя, значение)` | arg0: String, arg1: String | Xml | Устанавливает или изменяет атрибут текущего узла. |
| `AppendProlog(цель, данные)` | arg0: String, arg1: String | Empty | Добавляет инструкцию обработки в пролог документа. В RapidXML-режиме поддерживается только цель `xml`. |

Допустимые значения `тип_узла` для `AppendNode`: `ELEMENT`, `ATTRIBUTE`, `TEXT`, `CDATA_SECTION`, `ENTITY_REFERENCE`, `ENTITY`, `PROCESSING_INSTRUCTION`, `COMMENT`, `DOCUMENT`, `DOCUMENT_TYPE`, `DOCUMENT_FRAGMENT`, `NOTATION`.

## Пространства имен, форматирование и валидация в языке EME-L

В языке EME-L класс Xml предоставляет следующие вспомогательные методы.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `PreserveWhiteSpace()` | — | Empty | Включает сохранение незначащих пробелов. По умолчанию пробелы нормализуются. |
| `PreserveWhiteSpace(сохранять)` | arg0: Boolean | Empty | `TRUE` — сохранять незначащие пробелы; `FALSE` — удалять (по умолчанию). |
| `DefineNamespace(псевдоним, uri)` | arg0: String, arg1: String | Empty | Регистрирует псевдоним пространства имён для использования в XPath. |
| `ValidateFile(путь)` | arg0: String | String | Проверяет файл XML на валидность (DTD/XSD). Пустая строка при успехе. В RapidXML-режиме не реализован. |
| `SetCRLF(сохранять)` | arg0: Boolean | Empty | `TRUE` — при чтении текста возвращать CRLF без нормализации в LF; `FALSE` — нормализовать (по умолчанию). |

## Примеры использования класса Xml в языке EME-L

### Чтение XML из файла

```EME-L
XmlDoc = Object("Xml");
LoadResult = XmlDoc.LoadFile("D:\\Data\\message.xml");

If (LoadResult == "")
    'Читаем значение дочернего узла по имени в языке EME-L'
    Value = XmlDoc.GetRootNode().GetNode("Header").GetNode("DocumentNumber").GetText();
Else
    is_error(4, "Ошибка разбора XML: " + LoadResult);
End If
```

### Создание XML-документа

```EME-L
Xml = Object("Xml");
Xml.AppendProlog("xml", "version=\"1.0\" encoding=\"windows-1251\"");
Root = Xml.AppendNode("root");
Root.SetAttrib("version", "1.0");

Item = Root.AppendNode("item");
Item.SetAttrib("id", "1");
Item.PutText("Первый элемент");

Xml.SaveFile("D:\\Data\\output.xml");
```

### Перебор списка узлов с чтением атрибутов

```EME-L
Xml = Object("Xml");
Xml.LoadXml("<root><item id='1' name='A'/><item id='2' name='B'/></root>");
Items = Xml.GetRootNode().GetNodes("item");
Items.SetFirstNode();

arrNames = Object("Array");
While (Items.IsValidNode())
    Id = Items.GetAttrib("id");
    Name = Items.GetAttrib("name", "Без имени");
    arrNames.Add(Name);
    Items.SetNextNode();
End While
```

## См. также в языке EME-L

- [Класс String](String.md) — работа со строками и преобразования.
- [Класс DataStorage](DataStorage.md) — бинарный буфер для загрузки/сохранения XML.
- [Класс File](File.md) — работа с файлами.
