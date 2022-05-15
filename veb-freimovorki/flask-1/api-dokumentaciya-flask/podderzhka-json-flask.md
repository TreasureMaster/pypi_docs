# Поддержка JSON Flask

**Flask** использует **simplejson** для реализации **JSON**. Поскольку **simplejson** предоставляется как стандартной библиотекой, так и расширением, **Flask** сначала попробует **simplejson**, а затем вернется к модулю **stdlib json**. Кроме того, он делегирует доступ кодировщикам и декодерам **JSON** текущего приложения для упрощения настройки.

Итак, для начала, вместо того, чтобы делать:

```python
try:
    import simplejson as json
except ImportError:
    import json
```

Вместо этого вы можете просто сделать это:

```python
from flask import json
```

Примеры использования см. в документации по [json](https://docs.python.org/3/library/json.html#module-json) в стандартной библиотеке. Следующие расширения по умолчанию применяются к модулю JSON **stdlib**:

1. Объекты **datetime** сериализуются как строки [RFC 822](https://tools.ietf.org/html/rfc822.html).
2. Для любого объекта с методом `__html__` (например, [Markup](poleznye-funkcii-i-klassy-flask.md#klass-flask-markup)) этот метод будет вызван, а затем возвращаемое значение будет сериализовано в виде строки.

Функция **htmlsafe\_dumps ()** этого json-модуля также доступна как фильтр под названием `|tojson` в **Jinja2**. Обратите внимание, что в версиях **Flask** до `Flask 0.10` вы должны отключить экранирование с помощью `|safe`, если вы собираетесь использовать вывод `|tojson` внутри тегов `<script>`. Во `Flask 0.10` и более поздних версиях это происходит автоматически (но в любом случае безвредно включать `|safe`).

```markup
<script type=text/javascript>
    doSomethingWith({{ user.username|tojson|safe }});
</script>
```

{% hint style="info" %}
**Автоматическая сортировка ключей JSON:**

Переменная конфигурации **JSON\_SORT\_KEYS** ([обработка конфигурации](../rukovodstvo-polzovatelya-flask/obrabotka-konfiguracii-flask.md)) может быть установлена в значение **false**, чтобы предотвратить автоматическую сортировку ключей **Flask**. По умолчанию сортировка установлена, а сортировка вне контекста приложения включена.

Обратите внимание, что отключение сортировки ключей может вызвать проблемы при использовании HTTP-кешей на основе содержимого и функции рандомизации хэшей Python.
{% endhint %}

### flask.json.jsonify( _\*args_, _\*\*kwargs_ )

_Новое в версии 0.2_.

Эта функция обертывает [dumps ()](podderzhka-json-flask.md#flask-json-dumps), чтобы добавить несколько улучшений, облегчающих жизнь. Он превращает вывод **JSON** в объект [Response](obekty-response.md#class-flask-response-response-union-iterable-bytes-bytes-iterable-str-str-none-none-status-union-str-int-none-none-headers-union-mapping-str-union-str-int-iterable-union-str-int-iterable-tuple-str-union-str-int-none-none-mimetype-optional-str-none-content\_type-optional-str-none-direct\_passthrough-bool-false) с **mimetype** `application/json`. Для удобства он также преобразует несколько аргументов в массив или несколько аргументов ключевого слова в словарь. Это означает, что и `jsonify(1,2,3)`, и `jsonify([1,2,3])` сериализуются в `[1,2,3]`.

Для наглядности поведение сериализации **JSON** имеет следующие отличия от [dumps ()](podderzhka-json-flask.md#flask-json-dumps):

1. Единственный аргумент: передается напрямую в [dumps ()](podderzhka-json-flask.md#flask-json-dumps).
2. Несколько аргументов: преобразовываются в массив перед передачей в **dumps ()**.
3. Несколько аргументов ключевого слова: преобразуются в **dict** перед передачей в **dumps ()**.
4. Оба аргумента и **kwargs**: поведение не определено и вызовет исключение.

Пример использования:

```python
from flask import jsonify

@app.route('/_get_current_user')
def get_current_user():
    return jsonify(username=g.user.username,
                   email=g.user.email,
                   id=g.user.id)
```

Это отправит в браузер такой ответ JSON:

```javascript
{
    "username": "admin",
    "email": "admin@localhost",
    "id": 42
}
```

_Изменено в версии 0.11:_ Добавлена поддержка сериализации массивов верхнего уровня. Это создает угрозу безопасности в старых браузерах. Подробнее см. [Безопасность JSON](../dopolnitelnye-primechaniya-flask/soobrazheniya-bezopasnosti-flask.md#bezopasnost-json).

Ответ этой функции будет хорошо напечатан, если для параметра конфигурации **JSONIFY\_PRETTYPRINT\_REGULAR** установлено значение `True` или приложение **Flask** работает в режиме отладки. Сжатое (некрасивое) форматирование в настоящее время означает отсутствие отступов и пробелов после разделителей.

### flask.json.dumps( _obj_, _app=None_, _\*\*kwargs_ )

Сериализует _**obj**_ в строку в формате **JSON**. Если передается контекст приложения, используйте настроенный кодировщик текущего приложения ([json\_encoder](obekt-prilozheniya-flask.md#json\_encoder)) или вернитесь к [JSONEncoder](podderzhka-json-flask.md#class-flask-json-jsonencoder) по умолчанию.

Принимает те же аргументы, что и встроенный в Python [json.dumps ()](https://docs.python.org/3/library/json.html#json.dumps), и выполняет некоторую дополнительную настройку в зависимости от приложения. Если установлен пакет **simplejson**, это предпочтительнее.

**Параметры:**

* **obj** - Объект для сериализации в **JSON**.
* **app** - Экземпляр приложения для настройки кодировщика **JSON**. Использует **current\_app**, если не указано, и возвращается к кодировщику по умолчанию, когда не в контексте приложения.
* **kwargs** - В [json.dumps ()](https://docs.python.org/3/library/json.html#json.dumps) из Python передаются дополнительные аргументы.

_Изменено в версии 1.0.3:_ приложение **app** можно передавать напрямую, вместо того, чтобы требовать контекст приложения для настройки.

### flask.json.dump( _obj_, _fp_, _app=None_, _\*\*kwargs_ )

Как [dumps ()](podderzhka-json-flask.md#flask-json-dumps-obj-app-none-kwargs), но записывает в файловый объект.

### flask.json.loads( _s_, _app=None_, _\*\*kwargs_ )

Десериализует объект из строки _**s**_ в формате **JSON**. Если передается контекст приложения, используйте настроенный декодер текущего приложения ([json\_decoder](obekt-prilozheniya-flask.md#json\_decoder)) или вернитесь к [JSONDecoder](podderzhka-json-flask.md#class-flask-json-jsondecoder) по умолчанию.

Принимает те же аргументы, что и встроенный [json.loads ()](https://docs.python.org/3/library/json.html#json.loads), и выполняет некоторую дополнительную настройку в зависимости от приложения. Если установлен пакет **simplejson**, это предпочтительнее.

**Параметры:**

* **s** - Строка **JSON** для десериализации.
* **app** - Экземпляр приложения для настройки декодера **JSON**. Использует **current\_app**, если не указано, и возвращается к кодировщику по умолчанию, когда не в контексте приложения.
* **kwargs** - Дополнительные аргументы передаются в [json.loads ()](https://docs.python.org/3/library/json.html#json.loads).

_Изменено в версии 1.0.3:_ приложение **app** можно передавать напрямую, вместо того, чтобы требовать контекст приложения для настройки.

### flask.json.load( _fp_, _app=None_, _\*\*kwargs_ )

Подобно [loads ()](podderzhka-json-flask.md#flask-json-loads-s-app-none-kwargs), но читает из файлового объекта.

## (_class_) flask.json.JSONEncoder( _\*_, _skipkeys=False_, _ensure\_ascii=True_, _check\_circular=True_, _allow\_nan=True_, _sort\_keys=False_, _indent=None_, _separators=None_, _default=None_ )

Принимаемые параметры:

* &#x20;\*,
* &#x20;skipkeys=`False`,
* &#x20;ensure\_ascii=`True`,
* &#x20;check\_circular=`True`,
* &#x20;allow\_nan=`True`,
* &#x20;sort\_keys=`False`,
* &#x20;indent=`None`,
* &#x20;separators=`None`,
* &#x20;default=`None`

Кодировщик **JSON** **Flask** по умолчанию. Он расширяет кодировщик по умолчанию, также поддерживая объекты **datetime**, **UUID**, **dataclasses** и **Markup**.

Объекты **datetime** сериализуются как строки даты и времени **RFC 822**. Это то же самое, что и формат даты **HTTP**.

Чтобы поддерживать больше типов данных, переопределите метод [default ()](podderzhka-json-flask.md#default).

### default( _o_ )

Реализуйте этот метод в подклассе так, чтобы он возвращал сериализуемый объект для **o** или вызывал базовую реализацию (чтобы вызвать [TypeError](https://docs.python.org/3/library/exceptions.html#TypeError)).

Например, для поддержки произвольных итераторов вы можете реализовать значение по умолчанию следующим образом:

```python
def default(self, o):
    try:
        iterable = iter(o)
    except TypeError:
        pass
    else:
        return list(iterable)
    return JSONEncoder.default(self, o)
```

## (_class_) flask.json.JSONDecoder( _\*_, _object\_hook=None_, _parse\_float=None_, _parse\_int=None_, _parse\_constant=None_, _strict=True_, _object\_pairs\_hook=None_)

Принимаемые параметры:

* &#x20;\*,
* &#x20; object\_hook=`None`,
* &#x20;  parse\_float=`None`,
* &#x20;  parse\_int=`None`,
* &#x20;  parse\_constant=`None`,
* &#x20;  strict=`True`,
* &#x20;  object\_pairs\_hook=`None`

Декодер **JSON** по умолчанию. Это не меняет поведения декодера **simplejson** по умолчанию. Дополнительную информацию см. в документации по [json](https://docs.python.org/3/library/json.html#module-json). Этот декодер используется не только для функций загрузки этого модуля, но и для функции [Request](dannye-vkhodyashego-zaprosa-request.md#klass-flask-request-environ-wsgienvironment-populate\_request-bool-true-shallow-bool-false).

## JSON с тегами

Компактное представление для сериализации нестандартных типов **JSON** без потерь. [SecureCookieSessionInterface](sessii-flask.md#klass-flask-session-securecookiesessioninterface) использует это для сериализации данных сеанса, но может быть полезно в других местах. Его можно расширить для поддержки других типов.

## (class) flask.json.tag.TaggedJSONSerializer

Сериализатор, использующий систему тегов для компактного представления объектов, не являющихся типами **JSON**. Передано как промежуточный сериализатор **itsdangerous.Serializer**.

Поддерживаются следующие дополнительные типы:

* [dict](https://docs.python.org/3/library/stdtypes.html#dict)
* [tuple](https://docs.python.org/3/library/stdtypes.html#tuple)
* [bytes](https://docs.python.org/3/library/stdtypes.html#bytes)
* [Markup](poleznye-funkcii-i-klassy-flask.md#klass-flask-markup)
* [UUID](https://docs.python.org/3/library/uuid.html#uuid.UUID)
* [datetime](https://docs.python.org/3/library/datetime.html#datetime.datetime)

### default\_tags _= \[\<class 'flask.json.tag.TagDict'>, \<class 'flask.json.tag.PassDict'>, \<class 'flask.json.tag.TagTuple'>, \<class 'flask.json.tag.PassList'>, \<class 'flask.json.tag.TagBytes'>, \<class 'flask.json.tag.TagMarkup'>, \<class 'flask.json.tag.TagUUID'>, \<class 'flask.json.tag.TagDateTime'>]_

Классы по умолчанию:

* &#x20;_\<class 'flask.json.tag.TagDict'>,_
* &#x20;_\<class 'flask.json.tag.PassDict'>,_
* &#x20;_\<class 'flask.json.tag.TagTuple'>,_
* &#x20;_\<class 'flask.json.tag.PassList'>,_
* &#x20;_\<class 'flask.json.tag.TagBytes'>,_
* &#x20;_\<class 'flask.json.tag.TagMarkup'>,_
* &#x20;_\<class 'flask.json.tag.TagUUID'>,_
* &#x20;_\<class 'flask.json.tag.TagDateTime'>_

Классы тегов для привязки при создании сериализатора. Другие теги можно добавить позже с помощью [register ()](podderzhka-json-flask.md#register).

### dumps( _value_ )

Помечает значение и выгружает его в компактную строку **JSON**.

### loads( _value_ )

Загружает данные из строки **JSON** и десериализует все помеченные объекты.

### register( _tag\_class_, _force=False_, _index=None_ )

Регистрирует новый тег с помощью этого сериализатора.

**Параметры:**

* **tag\_class** - класс тега для регистрации. Будет создан экземпляр с этим экземпляром сериализатора.
* **force** - перезаписывает существующий тег. Если `false` (по умолчанию), возникает ошибка [KeyError](https://docs.python.org/3/library/exceptions.html#KeyError).
* **index** - индекс, чтобы вставить новый тег в порядке тегов. Полезно, когда новый тег является частным случаем существующего тега. Если `None` (по умолчанию), тег добавляется в конец заказа.

**Возбуждает:**

* **KeyError** - если ключ тега уже зарегистрирован и **force** не соответствует действительности.

### tag( _value_ )

При необходимости преобразует значение в представление с тегами.

### untag( _value_ )

Преобразует представление с тегами обратно в исходный тип.

## (class) flask.json.tag.JSONTag( _serializer_ )

Базовый класс для определения тегов типов для [TaggedJSONSerializer](podderzhka-json-flask.md#class-flask-json-tag-taggedjsonserializer).

### check( _value_ )

Проверяет, должно ли данное значение быть помечено этим тегом.

### key _= None_

Тег, которым нужно пометить сериализованный объект. Если `None`, этот тег используется только в качестве промежуточного шага во время тегирования.

### tag( _value_ )

Преобразует значение в допустимый тип **JSON** и добавляет вокруг него структуру тегов.

### to\_json( _value_ )

Преобразуйте объект Python в объект допустимого типа **JSON**. Тег будет добавлен позже.

### to\_python( _value_ )

Преобразуйте представление **JSON** обратно в правильный тип. Тег уже будет удален.

Давайте посмотрим на пример, который добавляет поддержку [OrderedDict](https://docs.python.org/3/library/collections.html#collections.OrderedDict). В словарях dicts нет порядка в Python или **JSON**, поэтому, чтобы справиться с этим, мы будем выгружать элементы в виде списка пар `[key, value]`. Подкласс [JSONTag](podderzhka-json-flask.md#class-flask-json-tag-jsontag-serializer) и присвойте ему новый ключ `' od'` для идентификации типа. Сериализатор сеанса сначала обрабатывает **dicts**, поэтому вставьте новый тег в начало заказа, поскольку **OrderedDict** должен быть обработан до **dict**.

```python
from flask.json.tag import JSONTag

class TagOrderedDict(JSONTag):
    __slots__ = ('serializer',)
    key = ' od'

    def check(self, value):
        return isinstance(value, OrderedDict)

    def to_json(self, value):
        return [[k, self.serializer.tag(v)] for k, v in iteritems(value)]

    def to_python(self, value):
        return OrderedDict(value)

app.session_interface.serializer.register(TagOrderedDict, index=0)
```
