# Flask-Marshmallow

## Flask + marshmallow для красивых API

**Flask-Marshmallow** — это тонкий слой интеграции для [Flask](http://flask.pocoo.org/) (веб-фреймворк Python) и [marshmallow](http://marshmallow.readthedocs.io/) (библиотека сериализации/десериализации объектов), который добавляет дополнительные функции в **marshmallow**, включая поля URL и гиперссылки для HATEOAS-ready API. Он также (необязательно) интегрируется с [Flask-SQLAlchemy](http://flask-sqlalchemy.pocoo.org/).

### Получи это сейчас

```bash
pip install flask-marshmallow
```

Создайте свое приложение.

```python
from flask import Flask
from flask_marshmallow import Marshmallow

app = Flask(__name__)
ma = Marshmallow(app)
```

Пишите свои модели.

```python
from your_orm import Model, Column, Integer, String, DateTime

class User(Model):
    email = Column(String)
    password = Column(String)
    date_created = Column(DateTime, auto_now_add=True)
```

Определите формат вывода с помощью marshmallow.

```python
class UserSchema(ma.Schema):
    class Meta:
        # Поля для показа
        fields = ("email", "date_created", "_links")

    # Умные гиперссылки
    _links = ma.Hyperlinks(
        {
            "self": ma.URLFor("user_detail", values=dict(id="<id>")),
            "collection": ma.URLFor("users"),
        }
    )

user_schema = UserSchema()
users_schema = UserSchema(many=True)
```

Выведите данные в свои представления.

```python
@app.route("/api/users/")
def users():
    all_users = User.all()
    return users_schema.dump(all_users)

@app.route("/api/users/<id>")
def user_detail(id):
    user = User.get(id)
    return user_schema.dump(user)

# {
#     "email": "fred@queen.com",
#     "date_created": "Fri, 25 Apr 2014 06:02:56 -0000",
#     "_links": {
#         "self": "/api/users/42",
#         "collection": "/api/users/"
#     }
# }
```

### Дополнительная интеграция Flask-SQLAlchemy

**Flask-Marshmallow** включает полезные дополнения для интеграции с [Flask-SQLAlchemy](http://flask-sqlalchemy.pocoo.org/) и [marshmallow-sqlalchemy](https://marshmallow-sqlalchemy.readthedocs.io/).

Чтобы включить интеграцию с SQLAlchemy, убедитесь, что установлены и Flask-SQLAlchemy, и marshmallow-sqlalchemy.

```bash
pip install -U flask-sqlalchemy marshmallow-sqlalchemy
```

Затем инициализируйте расширения [SQLAlchemy](https://flask-sqlalchemy.palletsprojects.com/en/2.x/api/#flask\_sqlalchemy.SQLAlchemy) и [Marshmallow](https://flask-marshmallow.readthedocs.io/en/latest/#flask\_marshmallow.Marshmallow) в указанном порядке.

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_marshmallow import Marshmallow

app = Flask(__name__)
app.config["SQLALCHEMY_DATABASE_URI"] = "sqlite:////tmp/test.db"

# Порядок имеет значение: инициализируйте SQLAlchemy до Marshmallow
db = SQLAlchemy(app)
ma = Marshmallow(app)
```

{% hint style="info" %}
**Примечание о порядке инициализации:**

Flask-SQLAlchemy **должен быть** инициализирован до Flask-Marshmallow.
{% endhint %}

Объявите свои модели как обычно.

```python
class Author(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(255))

class Book(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(255))
    author_id = db.Column(db.Integer, db.ForeignKey("author.id"))
    author = db.relationship("Author", backref="books")
```

Создавайте marshmallow [Schemas](https://marshmallow.readthedocs.io/en/latest/api\_reference.html#marshmallow.Schema) из своих моделей с помощью [SQLAlchemySchema](flask-marshmallow.md#sqlalchemyschema) или [SQLAlchemyAutoSchema](flask-marshmallow.md#sqlalchemyautoschema).

```python
class AuthorSchema(ma.SQLAlchemySchema):
    class Meta:
        model = Author

    id = ma.auto_field()
    name = ma.auto_field()
    books = ma.auto_field()

class BookSchema(ma.SQLAlchemyAutoSchema):
    class Meta:
        model = Book
        include_fk = True
```

Теперь вы можете использовать свою схему для создания дампа и загрузки объектов ORM.

```python
db.create_all()
author_schema = AuthorSchema()
book_schema = BookSchema()
author = Author(name="Chuck Paluhniuk")
book = Book(title="Fight Club", author=author)
db.session.add(author)
db.session.add(book)
db.session.commit()
author_schema.dump(author)
# {'id': 1, 'name': 'Chuck Paluhniuk', 'books': [1]}
```

API-интерфейс [SQLAlchemySchema](flask-marshmallow.md#sqlalchemyschema) почти идентичен [marshmallow\_sqlalchemy.SQLAlchemySchema](https://marshmallow-sqlalchemy.readthedocs.io/en/latest/api\_reference.html#marshmallow\_sqlalchemy.SQLAlchemySchema) со следующими исключениями:

* По умолчанию [SQLAlchemySchema](flask-marshmallow.md#sqlalchemyschema) использует сеанс с ограниченной областью действия, созданный **Flask-SQLAlchemy**.
* [SQLAlchemySchema](flask-marshmallow.md#sqlalchemyschema) является подклассом [flask\_marshmallow.Schema](flask-marshmallow.md#schema), поэтому он включает метод [jsonify](flask-marshmallow.md#jsonify).

{% hint style="info" %}
По умолчанию метод **jsonify** во Flask сортирует список ключей и возвращает согласованные результаты, чтобы гарантировать, что внешние кэши HTTP не будут уничтожены. В качестве побочного эффекта это переопределит **ordered = True** в классе **Meta** SQLAlchemySchema (если вы его установили). Чтобы отключить это, установите **JSON\_SORT\_KEYS=False** в конфигурации вашего приложения Flask. В производственной среде рекомендуется разрешить **jsonify** сортировать ключи, а не устанавливать в [SQLAlchemySchema](flask-marshmallow.md#sqlalchemyschema) **ordered=True**, чтобы минимизировать время генерации и максимизировать кешируемость результатов.
{% endhint %}

Вы также можете использовать поля [ma.HyperlinkRelated](flask-marshmallow.md#hyperlinkrelated), если хотите, чтобы отношения были представлены гиперссылками, а не первичными ключами.

```python
class BookSchema(ma.SQLAlchemyAutoSchema):
    class Meta:
        model = Book

    author = ma.HyperlinkRelated("author_detail")
```

```python
with app.test_request_context():
    print(book_schema.dump(book))
# {'id': 1, 'title': 'Fight Club', 'author': '/authors/1'}
```

Первым аргументом конструктора [HyperlinkRelated](flask-marshmallow.md#hyperlinkrelated) является имя представления, используемого для создания URL-адреса, точно так же, как вы передаете его функции [url\_for](https://flask.palletsprojects.com/en/2.0.x/api/#flask.url\_for). Если ваши модели и представления используют атрибут **id** в качестве первичного ключа, все готово; в противном случае необходимо указать имя атрибута, используемого в качестве первичного ключа.

Чтобы представить отношение «один ко многим», оберните экземпляр [HyperlinkRelated](flask-marshmallow.md#hyperlinkrelated) в поле [marshmallow.fields.List](https://marshmallow.readthedocs.io/en/latest/marshmallow.fields.html#marshmallow.fields.List), например:

```python
class AuthorSchema(ma.SQLAlchemyAutoSchema):
    class Meta:
        model = Author

    books = ma.List(ma.HyperlinkRelated("book_detail"))
```

```python
with app.test_request_context():
    print(author_schema.dump(author))
# {'id': 1, 'name': 'Chuck Paluhniuk', 'books': ['/books/1']}
```

## API

### flask\_marshmallow

Интегрирует библиотеку сериализации/десериализации marshmallow с вашим приложением Flask.

## Marshmallow

#### _class_ flask\_marshmallow.Marshmallow(_app=None_)

Класс-оболочка, который интегрирует Marshmallow с приложением Flask.

Чтобы использовать его, создайте экземпляр с помощью приложения:

```python
from flask import Flask

app = Flask(__name__)
ma = Marshmallow(app)
```

Объект предоставляет доступ к классу [Schema](flask-marshmallow.md#schema), всем полям в [marshmallow.fields](https://marshmallow.readthedocs.io/en/latest/marshmallow.fields.html#module-marshmallow.fields), а также полям, специфичным для Flask, в [flask\_marshmallow.fields](flask-marshmallow.md#flask\_marshmallow.fields).

Вы можете объявить схему следующим образом:

```python
class BookSchema(ma.Schema):
    class Meta:
        fields = ('id', 'title', 'author', 'links')

    author = ma.Nested(AuthorSchema)

    links = ma.Hyperlinks({
        'self': ma.URLFor('book_detail', values=dict(id='<id>')),
        'collection': ma.URLFor('book_list')
    })
```

Для интеграции с Flask-SQLAlchemy это расширение должно быть инициализировано **после** [flask\_sqlalchemy.SQLAlchemy](flask-sqlalchemy/api-flask-sqlalchemy.md#sqlalchemy).

```python
db = SQLAlchemy(app)
ma = Marshmallow(app)
```

Это дает вам доступ к **ma.SQLAlchemySchema** и **ma.SQLAlchemyAutoSchema**, которые генерируют классы marshmallow [Schema](https://marshmallow.readthedocs.io/en/latest/api\_reference.html#marshmallow.Schema) на основе переданной модели или таблицы.

```python
class AuthorSchema(ma.SQLAlchemyAutoSchema):
    class Meta:
        model = Author
```

#### Параметры:

* **app** (Flask) - Объект приложения Flask.

### init\_app(_app_)

Инициализирует приложение с расширением.

#### Параметры:

* **app** (Flask) - Объект приложения Flask.

## Schema

#### _class_ flask\_marshmallow.Schema(_\*_, _only: Optional\[Union\[Sequence\[str], Set\[str]]] = None_, _exclude: Union\[Sequence\[str], Set\[str]] = ()_, _many: bool = False_, _context: Optional\[Dict] = None_, _load\_only: Union\[Sequence\[str], Set\[str]] = ()_, _dump\_only: Union\[Sequence\[str], Set\[str]] = ()_, _partial: Union\[bool, Sequence\[str], Set\[str]] = False_, _unknown: Optional\[str] = None_)

Базовый сериализатор, с помощью которого можно определить пользовательские сериализаторы.

Дополнительные сведения об API схемы см. в [marshmallow.Schema](https://marshmallow.readthedocs.io/en/latest/api\_reference.html#marshmallow.Schema).

### jsonify()

#### jsonify(_obj_, _many=\<object object>_, _\*args_, _\*\*kwargs_)

Возвращает ответ JSON, содержащий сериализованные данные.

#### Параметры:

* **obj** - Объект для сериализации.
* **many** (_bool_) - Должен ли **obj** сериализоваться как экземпляр или как коллекция. Если не установлено, по умолчанию используется значение атрибута **many** в этой схеме.
* **kwargs** - Дополнительные аргументы ключевого слова, переданные в **flask.jsonify**.

_Изменено в версии 0.6.0_: принимает те же аргументы, что и [marshmallow.Schema.dump](https://marshmallow.readthedocs.io/en/latest/api\_reference.html#marshmallow.Schema.dump). Дополнительные аргументы ключевого слова передаются в **flask.jsonify**.

_Изменено в версии 0.6.3_: аргумент **many** для этого метода по умолчанию равен значению атрибута **many** в схеме. Ранее аргумент **many** этого метода по умолчанию имел значение `False`, независимо от значения **Schema.many**.

## pprint()

#### flask\_marshmallow.pprint(_obj_, _\*args_, _\*\*kwargs_) → None

Функция красивой печати, которая может красиво печатать **OrderedDicts**, как обычные словари. Полезно для печати вывода [marshmallow.Schema.dump()](https://marshmallow.readthedocs.io/en/latest/api\_reference.html#marshmallow.Schema.dump).

_<mark style="color:orange;">Устарело</mark>, начиная с версии 3.7.0_: `marshmallow.pprint` будет удален в **marshmallow 4**.

### flask\_marshmallow.fields

Пользовательские поля, специфичные для Flask.

См. модуль [marshmallow.fields](https://marshmallow.readthedocs.io/en/latest/marshmallow.fields.html#module-marshmallow.fields) для получения списка всех полей, доступных в библиотеке **marshmallow**.

## AbsoluteURLFor

#### _class_ flask\_marshmallow.fields.AbsoluteURLFor(_endpoint_, _values=None_, _\*\*kwargs_)

Поле, которое выводит абсолютный URL-адрес для конечной точки.

## AbsoluteUrlFor

псевдоним [flask\_marshmallow.fields.AbsoluteURLFor](flask-marshmallow.md#absoluteurlfor)

## Hyperlinks

#### _class_ flask\_marshmallow.fields.Hyperlinks(_schema_, _\*\*kwargs_)

Поле, которое выводит словарь гиперссылок, учитывая схему словаря с объектами [URLFor](flask-marshmallow.md#urlfor) в качестве значений.

Пример:

```python
_links = Hyperlinks({
    'self': URLFor('author', values=dict(id='<id>')),
    'collection': URLFor('author_list'),
})
```

Объекты <mark style="color:red;">URLFor</mark> могут быть вложены в словарь.

```python
_links = Hyperlinks({
    'self': {
        'href': URLFor('book', values=dict(id='<id>')),
        'title': 'book detail'
    }
})
```

#### Параметры:

* **schema** (_dict_) - словарь, который сопоставляет имена с полями URLFor.

## URLFor

#### _class_ flask\_marshmallow.fields.URLFor(_endpoint_, _values=None_, _\*\*kwargs_)

Поле, которое выводит URL-адрес для конечной точки. Действует аналогично функции Flask **url\_for**, за исключением того, что аргументы могут быть извлечены из объекта для сериализации, а **\*\*values** должны быть переданы в параметр **values**.

Использование:

```python
url = URLFor('author_get', values=dict(id='<id>'))
https_url = URLFor(
    'author_get',
    values=dict(
        id='<id>',
        _scheme='https',
        _external=True
    )
)
```

#### Параметры:

* **endpoint** (_str_) - Имя конечной точки Flask.
* **values** (_dict_) - Те же аргументы ключевого слова, что и у Flask **url\_for**, за исключением строковых аргументов, заключенных в `< >`, которые будут интерпретироваться как атрибуты, которые нужно извлечь из объекта.
* **kwargs** - аргументы ключевого слова для передачи в поле marshmallow (например, **required**).

## UrlFor

#### flask\_marshmallow.fields.UrlFor

псевдоним [flask\_marshmallow.fields.URLFor](flask-marshmallow.md#urlfor)

### flask\_marshmallow.sqla

Интеграция с **Flask-SQLAlchemy** и **marshmallow-sqlalchemy**. Предоставляет классы [SQLAlchemySchema](https://marshmallow-sqlalchemy.readthedocs.io/en/latest/api\_reference.html#marshmallow\_sqlalchemy.SQLAlchemySchema) и [SQLAlchemyAutoSchema](https://marshmallow-sqlalchemy.readthedocs.io/en/latest/api\_reference.html#marshmallow\_sqlalchemy.SQLAlchemyAutoSchema), которые используют сеанс с заданной областью действия из Flask-SQLAlchemy.

## DummySession

#### _class_ flask\_marshmallow.sqla.DummySession

Объект сеанса заполнителя.

## HyperlinkRelated

#### _class_ flask\_marshmallow.sqla.HyperlinkRelated(_endpoint_, _url\_key='id'_, _external=False_, _\*\*kwargs_)

Поле, генерирующее гиперссылки для указания ссылок между моделями, а не первичных ключей.

#### Параметры:

* **endpoint** (_str_) - Имя конечной точки Flask для сгенерированной гиперссылки.
* **url\_key** (_str_) - Атрибут, содержащий первичный ключ ссылки. По умолчанию `«id»`.
* **external** (_str_) - Установите значение `True`, если следует использовать абсолютные URL-адреса вместо относительных URL-адресов.

## SQLAlchemyAutoSchema

#### _class_ flask\_marshmallow.sqla.SQLAlchemyAutoSchema(_\*args_, _\*\*kwargs_)

SQLAlchemyAutoSchema, которая автоматически генерирует поля marshmallow из столбца модели или таблицы SQLAlchemy. По умолчанию использует сеанс с ограниченной областью действия из Flask-SQLAlchemy.

Дополнительные сведения об API **SQLAlchemyAutoSchema** см. в статье [marshmallow\_sqlalchemy.SQLAlchemyAutoSchema](https://marshmallow-sqlalchemy.readthedocs.io/en/latest/api\_reference.html#marshmallow\_sqlalchemy.SQLAlchemyAutoSchema).

### OPTIONS\_CLASS

псевдоним [flask\_marshmallow.sqla.SQLAlchemyAutoSchemaOpts](flask-marshmallow.md#sqlalchemyautoschemaopts)

## SQLAlchemyAutoSchemaOpts

#### _class_ flask\_marshmallow.sqla.SQLAlchemyAutoSchemaOpts(_meta_, _\*\*kwargs_)

## SQLAlchemySchema

#### _class_ flask\_marshmallow.sqla.SQLAlchemySchema(_\*args_, _\*\*kwargs_)

SQLAlchemySchema, которая связывает схему с моделью через параметр **Meta** класса **model**, который должен быть классом **db.Model** из [flask\_sqlalchemy](https://flask-sqlalchemy.palletsprojects.com/en/2.x/api/#module-flask\_sqlalchemy). По умолчанию использует сеанс с ограниченной областью действия из Flask-SQLAlchemy.

Дополнительные сведения об API **SQLAlchemySchema** см. в статье [marshmallow\_sqlalchemy.SQLAlchemySchema](https://marshmallow-sqlalchemy.readthedocs.io/en/latest/api\_reference.html#marshmallow\_sqlalchemy.SQLAlchemySchema).

### OPTIONS\_CLASS

псевдоним [flask\_marshmallow.sqla.SQLAlchemySchemaOpts](flask-marshmallow.md#sqlalchemyautoschemaopts-1)

## SQLAlchemySchemaOpts

#### _class_ flask\_marshmallow.sqla.SQLAlchemySchemaOpts(_meta_, _\*\*kwargs_)

## Полезные ссылки

## Информация о проекте
