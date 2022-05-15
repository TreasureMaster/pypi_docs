# FastCGI

**FastCGI** - это вариант развертывания на таких серверах, как [nginx](https://nginx.org/), [lighttpd](https://www.lighttpd.net/) и [cherokee](http://cherokee-project.com/); другие варианты см. в [uWSGI](uwsgi.md) и [автономных контейнерах WSGI](avtonomnye-konteinery-wsgi.md). Чтобы использовать приложение **WSGI** с любым из них, вам сначала понадобится сервер **FastCGI**. Самый популярный - это [flup](https://pypi.org/project/flup/), который мы будем использовать в этом руководстве. Убедитесь, что он установлен, чтобы продолжить.

{% hint style="info" %}
**Осторожно:**

Заранее убедитесь, что любые вызовы **app.run ()**, которые могут быть в вашем файле приложения, находятся внутри блока `if __name__ == '__main__':` или перемещены в отдельный файл. Просто убедитесь, что он не вызывается, потому что при этом всегда будет запускаться локальный сервер **WSGI**, который нам не нужен, если мы развернем это приложение на **FastCGI**.
{% endhint %}

## Создание файла .fcgi

Сначала вам нужно создать файл сервера **FastCGI**. Назовем его `yourapplication.fcgi`:

```python
#!/usr/bin/python
from flup.server.fcgi import WSGIServer
from yourapplication import app

if __name__ == '__main__':
    WSGIServer(app).run()
```

Этого достаточно для работы **Apache**, однако **nginx** и более старые версии **lighttpd** нуждаются в явной передаче сокета для связи с сервером **FastCGI**. Чтобы это сработало, вам нужно передать путь к сокету на **WSGIServer**:

```python
WSGIServer(application, bindAddress='/path/to/fcgi.sock').run()
```

Путь должен быть точно таким же, как вы указали в конфигурации сервера.

Сохраните файл `yourapplication.fcgi` где-нибудь, чтобы снова его найти. Имеет смысл разместить это в `/var/www/yourapplication` или в чем-то подобном.

Обязательно установите исполняемый бит в этом файле, чтобы серверы могли его выполнить:

```bash
$ chmod +x /var/www/yourapplication/yourapplication.fcgi
```

## Конфигурирование Apache

Приведенного выше примера достаточно для базового развертывания **Apache**, но ваш файл `.fcgi` появится в URL-адресе вашего приложения, например `example.com/yourapplication.fcgi/news/`. Есть несколько способов настроить ваше приложение так, чтобы `yourapplication.fcgi` не отображался в URL-адресе. Предпочтительный способ - использовать директивы конфигурации **ScriptAlias** и **SetHandler** для маршрутизации запросов на сервер **FastCGI**. В следующем примере **FastCgiServer** используется для запуска 5 экземпляров приложения, которое будет обрабатывать все входящие запросы:

```markup
LoadModule fastcgi_module /usr/lib64/httpd/modules/mod_fastcgi.so

FastCgiServer /var/www/html/yourapplication/app.fcgi -idle-timeout 300 -processes 5

<VirtualHost *>
    ServerName webapp1.mydomain.com
    DocumentRoot /var/www/html/yourapplication

    AddHandler fastcgi-script fcgi
    ScriptAlias / /var/www/html/yourapplication/app.fcgi/

    <Location />
        SetHandler fastcgi-script
    </Location>
</VirtualHost>
```

Эти процессы будут управляться **Apache**. Если вы используете автономный сервер **FastCGI**, вместо него можно использовать директиву **FastCgiExternalServer**. Обратите внимание, что в дальнейшем путь не является реальным, он просто используется как идентификатор для других директив, таких как **AliasMatch**:

```bash
FastCgiServer /var/www/html/yourapplication -host 127.0.0.1:3000
```

Если вы не можете установить **ScriptAlias**, например, на общем веб-хосте, вы можете использовать промежуточное программное обеспечение **WSGI** для удаления `yourapplication.fcgi` из URL-адресов. Установите `.htaccess`:

```markup
<IfModule mod_fcgid.c>
   AddHandler fcgid-script .fcgi
   <Files ~ (\.fcgi)>
       SetHandler fcgid-script
       Options +FollowSymLinks +ExecCGI
   </Files>
</IfModule>

<IfModule mod_rewrite.c>
   Options +FollowSymlinks
   RewriteEngine On
   RewriteBase /
   RewriteCond %{REQUEST_FILENAME} !-f
   RewriteRule ^(.*)$ yourapplication.fcgi/$1 [QSA,L]
</IfModule>
```

Установите `yourapplication.fcgi`:

```python
#!/usr/bin/python
#: необязательный путь к вашей локальной папке пакетов сайта python
import sys
sys.path.insert(0, '<your_local_path>/lib/python<your_python_version>/site-packages')

from flup.server.fcgi import WSGIServer
from yourapplication import app

class ScriptNameStripper(object):
   def __init__(self, app):
       self.app = app

   def __call__(self, environ, start_response):
       environ['SCRIPT_NAME'] = ''
       return self.app(environ, start_response)

app = ScriptNameStripper(app)

if __name__ == '__main__':
    WSGIServer(app).run()
```

## Конфигурирование lighttpd

Базовая конфигурация **FastCGI** для **lighttpd** выглядит так:

```python
fastcgi.server = ("/yourapplication.fcgi" =>
    ((
        "socket" => "/tmp/yourapplication-fcgi.sock",
        "bin-path" => "/var/www/yourapplication/yourapplication.fcgi",
        "check-local" => "disable",
        "max-procs" => 1
    ))
)

alias.url = (
    "/static/" => "/path/to/your/static/"
)

url.rewrite-once = (
    "^(/static($|/.*))$" => "$1",
    "^(/.*)$" => "/yourapplication.fcgi$1"
)
```

Не забудьте включить **FastCGI**, модули **alias** и **rewrite**. Эта конфигурация связывает приложение с `/yourapplication`. Если вы хотите, чтобы приложение работало в корневом URL-адресе, вам нужно обойти ошибку **lighttpd** с помощью промежуточного программного обеспечения **LighttpdCGIRootFix**.

Обязательно примените его, только если вы монтируете приложение в корневом URL-адресе. Также см. документацию **Lighty** для получения дополнительной информации о [FastCGI и Python](https://redmine.lighttpd.net/projects/lighttpd/wiki/Docs\_ModFastCGI) (обратите внимание, что явная передача сокета в **run ()** больше не требуется).

## Конфигурирование nginx

Установка приложений **FastCGI** на **nginx** немного отличается, потому что по умолчанию параметры **FastCGI** не пересылаются.

Базовая конфигурация **Flask** **FastCGI** для **nginx** выглядит так:

```perl
location = /yourapplication { rewrite ^ /yourapplication/ last; }
location /yourapplication { try_files $uri @yourapplication; }
location @yourapplication {
    include fastcgi_params;
    fastcgi_split_path_info ^(/yourapplication)(.*)$;
    fastcgi_param PATH_INFO $fastcgi_path_info;
    fastcgi_param SCRIPT_NAME $fastcgi_script_name;
    fastcgi_pass unix:/tmp/yourapplication-fcgi.sock;
}
```

Эта конфигурация связывает приложение с `/yourapplication`. Если вы хотите, чтобы он был в корневом URL-адресе, это немного проще, потому что вам не нужно выяснять, как вычислить **PATH\_INFO** и **SCRIPT\_NAME**:

```perl
location / { try_files $uri @yourapplication; }
location @yourapplication {
    include fastcgi_params;
    fastcgi_param PATH_INFO $fastcgi_script_name;
    fastcgi_param SCRIPT_NAME "";
    fastcgi_pass unix:/tmp/yourapplication-fcgi.sock;
}
```

## Запуск процессов FastCGI

Поскольку **nginx** и другие не загружают приложения **FastCGI**, вам придется делать это самостоятельно. [Супервизор может управлять процессами FastCGI](http://supervisord.org/configuration.html#fcgi-program-x-section-settings). Вы можете поискать другие менеджеры процессов **FastCGI** или написать сценарий для запуска файла `.fcgi` при загрузке, например используя сценарий **SysV** `init.d`. В качестве временного решения вы всегда можете запустить сценарий `.fcgi` внутри экрана **GNU**. См. подробные сведения `man screen` и обратите внимание, что это ручное решение, которое не сохраняется при перезапуске системы:

```bash
$ screen
$ /var/www/yourapplication/yourapplication.fcgi
```

## Отладка

Развертывания **FastCGI** обычно сложно отлаживать на большинстве веб-серверов. Очень часто единственное, что сообщает вам журнал сервера, - это что-то вроде «преждевременного окончания заголовков». Чтобы отладить приложение, единственное, что действительно может дать вам представление о том, почему оно ломается, - это переключиться на правильного пользователя и запустить приложение вручную.

В этом примере предполагается, что ваше приложение называется `application.fcgi`, а пользователь веб-сервера - `www-data`:

```bash
$ su www-data
$ cd /var/www/yourapplication
$ python application.fcgi
Traceback (most recent call last):
  File "yourapplication.fcgi", line 4, in <module>
ImportError: No module named yourapplication
```

В этом случае ошибка выглядит как «`yourapplication`» не находится на пути Python. Общие проблемы:

* Относительные пути используются. Не полагайтесь на текущий рабочий каталог.
* Код зависит от переменных среды, которые не установлены веб-сервером.
* Используются разные интерпретаторы Python.
