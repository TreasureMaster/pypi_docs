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
