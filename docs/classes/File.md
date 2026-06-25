# Класс File

> Синонимы для поиска: CIFile File класс EME-L, файловые операции EME-L, GetData PutData ReadLine PutEndFileData GetDataFromBegin GetDataFromEnd FileToArchive FileExtractFromArchive SetFilePath GetFilePath GetFileSize IsExist CreateFile FillStringArrayFromFile PutDataAsUTF8 ReadExif Rename Delete SetFileExt SetDir, CDataStorage EME-L, архивация EME-L, EXIF JPEG EME-L

Класс `File` в языке EME-L инкапсулирует доступ к файлам на диске. Класс File в языке EME-L предоставляет методы для чтения и записи файлов, управления компонентами пути, получения метаданных, выполнения файловых операций (переименование, удаление, создание каталогов, архивация) и чтения EXIF из изображений.

Объект класса `File` в языке EME-L создаётся через `Object()` и не привязан к контексту диалога или браузера — это самостоятельный класс для работы с файловой системой.

## Создание объекта класса File

```EME-L
objFile = Object("File");
objFile = Object("File", "D:\\Data\\report.txt");
```

Конструктор без аргументов создаёт объект файла без пути — путь к файлу задаётся позже методом `SetFilePath`.

## Компоненты пути

Методы класса File в языке EME-L работают с полным путём, хранящимся в объекте, и его частями.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetFilePath()` | — | String | Полный путь к файлу. |
| `SetFilePath(arg0)` | arg0: String | Empty | Устанавливает полный путь. |
| `GetDir()` | — | String | Директория без имени файла и завершающего слеша. |
| `SetDir(arg0)` | arg0: String | Empty | Заменяет директорию файла. |
| `GetFileExt()` | — | String | Расширение без точки. |
| `SetFileExt(arg0)` | arg0: String | Empty | Заменяет расширение. |
| `GetFileTitle()` | — | String | Имя файла без расширения. |
| `SetFileTitle(arg0)` | arg0: String | Empty | Заменяет имя без расширения. |
| `GetFileName()` | — | String | Имя файла с расширением. |
| `SetFileName(arg0)` | arg0: String | Empty | Заменяет имя файла. В текущей реализации меняет часть имени без расширения (эквивалент `SetFileTitle`), сохраняя прежнее расширение — для смены расширения используйте `SetFileExt`. |
| `GetDiskLetter()` | — | String | Буква диска из пути. |

## Чтение данных

Методы `GetData`, `GetDataFromBegin`, `GetDataFromEnd`, `ReadLine` класса File в языке EME-L возвращают объект `CDataStorage` — бинарное хранилище. Чтобы получить строку, вызовите `.AsString()` у результата, например `objFile.GetData().AsString()`.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetData()` | — | CDataStorage | Читает файл целиком. |
| `GetDataFromBegin(arg0)` | arg0: Integer | CDataStorage | N байт с начала файла; если N больше размера — весь файл. |
| `GetDataFromBegin(arg0, arg1)` | arg0: Integer (смещение), arg1: Integer (байт) | CDataStorage | N байт с позиции от начала; до конца файла, если превышает остаток. |
| `GetDataFromEnd(arg0)` | arg0: Integer | CDataStorage | N байт с конца файла; если N больше размера — весь файл. |
| `ReadLine()` | — | CDataStorage или Empty | Следующая строка. Разделитель по умолчанию CR+LF, буфер 1024 байта. Empty — достигнут конец файла. |
| `ReadLine(arg0)` | arg0: String (разделитель) | CDataStorage или Empty | Следующая строка с заданным разделителем. |
| `ReadLine(arg0, arg1)` | arg0: String (разделитель), arg1: Integer (буфер) | CDataStorage или Empty | То же с заданным размером буфера. |
| `FillStringArrayFromFile(arg0, arg1)` | arg0: Array (по ссылке), arg1: String (разделитель) | String | Заполняет массив строками по разделителю. Длина строки ≤ 64 КБ. Пустая строка — успех, иначе текст ошибки. |

`ReadLine` хранит внутреннюю позицию чтения — повторные вызовы последовательно читают строки файла класса File в языке EME-L. При достижении конца файла возвращается `Empty`.

## Запись данных

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `PutData(arg0)` | arg0: CDataStorage / массив байт / String | Integer | Перезаписывает файл. Возвращает количество записанных байт. |
| `PutData(arg0, arg1)` | arg0: данные, arg1: Boolean | Integer | То же; arg1=TRUE разрешает чтение файла другими процессами во время записи, FALSE (по умолчанию) — запрещает. |
| `PutDataFromBegin(arg0, arg1)` | arg0: String/CDataStorage, arg1: Integer (позиция) | Empty | Записывает с указанной позиции, обрезая файл до конца записанных данных. 0 — начало файла. |
| `PutEndFileData(arg0)` | arg0: String/CDataStorage | Empty | Добавляет данные в конец файла; создаёт файл, если его нет. |
| `PutEndFileData(arg0, arg1)` | arg0: данные, arg1: Boolean | Empty | То же с флагом совместного чтения (как в PutData). |
| `PutDataAsUTF8(arg0)` | arg0: String | Integer | Сохраняет текст в UTF-8 с BOM (конвертация из системной ANSI). 0 — успех, иначе код системной ошибки. |

## Метаданные и проверки

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetFileSize()` | — | Integer | Размер файла в байтах. Поддерживает файлы больше 2 ГБ. 0, если файл недоступен. |
| `GetSize()` | — | Integer | Размер файла. **Устаревший**: ограничение 2 ГБ — используйте `GetFileSize`. |
| `GetCreatedDateTime()` | — | DateTime | Дата и время создания. |
| `GetLastModifiedDateTime()` | — | DateTime | Дата и время последней модификации. |
| `GetLastAccessedDateTime()` | — | DateTime | Дата и время последнего доступа. |
| `GetDateTimeStamp()` | — | Integer | Числовой штамп на основе времени модификации. 0, если файл не существует. |
| `IsExist()` | — | Boolean | TRUE — файл или каталог существует. |
| `IsFilePathExists()` | — | Boolean | TRUE — путь существует. Синоним `IsExist`, реализован через другой системный вызов. |
| `IsReadWritePermission()` | — | Boolean | TRUE — есть права на чтение и запись. |

## Операции с файлом

Методы возвращают код системной ошибки (0 — успех), если не указано иное.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `Rename(arg0)` | arg0: String (новое полное имя) | Integer | Переименовывает или перемещает файл; при успехе обновляет путь объекта. |
| `Delete()` | — | Integer | Удаляет файл либо пустой каталог (по типу пути). |
| `CreateFile()` | — | Boolean | Создаёт файл и все промежуточные каталоги. Если имени файла нет в пути — создаются только каталоги. |
| `FileToArchive()` | — | Integer | Архивирует файл из текущего пути; архив рядом с исходником. 1 — успех, 0 — ошибка. |
| `FileToArchive(arg0)` | arg0: String (архив) | Integer | Архивирует файл из текущего пути в указанный архив. |
| `FileToArchive(arg0, arg1)` | arg0: String (исходник), arg1: String (архив) | Integer | Явные пути исходника и архива. |
| `FileToArchive(arg0, arg1, arg2)` | + arg2: Integer (таймаут, мс) | Integer | То же с таймаутом. |
| `FileToArchive(arg0, arg1, arg2, arg3)` | + arg3: String (ключ) | Integer | То же с ключом шифрования (зарезервирован для будущего использования). |
| `FileExtractFromArchive()` | — | Integer | Извлекает архив из текущего пути в его директорию. 1 — успех, 0 — ошибка. |
| `FileExtractFromArchive(arg0)` | arg0: String (директория) | Integer | Извлекает архив из текущего пути в указанную директорию. |
| `FileExtractFromArchive(arg0, arg1)` | arg0: String (архив), arg1: String (директория) | Integer | Явные пути архива и директории. |
| `FileExtractFromArchive(arg0, arg1, arg2)` | + arg2: Integer (таймаут, мс) | Integer | То же с таймаутом. |

## Прочее

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetLastErrorText()` | — | String | Текст последней системной ошибки; пустая строка, если ошибок не было. |
| `ReadExif(arg0)` | arg0: CIParameters (по ссылке) | Boolean | Читает EXIF из JPEG и сохраняет `TAG_DATETIME_ORIGINAL` (и GPS-теги при поддержке) в объект Parameters. |

## Примеры

Чтение всего файла в строку — метод GetData класса File в языке EME-L возвращает объект CDataStorage, метод AsString преобразует его в строку:

```EME-L
objFile = Object("File", "D:\\Data\\config.json");
If (objFile.IsExist())
    Content = objFile.GetData().AsString();
End If
```

Создание файла и запись данных методом PutEndFileData класса File в языке EME-L:

```EME-L
objFile = Object("File", "D:\\Data\\report.json");
objFile.CreateFile();
objFile.PutEndFileData(Content);
```

Изменение компонентов пути объекта класса File в языке EME-L:

```EME-L
objFile = Object("File", "D:\\Data\\report.txt");
objFile.SetFileExt("bak");
objFile.SetDir("D:\\Backup");
NewPath = objFile.GetFilePath();
```

Размер файла и архивация:

```EME-L
If (objFile.IsExist())
    Size = objFile.GetFileSize();
End If
objFile.FileToArchive("D:\\Data\\report.txt", "D:\\Data\\report.rar", 60000);
```

## См. также

- [Файловые функции](../system-functions/filesystem.md) — бесклассовые функции `is_path_to_file_ext`, `is_path_to_dir` и др. в языке EME-L
