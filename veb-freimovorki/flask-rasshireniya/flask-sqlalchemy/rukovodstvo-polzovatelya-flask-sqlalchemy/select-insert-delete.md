# Select, Insert, Delete

Теперь, когда вы [объявили модели](obyavlenie-modelei.md), пришло время запросить данные из базы данных. Мы будем использовать определения модели из главы [Quickstart](bystryi-start-flask-sqlalchemy.md).

### Вставка записей

Прежде чем мы сможем что-то запросить, нам нужно будет вставить некоторые данные. Все ваши модели должны иметь конструктор, поэтому обязательно добавьте его, если забыли. Конструкторы используются только вами, а не SQLAlchemy внутри, поэтому только от вас зависит, как вы их определяете.

Вставка данных в базу данных представляет собой трехэтапный процесс:

1. Создайте объект Python
2. Добавьте его в сессию
3. Зафиксировать сеанс

Сеанс здесь не сеанс Flask, а сеанс Flask-SQLAlchemy. По сути, это усиленная версия транзакции базы данных. Вот как это работает:

```python
>>> from yourapp import User
>>> me = User('admin', 'admin@example.com')
>>> db.session.add(me)
>>> db.session.commit()
```

Ладно, это было не сложно. Что происходит в какой момент? Прежде чем вы добавите объект в сеанс, SQLAlchemy в основном не планирует добавлять его в транзакцию. Это хорошо, потому что вы все еще можете отказаться от изменений. Например, подумайте о создании сообщения на странице, но вы хотите только передать сообщение в шаблон для предварительного рендеринга, а не хранить его в базе данных.

Затем вызов функции **add()** добавляет объект. Он выдаст оператор **INSERT** для базы данных, но, поскольку транзакция еще не зафиксирована, вы не получите идентификатор немедленно. Если вы сделаете коммит, у вашего пользователя будет идентификатор:

```python
>>> me.id
1
```

### Удаление записей

Удаление записей очень похоже, вместо **add()** используйте **delete()**:

```python
>>> db.session.delete(me)
>>> db.session.commit()
```

### Запрос записей

Так как же нам получить данные обратно из нашей базы данных? Для этой цели Flask-SQLAlchemy предоставляет атрибут **query** в вашем классе <mark style="color:red;">Model</mark>. При доступе к нему вы получите новый объект запроса по всем записям. Затем вы можете использовать такие методы, как **filter()**, для фильтрации записей, прежде чем запускать выбор с помощью **all()** или **first()**. Если вы хотите использовать первичный ключ, вы также можете использовать **get()**.

Следующие запросы предполагают следующие записи в базе данных:

| id | username | email             |
| -- | -------- | ----------------- |
| 1  | admin    | admin@example.com |
| 2  | peter    | peter@example.org |
| 3  | guest    | guest@example.com |

Получить пользователя по имени **username**:

```python
>>> peter = User.query.filter_by(username='peter').first()
>>> peter.id
2
>>> peter.email
u'peter@example.org'
```

То же, что и выше, но для несуществующего имени пользователя дает `None`:

```python
>>> missing = User.query.filter_by(username='missing').first()
>>> missing is None
True
```

Выбор группы пользователей по более сложному выражению:

```python
>>> User.query.filter(User.email.endswith('@example.com')).all()
[<User u'admin'>, <User u'guest'>]
```

Упорядочивание пользователей по чему-либо:

```python
>>> User.query.order_by(User.username).all()
[<User u'admin'>, <User u'guest'>, <User u'peter'>]
```

Ограничение пользователей:

```python
>>> User.query.limit(1).all()
[<User u'admin'>]
```

Получение пользователя по первичному ключу:

```python
>>> User.query.get(1)
<User u'admin'>
```

### Запросы в представлениях (views)

Если вы пишете функцию представления Flask, часто очень удобно возвращать ошибку 404 для отсутствующих записей. Поскольку это очень распространенная идиома, Flask-SQLAlchemy предоставляет помощник именно для этой цели. Вместо **get()** можно использовать **get\_or\_404()** и вместо **first()** - **first\_or\_404()**. Это приведет к ошибке 404 вместо возврата `None`:

```python
@app.route('/user/<username>')
def show_user(username):
    user = User.query.filter_by(username=username).first_or_404()
    return render_template('show_user.html', user=user)
```

Кроме того, если вы хотите добавить описание с помощью **abort()**, вы также можете использовать его в качестве аргумента.

```python
>>> User.query.filter_by(
...    username=username
... ).first_or_404(
...    description='There is no data with {}'.format(username)
... )
```
