CIFileFind FileFind поиск файлов файловая система CFileFind FindFile FindNextFile GetFileName GetFilePath IsDirectory IsDots MatchesMask Close FILE_ATTRIBUTE_READONLY FILE_ATTRIBUTE_HIDDEN FILE_ATTRIBUTE_SYSTEM FILE_ATTRIBUTE_DIRECTORY FILE_ATTRIBUTE_ARCHIVE FILE_ATTRIBUTE_NORMAL FILE_ATTRIBUTE_TEMPORARY FILE_ATTRIBUTE_COMPRESSED

# Класс FileFind

Класс `FileFind` в языке EME-L предназначен для поиска файлов и каталогов на локальной файловой системе. Соответствует C++ классу `CIFileFind` — инкапсулирует доступ к MFC `CFileFind`.

Класс предоставляет механизм последовательного перебора файлов и каталогов, соответствующих заданной маске имени. Поддерживает фильтрацию по атрибутам файла и проверку типа найденного объекта (файл, каталог, служебные метки `.` и `..`).

Основной сценарий использования в системе EME.WMS и прикладных скриптах EME-L: создание объекта, вызов `FindFile` с указанием маски, циклический вызов `FindNextFile` до исчерпания результатов, получение имени и пути через `GetFileName` и `GetFilePath`, проверка атрибутов через `IsDirectory`, `IsDots` и `MatchesMask`. По завершении работы рекомендуется вызывать `Close` для освобождения ресурсов.

## Создание объекта

```EME-L
'Создать объект FileFind для последовательного перебора файлов по маске
objFileFind = Object("FileFind");
```

В языке EME-L конструктор класса `FileFind` не принимает аргументов. После создания объекта для начала поиска необходимо вызвать метод `FindFile`, передав ему маску имени. До вызова `FindFile` методы `GetFileName`, `GetFilePath`, `IsDirectory`, `IsDots` и `MatchesMask` возвращают пустые значения или `FALSE`, так как текущий найденный объект отсутствует.

## Методы перебора файлов

В языке EME-L поиск файлов ведётся по маске, заданной в `FindFile`. Тот же объект `FileFind` используется на всех итерациях — повторно вызывать `FindFile` для перехода к следующему файлу не нужно.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `FindFile([mask])` | mask: String (необязательный) | Boolean | Начинает поиск файлов или каталогов по маске. Маска может содержать символы подстановки `*` и `?`, например `"C:\\*.txt"` или `"D:\\Data\\*.*"`. Если параметр опущен, передаётся пустая строка. `TRUE` — найден хотя бы один объект; `FALSE` — объекты отсутствуют. |
| `FindNextFile()` | — | Boolean | Переходит к следующему найденному файлу или каталогу для текущей маски. Должен вызываться после успешного `FindFile`. `TRUE` — следующий объект найден; `FALSE` — объекты закончились. |
| `Close()` | — | Empty | Завершает поиск и освобождает связанные системные ресурсы. Рекомендуется вызывать после окончания перебора. |

## Информация о текущем объекте

Методы `GetFileName` и `GetFilePath` возвращают сведения о текущем найденном файле или каталоге. Если поиск не был начат или объект не найден, возвращается пустая строка.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetFileName()` | — | String | Имя текущего найденного файла или каталога без пути. |
| `GetFilePath()` | — | String | Полный путь к текущему найденному файлу или каталогу, включая имя и расширение. |

## Проверки атрибутов

Эти методы позволяют отличить каталоги и служебные метки от обычных файлов в системе EME.WMS, а также проверить битовые атрибуты файла.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `IsDirectory()` | — | Boolean | `TRUE` — текущий объект является каталогом. `FALSE` — обычный файл или поиск не начат. |
| `IsDots()` | — | Boolean | `TRUE` — имя текущего объекта равно `.` или `..` (служебные метки текущего и родительского каталога). `FALSE` — не служебная метка или поиск не начат. |
| `MatchesMask(mask)` | mask: Integer | Boolean | Проверяет, соответствуют ли атрибуты текущего файла заданной битовой маске. Маска формируется побитовым ИЛИ констант `FILE_ATTRIBUTE_*`. |

### Константы файловых атрибутов для MatchesMask

В языке EME-L числовые значения констант совпадают с Windows API и передаются в `MatchesMask` целым числом.

| Константа | Значение | Описание |
|-----------|----------|----------|
| `FILE_ATTRIBUTE_READONLY` | `0x00000001` | Только для чтения. |
| `FILE_ATTRIBUTE_HIDDEN` | `0x00000002` | Скрытый файл. |
| `FILE_ATTRIBUTE_SYSTEM` | `0x00000004` | Системный файл. |
| `FILE_ATTRIBUTE_DIRECTORY` | `0x00000010` | Каталог. |
| `FILE_ATTRIBUTE_ARCHIVE` | `0x00000020` | Архивный. |
| `FILE_ATTRIBUTE_NORMAL` | `0x00000080` | Обычный файл. |
| `FILE_ATTRIBUTE_TEMPORARY` | `0x00000100` | Временный файл. |
| `FILE_ATTRIBUTE_COMPRESSED` | `0x00000800` | Сжатый файл. |

## Примеры

### Перебор текстовых файлов в каталоге

```EME-L
'#DEV_INITIALS 29.06.2026 EME company
Перебор текстовых файлов в каталоге и вывод их имён
objFileFind = Object("FileFind");
Found = objFileFind.FindFile("D:\\Data\\*.txt");
While (Found)
    If (Not objFileFind.IsDots())
        FileName = objFileFind.GetFileName();
        FilePath = objFileFind.GetFilePath();
    End If
    Found = objFileFind.FindNextFile();
End While
objFileFind.Close();
```

### Сбор файлов из каталога с исключением служебных меток

```EME-L
'#DEV_INITIALS 29.06.2026 EME company
Собрать все файлы из каталога логов в массив, пропуская . и ..
objFileFind = Object("FileFind");
arrFiles = Object("Array");
IsFind = objFileFind.FindFile(m_csLogDir + "*.*");
While (IsFind)
    If (Not objFileFind.IsDots())
        arrFiles.Add(objFileFind.GetFilePath());
    End If
    IsFind = objFileFind.FindNextFile();
End While
objFileFind.Close();
```

## См. также

- [Класс File](./File.md) — работа с отдельным файлом: чтение, запись, метаданные, операции в языке EME-L.
