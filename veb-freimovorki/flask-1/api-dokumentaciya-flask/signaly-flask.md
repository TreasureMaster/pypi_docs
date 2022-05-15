# API сигналов Flask

_Новое в версии 0.6_.

### signals.signals\_available

`True`, если имеется сигнализация. Это тот случай, когда установлен модуль [blinker](https://pypi.org/project/blinker/).

Во **Flask** существуют следующие сигналы:

### &#x20;flask.template\_rendered

Этот сигнал отправляется, когда шаблон был успешно визуализирован. Сигнал вызывается с экземпляром шаблона как _**template**_ и контекстом в качестве словаря (с именем _**context**_).

Пример подписчика:

```python
def log_template_renders(sender, template, context, **extra):
    sender.logger.debug('Rendering template "%s" with context %s',
                        template.name or 'string template',
                        context)

from flask import template_rendered
template_rendered.connect(log_template_renders, app)
```

### &#x20;flask.before\_render\_template

Этот сигнал отправляется до процесса рендеринга шаблона. Сигнал вызывается с экземпляром шаблона как _**template**_ и контекстом в качестве словаря (с именем _**context**_).

Пример подписчика:

```python
def log_template_renders(sender, template, context, **extra):
    sender.logger.debug('Rendering template "%s" with context %s',
                        template.name or 'string template',
                        context)

from flask import before_render_template
before_render_template.connect(log_template_renders, app)
```

### &#x20;flask.request\_started

Этот сигнал отправляется при настройке контекста запроса до того, как произойдет какая-либо обработка запроса. Поскольку контекст запроса уже привязан, подписчик может получить доступ к запросу с помощью стандартных глобальных прокси-серверов, таких как [request](dannye-vkhodyashego-zaprosa-request.md#flask-request).

Пример подписчика:

```python
def log_request(sender, **extra):
    sender.logger.debug('Request context is set up')

from flask import request_started
request_started.connect(log_request, app)
```

### &#x20;flask.request\_finished

Этот сигнал отправляется прямо перед отправкой ответа клиенту. Ему передается ответ, который должен быть отправлен именованный _**response**_.

Пример подписчика:

```python
def log_response(sender, response, **extra):
    sender.logger.debug('Request context is about to close down.  '
                        'Response: %s', response)

from flask import request_finished
request_finished.connect(log_response, app)
```

### &#x20;flask.got\_request\_exception

Этот сигнал отправляется, когда во время обработки запроса происходит исключение. Он отправляется _**перед**_ тем, как срабатывает стандартная обработка исключений, и даже в режиме отладки, где обработка исключений не происходит. Само исключение передается подписчику как _**exception**_.

Пример подписчика:

```python
def log_exception(sender, exception, **extra):
    sender.logger.debug('Got exception during processing: %s', exception)

from flask import got_request_exception
got_request_exception.connect(log_exception, app)
```

### &#x20;flask.request\_tearing\_down

Этот сигнал отправляется, когда запрос **teardown**. Он вызывается всегда, даже если возникает исключение. В настоящее время функции, прослушивающие этот сигнал, вызываются после обычных обработчиков удаления, но на это нельзя положиться.

Пример подписчика:

```python
def close_db_connection(sender, **extra):
    session.close()

from flask import request_tearing_down
request_tearing_down.connect(close_db_connection, app)
```

Начиная с `Flask 0.9`, ему также будет передан аргумент ключевого слова _**exc**_, который имеет ссылку на исключение, вызвавшее **teardown**, если оно было.

### &#x20;flask.appcontext\_tearing\_down

Этот сигнал отправляется, когда происходит **teardown** контекста приложения. Он вызывается всегда, даже если возникает исключение. В настоящее время функции, прослушивающие этот сигнал, вызываются после обычных обработчиков удаления, но на это нельзя положиться.

Пример подписчика:

```python
def close_db_connection(sender, **extra):
    session.close()

from flask import appcontext_tearing_down
appcontext_tearing_down.connect(close_db_connection, app)
```

Ему также будет передан аргумент ключевого слова _**exc**_, который имеет ссылку на исключение, вызвавшее **teardown**, если оно было.

### &#x20;flask.appcontext\_pushed

_Новое в версии 0.10_.

Этот сигнал отправляется при регистрации контекста приложения. Отправитель - это приложение. Обычно это полезно для модульных тестов, чтобы временно получить информацию. Например, его можно использовать для ранней установки ресурса в объект _**g**_.

Пример использования:

```python
from contextlib import contextmanager
from flask import appcontext_pushed

@contextmanager
def user_set(app, user):
    def handler(sender, **kwargs):
        g.user = user
    with appcontext_pushed.connected_to(handler, app):
        yield
```

И в тестовом коде:

```python
def test_user_me(self):
    with user_set(app, 'john'):
        c = app.test_client()
        resp = c.get('/users/me')
        assert resp.data == 'username=john'
```

### &#x20;flask.appcontext\_popped

_Новое в версии 0.10_.

Этот сигнал отправляется при выталкивании контекста приложения. Отправитель - это приложение. Обычно это соответствует сигналу [appcontext\_tearing\_down](signaly-flask.md#flask-appcontext\_tearing\_down).

### &#x20;flask.message\_flashed

_Новое в версии 0.10_.

Этот сигнал отправляется, когда приложение высвечивает сообщение. Сообщения отправляются как аргумент ключевого слова _**message**_, а категория - как _**category**_.

Пример подписчика:

```python
recorded = []
def record(sender, message, category, **extra):
    recorded.append((message, category))

from flask import message_flashed
message_flashed.connect(record, app)
```

## Класс signals.Namespace

Псевдоним для [blinker.base.Namespace](https://pythonhosted.org/blinker/index.html#blinker.base.Namespace), если **blinker** доступен, в противном случае - фиктивный класс, который создает ложные сигналы. Этот класс доступен для расширений **Flask**, которые хотят предоставить ту же резервную систему, что и сам **Flask**.

### signal( _name_, _doc=None_ )

Создает новый сигнал для этого пространства имен, если **blinker** доступен, в противном случае возвращает поддельный сигнал, у которого есть метод отправки, который ничего не сделает, но завершится ошибкой [RuntimeError](https://docs.python.org/3/library/exceptions.html#RuntimeError) для всех других операций, включая соединение.
