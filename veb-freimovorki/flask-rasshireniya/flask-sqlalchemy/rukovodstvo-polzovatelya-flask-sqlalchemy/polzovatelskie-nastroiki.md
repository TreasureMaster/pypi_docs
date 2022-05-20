# Пользовательские настройки

**Flask-SQLAlchemy** определяет разумные значения по умолчанию. Однако иногда требуется настройка. Существуют различные способы настройки определения моделей и взаимодействия с ними.

Эти настройки применяются при создании <mark style="color:red;">объекта SQLAlchemy</mark> и распространяются на все модели, производные от его класса **Model**.

## Класс модели

Все модели SQLAlchemy наследуются от декларативного базового класса. Это представлено как **db.Model** в Flask-SQLAlchemy, которое расширяют все модели. Это можно настроить, создав подкласс по умолчанию и передав пользовательский класс в **model\_class**.

В следующем примере каждой модели присваивается целочисленный первичный ключ или внешний ключ для наследования объединенных таблиц.

{% hint style="info" %}
Целочисленные первичные ключи для всего не обязательно являются лучшим дизайном базы данных (это зависит от требований вашего проекта), это только пример.
{% endhint %}

```python
from flask_sqlalchemy import Model, SQLAlchemy
import sqlalchemy as sa
from sqlalchemy.ext.declarative import declared_attr, has_inherited_table

class IdModel(Model):
    @declared_attr
    def id(cls):
        for base in cls.__mro__[1:-1]:
            if getattr(base, '__table__', None) is not None:
                type = sa.ForeignKey(base.id)
                break
        else:
            type = sa.Integer

        return sa.Column(type, primary_key=True)

db = SQLAlchemy(model_class=IdModel)

class User(db.Model):
    name = db.Column(db.String)

class Employee(User):
    title = db.Column(db.String)
```

## Миксины модели

Если поведение требуется только для некоторых моделей, а не для всех моделей, используйте классы примесей для настройки только этих моделей. Например, если некоторые модели должны отслеживать время их создания или обновления:

```python
from datetime import datetime

class TimestampMixin(object):
    created = db.Column(
        db.DateTime, nullable=False, default=datetime.utcnow)
    updated = db.Column(db.DateTime, onupdate=datetime.utcnow)

class Author(db.Model):
    ...

class Post(TimestampMixin, db.Model):
    ...
```

## Класс запроса

Также можно настроить то, что доступно для использования в специальном свойстве **query** моделей. Например, предоставив метод **get\_or**:

```python
from flask_sqlalchemy import BaseQuery, SQLAlchemy

class GetOrQuery(BaseQuery):
    def get_or(self, ident, default=None):
        return self.get(ident) or default

db = SQLAlchemy(query_class=GetOrQuery)

# получить пользователя по идентификатору id
# или вернуть экземпляр анонимного пользователя
user = User.query.get_or(user_id, anonymous_user)
```

И теперь все запросы, выполняемые из специального свойства **query** в моделях **Flask-SQLAlchemy**, могут использовать метод **get\_or** как часть своих запросов. Все отношения, определенные с помощью **db.relationship** (но не [sqlalchemy.orm.relationship()](https://docs.sqlalchemy.org/en/14/orm/relationship\_api.html#sqlalchemy.orm.relationship)), также будут снабжены этой функциональностью.

Также можно определить пользовательский класс запроса для отдельных отношений, указав ключевое слово **query\_class** в определении. Это работает как с **db.relationship**, так и с **sqlalchemy.relationship**:

```python
class MyModel(db.Model):
    cousin = db.relationship('OtherModel', query_class=GetOrQuery)
```

{% hint style="info" %}
Если класс запроса определен для отношения, он будет иметь приоритет над классом запроса, присоединенным к соответствующей модели.
{% endhint %}

Также можно определить конкретный класс запросов для отдельных моделей, переопределив атрибут класса **query\_class** в модели:

```python
class MyModel(db.Model):
    query_class = GetOrQuery
```

В этом случае метод **get\_or** будет доступен только для запросов, исходящих из **MyModel.query**.

## Метакласс модели

{% hint style="warning" %}
Метаклассы — это сложная тема, и вам, вероятно, не нужно настраивать их для достижения того, чего вы хотите. Здесь в основном задокументировано, чтобы показать, как отключить генерацию имени таблицы.
{% endhint %}

Метакласс модели отвечает за настройку внутренних компонентов SQLAlchemy при определении подклассов модели. Flask-SQLAlchemy добавляет некоторые дополнительные функции через примеси; его метакласс по умолчанию, **DefaultMeta**, наследует их все.

* **BindMetaMixin**: **\_\_bind\_key\_\_** извлекается из класса и применяется к таблице. См. [Несколько баз данных с привязками](neskolko-baz-dannykh-s-privyazkami.md).
* **NameMetaMixin**: если в модели не указано имя **\_\_tablename\_\_**, но указан первичный ключ, имя создается автоматически.

Вы можете добавить свои собственные варианты поведения, определив собственный метакласс и самостоятельно создав декларативную базу. Убедитесь, что вы по-прежнему наследуете миксины, которые хотите (или просто наследуете от метакласса по умолчанию).

Передача декларативного базового класса вместо базового класса простой модели, как показано выше, в **base\_class** приведет к тому, что Flask-SQLAlchemy будет использовать эту базу вместо создания ее с метаклассом по умолчанию.

```python
from flask_sqlalchemy import SQLAlchemy
from flask_sqlalchemy.model import DefaultMeta, Model

class CustomMeta(DefaultMeta):
    def __init__(cls, name, bases, d):
        # индивидуальная настройка класса может быть здесь

        # обязательно вызовите суперкласс
        super(CustomMeta, cls).__init__(name, bases, d)

    # пользовательские методы только для класса могут быть здесь

db = SQLAlchemy(model_class=declarative_base(
    cls=Model, metaclass=CustomMeta, name='Model'))
```

Вы также можете передать любые другие аргументы, которые вы хотите, в **declarative\_base()**, чтобы настроить базовый класс по мере необходимости.

### Отключение генерации имени таблицы

Некоторые проекты предпочитают устанавливать **\_\_tablename\_\_** каждой модели вручную, а не полагаться на обнаружение и генерацию Flask-SQLAlchemy. Генерацию имени таблицы можно отключить, определив собственный метакласс.

```python
from flask_sqlalchemy.model import BindMetaMixin, Model
from sqlalchemy.ext.declarative import DeclarativeMeta, declarative_base

class NoNameMeta(BindMetaMixin, DeclarativeMeta):
    pass

db = SQLAlchemy(model_class=declarative_base(
    cls=Model, metaclass=NoNameMeta, name='Model'))
```

Это создает базу, которая по-прежнему поддерживает функцию **\_\_bind\_key\_\_**, но не генерирует имена таблиц.
