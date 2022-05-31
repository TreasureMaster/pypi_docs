# Flask-ApiExceptions

**Flask-ApiExceptions** — это расширение Flask, предоставляющее базовые функции для сериализации неперехваченных исключений в виде ответов HTTP для REST API на основе JSON.

## Установка

Вы можете установить это расширение с помощью **pip**:

```bash
$ pip install flask_apiexceptions
```

Или вы можете клонировать репозиторий:

```bash
$ git clone https://github.com/jperras/flask_apiexceptions.git
```

## Запуск тестов

[Tox](https://pypi.python.org/pypi/tox) используется для запуска тестов, которые написаны с использованием [PyTest](https://docs.pytest.org/en/latest/). Чтобы запустить их, клонируйте репозиторий (указанный выше), убедитесь, что **tox** установлен и доступен, и запустите:

```bash
$ cd path/to/flask_apiexceptions
$ tox
```

## Использование

Этот пакет включает расширение с именем **JSONExceptionHandler**, которое можно добавить в ваше приложение обычным способом:

```python
from flask import Flask
from flask_apiexceptions import JSONExceptionHandler

app = Flask(__name__)
exception_handler = JSONExceptionHandler(app)
```

Расширение также можно инициализировать с помощью отложенной инициализации приложения, если вы используете фабрику приложений:

```python
exception_handler = JSONExceptionHandler()
exception_hander.init_app(app)
```

После инициализации расширение фактически ничего не делает по умолчанию. Вам нужно будет настроить его для обработки кодов ошибок **Werkzeug HTTP** или пользовательских классов исключений **Exception**.

### Обработка пользовательского класса исключений

Пример, показывающий, как мы можем вызвать пользовательское исключение в методе представления и преобразовать это исключение в ответ JSON:

```python
class MissingUserError(Exception):
    status_code = 404
    message = 'No such user exists.'

@app.route('/not-found')
def testing():
    raise MissingUserError()

ext = JSONExceptionHandler(app)
ext.register(code_or_exception=MissingUserError)

with app.app_context():
    with app.test_client() as c:
        rv = c.get('/not-found')

assert rv.status_code == 404
assert rv.headers['content-type'] == 'application/json'
assert json.loads(rv.data)['message'] == 'No such user exists.'
```

При этом используется `JSONExceptionHandler.default_handler()` для преобразования класса исключения **CustomError** в подходящий ответ. Он пытается проанализировать экземпляр исключения, возвращенный для атрибута **message** или **description**, а также проверяет, существует ли атрибут **status\_code**.

Если какое-либо из этих полей найдено, обработчик по умолчанию заполнит данные ответа данным сообщением и установит код состояния ответа. Если сообщение или код состояния отсутствуют, по умолчанию устанавливается ответ `{"message": "An error occurred!"}` с кодом состояния `HTTP/1.1 500 Internal Server Error`.

Если вы хотите обрабатывать пользовательские классы исключений по-другому, например, потому что у вас есть более сложные данные, захваченные в экземпляре исключения, или атрибуты не имеют удобного имени **message** или **description**, вы можете указать собственный обработчик для типа исключения. :

```python
from flask_apiexceptions import JSONExceptionHandler

app = Flask(__name__)
ext = JSONExceptionHandler(app)

class CaffeineError(Exception):
    teapot_code = 418
    special = {'foo': 'bar'}

def caffeine_handler(error):
    response = jsonify(data=error.special)
    response.status_code = error.teapot_code
    return response

@app.route('/testing')
def testing():
    raise CaffeineError()

ext.register(code_or_exception=CaffeineError, handler=caffeine_handler)

with app.app_context():
    with app.test_client() as c:
        rv = c.get('/testing')

assert rv.status_code == 418
assert rv.headers['content-type'] == 'application/json'
assert json.loads(rv.data)['data'] == CaffeineError.special
```

Кстати, именно так вы можете использовать тип содержимого ответа, отличный от `application/json`. Просто создайте свой собственный объект ответа вместо использования `jsonify()` в вашем обработчике, если он выдает действительный ответ в качестве возвращаемого значения.

## Использование объектов ApiException и ApiError

**Flask-ApiExceptions** включает в себя несколько удобных классов и метод-обработчик для настройки структурированных ответов API на ошибки. Они совершенно необязательны, но предоставляют некоторые разумные значения по умолчанию, которые должны охватывать большинство ситуаций.

Экземпляр **ApiException** упаковывает один или несколько экземпляров **ApiError**. В этом смысле **ApiException** — это просто контейнер для фактического сообщения об ошибке. Экземпляр **ApiError** принимает необязательные атрибуты **code**, **message** и **info**.

Идея состоит в том, что **code** должен быть идентификатором типа ошибки, например, **invalid-data** или **does-not-exist**. Поле **message** должно содержать более подробное и точное описание ошибки. Информационное поле **info** может использоваться для любых дополнительных метаданных или неструктурированной информации, которые могут потребоваться.

Информационное поле **info**, если оно используется, должно содержать данные, сериализуемые в формате JSON.

Чтобы использовать эти конструкции, вам необходимо зарегистрировать соответствующий класс исключений, а также **api\_exception\_handler**, предназначенный именно для этой цели:

```python
from flask_apiexceptions import (
    JSONExceptionHandler, ApiException, ApiError, api_exception_handler)

app = Flask(__name__)
ext = JSONExceptionHandler(app)
ext.register(code_or_exception=ApiException, handler=api_exception_handler)

@app.route('/custom')
def testing():
    error = ApiError(code='teapot', message='I am a little teapot.')
    raise ApiException(status_code=418, error=error)

with app.app_context():
    with app.test_client() as c:
        rv = c.get('/custom')

        # Ответ JSON выглядит так...
        # {"errors": [{"code": "teapot", "message": "I am a little teapot."}]}

assert rv.status_code == 418
assert rv.headers['content-type'] == 'application/json'

json_data = json.loads(rv.data)
assert json_data['errors'][0]['message'] == 'I am a little teapot.'
assert json_data['errors'][0]['code'] == 'teapot'
assert json_data['errors'][0]['info'] is None
```

Обратите внимание, что при использовании классов **ApiException** и **ApiError** код состояния устанавливается для экземпляра **ApiException**. Это имеет больше смысла, когда вы можете установить несколько объектов **ApiError** для одного и того же **ApiException**:

```python
from flask_apiexceptions import ApiException, ApiError

# ...

@app.route('/testing')
def testing():
    exc = ApiException(status_code=400)
    invalid_address_error = ApiError(code='invalid-data',
                                     message='The address provided is invalid.')
    invalid_phone_error = ApiError(code='invalid-data',
                                   message='The phone number does not exist.',
                                   info={'area_code': '555'})
    exc.add_error(invalid_address_error)
    exc.add_error(invalid_phone_error)

    raise exc

    # Формат ответа JSON:
    # {"errors": [
    #     {"code": "invalid-data", "message": "The address provided is invalid."},
    #     {"code": "invalid-data", "message": "The phone number does not exist.", "info": {"area_code": "444"}}
    # ]}
```

Если вы хотите, чтобы в **ApiException** была создана только одна ошибка **error**, это можно сделать с помощью конструктора последнего в виде сокращения:

```python
exc = ApiException(
    status_code=400,
    code='invalid-data',
    message='The address provided is invalid',
    info={'zip_code': '90210'})
```

что эквивалентно:

```python
exc = ApiException(status_code=400)
error=ApiError(
    code='invalid-data',
    message='The address provided is invalid',
    info={'zip_code': '90210'}))

exc.add_error(error)
```

Полезным шаблоном является создание подкласса **ApiException** в явно полезные типы исключений, для которых вы можете определить атрибуты уровня класса по умолчанию, которые будут использоваться для заполнения правильного объекта ошибки **error** при создании экземпляра. Например:

```python
class MissingResourceError(ApiException):
    status_code = 404
    message = "No such resource exists."
    code = 'not-found'

# ...

@app.route('/posts/<int:post_id>')
def post_by_id(post_id):
    """Получить один пост по идентификатору из базы данных."""

    post = Post.query.filter(Post.id == post_id).one_or_none()
    if post is None:
        raise MissingResourceError()

    # Ответ 404 с телом JSON:
    # {"errors": [
    #     {"code": "not-found", "message": "No such resource exists."}
    # ]}
```

Преимущество этого конкретного шаблона в том, что вы можете создавать _**семантически корректные**_ исключения в своей кодовой базе и можете обрабатывать их в стеке вызовов. Если вы их не обрабатываете, они просто передаются в обработчик исключений (если вы настроили `flask_apiexceptions.api_exception_handler` или аналогичный), зарегистрированный в **Flask**, и затем преобразуются в полезный ответ для запрашивающего клиента.

```python
class MissingResourceError(ApiException):
    status_code = 404
    message = "No such resource exists."
    code = 'not-found'

class Post(db.Model):
    # ...
    @classmethod
    def query_by_id(cls, post_id):
        """Запросить сообщение по id, вызвать исключение, если оно не найдено."""
        result = cls.query.filter(cls.id == post_id).one_or_none()
        if result is None:
            raise MissingResourceError()

        return result

@app.route('/posts/<int:post_id>')
def post_by_id(post_id):
    """Получить один пост по ID из базы данных."""

    try:
        post = Post.query_by_id(post_id)
    except MissingResourceError as e:
        # Мы можем делать все, что захотим, теперь, когда мы поймали исключение.
        # Для иллюстрации мы просто запишем это.
        app.logger.exception("Could not locate post!")

        # Будет всплывать исключение до тех пор,
        # пока оно не будет преобразовано в JSON для клиента.
        raise e
```

## API

Базовые классы исключений API.

## JSONExceptionHandler

#### class JSONExceptionHandler(object)

Расширение **Flask**, которое преобразует исключения Flask по умолчанию в эквивалентные им типы содержимого `application/json`.

```python
from application.libs import JSONExceptionHandler
exception_handler = JSONExceptionHandler()
exception_handler.init_app(app)
```

Если мы не знаем, какой HTTP-код назначить исключению, по умолчанию мы назначаем ему `500`. Это также обрабатывает не перехваченные исключения; например если наше приложение вызывает какой-либо подкласс **Exception**, для которого у нас нет явного обработчика, то мы, вероятно, где-то получили ошибку приложения для этого конкретного пути кода.

### \_\_init\_\_(app=None)

Инициализирует расширение.

Любые конфигурации по умолчанию, которые не требуют экземпляра приложения, должны быть помещены сюда.

### default\_handler(error=None)

Обработчик ошибок по умолчанию для регистрации в приложении.

Если объект **error** содержит атрибут **message**, то используем его как сообщение для нашего исключения. Если атрибута **message** нет, то создаем тип исключений Werkzeug, которые по умолчанию используют **description** вместо **message**.

Если наш объект **error** содержит определенный код ошибки **status\_code**, то используем его. Если нет, мы будем использовать наш **default\_status\_code**, который был определен для этого класса. Это гарантирует, что случайные исключения, создаваемые Python или внешними библиотеками, которые мы пропустили, являются ошибкой приложения.

### init\_app(app)

Инициализирует расширение с любыми требованиями к конфигурации уровня приложения.

Здесь мы регистрируем класс Werkzeug `HTTPException` вместе со всеми другими кодами исключений по умолчанию.

### register(code\_or\_exception, handler=None)

Регистрирует класс исключения _или_ числовой код с обработчиком исключения по умолчанию, предоставляемым этим расширением _или_ функцией, предоставленной в **handler** в аргументе.

### handle\_404(error=None)

Обработчик **Werkzeug 404** по умолчанию не включает сообщение **message** или описание **description**, что вызывает некоторые проблемы согласованности с нашими интерфейсами при получении 404.

## ApiError

#### class ApiError(object)

Содержит информацию, связанную с ошибкой использования API.

#### Параметры:

* **code** - семантически читаемый фрагмент ошибки, например, **invalid-password**
* **info** - информацию об источнике ошибки. Например, если ошибка была вызвана неверными отправленными данными, информация может содержать список полей, содержащих неверные данные. Например, `['username', 'other_field']`
* **message** - удобочитаемое описание того, что пошло не так

Все вышеперечисленные данные будут сериализованы в JSON для возврата клиенту.

### \_\_init\_\_(code=None, info=None, message=None)

### serialize()

Составляет словарь ответов.

## ApiException

#### class ApiException(Exception)

Исключение, которое может быть вызвано различными конечными точками представления API.

Может содержать один или несколько объектов [ApiError](flask-apiexceptions.md#apierror) и должен включать код состояния HTTP.

Код состояния по умолчанию - **500**, если он не установлен.

### \_\_init\_\_(status\_code=None, error=None, message=None, info=None, code=None)

Инициализирует объект-контейнер **ApiException**.

Если предоставляется экземпляр `error`, он будет добавлен как ошибка, содержащаяся в этой оболочке. Если установлено какое-либо из `message`, `info` или `code`, создается новый объект ошибки.

### add\_error(error)

Добавляет ошибку в список ошибок, содержащихся в этом экземпляре **ApiException**.

### _property_ errors

Getter для ошибок, которые в настоящее время хранятся в этом экземпляре.

### serialize()

Сериализирует ошибки, содержащиеся в этом объекте **ApiException**, в типы Python, которые легко конвертируются в JSON (или аналогичные).

## api\_exception\_handler(api\_exception)

Сериализует объекты, совместимые с **ApiException**, и назначает правильный код ответа. Это обработчик по умолчанию.
