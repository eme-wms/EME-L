# Класс ChunkContainer

> Синонимы для поиска: CIChunkContainer ChunkContainer класс EME-L, контейнер чанков EME-L, Chunk-объекты EME-L, CObjectChunk PropChunk EME-L, ObjectChunk EME-L, CDataStorage EME-L, AddChunk RemoveChunk GetFirstChunk GetNextChunk Clear Load Save, сериализация модели склада EME-L, ModelOptions Warehouse EME-L, параметры склада EME-L, спецификация документа EME-L

Класс `ChunkContainer` в языке EME-L — это контейнер для работы с Chunk-объектами (`ObjectChunk`, `PropChunk`), импортируемыми напрямую из ядра системы EME.WMS. Класс ChunkContainer в языке EME-L сохраняется в полях типа `CDataStorage` и используется для хранения структурированных данных модели (например, параметров склада, спецификаций документа) вне явных полей записи базы данных.

Объект класса `ChunkContainer` в языке EME-L создаётся через `Object()` и не привязан к контексту диалога или браузера — это самостоятельный класс для управления набором Chunk-объектов. Сохранение и загрузка контейнера в системе EME.WMS возможны как через объект `CDataStorage`, так и напрямую из поля базы данных по тройке «запись / поле / строка».

## Создание объекта класса ChunkContainer

```EME-L
objChunk = Object("ChunkContainer");
```

Конструктор без аргументов класса `ChunkContainer` в языке EME-L создаёт пустой контейнер. Chunk-объекты добавляются методом `AddChunk`, а загрузка предварительно сохранённого контейнера выполняется методами `Load`. В системе EME.WMS пустой контейнер обычно используется как временное хранилище до вызова `Load` или до наполнения через `AddChunk`.

## Управление содержимым

Методы класса `ChunkContainer` в языке EME-L для изменения внутреннего набора Chunk-объектов. После `AddChunk` возвращённая ссылка на `CObjectChunk` остаётся валидной до вызова `Clear` или `RemoveChunk` для этого объекта.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `Clear()` | — | Empty | Очищает контейнер, удаляя все Chunk-объекты из внутреннего хранилища. После вызова контейнер возвращается в пустое состояние. |
| `AddChunk()` | — | CObjectChunk (ссылка) | Создаёт новый объектный чанк (`CObjectChunk`) и добавляет его в контейнер. Возвращает ссылку на созданный объект. |
| `RemoveChunk(chunk)` | chunk: CObjectChunk (ссылка) | Integer | Удаляет указанный объектный чанк. 1 — объект успешно удалён, 0 — удаление не выполнено (например, объект не принадлежит контейнеру). |

## Перебор Chunk-объектов

Методы `GetFirstChunk` и `GetNextChunk` класса `ChunkContainer` в языке EME-L совместно используют внутренний итератор контейнера. После вызова `GetFirstChunk` итератор устанавливается на начало коллекции; последующие вызовы `GetNextChunk` продвигают его вперёд. При достижении конца коллекции возвращается `NULL` — это обязательный сигнал для завершения цикла перебора в системе EME.WMS.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetFirstChunk()` | — | CObjectChunk (ссылка) или NULL | Первый объектный чанк; устанавливает итератор на начало коллекции. NULL — контейнер пуст, объектов нет. |
| `GetNextChunk()` | — | CObjectChunk (ссылка) или NULL | Следующий объектный чанк, продвигая внутренний итератор вперёд. NULL — достигнут конец коллекции, больше объектов нет. |

Идиоматический перебор всех чанков в контейнере в языке EME-L — это цикл `While` с проверкой на `NULL`, начинающийся с `GetFirstChunk()` и продолжающийся вызовами `GetNextChunk()` до получения `NULL`.

## Загрузка и сохранение

Для загрузки и сохранения класса `ChunkContainer` в языке EME-L предусмотрены перегруженные формы: работа напрямую с базой (по записи, полю и строке) и работа через объект `CDataStorage`. Внутренне форма `Load(storage)` используется формой `Load(record, field, line)` после чтения поля в `CDataStorage`; аналогично для `Save`. Это разделение важно в системе EME.WMS при отладке потоков данных между ядром и скриптом.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `Load(record, field, line)` | record: String, field: String, line: Integer | Boolean | Загружает контейнер из поля БД по указанной записи, полю и строке. TRUE — данные успешно загружены, FALSE — ошибка при загрузке. |
| `Load(storage)` | storage: CDataStorage (ссылка) | Boolean | Загружает контейнер из объекта `CDataStorage`, содержащего сериализованные данные. TRUE — успех, FALSE — ошибка. |
| `Save(record, field, line)` | record: String, field: String, line: Integer | Boolean | Сохраняет контейнер в поле БД по указанной записи, полю и строке. TRUE — данные успешно сохранены, FALSE — ошибка при сохранении. |

> Метода `Save(storage)` в классе `ChunkContainer` не зарегистрировано — сохранение в `CDataStorage` выполняется внутренней реализацией `Save(record, field, line)` и недоступно из EME-L напрямую. Для получения сериализованного представления контейнера во внешнее хранилище используйте запись в поле БД, а затем чтение этого поля через класс `CEMEData`.

## Примеры

Создание контейнера, наполнение и сохранение в поле записи (синтезированный пример для иллюстрации жизненного цикла):

```EME-L
objChunk = Object("ChunkContainer");
objDataChunk = objChunk.AddChunk();
objChunk.Save("Warehouse", "ModelOptions", wareRef);
```

Реальный пример из системы EME.WMS — чтение параметров модели склада из поля `ModelOptions` записи `Warehouse` класса `Visualisation3D` с последующим перебором ObjectChunk-веток (`CommonWrhOptions`, `Sectors`); синтаксис EME-L адаптирован из исходного кода, имена переменных сохранены:

```EME-L
dbWare = Object("dsDB", "Warehouse");
dbWare.SetLine(wareRef);
Chunk = Object("ChunkContainer");
Chunk.Load(dbWare.GetModelOptions());
If (Chunk == NULL)
    Return;
End If

objChunk = Chunk.GetFirstChunk();
If (objChunk == NULL)
    Return;
End If

secChunk = objChunk.GetFirstObject();
While (secChunk != NULL)
    Prop = secChunk.GetFirstProperty();
    For (Prop = secChunk.GetFirstProperty(); Prop != NULL; Prop = secChunk.GetNextProperty())
        pName = Prop.GetName();
        pData = Prop.GetData();
    End For
    secChunk = objChunk.GetNextObject();
End While
```

В системе EME.WMS приведённый паттерн используется классами трёхмерной визуализации (`Visualisation3D`, `VizExPlan`) и классом конфигурации сообщений EAN (`Vendors`) — каждый создаёт `Object("ChunkContainer")` и читает структурированные опции модели из поля записи без явной десериализации в коде скрипта.

## См. также

- [Класс Archive](./Archive.md) — бинарная сериализация объектов в Base64 в языке EME-L; `Archive.PutRPC` принимает `DataStorage` того же формата, который используется для `Load(storage)`/`Save(record, field, line)` класса ChunkContainer.
