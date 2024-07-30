# Содержание модуля psycopg2

Интерфейс модуля соответствует стандарту, определенному в [DB API 2.0](https://www.python.org/dev/peps/pep-0249/).

### psycopg2.connect(_dsn=None_, _connection\_factory=None_, _cursor\_factory=None_, _async=False_, _\*\*kwargs_)

Создать новый сеанс базы данных и вернуть новый объект [connection](https://www.psycopg.org/docs/connection.html#connection).

Параметры подключения можно указать как [строку подключения libpq](https://www.postgresql.org/docs/current/static/libpq-connect.html#LIBPQ-CONNSTRING) с помощью параметра dsn:

```python
conn = psycopg2.connect("dbname=test user=postgres password=secret")
```

или с использованием набора ключевых аргументов:

```python
conn = psycopg2.connect(dbname="test", user="postgres", password="secret")
```

или используя смесь обоих: если в обоих источниках указано одно и то же имя параметра, значение kwargs будет иметь приоритет над значением dsn. Обратите внимание, что требуется либо dsn, либо по крайней мере один аргумент ключевого слова, связанный с подключением.

Основные параметры подключения:

* dbname – имя базы данных (database – устаревший псевдоним)
* user – имя пользователя, используемое для аутентификации
* password – пароль, используемый для аутентификации
* host – адрес хоста базы данных (по умолчанию – сокет UNIX, если не указан)
* port – номер порта подключения (по умолчанию – 5432, если не указан)

Любой другой параметр соединения, поддерживаемый клиентской библиотекой/сервером, может быть передан либо в строке соединения, либо как ключевое слово. Документация PostgreSQL содержит полный [список поддерживаемых параметров](https://www.postgresql.org/docs/current/static/libpq-connect.html#LIBPQ-PARAMKEYWORDS). Также обратите внимание, что те же параметры могут быть переданы клиентской библиотеке с помощью [переменных среды](https://www.postgresql.org/docs/current/static/libpq-envars.html).

С помощью параметра connection\_factory можно указать другой класс или фабрику соединений. Это должен быть вызываемый объект, принимающий аргумент строки dsn. Подробнее см. в разделе [Фабрики соединений и курсоров](https://www.psycopg.org/docs/advanced.html#subclassing-connection). Если указан cursor\_factory, то [cursor\_factory](https://www.psycopg.org/docs/connection.html#connection.cursor\_factory) соединения устанавливается на него. Если вам нужны только настраиваемые курсоры, вы можете использовать этот параметр вместо создания подкласса соединения.

При использовании `async=True` будет создано асинхронное соединение: см. раздел [Поддержка асинхронности](https://www.psycopg.org/docs/advanced.html#async-support), чтобы узнать о преимуществах и ограничениях. async\_ — допустимый псевдоним для версии Python, где async — ключевое слово.

_Изменено в версии 2.4.3_: в соединение передается любой аргумент ключевого слова. Ранее в качестве ключевых слов поддерживались только основные параметры (плюс sslmode).

_Изменено в версии 2.5_: добавлен параметр cursor\_factory.

_Изменено в версии 2.7_: можно указывать как аргументы dsn, так и ключевые слова.

_Изменено в версии 2.7_: добавлен псевдоним async\_.

{% hint style="info" %}
Смотри также:

* parse\_dsn
* [синтаксис строки подключения](https://www.postgresql.org/docs/current/static/libpq-connect.html#LIBPQ-CONNSTRING) libpq
* поддерживаемые [параметры подключения](https://www.postgresql.org/docs/current/static/libpq-connect.html#LIBPQ-PARAMKEYWORDS) libpq
* поддерживаемые [переменные окружения](https://www.postgresql.org/docs/current/static/libpq-envars.html) libpq
{% endhint %}

{% hint style="info" %}
**Расширение DB API**

Не связанные с подключением ключевые параметры — это расширения Psycopg для [DB API  2.0](https://www.python.org/dev/peps/pep-0249/).
{% endhint %}

* psycopg2.apilevel - Строковая константа, указывающая поддерживаемый уровень DB API. Для [psycopg2](https://www.psycopg.org/docs/module.html#module-psycopg2) это 2.0.
* psycopg2.threadsafety - Целочисленная константа, указывающая уровень безопасности потоков, поддерживаемый интерфейсом. Для [psycopg2](https://www.psycopg.org/docs/module.html#module-psycopg2) это 2, т. е. потоки могут совместно использовать модуль и соединение. Подробнее см. [Безопасность потоков и процессов](https://www.psycopg.org/docs/usage.html#thread-safety).
* psycopg2.paramstyle - Строковая константа, указывающая тип форматирования маркера параметра, ожидаемого интерфейсом. Для [psycopg2](https://www.psycopg.org/docs/module.html#module-psycopg2) это pyformat. См. также [Передача параметров в запросы SQL](https://www.psycopg.org/docs/usage.html#query-parameters).
* psycopg2.**libpq\_version -** Целочисленная константа, указывающая версию библиотеки libpq, с которой был скомпилирован этот модуль psycopg2 (в том же формате [server\_version](https://www.psycopg.org/docs/extensions.html#psycopg2.extensions.ConnectionInfo.server\_version)). Если это значение больше или равно 90100, то вы можете запросить версию фактически загруженной библиотеки с помощью функции [libpq\_version()](https://www.psycopg.org/docs/extensions.html#psycopg2.extensions.libpq\_version).

## Исключения

В соответствии с [DB API 2.0](https://www.python.org/dev/peps/pep-0249/) модуль предоставляет информацию об ошибках через следующие исключения:

### exception psycopg2.Warning

Исключение, выдаваемое для важных предупреждений, таких как усечение данных при вставке и т. д. Это подкласс Python StandardError ([Exception ](https://docs.python.org/3/library/exceptions.html#Exception)в Python 3).

### exception psycopg2.Error

Исключение, которое является базовым классом всех других исключений ошибок. Вы можете использовать его для перехвата всех ошибок с помощью одного единственного оператора except. Предупреждения не считаются ошибками и, таким образом, не используют этот класс в качестве базового. Это подкласс Python StandardError ([Exception](https://docs.python.org/3/library/exceptions.html#Exception) в Python 3).&#x20;

* pgerror - Строка, представляющая сообщение об ошибке, возвращаемое бэкендом, None, если недоступно.
* pgcode - Строка, представляющая код ошибки, возвращаемый бэкендом, None, если недоступно. Модуль [errorcodes](https://www.psycopg.org/docs/errorcodes.html#module-psycopg2.errorcodes) содержит символические константы, представляющие коды ошибок PostgreSQL.

```python
>>> try:
...     cur.execute("SELECT * FROM barf")
... except psycopg2.Error as e:
...     pass

>>> e.pgcode
'42P01'
>>> print(e.pgerror)
ERROR:  relation "barf" does not exist
LINE 1: SELECT * FROM barf
                      ^
```

* cursor - Курсор, из которого было вызвано исключение; [None](https://docs.python.org/3/library/constants.html#None), если неприменимо.
* diag - Объект диагностики [Diagnostics](https://www.psycopg.org/docs/extensions.html#psycopg2.extensions.Diagnostics), содержащий дополнительную информацию об ошибке.

```python
>>> try:
...     cur.execute("SELECT * FROM barf")
... except psycopg2.Error as e:
...     pass

>>> e.diag.severity
'ERROR'
>>> e.diag.message_primary
'relation "barf" does not exist'
```

_Новое в версии 2.5_.

{% hint style="info" %}
**Расширение DB API**

Атрибуты [pgerror](https://www.psycopg.org/docs/module.html#psycopg2.Error.pgerror), [pgcode](https://www.psycopg.org/docs/module.html#psycopg2.Error.pgcode), [cursor](https://www.psycopg.org/docs/module.html#psycopg2.Error.cursor) и [diag](https://www.psycopg.org/docs/module.html#psycopg2.Error.diag) являются расширениями Psycopg.
{% endhint %}

### exception psycopg2.InterfaceError

Исключение, вызываемое для ошибок, связанных с интерфейсом базы данных, а не с самой базой данных. Это подкласс [Error](https://www.psycopg.org/docs/module.html#psycopg2.Error).

### exception psycopg2.DatabaseError

Исключение, вызываемое для ошибок, связанных с базой данных. Это подкласс [Error](https://www.psycopg.org/docs/module.html#psycopg2.Error).

### exception psycopg2.DataError

Исключение, вызываемое для ошибок, связанных с проблемами с обработанными данными, такими как деление на ноль, числовое значение вне диапазона и т. д. Это подкласс [DatabaseError](https://www.psycopg.org/docs/module.html#psycopg2.DatabaseError).

### exception psycopg2.OperationalError

Исключение, вызываемое для ошибок, связанных с работой базы данных и не обязательно находящихся под контролем программиста, например, происходит неожиданное отключение, имя источника данных не найдено, транзакция не может быть обработана, произошла ошибка выделения памяти во время обработки и т. д. Это подкласс [DatabaseError](https://www.psycopg.org/docs/module.html#psycopg2.DatabaseError).

### exception psycopg2.IntegrityError

Исключение, вызываемое при нарушении реляционной целостности базы данных, например, при сбое проверки внешнего ключа. Это подкласс [DatabaseError](https://www.psycopg.org/docs/module.html#psycopg2.DatabaseError).

### exception psycopg2.InternalError

Исключение, возникающее при возникновении внутренней ошибки базы данных, например, курсор больше не действителен, транзакция не синхронизирована и т. д. Это подкласс [DatabaseError](https://www.psycopg.org/docs/module.html#psycopg2.DatabaseError).

### exception psycopg2.ProgrammingError

Исключение, возникающее при ошибках программирования, например, таблица не найдена или уже существует, синтаксическая ошибка в операторе SQL, указано неверное количество параметров и т. д. Это подкласс [DatabaseError](https://www.psycopg.org/docs/module.html#psycopg2.DatabaseError).

### exception psycopg2.NotSupportedError

Исключение, возникающее в случае использования метода или API базы данных, которые не поддерживаются базой данных, например, запрос rollback() для соединения, которое не поддерживает транзакцию или имеет отключенные транзакции. Это подкласс [DatabaseError](https://www.psycopg.org/docs/module.html#psycopg2.DatabaseError).

{% hint style="info" %}
**Расширение DB API**

Psycopg на самом деле вызывает другое исключение для каждой ошибки SQLSTATE, возвращаемой базой данных: классы доступны в модуле [psycopg2.errors](https://www.psycopg.org/docs/errors.html#module-psycopg2.errors). Каждый класс исключения является подклассом одного из классов исключений, определенных здесь, поэтому их не нужно специально перехватывать: перехват Error или DatabaseError обычно необходим для написания универсального обработчика ошибок; перехват конкретной ошибки, такой как NotNullViolation, может быть полезен для написания конкретных обработчиков исключений.
{% endhint %}

Вот схема наследования исключений:

```bash
StandardError
|__ Warning
|__ Error
    |__ InterfaceError
    |__ DatabaseError
        |__ DataError
        |__ OperationalError
        |__ IntegrityError
        |__ InternalError
        |__ ProgrammingError
        |__ NotSupportedError
```

## Типы объектов и конструкторов

{% hint style="info" %}
Этот раздел в основном дословно скопирован из спецификации [DB API 2.0](https://www.python.org/dev/peps/pep-0249/). Хотя эти объекты представлены в соответствии с DB API, Psycopg предлагает очень точные инструменты для преобразования данных между форматами Python и PostgreSQL. См. [Адаптация новых типов Python к синтаксису SQL](https://www.psycopg.org/docs/advanced.html#adapting-new-types) и [Приведение типов SQL к объектам Python](https://www.psycopg.org/docs/advanced.html#type-casting-from-sql-to-python)
{% endhint %}

Многим базам данных необходимо иметь входные данные в определенном формате для привязки к входным параметрам операции. Например, если входные данные предназначены для столбца DATE, то они должны быть привязаны к базе данных в определенном строковом формате. Аналогичные проблемы существуют для столбцов "Row ID" или больших двоичных элементов (например, BLOB-объектов или столбцов RAW). Это создает проблемы для Python, поскольку параметры метода `.execute*()` не типизированы. Когда модуль базы данных видит строковый объект Python, он не знает, следует ли его привязывать как простой столбец CHAR, как необработанный элемент BINARY или как DATE.

Чтобы преодолеть эту проблему, модуль должен предоставить конструкторы, определенные ниже, для создания объектов, которые могут содержать специальные значения. При передаче методам курсора модуль может затем определить правильный тип входного параметра и привязать его соответствующим образом.

Атрибут description объекта курсора возвращает информацию о каждом из столбцов результата запроса. type\_code должен сравниваться с одним из объектов Type, определенных ниже. Типы Objects могут быть равны более чем одному коду типа (например, DATETIME может быть равен кодам типа для столбцов даты, времени и временной метки; подробности см. в Советах по реализации ниже).

Модуль экспортирует следующие конструкторы и синглтоны:

### psycopg2.Date(year, month, day)

Эта функция создает объект, содержащий значение даты.

### psycopg2.Time(hour, minute, second)

Эта функция создает объект, содержащий значение времени.

### psycopg2.Timestamp(year, month, day, hour, minute, second)

Эта функция создает объект, содержащий значение отметки времени.

### psycopg2.DateFromTicks(ticks)

Эта функция создает объект, содержащий значение даты, из заданного значения тактов (количество секунд с начала эпохи; подробности см. в документации стандартного модуля Python time).

### psycopg2.TimeFromTicks(ticks)

Эта функция создает объект, содержащий значение времени, из заданного значения тактов (количество секунд с начала эпохи; подробности см. в документации стандартного модуля Python time).

### psycopg2.TimestampFromTicks(ticks)

Эта функция создает объект, содержащий значение временной метки из заданного значения ticks (количество секунд с начала эпохи; подробности см. в документации стандартного модуля Python time).

### psycopg2.Binary(string)

Эта функция создает объект, способный хранить двоичное (длинное) строковое значение.

{% hint style="info" %}
Все адаптеры, возвращаемые фабриками уровня модуля (варианты Binary, Date, Time, Timestamp и \*FromTicks), предоставляют обернутый объект (обычный объект Python, такой как datetime) в адаптированном атрибуте.
{% endhint %}

### psycopg2.STRING

Этот тип объекта используется для описания столбцов в базе данных, которые основаны на строках (например, CHAR).

### psycopg2.BINARY

Этот тип объекта используется для описания (длинных) двоичных столбцов в базе данных (например, LONG, RAW, BLOB).

### psycopg2.NUMBER

Этот тип объекта используется для описания числовых столбцов в базе данных.

### psycopg2.DATETIME

Этот тип объекта используется для описания столбцов даты/времени в базе данных.

### psycopg2.ROWID

Этот тип объекта используется для описания столбца "Row ID" в базе данных.
