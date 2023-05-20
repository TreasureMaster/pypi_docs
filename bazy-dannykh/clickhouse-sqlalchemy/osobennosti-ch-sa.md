# Особенности ch-sa

В этом разделе описываются функции, поддерживаемые текущим диалектом.

## Таблицы и определение моделей

Поддерживаются как декларативные, так и императивные таблицы:

```python
from sqlalchemy import create_engine, Column, MetaData, literal

from clickhouse_sqlalchemy import (
    Table, make_session, get_declarative_base, types, engines
)

uri = 'clickhouse://default:@localhost/test'

engine = create_engine(uri)
session = make_session(engine)
metadata = MetaData(bind=engine)

Base = get_declarative_base(metadata=metadata)

class Rate(Base):
    day = Column(types.Date, primary_key=True)
    value = Column(types.Int32, comment='Rate value')
    other_value = Column(types.DateTime)

    __table_args__ = (
        engines.Memory(),
        {'comment': 'Store rates'}
    )

another_table = Table('another_rate', metadata,
    Column('day', types.Date, primary_key=True),
    Column('value', types.Int32, server_default=literal(1)),
    engines.Memory()
)
```

Таблицы, созданные декларативным способом, имеют строчные буквы со словами, разделенными символами подчеркивания. Но вы можете легко установить свой собственный с помощью атрибута SQLAlchemy `__tablename__`.

Также можно использовать **func** proxy SQLAlchemy для реальных функций ClickHouse.

### Опции, специфичные для диалекта

Вы можете указать конкретный кодек для столбца:

```python
class Rate(Base):
    day = Column(types.Date, primary_key=True)
    value = Column(types.Int32)
    other_value = Column(
        types.DateTime,
        clickhouse_codec=('DoubleDelta', 'ZSTD')
    )

    __table_args__ = (
        engines.Memory(),
    )
```

```sql
CREATE TABLE rate (
    day Date,
    value Int32,
    other_value DateTime CODEC(DoubleDelta, ZSTD)
) ENGINE = Memory
```

**server\_default** будет отображаться как **DEFAULT**

```python
class Rate(Base):
    day = Column(types.Date, primary_key=True)
    value = Column(types.Int32)
    other_value = Column(
        types.DateTime, server_default=func.now()
    )

    __table_args__ = (
        engines.Memory(),
    )
```

```sql
CREATE TABLE rate (
    day Date,
    value Int32,
    other_value DateTime DEFAULT now()
) ENGINE = Memory
```

**MATERIALIZED** и **ALIAS** также поддерживаются

```python
class Rate(Base):
    day = Column(types.Date, primary_key=True)
    value = Column(types.Int32)
    other_value = Column(
        types.DateTime, clickhouse_materialized=func.now()
    )

    __table_args__ = (
        engines.Memory(),
    )
```

```sql
CREATE TABLE rate (
    day Date,
    value Int32,
    other_value DateTime MATERIALIZED now()
) ENGINE = Memory
```

```python
class Rate(Base):
    day = Column(types.Date, primary_key=True)
    value = Column(types.Int32)
    other_value = Column(
        types.DateTime, clickhouse_alias=func.now()
    )

    __table_args__ = (
        engines.Memory(),
    )
```

```sql
CREATE TABLE rate (
    day Date,
    value Int32,
    other_value DateTime ALIAS now()
) ENGINE = Memory
```

Вы также можете указать другой столбец как **default**, **materialized** и **alias**.

```python
class Rate(Base):
    day = Column(types.Date, primary_key=True)
    value = Column(types.Int32)
    other_value = Column(types.Int32, server_default=value)

    __table_args__ = (
        engines.Memory(),
    )
```

```sql
CREATE TABLE rate (
    day Date,
    value Int32,
    other_value Int32 DEFAULT value
) ENGINE = Memory
```

### Движки таблицы

Для каждой таблицы в **ClickHouse** требуется движок. Движок можно указать в декларативном `__table_args__`:

```python
from sqlalchemy import create_engine, MetaData, Column
from clickhouse_sqlalchemy import (
    get_declarative_base, types, engines
)

engine = create_engine('clickhouse://localhost')
metadata = MetaData(bind=engine)
Base = get_declarative_base(metadata=metadata)

class Statistics(Base):
    date = Column(types.Date, primary_key=True)
    sign = Column(types.Int8)
    grouping = Column(types.Int32)
    metric1 = Column(types.Int32)

    __table_args__ = (
        engines.CollapsingMergeTree(
            sign,
            partition_by=func.toYYYYMM(date),
            order_by=(date, grouping)
        ),
    )
```

Или в таблице:

```python
from sqlalchemy import create_engine, MetaData, Column, text
from clickhouse_sqlalchemy import (
    get_declarative_base, types, engines
)

engine = create_engine('clickhouse+native://localhost/default')
metadata = MetaData(bind=engine)

statistics = Table(
    'statistics', metadata,
    Column('date', types.Date, primary_key=True),
    Column('sign', types.Int8),
    Column('grouping', types.Int32),
    Column('metric1', types.Int32),

    engines.CollapsingMergeTree(
        'sign',
        partition_by=text('toYYYYMM(date)'),
        order_by=('date', 'grouping')
    )
)
```

Параметры механизма могут быть переменными столбцов или именами столбцов.

{% hint style="info" %}
Функции SQLAlchemy можно применять к переменным, но не к именам.

Это будет работать, `partition_by=func.toYYYYMM(date)`и это не будет: `partition_by=func.toYYYYMM('date')`. Вы должны использовать `partition_by=text('toYYYYMM(date)')` во втором случае.
{% endhint %}

В настоящее время поддерживаются движки:

* \*MergeTree
* Replicated\*MergeTree
* Distributed
* Buffer
* View/MaterializedView
* Log/TinyLog
* Memory
* Null
* File

У каждого движка свои параметры. Пожалуйста, обратитесь к документации ClickHouse о движках.

Настройки движка могут быть переданы в качестве дополнительных аргументов ключевого слова

```python
engines.MergeTree(
    partition_by=date,
    key='value'
)
```

Будет отображаться в

```sql
MergeTree()
PARTITION BY date
SETTINGS key=value
```

Более сложные примеры

```python
engines.MergeTree(order_by=func.tuple_())

engines.MergeTree(
    primary_key=('device_id', 'timestamp'),
    order_by=('device_id', 'timestamp'),
    partition_by=func.toYYYYMM(timestamp)
)

engines.MergeTree(
    partition_by=text('toYYYYMM(date)'),
    order_by=('date', func.intHash32(x)),
    sample_by=func.intHash32(x)
)

engines.MergeTree(
    partition_by=date,
    order_by=(date, x),
    primary_key=(x, y),
    sample_by=func.random(),
    key='value'
)

engines.CollapsingMergeTree(
    sign,
    partition_by=date,
    order_by=(date, x)
)

engines.ReplicatedCollapsingMergeTree(
    '/table/path', 'name',
    sign,
    partition_by=date,
    order_by=(date, x)
)

engines.VersionedCollapsingMergeTree(
    sign, version,
    partition_by=date,
    order_by=(date, x),
)

engines.SummingMergeTree(
    columns=(y, ),
    partition_by=date,
    order_by=(date, x)
)

engines.ReplacingMergeTree(
    version='version',
    partition_by='date',
    order_by=('date', 'x')
)
```

Таблицы могут быть отражены с движками

```python
from sqlalchemy import create_engine, MetaData
from clickhouse_sqlalchemy import Table

engine = create_engine('clickhouse+native://localhost/default')
metadata = MetaData(bind=engine)

statistics = Table('statistics', metadata, autoload=True)
```

{% hint style="info" %}
Отражение возможно для таблиц, созданных с использованием современного синтаксиса. Таблица со следующим движком не может быть отражена.
{% endhint %}

{% hint style="info" %}
Рефлексия движка может занять много времени, если в вашей базе данных много таблиц. Вы можете управлять отражением движка с помощью параметра соединения **engine\_reflection**.
{% endhint %}

### ON CLUSTER

Предложение **ON CLUSTER** будет автоматически добавлено в запросы **DDL** (`CREATE TABLE`, `DROP TABLE` и т. д.), если в `__table_args__` указан кластер.

```python
class TestTable(...):
    ...

    __table_args__ = (
        engines.ReplicatedMergeTree(...),
        {'clickhouse_cluster': 'my_cluster'}
    )
```

### TTL

Предложение **TTL** может отображаться во время создания таблицы.

```python
class TestTable(...):
    date = Column(types.Date, primary_key=True)
    x = Column(types.Int32)

    __table_args__ = (
        engines.MergeTree(ttl=date + func.toIntervalDay(1)),
    )
```

```sql
CREATE TABLE test_table (date Date, x Int32)
ENGINE = MergeTree()
TTL date + toIntervalDay(1)
```

Удаление

```python
from clickhouse_sqlalchemy.sql.ddl import ttl_delete

class TestTable(...):
    date = Column(types.Date, primary_key=True)
    x = Column(types.Int32)

    __table_args__ = (
        engines.MergeTree(
            ttl=ttl_delete(date + func.toIntervalDay(1))
        ),
    )
```

```sql
CREATE TABLE test_table (date Date, x Int32)
ENGINE = MergeTree()
TTL date + toIntervalDay(1) DELETE
```

Несколько предложений одновременно

```python
from clickhouse_sqlalchemy.sql.ddl import (
    ttl_delete,
    ttl_to_disk,
    ttl_to_volume
)

ttl = [
    ttl_delete(date + func.toIntervalDay(1)),
    ttl_to_disk(date + func.toIntervalDay(1), 'hdd'),
    ttl_to_volume(date + func.toIntervalDay(1), 'slow'),
]

class TestTable(...):
    date = Column(types.Date, primary_key=True)
    x = Column(types.Int32)

    __table_args__ = (
        engines.MergeTree(ttl=ttl),
    )
```

```sql
CREATE TABLE test_table (date Date, x Int32)
ENGINE = MergeTree()
TTL date + toIntervalDay(1) DELETE,
    date + toIntervalDay(1) TO DISK 'hdd',
    date + toIntervalDay(1) TO VOLUME 'slow'
```

### Пользовательские движки

Если какой-то движок еще не поддерживается, вы можете добавить новый в свой код следующим образом:

```python
from sqlalchemy import create_engine, MetaData, Column
from clickhouse_sqlalchemy import (
    Table, get_declarative_base, types
)
from clickhouse_sqlalchemy.engines.base import Engine

engine = create_engine('clickhouse://localhost/default')
metadata = MetaData(bind=engine)
Base = get_declarative_base(metadata=metadata)

class Kafka(Engine):
    def __init__(self, broker_list, topic_list):
        self.broker_list = broker_list
        self.topic_list = topic_list
        super(Kafka, self).__init__()

    @property
    def name(self):
        return (
            super(Kafka, self).name + '()' +
            '\nSETTINGS kafka_broker_list={},'
            '\nkafka_topic_list={}'.format(
                self.broker_list, self.topic_list
            )
        )

table = Table(
    'test', metadata,
    Column('x', types.Int32),
    Kafka(
        broker_list='host:port',
        topic_list = 'topic1,topic2,...'
    )
)
```

## Материализованные представления
