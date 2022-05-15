# Поля вывода

**Flask-RESTful** предоставляет простой способ контролировать, какие данные вы фактически отображаете в своем ответе. С модулем <mark style="color:red;">fields</mark> вы можете использовать любые объекты (модели ORM/пользовательские классы/и т. д.), которые вы хотите в своем ресурсе. <mark style="color:red;">fields</mark> также позволяют форматировать и фильтровать ответ, поэтому вам не нужно беспокоиться о раскрытии внутренних структур данных.

Также при взгляде на ваш код очень ясно, какие данные будут отображаться и как они будут отформатированы.

## Основное использование

Вы можете определить **dict** или **OrderedDict** полей, чьи ключи являются именами атрибутов или ключей объекта для отображения, а значения которых являются классом, который будет форматировать и возвращать значение для этого поля. В этом примере есть три поля: два — <mark style="color:red;">String</mark>, а одно — <mark style="color:red;">DateTime</mark>, отформатированное как строка даты RFC 822 (также поддерживается ISO 8601).

```python
from flask_restful import Resource, fields, marshal_with

resource_fields = {
    'name': fields.String,
    'address': fields.String,
    'date_updated': fields.DateTime(dt_format='rfc822'),
}

class Todo(Resource):
    @marshal_with(resource_fields, envelope='resource')
    def get(self, **kwargs):
        return db_get_todo()  # Некоторая функция, которая запрашивает БД
```

В этом примере предполагается, что у вас есть настраиваемый объект базы данных (todo) с атрибутами **name**, **address** и **date\_updated**. Любые дополнительные атрибуты объекта считаются частными и не будут отображаться в выходных данных. Необязательный аргумент ключевого слова **envelope** указывается для оборачивания результирующего вывода.

Декоратор <mark style="color:red;">marshal\_with</mark> — это то, что на самом деле берет ваш объект данных и применяет фильтрацию полей. Маршалинг может работать с отдельными объектами, словарями или списками объектов.

{% hint style="info" %}
<mark style="color:red;">marshal\_with</mark> — удобный декоратор, функционально эквивалентный

```python
class Todo(Resource):
    def get(self, **kwargs):
        return marshal(db_get_todo(), resource_fields), 200
```
{% endhint %}

Это явное выражение может использоваться для возврата кодов состояния HTTP, отличных от **200**, вместе с успешным ответом (об ошибках см. <mark style="color:red;">abort()</mark>).

## Переименование атрибутов

Часто имя вашего общедоступного поля отличается от вашего внутреннего имени поля. Чтобы настроить это сопоставление, используйте аргумент ключевого слова **attribute**.

```python
fields = {
    'name': fields.String(attribute='private_name'),
    'address': fields.String,
}
```

Лямбда (или любая вызываемая функция) также может быть указана в качестве **attribute**

```python
fields = {
    'name': fields.String(attribute=lambda x: x._private_name),
    'address': fields.String,
}
```

Доступ к вложенным свойствам также можно получить с помощью атрибута **attribute**

```python
fields = {
    'name': fields.String(attribute='people_list.0.person_dictionary.name'),
    'address': fields.String,
}
```

## Значения по умолчанию

Если по какой-то причине ваш объект данных не имеет атрибута в списке полей, вы можете указать возвращаемое значение по умолчанию **default** вместо `None`.

```python
fields = {
    'name': fields.String(default='Anonymous User'),
    'address': fields.String,
}
```

## Пользовательские поля и несколько значений

Иногда у вас есть свои собственные потребности в форматировании. Вы можете использовать класс <mark style="color:red;">fields.Raw</mark> и реализовать функцию форматирования. Это особенно полезно, когда атрибут хранит несколько фрагментов информации, например, битовое поле, отдельные биты которого представляют различные значения. Вы можете использовать поля для мультиплексирования одного атрибута в несколько выходных значений.

В этом примере предполагается, что бит 1 в атрибуте **flags** означает элемент «**Normal**» или «**Urgent**», а бит 2 означает «**Read**» или «**Unread**». Эти элементы может быть легко хранить в битовом поле, но для удобочитаемого вывода неплохо преобразовать их в отдельные строковые поля.

```python
class UrgentItem(fields.Raw):
    def format(self, value):
        return "Urgent" if value & 0x01 else "Normal"

class UnreadItem(fields.Raw):
    def format(self, value):
        return "Unread" if value & 0x02 else "Read"

fields = {
    'name': fields.String,
    'priority': UrgentItem(attribute='flags'),
    'status': UnreadItem(attribute='flags'),
}
```

## URL и другие конкретные поля

**Flask-RESTful** включает в себя специальное поле <mark style="color:red;">fields.Url</mark>, которое синтезирует **uri** для запрашиваемого ресурса. Это также хороший пример того, как добавить в ваш ответ данные, которых на самом деле нет в вашем объекте данных:

```python
class RandomNumber(fields.Raw):
    def output(self, key, obj):
        return random.random()

fields = {
    'name': fields.String,
    # todo_resource — это имя конечной точки при вызове api.add_resource().
    'uri': fields.Url('todo_resource'),
    'random': RandomNumber,
}
```

По умолчанию <mark style="color:red;">fields.Url</mark> возвращает относительный **uri**. Чтобы сгенерировать абсолютный **uri**, включающий схему, имя хоста и порт, передайте аргумент ключевого слова `absolute=True` в объявлении поля. Чтобы переопределить схему по умолчанию, передайте аргумент ключевого слова **scheme**:

```python
fields = {
    'uri': fields.Url('todo_resource', absolute=True),
    'https_uri': fields.Url('todo_resource', absolute=True, scheme='https')
}
```

## Сложные конструкции

У вас может быть плоская структура, которую <mark style="color:red;">marshal()</mark> преобразует во вложенную структуру.

```python
>>> from flask_restful import fields, marshal
>>> import json
>>>
>>> resource_fields = {'name': fields.String}
>>> resource_fields['address'] = {}
>>> resource_fields['address']['line 1'] = fields.String(attribute='addr1')
>>> resource_fields['address']['line 2'] = fields.String(attribute='addr2')
>>> resource_fields['address']['city'] = fields.String
>>> resource_fields['address']['state'] = fields.String
>>> resource_fields['address']['zip'] = fields.String
>>> data = {'name': 'bob', 'addr1': '123 fake street', 'addr2': '', 'city': 'New York', 'state': 'NY', 'zip': '10468'}
>>> json.dumps(marshal(data, resource_fields))
'{"name": "bob", "address": {"line 1": "123 fake street", "line 2": "", "state": "NY", "zip": "10468", "city": "New York"}}'
```

{% hint style="info" %}
Поле **address** на самом деле не существует в объекте данных, но любое из подполей может обращаться к атрибутам непосредственно из объекта, как если бы они не были вложенными.
{% endhint %}

## Поле списка

Вы также можете рассортировать поля как списки.

```python
>>> from flask_restful import fields, marshal
>>> import json
>>>
>>> resource_fields = {'name': fields.String, 'first_names': fields.List(fields.String)}
>>> data = {'name': 'Bougnazal', 'first_names' : ['Emile', 'Raoul']}
>>> json.dumps(marshal(data, resource_fields))
>>> '{"first_names": ["Emile", "Raoul"], "name": "Bougnazal"}'
```

## Дополнительно: вложенное поле

В то время как вложенные поля с помощью **dicts** могут превратить плоский объект данных во вложенный ответ, вы можете использовать <mark style="color:red;">Nested</mark> для деупорядочивания вложенных структур данных и их надлежащего отображения.

```python
>>> from flask_restful import fields, marshal
>>> import json
>>>
>>> address_fields = {}
>>> address_fields['line 1'] = fields.String(attribute='addr1')
>>> address_fields['line 2'] = fields.String(attribute='addr2')
>>> address_fields['city'] = fields.String(attribute='city')
>>> address_fields['state'] = fields.String(attribute='state')
>>> address_fields['zip'] = fields.String(attribute='zip')
>>>
>>> resource_fields = {}
>>> resource_fields['name'] = fields.String
>>> resource_fields['billing_address'] = fields.Nested(address_fields)
>>> resource_fields['shipping_address'] = fields.Nested(address_fields)
>>> address1 = {'addr1': '123 fake street', 'city': 'New York', 'state': 'NY', 'zip': '10468'}
>>> address2 = {'addr1': '555 nowhere', 'city': 'New York', 'state': 'NY', 'zip': '10468'}
>>> data = { 'name': 'bob', 'billing_address': address1, 'shipping_address': address2}
>>>
>>> json.dumps(marshal_with(data, resource_fields))
'{
    "billing_address":
        {
            "line 1": "123 fake street",
            "line 2": null,
            "state": "NY",
            "zip": "10468",
            "city": "New York"
        },
    "name": "bob",
    "shipping_address":
        {
            "line 1": "555 nowhere",
            "line 2": null,
            "state": "NY",
            "zip": "10468",
            "city": "New York"
        }
}'
```

В этом примере используются два вложенных поля. Конструктор **Nested** принимает набор полей для отображения в виде подполей. Важным отличием конструктора **Nested** от вложенных словарей (предыдущий пример) является контекст для атрибутов. В этом примере **billing\_address** — это сложный объект, который имеет свои собственные поля, а контекст, передаваемый во вложенное поле, — это подобъект, а не исходный объект данных. Другими словами: `data.billing_address.addr1` находится здесь в области действия, тогда как в предыдущем примере `data.addr1` был атрибутом местоположения. Помните: вложенные объекты **Nested** и объекты списка **List** создают новую область для атрибутов.

Используйте <mark style="color:red;">Nested</mark> с <mark style="color:red;">List</mark> для маршалинга списков более сложных объектов:

```python
user_fields = {
    'id': fields.Integer,
    'name': fields.String,
}

user_list_fields = {
    fields.List(fields.Nested(user_fields)),
}
```
