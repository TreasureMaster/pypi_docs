# Парсинг запроса

{% hint style="warning" %}
Вся часть парсера запросов **Flask-RESTful** планируется к удалению и будет заменена документацией о том, как интегрироваться с другими пакетами, которые лучше выполняют ввод/вывод (например, [marshmallow](https://marshmallow.readthedocs.io/)). Это означает, что он будет поддерживаться до версии 2.0, но считается _устаревшим_. Не волнуйтесь, если у вас есть код, использующий это сейчас, и вы хотите продолжать это делать, он не исчезнет в ближайшее время.
{% endhint %}

Интерфейс разбора запросов **Flask-RESTful**, [reqparse](../spravka-api-flask-restful/reqparse.md), смоделирован по образцу интерфейса [argparse](http://docs.python.org/dev/library/argparse.html). Он разработан для обеспечения простого и единообразного доступа к любой переменной в объекте **flask.request** в Flask.

## Основные аргументы

Вот простой пример парсера запросов. Он ищет два аргумента в словаре **flask.Request.values**: целое число и строку.

```python
from flask_restful import reqparse

parser = reqparse.RequestParser()
parser.add_argument('rate', type=int, help='Rate cannot be converted')
parser.add_argument('name')
args = parser.parse_args()
```

{% hint style="info" %}
Тип аргумента по умолчанию — строка в Юникоде. Это будет **str** в python3 и **unicode** в python2.
{% endhint %}

Если вы укажете значение справки **help**, оно будет отображаться как сообщение об ошибке, когда при анализе возникает ошибка типа. Если вы не укажете справочное сообщение, по умолчанию будет возвращено сообщение из самой ошибки типа. Подробнее см. в сообщениях об ошибках.

По умолчанию аргументы **не требуются**. Кроме того, аргументы, предоставленные в запросе, которые не являются частью **RequestParser**, будут игнорироваться.

Также обратите внимание: Аргументы, объявленные в парсере вашего запроса, но не установленные в самом запросе, по умолчанию имеют значение `None`.

## Требуемые аргументы

Чтобы потребовать передачи значения в качестве аргумента, просто добавьте `required=True` к вызову [add\_argument()](../spravka-api-flask-restful/reqparse.md#add\_argument).

```python
parser.add_argument('name', required=True,
help="Name cannot be blank!")
```

## Несколько значений и списков

Если вы хотите принять несколько значений для ключа в виде списка, вы можете передать `action='append'`

```python
parser.add_argument('name', action='append')
```

Это позволит вам делать такие запросы, как

```bash
curl http://api.example.com -d "name=bob" -d "name=sue" -d "name=joe"
```

И ваши аргументы будут выглядеть так

```python
args = parser.parse_args()
args['name']    # ['bob', 'sue', 'joe']
```

## Другие направления

Если по какой-то причине вы хотите, чтобы ваш аргумент сохранялся под другим именем после его анализа, вы можете использовать аргумент ключевого слова **dest**.

```python
parser.add_argument('name', dest='public_name')

args = parser.parse_args()
args['public_name']
```

## Расположение аргументов

По умолчанию [RequestParser](../spravka-api-flask-restful/reqparse.md#class-reqparse.requestparser-argument\_class-less-than-class-reqparse.argument-greater-than-namespace) пытается проанализировать значения из **flask.Request.values** и **flask.Request.json**.

Используйте аргумент **location** для [add\_argument()](../spravka-api-flask-restful/reqparse.md#add\_argument), чтобы указать альтернативные местоположения, из которых будут извлекаться значения. Можно использовать любую переменную в [flask.Request](https://flask.palletsprojects.com/en/1.1.x/api/#flask.Request). Например:

```python
# Смотреть только в теле POST
parser.add_argument('name', type=int, location='form')

# Смотреть только в строке запроса
parser.add_argument('PageSize', type=int, location='args')

# Из заголовков запроса
parser.add_argument('User-Agent', location='headers')

# Из http-куки
parser.add_argument('session_id', location='cookies')

# Из загрузок файлов
parser.add_argument('picture', type=werkzeug.datastructures.FileStorage, location='files')
```

{% hint style="info" %}
Используйте `type=list` только тогда, когда `location='json'`. [См. этот выпуск для более подробной информации](https://github.com/flask-restful/flask-restful/issues/380)
{% endhint %}

## Несколько местоположений

Можно указать несколько местоположений аргументов, передав список в **location**:

```python
parser.add_argument('text', location=['headers', 'values'])
```

Если указано несколько местоположений, аргументы из всех указанных местоположений объединяются в один [MultiDict](https://werkzeug.palletsprojects.com/en/0.16.x/datastructures/#werkzeug.datastructures.MultiDict). Последнее указанное местоположение имеет приоритет в результирующем наборе.

Если список расположения аргументов включает расположение заголовков **headers**, имена аргументов больше не будут нечувствительны к регистру и должны совпадать с их именами в заголовке (см. [str.title()](https://docs.python.org/3/library/stdtypes.html#str.title)). Указание `location='headers'` (не в виде списка) сохранит нечувствительность к регистру.

## Наследование парсера

Часто вы будете создавать разные синтаксические анализаторы для каждого ресурса, который вы пишете. Проблема в том, что у синтаксических анализаторов есть общие аргументы. Вместо того, чтобы переписывать аргументы, вы можете написать родительский синтаксический анализатор, содержащий все общие аргументы, а затем расширить синтаксический анализатор с помощью функции [copy()](../spravka-api-flask-restful/reqparse.md#copy). Вы также можете перезаписать любой аргумент родителя с помощью [replace\_argument()](../spravka-api-flask-restful/reqparse.md#replace\_argument) или полностью удалить его с помощью [remove\_argument()](../spravka-api-flask-restful/reqparse.md#remove\_argument). Например:

```python
from flask_restful import reqparse

parser = reqparse.RequestParser()
parser.add_argument('foo', type=int)

parser_copy = parser.copy()
parser_copy.add_argument('bar', type=int)

# в parser_copy есть оба - 'foo' и 'bar'

parser_copy.replace_argument('foo', required=True, location='json')
# 'foo' теперь является обязательной строкой, расположенной в json,
# а не int, как определено исходным парсером.

parser_copy.remove_argument('foo')
# parser_copy больше не имеет аргумента «foo»
```

## Обработка ошибок

По умолчанию **RequestParser** обрабатывает ошибки, прерывая работу при первой возникшей ошибке. Это может быть полезно, когда у вас есть аргументы, обработка которых может занять некоторое время. Тем не менее, часто бывает полезно собрать все ошибки вместе и отправить их обратно клиенту сразу. Это поведение можно указать либо на уровне приложения **Flask**, либо в конкретном экземпляре **RequestParser**. Чтобы вызвать **RequestParser** с опцией объединения ошибок, передайте аргумент **bundle\_errors**. Например

```python
from flask_restful import reqparse

parser = reqparse.RequestParser(bundle_errors=True)
parser.add_argument('foo', type=int, required=True)
parser.add_argument('bar', type=int, required=True)

# Если приходит запрос, не содержащий ни «foo», ни «bar»,
# возвращаемая ошибка будет выглядеть примерно так.

{
    "message":  {
        "foo": "foo error message",
        "bar": "bar error message"
    }
}

# Поведение по умолчанию вернет только первую ошибку

parser = RequestParser()
parser.add_argument('foo', type=int, required=True)
parser.add_argument('bar', type=int, required=True)

{
    "message":  {
        "foo": "foo error message"
    }
}
```

Ключ конфигурации приложения — `«BUNDLE_ERRORS»`. Например

```python
from flask import Flask

app = Flask(__name__)
app.config['BUNDLE_ERRORS'] = True
```

{% hint style="warning" %}
**BUNDLE\_ERRORS** — это глобальный параметр, который переопределяет параметр **bundle\_errors** в отдельных экземплярах [RequestParser](../spravka-api-flask-restful/reqparse.md).
{% endhint %}

## Сообщения об ошибках

Сообщения об ошибках для каждого поля можно настроить с помощью параметра **help** для аргумента (а также `RequestParser.add_argument`).

Если параметр **help** не указан, сообщение об ошибке для поля будет строковым представлением самой ошибки типа. Если предоставляется **help**, то сообщение об ошибке будет иметь значение **help**.

**help** может включать интерполяционный токен `{error_msg}`, который будет заменен строковым представлением типа **error**. Это позволяет настроить сообщение с сохранением исходной ошибки.

```python
from flask_restful import reqparse

parser = reqparse.RequestParser()
parser.add_argument(
    'foo',
    choices=('one', 'two'),
    help='Bad choice: {error_msg}'
)

# Если приходит запрос со значением «three» для `foo`:

{
    "message":  {
        "foo": "Bad choice: three is not a valid choice",
    }
}
```
