# Полезные функции и классы Flask

### flask.current\_app

Прокси-сервер приложения, обрабатывающего текущий запрос. Это полезно для доступа к приложению без необходимости его импорта или если его нельзя импортировать, например, при использовании шаблона фабрики приложений или в схемах элементов и расширениях.

Это доступно только при размещении [контекста приложения](../rukovodstvo-polzovatelya-flask/kontekst-prilozheniya-flask.md). Это происходит автоматически во время запросов и команд CLI. Им можно управлять вручную с помощью [app\_context ()](obekt-prilozheniya-flask.md#app\_context).

Это прокси. См. [Примечания к прокси](../rukovodstvo-polzovatelya-flask/kontekst-zaprosa-flask.md#primechaniya-k-proksi) для получения дополнительной информации.

### flask.has\_request\_context()

_Новое в версии 0.7_.

Если у вас есть код, который хочет проверить, есть ли контекст запроса или нет, эту функцию можно использовать. Например, вы можете захотеть воспользоваться информацией о запросе, если объект запроса доступен, но потерпеть неудачу, если он недоступен.

```python
class User(db.Model):

    def __init__(self, username, remote_addr=None):
        self.username = username
        if remote_addr is None and has_request_context():
            remote_addr = request.remote_addr
        self.remote_addr = remote_addr
```

В качестве альтернативы вы также можете просто проверить любой из связанных с контекстом объектов (например, [request](dannye-vkhodyashego-zaprosa-request.md#flask-request) или [g](globalnyi-obekt-prilozheniya-flask.md#flask-g)) на истинность:

```python
class User(db.Model):

    def __init__(self, username, remote_addr=None):
        self.username = username
        if remote_addr is None and request:
            remote_addr = request.remote_addr
        self.remote_addr = remote_addr
```

### flask.copy\_current\_request\_context( _f_ )

_Новое в версии 0.10_.

Вспомогательная функция, которая украшает функцию для сохранения текущего контекста запроса. Это полезно при работе с гринлетами. В момент оформления функции создается копия контекста запроса, которая затем передается при вызове функции. Текущий сеанс также включается в контекст скопированного запроса.

Пример:

```python
import gevent
from flask import copy_current_request_context

@app.route('/')
def index():
    @copy_current_request_context
    def do_some_work():
        # поработайте здесь, он может получить доступ к flask.request
        # или flask.session, как в противном случае в функции просмотра.
        ...
    gevent.spawn(do_some_work)
    return 'Regular response'
```

### flask.has\_app\_context()

_Новое в версии 0.9_.

Работает как [has\_request\_context ()](poleznye-funkcii-i-klassy-flask.md#flask-has\_request\_context), но для контекста приложения. Вы также можете просто выполнить логическую проверку объекта [current\_app](poleznye-funkcii-i-klassy-flask.md#flask-current\_app).

### flask.url\_for( _endpoint_, _\*\*values_ )

Создает URL-адрес указанной конечной точки с помощью предоставленного метода.

Переменные аргументы, которые неизвестны целевой конечной точке, добавляются к сгенерированному URL-адресу в качестве аргументов запроса. Если значение аргумента запроса - `None`, вся пара пропускается. В случае, если _**blueprints**_ активны, вы можете сокращать ссылки на один и тот же _**blueprint**_, добавив к локальной конечной точке префикс точки (`.`).

Это будет ссылаться на функцию _**index**_, локальную для текущего проекта:

```python
url_for('.index')
```

Для получения дополнительной информации перейдите к [быстрому старту](../rukovodstvo-polzovatelya-flask/bystryi-start-flask.md#sozdanie-url).

Значения конфигурации **APPLICATION\_ROOT** и **SERVER\_NAME** используются только при генерации URL-адресов вне контекста запроса.

Для интеграции приложений во [Flask](obekt-prilozheniya-flask.md#klass-flask-flask) есть перехватчик ошибок построения URL через [Flask.url\_build\_error\_handlers](obekt-prilozheniya-flask.md#url\_build\_error\_handlers). Функция **url\_for** приводит к ошибке **BuildError**, когда текущее приложение не имеет URL-адреса для данной конечной точки и значений. Когда это происходит, [current\_app](poleznye-funkcii-i-klassy-flask.md#flask-current\_app) вызывает свои **url\_build\_error\_handlers**, если он не равен `None`, который может вернуть строку для использования в качестве результата **url\_for** (вместо **url\_for** по умолчанию, чтобы вызвать исключение **BuildError**) или повторно вызвать исключение. Пример:

```python
def external_url_handler(error, endpoint, values):
    "Ищет внешний URL-адрес, когда url_for не может создать URL-адрес."
    # Это пример подключения build_error_handler.
    # Здесь lookup_url - это некоторая служебная функция,
    # которую вы создали, которая ищет конечную точку во внешнем реестре URL.
    url = lookup_url(endpoint, **values)
    if url is None:
        # Внешний поиск не имеет URL-адреса.
        # Повторно вызовите BuildError в контексте исходной трассировки.
        exc_type, exc_value, tb = sys.exc_info()
        if exc_value is error:
            raise exc_type, exc_value, tb
        else:
            raise error
    # url_for будет использовать этот результат вместо того, чтобы вызывать BuildError.
    return url

app.url_build_error_handlers.append(external_url_handler)
```

Здесь _**error**_ - это экземпляр **BuildError**, а _**endpoint**_ и _**values**_ - это аргументы, переданные в **url\_for**. Обратите внимание, что это предназначено для создания URL-адресов вне текущего приложения, а не для обработки ошибок **404 NotFound**.

#### Журнал изменений url\_for()

_Новое в версии 0.10:_ добавлен параметр _**\_scheme**_.

_Новое в версии 0.9:_ добавлены параметры _**\_anchor**_ и _**\_method**_.

_Новое в версии 0.9:_ вызывает **Flask.handle\_build\_error ()** в **BuildError**.

#### Параметры url\_for()

* **endpoint** - конечная точка URL (имя функции)
* **values** - переменные аргументы правила URL
* **\_external** - если установлено значение `True`, создается абсолютный URL. Адрес сервера можно изменить с помощью переменной конфигурации **SERVER\_NAME**, которая возвращается к заголовку Host, а затем к IP и порту запроса.
* **\_scheme** - строка, определяющая желаемую схему URL. Параметр _**\_external**_ должен иметь значение `True`, в противном случае возникает ошибка [ValueError](https://docs.python.org/3/library/exceptions.html#ValueError). Поведение по умолчанию использует ту же схему, что и текущий запрос, или **PREFERRED\_URL\_SCHEME** из [конфигурации приложения](../rukovodstvo-polzovatelya-flask/obrabotka-konfiguracii-flask.md), если контекст запроса недоступен. Начиная с `Werkzeug 0.10`, здесь также можно установить пустую строку для построения URL-адресов, зависящих от протокола.
* **\_anchor** - если предусмотрено, это добавляется как привязка к URL-адресу. Означает якорную ссылку на странице (через `#`).
* **\_method** - если предусмотрено, это явно указывает метод HTTP.

### flask.abort( _status: Union\[ int, Response ], \*args, \*\*kwargs_ ) → NoReturn

Вызывает исключение **HTTPException** для данного кода состояния или приложения **WSGI**.

Если указан код состояния, он будет найден в списке исключений и вызовет это исключение. Если передано приложение **WSGI**, оно обернет его в исключение прокси-сервера **WSGI** и вызовет его:

```python
abort(404)  # 404 Not Found
abort(Response('Hello World'))
```

### flask.redirect( _location: str_, _code: int = 302_, _Response: Optional\[ Type\[ Response ]] = None_ ) → Response

Возвращает объект ответа (приложение **WSGI**), который при вызове перенаправляет клиента в целевое расположение. Поддерживаемые коды: 301, 302, 303, 305, 307 и 308. 300 не поддерживается, потому что это не настоящее перенаправление, и 304, потому что это ответ на запрос с запросом с определенными заголовками **If-Modified-Since**.

#### Журнал изменений redirect()

_Новое в версии 0.10:_ теперь можно передать класс, используемый для объекта **Response**.

_Новое в версии 0.6:_ теперь местоположение может быть строкой в Юникоде, закодированной с помощью функции **iri\_to\_uri ()**.

#### Параметры redirect()

* **location** - место, куда должен быть перенаправлен ответ.
* **code** - код статуса перенаправления. по умолчанию 302.
* **Response (**_**class**_**)** - класс **Response**, используемый при создании экземпляра ответа. По умолчанию используется [werkzeug.wrappers.Response](https://werkzeug.palletsprojects.com/en/1.0.x/wrappers/#werkzeug.wrappers.Response), если не указано иное.

### flask.make\_response( _\*args_ )

_Новое в версии 0.6_.

Иногда необходимо установить дополнительные заголовки в представлении. Поскольку представления не должны возвращать объекты ответа, но могут возвращать значение, преобразованное в объект ответа самим **Flask**, становится сложно добавить к нему заголовки. Эту функцию можно вызвать вместо использования возврата, и вы получите объект ответа, который можно использовать для прикрепления заголовков.

Если представление выглядело так, и вы хотите добавить новый заголовок:

```python
def index():
    return render_template('index.html', foo=42)
```

Теперь вы можете сделать что-то вроде этого:

```python
def index():
    response = make_response(render_template('index.html', foo=42))
    response.headers['X-Parachutes'] = 'parachutes are cool'
    return response
```

Эта функция принимает те же аргументы, которые вы можете вернуть из функции просмотра. Это, например, создает ответ с кодом ошибки 404:

```python
response = make_response(render_template('not_found.html'), 404)
```

Другой вариант использования этой функции - заставить возвращаемое значение функции представления в ответ, что полезно с декораторами представления:

```python
response = make_response(view_function())
response.headers['X-Parachutes'] = 'parachutes are cool'
```

Внутренне эта функция выполняет следующие функции:

* если аргументы не переданы, создается новый аргумент ответа
* если передан один аргумент, с ним вызывается **flask.Flask.make\_response ()**.
* если передано более одного аргумента, аргументы передаются в функцию **flask.Flask.make\_response ()** как кортеж.

### flask.after\_this\_request( _f_ )

_Новое в версии 0.9_.

Выполняет функцию после этого запроса. Это полезно для изменения объектов ответа. Функция передает объект ответа и должна вернуть тот же или новый.

Пример:

```python
@app.route('/')
def index():
    @after_this_request
    def add_header(response):
        response.headers['X-Foo'] = 'Parachute'
        return response
    return 'Hello World!'
```

Это более полезно, если функция, отличная от функции просмотра, хочет изменить ответ. Например, подумайте о декораторе, который хочет добавить несколько заголовков без преобразования возвращаемого значения в объект ответа.

### flask.send\_file( _filename\_or\_fp_, _mimetype=None_, _as\_attachment=False_, _attachment\_filename=None_, _add\_etags=True_, _cache\_timeout=None_, _conditional=False_, _last\_modified=None_ )

_Новое в версии 0.2_.

Отправляет содержимое файла клиенту. Это будет использовать наиболее эффективный из доступных и настроенных методов. По умолчанию он пытается использовать поддержку **file\_wrapper** сервера **WSGI**. В качестве альтернативы вы можете установить для атрибута [use\_x\_sendfile](obekt-prilozheniya-flask.md#use\_x\_sendfile) приложения значение `True`, чтобы напрямую генерировать заголовок **X-Sendfile**. Однако для этого требуется поддержка базового веб-сервера для **X-Sendfile**.

По умолчанию он попытается угадать **mimetype** за вас, но вы также можете указать его явно. Для дополнительной безопасности вы, вероятно, захотите отправить определенные файлы в виде вложения (например, HTML). Для угадывания **mimetype** требуется указать _**filename**_ или _**attachment\_filename**_.

**ETags** также будут прикреплены автоматически, если указано _**filename**_. Вы можете отключить это, установив `add_etags = False`.

Если `conditional = True` и указано _**filename**_, этот метод попытается обновить поток ответов для поддержки запросов диапазона. Это позволит ответить на запрос частичным ответом содержания.

Никогда не передавайте в эту функцию имена файлов из пользовательских источников; вместо этого вам следует использовать [send\_from\_directory ()](poleznye-funkcii-i-klassy-flask.md#flask-send\_from\_directory).

#### Журнал изменений send\_file()

_Изменено в версии 1.1:_ Имя файла может быть объектом [PathLike](https://docs.python.org/3/library/os.html#os.PathLike).

_Новое в версии 1.1:_ Частичное содержимое поддерживает [BytesIO](https://docs.python.org/3/library/io.html#io.BytesIO).

_Изменено в версии 1.0.3:_ имена файлов кодируются **ASCII** вместо **Latin-1** для более широкой совместимости с серверами **WSGI**.

_Изменено в версии 1.0:_ поддерживаются имена файлов **UTF-8**, как указано в [RFC 2231](https://tools.ietf.org/html/rfc2231#section-4).

_Изменено в версии 0.12:_ имя файла больше не определяется автоматически из файловых объектов. Если вы хотите использовать автоматическую поддержку **mimetype** и **etag**, передайте путь к файлу через _**filename\_or\_fp**_ или _**attachment\_filename**_.

_Изменено в версии 0.12:_ имя _**attachment\_filename**_ предпочтительнее, чем _**filename**_ для определения типа MIME.

_Изменено в версии 0.9:_ _**cache\_timeout**_ получает значение по умолчанию из конфигурации приложения, когда `None`.

_Изменено в версии 0.7:_ угадывание **mimetype** и поддержка **etag** для файловых объектов устарели из-за ненадежности. Передайте имя файла, если можете, в противном случае прикрепите **etag** самостоятельно. Эта функция будет удалена в `Flask 1.0`.

_Новое в версии 0.5:_ добавлены параметры _**add\_etags**_, _**cache\_timeout**_ и _**conditional**_. По умолчанию теперь прикрепляются etags.

#### Параметры send\_file()

* **filename\_or\_fp** - имя файла для отправки. Это относительно [root\_path](obekt-prilozheniya-flask.md#root\_path), если указан относительный путь. В качестве альтернативы может быть предоставлен файловый объект, и в этом случае **X-Sendfile** может не работать и вернуться к традиционному методу. Перед вызовом **send\_file ()** убедитесь, что указатель файла находится в начале данных для отправки.
* **mimetype** - _**mimetype**_ файла, если он указан. Если указан путь к файлу, автоматическое обнаружение происходит как резерв, в противном случае возникает ошибка.
* **as\_attachment** - установите значение `True`, если вы хотите отправить этот файл с заголовком `Content-Disposition: attachment`.
* **attachment\_filename** - имя файла для вложения, если оно отличается от имени файла.
* **add\_etags** - установите значение `False`, чтобы отключить прикрепление тегов.
* **conditional** - установите значение `True`, чтобы включить условные ответы.
* **cache\_timeout** - тайм-аут в секундах для заголовков. Когда `None` (по умолчанию), это значение устанавливается [get\_send\_file\_max\_age ()](obekt-prilozheniya-flask.md#get\_send\_file\_max\_age) из [current\_app](poleznye-funkcii-i-klassy-flask.md#flask-current\_app).
* **last\_modified** - установите для заголовка **Last-Modified** это значение, [datetime](https://docs.python.org/3/library/datetime.html#datetime.datetime) или отметку времени. Если файл был передан, это отменяет его **mtime**.

### flask.send\_from\_directory( _directory_, _filename_, _\*\*options_ )

_Новое в версии 0.5_.

Отправляет файл из заданного каталога с помощью [send\_file ()](poleznye-funkcii-i-klassy-flask.md#flask-send\_file-filename\_or\_fp-mimetype-none-as\_attachment-false-attachment\_filename-none-add\_etags-true-cache\_timeout-none-conditional-false-last\_modified-none). Это безопасный способ быстрого доступа к статическим файлам из папки загрузки или чего-то подобного.

Пример использования:

```python
@app.route('/uploads/<path:filename>')
def download_file(filename):
    return send_from_directory(app.config['UPLOAD_FOLDER'],
                               filename, as_attachment=True)
```

{% hint style="info" %}
**Отправка файлов и производительность:**

Настоятельно рекомендуется активировать либо поддержку **X-Sendfile** на вашем веб-сервере, либо (если аутентификация не происходит) указать веб-серверу, что он должен обслуживать файлы по заданному пути самостоятельно, без вызова веб-приложения для повышения производительности.
{% endhint %}

#### Параметры send\_from\_directory()

* **directory** - каталог, в котором хранятся все файлы.
* **filename** - имя файла относительно этого каталога для загрузки.
* **options** - необязательные аргументы ключевого слова, которые напрямую перенаправляются в [send\_file ()](poleznye-funkcii-i-klassy-flask.md#flask-send\_file-filename\_or\_fp-mimetype-none-as\_attachment-false-attachment\_filename-none-add\_etags-true-cache\_timeout-none-conditional-false-last\_modified-none).

### flask.safe\_join( _directory_, _\*pathnames_ )

Безопасно объединение _**directory**_ и ноль или более ненадежных _**pathnames**_.

Пример использования:

```python
@app.route('/wiki/<path:filename>')
def wiki_page(filename):
    filename = safe_join(app.config['WIKI_FOLDER'], filename)
    with open(filename, 'rb') as fd:
        content = fd.read()  # Прочитать и обработать содержимое файла...
```

#### Параметры safe\_join()

* **directory** - доверенный базовый каталог.
* **pathnames** - ненадежные пути относительно этого каталога.

#### Исключения safe\_join()

* возбуждает [NotFound](https://werkzeug.palletsprojects.com/en/1.0.x/exceptions/#werkzeug.exceptions.NotFound), если один или несколько пройденных путей выпадают за его границы.

### flask.escape( _s_ ) → markup

Преобразует символы `&`, `<`, `>`, `'` и `"` в строке _**s**_ в безопасные для HTML последовательности. Используйте это, если вам нужно отобразить текст, который может содержать такие символы в HTML. Помечает возвращаемое значение как строку разметки.

## Класс flask.Markup

Строка, готовая к безопасной вставке в документ HTML или XML, либо потому, что она была экранирована, либо потому, что она была помечена как безопасная.

Передача объекта конструктору преобразует его в текст и обертывает, чтобы пометить его как безопасный без экранирования. Чтобы экранировать текст, используйте вместо этого метод класса [escape ()](poleznye-funkcii-i-klassy-flask.md#flask-escape-s-markup).

```python
>>> Markup('Hello, <em>World</em>!')
Markup('Hello, <em>World</em>!')
>>> Markup(42)
Markup('42')
>>> Markup.escape('Hello, <em>World</em>!')
Markup('Hello &lt;em&gt;World&lt;/em&gt;!')
```

Это реализует интерфейс `__html__()`, который используют некоторые фреймворки. Передача объекта, реализующего `__html__()`, обернет вывод этого метода, пометив его как безопасный.

```python
>>> class Foo:
...     def __html__(self):
...         return '<a href="/foo">foo</a>'
...
>>> Markup(Foo())
Markup('<a href="/foo">foo</a>')
```

Это подкласс текстового типа (**str** в Python 3, **unicode** в Python 2). Он имеет те же методы, что и этот тип, но все методы избегают своих аргументов и возвращают экземпляр **Markup**.

```python
>>> Markup('<em>%s</em>') % 'foo & bar'
Markup('<em>foo &amp; bar</em>')
>>> Markup('<em>Hello</em> ') + '<foo>'
Markup('<em>Hello</em> &lt;foo&gt;')
```

### _classmethod_ escape( _s_ )

Экранировать строку. Вызывает [escape ()](poleznye-funkcii-i-klassy-flask.md#flask-escape-s-markup) и гарантирует, что для подклассов будет возвращен правильный тип.

### striptags()

Удаляет [unescape ()](poleznye-funkcii-i-klassy-flask.md#unescape) разметку, удаление тегов и нормализацию пробелов до одиночных пробелов.

```python
>>> Markup('Main &raquo;        <em>About</em>').striptags()
'Main » About'
```

### unescape()

Преобразует экранированную разметку обратно в текстовую строку. При этом объекты HTML заменяются символами, которые они представляют.

```python
>>> Markup('Main &raquo; <em>About</em>').unescape()
'Main » <em>About</em>'
```
