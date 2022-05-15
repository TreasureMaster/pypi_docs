# Быстрый старт Flask-RESTful

Пришло время написать свой первый REST API. В этом руководстве предполагается, что вы имеете представление о Flask и уже установили **Flask** и **Flask-RESTful**. Если нет, то следуйте инструкциям в разделе «[Установка](ustanovka-flask-restful.md)».

## Минимальный API

Минимальный API **Flask-RESTful** выглядит так:

```python
from flask import Flask
from flask_restful import Resource, Api

app = Flask(__name__)
api = Api(app)

class HelloWorld(Resource):
    def get(self):
        return {'hello': 'world'}

api.add_resource(HelloWorld, '/')

if __name__ == '__main__':
    app.run(debug=True)
```

Сохраните это как `api.py` и запустите с помощью интерпретатора Python. Обратите внимание, что мы включили [режим отладки Flask](http://flask.pocoo.org/docs/quickstart/#debug-mode), чтобы обеспечить перезагрузку кода и улучшить сообщения об ошибках.

```bash
$ python api.py
 * Running on http://127.0.0.1:5000/
 * Restarting with reloader
```

{% hint style="warning" %}
Режим отладки никогда не следует использовать в производственной среде!
{% endhint %}

Теперь откройте новую консоль, чтобы проверить свой API с помощью curl.

```bash
$ curl http://127.0.0.1:5000/
{"hello": "world"}
```

## Маршрутизация ресурсов

Основным строительным блоком, предоставляемым **Flask-RESTful**, являются ресурсы. Ресурсы создаются на основе [подключаемых представлений Flask](http://flask.pocoo.org/docs/views/), предоставляя вам легкий доступ к нескольким методам HTTP, просто определяя методы в вашем ресурсе. Базовый CRUD-ресурс для приложения todo (конечно) выглядит так:

```python
from flask import Flask, request
from flask_restful import Resource, Api

app = Flask(__name__)
api = Api(app)

todos = {}

class TodoSimple(Resource):
    def get(self, todo_id):
        return {todo_id: todos[todo_id]}

    def put(self, todo_id):
        todos[todo_id] = request.form['data']
        return {todo_id: todos[todo_id]}

api.add_resource(TodoSimple, '/<string:todo_id>')

if __name__ == '__main__':
    app.run(debug=True)
```

Вы можете попробовать это так:

```bash
$ curl http://localhost:5000/todo1 -d "data=Remember the milk" -X PUT
{"todo1": "Remember the milk"}
$ curl http://localhost:5000/todo1
{"todo1": "Remember the milk"}
$ curl http://localhost:5000/todo2 -d "data=Change my brakepads" -X PUT
{"todo2": "Change my brakepads"}
$ curl http://localhost:5000/todo2
{"todo2": "Change my brakepads"}
```

Или из python, если у вас установлена библиотека **requests**:

```bash
>>> from requests import put, get
>>> put('http://localhost:5000/todo1', data={'data': 'Remember the milk'}).json()
{u'todo1': u'Remember the milk'}
>>> get('http://localhost:5000/todo1').json()
{u'todo1': u'Remember the milk'}
>>> put('http://localhost:5000/todo2', data={'data': 'Change my brakepads'}).json()
{u'todo2': u'Change my brakepads'}
>>> get('http://localhost:5000/todo2').json()
{u'todo2': u'Change my brakepads'}
```

**Flask-RESTful** понимает несколько видов возвращаемых значений из методов представления (views). Подобно Flask, вы можете вернуть любой итерируемый объект, и он будет преобразован в ответ, включая необработанные объекты ответа Flask. **Flask-RESTful** также поддерживает настройку кода ответа и заголовков ответа с использованием нескольких возвращаемых значений, как показано ниже:

```python
class Todo1(Resource):
    def get(self):
        # По умолчанию 200 OK
        return {'task': 'Hello world'}

class Todo2(Resource):
    def get(self):
        # Установите код ответа на 201
        return {'task': 'Hello world'}, 201

class Todo3(Resource):
    def get(self):
        # Установите код ответа на 201 и верните настраиваемые заголовки.
        return {'task': 'Hello world'}, 201, {'Etag': 'some-opaque-string'}
```

## Конечные точки (endpoints)

Много раз в API ваш ресурс будет иметь несколько URL-адресов. Вы можете передать несколько URL-адресов методу <mark style="color:red;">add\_resource()</mark> объекта **API**. Каждый будет перенаправлен на ваш ресурс <mark style="color:red;">Resource</mark>.

```python
api.add_resource(HelloWorld,
    '/',
    '/hello')
```

Вы также можете сопоставлять части пути как переменные с вашими методами ресурсов.

```python
api.add_resource(Todo,
    '/todo/<int:todo_id>', endpoint='todo_ep')
```

## Парсинг аргументов

Несмотря на то, что **Flask** обеспечивает легкий доступ к данным запроса (например, к строке запроса или данным, закодированным в форме **POST**), проверка данных формы по-прежнему усложняется. **Flask-RESTful** имеет встроенную поддержку проверки данных запроса с использованием библиотеки, похожей на [argparse](http://docs.python.org/dev/library/argparse.html).

```python
from flask_restful import reqparse

parser = reqparse.RequestParser()
parser.add_argument('rate', type=int, help='Rate to charge for this resource')
args = parser.parse_args()
```

{% hint style="info" %}
В отличие от модуля **argparse**, <mark style="color:red;">reqparse.RequestParser.parse\_args()</mark> возвращает словарь Python вместо пользовательской структуры данных.
{% endhint %}

Использование модуля <mark style="color:red;">reqparse</mark> также дает вам разумные сообщения об ошибках бесплатно. Если аргумент не проходит проверку, **Flask-RESTful** ответит ошибкой **400 Bad Request** и ответом с указанием ошибки.

```bash
$ curl -d 'rate=foo' http://127.0.0.1:5000/todos
{'status': 400, 'message': 'foo cannot be converted to int'}
```

Модуль <mark style="color:red;">inputs</mark> предоставляет ряд встроенных общих функций преобразования, таких как <mark style="color:red;">inputs.date()</mark> и <mark style="color:red;">inputs.url()</mark>.

Вызов **parse\_args** со `strict=True` гарантирует, что будет выдано сообщение об ошибке, если запрос включает аргументы, которые не определены вашим синтаксическим анализатором.

```python
args = parser.parse_args(strict=True)
```

## Форматирование данных

По умолчанию все поля в вашей возвращаемой итерации будут отображаться как есть. Хотя это прекрасно работает, когда вы имеете дело только со структурами данных Python, это может сильно разочаровать при работе с объектами. Чтобы решить эту проблему, **Flask-RESTful** предоставляет модуль <mark style="color:red;">fields</mark> и декоратор <mark style="color:red;">marshal\_with()</mark>. Подобно **Django ORM** и **WTForm**, вы используете модуль **fields** для описания структуры вашего ответа.

```python
from flask_restful import fields, marshal_with

resource_fields = {
    'task':   fields.String,
    'uri':    fields.Url('todo_ep')
}

class TodoDao(object):
    def __init__(self, todo_id, task):
        self.todo_id = todo_id
        self.task = task

        # Это поле не будет отправлено в ответ
        self.status = 'active'

class Todo(Resource):
    @marshal_with(resource_fields)
    def get(self, **kwargs):
        return TodoDao(todo_id='my_todo', task='Remember the milk')
```

В приведенном выше примере объект Python берется и подготавливается к сериализации. Декоратор <mark style="color:red;">marshal\_with()</mark> применит преобразование, описанное в **resource\_fields**. Единственным полем, извлекаемым из объекта, является задача. Поле <mark style="color:red;">fields.Url</mark> — это специальное поле, которое принимает имя конечной точки и генерирует URL-адрес для этой конечной точки в ответе. Многие из необходимых вам типов полей уже включены. Полный список см. в руководстве по <mark style="color:red;">fields</mark>.

## Полный пример

Сохраните этот пример в `api.py`.

```python
from flask import Flask
from flask_restful import reqparse, abort, Api, Resource

app = Flask(__name__)
api = Api(app)

TODOS = {
    'todo1': {'task': 'build an API'},
    'todo2': {'task': '?????'},
    'todo3': {'task': 'profit!'},
}


def abort_if_todo_doesnt_exist(todo_id):
    if todo_id not in TODOS:
        abort(404, message="Todo {} doesn't exist".format(todo_id))

parser = reqparse.RequestParser()
parser.add_argument('task')


# Todo
# показывает один элемент задачи и позволяет удалить элемент задачи
class Todo(Resource):
    def get(self, todo_id):
        abort_if_todo_doesnt_exist(todo_id)
        return TODOS[todo_id]

    def delete(self, todo_id):
        abort_if_todo_doesnt_exist(todo_id)
        del TODOS[todo_id]
        return '', 204

    def put(self, todo_id):
        args = parser.parse_args()
        task = {'task': args['task']}
        TODOS[todo_id] = task
        return task, 201


# TodoList
# показывает список всех задач и позволяет добавлять новые задачи с помощью POST
class TodoList(Resource):
    def get(self):
        return TODOS

    def post(self):
        args = parser.parse_args()
        todo_id = int(max(TODOS.keys()).lstrip('todo')) + 1
        todo_id = 'todo%i' % todo_id
        TODOS[todo_id] = {'task': args['task']}
        return TODOS[todo_id], 201

##
## Фактически настройте маршрутизацию ресурсов API здесь
##
api.add_resource(TodoList, '/todos')
api.add_resource(Todo, '/todos/<todo_id>')


if __name__ == '__main__':
    app.run(debug=True)
```

Пример использования

```bash
$ python api.py
 * Running on http://127.0.0.1:5000/
 * Restarting with reloader
```

GET список

```bash
$ curl http://localhost:5000/todos
{"todo1": {"task": "build an API"}, "todo3": {"task": "profit!"}, "todo2": {"task": "?????"}}
```

GET одну задачу

```bash
$ curl http://localhost:5000/todos/todo3
{"task": "profit!"}
```

DELETE задачу

```bash
$ curl http://localhost:5000/todos/todo2 -X DELETE -v

> DELETE /todos/todo2 HTTP/1.1
> User-Agent: curl/7.19.7 (universal-apple-darwin10.0) libcurl/7.19.7 OpenSSL/0.9.8l zlib/1.2.3
> Host: localhost:5000
> Accept: */*
>
* HTTP 1.0, assume close after body
< HTTP/1.0 204 NO CONTENT
< Content-Type: application/json
< Content-Length: 0
< Server: Werkzeug/0.8.3 Python/2.7.2
< Date: Mon, 01 Oct 2012 22:10:32 GMT
```

Добавить новую задачу

```bash
$ curl http://localhost:5000/todos -d "task=something new" -X POST -v

> POST /todos HTTP/1.1
> User-Agent: curl/7.19.7 (universal-apple-darwin10.0) libcurl/7.19.7 OpenSSL/0.9.8l zlib/1.2.3
> Host: localhost:5000
> Accept: */*
> Content-Length: 18
> Content-Type: application/x-www-form-urlencoded
>
* HTTP 1.0, assume close after body
< HTTP/1.0 201 CREATED
< Content-Type: application/json
< Content-Length: 25
< Server: Werkzeug/0.8.3 Python/2.7.2
< Date: Mon, 01 Oct 2012 22:12:58 GMT
<
* Closing connection #0
{"task": "something new"}
```

Обновить задачу

```bash
$ curl http://localhost:5000/todos/todo3 -d "task=something different" -X PUT -v

> PUT /todos/todo3 HTTP/1.1
> Host: localhost:5000
> Accept: */*
> Content-Length: 20
> Content-Type: application/x-www-form-urlencoded
>
* HTTP 1.0, assume close after body
< HTTP/1.0 201 CREATED
< Content-Type: application/json
< Content-Length: 27
< Server: Werkzeug/0.8.3 Python/2.7.3
< Date: Mon, 01 Oct 2012 22:13:00 GMT
<
* Closing connection #0
{"task": "something different"}
```
