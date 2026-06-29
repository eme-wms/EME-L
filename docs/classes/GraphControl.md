# Класс GraphControl

> Синонимы для поиска: CIGraphControl GraphControl класс EME-L, графический слайд EME-L, GraphWindow EME-L, графический контрол EME-L, CreateRectangle CreatePolygon CreateRhomb CreateCross CreateHierarchyElement CreateArrow CreateImage CreateDiagram CreateTimeTable CreateHistogramD Create3Diagram CreateHistogramKrostD FillGraphQuery FindElement GetElement GetEventElement GetFirstElement GetNextElement GetFocusedElement AddElementToSelection IsMultiSelection GetSelectionAsArray ClearSelection SetSelectionStyle GetScale SetScale GetMaxScale SetMaxScale GetMinScale SetMinScale SetScrollingPosition ScrollIntoView EnsureVisible GetHandScrollMode SetHandScrollMode SetBackgroundColor SetMargins Empty SetAnisotropicMode EnableToolTips SetNewMode SetRedraw RedrawControl SetFocus EnableFocus EnableFocusForRClick AddElement RemoveElement SetNumberOfElements MakeSeparated SetDragAndDropEnable GetDragAndDropEnable GetTextWidth GetTextHeight GetElementsRect GetClientRect GetOptimalSize IsFixedArea DisableFixedArea FixArea Print GetVersion CreateRef EmptyOnUpdate AutoSelectionOnClick DrawSelection CustomMouseWheel NoModify

Класс `GraphControl` в языке EME-L — класс для программного создания и управления графическими элементами в графическом слайде диалога. В языке EME-L класс GraphControl предоставляет механизм создания графических примитивов (прямоугольники, многоугольники, ромбы, кресты, стрелки), диаграмм, временных рядов, столбчатых диаграмм, а также размещения изображений. Класс GraphControl поддерживает управление масштабом, прокруткой, выделением элементов, фокусом и печатью содержимого слайда.

Объект класса `GraphControl` в языке EME-L создаётся с привязкой к графическому слайду по имени, по имени объекта и мембера класса, или к текущему контексту выполнения. Класс GraphControl используется в прикладных скриптах EME-L для программного формирования интерактивных графических схем и визуализации данных в системе EME.WMS.

Основные возможности класса GraphControl в языке EME-L:

- Создание и размещение графических элементов с заданными координатами и размерами
- Создание диаграмм с возвратом объекта [Diagram](./Diagram.md) для детальной настройки
- Управление масштабом отображения: получение и установка текущего, минимального и максимального масштаба
- Работа с выделением элементов: одиночное и множественное выделение, получение выделенных элементов в массив
- Прокрутка и позиционирование: скроллирование к элементу, установка положения прокрутки
- Поиск элементов по имени или координатам, получение элемента, инициировавшего событие
- Настройка внешнего вида: цвет фона, поля, режим растягивания, стиль выделения, всплывающие подсказки
- Печать содержимого графического слайда
- Заполнение графика данными на основе запроса к базе данных

## Создание объекта класса GraphControl

Класс `GraphControl` в языке EME-L имеет пять конструкторов — от нуля до четырёх аргументов. Каждый конструктор подключает объект к графическому слайду диалога различными способами.

```EME-L
'Конструктор без аргументов — подключение к текущему контексту'
gc = Object("GraphControl");
```

```EME-L
'Конструктор с именем графического слайда'
gc = Object("GraphControl", "Plan");
```

```EME-L
'Конструктор с именем объекта класса и именем мембера'
gc = Object("GraphControl",, "Plan");
```

```EME-L
'Конструктор с именем объекта, мембером и контекстом вызова'
gc = Object("GraphControl",, "Plan", 0);
```

```EME-L
'Конструктор с режимом мягкой обработки отсутствия слайда'
gc = Object("GraphControl",, "Plan", 0, TRUE);
```

В системе EME.WMS при `softMode = TRUE` (последний конструктор) и отсутствии графического слайда объект класса GraphControl создаётся без привязки — вызовы методов на нём безопасны, но не оказывают эффекта. При `softMode = FALSE` генерируется исключение. Если орган не найден в диалоге, генерируется ошибка «Орган %s не найден!». Если орган не является графическим слайдом, генерируется ошибка «Орган %s не является графическим слайдом!».

| Конструктор | Привязка | Поведение при отсутствии слайда |
|-------------|----------|---------------------------------|
| `Object("GraphControl")` | Текущий контекст выполнения | Генерирует исключение |
| `Object("GraphControl", name)` | Графический слайд по имени | Генерирует исключение |
| `Object("GraphControl", objectName, memberName)` | Имя объекта класса + имя мембера | Генерирует исключение |
| `Object("GraphControl", objectName, memberName, context)` | + контекст вызова (Integer) | Генерирует исключение |
| `Object("GraphControl", objectName, memberName, context, softMode)` | + способ обработки отсутствия (Boolean) | `softMode = TRUE` — игнорировать ошибку, объект непривязанный; `FALSE` — генерировать исключение |

## Создание графических элементов

Методы класса GraphControl в языке EME-L создают графические примитивы на слайде и возвращают ссылку на созданный элемент для дальнейшей настройки.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `CreateRectangle(x, y, cx, cy)` | x, y, cx, cy: Integer | Reference (CSquareElement) | Прямоугольник: координаты центра (x, y), ширина cx, высота cy. |
| `CreatePolygon(x, y)` | x, y: Integer | Reference | Многоугольник с центром в заданной точке; форма настраивается на стороне элемента. |
| `CreateRhomb(x, y, cx, cy)` | x, y, cx, cy: Integer | Reference | Ромб: координаты центра (x, y), ширина диагонали по X — cx, по Y — cy. |
| `CreateCross(x, y, cx, cy)` | x, y, cx, cy: Integer | Reference | Диагональный крест: координаты центра (x, y), размеры обрамляющего прямоугольника cx × cy. |
| `CreateHierarchyElement(x, y, cx, cy)` | x, y, cx, cy: Integer | Reference | Группировочный (иерархический) элемент с центром (x, y) и размерами cx × cy. |
| `CreateArrow(x1, y1, x2, y2)` | x1, y1, x2, y2: Integer | Reference (CArrow) | Стрелка от точки (x1, y1) к точке (x2, y2). |
| `CreateImage(x, y, cx, cy)` | x, y, cx, cy: Integer | Reference | Прямоугольное изображение с центром (x, y) и размерами cx × cy. Доступно с версии ядра 25+. |

## Диаграммы и графики

Методы класса GraphControl для создания диаграмм и графиков в языке EME-L. Метод `CreateDiagram` возвращает объект Diagram, методы `CreateTimeTable`, `CreateHistogramD`, `Create3Diagram`, `CreateHistogramKrostD` возвращают ссылки на специализированные объекты графиков.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `CreateDiagram(x, y, cx, cy)` | x, y, cx, cy: Integer | Reference (Diagram) | Создаёт объект, содержащий графики (диаграмму), в области с центром (x, y) и размерами cx × cy. Возвращает объект [Diagram](./Diagram.md) для детальной настройки. |
| `CreateTimeTable()` | — | Reference | Создаёт объект временных рядов для последующего заполнения данными. |
| `FillGraphQuery(timeTable, query, fieldX, fieldY, fieldGraph)` | timeTable: Reference, query: Reference (Query), fieldX, fieldY, fieldGraph: String | Integer или Empty | Заполняет объект временных рядов `timeTable` данными из результата запроса `query`: координата X берётся из поля `fieldX`, значение Y — из `fieldY`, номер графика — из `fieldGraph`. Возвращает количество уникальных графиков, для которых добавлены данные. Возвращает Empty, если хотя бы одно поле отсутствует в запросе или результат пуст. |
| `CreateHistogramD()` | — | Reference | Столбчатая диаграмма. Доступно с версии ядра 24+. |
| `Create3Diagram()` | — | Reference | Диаграмма с треугольниками. Доступно с версии ядра 26+. |
| `CreateHistogramKrostD()` | — | Reference | Столбчатая диаграмма (вариант Krost). Доступно с версии ядра 30+. |

Метод `FillGraphQuery` класса GraphControl в языке EME-L перебирает все строки результата запроса и добавляет значения в соответствующие временные ряды объекта TimeTable. Для каждой строки номер графика определяется полем `fieldGraph`. Значения Y суммируются, если несколько строк запроса попадают в одну точку X одного графика.

## Поиск и навигация по элементам

Методы класса GraphControl в языке EME-L для поиска элементов на графическом слайде и их перебора.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetEventElement()` | — | Reference или Empty | Элемент, инициировавший текущее событие. Empty — событие не связано с элементом. |
| `GetElement(x, y)` | x, y: Integer | Reference или Empty | Элемент под заданными координатами. Empty — элемента нет. Доступно с версии ядра 32.1+. |
| `FindElement(name)` | name: String | Reference или Empty | Поиск элемента по имени. Empty — не найден. |
| `GetFirstElement()` | — | Reference (CSchematicElementBase) или Empty | Первый элемент в контейнере слайда. |
| `GetNextElement(prevElement)` | prevElement: Reference | Reference или Empty | Следующий элемент после `prevElement`. Используется совместно с `GetFirstElement` для перебора всех элементов. Empty — элементов больше нет. |
| `GetNumberOfElements()` | — | Integer | Количество элементов на слайде. Доступно с версии ядра 23+. |
| `GetFocusedElement()` | — | Reference или Empty | Элемент с клавиатурным фокусом. Empty — фокус не установлен. |

## Выделение элементов

Методы класса GraphControl для управления выделением элементов на графическом слайде в языке EME-L.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `AddElementToSelection(element)` | element: Reference | Reference | Добавляет элемент в группу выделения. Возвращает тот же элемент. |
| `IsMultiSelection()` | — | Boolean | TRUE — выделено более одного элемента. |
| `GetSelectionAsArray(array)` | array: Array (по ссылке) | Empty | Заполняет массив `array` всеми выделенными элементами. |
| `ClearSelection()` | — | Empty | Очищает текущее выделение. |
| `SetSelectionStyle(style)` | style: Integer | Integer | Устанавливает стиль выделения: `0` — «бегущие муравьи» (по умолчанию), `1` — рамка. Возвращает предыдущий стиль. Доступно с версии ядра 32+. |

## Масштабирование

Управление масштабом отображения графического слайда в языке EME-L. Масштаб задаётся вещественным числом.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetScale()` | — | Real | Текущий масштаб отображения слайда. |
| `SetScale(scale)` | scale: Real | Reference (GraphControl) | `0` — отключить масштабирование; `>0` — включить с указанным значением. |
| `GetMaxScale()` | — | Real | Максимальный масштаб. |
| `SetMaxScale(scale)` | scale: Real | Reference (GraphControl) | Устанавливает максимальный масштаб. |
| `GetMinScale()` | — | Real | Минимальный масштаб. |
| `SetMinScale(scale)` | scale: Real | Reference (GraphControl) | Устанавливает минимальный масштаб. |

## Прокрутка и позиционирование

Методы класса GraphControl для управления прокруткой графического слайда в языке EME-L.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetScrollingPosition(x, y)` | x, y: Integer | Empty | Прокручивает окно слайда к заданной точке. |
| `ScrollIntoView(element)` | element: Reference | Reference | Прокручивает слайд так, чтобы элемент стал видимым. Возвращает тот же элемент. |
| `EnsureVisible()` | — | Reference (GraphControl) | Прокручивает слайд так, чтобы элемент с фокусом стал видимым. Доступно с версии ядра 19+. |
| `GetHandScrollMode()` | — | Boolean | TRUE — режим скроллинга рукой (аналог Adobe Photoshop) включён. Доступно с версии ядра 20+. |
| `SetHandScrollMode(enable)` | enable: Boolean или Integer | Empty | Включает/выключает режим скроллинга рукой. `TRUE`/ненулевое — включить, `FALSE`/0 — выключить. Доступно с версии ядра 20+. |

## Внешний вид

Настройка внешнего вида графического слайда в языке EME-L. Методы управляют цветом фона, полями, режимом растягивания и подсказками.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetBackgroundColor(color)` | color: Integer (RGB) | Reference (GraphControl) или Empty | Устанавливает цвет фона слайда в формате RGB (одно целое). При неверном типе аргумента возвращает Empty. |
| `SetBackgroundColor(r, g, b)` | r, g, b: Integer (0–255) | Reference (GraphControl) или Empty | То же через раздельные компоненты RGB. |
| `SetMargins(margins)` | margins: Integer | Reference (GraphControl) | Размер полей окна слайда в логических пикселях. |
| `Empty()` | — | Reference (GraphControl) | Очищает слайд от всех элементов и инициализирует значения по умолчанию. |
| `SetAnisotropicMode()` | — | Reference (GraphControl) | Включает режим растягивания изображения по всему органу, игнорируя соотношение сторон. |
| `EnableToolTips([enable])` | enable: Boolean (необязательный) | Boolean | Включает (`TRUE`, по умолчанию) или отключает (`FALSE`) всплывающие подсказки для элементов. Возвращает новое состояние. Доступно с версии ядра 28+. |
| `SetNewMode([enable])` | enable: Boolean (необязательный) | Empty | Включает (`TRUE`, по умолчанию) или отключает новый режим работы графического контрола. Доступно с версии ядра 28+. |
| `SetRedraw(enable)` | enable: Boolean | Boolean | Разрешает/запрещает перерисовку окна при изменении структуры элементов. Возвращает предыдущее состояние перерисовки. Доступно с версии ядра 19+. |
| `RedrawControl()` | — | Reference (GraphControl) | Принудительно обновляет окно слайда. Доступно с версии ядра 19+. |

Метод `SetBackgroundColor` класса GraphControl в языке EME-L допускает вызов с одним аргументом (RGB-цвет одним целым числом) или с тремя (раздельные компоненты r, g, b). Вызов без аргументов или с двумя аргументами генерирует ошибку. При передаче аргумента неверного типа (не целое число) метод возвращает пустое значение и не изменяет цвет.

## Фокус

Управление клавиатурным фокусом ввода в классе GraphControl в языке EME-L.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetFocus([element])` | element: Reference (необязательный) | Reference (GraphControl) | Передаёт фокус окну слайда или указанному элементу. Без аргументов — окну. Доступно с версии ядра 28+ для передачи фокуса конкретному элементу. |
| `EnableFocus()` | — | Reference (GraphControl) | Включает подсветку фокуса выбранного объекта. |
| `EnableFocusForRClick()` | — | Reference (GraphControl) | Включает подсветку фокуса при щелчке правой кнопкой мыши. Если элемент не выделен, выделяет его (с учётом клавиши Ctrl для множественного выделения). |

## Управление элементами

Методы класса GraphControl для добавления, удаления и конфигурации элементов на графическом слайде в языке EME-L.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `AddElement(element)` | element: Reference | Boolean | Добавляет существующий элемент в слайд. `TRUE` — успех. Доступно с версии ядра 32.1+. |
| `RemoveElement(nameOrElement[, deleteElement])` | nameOrElement: String или Reference, deleteElement: Boolean (необязательный, по умолчанию TRUE) | Boolean | Удаляет элемент по имени или ссылке. `TRUE` — успех. Доступно с версии ядра 20+. |
| `SetNumberOfElements(count)` | count: Integer | Empty | Предварительно инициализирует массивы данных для заданного количества элементов. Используется при большом количестве объектов. Доступно с версии ядра 23+. |
| `MakeSeparated()` | — | Empty | Разрешает привязку объектов к границам окна слайда. |
| `SetDragAndDropEnable(enable)` | enable: Boolean или Integer | Empty | Разрешает/запрещает перетаскивание элементов. `TRUE`/ненулевое — разрешить. Доступно с версии ядра 20+. |
| `GetDragAndDropEnable()` | — | Boolean | TRUE — перетаскивание разрешено. Доступно с версии ядра 20+. |

## Измерения и область

Методы класса GraphControl для измерения размеров текста, элементов и клиентской области графического слайда в языке EME-L.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetTextWidth(text, fontHeight, bold)` | text: String, fontHeight: Integer, bold: Boolean | Integer | Ширина строки в пикселях для заданных параметров шрифта. Доступно с версии ядра 28+. |
| `GetTextHeight(text, fontHeight, bold)` | text: String, fontHeight: Integer, bold: Boolean | Integer | Высота строки в пикселях. Доступно с версии ядра 28+. |
| `GetElementsRect(mode)` | mode: Integer | Integer | Размер прямоугольника, охватывающего все элементы: `mode = 0` — ширина, `mode = 1` — высота (в пикселях). Доступно с версии ядра 28+. |
| `GetClientRect(rect)` | rect: CIRect (по ссылке) | Boolean | Записывает координаты клиентской области слайда в объект CIRect. `TRUE` — успех. Доступно с версии ядра 28+. |
| `GetOptimalSize(rect[, scale])` | rect: CIRect (по ссылке), scale: Real (необязательный, по умолчанию −1) | Boolean | Записывает оптимально-минимальный размер слайда в `rect` (left и top обнуляются). `TRUE` — успех. Доступно с версии ядра 28+. |

## Фиксация областей

Методы класса GraphControl для фиксации областей графического слайда в языке EME-L. Доступны с версии ядра 34+.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `IsFixedArea()` | — | Boolean | TRUE — режим фиксации областей включён. |
| `DisableFixedArea()` | — | Boolean | Отключает режим фиксации. `TRUE` — успех. |
| `FixArea(element)` | element: Reference | Boolean | Фиксирует область по фигуре (вызов с одним аргументом). `TRUE` — успех. |
| `FixArea(x, y)` | x, y: Integer | Boolean | Фиксирует область по точке (вызов с двумя аргументами). `TRUE` — успех. |

## Прочее

Прочие методы класса GraphControl в языке EME-L — печать, получение версии, создание объекта-ссылки.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `Print()` | — | Reference (GraphControl) | Распечатывает содержимое слайда; перед печатью запрашивает выбор принтера. |
| `GetVersion()` | — | Integer | Версия графического контрола. |
| `CreateRef(element, x, y[, text][, name])` | element: Reference, x, y: Integer, text: String (необязательный), name: String (необязательный) | Reference | Создаёт объект-ссылку на существующий элемент с размещением в точке (x, y). Доступно с версии ядра 35+. |

## Флаги

Флаги класса GraphControl в языке EME-L устанавливаются как свойства объекта через присваивание `gc.FlagName = TRUE`.

| Флаг | Тип | Версия ядра | Описание |
|------|-----|-------------|----------|
| `EmptyOnUpdate` | Boolean | 28+ | Очищать слайд при обновлении. |
| `AutoSelectionOnClick` | Double-flag | 28+ | Автовыделение элемента по щелчку мыши. |
| `DrawSelection` | Double-flag | 28+ | Отрисовка выделения. |
| `CustomMouseWheel` | Double-flag | 28+ | Кастомная обработка колеса мыши. |
| `NoModify` | Double-flag | 32+ | Запрет модификации элементов. |

## Примеры

Создание прямоугольника и настройка цвета фона:

```EME-L
'Создать графический контрол по имени слайда и задать цвет фона'
gc = Object("GraphControl",, "Plan");
gc.SetBackgroundColor(255, 255, 204);
rect = gc.CreateRectangle(100, 100, 50, 30);
```

Перебор всех элементов слайда:

```EME-L
gc = Object("GraphControl",, "Plan");
elem = gc.GetFirstElement();
While (elem.IsValid())
    'Обработать элемент elem'
    elem = gc.GetNextElement(elem);
End While
```

Заполнение графика данными из запроса:

```EME-L
gc = Object("GraphControl",, "Chart");
tt = gc.CreateTimeTable();
q = Object("Query");
q.AddLine("SELECT x, y, graph_no FROM chart_data");
nGraphs = gc.FillGraphQuery(tt, q, "x", "y", "graph_no");
'nGraphs — количество графиков, в которые добавлены данные'
```

## См. также

- [Класс Diagram](./Diagram.md) — объект, возвращаемый `CreateDiagram` для настройки графиков
- [Класс Query](./../language/advanced/api-and-patterns.md) — источник данных для `FillGraphQuery`
