# Быстрый старт Flask-SQLAlchemy

**Flask-SQLAlchemy** удобен в использовании, невероятно прост для базовых приложений и легко расширяется для более крупных приложений. Чтобы получить полное руководство, ознакомьтесь с документацией API по <mark style="color:red;">классу SQLAlchemy</mark>.

### Установка

Установите и обновите с помощью [pip](https://pip.pypa.io/en/stable/quickstart/):

```bash
$ pip install -U Flask-SQLAlchemy
```

### Минимальное приложение

Для общего случая наличия одного приложения Flask все, что вам нужно сделать, это создать свое приложение **Flask**, загрузить выбранную конфигурацию, а затем создать <mark style="color:red;">объект SQLAlchemy</mark>, передав ему приложение.

После создания этот объект содержит все функции и помощники как из **sqlalchemy**, так и из **sqlalchemy.orm**. Кроме того, он предоставляет класс с именем **Model**, который является декларативной базой, которую можно использовать для объявления моделей:

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:////tmp/test.db'
db = SQLAlchemy(app)

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)

    def __repr__(self):
        return '<User %r>' % self.username
```

Чтобы создать исходную базу данных, просто импортируйте объект **db** из интерактивной оболочки Python и запустите метод <mark style="color:red;">SQLAlchemy.create\_all()</mark> для создания таблиц и базы данных:

```python
>>> from yourapplication import db
>>> db.create_all()
```

Бум, и вот ваша база данных. Теперь, чтобы создать несколько пользователей:

```python
>>> from yourapplication import User
>>> admin = User(username='admin', email='admin@example.com')
>>> guest = User(username='guest', email='guest@example.com')
```

Но их еще нет в базе, поэтому давайте удостоверимся, чтобы они там появились:

```python
>>> db.session.add(admin)
>>> db.session.add(guest)
>>> db.session.commit()
```

Доступ к данным в базе данных очень прост:

```python
>>> User.query.all()
[<User u'admin'>, <User u'guest'>]
>>> User.query.filter_by(username='admin').first()
<User u'admin'>
```

Обратите внимание, что мы никогда не определяли метод **\_\_init\_\_** для класса **User**? Это связано с тем, что SQLAlchemy добавляет неявный конструктор ко всем классам моделей, который принимает аргументы ключевого слова для всех своих столбцов и отношений. Если вы решите переопределить конструктор по какой-либо причине, обязательно продолжайте принимать **\*\*kwargs** и вызывайте суперконструктор с этими **\*\*kwargs**, чтобы сохранить это поведение:

```python
class Foo(db.Model):
    # ...
    def __init__(self, **kwargs):
        super(Foo, self).__init__(**kwargs)
        # делать нестандартные вещи
```

### Простые отношения

SQLAlchemy подключается к реляционным базам данных, а реляционные базы данных действительно хороши в отношениях. Таким образом, у нас будет пример приложения, в котором используются две таблицы, связанные друг с другом:

```python
from datetime import datetime

class Post(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(80), nullable=False)
    body = db.Column(db.Text, nullable=False)
    pub_date = db.Column(db.DateTime, nullable=False,
        default=datetime.utcnow)

    category_id = db.Column(db.Integer, db.ForeignKey('category.id'),
        nullable=False)
    category = db.relationship('Category',
        backref=db.backref('posts', lazy=True))

    def __repr__(self):
        return '<Post %r>' % self.title

class Category(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(50), nullable=False)

    def __repr__(self):
        return '<Category %r>' % self.name
```

Сначала создадим несколько объектов:

```python
>>> py = Category(name='Python')
>>> Post(title='Hello Python!', body='Python is pretty cool', category=py)
>>> p = Post(title='Snakes', body='Ssssssss')
>>> py.posts.append(p)
>>> db.session.add(py)
```

Как видите, нет необходимости добавлять объекты **Post** в сеанс. Поскольку **Category** является частью сеанса, _**все объекты, связанные с ней отношениями, также будут добавлены**_. Неважно, вызывается ли **db.session.add()** до или после создания этих объектов. Ассоциация также может быть выполнена с любой стороны отношения — так что сообщение может быть создано с категорией или добавлено в список сообщений категории.

Смотрим посты. Доступ к ним загрузит их из базы данных, поскольку отношения загружаются лениво, но вы, вероятно, не заметите разницы — загрузка списка происходит довольно быстро:

```python
>>> py.posts
[<Post 'Hello Python!'>, <Post 'Snakes'>]
```

Хотя отложенная загрузка отношений выполняется быстро, она может легко стать серьезным узким местом, если вы в конечном итоге инициируете дополнительные запросы в цикле для более чем нескольких объектов. В этом случае SQLAlchemy позволяет переопределить стратегию загрузки на уровне запроса. Если вы хотите, чтобы один запрос загрузил все категории и их сообщения, вы можете сделать это следующим образом:

```python
>>> from sqlalchemy.orm import joinedload
>>> query = Category.query.options(joinedload('posts'))
>>> for category in query:
...     print category, category.posts
<Category u'Python'> [<Post u'Hello Python!'>, <Post u'Snakes'>]
```

Если вы хотите получить объект запроса для этой связи, вы можете сделать это с помощью **with\_parent()**. Например, давайте исключим этот пост о змеях:

```python
>>> Post.query.with_parent(py).filter(Post.title != 'Snakes').all()
[<Post 'Hello Python!'>]
```

### Дорога к просветлению

Единственное, что вам нужно знать по сравнению с простой SQLAlchemy:

1. <mark style="color:red;">SQLAlchemy</mark> дает вам доступ к следующим вещам:
   * все функции и классы из **sqlalchemy** и **sqlalchemy.orm**
   * предварительно настроенный сеанс с ограниченной областью действия, называемый **session**
   * <mark style="color:red;">metadata</mark>
   * <mark style="color:red;">engine</mark>
   * методы <mark style="color:red;">SQLAlchemy.create\_all()</mark> и <mark style="color:red;">SQLAlchemy.drop\_all()</mark> для создания и удаления таблиц в соответствии с моделями.
   * базовый класс <mark style="color:red;">Model</mark>, который является сконфигурированной декларативной базой.
2. Декларативный базовый класс <mark style="color:red;">Model</mark> ведет себя как обычный класс Python, но имеет прикрепленный атрибут запроса **query**, который можно использовать для запроса модели. (<mark style="color:red;">Model</mark> и <mark style="color:red;">BaseQuery</mark>)
3. Вы должны зафиксировать сеанс, но вам не нужно удалять его в конце запроса, **Flask-SQLAlchemy** сделает это за вас.
