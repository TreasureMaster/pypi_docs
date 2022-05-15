# Расширение Flask-RESTful

Мы понимаем, что у всех разные потребности в среде REST. **Flask-RESTful** старается быть максимально гибким, но иногда вы можете обнаружить, что встроенной функциональности недостаточно для удовлетворения ваших потребностей. **Flask-RESTful** имеет несколько различных точек расширения, которые могут помочь в этом случае.

## Переговоры о содержании

По умолчанию **Flask-RESTful** настроен только на поддержку **JSON**. Мы приняли это решение, чтобы предоставить сопровождающим API полный контроль над поддержкой формата API; так что через год вам не придется поддерживать людей, использующих CSV-представление вашего API, о существовании которого вы даже не подозревали. Чтобы добавить дополнительные типы мультимедиа в ваш API, вам необходимо объявить поддерживаемые представления в объекте <mark style="color:red;">Api</mark>.

```python
app = Flask(__name__)
api = Api(app)

@api.representation('application/json')
def output_json(data, code, headers=None):
    resp = make_response(json.dumps(data), code)
    resp.headers.extend(headers or {})
    return resp
```

Эти функции представления должны возвращать объект Flask [Response](https://flask.palletsprojects.com/en/1.1.x/api/#flask.Response).

{% hint style="info" %}
**Flask-RESTful** использует модуль [json](https://docs.python.org/3/library/json.html#module-json) из стандартной библиотеки Python вместо [flask.json](https://flask.palletsprojects.com/en/1.1.x/api/#module-flask.json), потому что сериализатор Flask JSON включает возможности сериализации, которых нет в спецификации JSON. Если вашему приложению нужны эти настройки, вы можете заменить представление JSON по умолчанию на представление JSON с помощью модуля Flask JSON, как описано выше.
{% endhint %}

Можно настроить, как представление JSON **Flask-RESTful** по умолчанию будет форматировать JSON, указав атрибут **RESTFUL\_JSON** в конфигурации приложения. Этот параметр представляет собой словарь с ключами, соответствующими аргументам ключевого слова [json.dumps()](https://docs.python.org/3/library/json.html#json.dumps).

```python
class MyConfig(object):
    RESTFUL_JSON = {'separators': (', ', ': '),
                    'indent': 2,
                    'cls': MyCustomEncoder}
```

{% hint style="info" %}
Если приложение работает в режиме отладки (`app.debug = True`) и либо **sort\_keys**, либо **indent** не объявлены в параметре конфигурации **RESTFUL\_JSON**, Flask-RESTful предоставит значения по умолчанию **True** и **4** соответственно.
{% endhint %}

## Пользовательские поля и входные данные

Одним из наиболее распространенных дополнений к **Flask-RESTful** является определение пользовательских типов или полей на основе ваших собственных типов данных.

### Поля

Пользовательские поля вывода позволяют выполнять собственное форматирование вывода без необходимости напрямую изменять внутренние объекты. Все, что вам нужно сделать, это создать подкласс <mark style="color:red;">Raw</mark> и реализовать метод <mark style="color:red;">format()</mark>:

```python
class AllCapsString(fields.Raw):
    def format(self, value):
        return value.upper()

# пример использования
fields = {
    'name': fields.String,
    'all_caps_name': AllCapsString(attribute=name),
}
```

### Входные данные

Для синтаксического анализа аргументов может потребоваться выполнить пользовательскую проверку. Создание собственных типов ввода позволяет с легкостью расширить синтаксический анализ запросов.

```python
def odd_number(value):
    if value % 2 == 0:
        raise ValueError("Value is not odd")

    return value
```

Парсер запросов также предоставит вам доступ к имени аргумента в случаях, когда вы хотите сослаться на имя в сообщении об ошибке.

```python
def odd_number(value, name):
    if value % 2 == 0:
        raise ValueError(
            "The parameter '{}' is not odd. You gave us the value: {}".format(
                name, value
            )
        )

    return value
```

Вы также можете преобразовать значения общедоступных параметров во внутренние представления:

```python
# отображает строки в их внутреннее целочисленное представление
# 'init' => 0
# 'in-progress' => 1
# 'completed' => 2

def task_status(value):
    statuses = [u"init", u"in-progress", u"completed"]
    return statuses.index(value)
```

Затем вы можете использовать эти пользовательские типы ввода в своем <mark style="color:red;">RequestParser</mark>:

```python
parser = reqparse.RequestParser()
parser.add_argument('OddNumber', type=odd_number)
parser.add_argument('Status', type=task_status)
args = parser.parse_args()
```

## Форматы ответов

Для поддержки других представлений (xml, csv, html) вы можете использовать декоратор <mark style="color:red;">representation()</mark>. У вас должна быть ссылка на ваш API.

```python
api = Api(app)

@api.representation('text/csv')
def output_csv(data, code, headers=None):
    pass
    # реализовать вывод csv!
```

Эти выходные функции принимают три параметра: **data**, **code** и **headers**.

**data** — это объект, который вы возвращаете из своего метода ресурса, **code** — это код состояния HTTP, который он ожидает, а **headers** — это любые заголовки HTTP, которые нужно установить в ответе. Ваша функция вывода должна возвращать объект [flask.Response](https://flask.palletsprojects.com/en/1.1.x/api/#flask.Response).

```python
def output_json(data, code, headers=None):
    """Делает ответ Flask с кодированным телом JSON"""
    resp = make_response(json.dumps(data), code)
    resp.headers.extend(headers or {})
    return resp
```

Другой способ добиться этого — создать подкласс класса API и предоставить свои собственные функции вывода.

```python
class Api(restful.Api):
    def __init__(self, *args, **kwargs):
        super(Api, self).__init__(*args, **kwargs)
        self.representations = {
            'application/xml': output_xml,
            'text/html': output_html,
            'text/csv': output_csv,
            'application/json': output_json,
        }
```

## Декораторы методов ресурсов

В классе <mark style="color:red;">Resource</mark> есть свойство, называемое **method\_decorators**. Вы можете создать подкласс ресурса и добавить свои собственные декораторы, которые будут добавлены ко всем функциям метода в ресурсе. Например, если вы хотите встроить пользовательскую аутентификацию в каждый запрос.

```python
def authenticate(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        if not getattr(func, 'authenticated', True):
            return func(*args, **kwargs)

        acct = basic_authentication()  # функция поиска пользовательской учетной записи

        if acct:
            return func(*args, **kwargs)

        flask_restful.abort(401)
    return wrapper


class Resource(flask_restful.Resource):
    method_decorators = [authenticate]   # применяется ко всем унаследованным ресурсам
```

В качестве альтернативы вы можете указать словарь итерируемых объектов, которые сопоставляются с методами HTTP, и декораторы будут применяться только к соответствующим запросам.

```python
def cache(f):
    @wraps(f)
    def cacher(*args, **kwargs):
        # кеширование
    return cacher

class MyResource(restful.Resource):
    method_decorators = {'get': [cache]}

     def get(self, *args, **kwargs):
        return something_interesting(*args, **kwargs)

     def post(self, *args, **kwargs):
        return create_something(*args, **kwargs)
```

В этом случае декоратор кэширования будет применяться только к запросу **GET**, а не к запросу **POST**.

Поскольку ресурсы **Flask-RESTful** на самом деле являются объектами представления Flask, вы также можете использовать [стандартные декораторы представления flask](http://flask.pocoo.org/docs/views/#decorating-views).

## Пользовательские обработчики ошибок

Обработка ошибок — сложная задача. Ваше приложение Flask может носить несколько шляп, но вы хотите обрабатывать все ошибки **Flask-RESTful** с правильным типом содержимого и синтаксисом ошибок как ваши запросы уровня **200**.

**Flask-RESTful** вызовет функцию <mark style="color:red;">handle\_error()</mark> при любой ошибке **400** или **500**, возникающей на маршруте **Flask-RESTful**, и оставит другие маршруты в покое. Вы можете захотеть, чтобы ваше приложение возвращало сообщение об ошибке с правильным типом носителя при ошибках **404 Not Found**; в этом случае используйте параметр **catch\_all\_404s** конструктора <mark style="color:red;">Api</mark>.

```python
app = Flask(__name__)
api = flask_restful.Api(app, catch_all_404s=True)
```

Тогда **Flask-RESTful** будет обрабатывать ошибки **404** в дополнение к ошибкам на своих маршрутах.

Иногда вы хотите сделать что-то особенное, когда возникает ошибка — войти в файл, отправить электронное письмо и т. д. Используйте метод **got\_request\_exception()**, чтобы прикрепить пользовательские обработчики ошибок к исключению.

```python
def log_exception(sender, exception, **extra):
    """ Зарегистрируйте исключение в нашей системе ведения журнала """
    sender.logger.debug('Got exception during processing: %s', exception)

from flask import got_request_exception
got_request_exception.connect(log_exception, app)
```

### Определить пользовательские сообщения об ошибках

Вы можете захотеть вернуть конкретное сообщение и/или код состояния, когда во время запроса возникают определенные ошибки. Вы можете указать **Flask-RESTful**, как вы хотите обрабатывать каждую ошибку/исключение, чтобы вам не пришлось заполнять код API блоками `try/except`.

```python
errors = {
    'UserAlreadyExistsError': {
        'message': "A user with that username already exists.",
        'status': 409,
    },
    'ResourceDoesNotExist': {
        'message': "A resource with that ID no longer exists.",
        'status': 410,
        'extra': "Any extra information you want.",
    },
}
```

Включение ключа «**status**» установит код состояния ответа. Если не указано, по умолчанию будет **500**.

Как только ваш словарь ошибок определен, просто передайте его конструктору <mark style="color:red;">Api</mark>.

```python
app = Flask(__name__)
api = flask_restful.Api(app, errors=errors)
```

{% hint style="info" %}
Пользовательские _исключения_ должны иметь [HTTPException](https://werkzeug.palletsprojects.com/en/0.16.x/exceptions/#werkzeug.exceptions.HTTPException) в качестве базового исключения **Exception**.
{% endhint %}
