IrrTransformation CIIrrTransformation AddChild Recalc SetScale SetRotation SetPosition GetAbsolutePosition GetAbsoluteRotation 3D transformation Irrlicht узел преобразований дочерний узел масштаб вращение положение абсолютные координаты

# Класс IrrTransformation — узел 3D-преобразований Irrlicht

Класс `IrrTransformation` (C++ `CIIrrTransformation`) в языке EME-L — класс узла преобразований движка Irrlicht. Представляет самостоятельный узел, который хранит и управляет локальными трёхмерными преобразованиями: положением (`RelativeTranslation`), поворотом (`RelativeRotation`) и масштабом (`RelativeScale`) относительно родительского узла, а также собственной иерархией дочерних узлов (`Children`).

Класс регистрируется в `p_System.cpp:1137` как `REGISTRY_EMEL_CLASS(IrrTransformation, "Класс 3D-преобразований Irrlicht")` и доступен в скриптах только при включённой поддержке Irrlicht (препроцессорный флаг `IRRLICHT_IS_AVAILABLE` — весь файл `i_IrrTransformation.cpp` обёрнут в `#ifdef IRRLICHT_IS_AVAILABLE / #endif`). Реализация находится в `i_IrrTransformation.cpp`, блок `BEGIN_IMPLEMENT(CIIrrTransformation)` содержит 7 зарегистрированных методов.

> В отличие от специализированных узлов сцены Irrlicht (`IrrCube`, `IrrBillboard`, `IrrMesh` и др.), класс `IrrTransformation` не наследует `IrrNode` и не подключается к иерархии сцены Irrlicht через `Add*` методы родительского узла. Это самостоятельный класс преобразований со своей иерархией дочерних узлов, не отображаемый на сцене напрямую. Внутри движка он используется как вспомогательная структура для накопления матрицы трансформации (например, в `i_IrrMeshCreator.cpp:1386` создаётся как C++ локальная переменная для пересчёта координат меша).

## Создание объекта

Класс `IrrTransformation` создаётся через `Object()` без аргументов. В блоке `BEGIN_IMPLEMENT(CIIrrTransformation)` нет макросов `IMPLEMENT_CTOR` — зарегистрированных конструкторов с аргументами нет. В исходном коде C++ определены два конструктора: `CIIrrTransformation()` (без аргументов) и `CIIrrTransformation(int argn, COleVariant args[])` (с аргументами). Оба полностью эквивалентны — вариант с аргументами игнорирует их и инициализирует узел теми же значениями по умолчанию, что и пустой конструктор.

```EME-L
'Создать корневой узел преобразований в языке EME-L'
objTrans = Object("IrrTransformation");
```

При создании применяются значения по умолчанию, заданные в теле конструктора C++ через `core::vector3df(...)`:

| Параметр | Поле C++ | Значение по умолчанию |
|----------|----------|-----------------------|
| Положение | `RelativeTranslation` | `(0.0, 0.0, 0.0)` |
| Вращение | `RelativeRotation` | `(0.0, 0.0, 0.0)` |
| Масштаб | `RelativeScale` | `(1.0f, 1.0f, 1.0f)` |
| Родительский узел | `Parent` | `NULL` (корень иерархии) |

Дочерние узлы создаются методом `AddChild()`, который возвращает ссылку на новый объект `IrrTransformation`, автоматически привязанный к текущему узлу как дочерний. Деструктор `~CIIrrTransformation()` вызывает `removeAll()` и автоматически удаляет все дочерние узлы — вручную освобождать память не нужно (в системе EME.WMS и языке EME-L нет операторов `new`/`delete`).

## Локальные преобразования

Методы `SetScale`, `SetRotation`, `SetPosition` задают локальные значения преобразования узла относительно его родителя. Все три метода зарегистрированы через `IMPLEMENT_METHOD` с числом аргументов 3 и принимают по три вещественных числа — по одному на ось X, Y, Z. Возвращают `Empty` (в C++ возвращают `COleVariant()` без значения).

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetScale(arg0, arg1, arg2)` | arg0: Real (X), arg1: Real (Y), arg2: Real (Z) | Empty | Задаёт локальный масштаб узла по осям X, Y и Z. В C++ вызывает `setScale(core::vector3df(...))`. |
| `SetRotation(arg0, arg1, arg2)` | arg0: Real (X, градусы), arg1: Real (Y, градусы), arg2: Real (Z, градусы) | Empty | Задаёт локальные углы вращения узла в градусах по осям X, Y и Z. В C++ вызывает `setRotation(core::vector3df(...))`. |
| `SetPosition(arg0, arg1, arg2)` | arg0: Real (X), arg1: Real (Y), arg2: Real (Z) | Empty | Задаёт локальное положение узла относительно родительского узла по осям X, Y и Z. В C++ вызывает `setPosition(core::vector3df(...))`. |

Значения `args[N]` извлекаются через `variant2real` и приводятся к `float` перед передачей в `core::vector3df`.

## Иерархия дочерних узлов

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `AddChild()` | — | IrrTransformation (ссылка) | Создаёт новый дочерний узел преобразований, добавляет его в иерархию текущего узла и возвращает ссылку на него. Зарегистрирован через `IMPLEMENT_MTYPED(CIIrrTransformation, CIIrrTransformation, AddChild, 0)` — возвращаемый тип `IrrTransformation`. В C++ создаёт `new CIIrrTransformation()`, вызывает `addChild(pChild)` и возвращает ссылку через `COleVariant` с `vt = VT_OBJECT`. |
| `Recalc()` | — | Empty | Пересчитывает абсолютные 3D-преобразования текущего узла с учётом родительских преобразований, а затем рекурсивно пересчитывает все дочерние узлы. В C++ вызывает `updateAbsolutePosition()`, затем в цикле по `Children` вызывает `(*it)->Recalc()` для каждого дочернего узла. |

Метод `Recalc` должен вызываться после изменения любого локального преобразования (масштаба, вращения, положения), чтобы абсолютные координаты и углы дочерних узлов обновились. Пересчёт выполняется рекурсивно по всей иерархии: сначала обновляется абсолютная матрица текущего узла, затем всех его дочерних узлов. Значения `GetAbsolutePosition`/`GetAbsoluteRotation` корректны только после вызова `Recalc()`.

## Абсолютные преобразования

Методы `GetAbsolutePosition` и `GetAbsoluteRotation` записывают вычисленные абсолютные значения в переданный массив `Array` размером 3 элемента. Оба метода зарегистрированы через `IMPLEMENT_METHOD` с числом аргументов 1 и возвращают `Empty`.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetAbsolutePosition(arg0)` | arg0: Array (по ссылке, 3 элемента) | Empty | Записывает абсолютные координаты текущего узла в массив: `[0]` — X, `[1]` — Y, `[2]` — Z. В C++ вызывает `getAbsolutePosition()` и записывает `vector3D.X/Y/Z` как `COleVariant((double)...)` через `pVector3D->SetAt(...)`. Метод принудительно устанавливает размер массива в 3 (`SetSize(3)`). |
| `GetAbsoluteRotation(arg0)` | arg0: Array (по ссылке, 3 элемента) | Empty | Записывает абсолютные углы вращения текущего узла в градусах в массив: `[0]` — X, `[1]` — Y, `[2]` — Z. В C++ вызывает `getAbsoluteRotation()` и записывает значения аналогично `GetAbsolutePosition`. |

Методы получают объект `CIArray` из `args[0]` через `GetObject<CIArray>(args[0])->GetData()`, изменяют размер массива на 3 элемента (`SetSize(3)`) и записывают вещественные значения. Перед вызовом необходимо создать объект `Array` (через `is_array()`) и передать его по ссылке. Значения корректны только после вызова `Recalc()`.

## Примеры

Построение иерархии преобразований с двумя дочерними узлами и получение абсолютных координат:

```EME-L
'Создать корневой узел преобразований в языке EME-L'
objRoot = Object("IrrTransformation");

'Задать смещение корневого узла'
objRoot.SetPosition(10.0, 0.0, 0.0);

'Создать дочерний узел и задать его локальное смещение'
objChild = objRoot.AddChild();
objChild.SetPosition(0.0, 5.0, 0.0);

'Пересчитать абсолютные преобразования всей иерархии'
objRoot.Recalc();

'Получить абсолютные координаты дочернего узла'
arrCoords = is_array();
objChild.GetAbsolutePosition(arrCoords);
'arrCoords[0] = 10.0, arrCoords[1] = 5.0, arrCoords[2] = 0.0'
```

Задание масштаба и вращения с последующим пересчётом:

```EME-L
'Создать узел и задать локальные преобразования'
objTrans = Object("IrrTransformation");
objTrans.SetScale(2.0, 2.0, 2.0);
objTrans.SetRotation(0.0, 45.0, 0.0);
objTrans.SetPosition(1.0, 0.0, 1.0);

'Пересчитать абсолютные значения'
objTrans.Recalc();

'Получить абсолютные углы поворота'
arrRot = is_array();
objTrans.GetAbsoluteRotation(arrRot);
'arrRot содержит абсолютные углы в градусах'
```

### Реальный пример — позиционирование в WMS-сцене

В прикладных скриптах 3D-WMS модуля системы EME.WMS (`Wms3DRackComponent.txt`, `Wms3DStaffComponent.txt`) класс `IrrTransformation` используется для построения двухуровневой иерархии точек привязки камеры: корневой узел задаёт координаты сектора, дочерний — точку обзора. Это рабочий паттерн применения класса в реальном коде.

```EME-L
'Корневой узел — позиция сектора на плане склада'
Transformation = Object("IrrTransformation");
SectorPoint = Transformation.AddChild();
SectorPoint.SetRotation(0.0, r_Sector.GetAngle() + 180.0, 0.0);
SectorPoint.SetPosition(r_Sector.GetX(), 0.0, -r_Sector.GetY());

'Дочерний узел — точка камеры/курсора внутри сектора'
CursorPoint = SectorPoint.AddChild();
CursorPoint.SetRotation(0.0, 0.0, 0.0);

'Пересчёт абсолютных координат по всей иерархии в языке EME-L'
Transformation.Recalc();

'Чтение абсолютных координат и углов дочернего узла'
Rotation = is_array();
Position = is_array();
CursorPoint.GetAbsoluteRotation(Rotation);
CursorPoint.GetAbsolutePosition(Position);
'Position[0..2] — абсолютные X/Y/Z в сцене;'
'Rotation[0..2] — абсолютные углы поворота в градусах'
```

## См. также

- Класс `IrrNode` — базовый класс узла сцены Irrlicht; `IrrTransformation` не наследует его, но решает схожую задачу управления преобразованиями.
- Класс `IrrDevice` — устройство Irrlicht; создаёт корневой узел сцены, к которому подключаются специализированные узлы.
- Класс `Array` — объект `Array` используется для получения абсолютных координат и углов методами `GetAbsolutePosition` и `GetAbsoluteRotation` (создаётся через `is_array()`).
