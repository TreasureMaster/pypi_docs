# Быстрый старт ch-sa

Эта страница дает хорошее введение в **clickhouse-sqlalchemy**. Предполагается, что у вас уже установлен **clickhouse-sqlalchemy**. Если вы этого не сделаете, перейдите в раздел «[Установка](ustanovka-ch-sa.md)».

Следует отметить, что сессия должна быть создана с помощью `clickhouse_sqlalchemy.make_session`. В противном случае `session.query` и `session.execute` не будут иметь расширения **ClickHouse SQL**. То же самое относится к таблице **Table** и **get\_declarative\_base**.

Давайте определим некоторую таблицу, вставим в нее данные и запросим вставленные данные.

```python
from sqlalchemy import create_engine, Column, MetaData

from clickhouse_sqlalchemy import (
    Table, make_session, get_declarative_base, types, engines
)

uri = 'clickhouse+native://localhost/default'

engine = create_engine(uri)
session = make_session(engine)
metadata = MetaData(bind=engine)

Base = get_declarative_base(metadata=metadata)

class Rate(Base):
    day = Column(types.Date, primary_key=True)
    value = Column(types.Int32)

    __table_args__ = (
        engines.Memory(),
    )

# Emits CREATE TABLE statement
Rate.__table__.create()
```

Теперь пришло время вставить некоторые данные

```python
from datetime import date, timedelta

from sqlalchemy import func

today = date.today()
rates = [
    {'day': today - timedelta(i), 'value': 200 - i}
    for i in range(100)
]
```

Давайте запросим вставленные данные

```python
session.execute(Rate.__table__.insert(), rates)

session.query(func.count(Rate.day)) \
    .filter(Rate.day > today - timedelta(20)) \
    .scalar()
```

Теперь вы готовы [настроить подключение](nastroika-soedineniya-ch-sa.md) и увидеть поддержку [дополнительных функций](osobennosti-ch-sa.md) ClickHouse.
