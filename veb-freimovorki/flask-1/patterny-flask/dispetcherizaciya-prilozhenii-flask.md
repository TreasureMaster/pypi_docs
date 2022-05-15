# Диспетчеризация приложений Flask

Диспетчеризация приложений - это процесс объединения нескольких приложений **Flask** на уровне **WSGI**. Вы можете комбинировать не только приложения **Flask**, но и любое приложение **WSGI**. Это позволит вам запускать приложения **Django** и **Flask** в одном интерпретаторе одновременно, если хотите. Полезность этого зависит от того, как приложения работают внутри.

Принципиальное отличие от [модульного подхода](bolshie-prilozheniya-flask.md) заключается в том, что в этом случае вы запускаете одни и те же или разные приложения **Flask**, которые полностью изолированы друг от друга. Они работают в разных конфигурациях и отправляются на уровне **WSGI**.

## Работа с этим документом

Каждый из приведенных ниже методов и примеров приводит к созданию объекта **application**, который может быть запущен с любым сервером **WSGI**. Для производства см. [Варианты развертывания](../rukovodstvo-polzovatelya-flask/flask-i-varianty-razvertyvaniya/). Для разработки **Werkzeug** предоставляет встроенный сервер для разработки, доступный по адресу [werkzeug.serving.run\_simple ()](https://werkzeug.palletsprojects.com/en/1.0.x/serving/#werkzeug.serving.run\_simple):

```python
from werkzeug.serving import run_simple
run_simple('localhost', 5000, application, use_reloader=True)
```

Обратите внимание, что [run\_simple](https://werkzeug.palletsprojects.com/en/1.0.x/serving/#werkzeug.serving.run\_simple) не предназначен для использования в производственной среде. Используйте [полноценный сервер WSGI](../rukovodstvo-polzovatelya-flask/flask-i-varianty-razvertyvaniya/).

Чтобы использовать интерактивный отладчик, отладка должна быть включена как в приложении, так и на простом сервере. Вот пример «hello world» с отладкой и **run\_simple**:

```python
from flask import Flask
from werkzeug.serving import run_simple

app = Flask(__name__)
app.debug = True

@app.route('/')
def hello_world():
    return 'Hello World!'

if __name__ == '__main__':
    run_simple('localhost', 5000, app,
               use_reloader=True, use_debugger=True, use_evalex=True)
```

## Объединение приложений

Если у вас есть полностью разделенные приложения и вы хотите, чтобы они работали рядом друг с другом в одном процессе интерпретатора Python, вы можете воспользоваться **werkzeug.wsgi.DispatcherMiddleware**. Идея здесь в том, что каждое приложение **Flask** является допустимым приложением **WSGI**, и они объединяются промежуточным программным обеспечением диспетчера в более крупное приложение, которое отправляется на основе префикса.

Например, у вас может быть основное приложение, работающее на `/`, а ваш бэкэнд-интерфейс - на `/backend`:

```python
from werkzeug.middleware.dispatcher import DispatcherMiddleware
from frontend_app import application as frontend
from backend_app import application as backend

application = DispatcherMiddleware(frontend, {
    '/backend': backend
})
```

## Отправка по субдомену

Иногда вам может понадобиться использовать несколько экземпляров одного и того же приложения с разными конфигурациями. Предполагая, что приложение создано внутри функции, и вы можете вызвать эту функцию для ее создания, это действительно легко реализовать. Чтобы разработать приложение для поддержки создания новых экземпляров в функциях, взгляните на паттерн [Application Factories](fabriki-prilozheniya-flask.md).

Очень распространенный пример - создание приложений для каждого поддомена. Например, вы настраиваете свой веб-сервер для отправки всех запросов для всех поддоменов вашему приложению, а затем используете информацию о поддомене для создания экземпляров, специфичных для пользователя. После того, как ваш сервер настроен для прослушивания всех поддоменов, вы можете использовать очень простое приложение **WSGI** для создания динамического приложения.

В этом отношении идеальным уровнем абстракции является уровень **WSGI**. Вы пишете собственное приложение **WSGI**, которое просматривает приходящий запрос и делегирует его вашему приложению **Flask**. Если это приложение еще не существует, оно динамически создается и запоминается:

```python
from threading import Lock

class SubdomainDispatcher(object):

    def __init__(self, domain, create_app):
        self.domain = domain
        self.create_app = create_app
        self.lock = Lock()
        self.instances = {}

    def get_application(self, host):
        host = host.split(':')[0]
        assert host.endswith(self.domain), 'Configuration error'
        subdomain = host[:-len(self.domain)].rstrip('.')
        with self.lock:
            app = self.instances.get(subdomain)
            if app is None:
                app = self.create_app(subdomain)
                self.instances[subdomain] = app
            return app

    def __call__(self, environ, start_response):
        app = self.get_application(environ['HTTP_HOST'])
        return app(environ, start_response)
```

Затем этот диспетчер можно использовать следующим образом:

```python
from myapplication import create_app, get_user_for_subdomain
from werkzeug.exceptions import NotFound

def make_app(subdomain):
    user = get_user_for_subdomain(subdomain)
    if user is None:
        # если для этого поддомена нет пользователя,
        # мы все равно должны вернуть приложение WSGI,
        # которое обрабатывает этот запрос.
        # Затем мы можем просто вернуть исключение NotFound() как приложение,
        # которое будет отображать страницу 404 по умолчанию.
        # Вы также можете перенаправить пользователя на главную страницу,
        # а затем
        return NotFound()

    # в противном случае создайте приложение для конкретного пользователя
    return create_app(user)

application = SubdomainDispatcher('example.com', make_app)
```

## Отправка по пути

Отправка по пути в URL очень похожа. Вместо того, чтобы смотреть на заголовок **Host**, чтобы определить поддомен, просто просматривается путь запроса до первой косой черты:

```python
from threading import Lock
from werkzeug.wsgi import pop_path_info, peek_path_info

class PathDispatcher(object):

    def __init__(self, default_app, create_app):
        self.default_app = default_app
        self.create_app = create_app
        self.lock = Lock()
        self.instances = {}

    def get_application(self, prefix):
        with self.lock:
            app = self.instances.get(prefix)
            if app is None:
                app = self.create_app(prefix)
                if app is not None:
                    self.instances[prefix] = app
            return app

    def __call__(self, environ, start_response):
        app = self.get_application(peek_path_info(environ))
        if app is not None:
            pop_path_info(environ)
        else:
            app = self.default_app
        return app(environ, start_response)
```

Большая разница между этим и субдоменом заключается в том, что этот возвращается к другому приложению, если функция создателя возвращает `None`:

```python
from myapplication import create_app, default_app, get_user_for_prefix

def make_app(prefix):
    user = get_user_for_prefix(prefix)
    if user is not None:
        return create_app(user)

application = PathDispatcher(default_app, make_app)
```
