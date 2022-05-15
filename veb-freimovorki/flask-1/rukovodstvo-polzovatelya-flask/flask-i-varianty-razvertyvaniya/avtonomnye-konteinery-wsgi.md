# Автономные контейнеры WSGI

Есть популярные серверы, написанные на Python, которые содержат приложения **WSGI** и обслуживают HTTP. Эти серверы работают автономно; вы можете проксировать к ним со своего веб-сервера. Обратите внимание на раздел «[Настройки прокси](avtonomnye-konteinery-wsgi.md#nastroika-proksi)», если у вас возникнут проблемы.

## Gunicorn

[Gunicorn](https://gunicorn.org/) «Green Unicorn» - это HTTP-сервер **WSGI** для **UNIX**. Это рабочая модель пре-форка, портированная из проекта **Ruby Unicorn**. Он поддерживает как [eventlet](https://eventlet.net/), так и [greenlet](https://greenlet.readthedocs.io/en/latest/). Запустить приложение **Flask** на этом сервере довольно просто:

```bash
$ gunicorn myproject:app
```

[Gunicorn](https://gunicorn.org/) предоставляет множество параметров командной строки - см. `gunicorn -h`. Например, чтобы запустить приложение **Flask** с 4-мя рабочими процессами (`-w 4`), привязанными к локальному порту 4000 (`-b 127.0.0.1:4000`):

```bash
$ gunicorn -w 4 -b 127.0.0.1:4000 myproject:app
```

Команда **gunicorn** ожидает имена вашего модуля или пакета приложения и экземпляра приложения в модуле. Если вы используете шаблон фабрики приложений, вы можете передать вызов этому:

```bash
$ gunicorn "myproject:create_app()"
```

## uWSGI

[uWSGI](https://uwsgi-docs.readthedocs.io/en/latest/) - это быстрый сервер приложений, написанный на C. Он очень настраиваемый, что делает его более сложным в настройке, чем **Gunicorn**.

Запуск [uWSGI HTTP Router](https://uwsgi-docs.readthedocs.io/en/latest/HTTP.html#the-uwsgi-http-https-router):

```bash
$ uwsgi --http 127.0.0.1:5000 --module myproject:app
```

Для более оптимизированной настройки см. [Настройка uWSGI и NGINX](uwsgi.md).

## Gevent

[Gevent](http://www.gevent.org/) - это сетевая библиотека Python на основе сопрограмм, которая использует [greenlet](https://greenlet.readthedocs.io/en/latest/) для обеспечения высокоуровневого синхронного API поверх цикла обработки событий [libev](http://software.schmorp.de/pkg/libev.html):

```python
from gevent.pywsgi import WSGIServer
from yourapplication import app

http_server = WSGIServer(('', 5000), app)
http_server.serve_forever()
```

## Twisted Web

[Twisted Web](https://twistedmatrix.com/trac/wiki/TwistedWeb) - это веб-сервер, поставляемый с [Twisted](https://twistedmatrix.com/trac/), зрелой неблокирующей сетевой библиотекой, управляемой событиями. **Twisted Web** поставляется со стандартным контейнером **WSGI**, которым можно управлять из командной строки с помощью утилиты **twistd**:

```bash
$ twistd web --wsgi myproject.app
```

В этом примере будет запущено приложение **Flask** с именем **app** из модуля **myproject**.

**Twisted Web** поддерживает множество флагов и опций, как и утилита **twistd**; см. `twistd -h` и `twistd web -h` для получения дополнительной информации. Например, чтобы запустить **Twisted Web**-сервер на переднем плане через порт **8080** с приложением из **myproject**:

```bash
$ twistd -n web --port tcp:8080 --wsgi myproject.app
```

## Настройка прокси

Если вы развертываете свое приложение с использованием одного из этих серверов за HTTP-прокси, вам необходимо переписать несколько заголовков, чтобы приложение работало. Два проблемных значения в среде **WSGI** обычно - это **REMOTE\_ADDR** и **HTTP\_HOST**. Вы можете настроить **httpd** для передачи этих заголовков или исправить их в промежуточном программном обеспечении. **Werkzeug** поставляет исправление, которое решит некоторые общие настройки, но вы можете написать свое собственное промежуточное ПО **WSGI** для определенных настроек.

Вот простая конфигурация **nginx**, которая передает прокси в приложение, обслуживаемое на локальном хосте через порт `8000`, с соответствующими заголовками:

```bash
server {
    listen 80;

    server_name _;

    access_log  /var/log/nginx/access.log;
    error_log  /var/log/nginx/error.log;

    location / {
        proxy_pass         http://127.0.0.1:8000/;
        proxy_redirect     off;

        proxy_set_header   Host                 $host;
        proxy_set_header   X-Real-IP            $remote_addr;
        proxy_set_header   X-Forwarded-For      $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto    $scheme;
    }
}
```

Если ваш **httpd** не предоставляет эти заголовки, наиболее распространенная установка вызывает хост, установленный из `X-Forwarded-Host`, и удаленный адрес из `X-Forwarded-For`:

```python
from werkzeug.middleware.proxy_fix import ProxyFix
app.wsgi_app = ProxyFix(app.wsgi_app, x_proto=1, x_host=1)
```

{% hint style="info" %}
**Доверенные заголовки:**

Имейте в виду, что использование такого промежуточного программного обеспечения в настройке без прокси является проблемой безопасности, поскольку оно будет слепо доверять входящим заголовкам, которые могут быть подделаны злоумышленниками.
{% endhint %}

Если вы хотите переписать заголовки из другого заголовка, вы можете использовать такой фиксатор:

```python
class CustomProxyFix(object):

    def __init__(self, app):
        self.app = app

    def __call__(self, environ, start_response):
        host = environ.get('HTTP_X_FHOST', '')
        if host:
            environ['HTTP_HOST'] = host
        return self.app(environ, start_response)

app.wsgi_app = CustomProxyFix(app.wsgi_app)
```
