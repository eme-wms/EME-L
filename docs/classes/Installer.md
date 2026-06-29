# Класс Installer

> Синонимы для поиска: CIInstaller Installer класс EME-L, генератор инсталляционного скрипта NSIS EME-L, NSIS-скрипт установщика EME DB, Nullsoft Scriptable Install System EME-L, Object("Installer"), Generate EME-L, создание установщика EME.WMS, OutFile Postfix BrandingText HeaderBitmap WelcomeBitmap InstallerIcon LicenseFile SupportMail WebSite ProjectName MajorVersion MinorVersion KernelFolder MetaprojFolder DatFolder HashdbFolder PDTEmulatorFile DocFolder ManualFolder SenselockFolder TPLFolder DashboardsFolder VideoFolder PostgreSQLInstaller PSQLODBCInstaller JDKInstaller TaskMonitorJavaFolder TaskMonitorWebFolder UsingProgramModulesFlag UsingStandardFolderFlag TrialVersionFlag DatHttpShortcutFlag Server Workstation WorkstationDataslice CurrentSection SectionName OutPath ServerShortCut WsShortCut MonoShortCut SetOutFile SetPostfix SetBrandingText SetHeaderBitmap SetKernelFolder SetMetaprojFolder SetDatFolder SetServer SetWorkstation SetGenerate nsi установщик EME.WMS сборка дистрибутива

Класс `Installer` в языке EME-L — генератор инсталляционного скрипта EME DB для Nullsoft Scriptable Install System (NSIS). Класс `Installer` в системе EME.WMS формирует `.nsi`-скрипт установщика на основе заданных свойств: папок с файлами ядра, метапроекта и данных, флагов включения серверных служб и рабочих станций, путей к дистрибутивам PostgreSQL, PSQLODBC, JDK, драйвера Senselock, порталов 3PL и Дашбордов, а также параметров оформления (картинки, иконки, лицензионное соглашение).

Объект класса `Installer` в языке EME-L создаётся через `Object()` без аргументов, после чего конфигурируется установкой свойств и завершается вызовом `Generate()`. Метод `Generate()` записывает готовый NSIS-скрипт в файл `<OutFile>.nsi` в текущей рабочей директории. Класс используется в скриптах сборки дистрибутивов EME.WMS и не привязан к контексту диалога или браузера — это самостоятельный класс для генерации установщика.

## Создание объекта класса Installer

Класс `Installer` в языке EME-L имеет один конструктор — без аргументов. Все параметры установщика задаются через свойства после создания объекта.

```EME-L
'Создать генератор инсталляционного скрипта NSIS'
nsi = Object("Installer");
```

Конструктор класса `Installer` в языке EME-L инициализирует параметры значениями по умолчанию: одна секция данных (`CurrentSection = 0`, массив `m_data` имеет размер 1), `UsingProgramModulesFlag = TRUE`, `UsingStandardFolderFlag = TRUE`, `TrialVersionFlag = FALSE`, `DatHttpShortcutFlag = FALSE`, `Server = FALSE`, `Workstation = FALSE`, `WorkstationDataslice = FALSE`, `MajorVersion = 0`, `MinorVersion = 0`. Конструктор также перечисляет иконки EME, Mono, Server, Internet Explorer и почты из ресурсов модуля `db.dll` для последующего использования в ярлыках установщика.

## Свойства файлов и оформления класса Installer

Свойства класса `Installer` в языке EME-L, задающие имена выходных файлов, текст торговой марки, графические ресурсы и лицензионное соглашение. Большинство свойств имеют значения по умолчанию, подставляемые при вызове `Generate()`, если свойство не задано. В языке EME-L свойства доступны как через сеттер (`SetOutFile`), так и через геттер (`GetOutFile`).

| Свойство | Тип | Описание |
|----------|-----|----------|
| `OutFile` | String | Имя файлов инсталлятора (`<OutFile>.nsi`, `<OutFile>.exe`). По умолчанию при `Generate()` формируется из идентификатора проекта, старшей и младшей версий и постфикса. |
| `Postfix` | String | Окончание имён папок и файлов (чтобы различать чистые и демо данные). |
| `BrandingText` | String | Текст торговой марки. По умолчанию при `Generate()` — `ООО "EME"`. |
| `HeaderBitmap` | String | Файл картинки заголовка страницы инсталлятора. По умолчанию — `InstallerHeader.bmp` из папки ресурсов ядра (для версии ядра ниже 52.0). |
| `HeaderUnBitmap` | String | Файл картинки заголовка страницы деинсталлятора. |
| `WelcomeBitmap` | String | Файл картинки начальной страницы инсталлятора. По умолчанию — `InstallerWelcome.bmp` из папки ресурсов ядра (для версии ядра ниже 52.0). |
| `WelcomeUnBitmap` | String | Файл картинки начальной страницы деинсталлятора. |
| `InstallerIcon` | String | Файл иконки установщика. По умолчанию — `InstallerIcon.ico` из папки ресурсов ядра (для версии ядра ниже 52.0). |
| `UninstallerIcon` | String | Файл иконки деинсталлятора. |
| `LicenseFile` | String | Файл с текстом лицензионного соглашения. По умолчанию — `LicenseAgreement.rtf` из папки ресурсов ядра (для версии ядра ниже 52.0). |
| `SupportMail` | String | Адрес почты службы технической поддержки. По умолчанию при `Generate()` — `eme@eme.ru`. |
| `WebSite` | String | Адрес веб-сайта. |
| `ProjectName` | String | Имя проекта (отображается в инсталляторе). Если не задано, формируется из идентификатора проекта как `EME.<identifier>`. |
| `MajorVersion` | Integer | Старшая версия проекта. Если 0 при вызове `Generate()`, пытается получить значение из EME-L класса `ProjectVersion`; иначе устанавливается 1. |
| `MinorVersion` | Integer | Младшая версия проекта. Если 0 при вызове `Generate()`, пытается получить значение из EME-L класса `ProjectVersion`; иначе устанавливается 0. |

## Свойства папок и путей класса Installer

Свойства класса `Installer` в языке EME-L, задающие пути к папкам с файлами ядра, метапроекта, данными базы, документацией, видеороликами, дистрибутивами и веб-порталами. Если свойство папки не задано (пустая строка), соответствующая секция не включается в инсталлятор.

| Свойство | Тип | Описание |
|----------|-----|----------|
| `KernelFolder` | String | Папка с файлами ядра. |
| `MetaprojFolder` | String | Папка с файлами метапроекта. |
| `DatFolder` | String | Папка с данными основной базы. |
| `HashdbFolder` | String | Папка с ХЭШБД. Если не задана, секция хэш-базы не включается. |
| `PDTEmulatorFile` | String | Файл с приложением для ТСД, собранным для эмулятора (используется в учебной станции). |
| `DocFolder` | String | Папка с документацией по функционалу и внедрению. |
| `ManualFolder` | String | Папка с руководством администратора системы и программиста. |
| `SenselockFolder` | String | Папка с дистрибутивом драйвера ключа Senselock. |
| `TPLFolder` | String | Папка с данными портала 3PL. |
| `DashboardsFolder` | String | Папка с данными портала Дашбордов. |
| `VideoFolder` | String | Папка с видеороликами (подбираются по формату `avi`). |
| `PostgreSQLInstaller` | String | Путь до инсталлятора PostgreSQL. |
| `PSQLODBCInstaller` | String | Путь до инсталлятора PSQLODBC. |
| `JDKInstaller` | String | Путь до инсталлятора JDK. |
| `TaskMonitorJavaFolder` | String | Папка с десктоп-монитором заданий Java. |
| `TaskMonitorWebFolder` | String | Папка с веб-порталом монитора заданий. |

## Свойства-флаги класса Installer

Свойства класса `Installer` в языке EME-L, включающие или отключающие секции инсталлятора и режимы генерации. В языке EME-L флаги принимают значения `TRUE` (-1) или `FALSE` (0).

| Свойство | Тип | По умолчанию | Описание |
|----------|-----|--------------|----------|
| `UsingProgramModulesFlag` | Boolean | TRUE | Флаг использования записи «Программные модули» для построения списка соответствующих файлов (ядра и метапроекта). |
| `UsingStandardFolderFlag` | Boolean | TRUE | Папки берутся из настроек БД, а не задаются пользователем. |
| `TrialVersionFlag` | Boolean | FALSE | Флаг ознакомительной версии. |
| `DatHttpShortcutFlag` | Boolean | FALSE | Флаг создания ярлыка запуска чистых данных в режиме HTTP-сервера (для работы с ТСД). |
| `Server` | Boolean | FALSE | Флаг создания серверных служб БД: `srv`, `http`, `bc`. Хэшбд устанавливается, если заполнена папка `HashdbFolder`, без дополнительного флага. |
| `Workstation` | Boolean | FALSE | Флаг создания папок и ярлыков рабочего места оператора: `netvm`, `fullvm`, монитор заданий Java. |
| `WorkstationDataslice` | Boolean | FALSE | Флаг создания учебной рабочей станции с монитором заданий Java `mono + dataslice`. |

## Свойства секций данных класса Installer

Свойства класса `Installer` в языке EME-L для управления секциями данных установщика. Установщик в системе EME.WMS поддерживает несколько секций данных (массив `m_data`), переключаемых через `CurrentSection`. Каждая секция имеет собственные имя, путь установки и ярлыки. Это позволяет создавать установщики с несколькими копиями базы (например, серверная и учебная).

| Свойство | Тип | Описание |
|----------|-----|----------|
| `CurrentSection` | Integer | Индекс текущей секции данных. При выходе за границы массива массив расширяется. |
| `SectionName` | String | Имя текущей секции данных (отображается в выборе компонентов установщика). |
| `OutPath` | String | Путь к установочным данным текущей секции (относительно `$INSTDIR`). |
| `ServerShortCut` | String | Параметры ярлыка сервера текущей секции. |
| `WsShortCut` | String | Параметры ярлыка рабочей станции текущей секции. |
| `MonoShortCut` | String | Параметры ярлыка `mono` текущей секции. |

## Метод Generate класса Installer

Метод `Generate` класса `Installer` в языке EME-L — единственный метод класса. Формирует NSIS-скрипт установщика на основе текущих свойств объекта и записывает его в файл `<OutFile>.nsi` в текущей рабочей директории.

| Метод | Аргументы | Возвращает | Описание |
|-------|-----------|------------|----------|
| `Generate()` | — | Empty | Формирует NSIS-скрипт и записывает его в файл `<OutFile>.nsi`. Возвращает пустой вариант. |

Метод `Generate()` класса `Installer` в языке EME-L определяет идентификатор проекта, формирует отображаемое имя программы, папку установки и ключи реестра. Если свойство `MajorVersion` равно 0, метод пытается получить старшую и младшую версии из EME-L класса `ProjectVersion` (при отсутствии метода `GetMajorVersion` или `GetMinorVersion` выводит сообщение об ошибке через `is_message`); иначе устанавливает версию 1.0. Для незаполненных свойств `OutFile`, `BrandingText`, `HeaderBitmap`, `HeaderUnBitmap`, `WelcomeBitmap`, `WelcomeUnBitmap`, `InstallerIcon` и `LicenseFile` подставляет значения по умолчанию.

В зависимости от флагов и папок объекта `Installer` метод `Generate()` формирует секции установки: ядро и метапроект, серверные службы (БД `srv`, HTTP-сервер `http`, мозговой компьютер `bc`, хэшбд), рабочие станции оператора и учебную станцию, документацию, руководство, видеоролики, драйвер Senselock, PostgreSQL, PSQLODBC, JDK, монитор заданий Java, веб-портал монитора заданий, портал 3PL и Дашборды, а также секцию деинсталляции. Установщик поддерживает типы установки: «Полная», «Сервер», «Рабочее место», «Веб-порталы», «Обучающая».

## Примеры работы с классом Installer

Создание базового скрипта установщика с ядром, метапроектом и серверными службами в языке EME-L:

```EME-L
nsi = Object("Installer");
nsi.SetUsingProgramModulesFlag(TRUE);
nsi.SetKernelFolder("C:\\Build\\kernel");
nsi.SetMetaprojFolder("C:\\Build\\metaproj");
nsi.SetDatFolder("C:\\Build\\dat");
nsi.SetServer(TRUE);
nsi.SetBrandingText("www.eme-wms.ru, ООО ~EME~ " + is_year(is_date()));
nsi.SetSupportMail("wms@eme.ru");
nsi.SetWebSite("www.eme-wms.ru");
nsi.Generate();
```

Создание установщика с порталом 3PL и инсталлятором PostgreSQL в языке EME-L:

```EME-L
nsi = Object("Installer");
nsi.SetUsingStandardFolderFlag(TRUE);
nsi.SetServer(TRUE);
nsi.SetPostgreSQLInstaller("C:\\Distrib\\postgresql-15.4.exe");
nsi.SetPSQLODBCInstaller("C:\\Distrib\\psqlodbc_x64.msi");
nsi.SetTPLFolder("C:\\Build\\TPLPortal");
nsi.SetJDKInstaller("C:\\Distrib\\jdk-17.exe");
nsi.SetTaskMonitorWebFolder("C:\\Build\\TaskMonitorWeb");
nsi.SetDocFolder("C:\\Build\\doc");
nsi.SetLicenseFile("C:\\Resources\\LicenseAgreement.rtf");
nsi.Generate();
```

## См. также

- [Класс File](./File.md) — работа с файловой системой в языке EME-L, используется для построения путей к ресурсам установщика.
- [Класс Archive](./Archive.md) — класс бинарной сериализации в языке EME-L, может применяться для упаковки данных дистрибутива.
