# Установка ch-sa

## Версия Python

**Clickhouse-sqlalchemy** поддерживает **Python 2.7** и новее.

## Зависимости

Эти дистрибутивы будут установлены автоматически при установке **clickhouse-sqlalchemy**:

* [clickhouse-driver](https://pypi.org/project/clickhouse-driver/) - драйвер ClickHouse Python с поддержкой собственного (TCP) интерфейса.
* [requests](https://pypi.org/project/requests/) - простая и элегантная библиотека HTTP.
* [ipaddress](https://pypi.org/project/ipaddress/) - backport модуль ipaddress.
* [asynch](https://pypi.org/project/asynch/) - драйвер asyncio ClickHouse Python с поддержкой собственного (TCP) интерфейса.

Если вы планируете использовать **clickhouse-driver** со сжатием, вам также следует установить дополнительные средства сжатия. См. [документацию](https://clickhouse-driver.readthedocs.io/) по драйверу clickhouse.

## Установка из PyPI

Пакет можно установить с помощью **pip**:

```bash
pip install clickhouse-sqlalchemy
```

## Установка с Github

Разрабатываемую версию можно установить прямо с **github**:

```bash
pip install git+https://github.com/xzkostyan/clickhouse-sqlalchemy@master#egg=clickhouse-sqlalchemy
```
