CITranslator Translator класс EME-L локализация перевод интерфейсных текстов Локализация tr TR макрос Db.RUS dbassist.RUS браузер диалог отчёт настройки Localization.txt SetTraceFile SetChangeCode GiveMoot GiveMootFromFile GetTextFromTRCXX GetTextFromTRinFile GetTextFromTrEMEL GetTextFromTrQuery GetTextFromTrEMELClass TranslateAllSystemDialogs TranslateSystemDialog GetTextTranslateFromDialogTemplate GetTextTranslateFromBrowser PutTranslateTextToBrowser get_texts_for_translation_from_report GetTextFromDBRUS GetTextFromUserMenu GetTextFromField GetTextFromDBSettings GetTextFromModuleSettings GetTextFromDBNames ReloadTranslatedTexts SaveTranslatedTexts InsertCR

# Класс Translator

Класс `Translator` в языке EME-L — класс для локализации и перевода интерфейсных текстов проекта, хранящихся в базе данных, исходном коде C++, классах EME-L, шаблонах диалогов, браузерах, отчётах и настройках системы. В системе EME.WMS класс `Translator` предоставляет инструменты для сбора русскоязычных строк, подлежащих переводу, и записи их в справочную запись «Локализация», откуда переводы могут быть выгружены, отредактированы и загружены обратно в память приложения.

## Создание объекта Translator

В языке EME-L объект `Translator` создаётся без параметров. При создании устанавливается внутреннее имя переводящей функции `tr` и сбрасывается флаг изменения исходного кода (`isChangeCode = false`).

```EME-L
'Создание локализатора без параметров в EME-L'
translator = Object("Translator");
```

> В прикладных скриптах системы EME.WMS иногда встречаются вызовы `Object("Translator", "D:/translatorFull.log", "D:/translatorBrief.log")` с одним, двумя или тремя путями к файлам трассировки. Однако в C++-реестре `c_emel_class_method/i_Translator.cpp` зарегистрирован только конструктор без аргументов. Для настройки файлов трассировки в EME-L следует использовать метод `SetTraceFile` после создания объекта.

## Настройка трассировки и режима изменения кода

Методы класса `Translator` в языке EME-L для настройки трассировки и флага изменения исходных файлов.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `SetTraceFile(filePath)` | filePath: String | Empty | Задаёт полный путь к файлу полной трассировки работы объекта `Translator` в EME-L. |
| `SetTraceFile(fullTraceFilePath, briefTraceFilePath)` | fullTraceFilePath: String, briefTraceFilePath: String | Empty | Задаёт пути к файлам полной и краткой трассировки в EME-L. |
| `SetChangeCode(isChangeCode)` | isChangeCode: Boolean | Empty | Устанавливает флаг разрешения изменения исходных файлов. `true` — разрешить изменение исходных файлов, `false` — только собирать тексты для перевода. |

## Ручная обработка спорных текстов в EME-L

В языке EME-L класс `Translator` поддерживает ручную обработку спорных текстов в исходных файлах C++.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GiveMoot(directoryPath)` | directoryPath: String | Empty | Рекурсивно обходит директорию с файлами `.h`, `.cpp` и `.c` и для каждого файла вызывает `GiveMootFromFile`. |
| `GiveMootFromFile(filePath)` | filePath: String | Empty | Выполняет ручную обработку спорных текстов в указанном файле C++. |

## Сбор текстов для перевода из C++ и EME-L

В системе EME.WMS класс `Translator` собирает тексты для перевода из макросов `TR` в C++ и `tr` в EME-L.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetTextFromTRCXX(directoryPath)` | directoryPath: String | Empty | Рекурсивно собирает тексты для перевода из макросов `TR` во всех файлах `.h`, `.cpp` и `.c` в указанной директории. |
| `GetTextFromTRinFile(filePath)` | filePath: String | Empty | Собирает тексты для перевода из макросов `TR` в указанном файле C++. |
| `GetTextFromTrEMEL()` | — | Integer (0) | Собирает тексты для перевода из макросов `tr` во всех классах EME-L и запросах. |
| `GetTextFromTrQuery(queryRef)` | queryRef: Integer | Integer (0) | Собирает тексты для перевода из макросов `tr` в указанном запросе EME-L. |
| `GetTextFromTrEMELClass(classRef)` | classRef: Integer | Integer (0) | Собирает тексты для перевода из макросов `tr` в указанном классе EME-L. |

## Сбор текстов из шаблонов диалогов, браузеров и отчётов

В языке EME-L класс `Translator` извлекает переводимые строки из шаблонов диалогов, браузеров и отчётов системы EME.WMS.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `TranslateAllSystemDialogs(directoryPath)` | directoryPath: String | Empty | Рекурсивно обрабатывает ресурсные файлы диалогов `.rc` в указанной директории. |
| `TranslateSystemDialog(filePath)` | filePath: String | Empty | Обрабатывает один ресурсный файл диалога `.rc`. |
| `GetTextTranslateFromDialogTemplate(dialogTemplateRef)` | dialogTemplateRef: Integer | Empty | Собирает тексты для перевода из шаблона диалога (заголовок, органы управления, закладки, подсказки, контекстная справка, колонки таблиц, связанные отчёты). |
| `GetTextTranslateFromBrowser(browserRef)` | browserRef: Integer | Integer (0) | Собирает тексты для перевода из шаблона браузера. |
| `PutTranslateTextToBrowser(browserRef)` | browserRef: Integer | Integer (0) | Применяет переведённые тексты к шаблону браузера. |
| `get_texts_for_translation_from_report(reportRef)` | reportRef: Integer | Integer (0) | Собирает тексты для перевода из отчёта (статические ячейки и заголовки). |

## Системные сообщения, меню и настройки в Translator

В системе EME.WMS класс `Translator` выгружает системные сообщения, меню и настройки для перевода.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `GetTextFromDBRUS()` | — | Integer (0) | Выгружает системные сообщения из файлов `Db.RUS` и `dbassist.RUS` в запись «Локализация». |
| `GetTextFromUserMenu()` | — | Integer (0) | Выгружает названия пунктов меню администратора в запись «Локализация». |
| `GetTextFromField(fieldObj)` | fieldObj: CEMEFld (ссылка) | Empty | Выгружает содержимое текстового поля записи в запись «Локализация» для перевода. |
| `GetTextFromDBSettings()` | — | Empty | Выгружает настройки рабочей станции («Дерево настроек») в запись «Локализация». |
| `GetTextFromModuleSettings()` | — | Empty | Выгружает настройки модулей («Дерево настроек модулей») в запись «Локализация». |
| `GetTextFromDBNames()` | — | Empty | Выгружает названия объектов метаданных базы данных (атрибуты, поля, записи) в запись «Локализация». Доступен начиная с версии ядра 47.3. |

## Управление переводами в памяти

В языке EME-L класс `Translator` управляет переводами, загруженными в память приложения.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `ReloadTranslatedTexts()` | — | Empty | Перезагружает переводы в память из записи «Локализация». |
| `ReloadTranslatedTexts(languageId)` | languageId: String | Empty | Перезагружает переводы в память для указанного языка (например, `"RUS"` или `"ENG"`). |
| `SaveTranslatedTexts()` | — | Empty | Сохраняет переводы из памяти в файл `Localization.txt`. В сборках без `NEW_SEMAPHORE` метод является заглушкой. |

## Вспомогательные методы Translator

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `InsertCR(text)` | text: String | String | Заменяет одиночные символы перевода строки `LF` (`\n`) на последовательность `CRLF` (`\r\n`). |

## Примеры использования класса Translator в EME-L

Пример полного цикла локализации: сбор текстов из классов EME-L и сохранение в файл.

```EME-L
'Создать локализатор и переключить язык на ENG в EME-L'
translator = Object("Translator");
translator.ReloadTranslatedTexts("ENG");

'Собрать тексты из всех классов EME-L и запросов'
translator.GetTextFromTrEMEL();

'Сохранить переводы в Localization.txt для редактирования'
translator.SaveTranslatedTexts();

is_message("Локализация", "Тексты для перевода собраны", 1);
```

Пример сбора текстов из шаблонов диалогов в EME-L.

```EME-L
'Собрать тексты из всех шаблонов диалогов'
translator = Object("Translator");
translator.ReloadTranslatedTexts("ENG");

dbDialogTemplates = Object("dsDB", "DialogTemplates");
Loop (dbDialogTemplates)
    translator.GetTextTranslateFromDialogTemplate(dbDialogTemplates.GetLine());
End Loop

translator.SaveTranslatedTexts();

is_message("Локализация", "Тексты диалогов собраны", 1);
```

Пример сбора текстов из отчётов в EME-L.

```EME-L
'Собрать тексты из всех шаблонов отчётов'
translator = Object("Translator");
translator.ReloadTranslatedTexts("ENG");

dbReports = Object("dsDB", "Шаблоны отчетов");
Loop (dbReports)
    translator.get_texts_for_translation_from_report(dbReports.GetLine());
End Loop

translator.SaveTranslatedTexts();

is_message("Локализация", "Тексты отчётов собраны", 1);
```

Пример сбора текстов из полей записи в EME-L.

```EME-L
'Выгрузить текстовые поля записи для перевода'
translator = Object("Translator");
translator.ReloadTranslatedTexts("ENG");

r_ContextFindParams = Object("dsDB", "ContextFindParams");
translator.GetTextFromField(r_ContextFindParams.GetContextFld());
translator.GetTextFromField(r_ContextFindParams.GetObjectFld());
translator.GetTextFromField(r_ContextFindParams.GetRecordNameFld());
translator.GetTextFromField(r_ContextFindParams.GetFieldNameFld());

translator.SaveTranslatedTexts();

is_message("Локализация", "Тексты полей собраны", 1);
```

Пример настройки трассировки локализатора в EME-L.

```EME-L
'Настроить файлы трассировки перед сбором текстов'
translator = Object("Translator");
translator.SetTraceFile("D:/translatorFull.log", "D:/translatorBrief.log");

'Собрать текста из C++-исходников ядра'
translator.GetTextFromTRCXX("D:/dbtrunk/kernel.w95/Src/");
```

## См. также

- [Класс StringBuilder](StringBuilder.md) — построение строковых буферов в EME-L.
