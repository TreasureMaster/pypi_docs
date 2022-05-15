# Данные входящего запроса Request

## Класс flask.Request( _environ: WSGIEnvironment_, _populate\_request: bool = True_, _shallow: bool = False_)

Объект запроса, используемый по умолчанию в **Flask**. Запоминает совпадающую конечную точку и аргументы просмотра.

Это то, что заканчивается [request](dannye-vkhodyashego-zaprosa-request.md#flask-request). Если вы хотите заменить используемый объект запроса, вы можете создать его подкласс и установить [request\_class](obekt-prilozheniya-flask.md#request\_class) для своего подкласса.

Объект запроса является подклассом [Request](https://werkzeug.palletsprojects.com/en/1.0.x/wrappers/#werkzeug.wrappers.Request) из **Werkzeug** и предоставляет все атрибуты, определенные **Werkzeug**, плюс несколько специфичных для **Flask**.

### environ

Базовая среда **WSGI**.

### path, full\_path, script\_root, url, base\_url, url\_root

Предоставляет различные способы взглянуть на текущий [RFC 3987](https://tools.ietf.org/html/rfc3987.html). Представьте, что ваше приложение прослушивает следующий корень приложения:

```bash
http://www.example.com/myapplication
```

И пользователь запрашивает следующий URI:

```bash
http://www.example.com/myapplication/%CF%80/page.html?x=y
```

В этом случае значения вышеупомянутых атрибутов будут следующими:

| Атрибут            | Значение                                                   |
| ------------------ | ---------------------------------------------------------- |
| _**path**_         |  `u'/π/page.html'`                                         |
| _**full\_path**_   |  `u'/π/page.html?x=y'`                                     |
| _**script\_root**_ |  `u'/myapplication'`                                       |
| _**base\_url**_    |  `u'http://www.example.com/myapplication/π/page.html'`     |
| _**url**_          |  `u'http://www.example.com/myapplication/π/page.html?x=y'` |
| _**url\_root**_    |  `u'http://www.example.com/myapplication/'`                |

### (_property_) accept\_charsets

Список кодировок, поддерживаемых этим клиентом как объект **Werkzeug** [CharsetAccept](https://werkzeug.palletsprojects.com/en/1.0.x/datastructures/#werkzeug.datastructures.CharsetAccept).

### (_property_) accept\_encodings

Список кодировок, которые принимает этот клиент. Кодировки в термине HTTP - это кодировки сжатия, такие как gzip. Для кодировки посмотрите **accept\_charset**.

### (_property_) accept\_languages

Список языков, которые этот клиент принимает в качестве объекта **Werkzeug** [LanguageAccept](https://werkzeug.palletsprojects.com/en/1.0.x/datastructures/#werkzeug.datastructures.LanguageAccept).

### (_property_) accept\_mimetypes

Список _**mimetypes**_, которые этот клиент поддерживает как объект **Werkzeug** [MIMEAccept](https://werkzeug.palletsprojects.com/en/1.0.x/datastructures/#werkzeug.datastructures.MIMEAccept).

### access\_control\_request\_headers

Отправляется с запросом предварительной проверки, чтобы указать, какие заголовки будут отправлены с запросом перекрестного происхождения. Установите **access\_control\_allow\_headers** в _**response**_, чтобы указать, какие заголовки разрешены.

### access\_control\_request\_method

Отправляется с запросом предварительной проверки, чтобы указать, какой метод будет использоваться для запроса перекрестного происхождения. Установите **access\_control\_allow\_methods** в _**response**_, чтобы указать, какие методы разрешены.

### (_property_) access\_route

Если существует перенаправленный заголовок, это список всех IP-адресов от клиентского IP до последнего прокси-сервера.

### (_classmethod_) application( _f: Callable\[\[Request], WSGIApplication]_ ) → WSGIApplication

Декорирует функцию как респондент, который принимает запрос в качестве последнего аргумента. Это работает как декоратор **responseder ()**, но функции передается объект запроса в качестве последнего аргумента, и объект запроса будет автоматически закрыт:

```python
@Request.application
def my_wsgi_app(request):
    return Response('Hello World!')
```

Начиная с версии `Werkzeug 0.14`, исключения HTTP автоматически перехватываются и преобразуются в ответы вместо сбоя.

**Параметры**: _**f**_ - вызываемый **WSGI** для декоратора.

**Возвращает**: новый вызываемый **WSGI**.

### (_property_) args

Анализируемые параметры URL-адреса (часть в URL-адресе после вопросительного знака).

По умолчанию эта функция возвращает из **Werkzeug** [ImmutableMultiDict](https://werkzeug.palletsprojects.com/en/1.0.x/datastructures/#werkzeug.datastructures.ImmutableMultiDict). Это можно изменить, задав для параметра [parameter\_storage\_class](dannye-vkhodyashego-zaprosa-request.md#parameter\_storage\_class) другой тип. Это может быть необходимо, если важен порядок данных в форме.

### (_property_) authorization

Объект **Authorization** в распарсенном виде.

### (_property_) base\_url

Как [url](dannye-vkhodyashego-zaprosa-request.md#path-full\_path-script\_root-url-base\_url-url\_root), но без строки запроса См. также: _**trusted\_hosts**_.

### (_property_) blueprint

Название текущей схемы _**blueprint**_

### (_property_) cache\_control

Объект **Werkzeug** [RequestCacheControl](https://werkzeug.palletsprojects.com/en/1.0.x/datastructures/#werkzeug.datastructures.RequestCacheControl) для заголовков элементов управления входящим кешем.

### close() → None

_Новое в версии 0.9_.

Закрывает связанные ресурсы этого объекта запроса. Это явно закрывает все дескрипторы файлов. Вы также можете использовать объект запроса в операторе **with**, который автоматически закроет его.

### content\_encoding

_Новое в версии 0.9_.

Поле заголовка объекта **Content-Encoding** используется как модификатор медиа-типа. Когда он присутствует, его значение указывает, какие дополнительные кодировки контента были применены к телу объекта, и, следовательно, какие механизмы декодирования должны быть применены для получения медиа-типа, на который ссылается поле заголовка **Content-Type**.

### (_property_) content\_length

Поле заголовка объекта **Content-Length** указывает размер тела объекта в байтах или, в случае метода **HEAD**, размер тела объекта, которое было бы отправлено, если бы запрос был **GET**.

### content\_md5

_Новое в версии 0.9_.

Поле заголовка объекта **Content-MD5**, как определено в **RFC 1864**, представляет собой дайджест **MD5** тела объекта с целью обеспечения сквозной проверки целостности сообщения (**MIC**) тела объекта. (Примечание: **MIC** хорош для обнаружения случайной модификации тела объекта при передаче, но не является защитой от злонамеренных атак.)

### content\_type

Поле заголовка объекта **Content-Type** указывает тип носителя тела объекта, отправленного получателю, или, в случае метода **HEAD**, тип носителя, который был бы отправлен, если бы запрос был **GET**.

### (_property_) cookies

Словарь [dict](https://docs.python.org/3/library/stdtypes.html#dict) с содержимым всех файлов **cookie**, переданных с запросом.

### (_property_) data

Содержит данные входящего запроса в виде строки на случай, если он пришел с _**mimetype**_, который **Werkzeug** не обрабатывает.

### date

Поле общего заголовка **Date** представляет дату и время, когда было отправлено сообщение, и имеет ту же семантику, что и исходная дата в **RFC 822**.

### dict\_storage\_class

псевдоним [werkzeug.datastructures.ImmutableMultiDict](https://werkzeug.palletsprojects.com/en/1.0.x/datastructures/#werkzeug.datastructures.ImmutableMultiDict)

### (_property_) endpoint

Конечная точка, соответствующая запросу. Это в сочетании с [view\_args](dannye-vkhodyashego-zaprosa-request.md#view\_args) может использоваться для восстановления того же или измененного URL-адреса. Если при сопоставлении произошло исключение, это будет `None`.

### (_property_) files

Объект **Werkzeug** [MultiDict](https://werkzeug.palletsprojects.com/en/1.0.x/datastructures/#werkzeug.datastructures.MultiDict), содержащий все загруженные файлы. Каждый ключ в **files** - это имя из `<input type="file" name="">`. Каждое значение в **files** является объектом **Werkzeug** [FileStorage](https://werkzeug.palletsprojects.com/en/1.0.x/datastructures/#werkzeug.datastructures.FileStorage).

Он в основном ведет себя как стандартный файловый объект, который вы знаете из Python, с той разницей, что он также имеет функцию [save ()](https://werkzeug.palletsprojects.com/en/1.0.x/datastructures/#werkzeug.datastructures.FileStorage.save), которая может сохранять файл в файловой системе.

Обратите внимание, что **files** будут содержать данные только в том случае, если метод запроса был **POST**, **PUT** или **PATCH**, а `<form>`, отправленная в запрос, имела `enctype = "multipart/form-data"`. В противном случае он будет пустым.

См. документацию [MultiDict](https://werkzeug.palletsprojects.com/en/1.0.x/datastructures/#werkzeug.datastructures.MultiDict) / [FileStorage](https://werkzeug.palletsprojects.com/en/1.0.x/datastructures/#werkzeug.datastructures.FileStorage) для получения более подробной информации об используемой структуре данных.

### (_property_) form

Параметры формы. По умолчанию эта функция возвращает [ImmutableMultiDict](https://werkzeug.palletsprojects.com/en/1.0.x/datastructures/#werkzeug.datastructures.ImmutableMultiDict). Это можно изменить, задав для параметра [parameter\_storage\_class](dannye-vkhodyashego-zaprosa-request.md#parameter\_storage\_class) другой тип. Это может быть необходимо, если важен порядок данных в форме.

Имейте в виду, что загрузка файлов завершается не здесь, а в атрибуте [files](dannye-vkhodyashego-zaprosa-request.md#property-files).

_Изменено в версии 0.9:_ до `Werkzeug 0.9` он содержал только данные формы для запросов **POST** и **PUT**.

### form\_data\_parser\_class

псевдоним [werkzeug.formparser.FormDataParser](https://werkzeug.palletsprojects.com/en/1.0.x/http/#werkzeug.formparser.FormDataParser)

### (_classmethod_) from\_values( _\*args_, _\*\*kwargs_ ) → werkzeug.wrappers.request.Request

Создает новый объект запроса на основе предоставленных значений. Если задана среда, пропущенные значения заполняются оттуда. Этот метод полезен для небольших скриптов, когда вам нужно имитировать запрос с URL-адреса. Не используйте этот метод для модульного тестирования, существует полнофункциональный клиентский объект (**Client**), который позволяет создавать составные запросы, поддерживает файлы **cookie** и т. д.

Он принимает те же параметры, что и [EnvironBuilder](https://werkzeug.palletsprojects.com/en/1.0.x/test/#werkzeug.test.EnvironBuilder).

_Изменено в версии 0.5:_ этот метод теперь принимает те же аргументы, что и [EnvironBuilder](https://werkzeug.palletsprojects.com/en/1.0.x/test/#werkzeug.test.EnvironBuilder). Из-за этого параметр окружения теперь называется _**environment\_overrides**_.

**Возвращает**: объект _**request**_.

### (_property_) full\_path

Запрошенный путь, включая строку запроса.

### get\_data( _cache: bool = True_, _as\_text: bool = False_, _parse\_form\_data: bool = False_ ) → bytes

_Новое в версии 0.9_.

Считывает буферизованные входящие данные от клиента в однобайтовый объект. По умолчанию это кешируется, но это поведение можно изменить, установив для _**cache**_ значение `False`.

Обычно вызывать этот метод без предварительной проверки длины содержимого - плохая идея, поскольку клиент может отправить десятки мегабайт или более, чтобы вызвать проблемы с памятью на сервере.

Обратите внимание, что если данные формы уже были проанализированы, этот метод ничего не вернет, поскольку синтаксический анализ данных формы не кэширует данные, как этот метод. Чтобы неявно вызывать функцию синтаксического анализа данных формы, установите _**parse\_form\_data**_ значение `True`. Когда это будет сделано, возвращаемое значение этого метода будет пустой строкой, если синтаксический анализатор формы обрабатывает данные. Как правило, в этом нет необходимости, поскольку если все данные кэшируются (что по умолчанию), синтаксический анализатор формы будет использовать кэшированные данные для анализа данных формы. В любом случае, прежде чем вызывать этот метод, всегда помните о проверке длины содержимого, чтобы избежать исчерпания памяти сервера.

Если _**as\_text**_ установлен в `True`, возвращаемое значение будет декодированной строкой.

### get\_json( _force: bool = False_, _silent: bool = False_, _cache: bool = True_ )

Парсит [data](dannye-vkhodyashego-zaprosa-request.md#property-data) как **JSON**.

Если тип _**mimetype**_ не указывает на **JSON** (`application/json`, см. [is\_json ()](dannye-vkhodyashego-zaprosa-request.md#property-is\_json)), возвращается значение `None`.

Если синтаксический анализ завершается неудачно, вызывается [on\_json\_loading\_failed ()](dannye-vkhodyashego-zaprosa-request.md#on\_json\_loading\_failed), и его возвращаемое значение используется в качестве возвращаемого значения.

**Параметры:**

* **force** - Игнорируйте _**mimetype**_ и всегда пытайтесь разобрать **JSON**.
* **silent** - Отключает ошибки синтаксического анализа и вместо этого возвращает `None`.
* **cache** - Сохраняет проанализированный **JSON**, чтобы вернуть его для последующих вызовов.

### (_property_) host

Только хост, включая порт, если он доступен. См. также: _**trusted\_hosts**_.

### (_property_) host\_url

Просто хост со схемой как **IRI**. См. также: _**trusted\_hosts**_.

### (_property_) if\_match

Объект, содержащий все теги в заголовке **If-Match**.

**Возвращает тип**: [ETags](https://werkzeug.palletsprojects.com/en/1.0.x/datastructures/#werkzeug.datastructures.ETags).

### (_property_) if\_modified\_since

Распарсенный заголовок **If-Modified-Since** как объект **datetime**.

### (_property_) if\_none\_match

Объект, содержащий все теги в заголовке **If-None-Match**.

**Возвращает тип**: [ETags](https://werkzeug.palletsprojects.com/en/1.0.x/datastructures/#werkzeug.datastructures.ETags).

### (_property_) if\_range

_Новое в версии 0.7_.

Разобранный заголовок **If-Range**.

**Возвращает тип**: [IFRange](https://werkzeug.palletsprojects.com/en/1.0.x/datastructures/#werkzeug.datastructures.IfRange).

### (_property_) if\_unmodified\_since

Разобранный заголовок **If-Unmodified-Since** как объект **datetime**.

### (_property_) is\_json

Убедитесь, что тип _**mimetype**_ указывает на данные **JSON**: `application/json` или `application/* + json`.

### is\_multiprocess

логическое значение `True`, если приложение обслуживается сервером **WSGI**, который порождает несколько процессов.

### is\_multithread

логическое значение `True`, если приложение обслуживается многопоточным сервером **WSGI**.

### is\_run\_once

логическое значение `True`, если приложение будет выполняться только один раз за время существования процесса. Это относится, например, к **CGI**, но не гарантируется, что выполнение произойдет только один раз.

### (_property_) is\_secure

`True`, если запрос защищен.

### (_property_) json

Распарсенные данные **JSON**, если [mimetype](dannye-vkhodyashego-zaprosa-request.md#property-mimetype) указывает на JSON (`application/json`, см. [is\_json ()](dannye-vkhodyashego-zaprosa-request.md#property-is\_json)).

Вызывает [get\_json ()](dannye-vkhodyashego-zaprosa-request.md#get\_json-force-bool-false-silent-bool-false-cache-bool-true) с аргументами по умолчанию.

### json\_module _= \<module 'json' from '/home/docs/.pyenv/versions/3.7.9/lib/python3.7/json/\_\_init\_\_.py'>_

### list\_storage\_class

псевдоним [werkzeug.datastructures.ImmutableList](https://werkzeug.palletsprojects.com/en/1.0.x/datastructures/#werkzeug.datastructures.ImmutableList)

### make\_form\_data\_parser() → werkzeug.formparser.FormDataParser

_Новое в версии 0.8_.

Создает парсер данных формы. Создает экземпляр [form\_data\_parser\_class](dannye-vkhodyashego-zaprosa-request.md#form\_data\_parser\_class) с некоторыми параметрами.

### (_property_) max\_content\_length

Доступное только для чтения представление ключа конфигурации **MAX\_CONTENT\_LENGTH**.

### max\_forwards

Поле заголовка запроса **Max-Forwards** предоставляет механизм с методами **TRACE** и **OPTIONS** для ограничения количества прокси или шлюзов, которые могут пересылать запрос на следующий входящий сервер.

### (_property_) mimetype

Как [content\_type](dannye-vkhodyashego-zaprosa-request.md#content\_type), но без параметров (например, без кодировки, типа и т. д.) И всегда в нижнем регистре. Например, если тип содержимого - `text/HTML; charset = utf-8` _**mimetype**_ будет `'text/html'`.

### (_property_) mimetype\_params

Параметры _**mimetype**_ как **dict**. Например, если тип содержимого - `text/html; charset=utf-8` параметры будут такими: `{'charset': 'utf-8'}`.

### on\_json\_loading\_failed()

Вызывается, если синтаксический анализ [get\_json ()](dannye-vkhodyashego-zaprosa-request.md#get\_json-force-bool-false-silent-bool-false-cache-bool-true) не выполняется и не отключается. Если этот метод возвращает значение, оно используется как возвращаемое значение для **get\_json ()**. Реализация по умолчанию вызывает [BadRequest](https://werkzeug.palletsprojects.com/en/1.0.x/exceptions/#werkzeug.exceptions.BadRequest).

### origin

Хост, с которого исходил запрос. Установите **access\_control\_allow\_origin** в _**response**_, чтобы указать, какие источники разрешены.

### parameter\_storage\_class

псевдоним [werkzeug.datastructures.ImmutableMultiDict](https://werkzeug.palletsprojects.com/en/1.0.x/datastructures/#werkzeug.datastructures.ImmutableMultiDict)

### (_property_) pragma

Поле общего заголовка **Pragma** используется для включения директив, зависящих от реализации, которые могут применяться к любому получателю в цепочке request/response. Все директивы **pragma** определяют необязательное поведение с точки зрения протокола; однако некоторые системы МОГУТ требовать, чтобы поведение соответствовало директивам.

### (_property_) range

_Новое в версии 0.7_.

Распарсенный заголовок **Range**.

**Возвращает тип**: [Range](https://werkzeug.palletsprojects.com/en/1.0.x/datastructures/#werkzeug.datastructures.Range).

### referrer

Поле заголовка запроса **Referer\[sic]** позволяет клиенту указать для удобства сервера адрес (URI) ресурса, из которого был получен **Request-URI** («referrer», хотя поле заголовка написано с ошибкой).

### remote\_user

Если сервер поддерживает аутентификацию пользователя и сценарий защищен, этот атрибут содержит имя пользователя, под которым он прошел аутентификацию.

### routing\_exception _= None_

Если сопоставление URL-адреса не удалось, это исключение, которое будет вызвано / было вызвано как часть обработки запроса. Обычно это исключение [NotFound](https://werkzeug.palletsprojects.com/en/1.0.x/exceptions/#werkzeug.exceptions.NotFound) или что-то подобное.

### (_property_) script\_root

Корневой путь скрипта без косой черты в конце.

### (_property_) stream

Если входящие данные формы не были закодированы с использованием известного _**mimetype**_, данные сохраняются в этом потоке без изменений для использования. В большинстве случаев лучше использовать [data](dannye-vkhodyashego-zaprosa-request.md#property-data), которые предоставят вам эти данные в виде строки. Поток возвращает данные только один раз.

В отличие от **input\_stream**, этот поток должным образом защищен, чтобы вы не могли случайно прочитать больше длины ввода. **Werkzeug** всегда будет внутренне обращаться к этому потоку для чтения данных, что позволяет обернуть этот объект потоком, выполняющим фильтрацию.

_Изменено в версии 0.9:_ этот поток теперь всегда доступен, но позже может быть использован анализатором формы. Ранее поток устанавливался только в том случае, если не выполнялся синтаксический анализ.

### (_property_) url

Реконструированный текущий URL-адрес как IRI. См. также: _**trusted\_hosts**_.

### (_property_) url\_charset

_Новое в версии 0.6_.

Кодировка, которая предполагается для URL-адресов. По умолчанию значение **charset**.

### (_property_) url\_root

Полный корень URL-адреса (с именем хоста), это корень приложения как IRI. См. также: _**trusted\_hosts**_.

### url\_rule _= None_

_Новое в версии 0.6_.

Правило внутреннего URL-адреса, соответствующее запросу. Это может быть полезно для проверки того, какие методы разрешены для URL-адреса из обработчика до / после (`request.url_rule.methods`) и т. д. Хотя, если метод запроса был недопустимым для правила URL-адреса, действительный список вместо этого доступен в **routing\_exception.valid\_methods** (атрибут исключения **Werkzeug** [MethodNotAllowed](https://werkzeug.palletsprojects.com/en/1.0.x/exceptions/#werkzeug.exceptions.MethodNotAllowed)), потому что запрос никогда не был внутренне привязан.

### (_property_) user\_agent

Текущий пользовательский агент.

### (_property_) values

[werkzeug.datastructures.CombinedMultiDict](https://werkzeug.palletsprojects.com/en/1.0.x/datastructures/#werkzeug.datastructures.CombinedMultiDict), сочетающий [args](dannye-vkhodyashego-zaprosa-request.md#property-args) и [form](dannye-vkhodyashego-zaprosa-request.md#property-form).

### view\_args _= None_

Словарь аргументов представления, соответствующие запросу. Если при сопоставлении произошло исключение, это будет `None`.

### (_property_) want\_form\_data\_parsed

_Новое в версии 0.8_.

Возвращает `True`, если метод запроса несет контент. Начиная с `Werkzeug 0.9` это будет иметь место, если передается тип контента.

## flask.request

Для доступа к данным входящего запроса вы можете использовать глобальный объект _**request**_. **Flask** анализирует данные входящего запроса и предоставляет вам доступ к ним через этот глобальный объект. Внутренне **Flask** гарантирует, что вы всегда получаете правильные данные для активного потока, если вы находитесь в многопоточной среде.

Это прокси. См. [примечания о прокси](../rukovodstvo-polzovatelya-flask/kontekst-zaprosa-flask.md#primechaniya-k-proksi) для получения дополнительной информации.

Объект запроса является экземпляром подкласса [Request](https://werkzeug.palletsprojects.com/en/1.0.x/wrappers/#werkzeug.wrappers.Request) и предоставляет все атрибуты, определенные **Werkzeug**. Это просто краткий обзор наиболее важных из них.
