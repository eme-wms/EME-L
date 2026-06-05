# Транзакции сервера и RPC

> Synonyms for search: транзакции сервера, RPC, is_server_transaction_dump, is_server_transaction_data, is_server_emel, is_server_first_transaction, is_last_rpc_date, is_last_rpc_time, is_last_rpc_number, is_reset_last_rpc_info, is_process_rpc_over_file_system, is_OLE_serverbusydelay, is_type_version, дамп транзакций, серверный EME-L


### is_server_transaction_dump — дамп транзакций

на сервере делает дамп транзакций в объект запроса. На рабочей станции подготавливает запрос.

| Параметр | Тип | Обязательный | Описание |
|----------|-----|-------------|----------|
| args[0] | Object(Query) | Да | Пустой запрос для приёма дампа |
| args[1] | Any | Нет | Имя/номер записи (NULL_RECORD — все) |
| args[2] | Any | Нет | Имя/номер поля (NULL_FIELD — все) |
| args[3] | Any | Нет | Номер строки (NULL_REF — все) |
| args[4] | Integer | Нет | Номер транзакции с (0 — все) |
| args[5] | Integer | Нет | Номер транзакции по |
| args[6] | String | Нет | Данные для поиска |
| args[7] | DateTime | Нет | Дата начала |
| args[8] | DateTime | Нет | Дата окончания |
| args[9] | DateTime | Нет | Время начала |
| args[10] | DateTime | Нет | Время окончания |
| args[11] | Boolean | Нет | Не показывать служебную информацию |
| args[12] | String (ByRef) | Нет | Строка для возврата истории поиска (по RefOf) |

**Возвращает**: `Boolean` — TRUE при успехе.

```EME-L
q = Object("Query")
result = is_server_transaction_dump(q, NULL_RECORD, NULL_FIELD, NULL_REF)
```

---

### is_server_transaction_data — данные транзакции

на сервере возвращает данные одной команды транзакции. На рабочей станции не делает ничего.

| Параметр | Тип | Обязательный | Описание |
|----------|-----|-------------|----------|
| args[0] | Integer | Да | Номер транзакции |
| args[1] | Integer | Да | Номер команды в транзакции |
| args[2] | Integer | Да | Номер строки записи |

**Возвращает**: `String` — данные команды или пустое значение.

```EME-L
data = is_server_transaction_data(trans_no, cmd_no)
```

---

### is_server_first_transaction — границы транзакций

на сервере возвращает номера первой и последней транзакции в хранилище, а также дату и время первой транзакции.

| Параметр | Тип | Обязательный | Описание |
|----------|-----|-------------|----------|
| args[0] | Integer (ByRef) | Да | Ссылка на переменную для номера первой транзакции |
| args[1] | Integer (ByRef) | Да | Ссылка на переменную для номера последней транзакции |
| args[2] | DateTime (ByRef) | Да | Ссылка на переменную для даты первой транзакции |
| args[3] | DateTime (ByRef) | Да | Ссылка на переменную для времени первой транзакции |

**Возвращает**: `Boolean` — TRUE если транзакции есть.

```EME-L
first = 0
last = 0
' Получить границы транзакций на сервере (TRUE — транзакции есть) '
result = is_server_first_transaction(first, last)
```

---

### is_server_emel — вызов серверного EME-L

вызывает на сервере метод Resident::OnServerEmel и передаёт параметры.

| Параметр | Тип | Обязательный | Описание |
|----------|-----|-------------|----------|
| args[0] | Integer | Да | Номер команды |
| args[1] | Object(Parameters) | Да | Объект класса "Parameters" |

**Возвращает**: `String` — результат метода OnServerEmel.

```EME-L
Params = Object("Parameters")
Params.Put("key", "value")
result = is_server_emel(1, Params)
```

---

### is_transaction_no — номер транзакции

возвращает текущее значение номера транзакции.

**Параметры**: нет.

**Возвращает**: `Integer` — номер транзакции.

```EME-L
tno = is_transaction_no()
```

---

## RPC

### is_process_rpc_over_file_system — RPC через ФС

Обрабатывает удалённый вызов процедур (RPC) поверх файловой системы. Вызов FileSystem::RunRPC.

**Параметры**: нет.

**Возвращает**: `Boolean` — результат выполнения RPC.

```EME-L
is_process_rpc_over_file_system()
```

---

### is_last_rpc_date — дата последнего RPC

возвращает дату последнего RPC.

**Параметры**: нет.

**Возвращает**: `DateTime` — дата.

```EME-L
d = is_last_rpc_date()
```

---

### is_last_rpc_time — время последнего RPC

возвращает время последнего RPC.

**Параметры**: нет.

**Возвращает**: `DateTime` — время.

```EME-L
t = is_last_rpc_time()
```

---

### is_last_rpc_number — номер последнего RPC

возвращает номер последнего RPC.

**Параметры**: нет.

**Возвращает**: `Integer` — номер RPC.

```EME-L
n = is_last_rpc_number()
```

---

### is_reset_last_rpc_info — сброс информации RPC

сбрасывает информацию о последнем RPC.

**Параметры**: нет.

**Возвращает**: пустое значение.

```EME-L
is_reset_last_rpc_info()
```

---

