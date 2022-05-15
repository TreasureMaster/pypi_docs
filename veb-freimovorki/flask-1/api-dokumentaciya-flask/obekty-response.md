# Объекты Response

## (_class_) flask.Response( _response: Union\[Iterable\[bytes], bytes, Iterable\[str], str, None] = None, status: Union\[str, int, None] = None, headers: Union\[Mapping\[str, Union\[str, int, Iterable\[Union\[str, int]]]], Iterable\[Tuple\[str, Union\[str, int]]], None] = None, mimetype: Optional\[str] = None, content\_type: Optional\[str] = None, direct\_passthrough: bool = False_)

Список аргументов:

* &#x20;**response**: Union\[Iterable\[bytes], bytes, Iterable\[str], str, None] = `None`,
* &#x20;**status:** Union\[str, int, None] = `None`,
* &#x20;**headers:** Union\[Mapping\[str, Union\[str, int, Iterable\[Union\[str, int]]]], Iterable\[Tuple\[str, Union\[str, int]]], None] = `None`,
* &#x20;**mimetype:** Optional\[str] = `None`,
* &#x20;**content\_type:** Optional\[str] = `None`,
* &#x20;**direct\_passthrough:** bool = `False`

Объект ответа, который по умолчанию используется в **Flask**. Работает так же, как объект ответа от **Werkzeug**, но по умолчанию настроен на _**mimetype**_ HTML. Довольно часто вам не нужно создавать этот объект самостоятельно, потому что [make\_response ()](obekt-prilozheniya-flask.md#make\_response-rv) позаботится об этом за вас.

Если вы хотите заменить используемый объект ответа, вы можете создать его подкласс и установить [response\_class](obekt-prilozheniya-flask.md#response\_class) для своего подкласса.

_Изменено в версии 1.0:_ в ответ, как и в запросе, добавлена поддержка **JSON**. Это полезно при тестировании, чтобы получить данные ответа тестового клиента в формате **JSON**.

_Изменено в версии 1.0:_ Добавлен [max\_cookie\_size](obekty-response.md#property-max\_cookie\_size).

### headers

Объект [Headers](https://werkzeug.palletsprojects.com/en/1.0.x/datastructures/#werkzeug.datastructures.Headers), представляющий заголовки ответа.

### status

Строка со статусом ответа.

### status\_code

Статус ответа как целое число.

### (_property_) data

Дескриптор, вызывающий **get\_data ()** и **set\_data ()**.

### get\_json( _force: bool = False_, _silent: bool = False_ ) → Optional\[Any]

Парсит [data](obekty-response.md#property-data) как **JSON**. Полезно при тестировании.

Если тип _**mimetype**_ не указывает на **JSON** (`application/json`, см. [is\_json ()](obekty-response.md#property-is\_json)), возвращается значение `None`.

В отличие от [Request.get\_json ()](dannye-vkhodyashego-zaprosa-request.md#get\_json-force-bool-false-silent-bool-false-cache-bool-true) результат не кешируется.

**Параметры:**

* **force** - Игнорирует _**mimetype**_ и всегда пытается разобрать **JSON**.
* **silent** - Отключает ошибки синтаксического анализа и вместо этого возвращает `None`.

### (property) is\_json

Убедитесь, что тип _**mimetype**_ указывает на данные **JSON**: `application/json` или `application/* + json`.

### (property) max\_cookie\_size

Доступное только для чтения представление конфигурационного ключа [MAX\_COOKIE\_SIZE](../rukovodstvo-polzovatelya-flask/obrabotka-konfiguracii-flask.md#max\_cookie\_size).

См. [max\_cookie\_size](https://werkzeug.palletsprojects.com/en/1.0.x/wrappers/#werkzeug.wrappers.BaseResponse.max\_cookie\_size) в документации **Werkzeug**.

### (property) mimetype

_**mimetype**_ (тип содержимого без кодировки и т. д.)

### set\_cookie( _key: str_, _value: str = ''_, _max\_age: Union\[datetime.timedelta_, _int_, _None] = None_, _expires: Union\[str_, _datetime.datetime_, _int_, _float_, _None] = None_, _path: Optional\[str] = '/'_, _domain: Optional\[str] = None_, _secure: bool = False_, _httponly: bool = False_, _samesite: Optional\[str] = None_ ) → None

Список аргументов:

* &#x20;key: str,
* &#x20;value: str = `''`,
* &#x20;max\_age: Union\[datetime.timedelta, int, None] = `None`,
* &#x20;expires: Union\[str, datetime.datetime, int, float, None] = `None`,
* &#x20;path: Optional\[str] = `'/'`,
* &#x20;domain: Optional\[str] = `None`,
* &#x20;secure: bool = `False`,
* &#x20;httponly: bool = `False`,
* &#x20;samesite: Optional\[str] = `None`

Устанавливает cookie.

Предупреждение появляется, если размер заголовка **cookie** превышает [max\_cookie\_size](obekty-response.md#property-max\_cookie\_size), но заголовок все равно будет установлен.

**Параметры:**

* **key** - ключ (имя) устанавливаемого файла **cookie**.
* **value** - значение **cookie**.
* **max\_age** - должно быть количество секунд или `None` (по умолчанию), если файл **cookie** должен храниться только в течение сеанса браузера клиента.
* **expires** - должен быть объектом _**datetime**_ или отметкой времени UNIX.
* **path** - ограничивает **cookie** заданным путем, по умолчанию он будет охватывать весь домен.
* **domain** - если вы хотите установить междоменный файл **cookie**. Например, `domain = ".example.com"` установит файл **cookie**, доступный для чтения доменам `www.example.com`, `foo.example.com` и т. д. В противном случае файл **cookie** будет доступен для чтения только домену, который его установил.
* **secure** - Если `True`, **cookie** будет доступен только через HTTPS.
* **httponly** - Запретить доступ **JavaScript** к **cookie**.
* **samesite** - Ограничивает область действия **cookie** только прикреплением к запросам, относящимся к «тому же сайту».
