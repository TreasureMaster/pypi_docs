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
