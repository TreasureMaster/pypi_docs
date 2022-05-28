# Flask-HTTPAuth

**Flask-HTTPAuth** — это расширение Flask, которое упрощает использование HTTP-аутентификации с маршрутами Flask.

## Примеры базовой аутентификации

В следующем примере приложение использует обычную HTTP-аутентификацию для защиты маршрута `'/'`:

```python
from flask import Flask
from flask_httpauth import HTTPBasicAuth
from werkzeug.security import generate_password_hash, check_password_hash

app = Flask(__name__)
auth = HTTPBasicAuth()

users = {
    "john": generate_password_hash("hello"),
    "susan": generate_password_hash("bye")
}

@auth.verify_password
def verify_password(username, password):
    if username in users and \
            check_password_hash(users.get(username), password):
        return username

@app.route('/')
@auth.login_required
def index():
    return "Hello, {}!".format(auth.current_user())

if __name__ == '__main__':
    app.run()
```

Функция, украшенная декоратором **verify\_password**, получает имя пользователя **username** и пароль **password**, отправленные клиентом. Если учетные данные принадлежат пользователю, функция должна вернуть объект пользователя. Если учетные данные недействительны, функция может вернуть `None` или `False`. Затем объект пользователя можно запросить из метода **current\_user()** экземпляра аутентификации.

## Пример дайджест-аутентификации

В следующем примере используется аутентификация HTTP **Digest**:

```python
from flask import Flask
from flask_httpauth import HTTPDigestAuth

app = Flask(__name__)
app.config['SECRET_KEY'] = 'secret key here'
auth = HTTPDigestAuth()

users = {
    "john": "hello",
    "susan": "bye"
}

@auth.get_password
def get_pw(username):
    if username in users:
        return users.get(username)
    return None

@app.route('/')
@auth.login_required
def index():
    return "Hello, {}!".format(auth.username())

if __name__ == '__main__':
    app.run()
```

## Пример аутентификации по токену

В следующем примере приложения используется настраиваемая схема проверки подлинности HTTP для защиты маршрута `'/'` с помощью токена:

```python
from flask import Flask
from flask_httpauth import HTTPTokenAuth

app = Flask(__name__)
auth = HTTPTokenAuth(scheme='Bearer')

tokens = {
    "secret-token-1": "john",
    "secret-token-2": "susan"
}

@auth.verify_token
def verify_token(token):
    if token in tokens:
        return tokens[token]

@app.route('/')
@auth.login_required
def index():
    return "Hello, {}!".format(auth.current_user())

if __name__ == '__main__':
    app.run()
```

**HTTPTokenAuth** — это универсальный обработчик проверки подлинности, который можно использовать с нестандартными схемами проверки подлинности, при этом имя схемы указывается в качестве аргумента в конструкторе. В приведенном выше примере заголовок **WWW-Authenticate**, предоставленный сервером, будет использовать **Bearer** в качестве схемы:

```http
WWW-Authenticate: Bearer realm="Authentication Required"
```

Обратный вызов **verify\_token** получает учетные данные аутентификации, предоставленные клиентом в заголовке **Authorization**. Это может быть простой токен или может содержать несколько аргументов, которые функция должна будет проанализировать и извлечь из строки. Как и в случае с **verify\_password**, функция должна возвращать объект пользователя, если токен действителен.

В [каталоге примеров](https://github.com/miguelgrinberg/Flask-HTTPAuth/tree/main/examples) вы можете найти полный пример, в котором используются токены JWS. Токены JWS аналогичны токенам JWT. Однако использование токенов JWT потребует внешней зависимости.

## Использование нескольких схем аутентификации

Иногда приложениям необходимо поддерживать комбинацию методов аутентификации. Например, веб-приложение может быть аутентифицировано путем отправки идентификатора и секрета клиента в рамках обычной аутентификации, в то время как сторонние API-клиенты используют токен носителя **JWS** или **JWT**. Класс **MultiAuth** позволяет защитить маршрут более чем одним объектом аутентификации. Чтобы предоставить доступ к конечной точке, один из методов аутентификации должен пройти проверку.

В [каталоге примеров](https://github.com/miguelgrinberg/Flask-HTTPAuth/tree/main/examples) вы можете найти полный пример, использующий базовую и токеновую аутентификацию.

## Роли пользователей

**Flask-HTTPAuth** включает в себя простую систему аутентификации на основе ролей, которую можно дополнительно добавить, чтобы обеспечить дополнительный уровень детализации при фильтрации доступа к маршрутам. Чтобы включить поддержку ролей, напишите функцию, которая возвращает список ролей для данного пользователя, и украсьте его декоратором **get\_user\_roles**:

```python
@auth.get_user_roles
def get_user_roles(user):
    return user.get_roles()
```

Чтобы ограничить доступ к маршруту пользователям с заданной ролью, добавьте аргумент **role** в декоратор **login\_required**:

```python
@app.route('/admin')
@auth.login_required(role='admin')
def admins_only():
    return "Hello {}, you are an admin!".format(auth.current_user())
```

Аргумент **role** может принимать список ролей, и в этом случае пользователям, имеющим любую из заданных ролей, будет предоставлен доступ:

```python
@app.route('/admin')
@auth.login_required(role=['admin', 'moderator'])
def admins_only():
    return "Hello {}, you are an admin or a moderator!".format(auth.current_user())
```

В самом сложном случае пользователи могут быть отфильтрованы по нескольким ролям:

```python
@app.route('/admin')
@auth.login_required(role=['user', ['moderator', 'contributor']])
def admins_only():
    return "Hello {}, you are a user or a moderator/contributor!".format(auth.current_user())
```

## Рекомендации по развертыванию

Имейте в виду, что некоторые веб-серверы по умолчанию не передают заголовки авторизации приложению **WSGI**. Например, если вы используете **Apache** с **mod\_wsgi**, вы должны установить параметр `WSGIPassAuthorization On`, как [описано здесь](https://code.google.com/p/modwsgi/wiki/ConfigurationDirectives#WSGIPassAuthorization/).

## Устаревшие основные параметры аутентификации

До появления описанного выше **verify\_password** существовали другие более простые механизмы для реализации базовой аутентификации. Хотя они устарели, они все еще поддерживаются. Однако следует отдать предпочтение обратному вызову **verify\_password**, поскольку он обеспечивает большую безопасность и гибкость.

Обратный вызов **get\_password** должен возвращать пароль, связанный с именем пользователя, указанным в качестве аргумента. **Flask-HTTPAuth** разрешит доступ, только если `get_password(username) == password`. Пример:

```python
@auth.get_password
def get_password(username):
    return get_password_for_username(username)
```

Использование только этого обратного вызова, как правило, не является хорошей идеей, поскольку для этого требуется, чтобы пароли были доступны в виде открытого текста на сервере. В более вероятном сценарии, когда пароли хранятся хешированными в базе данных пользователей, необходим дополнительный обратный вызов, чтобы определить, как хешировать пароль:

```python
@auth.hash_password
def hash_pw(password):
    return hash_password(password)
```

В этом примере вам нужно заменить **hash\_password()** конкретной функцией хеширования, используемой в вашем приложении. Когда предоставляется обратный вызов **hash\_password**, доступ будет предоставлен, когда `get_password(username) == hash_password(password)`.

Если алгоритм хеширования требует, чтобы имя пользователя было известно, то обратный вызов может принимать два аргумента вместо одного:

```python
@auth.hash_password
def hash_pw(username, password):
    salt = get_salt(username)
    return hash_password(password, salt)
```

## Документация API

## HTTPBasicAuth

## HTTPDigestAuth

## HTTPTokenAuth

## HTTPMultiAuth
