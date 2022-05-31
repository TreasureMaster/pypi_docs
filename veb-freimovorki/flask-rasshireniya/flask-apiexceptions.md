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
