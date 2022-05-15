# Ведение журнала Flask

**Flask** использует стандартное ведение журнала Python [logging](https://docs.python.org/3/library/logging.html#module-logging). Сообщения о вашем приложении **Flask** регистрируются с помощью [app.logger](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#logger), имя которого совпадает с именем [app.name](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#name). Этот регистратор также может использоваться для регистрации ваших собственных сообщений.

```python
@app.route('/login', methods=['POST'])
def login():
    user = get_user(request.form['username'])

    if user.check_password(request.form['password']):
        login_user(user)
        app.logger.info('%s logged in successfully', user.username)
        return redirect(url_for('index'))
    else:
        app.logger.info('%s failed to log in', user.username)
        abort(401)
```

## Базовая конфигурация

Если вы хотите настроить ведение журнала для своего проекта, вы должны сделать это как можно скорее при запуске программы. Если доступ к [app.logger](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#logger) осуществляется до настройки ведения журнала, он добавит обработчик по умолчанию. Если возможно, настройте ведение журнала перед созданием объекта приложения.

В этом примере используется [dictConfig ()](https://docs.python.org/3/library/logging.config.html#logging.config.dictConfig) для создания конфигурации ведения журнала, аналогичной настройке **Flask** по умолчанию, за исключением всех журналов:

```python
from logging.config import dictConfig

dictConfig({
    'version': 1,
    'formatters': {'default': {
        'format': '[%(asctime)s] %(levelname)s in %(module)s: %(message)s',
    }},
    'handlers': {'wsgi': {
        'class': 'logging.StreamHandler',
        'stream': 'ext://flask.logging.wsgi_errors_stream',
        'formatter': 'default'
    }},
    'root': {
        'level': 'INFO',
        'handlers': ['wsgi']
    }
})

app = Flask(__name__)
```

### Конфигурация по умолчанию

Если вы не настраиваете ведение журнала самостоятельно, **Flask** автоматически добавит [StreamHandler](https://docs.python.org/3/library/logging.handlers.html#logging.StreamHandler) в [app.logger](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#logger). Во время запросов он будет записывать в поток, указанный сервером **WSGI** в `environ['wsgi.errors']` (обычно это [sys.stderr](https://docs.python.org/3/library/sys.html#sys.stderr)). Вне запроса он будет регистрироваться в [sys.stderr](https://docs.python.org/3/library/sys.html#sys.stderr).

### Удаление обработчика по умолчанию

Если вы настроили ведение журнала после доступа к [app.logger](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#logger) и вам нужно удалить обработчик по умолчанию, вы можете импортировать и удалить его:

```python
from flask.logging import default_handler

app.logger.removeHandler(default_handler)
```

## Электронная почта об ошибках для администратора

При запуске приложения на удаленном сервере для производства вы, вероятно, не будете часто просматривать сообщения журнала. Сервер **WSGI**, вероятно, будет отправлять сообщения журнала в файл, и вы будете проверять этот файл только в том случае, если пользователь сообщит вам, что что-то пошло не так.

Чтобы своевременно обнаруживать и исправлять ошибки, вы можете настроить [logging.handlers.SMTPHandler](https://docs.python.org/3/library/logging.handlers.html#logging.handlers.SMTPHandler) для отправки электронного письма при регистрации ошибок и выше.

```python
import logging
from logging.handlers import SMTPHandler

mail_handler = SMTPHandler(
    mailhost='127.0.0.1',
    fromaddr='server-error@example.com',
    toaddrs=['admin@example.com'],
    subject='Application Error'
)
mail_handler.setLevel(logging.ERROR)
mail_handler.setFormatter(logging.Formatter(
    '[%(asctime)s] %(levelname)s in %(module)s: %(message)s'
))

if not app.debug:
    app.logger.addHandler(mail_handler)
```

Для этого необходимо, чтобы на том же сервере был настроен SMTP-сервер. См. документацию Python для получения дополнительной информации о настройке обработчика.

## Внедрение информации запроса

Просмотр дополнительной информации о запросе, такой как IP-адрес, может помочь отладить некоторые ошибки. Вы можете создать подкласс [logging.Formatter](https://docs.python.org/3/library/logging.html#logging.Formatter), чтобы вводить собственные поля, которые можно использовать в сообщениях. Вы можете изменить средство форматирования для обработчика **Flask** по умолчанию, обработчика почты, определенного выше, или любого другого обработчика.

```python
from flask import has_request_context, request
from flask.logging import default_handler

class RequestFormatter(logging.Formatter):
    def format(self, record):
        if has_request_context():
            record.url = request.url
            record.remote_addr = request.remote_addr
        else:
            record.url = None
            record.remote_addr = None

        return super().format(record)

formatter = RequestFormatter(
    '[%(asctime)s] %(remote_addr)s requested %(url)s\n'
    '%(levelname)s in %(module)s: %(message)s'
)
default_handler.setFormatter(formatter)
mail_handler.setFormatter(formatter)
```

## Другие библиотеки

Другие библиотеки могут широко использовать ведение журнала, и вы также хотите видеть соответствующие сообщения из этих журналов. Самый простой способ сделать это - добавить обработчики к корневому регистратору, а не только к регистратору приложения.

```python
from flask.logging import default_handler

root = logging.getLogger()
root.addHandler(default_handler)
root.addHandler(mail_handler)
```

В зависимости от вашего проекта, может быть более полезным настроить каждый регистратор, который вам нужен, отдельно, а не настраивать только корневой регистратор.

```python
for logger in (
    app.logger,
    logging.getLogger('sqlalchemy'),
    logging.getLogger('other_package'),
):
    logger.addHandler(default_handler)
    logger.addHandler(mail_handler)
```

### Werkzeug

**Werkzeug** регистрирует основную информацию о запросах / ответах в регистраторе `'werkzeug'`. Если у корневого регистратора нет настроенных обработчиков, **Werkzeug** добавляет [StreamHandler](https://docs.python.org/3/library/logging.handlers.html#logging.StreamHandler) к своему регистратору.

### Расширения Flask

В зависимости от ситуации расширение может выбрать регистрацию в [app.logger](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#logger) или в собственном именованном регистраторе. За подробностями обращайтесь к документации каждого расширения.
