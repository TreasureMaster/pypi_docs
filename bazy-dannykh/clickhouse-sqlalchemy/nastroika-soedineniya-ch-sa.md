# Настройка соединения ch-sa

ClickHouse SQLAlchemy использует следующий синтаксис для строки подключения:

```
clickhouse+driver://user:password@host:NNNN/database
```

Где:

* **driver** — драйвер для использования. Возможные варианты: **http**, **native**, **asynch**. По умолчанию - **http**. Когда вы опускаете **driver**, используется **http**.
* **database** - это база данных, к которой подключается. По умолчанию - **default**.
* **user** - пользователь базы данных. По умолчанию - **default**.
* **password** - пароль пользователя. По умолчанию `''` (без пароля).
* **port** - можно настроить, если сервер ClickHouse прослушивает нестандартный порт.

Драйверу передаются дополнительные параметры.

## Общие параметры

* **engine\_reflection** управляет отражением движка таблицы во время отражения таблицы. Рефлексия движка может быть очень медленной, если у вас тысячи таблиц. Вы можете отключить отражение, установив для этого параметра значение `false`. Возможные варианты: `true/false`. Значение по умолчанию `true`.
* **server\_version** можно использовать для исключения запроса инициализации `select version()`. Как правило, вы не должны устанавливать этот параметр, и версия сервера будет определена автоматически.

## Параметры драйвера

В строке запроса можно указать несколько параметров.

### HTTP

* **port** — это порт, к которому привязан сервер ClickHouse. По умолчанию `8123`.
* **timeout** в секундах. Тайм-аута по умолчанию нет.
* **protocol** для использования. Возможные варианты: **http**, **https**. По умолчанию - **http**.
* **verify** управляет проверкой сертификата в протоколе **https**. Возможные варианты: `true/false`. Значение по умолчанию `true`.

Простой пример DSN:

```
clickhouse+http://host/db
```

Пример DSN для https-порта ClickHouse:

```
clickhouse+http://user:password@host:8443/db?protocol=https
```

При использовании **nginx** в качестве прокси-сервера для сервера ClickHouse строка подключения может выглядеть так:

```
clickhouse+http://user:password@host:8124/test?protocol=https
```

Где **8124** — порт прокси.

Если вам нужен контроль над базовым HTTP-соединением, передайте экземпляр [requests.Session](https://requests.readthedocs.io/en/master/user/advanced/#session-objects) в `create_engine()`, например:

```python
from sqlalchemy import create_engine
from requests import Session

engine = create_engine(
    'clickhouse+http://localhost/test',
    connect_args={'http_session': Session()}
)
```

### Native

Обратите внимание, что соединение native **не шифруется**. Все данные, включая имя пользователя/пароль, передаются в виде обычного текста. Вы должны использовать это соединение через SSH или VPN (например) при общении через ненадежную сеть.

Простой пример DSN:

```
clickhouse+native://host/db
```

Все параметры строки подключения проксируются в **clickhouse-driver**. Смотрите [его параметры](https://clickhouse-driver.readthedocs.io/en/latest/api.html#clickhouse\_driver.connection.Connection).

Пример DSN со сжатием **LZ4**, защищенным сертификатом Let’s Encrypt на стороне сервера:

```python
import certify

dsn = (
    'clickhouse+native://user:pass@host/db?compression=lz4&'
    'secure=True&ca_certs={}'.format(certify.where())
)
```

Пример с несколькими хостами

```
clickhouse+native://wronghost/default?alt_hosts=localhost:9000
```

### Asynch

То же, что **native**.

Простой пример DSN:

```
clickhouse+asynch://host/db
```

Все параметры строки подключения проксируются в **asynch**. Смотрите [его параметры](https://github.com/long2ice/asynch/blob/dev/asynch/connection.py).
