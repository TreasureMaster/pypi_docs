# Объявление моделей

Обычно **Flask-SQLAlchemy** ведет себя как правильно настроенная декларативная база из [декларативного](https://docs.sqlalchemy.org/en/14/orm/extensions/declarative/api.html#module-sqlalchemy.ext.declarative) расширения. Поэтому мы рекомендуем прочитать документацию по SQLAlchemy для полного ознакомления. Однако наиболее распространенные варианты использования также задокументированы здесь.

Что нужно иметь в виду:

* Базовый класс для всех ваших моделей называется `db.Model`. Он хранится в экземпляре SQLAlchemy, который вам нужно создать. Дополнительные сведения см. в разделе «[Быстрый старт](bystryi-start-flask-sqlalchemy.md)».
* Некоторые части, которые требуются в SQLAlchemy, являются необязательными в **Flask-SQLAlchemy**. Например, имя таблицы автоматически устанавливается для вас, если оно не переопределено. Оно получено из имени класса, преобразованного в нижний регистр, а «CamelCase» преобразован в «camel\_case». Чтобы переопределить имя таблицы, установите атрибут класса **\_\_tablename\_\_**.

### Простой пример

Очень простой пример:

```python
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)

    def __repr__(self):
        return '<User %r>' % self.username
```

Используйте [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column) для определения столбца. Имя столбца — это имя, которому вы его назначаете. Если вы хотите использовать другое имя в таблице, вы можете указать необязательный первый аргумент, который представляет собой строку с желаемым именем столбца. Первичные ключи помечаются с помощью `primary_key=True`. Несколько ключей могут быть помечены как первичные ключи, и в этом случае они становятся **составным** первичным ключом.

Типы столбцов являются первым аргументом [Column](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Column). Вы можете либо предоставить их напрямую, либо вызвать их, чтобы уточнить их (например, указать длину). Наиболее распространены следующие виды:

| Тип колонки                                                                                            | Описание                                                                                                                  |
| ------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------- |
| [`Integer`](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.Integer)         | целое число                                                                                                               |
| [`String(size)`](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.String)     | строка максимальной длины (необязательно в некоторых базах данных, например PostgreSQL)                                   |
| [`Text`](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.Text)               | какой-то более длинный текст юникода                                                                                      |
| [`DateTime`](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.DateTime)       | дата и время, выраженные как объект [datetime](https://docs.python.org/3/library/datetime.html#datetime.datetime) Python. |
| [`Float`](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.Float)             | хранит значения с плавающей запятой                                                                                       |
| [`Boolean`](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.Boolean)         | хранит логическое значение                                                                                                |
| [`PickleType`](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.PickleType)   | хранит консервированный объект Python                                                                                     |
| [`LargeBinary`](https://docs.sqlalchemy.org/en/14/core/type\_basics.html#sqlalchemy.types.LargeBinary) | хранит большие произвольные двоичные данные                                                                               |

### Отношения «один ко многим»

Наиболее распространенными отношениями являются отношения «один ко многим». Поскольку отношения объявляются до того, как они будут установлены, вы можете использовать строки для ссылки на классы, которые еще не созданы (например, если **Person** определяет отношение к **Address**, которое объявляется позже в файле).

Отношения выражаются с помощью функции [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship). Однако внешний ключ должен быть объявлен отдельно с классом [ForeignKey](https://docs.sqlalchemy.org/en/14/core/constraints.html#sqlalchemy.schema.ForeignKey):

```python
class Person(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(50), nullable=False)
    addresses = db.relationship('Address', backref='person', lazy=True)

class Address(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    email = db.Column(db.String(120), nullable=False)
    person_id = db.Column(db.Integer, db.ForeignKey('person.id'),
        nullable=False)
```

Что делает [db.relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship)? Эта функция возвращает новое свойство, которое может делать несколько вещей. В этом случае мы сказали ему указать на класс **Address** и загрузить несколько из них. Откуда он знает, что это вернет более одного адреса? Потому что SQLAlchemy угадывает полезное значение по умолчанию из вашего объявления. Если вы хотите иметь связь один к одному, вы можете передать `uselist=False` в функцию [relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship).

Поскольку человек без имени или адрес электронной почты без связанного адреса не имеет смысла, `nullable=False` указывает SQLAlchemy создать столбец как **NOT NULL**. Это подразумевается для столбцов первичного ключа, но рекомендуется указать его для всех остальных столбцов, чтобы другим людям, работающим над вашим кодом, было понятно, что вы действительно хотите столбец, допускающий значение **NULL**, а не просто забыли его добавить.

Так что же означают **backref** и **lazy**? **backref** — это простой способ также объявить новое свойство в классе **Address**. Затем вы также можете использовать **my\_address.person**, чтобы связаться с человеком по этому адресу. **lazy** определяет, когда SQLAlchemy будет загружать данные из базы данных:

* `'select' / True` (значение по умолчанию, но явное лучше, чем неявное) означает, что SQLAlchemy будет загружать данные по мере необходимости за один раз, используя стандартный оператор выбора.
* `'joined' / False` указывает SQLAlchemy загрузить отношение в том же запросе, что и родитель, используя оператор **JOIN**.
* `'subquery'` работает как `'joined'`, но вместо этого SQLAlchemy будет использовать подзапрос.
* `'dynamic'` является особенным и может быть полезен, если у вас много элементов и вы всегда хотите применять к ним дополнительные фильтры SQL. Вместо загрузки элементов SQLAlchemy вернет другой объект запроса, который вы можете уточнить перед загрузкой элементов. Обратите внимание, что это нельзя превратить в другую стратегию загрузки при запросе, поэтому часто рекомендуется избегать использования этого в пользу `lazy=True`. Объект запроса, эквивалентный динамическому отношению `user.addresses`, может быть создан с помощью **Address.query.with\_parent(user)**, при этом при необходимости можно использовать ленивую или нетерпеливую загрузку самого отношения.

Как вы определяете ленивый статус для обратных ссылок? С помощью функции [backref()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.backref):

```python
class Person(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(50), nullable=False)
    addresses = db.relationship('Address', lazy='select',
        backref=db.backref('person', lazy='joined'))
```

### Отношения «многие ко многим»

Если вы хотите использовать отношения «многие ко многим», вам необходимо определить вспомогательную таблицу, которая будет использоваться для этого отношения. Для этой вспомогательной таблицы настоятельно рекомендуется использовать не модель, а реальную таблицу:

```python
tags = db.Table('tags',
    db.Column('tag_id', db.Integer, db.ForeignKey('tag.id'), primary_key=True),
    db.Column('page_id', db.Integer, db.ForeignKey('page.id'), primary_key=True)
)

class Page(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    tags = db.relationship('Tag', secondary=tags, lazy='subquery',
        backref=db.backref('pages', lazy=True))

class Tag(db.Model):
    id = db.Column(db.Integer, primary_key=True)
```

Здесь мы настроили загрузку тегов `Page.tags` сразу после загрузки страницы, но с использованием отдельного запроса. Это всегда приводит к двум запросам при извлечении страницы, но при запросе нескольких страниц вы не получите дополнительных запросов.

С другой стороны, список страниц для тега нужен редко. Например, вам не понадобится этот список при получении тегов для конкретной страницы. Поэтому обратная ссылка настроена на ленивую загрузку, поэтому доступ к ней в первый раз вызовет запрос на получение списка страниц для этого тега. Если вам нужно применить дополнительные параметры запроса в этом списке, вы можете либо переключиться на стратегию `'dynamic'` — с упомянутыми выше недостатками — либо получить объект запроса, используя **Page.query.with\_parent(some\_tag)**, а затем использовать его точно так, как вы будете с объектом запроса из динамической связи.
