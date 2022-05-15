# Промежуточное использование

На этой странице рассказывается о создании немного более сложного приложения **Flask-RESTful**, в котором будут рассмотрены некоторые рекомендации по настройке реального API на основе **Flask-RESTful**. Раздел [Quickstart](bystryi-start-flask-restful.md) отлично подходит для начала работы с вашим первым приложением Flask-RESTful, поэтому, если вы новичок в Flask-RESTful, вам лучше сначала проверить это.

## Структура проекта

Существует множество различных способов организовать ваше приложение **Flask-RESTful**, но здесь мы опишем тот, который довольно хорошо масштабируется с более крупными приложениями и поддерживает организацию на хорошем уровне.

Основная идея состоит в том, чтобы разделить ваше приложение на три основные части: маршруты, ресурсы и любую общую инфраструктуру.

Вот пример структуры каталогов:

```bash
myapi/
    __init__.py
    app.py          # этот файл содержит ваше приложение и маршруты
    resources/
        __init__.py
        foo.py      # содержит логику для /Foo
        bar.py      # содержит логику для /Bar
    common/
        __init__.py
        util.py     # просто общая инфраструктура
```

Общий каталог, вероятно, будет просто содержать набор вспомогательных функций для удовлетворения общих потребностей вашего приложения. Он также может содержать, например, любые настраиваемые типы ввода/вывода, необходимые вашим ресурсам для выполнения работы.

В файлах ресурсов у вас есть только объекты ресурсов. Итак, вот как может выглядеть `foo.py`:

```python
from flask_restful import Resource

class Foo(Resource):
    def get(self):
        pass
    def post(self):
        pass
```

Ключ к этой настройке лежит в `app.py`:

```python
from flask import Flask
from flask_restful import Api
from myapi.resources.foo import Foo
from myapi.resources.bar import Bar
from myapi.resources.baz import Baz

app = Flask(__name__)
api = Api(app)

api.add_resource(Foo, '/Foo', '/Foo/<string:id>')
api.add_resource(Bar, '/Bar', '/Bar/<string:id>')
api.add_resource(Baz, '/Baz', '/Baz/<string:id>')
```

Как вы можете себе представить, в особенно большом или сложном API этот файл оказывается очень ценным в качестве исчерпывающего списка всех маршрутов и ресурсов в вашем API. Вы также можете использовать этот файл для установки любых значений конфигурации ([before\_request()](https://flask.palletsprojects.com/en/1.1.x/api/#flask.Flask.before\_request), [after\_request()](https://flask.palletsprojects.com/en/1.1.x/api/#flask.Flask.after\_request)). По сути, этот файл настраивает весь ваш API.

Вещи в общем каталоге — это просто вещи, которые вы хотели бы поддерживать в своих модулях ресурсов.

## Использование с шаблонами (blueprints)

См. «[Модульные приложения с чертежами](https://flask.palletsprojects.com/en/1.1.x/blueprints/#blueprints)» в документации **Flask**, чтобы узнать, что такое чертежи и почему вы должны их использовать. Вот пример того, как связать <mark style="color:red;">Api</mark> с [Blueprint](https://flask.palletsprojects.com/en/1.1.x/api/#flask.Blueprint).

```python
from flask import Flask, Blueprint
from flask_restful import Api, Resource, url_for

app = Flask(__name__)
api_bp = Blueprint('api', __name__)
api = Api(api_bp)

class TodoItem(Resource):
    def get(self, id):
        return {'task': 'Say "Hello, World!"'}

api.add_resource(TodoItem, '/todos/<int:id>')
app.register_blueprint(api_bp)
```

{% hint style="info" %}
Вызов <mark style="color:red;">Api.init\_app()</mark> здесь не требуется, поскольку регистрация схемы в приложении обеспечивает настройку маршрутизации для приложения.
{% endhint %}

## Пример полного анализа параметров

В другом месте документации мы подробно описали, как использовать пример **reqparse**. Здесь мы настроим ресурс с несколькими входными параметрами, которые реализуют большее количество опций. Мы определим ресурс с именем «**User**».

```python
from flask_restful import fields, marshal_with, reqparse, Resource

def email(email_str):
    """Возвращает email_str, если он действителен, в противном случае вызывает исключение."""
    if valid_email(email_str):
        return email_str
    else:
        raise ValueError('{} is not a valid email'.format(email_str))

post_parser = reqparse.RequestParser()
post_parser.add_argument(
    'username', dest='username',
    location='form', required=True,
    help='The user\'s username',
)
post_parser.add_argument(
    'email', dest='email',
    type=email, location='form',
    required=True, help='The user\'s email',
)
post_parser.add_argument(
    'user_priority', dest='user_priority',
    type=int, location='form',
    default=1, choices=range(5), help='The user\'s priority',
)

user_fields = {
    'id': fields.Integer,
    'username': fields.String,
    'email': fields.String,
    'user_priority': fields.Integer,
    'custom_greeting': fields.FormattedString('Hey there {username}!'),
    'date_created': fields.DateTime,
    'date_updated': fields.DateTime,
    'links': fields.Nested({
        'friends': fields.Url('user_friends'),
        'posts': fields.Url('user_posts'),
    }),
}

class User(Resource):

    @marshal_with(user_fields)
    def post(self):
        args = post_parser.parse_args()
        user = create_user(args.username, args.email, args.user_priority)
        return user

    @marshal_with(user_fields)
    def get(self, id):
        args = post_parser.parse_args()
        user = fetch_user(id)
        return user
```

Как видите, мы создаем **post\_parser** специально для обработки аргументов, предоставленных в **POST**. Давайте пройдемся по определению каждого аргумента.

```python
post_parser.add_argument(
    'username', dest='username',
    location='form', required=True,
    help='The user\'s username',
)
```

Поле имени пользователя **username** самое обычное из всех. Оно берет строку из тела POST и преобразует ее в строковый тип. Этот аргумент обязателен (`required=True`), что означает, что если он не указан, **Flask-RESTful** автоматически вернет **400** с сообщением типа «_the username field is required_».

```python
post_parser.add_argument(
    'email', dest='email',
    type=email, location='form',
    required=True, help='The user\'s email',
)
```

Поле электронной почты **email** имеет настраиваемый тип электронной почты. Несколькими строками ранее мы определили функцию электронной почты `email()`, которая принимает строку и возвращает ее, если тип действителен, в противном случае она вызывает исключение, сообщая, что тип электронной почты недействителен.

```python
post_parser.add_argument(
    'user_priority', dest='user_priority',
    type=int, location='form',
    default=1, choices=range(5), help='The user\'s priority',
)
```

Тип **user\_priority** использует аргумент **choices**. Это означает, что если предоставленное значение **user\_priority** не попадает в диапазон, указанный аргументом **choices** (в данном случае `[0, 1, 2, 3, 4]`), **Flask-RESTful** автоматически ответит **400** и описанием ошибки сообщения.

Это покрывает входы. Мы также определили некоторые интересные типы полей в словаре **user\_fields**, чтобы продемонстрировать несколько более экзотических типов.

```python
user_fields = {
    'id': fields.Integer,
    'username': fields.String,
    'email': fields.String,
    'user_priority': fields.Integer,
    'custom_greeting': fields.FormattedString('Hey there {username}!'),
    'date_created': fields.DateTime,
    'date_updated': fields.DateTime,
    'links': fields.Nested({
        'friends': fields.Url('user_friends', absolute=True),
        'posts': fields.Url('user_friends', absolute=True),
    }),
}
```

Во-первых, это <mark style="color:red;">fields.FormattedString</mark>.

```python
'custom_greeting': fields.FormattedString('Hey there {username}!'),
```

Это поле в основном используется для интерполяции значений из ответа в другие значения. В этом случае **custom\_greeting** всегда будет содержать значение, возвращаемое из поля **username**.

Затем проверьте <mark style="color:red;">fields.Nested</mark>.

```python
'links': fields.Nested({
    'friends': fields.Url('user_friends', absolute=True),
    'posts': fields.Url('user_posts', absolute=True),
}),
```

Это поле используется для создания подобъекта в ответе. В этом случае мы хотим создать подобъект ссылок, содержащий URL-адреса связанных объектов. Обратите внимание, что мы передали **fields.Nested** еще один **dict**, который построен таким образом, что сам по себе является приемлемым аргументом для <mark style="color:red;">marshal()</mark>.

Наконец, мы использовали тип поля <mark style="color:red;">fields.Url</mark>.

```python
'friends': fields.Url('user_friends', absolute=True),
'posts': fields.Url('user_friends', absolute=True),
```

Он принимает в качестве своего первого параметра имя конечной точки, связанной с URL-адресами объектов в подобъекте ссылок. Передача `absolute=True` гарантирует, что сгенерированные URL-адреса будут включать имя хоста.

## Передача параметров конструктора в ресурсы

Реализация <mark style="color:red;">Resource</mark> может потребовать внешних зависимостей. Эти зависимости лучше всего передавать через конструктор, чтобы слабо связать друг друга. Метод <mark style="color:red;">Api.add\_resource()</mark> имеет два аргумента ключевого слова: **resource\_class\_args** и **resource\_class\_kwargs**. Их значения будут перенаправлены и переданы в конструктор реализации вашего ресурса.

Итак, у вас может быть <mark style="color:red;">Resource</mark>:

```python
from flask_restful import Resource

class TodoNext(Resource):
    def __init__(self, **kwargs):
        # smart_engine — это зависимость от черного ящика
        self.smart_engine = kwargs['smart_engine']

    def get(self):
        return self.smart_engine.next_todo()
```

Вы можете внедрить необходимую зависимость в **TodoNext** следующим образом:

```python
smart_engine = SmartEngine()

api.add_resource(TodoNext, '/next',
    resource_class_kwargs={ 'smart_engine': smart_engine })
```

Та же идея применяется для пересылки _**args**_.
