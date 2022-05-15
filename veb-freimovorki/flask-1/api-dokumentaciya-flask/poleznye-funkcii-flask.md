# Полезные внутренние функции Flask

## Класс flask.ctx.RequestContext( _app_, _environ_, _request=None_, _session=None_ )

Контекст запроса содержит всю информацию, относящуюся к запросу. Он создается в начале запроса и помещается в _**\_request\_ctx\_stack**_ и удаляется в конце. Он создаст адаптер URL и объект запроса для предоставленной среды **WSGI**.

Не пытайтесь использовать этот класс напрямую, вместо этого используйте [test\_request\_context ()](obekt-prilozheniya-flask.md#test\_request\_context-args-kwargs) и [request\_context ()](obekt-prilozheniya-flask.md#request\_context-environ) для создания этого объекта.

Когда контекст запроса открывается, он оценивает все функции, зарегистрированные в приложении, для выполнения **teardown** ([teardown\_request ()](obekt-prilozheniya-flask.md#teardown\_request-f)).

Контекст запроса автоматически выталкивается в конце запроса для вас. В режиме отладки контекст запроса сохраняется в случае возникновения исключений, чтобы интерактивные отладчики имели возможность проанализировать данные. В версии `0.4` это также может быть принудительно для запросов, которые не завершились ошибкой и вне режима **DEBUG**. Если установить для `'flask._preserve_context'` значение `True` в среде **WSGI**, контекст не будет появляться в конце запроса. Это используется [test\_client ()](obekt-prilozheniya-flask.md#test\_client-use\_cookies-true-kwargs), например, для реализации функции отложенной очистки.

Вы можете найти это полезным для юнит-тестов, когда вам понадобится информация из локального контекста в течение некоторого времени. Убедитесь, что вы правильно **pop ()** стек в этой ситуации, иначе ваши **unittests** будут утечкой памяти.

### copy()

_Новое в версии 0.10_.

Создает копию этого контекста запроса с тем же объектом запроса. Это можно использовать для перемещения контекста запроса в другой гринлет. Поскольку фактический объект запроса тот же, его нельзя использовать для перемещения контекста запроса в другой поток, если доступ к объекту запроса не заблокирован.

_Изменено в версии 1.1:_ используется текущий объект сеанса вместо перезагрузки исходных данных. Это предотвращает указание _**flask.session**_ на устаревший объект.

### match\_request()

Может быть переопределен подклассом, чтобы подключиться к сопоставлению запроса.

### pop( _exc=\<object object>_ )

Выдвигает контекст запроса и тем самым отвязывает его. Это также вызовет выполнение функций, зарегистрированных декоратором [teardown\_request ()](obekt-prilozheniya-flask.md#teardown\_request-f).

_Изменено в версии 0.9:_ Добавлен аргумент _**exc**_.

### push()

Привязывает контекст запроса к текущему контексту.

## flask.\_request\_ctx\_stack

Внутренний [LocalStack](https://werkzeug.palletsprojects.com/en/1.0.x/local/#werkzeug.local.LocalStack), содержащий экземпляры [RequestContext](poleznye-funkcii-flask.md#klass-flask-ctx-requestcontext-app-environ-request-none-session-none). Как правило, доступ к прокси-серверам [request](dannye-vkhodyashego-zaprosa-request.md#flask-request) и [session](sessii-flask.md#klass-flask-session) должен осуществляться вместо стека. Может быть полезно получить доступ к стеку в коде расширения.

Следующие атрибуты всегда присутствуют на каждом уровне стека:

* _**app**_ - активное приложение **Flask**.
* _**url\_adapter**_ - адаптер URL-адреса, который использовался для сопоставления запроса.
* _**request**_ - текущий объект запроса.
* _**session**_ - активный объект сеанса.
* _**g**_ - объект со всеми атрибутами объекта [flask.g](globalnyi-obekt-prilozheniya-flask.md#flask-g).
* _**flashes**_ - внутренний кеш для мигающих сообщений.

Пример использования:

```python
from flask import _request_ctx_stack

def get_session():
    ctx = _request_ctx_stack.top
    if ctx is not None:
        return ctx.session
```

## Класс flask.ctx.AppContext( _app_ )

Контекст приложения неявно связывает объект приложения с текущим потоком или гринлетом, аналогично тому, как [RequestContext](poleznye-funkcii-flask.md#klass-flask-ctx-requestcontext-app-environ-request-none-session-none) связывает информацию запроса. Контекст приложения также неявно создается, если создается контекст запроса, но приложение не находится поверх контекста отдельного приложения.

### pop( _exc=\<object object>_ )

Выталкивает контекст приложения.

### push()

Привязывает контекст приложения к текущему контексту.

## flask.\_app\_ctx\_stack

_Новое в версии 0.9_.

Внутренний [LocalStack](https://werkzeug.palletsprojects.com/en/1.0.x/local/#werkzeug.local.LocalStack), содержащий экземпляры [AppContext](poleznye-funkcii-flask.md#klass-flask-ctx-appcontext-app). Как правило, вместо стека следует обращаться к прокси [current\_app](poleznye-funkcii-i-klassy-flask.md#flask-current\_app) и [g](globalnyi-obekt-prilozheniya-flask.md#flask-g). Расширения могут обращаться к контекстам в стеке как к пространству имен для хранения данных.

## Класс flask.blueprints.BlueprintSetupState( _blueprint_, _app_, _options_, _first\_registration_ )

Объект временного хранения для регистрации _**blueprint**_ в приложении. Экземпляр этого класса создается методом [make\_setup\_state ()](obekty-blueprint.md#make\_setup\_state-app-options-first\_registration-false) и затем передается всем функциям обратного вызова регистров.

### add\_url\_rule( _rule_, _endpoint=None_, _view\_func=None_, _\*\*options_ )

Вспомогательный метод для регистрации правила (и, возможно, функции просмотра) в приложении. Конечная точка автоматически получает префикс с именем схемы _**blueprint**_.

### app _= None_

ссылка на текущее приложение

### blueprint _= None_

ссылка на схему _**blueprint**_, создавшую это состояние настройки.

### first\_registration

поскольку _**blueprints**_ могут быть зарегистрированы в приложении несколько раз, и не все хотят регистрироваться в нем несколько раз, этот атрибут можно использовать, чтобы выяснить, был ли _**blueprint**_ уже зарегистрирован в прошлом.

### options _= None_

словарь со всеми параметрами, которые были переданы методу [register\_blueprint ()](obekt-prilozheniya-flask.md#register\_blueprint-blueprint-options).

### subdomain _= None_

Поддомен, для которого должен быть активен проект, в противном случае - `None`.

### url\_defaults _= None_

Словарь со значениями URL-адреса по умолчанию, который добавляется к каждому URL-адресу, определенному в схеме _**blueprint**_.

### url\_prefix _= None_

Префикс, который следует использовать для всех URL-адресов, определенных в схеме _**blueprint**_.
