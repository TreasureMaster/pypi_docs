# Ошибки приложения Flask

## Ошибки приложения

_Новое в версии 0.3_.

Приложения терпят неудачу, серверы терпят неудачу. Рано или поздно вы увидите исключение в производстве. Даже если ваш код на 100% верен, вы все равно будете время от времени видеть исключения. Почему? Потому что все остальное потерпит неудачу. Вот несколько ситуаций, в которых отличный код может привести к ошибкам сервера:

* клиент преждевременно завершил запрос, а приложение все еще считывало входящие данные
* сервер базы данных был перегружен и не смог обработать запрос
* файловая система заполнена
* жесткий диск сломался
* внутренний сервер перегружен
* ошибка программирования в используемой вами библиотеке
* сетевое подключение сервера к другой системе не удалось

И это лишь небольшая часть проблем, с которыми вы можете столкнуться. Так как же нам справиться с подобными проблемами? По умолчанию, если ваше приложение работает в производственном режиме, **Flask** отобразит для вас очень простую страницу и зарегистрирует исключение в [logger](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#logger).

Но вы можете сделать больше, и мы рассмотрим некоторые более эффективные настройки для устранения ошибок.

### Инструменты регистрации ошибок

Отправка сообщений об ошибках, даже если они критически важны, может стать непосильной, если достаточное количество пользователей обнаруживают ошибку, а файлы журналов обычно никогда не просматриваются. Вот почему мы рекомендуем использовать [Sentry](https://sentry.io/welcome/) для устранения ошибок приложения. Он доступен как проект с открытым исходным кодом [на GitHub](https://github.com/getsentry/sentry), а также доступен как [размещенная версия](https://sentry.io/signup/), которую вы можете попробовать бесплатно. **Sentry** собирает повторяющиеся ошибки, фиксирует полную трассировку стека и локальные переменные для отладки и отправляет вам письма на основе новых ошибок или пороговых значений частоты.

Чтобы использовать **Sentry**, вам необходимо установить клиент **sentry-sdk** с дополнительными зависимостями **Flask**:

```bash
$ pip install sentry-sdk[flask]
```

А затем добавьте это в свое приложение **Flask**:

```python
import sentry_sdk
from sentry_sdk.integrations.flask import FlaskIntegration

sentry_sdk.init('YOUR_DSN_HERE',integrations=[FlaskIntegration()])
```

Значение **YOUR\_DSN\_HERE** необходимо заменить значением **DSN**, полученным при установке **Sentry**.

После установки о сбоях, приводящих к внутренней ошибке сервера, автоматически сообщается в **Sentry**, и оттуда вы можете получать уведомления об ошибках.

Продолжение гласит:

* **Sentry** также поддерживает отлов ошибок из вашей рабочей очереди (**RQ**, **Celery**) аналогичным образом. Дополнительную информацию см. в [документации Python SDK](https://docs.sentry.io/platforms/python/).
* [Начало работы с **Sentry**](https://docs.sentry.io/platforms/)****
* ****[**Flask**-ориентированная документация](https://docs.sentry.io/platforms/python/guides/flask/)

### Обработчики ошибок

Возможно, вы захотите показать пользователю настраиваемые страницы ошибок при возникновении ошибки. Это можно сделать, зарегистрировав обработчики ошибок.

Обработчик ошибок - это обычная функция просмотра, которая возвращает ответ, но вместо регистрации для маршрута он регистрируется для исключения или кода состояния HTTP, который будет вызван при попытке обработать запрос.

### Регистрация

Зарегистрируйте обработчики, декорировав функцию [errorhandler ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#error\_handler). Или используйте [register\_error\_handler ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#register\_error\_handler), чтобы зарегистрировать функцию позже. Не забудьте установить код ошибки при возврате ответа.

```python
@app.errorhandler(werkzeug.exceptions.BadRequest)
def handle_bad_request(e):
    return 'bad request!', 400

# или бех декоратора
app.register_error_handler(400, handle_bad_request)
```

Подклассы [werkzeug.exceptions.HTTPException](https://werkzeug.palletsprojects.com/en/1.0.x/exceptions/#werkzeug.exceptions.HTTPException), такие как [BadRequest](https://werkzeug.palletsprojects.com/en/1.0.x/exceptions/#werkzeug.exceptions.BadRequest) и их HTTP-коды, взаимозаменяемы при регистрации обработчиков. (`BadRequest.code == 400`)

Нестандартные коды HTTP нельзя зарегистрировать по коду, потому что они не известны **Werkzeug**. Вместо этого определите подкласс [HTTPException](https://werkzeug.palletsprojects.com/en/1.0.x/exceptions/#werkzeug.exceptions.HTTPException) с соответствующим кодом, зарегистрируйте и вызовите этот класс исключения.

```python
class InsufficientStorage(werkzeug.exceptions.HTTPException):
    code = 507
    description = 'Not enough storage space.'

app.register_error_handler(InsufficientStorage, handle_507)

raise InsufficientStorage()
```

Обработчики могут быть зарегистрированы для любого класса исключений, а не только для подклассов [HTTPException](https://werkzeug.palletsprojects.com/en/1.0.x/exceptions/#werkzeug.exceptions.HTTPException) или кодов состояния HTTP. Обработчики могут быть зарегистрированы для определенного класса или для всех подклассов родительского класса.

### Обработка

Когда **Flask** перехватывает исключение во время обработки запроса, сначала выполняется поиск по коду. Если для кода не зарегистрирован обработчик, он просматривается по его иерархии классов; выбирается наиболее конкретный обработчик. Если обработчик не зарегистрирован, подклассы [HTTPException](https://werkzeug.palletsprojects.com/en/1.0.x/exceptions/#werkzeug.exceptions.HTTPException) показывают общее сообщение о своем коде, а другие исключения преобразуются в общую **500 Internal Server Error**.

Например, если возникает экземпляр [ConnectionRefusedError](https://docs.python.org/3/library/exceptions.html#ConnectionRefusedError) и зарегистрирован обработчик для [ConnectionError](https://docs.python.org/3/library/exceptions.html#ConnectionError) и **ConnectionRefusedError**, более конкретный обработчик **ConnectionRefusedError** вызывается с экземпляром исключения для генерации ответа.

Обработчики, зарегистрированные в схеме **Blueprint**, имеют приоритет над обработчиками, зарегистрированными глобально в приложении, при условии, что схема обрабатывает запрос, вызывающий исключение. Однако схема не может обрабатывать ошибки маршрутизации **404**, потому что ошибка **404** возникает на уровне маршрутизации до того, как можно определить схему.

### Универсальные обработчики исключений

Можно зарегистрировать обработчики ошибок для очень общих базовых классов, таких как **HTTPException** или даже **Exception**. Однако имейте в виду, что они поймают больше, чем вы могли ожидать.

Обработчик ошибок для **HTTPException** может быть полезен, например, для преобразования страниц ошибок **HTML** по умолчанию в **JSON**. Однако этот обработчик будет запускать вещи, которые вы не вызываете напрямую, например ошибки **404** и **405** во время маршрутизации. Будьте внимательны при создании обработчика, чтобы не потерять информацию об ошибке **HTTP**.

```python
from flask import json
from werkzeug.exceptions import HTTPException

@app.errorhandler(HTTPException)
def handle_exception(e):
    """Возвращает JSON вместо HTML при ошибках HTTP."""
    # начинает с правильных заголовков и кода состояния из ошибки
    response = e.get_response()
    # заменяет тело на JSON
    response.data = json.dumps({
        "code": e.code,
        "name": e.name,
        "description": e.description,
    })
    response.content_type = "application/json"
    return response
```

Обработчик ошибок для **Exception** может показаться полезным для изменения способа представления пользователю всех ошибок, даже необработанных. Однако это похоже на действие `except Exception:` в Python, он фиксирует **все** необработанные ошибки в этом случае, включая все коды состояния **HTTP**. В большинстве случаев будет безопаснее зарегистрировать обработчики для более конкретных исключений. Поскольку экземпляры **HTTPException** являются действительными ответами **WSGI**, вы также можете передать их напрямую.

```python
from werkzeug.exceptions import HTTPException

@app.errorhandler(Exception)
def handle_exception(e):
    # проход через HTTP-ошибки
    if isinstance(e, HTTPException):
        return e

    # теперь вы обрабатываете только исключения HTTP
    return render_template("500_generic.html", e=e), 500
```

Обработчики ошибок по-прежнему соблюдают иерархию классов исключений. Если вы зарегистрируете обработчики как для **HTTPException**, так и для **Exception**, обработчик исключений не будет обрабатывать подклассы **HTTPException**, поскольку он является более конкретным обработчиком **HTTPException**.

### Необработанные исключения

Если для исключения не зарегистрирован обработчик ошибок, вместо этого будет возвращена ошибка **500 Internal Server Error**. См. [flask.Flask.handle\_exception ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#handle\_exception) для получения информации об этом поведении.

Если для **InternalServerError** зарегистрирован обработчик ошибок, он будет вызван. Начиная с `Flask 1.1.0`, этому обработчику ошибок всегда будет передаваться экземпляр **InternalServerError**, а не исходная необработанная ошибка. Исходная ошибка доступна как `e.original_exception`. До `Werkzeug 1.0.0` этот атрибут будет существовать только во время необработанных ошибок, используйте **getattr** для доступа к нему для совместимости.

```python
@app.errorhandler(InternalServerError)
def handle_500(e):
    original = getattr(e, "original_exception", None)

    if original is None:
        # прямая ошибка 500, например abort (500)
        return render_template("500.html"), 500

    # обернутая необработанная ошибка
    return render_template("500_unhandled.html", e=original), 500
```

### Logging

См. [Logging](logging-flask.md) для получения информации о том, как регистрировать исключения, например, отправляя их администраторам по электронной почте.

## Отладка ошибок приложений

Для производственных приложений настройте приложение с ведением журнала и уведомлениями, как описано в разделе [Ошибки приложения](oshibki-prilozheniya-flask.md#oshibki-prilozheniya). В этом разделе приведены указатели при отладке конфигурации развертывания и более глубоком изучении с помощью полнофункционального отладчика Python.

### Если сомневаетесь, запускайте вручную

Возникли проблемы с настройкой приложения для работы? Если у вас есть доступ оболочки к вашему хосту, убедитесь, что вы можете запустить приложение вручную из оболочки в среде развертывания. Обязательно запускайте ту же учетную запись пользователя, что и настроенное развертывание, для устранения проблем с разрешениями. Вы можете использовать встроенный сервер разработки **Flask** с `debug = True` на вашем рабочем хосте, что помогает выявлять проблемы с конфигурацией, но обязательно сделайте это временно в контролируемой среде. Не запускайте в продакшене с `debug = True`.

### Работа с отладчиками

Чтобы копнуть глубже, возможно, для отслеживания выполнения кода, **Flask** предоставляет отладчик из коробки (см. [Режим отладки](bystryi-start-flask.md#rezhim-otladki)). Если вы хотите использовать другой отладчик Python, обратите внимание, что отладчики мешают друг другу. Вы должны установить некоторые параметры, чтобы использовать ваш любимый отладчик:

* **debug** - следует ли включать режим отладки и перехватывать исключения
* **use\_debugger** - использовать ли внутренний отладчик **Flask**
* **use\_reloader** - следует ли перезагружать и разветвлять процесс, если модули были изменены

**debug** должен иметь значение `True` (т. е. должны перехватывать исключения), чтобы два других параметра имели какое-либо значение.

Если вы используете **Aptana / Eclipse** для отладки, вам необходимо установить для **use\_debugger** и **use\_reloader** значение `False`.

Возможный полезный шаблон для настройки - установить следующее в вашем `config.yaml` (конечно, измените блок в соответствии с вашим приложением):

```yaml
FLASK:
    DEBUG: True
    DEBUG_WITH_APTANA: True
```

Тогда в точке входа вашего приложения (`main.py`) у вас может быть что-то вроде:

```python
if __name__ == "__main__":
    # Чтобы разрешить aptana получать ошибки, установите use_debugger = False
    app = create_app(config="config.yaml")

    use_debugger = app.debug and not(app.config.get('DEBUG_WITH_APTANA'))
    app.run(use_debugger=use_debugger, debug=app.debug,
            use_reloader=use_debugger, host='0.0.0.0')
```
