# uWSGI

**uWSGI** - это вариант развертывания на таких серверах, как [nginx](https://nginx.org/), [lighttpd](https://www.lighttpd.net/) и [cherokee](http://cherokee-project.com/); см. другие варианты в [FastCGI](fastcgi.md) и [автономных контейнерах WSGI](avtonomnye-konteinery-wsgi.md). Чтобы использовать приложение **WSGI** с протоколом **uWSGI**, вам сначала понадобится сервер **uWSGI**. **uWSGI** - это и протокол, и сервер приложений; сервер приложений может обслуживать протоколы **uWSGI**, **FastCGI** и **HTTP**.

Самый популярный сервер **uWSGI** - это [uwsgi](https://uwsgi-docs.readthedocs.io/en/latest/), который мы будем использовать в этом руководстве. Убедитесь, что он установлен, чтобы продолжить.

{% hint style="info" %}
**Осторожно:**

Заранее убедитесь, что любые вызовы **app.run ()**, которые могут быть в вашем файле приложения, находятся внутри блока `if __name__ == '__main__':` или перемещены в отдельный файл. Просто убедитесь, что он не вызывается, потому что при этом всегда будет запускаться локальный сервер **WSGI**, который нам не нужен, если мы развернем это приложение в **uWSGI**.
{% endhint %}

## Старт вашего приложения с uWSGI

**uwsgi** предназначен для работы с вызываемыми объектами **WSGI**, которые находятся в модулях Python.

Для приложения **Flask** в `myapp.py` используйте следующую команду:

```bash
$ uwsgi -s /tmp/yourapplication.sock --manage-script-name --mount /yourapplication=myapp:app
```

`--manage-script-name` переместит обработку **SCRIPT\_NAME** в **uwsgi**, так как это более разумно. Он используется вместе с директивой `--mount`, которая направляет запросы к `/yourapplication` на `myapp:app`. Если ваше приложение доступно на корневом уровне, вы можете использовать одиночный `/` вместо `/yourapplication`. **myapp** относится к имени файла вашего флеш-приложения (без расширения) или модуля, который предоставляет **app**. **app** вызывается внутри вашего приложения (обычно в строке написано `app = Flask (__name__)`.

Если вы хотите развернуть приложение **Flask** внутри виртуальной среды, вам также необходимо добавить `--virtualenv  /path/to/virtual/environment`. Вам также может потребоваться добавить `--plugin python` или `--plugin python3` в зависимости от того, какую версию python вы используете для своего проекта.

## Конфигурирование nginx

Базовая конфигурация **Flask** в **nginx** выглядит так:

```bash
location = /yourapplication { rewrite ^ /yourapplication/; }
location /yourapplication { try_files $uri @yourapplication; }
location @yourapplication {
  include uwsgi_params;
  uwsgi_pass unix:/tmp/yourapplication.sock;
}
```

Эта конфигурация связывает приложение с `/yourapplication`. Если вы хотите, чтобы он был в корневом URL-адресе, это немного проще:

```bash
location / { try_files $uri @yourapplication; }
location @yourapplication {
    include uwsgi_params;
    uwsgi_pass unix:/tmp/yourapplication.sock;
}
```
