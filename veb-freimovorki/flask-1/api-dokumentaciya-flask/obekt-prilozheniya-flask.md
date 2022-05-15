# Объект приложения Flask

## Класс flask.Flask( _import\_name_, _static\_url\_path=None_, _static\_folder='static'_, _static\_host=None_, _host\_matching=False_, _subdomain\_matching=False_, _template\_folder='templates'_, _instance\_path=None_, _instance\_relative\_config=False_, _root\_path=None_ )

Объект **flask** реализует приложение **WSGI** и действует как центральный объект. Передается имя модуля или пакета приложения. После создания он будет действовать как центральный реестр для функций просмотра, правил URL-адресов, конфигурации шаблона и многого другого.

Имя пакета используется для разрешения ресурсов внутри пакета или папки, в которой находится модуль, в зависимости от того, разрешается ли параметр пакета в фактический пакет python (папка с файлом `__init__.py` внутри) или стандартный модуль ( просто файл `.py`).

Для получения дополнительной информации о загрузке ресурсов см. [open\_resource ()](obekt-prilozheniya-flask.md#open\_resource).

Обычно вы создаете экземпляр **Flask** в своем основном модуле или в файле `__init__.py` своего пакета следующим образом:

```python
from flask import Flask
app = Flask(__name__)
```

{% hint style="info" %}
**О первом параметре:**

Идея первого параметра состоит в том, чтобы дать **Flask** представление о том, что принадлежит вашему приложению. Это имя используется для поиска ресурсов в файловой системе, может использоваться расширениями для улучшения отладочной информации и многого другого.

Поэтому важно, что вы там предоставляете. Если вы используете один модуль, **\_\_name\_\_** всегда будет правильным значением. Однако, если вы используете пакет, обычно рекомендуется жестко указать имя вашего пакета там.

Например, если ваше приложение определено в **yourapplication/app.py**, вы должны создать его с помощью одной из двух версий ниже:

```python
app = Flask('yourapplication')
app = Flask(__name__.split('.')[0])
```

Почему так? Приложение будет работать даже с **\_\_name\_\_**, благодаря тому, как ищутся ресурсы. Однако это сделает отладку более болезненной. Некоторые расширения могут делать предположения на основе имени импорта вашего приложения. Например, расширение **Flask-SQLAlchemy** будет искать в вашем приложении код, который запускал SQL-запрос в режиме отладки. Если имя импорта не задано должным образом, эта отладочная информация теряется. (Например, он будет обрабатывать только запросы SQL в **yourapplication.app**, а не **yourapplication.views.frontend**)
{% endhint %}

#### Журнал изменений Flask()

_Новое в версии 1.0:_ добавлены параметры **host\_matching** и **static\_host**.

_Новое в версии 1.0:_ добавлен параметр **subdomain\_matching**. Сопоставление поддоменов необходимо включить вручную. Установка [SERVER\_NAME](../rukovodstvo-polzovatelya-flask/obrabotka-konfiguracii-flask.md#server\_name) не включает его неявно.

_Новое в версии 0.11_: добавлен параметр **root\_path**.

_Новое в версии 0.8:_ добавлены параметры **instance\_path** и **instance\_relative\_config**.

_Новое в версии 0.7:_ добавлены параметры **static\_url\_path**, **static\_folder** и **template\_folder**.

### Параметры класса Flask

* **import\_name** - название пакета приложения
* **static\_url\_path -** можно использовать для указания другого пути для статических файлов в Интернете. По умолчанию используется имя папки _**static\_folder**_.
* **static\_folder -** Папка со статическими файлами, которая обслуживается по `static_url_path`. Относительно `root_path` приложения или абсолютного пути. По умолчанию `'static'`.
* **static\_host -** хост, который будет использоваться при добавлении статического маршрута. По умолчанию `None`. Требуется при использовании `host_matching = True` с настроенной _**static\_folder**_.
* **host\_matching -** устанавливает атрибут `url_map.host_matching`. По умолчанию `False`.
* **subdomain\_matching -** при сопоставлении маршрутов учитывайте субдомен относительно [SERVER\_NAME](../rukovodstvo-polzovatelya-flask/obrabotka-konfiguracii-flask.md#server\_name). По умолчанию `False`.
* **template\_folder -** папка, содержащая шаблоны, которые должно использовать приложение. По умолчанию папка `'templates'` находится в корневом каталоге приложения.
* **instance\_path -** Альтернативный путь экземпляра для приложения. По умолчанию предполагается, что папка `'instance'` рядом с пакетом или модулем является путем к экземпляру.
* **instance\_relative\_config -** если установлено значение `True`, относительные имена файлов для загрузки конфигурации предполагаются относительно пути экземпляра, а не корня приложения.
* **root\_path - Flask** по умолчанию автоматически рассчитает путь к корню приложения. В определенных ситуациях это не может быть достигнуто (например, если пакет является пакетом пространства имен Python 3), и его необходимо определить вручную.

### add\_template\_filter( _f_, _name=None_ )

Регистрирует настраиваемый шаблон фильтра. Работает точно так же, как декоратор [template\_filter ()](obekt-prilozheniya-flask.md#template\_filter).

**Параметры**: _**name**_ - необязательное имя фильтра, иначе будет использовано имя функции.

### add\_template\_global( _f_, _name=None_ )

_Новое в версии 0.10_.

Регистрирует глобальную функцию настраиваемого шаблона. Работает точно так же, как декоратор [template\_global ()](obekt-prilozheniya-flask.md#template\_global).

**Параметры**: _**name**_ - необязательное имя глобальной функции, иначе будет использовано имя функции.

### add\_template\_test( _f_, _name=None_ )

_Новое в версии 0.10_.

Регистрирует собственный шаблон теста. Работает точно так же, как декоратор [template\_test ()](obekt-prilozheniya-flask.md#template\_test).

**Параметры**: _**name**_ - необязательное имя теста, иначе будет использовано имя функции.

### add\_url\_rule( _rule_, _endpoint=None_, _view\_func=None_, _provide\_automatic\_options=None_, _\*\*options_ )

Подключает правило URL. Работает точно так же, как декоратор [route ()](obekt-prilozheniya-flask.md#route). Если предоставлена _**view\_func**_, она будет зарегистрирована в конечной точке.

В основе этот пример:

```python
@app.route('/')
def index():
    pass
```

Эквивалентно следующему:

```python
def index():
    pass
app.add_url_rule('/', 'index', index)
```

Если _**view\_func**_ не указан, вам нужно будет подключить конечную точку к функции просмотра следующим образом:

```python
app.view_functions['index'] = index
```

Внутренне [route ()](obekt-prilozheniya-flask.md#route) вызывает **add\_url\_rule ()**, поэтому, если вы хотите настроить поведение с помощью подкласса, вам нужно только изменить этот метод.

Дополнительные сведения см. в разделе «[Регистрация маршрутов URL](registraciya-flask-marshrutov-url.md)».

#### Журнал изменений add\_url\_rule()

_Изменено в версии 0.6:_ **OPTIONS** добавляется автоматически как метод.

_Изменено в версии 0.2:_ добавлен параметр _**view\_func**_.

#### Параметры add\_url\_rule()

* **rule** - правило URL в виде строки
* **endpoint** - конечная точка для зарегистрированного правила URL. Сам **Flask** принимает имя функции просмотра как конечную точку
* **view\_func** - функция, вызываемая при обслуживании запроса к предоставленной конечной точке
* **provide\_automatic\_options** - определяет, должен ли метод **OPTIONS** добавляться автоматически. Этим также можно управлять, установив `view_func.provide_automatic_options = False` перед добавлением правила.
* **options** - параметры, которые будут перенаправлены в базовый объект [Rule](https://werkzeug.palletsprojects.com/en/1.0.x/routing/#werkzeug.routing.Rule). Изменением в **Werkzeug** является обработка параметров метода. Методы - это список методов, которыми должно быть ограничено это правило (**GET**, **POST** и т. д.). По умолчанию правило просто прослушивает **GET** (и неявно **HEAD**). Начиная с `Flask 0.6`, **OPTIONS** неявно добавляются и обрабатываются стандартной обработкой запросов.

### after\_request( _f_ )

Регистрирует функцию, которая будет запускаться после каждого запроса.

Ваша функция должна принимать один параметр, экземпляр [response\_class](obekt-prilozheniya-flask.md#response\_class) и возвращать новый объект ответа или то же самое (см. [process\_response ()](obekt-prilozheniya-flask.md#process\_response)).

Начиная с `Flask 0.7` эта функция может не выполняться в конце запроса в случае возникновения необработанного исключения.

### after\_request\_funcs _= None_

Словарь со списками функций, которые следует вызывать после каждого запроса. Ключ словаря - это имя _**blueprint**_, для которой активна эта функция, `None` для всех запросов. Например, это можно использовать для закрытия соединений с базой данных. Чтобы зарегистрировать здесь функцию, используйте декоратор [after\_request ()](obekt-prilozheniya-flask.md#after\_request-f).

### app\_context()

_Новое в версии 0.9_.

Создает [AppContext](poleznye-funkcii-flask.md#klass-flask-ctx-appcontext). Используйте как блок **with** для проталкивания контекста, который будет указывать [current\_app](poleznye-funkcii-i-klassy-flask.md#flask-current\_app) на это приложение.

Контекст приложения автоматически передается [RequestContext.push ()](poleznye-funkcii-flask.md#push) при обработке запроса и при выполнении команды CLI. Используйте это, чтобы вручную создать контекст вне этих ситуаций.

```python
with app.app_context():
    init_db()
```

См. «[Контекст приложения](../rukovodstvo-polzovatelya-flask/kontekst-prilozheniya-flask.md)».

### app\_ctx\_globals\_class

псевдоним [flask.ctx.\_AppCtxGlobals](globalnyi-obekt-prilozheniya-flask.md#klass-flask-ctx-\_appctxglobals)

### auto\_find\_instance\_path()

_Новое в версии 0.8_.

Пытается найти путь к экземпляру, если он не был предоставлен конструктору класса приложения. По сути, он вычислит путь к папке с именем `instance` рядом с вашим основным файлом или пакетом.

### before\_first\_request( _f_ )

_Новое в версии 0.8_.

Регистрирует функцию, которая будет запускаться перед первым запросом к этому экземпляру приложения.

Функция будет вызываться без аргументов, а ее возвращаемое значение игнорируется.

### before\_first\_request\_funcs _= None_

_Новое в версии 0.8_.

Список функций, которые будут вызываться в начале первого запроса к этому экземпляру. Чтобы зарегистрировать функцию, используйте декоратор [before\_first\_request ()](obekt-prilozheniya-flask.md#before\_first\_request-f).

### before\_request( _f_ )

Регистрирует функцию для запуска перед каждым запросом.

Например, это можно использовать для открытия соединения с базой данных или для загрузки вошедшего в систему пользователя из сеанса.

Функция будет вызываться без аргументов. Если он возвращает значение, отличное от `None`, значение обрабатывается так, как если бы оно было возвращаемым значением из представления, и дальнейшая обработка запроса прекращается.

### before\_request\_funcs _= None_

Словарь со списками функций, которые будут вызываться в начале каждого запроса. Ключ словаря - это имя схемы _ **blueprint**_, для которой активна эта функция, или `None` для всех запросов. Чтобы зарегистрировать функцию, используйте декоратор [before\_request ()](obekt-prilozheniya-flask.md#before\_request-f).

### blueprints _= None_

_Новое в версии 0.7_.

все прилагаемые схемы _ **blueprints**_ в словаре по именам. _**Blueprints**_ можно прикреплять несколько раз, поэтому этот словарь не сообщает вам, как часто они прикреплялись.

### config _= None_

Словарь конфигурации как [Config](konfigurirovanie-flask.md#klass-flask-config). Он ведет себя точно так же, как обычный словарь, но поддерживает дополнительные методы для загрузки конфигурации из файлов.

### config\_class

псевдоним **flask.config.Config**

### context\_processor( _f_ )

Регистрирует функцию обработчика контекста шаблона.

### create\_global\_jinja\_loader()

_Новое в версии 0.7_.

Создает загрузчик для среды **Jinja2**. Может использоваться для переопределения только загрузчика и сохранения остальных без изменений. Не рекомендуется отменять эту функцию. Вместо этого следует переопределить функцию [jinja\_loader ()](obekt-prilozheniya-flask.md#jinja\_loader).

Глобальный загрузчик осуществляет обмен между загрузчиками приложения и отдельными схемами _**blueprints**_.

### create\_jinja\_environment()

_Новое в версии 0.5_.

Создает среду **Jinja** на основе [jinja\_options](obekt-prilozheniya-flask.md#jinja\_options) и различных связанных с **Jinja** методов приложения. Изменение **jinja\_options** после этого не даст никакого эффекта. Также добавляет в среду глобальные объекты и фильтры, связанные с **Flask**.

_Изменено в версии 0.11:_ `Environment.auto_reload` установлен в соответствии с опцией конфигурации **TEMPLATES\_AUTO\_RELOAD**.

### create\_url\_adapter()

_Новое в версии 0.6_.

Создает адаптер URL для данного запроса. Адаптер URL-адреса создается в точке, где контекст запроса еще не настроен, поэтому запрос передается явно.

_Изменено в версии 1.0:_ [SERVER\_NAME](../rukovodstvo-polzovatelya-flask/obrabotka-konfiguracii-flask.md#server\_name) больше не включает неявное сопоставление поддоменов. Вместо этого используйте **subdomain\_matching**.

_Изменено в версии 0.9:_ теперь его также можно вызывать без объекта запроса при создании адаптера URL для контекста приложения.

### _свойство_ debug

Включен ли режим отладки. При использовании `flask run` для запуска сервера разработки интерактивный отладчик будет отображаться для необработанных исключений, и сервер будет перезагружен при изменении кода. Это соответствует ключу конфигурации [DEBUG](../rukovodstvo-polzovatelya-flask/obrabotka-konfiguracii-flask.md#debug). Это включено, когда [env](obekt-prilozheniya-flask.md#env) является `«development»` и переопределено переменной среды **FLASK\_DEBUG**. Если задано в коде, он может вести себя не так, как ожидалось.

**Не включайте режим отладки при развертывании в производственной среде.**

По умолчанию: `True`, если [env](obekt-prilozheniya-flask.md#env) - `'development'`, или `False` в противном случае.

### default\_config = { _список параметров ниже_ }

Параметры конфигурации по умолчанию.

* &#x20;_'APPLICATION\_ROOT': '/',_
* &#x20;_'DEBUG': None,_
* &#x20;_'ENV': None,_
* &#x20;_'EXPLAIN\_TEMPLATE\_LOADING': False,_
* &#x20;_'JSONIFY\_MIMETYPE': 'application/json',_
* &#x20;_'JSONIFY\_PRETTYPRINT\_REGULAR': False,_
* &#x20;_'JSON\_AS\_ASCII': True,_
* &#x20;_'JSON\_SORT\_KEYS': True,_
* &#x20;_'MAX\_CONTENT\_LENGTH': None,_
* &#x20;_'MAX\_COOKIE\_SIZE': 4093,_
* &#x20;_'PERMANENT\_SESSION\_LIFETIME': datetime.timedelta(days=31),_
* &#x20;_'PREFERRED\_URL\_SCHEME': 'http',_
* &#x20;_'PRESERVE\_CONTEXT\_ON\_EXCEPTION': None,_
* &#x20;_'PROPAGATE\_EXCEPTIONS': None,_
* &#x20;_'SECRET\_KEY': None,_
* &#x20;_'SEND\_FILE\_MAX\_AGE\_DEFAULT': datetime.timedelta(seconds=43200),_
* &#x20;_'SERVER\_NAME': None,_
* &#x20;_'SESSION\_COOKIE\_DOMAIN': None,_
* &#x20;_'SESSION\_COOKIE\_HTTPONLY': True,_
* &#x20;_'SESSION\_COOKIE\_NAME': 'session',_
* &#x20;_'SESSION\_COOKIE\_PATH': None,_
* &#x20;_'SESSION\_COOKIE\_SAMESITE': None,_
* &#x20;_'SESSION\_COOKIE\_SECURE': False,_
* &#x20;_'SESSION\_REFRESH\_EACH\_REQUEST': True,_
* &#x20;_'TEMPLATES\_AUTO\_RELOAD': None,_
* &#x20;_'TESTING': False,_
* &#x20;_'TRAP\_BAD\_REQUEST\_ERRORS': None,_
* &#x20;_'TRAP\_HTTP\_EXCEPTIONS': False,_
* &#x20;_'USE\_X\_SENDFILE': False_

### dispatch\_request()

Осуществляет отправку запроса. Соответствует URL-адресу и возвращает возвращаемое значение представления или обработчика ошибок. Это не обязательно должен быть объект ответа. Чтобы преобразовать возвращаемое значение в правильный объект ответа, вызовите [make\_response ()](poleznye-funkcii-i-klassy-flask.md#flask-make\_response-args).

_Изменено в версии 0.7:_ это больше не обрабатывает исключения, этот код был перемещен в новый [full\_dispatch\_request ()](obekt-prilozheniya-flask.md#full\_dispatch\_request).

### do\_teardown\_appcontext( _exc=\<object object>_ )

_Новое в версии 0.9_.

Вызывается прямо перед открытием контекста приложения.

При обработке запроса контекст приложения появляется после контекста запроса. См. [do\_teardown\_request ()](obekt-prilozheniya-flask.md#do\_teardown\_request).

Это вызывает все функции, оформленные с помощью **teardown\_appcontext ()**. Затем отправляется сигнал [appcontext\_tearing\_down](signaly-flask.md#flask-appcontext\_tearing\_down).

Это вызывается [AppContext.pop ()](poleznye-funkcii-flask.md#pop-1).

### do\_teardown\_request( _exc=\<object object>_ )

Вызывается после отправки запроса и возврата ответа, непосредственно перед открытием контекста запроса.

Это вызывает все функции, оформленные с помощью [teardown\_request ()](obekt-prilozheniya-flask.md#teardown\_request) и [Blueprint.teardown\_request ()](obekty-blueprint.md#teardown\_request), если _**blueprint**_ обработал запрос. Наконец, отправляется сигнал [request\_tearing\_down](signaly-flask.md#flask-request\_tearing\_down).

Это вызывается [RequestContext.pop ()](poleznye-funkcii-flask.md#pop), который может быть отложен во время тестирования для сохранения доступа к ресурсам.

**Параметры**: _**exc**_ - При отправке запроса возникло необработанное исключение. Обнаруживается из текущей информации об исключении, если не передан. Передается каждой _**teardown**_ функции.

_Изменено в версии 0.9:_ Добавлен аргумент _**exc**_.

### endpoint( _endpoint_ )

Декоратор для регистрации функции как конечной точки. Пример:

```python
@app.endpoint('example.endpoint')
def example():
    return "example"
```

**Параметры**: _**endpoint**_ - имя конечной точки.

### env

В какой среде работает приложение. **Flask** и расширения могут включать поведение в зависимости от среды, например включение режима отладки. Соответствует конфигурационному ключу [ENV](../rukovodstvo-polzovatelya-flask/obrabotka-konfiguracii-flask.md#env). Это устанавливается переменной среды **FLASK\_ENV** и может вести себя не так, как ожидалось, если установлено в коде.

**Не включайте** `'development'` **при развертывании в производственной среде.**

По умолчанию: `'production'`

### error\_handler\_spec _= None_

Словарь всех зарегистрированных обработчиков ошибок. Ключом является `None` для обработчиков ошибок, активных в приложении, в противном случае ключ - это имя схемы _**blueprint**_. Каждый ключ указывает на другой словарь, где ключ является кодом состояния исключения http. Специальный ключ `None` указывает на список кортежей, где первый элемент - это класс для проверки экземпляра, а второй - функция обработчика ошибок.

Чтобы зарегистрировать обработчик ошибок, используйте декоратор [errorhandler ()](obekt-prilozheniya-flask.md#errorhandler).

### errorhandler( _code\_or\_exception_ )

Регистрирует функцию для обработки ошибок по коду или классу исключения.

Декоратор, который используется для регистрации функции с кодом ошибки. Пример:

```python
@app.errorhandler(404)
def page_not_found(error):
    return 'This page does not exist', 404
```

Вы также можете зарегистрировать обработчики для произвольных исключений:

```python
@app.errorhandler(DatabaseError)
def special_exception_handler(error):
    return 'Database connection failed', 500
```

_Новое в версии 0.7:_ использование [register\_error\_handler ()](obekt-prilozheniya-flask.md#register\_error\_handler) вместо непосредственного изменения [error\_handler\_spec](obekt-prilozheniya-flask.md#error\_handler\_spec-none) для обработчиков ошибок всего приложения.

_Новое в версии 0.7:_ теперь можно дополнительно регистрировать пользовательские типы исключений, которые не обязательно должны быть подклассом класса [HTTPException](https://werkzeug.palletsprojects.com/en/1.0.x/exceptions/#werkzeug.exceptions.HTTPException).

**Параметры**: _**code\_or\_exception**_ - код как целое число для обработчика или произвольное исключение.

### extensions _= None_

_Новое в версии 0.7._

место, где расширения могут сохранять конкретное состояние приложения. Например, здесь расширение может хранить движки баз данных и подобные вещи. Для обратной совместимости расширения должны регистрироваться следующим образом:

```python
if not hasattr(app, 'extensions'):
    app.extensions = {}
app.extensions['extensionname'] = SomeObject()
```

Ключ должен совпадать с названием модуля расширения. Например, в случае расширения `«Flask-Foo»` в **flask\_foo** ключом будет `'foo'`.

### full\_dispatch\_request()

_Новое в версии 0.7._

Отправляет запрос и, кроме того, выполняет предварительную и постобработку запроса, а также перехват исключений HTTP и обработку ошибок.

### get\_send\_file\_max\_age( _filename_ )

_Новое в версии 0.9._

Предоставляет _**cache\_timeout**_ по умолчанию для функций [send\_file ()](poleznye-funkcii-i-klassy-flask.md#flask-send\_file-filename\_or\_fp-mimetype-none-as\_attachment-false-attachment\_filename-none-add\_etags-true-cache\_timeout-none-conditional-false-last\_modified-none).

По умолчанию эта функция возвращает **SEND\_FILE\_MAX\_AGE\_DEFAULT** из конфигурации [current\_app](poleznye-funkcii-i-klassy-flask.md#flask-current\_app).

Статические файловые функции, такие как [send\_from\_directory ()](poleznye-funkcii-i-klassy-flask.md#flask-send\_from\_directory-directory-filename-options), используют эту функцию, а [send\_file ()](poleznye-funkcii-i-klassy-flask.md#flask-send\_file-filename\_or\_fp-mimetype-none-as\_attachment-false-attachment\_filename-none-add\_etags-true-cache\_timeout-none-conditional-false-last\_modified-none) вызывает эту функцию в [current\_app](poleznye-funkcii-i-klassy-flask.md#flask-current\_app), когда заданное _**cache\_timeout**_ равно `None`. Если в **send\_file ()** задано _**cache\_timeout**_, используется этот тайм-аут; в противном случае вызывается этот метод.

Это позволяет подклассам изменять поведение при отправке файлов на основе имени файла. Например, чтобы установить время ожидания кеширования для файлов `.js` равным 60 секундам:

```python
class MyFlask(flask.Flask):
    def get_send_file_max_age(self, name):
        if name.lower().endswith('.js'):
            return 60
        return flask.Flask.get_send_file_max_age(self, name)
```

### _свойство_ got\_first\_request

_Новое в версии 0.8._

Для этого атрибута установлено значение `True`, если приложение начало обрабатывать первый запрос.

### handle\_exception( _e_ )

_Новое в версии 0.3._

Обработка исключения, с которым не связан обработчик ошибок, или которое было вызвано обработчиком ошибок. Это всегда вызывает ошибку `500 InternalServerError`.

Всегда отправляет сигнал [got\_request\_exception](signaly-flask.md#flask-got\_request\_exception).

Если [propagate\_exceptions](obekt-prilozheniya-flask.md#svoistvo-propagate\_exceptions) имеет значение `True`, например, в режиме отладки, ошибка будет повторно вызвана, чтобы отладчик мог ее отобразить. В противном случае регистрируется исходное исключение и возвращается [InternalServerError](https://werkzeug.palletsprojects.com/en/1.0.x/exceptions/#werkzeug.exceptions.InternalServerError).

Если обработчик ошибок зарегистрирован для **InternalServerError** или **500**, он будет использован. Для согласованности обработчик всегда будет получать **InternalServerError**. Исходное необработанное исключение доступно как `e.original_exception`.

{% hint style="info" %}
До **Werkzeug 1.0.0** ошибка **InternalServerError** не всегда имела атрибут **original\_exception**. Используйте **getattr (e, "original\_exception", None)**, чтобы смоделировать поведение для совместимости.
{% endhint %}

_Изменено в версии 1.1.0:_ Всегда передает экземпляр **InternalServerError** в обработчик, устанавливая для **original\_exception** значение необработанной ошибки.

_Изменено в версии 1.1.0:_ функции **after\_request** и другая финализация выполняется даже для ответа **500** по умолчанию, когда нет обработчика.

### handle\_http\_exception( _e_ )

_Новое в версии 0.3._

Обрабатывает исключение HTTP. По умолчанию это вызовет зарегистрированные обработчики ошибок и вернет исключение в качестве ответа.

_Изменено в версии 1.0.3:_ исключение **RoutingException**, используемое внутри для таких действий, как перенаправление косой черты во время маршрутизации, не передается обработчикам ошибок.

_Изменено в версии 1.0:_ Исключения ищутся по коду **и** с помощью MRO, поэтому подклассы **HTTPExcpetion** могут обрабатываться с помощью универсального обработчика для базового **HTTPException**.

### handle\_url\_build\_error( _error_, _endpoint_, _values_ )

Обработка ошибки **BuildError** в [url\_for ()](poleznye-funkcii-i-klassy-flask.md#flask-url\_for-endpoint-values).

### handle\_user\_exception( _e_ )

_Новое в версии 0.7._

Этот метод вызывается всякий раз, когда возникает исключение, которое необходимо обработать. Особым случаем является исключение **HTTPException**, которое перенаправляется методу [handle\_http\_exception ()](obekt-prilozheniya-flask.md#handle\_http\_exception-e). Эта функция либо вернет значение ответа, либо повторно вызовет исключение с той же трассировкой.

_Изменено в версии 1.0:_ Ключевые ошибки, возникающие из данных запроса, таких как **form**, показывают неверный ключ в режиме отладки, а не общее сообщение о неверном запросе.

### _свойство_ has\_static\_folder

_Новое в версии 0.5._

Это `True`, если в контейнере объекта, привязанного к пакету, есть папка для статических файлов.

### import\_name _= None_

Имя пакета или модуля, которому принадлежит это приложение. Не изменяйте это значение, если оно установлено конструктором.

### inject\_url\_defaults( _endpoint_, _values_ )

_Новое в версии 0.7._

Внедряет URL-адреса по умолчанию для данной конечной точки непосредственно в переданный словарь значений. Это используется внутри и автоматически вызывается при построении URL.

### instance\_path _= None_

_Новое в версии 0.8._

Содержит путь к папке экземпляра.

### iter\_blueprints()

_Новое в версии 0.11._

Обходит все схемы _**blueprints**_ в порядке их регистрации.

### jinja\_env

Среда **Jinja**, используемая для загрузки шаблонов.

Среда создается при первом обращении к этому свойству. Изменение [jinja\_options](obekt-prilozheniya-flask.md#jinja\_options) после этого не даст никакого эффекта.

### jinja\_environment

псевдоним **flask.templating.Environment**

### jinja\_loader

_Новое в версии 0.5._

Загрузчик **Jinja** для этого связанного с пакетом объекта.

### jinja\_options _= { 'extensions': \[ 'jinja2.ext.autoescape', 'jinja2.ext.with\_' ] }_

Параметры, которые передаются в среду **Jinja** в [create\_jinja\_environment ()](obekt-prilozheniya-flask.md#create\_jinja\_environment). Изменение этих параметров после создания среды (доступ к [jinja\_env](obekt-prilozheniya-flask.md#jinja\_env)) не повлияет.

_Изменено в версии 1.1.0:_ это **dict** вместо **ImmutableDict**, чтобы упростить настройку.

### json\_decoder

псевдоним [flask.json.JSONDecoder](podderzhka-json-flask.md#class-flask-json-jsondecoder)

### json\_encoder

псевдоним [flask.json.JSONEncoder](podderzhka-json-flask.md#class-flask-json-jsonencoder)

### log\_exception( _exc\_info_ )

_Новое в версии 0.8._

Регистрирует в журнале исключение. Это вызывается [handle\_exception ()](obekt-prilozheniya-flask.md#handle\_exception-e), если отладка отключена, прямо перед вызовом обработчика. Реализация по умолчанию регистрирует исключение как ошибку в [logger](obekt-prilozheniya-flask.md#logger).

### logger

_Новое в версии 0.3._

Стандартный Python [Logger](https://docs.python.org/3/library/logging.html#logging.Logger) для приложения с тем же именем, что и [name](obekt-prilozheniya-flask.md#name).

В режиме отладки журналируемый **level** будет установлен на **DEBUG**.

Если обработчики не настроены, будет добавлен обработчик по умолчанию. См. «[Ведение журнала](../rukovodstvo-polzovatelya-flask/logging-flask.md)» для получения дополнительной информации.

_Изменено в версии 1.1.0:_ Регистратор **logger** использует то же имя, что и [name](obekt-prilozheniya-flask.md#name), а не жестко запрограммированный `"flask.app"`.

_Изменено в версии 1.0.0:_ Упрощено поведение. Регистратор **logger** всегда называется `«flask.app»`. Уровень устанавливается только во время настройки, `app.debug` не проверяется каждый раз. Используется только один формат, а не разные в зависимости от `app.debug`. Обработчики не удаляются, и обработчик добавляется только в том случае, если обработчики еще не настроены.

### make\_config( _instance\_relative=False_ )

_Новое в версии 0.8._

Используется для создания атрибута **config** конструктором **Flask**. Параметр _**instance\_relative**_ передается из конструктора **Flask** (он называется _**instance\_relative\_config**_) и указывает, должна ли конфигурация быть относительно пути экземпляра или корневого пути приложения.

### make\_default\_options\_response()

_Новое в версии 0.7._

Этот метод вызывается для создания ответа **OPTIONS** по умолчанию. Это можно изменить путем создания подклассов, чтобы изменить поведение ответов **OPTIONS** по умолчанию.

### make\_null\_session()

_Новое в версии 0.7._

Создает новый экземпляр отсутствующего сеанса. Вместо переопределения этого метода мы рекомендуем заменить [session\_interface](obekt-prilozheniya-flask.md#session\_interface).

### make\_response( _rv_ )

Преобразуйте возвращаемое значение из функции представления в экземпляр [response\_class](obekt-prilozheniya-flask.md#response\_class).

**Параметры: **_**rv**_** -**

возвращаемое значение из функции просмотра. Функция просмотра должна возвращать ответ. Возврат `None` или завершение просмотра без возврата недопустимы. Для _**view\_rv**_ разрешены следующие типы:

* **str** (**unicode** в Python 2) - Объект ответа создается со строкой, закодированной в UTF-8, в качестве тела.
* **bytes** (**str** в Python 2) - Объект ответа создается с байтами в качестве тела.
* **dict** - Словарь, который будет обработан _**jsonify**_ перед возвратом.
* **tuple** - Либо `(body, status, headers)`, `(body, status)` или `(body, headers)`, где _**body**_ - это любой из других разрешенных здесь типов, _**status**_ - это строка или целое число, а _**headers**_ - это словарь или список кортежей `(key, value)`. Если _**body**_ является экземпляром [response\_class](obekt-prilozheniya-flask.md#response\_class), _**status**_ перезаписывает существующее значение, а _**headers**_ расширяются.
* [response\_class](obekt-prilozheniya-flask.md#response\_class) - Объект возвращается без изменений.
* другой класс [Response](https://werkzeug.palletsprojects.com/en/1.0.x/wrappers/#werkzeug.wrappers.Response) - Объект приводится к [response\_class](obekt-prilozheniya-flask.md#response\_class).
* [callable()](https://docs.python.org/3/library/functions.html#callable) - Функция вызывается как приложение **WSGI**. Результат используется для создания объекта ответа.

_Изменено в версии 0.9:_ ранее кортеж интерпретировался как аргументы для объекта ответа.

### make\_shell\_context()

_Новое в версии 0.11._

Возвращает контекст оболочки для интерактивной оболочки для этого приложения. Это запускает все зарегистрированные процессоры контекста оболочки.

### name

_Новое в версии 0.8._

Название приложения. Обычно это имя импорта, с той разницей, что оно определяется из файла запуска, если имя импорта является основным. Это имя используется в качестве отображаемого имени, когда **Flask** требуется имя приложения. Его можно установить и переопределить, чтобы изменить значение.

### open\_instance\_resource( _resource_, _mode='rb'_ )

Открывает ресурс из папки экземпляра приложения ([instance\_path](obekt-prilozheniya-flask.md#instance\_path-none)). В остальном работает как [open\_resource ()](obekt-prilozheniya-flask.md#open\_resource). Ресурсы экземпляра также могут быть открыты для записи.

**Параметры:**

* **resource** - название ресурса. Для доступа к ресурсам внутри подпапок используйте косую черту в качестве разделителя.
* **mode** - режим открытия файла ресурсов, по умолчанию `«rb»`.

### open\_resource( _resource_, _mode='rb'_ )

Открывает ресурс из папки ресурсов приложения. Чтобы увидеть, как это работает, рассмотрим следующую структуру папок:

```bash
/myapplication.py
/schema.sql
/static
    /style.css
/templates
    /layout.html
    /index.html
```

Если вы хотите открыть файл `schema.sql`, сделайте следующее:

```python
with app.open_resource('schema.sql') as f:
    contents = f.read()
    do_something_with(contents)
```

**Параметры:**

* **resource** - название ресурса. Для доступа к ресурсам внутри подпапок используйте косую черту в качестве разделителя.
* **mode** - Открыть файл в этом режиме. Поддерживается только чтение, допустимые значения - `«r»` (или `«rt»`) и `«rb»`.

### open\_session( _request_ )

Создает или открывает новый сеанс **session**. Реализация по умолчанию хранит все данные сеанса в подписанном файле **cookie**. Для этого необходимо установить [secret\_key](obekt-prilozheniya-flask.md#secret\_key). Вместо переопределения этого метода мы рекомендуем заменить [session\_interface](obekt-prilozheniya-flask.md#session\_interface).

**Параметры**: _**request**_ - экземпляр [request\_class](obekt-prilozheniya-flask.md#request\_class).

### permanent\_session\_lifetime

[timedelta](https://docs.python.org/3/library/datetime.html#datetime.timedelta), который используется для установки даты истечения срока постоянного сеанса. Значение по умолчанию - 31 день, поэтому постоянный сеанс просуществует примерно один месяц.

Этот атрибут также можно настроить из **config** с помощью ключа конфигурации **PERMANENT\_SESSION\_LIFETIME**. По умолчанию `timedelta (days = 31)`

### preprocessor\_request()

Вызывается перед отправкой запроса. Вызывает [url\_value\_preprocessors](obekt-prilozheniya-flask.md#url\_value\_preprocessors), зарегистрированные в приложении, и текущий _**blueprint**_ (если есть). Затем вызывает [before\_request\_funcs](obekt-prilozheniya-flask.md#before\_request\_funcs-none), зарегистрированный в приложении и _**blueprint**_.

Если какой-либо обработчик [before\_request ()](obekt-prilozheniya-flask.md#before\_request-f) возвращает значение, отличное от `None`, значение обрабатывается так, как если бы оно было возвращаемым значением из представления, и дальнейшая обработка запроса прекращается.

### _свойство_ preserve\_context\_on\_exception

_Новое в версии 0.7._

Возвращает значение конфигурации **PRESERVE\_CONTEXT\_ON\_EXCEPTION**, если оно установлено, в противном случае возвращается разумное значение по умолчанию.

### process\_response( _response_ )

Может быть переопределено для изменения объекта ответа перед его отправкой на сервер **WSGI**. По умолчанию это вызовет все декорированные функции [after\_request ()](obekt-prilozheniya-flask.md#after\_request-f).

_Изменено в версии 0.5:_ Начиная с `Flask 0.5` функции, зарегистрированные для после выполнения запроса, вызываются в обратном порядке регистрации.

**Параметры**: _**response**_ - объект [response\_class](obekt-prilozheniya-flask.md#response\_class).

**Возвращает**: новый объект ответа или тот же самый, должен быть экземпляром [response\_class](obekt-prilozheniya-flask.md#response\_class).

### _свойство_ propagate\_exceptions

_Новое в версии 0.7._

Возвращает значение конфигурации **PROPAGATE\_EXCEPTIONS**, если оно установлено, в противном случае возвращается разумное значение по умолчанию.

### register\_blueprint( _blueprint_, _\*\*options_ )

_Новое в версии 0.7._

Регистрирует [Blueprint](obekty-blueprint.md#klass-flask-blueprint) в приложении. Аргументы ключевого слова, переданные этому методу, переопределят значения по умолчанию, установленные в _**blueprint**_.

Вызывает метод [register ()](obekty-blueprint.md#register) схемы _**blueprint**_ после записи схемы в [blueprints](obekt-prilozheniya-flask.md#blueprints-none) приложения.

**Параметры:**

* **blueprint** - Схема _**blueprint**_ для регистрации.
* **url\_prefix** - Маршруты схем _**blueprints**_ будут иметь этот префикс.
* **subdomain** - Маршруты **Blueprint** будут совпадать на этом поддомене.
* **url\_defaults** - Маршруты **Blueprint** будут использовать эти значения по умолчанию для аргументов представления.
* **options** - Дополнительные аргументы ключевых слов передаются в [BlueprintSetupState](poleznye-funkcii-flask.md#klass-flask-blueprints-blueprintsetupstate). Доступ к ним можно получить в обратных вызовах [record ()](obekty-blueprint.md#record).

### register\_error\_handler( _code\_or\_exception_, _f_ )

_Новое в версии 0.7._

Альтернативная функция присоединения ошибки к декоратору [errorhandler ()](obekt-prilozheniya-flask.md#errorhandler-code\_or\_exception), более простая для использования без использования декоратора.

### request\_class

псевдоним **flask.wrappers.Request**

### request\_context( _environ_ )

Создает [RequestContext](poleznye-funkcii-flask.md#klass-flask-ctx-requestcontext), представляющий среду **WSGI**. Используйте блок **with**, чтобы протолкнуть контекст, который сделает точку [request](dannye-vkhodyashego-zaprosa-request.md#flask-request) при этом запросе.

См. [Контекст запроса](../rukovodstvo-polzovatelya-flask/kontekst-zaprosa-flask.md).

Обычно вы не должны вызывать это из своего собственного кода. Контекст запроса автоматически передается [wsgi\_app ()](obekt-prilozheniya-flask.md#wsgi\_app) при обработке запроса. Используйте [test\_request\_context ()](obekt-prilozheniya-flask.md#test\_request\_context) для создания среды и контекста вместо этого метода.

**Параметры**: _**environ**_ - среда **WSGI**.

### response\_class

псевдоним **flask.wrappers.Response**

### root\_path _= None_

Абсолютный путь к пакету в файловой системе. Используется для поиска ресурсов, содержащихся в пакете.

### route( _rule_, _\*\*options_ )

Декоратор, который используется для регистрации функции просмотра для данного правила URL. Это делает то же самое, что и [add\_url\_rule ()](obekt-prilozheniya-flask.md#add\_url\_rule-rule-endpoint-none-view\_func-none-provide\_automatic\_options-none-options), но предназначено для использования декоратора:

```python
@app.route('/')
def index():
    return 'Hello World'
```

Дополнительные сведения см. в разделе «[Регистрация маршрутов URL](registraciya-flask-marshrutov-url.md)».

**Параметры:**

* **rule** - правило URL в виде строки
* **endpoint** - конечная точка для зарегистрированного правила URL. Сам **Flask** принимает имя функции просмотра как конечную точку
* **options** - параметры, которые будут перенаправлены в базовый объект [Rule](https://werkzeug.palletsprojects.com/en/1.0.x/routing/#werkzeug.routing.Rule). Изменением в **Werkzeug** является обработка параметров метода. _**methods**_ - это список методов, которыми должно быть ограничено это правило (**GET**, **POST** и т. д.). По умолчанию правило просто прослушивает **GET** (и неявно **HEAD**). Начиная с `Flask 0.6`, **OPTIONS** неявно добавляются и обрабатываются стандартной обработкой запросов.

### run( _host=None_, _port=None_, _debug=None_, _load\_dotenv=True_, _\*\*options_ )

Запускает приложение на локальном сервере разработки.

Не используйте **run ()** в производственных условиях. Он не предназначен для удовлетворения требований к безопасности и производительности рабочего сервера. Вместо этого ознакомьтесь с рекомендациями по серверу **WSGI** в разделе [Варианты развертывания](../rukovodstvo-polzovatelya-flask/flask-i-varianty-razvertyvaniya/).

Если установлен флаг [debug](obekt-prilozheniya-flask.md#svoistvo-debug), сервер автоматически перезагрузится для изменений кода и покажет отладчик в случае возникновения исключения.

Если вы хотите запустить приложение в режиме отладки **debug**, но отключить выполнение кода в интерактивном отладчике, вы можете передать `use_evalex = False` в качестве параметра. Это сохранит экран трассировки отладчика активным, но отключит выполнение кода.

Не рекомендуется использовать эту функцию для разработки **development** с автоматической перезагрузкой, так как она плохо поддерживается. Вместо этого вам следует использовать поддержку **run** сценария командной строки **flask**.

{% hint style="info" %}
**Иметь ввиду:**

**Flask** подавит любую ошибку сервера с помощью общей страницы ошибок, если он не находится в режиме отладки. Таким образом, чтобы включить только интерактивный отладчик без перезагрузки кода, вы должны вызвать **run ()** с **debug = True** и **use\_reloader = False**. Установка **use\_debugger** в **True** без перехода в режим отладки не приведет к обнаружению каких-либо исключений, потому что их не будет.
{% endhint %}

**Параметры:**

* **host** - имя хоста для прослушивания. Установите значение `'0.0.0.0'`, чтобы сервер также был доступен извне. По умолчанию `'127.0.0.1'` или хост в переменной конфигурации **SERVER\_NAME**, если она присутствует.
* **port** - порт веб-сервера. По умолчанию **5000** или порт, определенный в переменной конфигурации **SERVER\_NAME**, если она присутствует.
* **debug** - если указан, включает или отключает режим отладки. См. [debug](obekt-prilozheniya-flask.md#svoistvo-debug).
* **load\_dotenv** - Загружает ближайшие файлы `.env` и `.flaskenv`, чтобы установить переменные среды. Также изменит рабочий каталог на каталог, содержащий первый найденный файл.
* options - параметры, которые будут перенаправлены на базовый сервер **Werkzeug**. См. [Werkzeug.serving.run\_simple ()](https://werkzeug.palletsprojects.com/en/1.0.x/serving/#werkzeug.serving.run\_simple) для получения дополнительной информации.

_Изменено в версии 1.0:_ если он установлен, **python-dotenv** будет использоваться для загрузки переменных среды из файлов `.env` и `.flaskenv`. Если установлено, переменные среды **FLASK\_ENV** и **FLASK\_DEBUG** переопределят [env](obekt-prilozheniya-flask.md#env) и [debug](obekt-prilozheniya-flask.md#svoistvo-debug). По умолчанию включен потоковый режим.

_Изменено в версии 0.10:_ порт по умолчанию теперь выбирается из переменной **SERVER\_NAME**.

### save\_session( _session_, _response_ )

Сохраняет сеанс, если ему нужны обновления. Для реализации по умолчанию проверьте [open\_session ()](obekt-prilozheniya-flask.md#open\_session-request). Вместо переопределения этого метода мы рекомендуем заменить [session\_interface](obekt-prilozheniya-flask.md#session\_interface).

**Параметры:**

* **session** - сеанс, который нужно сохранить (объект **SecureCookie**)
* **response** - экземпляр [response\_class](obekt-prilozheniya-flask.md#response\_class)

### secret\_key

Если установлен секретный ключ, криптографические компоненты могут использовать его для подписи файлов **cookie** и других вещей. Установите сложное случайное значение, например, если вы хотите использовать безопасный файл **cookie**.

Этот атрибут также можно настроить из конфигурации с помощью ключа конфигурации [SECRET\_KEY](../rukovodstvo-polzovatelya-flask/obrabotka-konfiguracii-flask.md#secret\_key). По умолчанию `None`.

### select\_jinja\_autoescape( _filename_ )

_Новое в версии 0.5._

Возвращает `True`, если автоматическое экранирование должно быть активным для данного имени шаблона. Если имя шаблона не указано, возвращает `True`.

### send\_file\_max\_age\_default

[timedelta](https://docs.python.org/3/library/datetime.html#datetime.timedelta), который используется как _**cache\_timeout**_ по умолчанию для функций [send\_file ()](poleznye-funkcii-i-klassy-flask.md#flask-send\_file-filename\_or\_fp-mimetype-none-as\_attachment-false-attachment\_filename-none-add\_etags-true-cache\_timeout-none-conditional-false-last\_modified-none). По умолчанию 12 часов.

Этот атрибут также можно настроить из конфигурации с помощью ключа конфигурации **SEND\_FILE\_MAX\_AGE\_DEFAULT**. Эта переменная конфигурации также может быть установлена с целочисленным значением, используемым в качестве секунд. По умолчанию `timedelta (hours = 12)`

### send\_static\_file( _filename_ )

_Новое в версии 0.5._

Функция, используемая внутри для отправки статических файлов из статической папки в браузер.

### session\_cookie\_name

Безопасный файл **cookie** использует это в качестве имени файла **cookie** сеанса.

Этот атрибут также можно настроить из конфигурации с помощью ключа конфигурации **SESSION\_COOKIE\_NAME**. По умолчанию `'session'`

### session\_interface _= \<flask.sessions.SecureCookieSessionInterface object>_

_Новое в версии 0.8._

используемый интерфейс сеанса. По умолчанию здесь используется экземпляр [SecureCookieSessionInterface](sessii-flask.md#klass-flask-session-securecookiesessioninterface).

### shell\_context\_processor( _f_ )

_Новое в версии 0.11._

Регистрирует функцию процессора контекста оболочки.

### shell\_context\_processors _= None_

_Новое в версии 0.11._

Список функций процессора контекста оболочки, которые должны запускаться при создании контекста оболочки.

### should\_ignore\_error( _error_ )

_Новое в версии 0.10._

Это вызывается, чтобы выяснить, следует ли игнорировать ошибку или нет в отношении системы **teardown**. Если эта функция возвращает `True`, обработчики **teardown** не передадут ошибку.

### _свойство_ static\_folder

Абсолютный путь к настроенной статической папке.

### _свойство_ static\_url\_path

Префикс URL-адреса, с которого будет доступен статический маршрут.

Если он не был настроен во время инициализации, он является производным от [static\_folder](obekt-prilozheniya-flask.md#svoistvo-static\_folder).

### teardown\_appcontext( _f_ )

_Новое в версии 0.9._

Регистрирует функцию, которая будет вызываться при завершении контекста приложения. Эти функции обычно также вызываются при открытии контекста запроса.

Пример:

```python
ctx = app.app_context()
ctx.push()
...
ctx.pop()
```

Когда в приведенном выше примере выполняется `ctx.pop ()`, функции **teardown** вызываются непосредственно перед перемещением контекста приложения из стека активных контекстов. Это становится актуальным, если вы используете такие конструкции в тестах.

Поскольку контекст запроса обычно также управляет контекстом приложения, он также будет вызываться, когда вы вставляете контекст запроса.

Когда функция **teardown** была вызвана из-за необработанного исключения, ей будет передан объект ошибки. Если [errorhandler ()](obekt-prilozheniya-flask.md#errorhandler-code\_or\_exception) зарегистрирован, он обработает исключение, а **teardown** не получит его.

Возвращаемые значения функций **teardown** игнорируются.

### teardown\_appcontext\_funcs _= None_

_Новое в версии 0.9._

Список функций, вызываемых при уничтожении контекста приложения. Поскольку контекст приложения также разрушается, если запрос завершается, это место для хранения кода, который отключается от баз данных.

### teardown\_request( _f_ )

Регистрирует функцию, которая будет запускаться в конце каждого запроса, независимо от того, было исключение или нет. Эти функции выполняются при появлении контекста запроса, даже если фактический запрос не был выполнен.

Пример:

```python
ctx = app.test_request_context()
ctx.push()
...
ctx.pop()
```

Когда в приведенном выше примере выполняется `ctx.pop ()`, функции **teardown** вызываются непосредственно перед тем, как контекст запроса перемещается из стека активных контекстов. Это становится актуальным, если вы используете такие конструкции в тестах.

Обычно функции **teardown** должны предпринимать все необходимые шаги, чтобы избежать их сбоя. Если они все же выполняют код, который может дать сбой, им придется окружать выполнение этого кода операторами `try / except` и регистрировать возникающие ошибки.

Когда функция **teardown** была вызвана из-за исключения, ей будет передан объект ошибки.

Возвращаемые значения функций **teardown** игнорируются.

{% hint style="info" %}
**Примечание отладки:**

В режиме отладки **Flask** не будет немедленно отключать запрос об исключении. Вместо этого он будет поддерживать его в рабочем состоянии, чтобы интерактивный отладчик мог получить к нему доступ. Этим поведением можно управлять с помощью переменной конфигурации **PRESERVE\_CONTEXT\_ON\_EXCEPTION**.
{% endhint %}

### teardown\_request\_funcs _= None_

_Новое в версии 0.7._

Словарь со списками функций, которые вызываются после каждого запроса, даже если произошло исключение. Ключ словаря - это имя схемы _**blueprint**_, для которой активна эта функция, `None` для всех запросов. Этим функциям не разрешено изменять запрос, и их возвращаемые значения игнорируются. Если во время обработки запроса возникло исключение, оно передается каждой функции **teardown\_request**. Чтобы зарегистрировать здесь функцию, используйте декоратор [teardown\_request ()](obekt-prilozheniya-flask.md#teardown\_request-f).

### template\_context\_processors _= None_

Словарь со списком функций, которые вызываются без аргументов для заполнения контекста шаблона. Ключ словаря - это имя схемы _**blueprint**_, для которой активна эта функция, `None` для всех запросов. Каждый возвращает словарь, в котором обновляется контекст шаблона. Чтобы зарегистрировать здесь функцию, используйте декоратор [context\_processor ()](obekt-prilozheniya-flask.md#context\_processor-f).

### template\_filter( _name=None_ )

Декоратор, который используется для регистрации настраиваемого фильтра шаблона. Вы можете указать имя для фильтра, иначе будет использовано имя функции. Пример:

```python
@app.template_filter()
def reverse(s):
    return s[::-1]
```

**Параметры**: _**name**_ - необязательное имя фильтра, иначе будет использовано имя функции.

### template\_folder _= None_

Расположение файлов шаблона, которые нужно добавить в поиск по шаблону. `None`, если не нужно добавлять шаблоны.

### template\_global( _name=None_ )

Декоратор, который используется для регистрации глобальной функции настраиваемого шаблона. Вы можете указать имя для глобальной функции, в противном случае будет использовано имя функции. Пример:

```python
@app.template_global()
def double(n):
    return 2 * n
```

**Параметры**: _**name**_ - необязательное имя глобальной функции, иначе будет использовано имя функции.

### template\_test( _name=None_ )

Декоратор, который используется для регистрации пользовательского шаблона _**test**_. Вы можете указать имя для теста, иначе будет использовано имя функции. Пример:

```python
@app.template_test()
def is_prime(n):
    if n == 2:
        return True
    for i in range(2, int(math.ceil(math.sqrt(n))) + 1):
        if n % i == 0:
            return False
    return True
```

**Параметры**: _**name**_ - необязательное имя теста, иначе будет использовано имя функции.

### _свойство_ templates\_auto\_reload

Обновляет шаблоны, когда они будут изменены. Используется функцией [create\_jinja\_environment ()](obekt-prilozheniya-flask.md#create\_jinja\_environment). Этот атрибут можно настроить с помощью [TEMPLATES\_AUTO\_RELOAD](../rukovodstvo-polzovatelya-flask/obrabotka-konfiguracii-flask.md#templates\_auto\_reload). Если не установлен, он будет включен в режиме отладки.

_Новое в версии 1.0:_ это свойство было добавлено, но базовая конфигурация и поведение уже существовали.

### test\_cli\_runner( _\*\*kwargs_ )

_Новое в версии 1.0._

Создает средство выполнения CLI для тестирования команд CLI. См. раздел «[Тестирование команд интерфейса командной строки](../rukovodstvo-polzovatelya-flask/testirovanie-prilozhenii-flask.md#testirovanie-cli-komand)».

Возвращает экземпляр [test\_cli\_runner\_class](obekt-prilozheniya-flask.md#test\_cli\_runner\_class), по умолчанию [FlaskCliRunner](testovye-client-i-cli-runner.md#klass-flask-testing-flaskclirunner). Объект приложения **Flask** передается в качестве первого аргумента.

### test\_cli\_runner\_class _= None_

_Новое в версии 1.0._

Подкласс [CliRunner](https://click.palletsprojects.com/en/7.x/api/#click.testing.CliRunner), по умолчанию [FlaskCliRunner](testovye-client-i-cli-runner.md#klass-flask-testing-flaskclirunner), который используется [**test\_cli\_runner ()**](obekt-prilozheniya-flask.md#test\_cli\_runner-kwargs). Его метод `__init__` должен принимать объект приложения **Flask** в качестве первого аргумента.

### test\_client( _use\_cookies=True_, _\*\*kwargs_ )

Создает тестового клиента для этого приложения. Для получения информации о модульном тестировании перейдите к [Тестирование приложений Flask](../rukovodstvo-polzovatelya-flask/testirovanie-prilozhenii-flask.md).

Обратите внимание: если вы тестируете утверждения или исключения в коде приложения, вы должны установить `app.testing = True`, чтобы исключения распространялись на тестовый клиент. В противном случае исключение будет обрабатываться приложением (не видимым для тестового клиента), и единственным признаком **AssertionError** или другого исключения будет ответ с кодом состояния `500` для тестового клиента. См. атрибут [testing](obekt-prilozheniya-flask.md#testing). Например:

```python
app.testing = True
client = app.test_client()
```

Тестовый клиент может использоваться в блоке **with**, чтобы отложить закрытие контекста до конца блока **with**. Это полезно, если вы хотите получить доступ к локальным контекстам для тестирования:

```python
with app.test_client() as c:
    rv = c.get('/?vodka=42')
    assert request.args['vodka'] == '42'
```

Кроме того, вы можете передать необязательные аргументы ключевого слова, которые затем будут переданы конструктору [test\_client\_class](obekt-prilozheniya-flask.md#test\_client\_class) приложения. Например:

```python
from flask.testing import FlaskClient

class CustomClient(FlaskClient):
    def __init__(self, *args, **kwargs):
        self._authentication = kwargs.pop("authentication")
        super(CustomClient,self).__init__( *args, **kwargs)

app.test_client_class = CustomClient
client = app.test_client(authentication='Basic ....')
```

См. [FlaskClient](testovye-client-i-cli-runner.md#klass-flask-testing-flaskclient) для получения дополнительной информации.

_Изменено в версии 0.11:_ Добавлен _**\*\*kwargs**_ для поддержки передачи дополнительных аргументов ключевого слова в конструктор [test\_client\_class](obekt-prilozheniya-flask.md#test\_client\_class).

_Новое в версии 0.7:_ был добавлен параметр _**use\_cookies**_, а также возможность переопределить используемый клиент, установив атрибут [test\_client\_class](obekt-prilozheniya-flask.md#test\_client\_class).

_Изменено в версии 0.4:_ добавлена поддержка использования с **with** блокировкой для клиента.

### test\_client\_class _= None_

_Новое в версии 0.7._

тестовый клиент, который используется, когда используется _**test\_client**_.

### test\_request\_context( _\*args_, _\*\*kwargs_ )

Создает [RequestContext](poleznye-funkcii-flask.md#klass-flask-ctx-requestcontext) для среды **WSGI**, созданной из заданных значений. Это в основном полезно во время тестирования, когда вы можете запустить функцию, которая использует данные запроса, не отправляя полный запрос.

См. [Контекст запроса](../rukovodstvo-polzovatelya-flask/kontekst-zaprosa-flask.md).

Используйте блок **with**, чтобы протолкнуть контекст, который сделает точку [request](dannye-vkhodyashego-zaprosa-request.md#flask-request) на запрос для созданной среды.

```python
with test_request_context(...):
    generate_report()
```

При использовании оболочки может быть проще вручную нажимать и извлекать контекст, чтобы избежать отступов.

```python
ctx = app.test_request_context(...)
ctx.push()
...
ctx.pop()
```

Принимает те же аргументы, что и [EnvironBuilder](https://werkzeug.palletsprojects.com/en/1.0.x/test/#werkzeug.test.EnvironBuilder) **Werkzeug**, с некоторыми значениями по умолчанию из приложения. См. Связанные документы **Werkzeug** для большей части доступных аргументов. Здесь указано поведение, специфичное для **Flask**.

**Параметры:**

* **path** - Запрашиваемый URL-путь.
* **base\_url** - Базовый URL-адрес, по которому приложение обслуживается, относительно _**path**_. Если не указан, создается на основе [PREFERRED\_URL\_SCHEME](../rukovodstvo-polzovatelya-flask/obrabotka-konfiguracii-flask.md#preferred\_url\_scheme), _**subdomain**_, [SERVER\_NAME](../rukovodstvo-polzovatelya-flask/obrabotka-konfiguracii-flask.md#server\_name) и [APPLICATION\_ROOT](../rukovodstvo-polzovatelya-flask/obrabotka-konfiguracii-flask.md#application\_root).
* **subdomain** - Имя поддомена, которое нужно добавить к [SERVER\_NAME](../rukovodstvo-polzovatelya-flask/obrabotka-konfiguracii-flask.md#server\_name).
* **url\_scheme** - Схема для использования вместо [PREFERRED\_URL\_SCHEME](../rukovodstvo-polzovatelya-flask/obrabotka-konfiguracii-flask.md#preferred\_url\_scheme).
* **data** - Тело запроса в виде строки или словаря ключей и значений формы.
* **json** - Если задан, он сериализуется как **JSON** и передается как **data**. Также по умолчанию **content\_type** равно `application/json`.
* **args** - другие позиционные аргументы, переданные в [EnvironBuilder](https://werkzeug.palletsprojects.com/en/1.0.x/test/#werkzeug.test.EnvironBuilder).
* **kwargs** - другие аргументы ключевого слова, переданные в [EnvironBuilder](https://werkzeug.palletsprojects.com/en/1.0.x/test/#werkzeug.test.EnvironBuilder).

### testing

Флаг тестирования. Установите для него значение `True`, чтобы включить тестовый режим расширений **Flask** (а в будущем, вероятно, и самого **Flask**). Например, это может активировать помощников по тестированию, которые требуют дополнительных затрат времени выполнения, которые не должны быть включены по умолчанию.

Если это включено, и **PROPAGATE\_EXCEPTIONS** не изменится по сравнению со значением по умолчанию, он неявно включен.

Этот атрибут также можно настроить из конфигурации с помощью ключа конфигурации **TESTING**. По умолчанию `False`.

### trap\_http\_exception( _e_ )

_Новое в версии 0.8._

Проверяет, следует ли перехватить исключение HTTP. По умолчанию это вернет `False` для всех исключений, кроме ошибки неверного ключа запроса, если **TRAP\_BAD\_REQUEST\_ERRORS** установлен в `True`. Он также возвращает `True`, если **TRAP\_HTTP\_EXCEPTIONS** установлен в `True`.

_Изменено в версии 1.0:_ Ошибки неверного запроса по умолчанию не перехватываются в режиме отладки.

### update\_template\_context( _context_ )

Обновляет контекст шаблона, добавив некоторые часто используемые переменные. Это вводит **request**, **session**, **config** и **g** в контекст шаблона, а также все, что процессоры контекста шаблона хотят внедрить. Обратите внимание, что, начиная с `Flask 0.6`, исходные значения в контексте не будут переопределены, если обработчик контекста решит вернуть значение с тем же ключом.

**Параметры**: _**context**_ - контекст как словарь, который обновляется для добавления дополнительных переменных.

### url\_build\_error\_handlers _= None_

_Новое в версии 0.9._

Список функций, которые вызываются, когда [url\_for ()](poleznye-funkcii-i-klassy-flask.md#flask-url\_for-endpoint-values) вызывает ошибку **BuildError**. Каждая зарегистрированная здесь функция вызывается с _**error**_, _**endpoint**_ и _**values**_. Если функция возвращает `None` или вызывает ошибку **BuildError**, выполняется попытка следующей функции.

### url\_default\_functions _= None_

_Новое в версии 0.7._

Словарь со списками функций, которые можно использовать в качестве препроцессоров значений URL. Ключ `None` здесь используется для обратных вызовов всего приложения, в противном случае ключ - это имя схемы _**blueprint**_. Каждая из этих функций имеет возможность изменить словарь значений URL-адресов, прежде чем они будут использоваться в качестве аргументов ключевого слова функции просмотра. Для каждой зарегистрированной функции необходимо также предоставить функцию [url\_defaults ()](obekt-prilozheniya-flask.md#url\_defaults), которая автоматически снова добавляет параметры, которые были удалены таким образом.

### url\_defaults( _f_ )

Функция обратного вызова для значений URL по умолчанию для всех функций просмотра приложения. Он вызывается с конечной точкой и значениями и должен обновлять переданные значения.

### url\_map _= None_

[Map](https://werkzeug.palletsprojects.com/en/1.0.x/routing/#werkzeug.routing.Map) для этого экземпляра. Вы можете использовать это для изменения конвертеров маршрутизации после создания класса, но до подключения каких-либо маршрутов. Пример:

```python
from werkzeug.routing import BaseConverter

class ListConverter(BaseConverter):
    def to_python(self, value):
        return value.split(',')
    def to_url(self, values):
        return ','.join(super(ListConverter, self).to_url(value)
                        for value in values)

app = Flask(__name__)
app.url_map.converters['list'] = ListConverter
```

### url\_map\_class

псевдоним [werkzeug.routing.Map](https://werkzeug.palletsprojects.com/en/1.0.x/routing/#werkzeug.routing.Map)

### url\_rule\_class

псевдоним [werkzeug.routing.Rule](https://werkzeug.palletsprojects.com/en/1.0.x/routing/#werkzeug.routing.Rule)

### url\_value\_preprocessor( _f_ )

Регистрирует функцию препроцессора значения URL для всех функций просмотра в приложении. Эти функции будут вызываться перед функциями [before\_request ()](obekt-prilozheniya-flask.md#before\_request-f).

Функция может изменять значения, полученные из сопоставленного URL-адреса, прежде чем они будут переданы в представление. Например, это можно использовать для извлечения значения кода общего языка и помещения его в **g**, а не для передачи его в каждое представление.

Функции передается имя конечной точки и словарь значений. Возвращаемое значение игнорируется.

### url\_value\_preprocessors

_Новое в версии 0.7._

Словарь со списками функций, которые вызываются перед функциями [before\_request\_funcs](obekt-prilozheniya-flask.md#before\_request\_funcs-none). Ключ словаря - это имя схемы _**blueprint**_, для которой активна эта функция, или `None` для всех запросов. Чтобы зарегистрировать функцию, используйте [url\_value\_preprocessor ()](obekt-prilozheniya-flask.md#url\_value\_preprocessor-f).

### use\_x\_sendfile

_Новое в версии 0.2._

Включите это, если хотите использовать функцию **X-Sendfile**. Имейте в виду, что сервер должен это поддерживать. Это влияет только на файлы, отправленные с помощью метода [send\_file ()](poleznye-funkcii-i-klassy-flask.md#flask-send\_file-filename\_or\_fp-mimetype-none-as\_attachment-false-attachment\_filename-none-add\_etags-true-cache\_timeout-none-conditional-false-last\_modified-none).

Этот атрибут также можно настроить из конфигурации с помощью ключа конфигурации **USE\_X\_SENDFILE**. По умолчанию `False`.

### view\_functions _= None_

Словарь всех зарегистрированных функций просмотра. Ключи будут именами функций, которые также используются для генерации URL-адресов, а значения - самими объектами функций. Чтобы зарегистрировать функцию просмотра, используйте декоратор [route ()](obekt-prilozheniya-flask.md#route-rule-options).

### wsgi\_app( _environ_, _start\_response_ )

Фактическое приложение **WSGI**. Это не реализовано в **\_\_call\_\_()**, поэтому промежуточное ПО можно применять без потери ссылки на объект приложения. Вместо этого:

```python
app = MyMiddleware(app)
```

Вместо этого лучше сделать следующее:

```python
app.wsgi_app = MyMiddleware(app.wsgi_app)
```

Тогда у вас все еще есть исходный объект приложения, и вы можете продолжать вызывать его методы.

_Изменено в версии 0.7:_ события **teardown** для контекстов запроса и приложения вызываются даже при возникновении необработанной ошибки. Другие события могут не вызываться в зависимости от того, когда во время отправки возникает ошибка. См. [Обратные вызовы и ошибки](../rukovodstvo-polzovatelya-flask/kontekst-zaprosa-flask.md#obratnye-vyzovy-i-oshibki).

**Параметры:**

* **environ** - Среда **WSGI**.
* **start\_response** - Вызываемый объект, принимающий код состояния, список заголовков и необязательный контекст исключения для запуска ответа.
