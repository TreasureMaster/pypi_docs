# SQLAlchemy в Flask

Многие люди предпочитают [SQLAlchemy](https://www.sqlalchemy.org/) для доступа к базе данных. В этом случае рекомендуется использовать пакет вместо модуля для вашего приложения **Flask** и поместить модели в отдельный модуль ([большие приложения](bolshie-prilozheniya-flask.md)). Хотя в этом нет необходимости, в этом есть большой смысл.

Есть четыре очень распространенных способа использования **SQLAlchemy**. Я обрисую здесь каждую из них:

## Расширение Flask-SQLAlchemy

Поскольку **SQLAlchemy** - это общий уровень абстракции базы данных и объектно-реляционное сопоставление, требующее немного усилий по настройке, существует расширение **Flask**, которое сделает это за вас. Это рекомендуется, если вы хотите быстро приступить к работе.

Вы можете скачать [Flask-SQLAlchemy](https://flask-sqlalchemy.palletsprojects.com/en/2.x/) из [PyPI](https://pypi.org/project/Flask-SQLAlchemy/).

## Декларативное расширение

Декларативное расширение в **SQLAlchemy** - это самый последний метод использования **SQLAlchemy**. Он позволяет вам определять таблицы и модели за один раз, подобно тому, как работает **Django**. В дополнение к следующему тексту я рекомендую официальную документацию по [декларативному](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/) расширению.

Вот пример модуля `database.py` для вашего приложения:

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import scoped_session, sessionmaker
from sqlalchemy.ext.declarative import declarative_base

engine = create_engine('sqlite:////tmp/test.db', convert_unicode=True)
db_session = scoped_session(sessionmaker(autocommit=False,
                                         autoflush=False,
                                         bind=engine))
Base = declarative_base()
Base.query = db_session.query_property()

def init_db():
    # импортируйте сюда все модули, которые могут определять модели,
    # чтобы они были правильно зарегистрированы в метаданных.
    # В противном случае вам придется сначала импортировать их,
    # прежде чем вызывать init_db ()
    import yourapplication.models
    Base.metadata.create_all(bind=engine)
```

Чтобы определить свои модели, просто создайте подкласс базового класса **Base**, который был создан приведенным выше кодом. Если вам интересно, почему нам не нужно заботиться о потоках здесь (как мы это делали в приведенном выше примере **SQLite3** с объектом [g](../api-dokumentaciya-flask/globalnyi-obekt-prilozheniya-flask.md#flask-g)): это потому, что **SQLAlchemy** делает это за нас уже с **scoped\_session**.

Чтобы декларативно использовать **SQLAlchemy** с вашим приложением, вам просто нужно поместить следующий код в модуль вашего приложения. **Flask** автоматически удалит сеансы базы данных в конце запроса или при завершении работы приложения:

```python
from yourapplication.database import db_session

@app.teardown_appcontext
def shutdown_session(exception=None):
    db_session.remove()
```

Вот пример модели (поместите это, например, в `models.py`):

```python
from sqlalchemy import Column, Integer, String
from yourapplication.database import Base

class User(Base):
    __tablename__ = 'users'
    id = Column(Integer, primary_key=True)
    name = Column(String(50), unique=True)
    email = Column(String(120), unique=True)

    def __init__(self, name=None, email=None):
        self.name = name
        self.email = email

    def __repr__(self):
        return '<User %r>' % (self.name)
```

Для создания базы данных вы можете использовать функцию **init\_db**:

```python
>>> from yourapplication.database import init_db
>>> init_db()
```

Вы можете вставить записи в базу данных следующим образом:

```python
>>> from yourapplication.database import db_session
>>> from yourapplication.models import User
>>> u = User('admin', 'admin@localhost')
>>> db_session.add(u)
>>> db_session.commit()
```

Запросы тоже просты:

```python
>>> User.query.all()
[<User u'admin'>]
>>> User.query.filter(User.name == 'admin').first()
<User u'admin'>
```

## Реляционное сопоставление объектов вручную

У ручного объектного реляционного сопоставления есть несколько преимуществ и несколько недостатков по сравнению с декларативным подходом, описанным выше. Основное отличие состоит в том, что вы определяете таблицы и классы отдельно и сопоставляете их вместе. Он более гибкий, но в нем немного больше возможностей для ввода. В целом он работает как декларативный подход, поэтому не забудьте также разделить свое приложение на несколько модулей в пакете.

Вот пример модуля `database.py` для вашего приложения:

```python
from sqlalchemy import create_engine, MetaData
from sqlalchemy.orm import scoped_session, sessionmaker

engine = create_engine('sqlite:////tmp/test.db', convert_unicode=True)
metadata = MetaData()
db_session = scoped_session(sessionmaker(autocommit=False,
                                         autoflush=False,
                                         bind=engine))
def init_db():
    metadata.create_all(bind=engine)
```

Как и в декларативном подходе, вам необходимо закрывать сеанс после каждого запроса или выключения контекста приложения. Поместите это в свой модуль приложения:

```python
from yourapplication.database import db_session

@app.teardown_appcontext
def shutdown_session(exception=None):
    db_session.remove()
```

Вот пример таблицы и модели (поместите это в `models.py`):

```python
from sqlalchemy import Table, Column, Integer, String
from sqlalchemy.orm import mapper
from yourapplication.database import metadata, db_session

class User(object):
    query = db_session.query_property()

    def __init__(self, name=None, email=None):
        self.name = name
        self.email = email

    def __repr__(self):
        return '<User %r>' % (self.name)

users = Table('users', metadata,
    Column('id', Integer, primary_key=True),
    Column('name', String(50), unique=True),
    Column('email', String(120), unique=True)
)
mapper(User, users)
```

Запросы и вставка работают точно так же, как в приведенном выше примере.

## Уровень абстракции SQL

Если вы просто хотите использовать уровень абстракции системы баз данных (и SQL), вам в основном нужен только движок:

```python
from sqlalchemy import create_engine, MetaData, Table

engine = create_engine('sqlite:////tmp/test.db', convert_unicode=True)
metadata = MetaData(bind=engine)
```

Затем вы можете объявить таблицы в своем коде, как в примерах выше, или автоматически загрузить их:

```python
from sqlalchemy import Table

users = Table('users', metadata, autoload=True)
```

Для вставки данных вы можете использовать метод **insert**. Сначала мы должны установить соединение, чтобы мы могли использовать транзакцию:

```python
>>> con = engine.connect()
>>> con.execute(users.insert(), name='admin', email='admin@localhost')
```

**SQLAlchemy** автоматически выполнит фиксацию за нас.

Чтобы запросить свою базу данных, вы используете движок напрямую или используете соединение:

```python
>>> users.select(users.c.id == 1).execute().first()
(1, u'admin', u'admin@localhost')
```

Эти результаты также являются кортежами типа **dict**:

```python
>>> r = users.select(users.c.id == 1).execute().first()
>>> r['name']
u'admin'
```

Вы также можете передавать строки операторов **SQL** методу **execute ()**:

```python
>>> engine.execute('select * from users where id = :1', [1]).first()
(1, u'admin', u'admin@localhost')
```

Для получения дополнительной информации о SQLAlchemy перейдите на [веб-сайт](https://www.sqlalchemy.org/).
