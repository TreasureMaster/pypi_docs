# MongoDB с MongoEngine

Использование базы данных документов, такой как **MongoDB**, является распространенной альтернативой реляционным базам данных **SQL**. Этот шаблон показывает, как использовать [MongoEngine](http://mongoengine.org/), библиотеку сопоставления документов, для интеграции с **MongoDB**.

Требуются работающий сервер **MongoDB** и [Flask-MongoEngine](http://docs.mongoengine.org/projects/flask-mongoengine/en/latest/).

```bash
pip install flask-mongoengine
```

## Конфигурация

Базовую настройку можно выполнить, указав **MONGODB\_SETTINGS** в `app.config` и создав экземпляр **MongoEngine**.

```python
from flask import Flask
from flask_mongoengine import MongoEngine

app = Flask(__name__)
app.config['MONGODB_SETTINGS'] = {
    "db": "myapp",
}
db = MongoEngine(app)
```

## Отображение документов

Чтобы объявить модель, представляющую документ **Mongo**, создайте класс, наследующий от **Document**, и объявите каждое из полей.

```python
import mongoengine as me

class Movie(me.Document):
    title = me.StringField(required=True)
    year = me.IntField()
    rated = me.StringField()
    director = me.StringField()
    actors = me.ListField()
```

Если в документе есть вложенные поля, используйте **EmbeddedDocument** для определения полей встроенного документа и **EmbeddedDocumentField** для объявления его в родительском документе.

```python
class Imdb(me.EmbeddedDocument):
    imdb_id = me.StringField()
    rating = me.DecimalField()
    votes = me.IntField()

class Movie(me.Document):
    ...
    imdb = me.EmbeddedDocumentField(Imdb)
```

## Создание данных

Создайте экземпляр своего класса документа с ключевыми аргументами для полей. Вы также можете присвоить значения атрибутам поля после создания экземпляра. Затем вызовите `doc.save ()`.

```python
bttf = Movie(title="Back To The Future", year=1985)
bttf.actors = [
    "Michael J. Fox",
    "Christopher Lloyd"
]
bttf.imdb = Imdb(imdb_id="tt0088763", rating=8.5)
bttf.save()
```

## Запросы

Используйте атрибут класса **objects** для выполнения запросов. Аргумент ключевого слова ищет в поле равное значение.

```python
bttf = Movies.objects(title="Back To The Future").get_or_404()
```

Можно использовать операторы запроса, объединив их с именем поля с помощью двойного подчеркивания. **objects** и запросы, возвращаемые при его вызове, являются повторяемыми.

```python
some_theron_movie = Movie.objects(actors__in=["Charlize Theron"]).first()

for recents in Movie.objects(year__gte=2017):
    print(recents.title)
```

## Документация

Есть еще много способов определять и запрашивать документы с помощью **MongoEngine**. Для получения дополнительной информации ознакомьтесь с [официальной документацией](http://mongoengine.org/).

**Flask-MongoEngine** добавляет полезные утилиты поверх **MongoEngine**. Также ознакомьтесь с их [документацией](http://docs.mongoengine.org/projects/flask-mongoengine/en/latest/).
