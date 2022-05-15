# Параметры развертывания Flask

Несмотря на то, что встроенный сервер **Flask** легкий и простой в использовании, **он не подходит для производства**, так как плохо масштабируется. Некоторые параметры, доступные для правильной работы **Flask** в производственной среде, описаны здесь.

Если вы хотите развернуть приложение **Flask** на сервере **WSGI**, не указанном здесь, обратитесь к документации сервера о том, как использовать с ним приложение **WSGI**. Просто помните, что ваш объект приложения **Flask** - это фактическое приложение **WSGI**.

## Варианты размещения

* [Развертывание Flask на Heroku](https://devcenter.heroku.com/articles/getting-started-with-python)
* [Развертывание Flask на Google App Engine](https://cloud.google.com/appengine/docs/standard/python3/runtime)
* [Развертывание Flask на AWS Elastic Beanstalk](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create-deploy-python-flask.html)
* [Развертывание в Azure (IIS)](https://docs.microsoft.com/en-us/azure/app-service/configure-language-python)
* [Развертывание на PythonAnywhere](https://help.pythonanywhere.com/pages/Flask/)

## Самостоятельные варианты

* [Автономные контейнеры WSGI](avtonomnye-konteinery-wsgi.md)
  * [Gunicorn](avtonomnye-konteinery-wsgi.md#gunicorn)
  * [uWSGI](avtonomnye-konteinery-wsgi.md#uwsgi)
  * [Gevent](avtonomnye-konteinery-wsgi.md#gevent)
  * [Twisted Web](avtonomnye-konteinery-wsgi.md#twisted-web)
  * [Настройка прокси](avtonomnye-konteinery-wsgi.md#nastroika-proksi)
* [uWSGI](uwsgi.md)
  * [Старт вашего приложения с uwsgi](uwsgi.md#start-vashego-prilozheniya-s-uwsgi)
  * [Конфигурирование nginx](uwsgi.md#konfigurirovanie-nginx)
* [mod\_wsgi (Apache)](mod\_wsgi-apache.md)
  * [Установка mod\_wsgi](mod\_wsgi-apache.md#ustanovka-mod\_wsgi)
  * [Создание файла .wsgi](mod\_wsgi-apache.md#sozdanie-faila-wsgi)
  * [Конфигурирование Apache](mod\_wsgi-apache.md#konfigurirovanie-apache)
  * [Исправление проблем](mod\_wsgi-apache.md#ispravlenie-problem)
  * [Поддержка автоматической перезагрузки](mod\_wsgi-apache.md#podderzhka-avtomaticheskoi-perezagruzki)
  * [Работа с виртуальными окружениями](mod\_wsgi-apache.md#rabota-s-virtualnymi-okruzheniyami)
* [FastCGI](fastcgi.md)
  * [Создание файла .fcgi](fastcgi.md#sozdanie-faila-fcgi)
  * [Конфигурирование Apache](fastcgi.md#konfigurirovanie-apache)
  * [Конфигурирование lighttpd](fastcgi.md#konfigurirovanie-lighttpd)
  * [Конфигурирование nginx](fastcgi.md#konfigurirovanie-nginx)
  * [Запуск процессов FastCGI](fastcgi.md#zapusk-processov-fastcgi)
  * [Отладка](fastcgi.md#otladka)
* [CGI](cgi.md)
  * [Создание файла .cgi](cgi.md#sozdanie-faila-cgi)
  * [Настройка сервера](cgi.md#nastroika-servera)
