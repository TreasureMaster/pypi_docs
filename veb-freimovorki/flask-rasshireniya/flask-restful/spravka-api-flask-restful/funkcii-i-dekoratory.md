# Функции и декораторы

## marshal()

#### flask\_restful.marshal(_data_, _fields_, _envelope=None_)

Принимает необработанные данные (в виде dict, list, object) и dict полей для вывода и фильтрует данные на основе этих полей.

#### Параметры:

* **data** - фактический объект(ы), из которого берутся поля
* **fields** - словарь, чьи ключи составят окончательный сериализованный вывод ответа
* **envelope** - необязательный ключ, который будет использоваться для упаковки сериализованного ответа

```python
>>> from flask_restful import fields, marshal
>>> data = { 'a': 100, 'b': 'foo' }
>>> mfields = { 'a': fields.Raw }

>>> marshal(data, mfields)
OrderedDict([('a', 100)])

>>> marshal(data, mfields, envelope='data')
OrderedDict([('data', OrderedDict([('a', 100)]))])
```

## marshal\_with()

#### flask\_restful.marshal\_with(_fields_, _envelope=None_)

Декоратор, применяющий сортировку к возвращаемым значениям ваших методов.

```python
>>> from flask_restful import fields, marshal_with
>>> mfields = { 'a': fields.Raw }
>>> @marshal_with(mfields)
... def get():
...     return { 'a': 100, 'b': 'foo' }
...
...
>>> get()
OrderedDict([('a', 100)])
```

```python
>>> @marshal_with(mfields, envelope='data')
... def get():
...     return { 'a': 100, 'b': 'foo' }
...
...
>>> get()
OrderedDict([('data', OrderedDict([('a', 100)]))])
```

см. [flask\_restful.marshal()](funkcii-i-dekoratory.md#marshal)

## marshal\_with\_field()

#### flask\_restful.marshal\_with\_field(_field_)

Декоратор, который форматирует возвращаемые значения ваших методов с помощью одного поля.

```python
>>> from flask_restful import marshal_with_field, fields
>>> @marshal_with_field(fields.List(fields.Integer))
... def get():
...     return ['1', 2, 3.0]
...
>>> get()
[1, 2, 3]
```

см. [flask\_restful.marshal\_with()](funkcii-i-dekoratory.md#marshal\_with)

## abort()

#### flask\_restful.abort(_http\_status\_code_, _\*\*kwargs_)

Поднимает **HTTPException** для данного **http\_status\_code**. Прикрепите любые аргументы ключевого слова к исключению для последующей обработки.
