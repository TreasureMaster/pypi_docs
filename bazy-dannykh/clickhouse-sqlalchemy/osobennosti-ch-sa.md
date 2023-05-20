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

Материализованные представления могут быть определены так же, как и модели. Определение состоит из двух шагов:

* определение хранилища (таблица, в которой будут храниться данные)
* Определение запроса **SELECT**

```python
from clickhouse_sqlalchemy import MaterializedView, select

class Statistics(Base):
    date = Column(types.Date, primary_key=True)
    sign = Column(types.Int8, nullable=False)
    grouping = Column(types.Int32, nullable=False)
    metric1 = Column(types.Int32, nullable=False)

    __table_args__ = (
        engines.CollapsingMergeTree(
            sign,
            partition_by=func.toYYYYMM(date),
            order_by=(date, grouping)
        ),
    )


# Определить хранилище для материализованного представления
class GroupedStatistics(Base):
    date = Column(types.Date, primary_key=True)
    metric1 = Column(types.Int32, nullable=False)

    __table_args__ = (
        engines.SummingMergeTree(
            partition_by=func.toYYYYMM(date),
            order_by=(date, )
        ),
    )


Stat = Statistics

# Определить SELECT для материализованного представления
MatView = MaterializedView(GroupedStatistics, select([
    Stat.date.label('date'),
    func.sum(Stat.metric1 * Stat.sign).label('metric1')
]).where(
    Stat.grouping > 42
).group_by(
    Stat.date
))

Stat.__table__.create()
MatView.create()
```

Определение материализованных представлений в коде полезно для дальнейших миграций. Автогенерация может уменьшить возможные человеческие ошибки в случае столбцов и материализованных представлений.

{% hint style="info" %}
В настоящее время невозможно обнаружить движок **database** во время запуска. Необходимо указать, будет ли материализованное представление использовать синтаксис `TO [db.]name`.
{% endhint %}

Сейчас есть два движка баз данных: **Ordinary** и **Atomic**.

Если в вашей базе данных используется движок **Ordinary**, внутренняя таблица будет создана автоматически для материализованного представления. Вы можете управлять генерацией имени, только определив класс для внутренней таблицы с соответствующим именем. `class GroupedStatistics` в примере выше.

Если в вашей базе данных есть внутренние таблицы **Atomic Engine**, которые не используются для материализованного представления, вы должны добавить **use\_to** для объекта материализованного представления: `MaterializedView(..., use_to=True)`. Вы можете дополнительно указать имя материализованного представления с именем `name=...`. По умолчанию имя представления — это имя таблицы с `mv_suffix = '_mv'`.

Примеры:

* `MaterializedView(TestTable, use_to=True)` — это объявление материализованного представления **test\_table\_mv**.
* `MaterializedView(TestTable, use_to=True, name='my_mv')` — это объявление материализованного представления **my\_mv**.
* `MaterializedView(TestTable, use_to=True, mv_suffix='_mat_view')` — это объявление материализованного представления **test\_table\_mat\_view**.

Вы можете указать кластер для материализованного представления в определении внутренней таблицы.

```python
class GroupedStatistics(...):
    ...

    __table_args__ = (
        engines.ReplicatedSummingMergeTree(...),
        {'clickhouse_cluster': 'my_cluster'}
    )
```

## Базовая поддержка DDL

Вы можете делать простой DDL. Пример таблицы `CREATE/DROP`:

```python
table = Rate.__table__
table.create()
another_table.create()

another_table.drop()
table.drop()
```

## Цепочка методов запроса

Поддерживаются общие **order\_by**, **filter**, **limit**, **offset** и т. д., а также специфичный для ClickHouse **final** и другие.

```python
session.query(func.count(Rate.day)) \
    .filter(Rate.day > today - timedelta(20)) \
    .scalar()

session.query(Rate.value) \
    .order_by(Rate.day.desc()) \
    .first()

session.query(Rate.value) \
    .order_by(Rate.day) \
    .limit(10) \
    .all()

session.query(func.sum(Rate.value)) \
    .scalar()
```

## INSERT

Простой пакетный INSERT:

```python
from datetime import date, timedelta
from sqlalchemy import func

today = date.today()
rates = [
    {'day': today - timedelta(i), 'value': 200 - i}
    for i in range(100)
]

# Emits single INSERT statement.
session.execute(table.insert(), rates)
```

Оператор INSERT FROM SELECT:

```python
from sqlalchemy import cast

# Метки должны присутствовать.
select_query = session.query(
    Rate.day.label('day'),
    cast(Rate.value * 1.5, types.Int32).label('value')
).subquery()

# Emits single INSERT FROM SELECT statement
session.execute(
    another_table.insert()
    .from_select(['day', 'value'], select_query)
)
```

## UPDATE и DELETE

Оператор обновления SQLAlchemy сопоставляется с оператором **ALTER UPDATE** ClickHouse.

```python
tbl = Table(...)
session.execute(t1.update().where(t1.c.x == 25).values(x=5))
```

или

```python
tbl = Table(...)
session.execute(update(t1).where(t1.c.x == 25).values(x=5))
```

становится

```sql
ALTER TABLE ... UPDATE x=5 WHERE x = 25
```

Оператор Delete также поддерживается и отображается в **ALTER DELETE**.

```python
tbl = Table(...)
session.execute(t1.delete().where(t1.c.x == 25))
```

или

```python
tbl = Table(...)
session.execute(delete(t1).where(t1.c.x == 25))
```

становится

```sql
ALTER TABLE ... DELETE WHERE x = 25
```

Многие другие функции SQLAlchemy поддерживаются по умолчанию. Пример **UNION ALL**:

```python
from sqlalchemy import union_all

select_rate = session.query(
    Rate.day.label('date'),
    Rate.value.label('x')
)
select_another_rate = session.query(
    another_table.c.day.label('date'),
    another_table.c.value.label('x')
)

union_all(select_rate, select_another_rate) \
    .execute() \
    .fetchone()
```

## Расширения SELECT

Диалект поддерживает некоторые расширения ClickHouse для запроса **SELECT**.

### SAMPLE

```python
session.query(table.c.x).sample(0.1)
```

или

```python
select([table.c.x]).sample(0.1)
```

становится

```sql
SELECT ... FROM ... SAMPLE 0.1
```

### LIMIT BY

```python
session.query(table.c.x).order_by(table.c.x) \
    .limit_by([table.c.x], offset=1, limit=2)
```

или

```python
select([table.c.x]).order_by(table.c.x) \
    .limit_by([table.c.x], offset=1, limit=2)
```

становится

```sql
SELECT ... FROM ... ORDER BY ... LIMIT 1, 2 BY ...
```

### Lambda

```python
from clickhouse_sqlalchemy.ext.clauses import Lambda

session.query(
    func.arrayFilter(
        Lambda(lambda x: x.like('%World%')),
        literal(
            ['Hello', 'abc World'],
            types.Array(types.String)
        )
    ).label('test')
)
```

становится

```sql
SELECT arrayFilter(
    x -> x LIKE '%%World%%',
    ['Hello', 'abc World']
) AS test
```

### JOIN

Соединение ClickHouse немного мощнее, чем обычное соединение SQL. В этом диалекте соединение параметризуется следующими аргументами:

* тип: `INNER|LEFT|RIGHT|FULL|CROSS`
* строгость: `OUTER|SEMI|ANTI|ANY|ASOF`
* распределение: `GLOBAL`

Вот некоторые примеры

```python
session.query(t1.c.x, t2.c.x).join(
    t2,
    t1.c.x == t2.c.y,
    type='inner',
    strictness='all',
    distribution='global'
)
```

или

```python
select([t1.c.x, t2.c.x]).join(
    t2,
    t1.c.x == t2.c.y,
    type='inner',
    strictness='all',
    distribution='global'
)
```

становится

```sql
SELECT ... FROM ... GLOBAL ALL INNER JOIN ... ON ...
```

Вы также можете управлять параметрами соединения с помощью собственных параметров SQLAlchemy: **isouter** и **full**.

```python
session.query(t1.c.x, t2.c.x).join(
    t2,
    t1.c.x == t2.c.y,
    isouter=True,
    full=True
)
```

становится

```sql
SELECT ... FROM ... FULL OUTER JOIN ... ON ...
```

### ARRAY JOIN

```python
session.query(...).array_join(...)
```

или

```python
select([...]).array_join(...)
```

становится

```sql
SELECT ... FROM ... ARRAY JOIN ...
```

### WITH CUBE/ROLLUP/TOTALS

```python
session.query(table.c.x).group_by(table.c.x).with_cube()
session.query(table.c.x).group_by(table.c.x).with_rollup()
session.query(table.c.x).group_by(table.c.x).with_totals()
```

или

```python
select([table.c.x]).group_by(table.c.x).with_cube()
select([table.c.x]).group_by(table.c.x).with_rollup()
select([table.c.x]).group_by(table.c.x).with_totals()
```

становится (соответственно)

```sql
SELECT ... FROM ... GROUP BY ... WITH CUBE
SELECT ... FROM ... GROUP BY ... WITH ROLLUP
SELECT ... FROM ... GROUP BY ... WITH TOTALS
```

### FINAL

{% hint style="info" %}
В настоящее время предложение **FINAL** поддерживается только для таблицы, указанной в предложении **FROM**.
{% endhint %}

```python
session.query(table.c.x).final().group_by(table.c.x)
```

или

```python
select([table.c.x]).final().group_by(table.c.x)
```

становится

```sql
SELECT ... FROM ... FINAL GROUP BY ...
```

## Разное

### Пакетирование (batching)

Вы можете захотеть получать очень большие наборы результатов по частям.

```python
session.query(...).yield_per(N)
```

{% hint style="info" %}
Это поддерживается только в драйвере **native**.
{% endhint %}

В этом случае используется **execute\_iter** от **clickhouse-driver**, а для параметра **max\_block\_size** установлено значение **N**.

Есть побочный эффект. Если следующий запрос будет отправлен до конца итерации по запросу с **yield**, возникнет ошибка. Пример

```python
def gen(session):
    yield from session.query(...).yield_per(N)

rv = gen(session)

# Будет ошибка
session.query(...).all()
```

Чтобы избежать этого побочного эффекта, вы должны создать еще один сеанс

```python
class another_session():
    def __init__(self, engine):
        self.engine = engine
        self.session = None

    def __enter__(self):
        self.session = make_session(self.engine)
        return self.session

    def __exit__(self, *exc_info):
        self.session.close()

def gen(session):
    with another_session(session.bind) as new_session:
        yield from new_session.query(...).yield_per(N)

rv = gen(session)

# Не будет ошибки
session.query(...).all()
```

### Варианты выполнения

{% hint style="info" %}
Это поддерживается только в **native** и **asynch** драйверах.
{% endhint %}

Вы можете переопределить настройки сервера ClickHouse по умолчанию и передать нужные настройки с помощью **execute\_options**. Установите более низкий приоритет для запроса и ограничьте максимальное количество потоков для выполнения запроса.

```python
settings = {'max_threads': 2, 'priority': 10}

session.query(...).execution_options(settings=settings)
```

Вы можете передавать внешние таблицы на сервер ClickHouse с параметрами исполнения

```python
table = Table(
    'ext_table1', metadata,
    Column('id', types.UInt64),
    Column('name', types.String),
    clickhouse_data=[(x, 'name' + str(x)) for x in range(10)],
    extend_existing=True
)

session.query(func.sum(table.c.id)) \
    .execution_options(external_tables=[table])
    .scalar()
```
