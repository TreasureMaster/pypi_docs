# Добавление переопределений метода HTTP

Некоторые прокси-серверы HTTP не поддерживают произвольные методы HTTP или новые методы HTTP (например, **PATCH**). В этом случае можно «проксировать» методы HTTP через другой метод HTTP с полным нарушением протокола.

Это работает, позволяя клиенту выполнять HTTP-запрос **POST** и устанавливать заголовок `X-HTTP-Method-Override`. Затем метод заменяется значением заголовка перед передачей во **Flask**.

Это может быть выполнено с помощью промежуточного программного обеспечения HTTP:

```python
class HTTPMethodOverrideMiddleware(object):
    allowed_methods = frozenset([
        'GET',
        'HEAD',
        'POST',
        'DELETE',
        'PUT',
        'PATCH',
        'OPTIONS'
    ])
    bodyless_methods = frozenset(['GET', 'HEAD', 'OPTIONS', 'DELETE'])

    def __init__(self, app):
        self.app = app

    def __call__(self, environ, start_response):
        method = environ.get('HTTP_X_HTTP_METHOD_OVERRIDE', '').upper()
        if method in self.allowed_methods:
            environ['REQUEST_METHOD'] = method
        if method in self.bodyless_methods:
            environ['CONTENT_LENGTH'] = '0'
        return self.app(environ, start_response)
```

Чтобы использовать это с **Flask**, оберните объект приложения промежуточным программным обеспечением:

```python
from flask import Flask

app = Flask(__name__)
app.wsgi_app = HTTPMethodOverrideMiddleware(app.wsgi_app)
```
