# Введение в контексты

Если вы планируете использовать только одно приложение, вы можете пропустить эту главу. Просто передайте свое приложение <mark style="color:red;">конструктору SQLAlchemy</mark>, и все готово. Однако, если вы хотите использовать более одного приложения или создать приложение динамически в функции, вам нужно прочитать.

Если вы определяете свое приложение в функции, но <mark style="color:red;">объект SQLAlchemy</mark> глобально, как последний узнает о первом? Ответ — функция <mark style="color:red;">init\_app()</mark>:

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

def create_app():
    app = Flask(__name__)
    db.init_app(app)
    return app
```

Что она делает, так это подготавливает приложение к работе с <mark style="color:red;">SQLAlchemy</mark>. Однако теперь это не связывает <mark style="color:red;">объект SQLAlchemy</mark> с вашим приложением. Почему это не так? Потому что может быть создано более одного приложения.

Так как же <mark style="color:red;">SQLAlchemy</mark> узнает о вашем приложении? Вам нужно будет настроить контекст приложения. Если вы работаете внутри функции представления Flask или команды CLI, это происходит автоматически. Однако, если вы работаете внутри интерактивной оболочки, вам придется сделать это самостоятельно (см. [Создание контекста приложения](https://flask.palletsprojects.com/en/1.0.x/appcontext/#manually-push-a-context)).

Если вы попытаетесь выполнить операции с базой данных вне контекста приложения, вы увидите следующую ошибку:

{% hint style="warning" %}
Приложение не найдено. Либо работайте внутри функции представления, либо протолкните (push) контекст приложения.
{% endhint %}

В двух словах, сделайте что-то вроде этого:

```python
>>> from yourapp import create_app
>>> app = create_app()
>>> app.app_context().push()
```

В качестве альтернативы используйте оператор **with**, чтобы позаботиться об установке и удалении:

```python
def my_function():
    with app.app_context():
        user = db.User(...)
        db.session.add(user)
        db.session.commit()
```

Некоторые функции внутри **Flask-SQLAlchemy** также могут опционально принимать приложение для работы:

```python
>>> from yourapp import db, create_app
>>> db.create_all(app=create_app())
```
