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

#### _class_ flask\_httpauth.HTTPBasicAuth

Этот класс обрабатывает базовую аутентификацию HTTP для маршрутов Flask.

### \_\_init\_\_(_scheme=None_, _realm=None_)

Создает базовый объект аутентификации.

Если указан необязательный аргумент схемы, он будет использоваться вместо стандартной `«Basic»` схемы в ответе **WWW-Authenticate**. Довольно распространенной практикой является использование пользовательской схемы, чтобы браузеры не предлагали пользователю войти в систему.

Аргумент **realm** можно использовать для предоставления области, определенной приложением, с заголовком **WWW-Authenticate**.

### verify\_password(_verify\_password\_callback_)

Если определено, эта функция обратного вызова будет вызываться платформой для проверки правильности комбинации имени пользователя и пароля, предоставленной клиентом. Функция обратного вызова принимает два аргумента: имя пользователя **username** и пароль **password**. Он должен возвращать пользовательский объект, если учетные данные действительны, или `True`, если пользовательский объект недоступен. В случае неудачной аутентификации он должен возвращать `None` или `False`. Пример использования:

```python
@auth.verify_password
def verify_password(username, password):
    user = User.query.filter_by(username).first()
    if user and passlib.hash.sha256_crypt.verify(password, user.password_hash):
        return user
```

Если этот обратный вызов определен, он также вызывается, когда запрос не имеет заголовка **Authorization** с учетными данными пользователя, и в этом случае аргументы имени пользователя **username** и пароля **password** устанавливаются в пустые строки. В этом случае приложение может выбрать возврат `True`, что позволит _**анонимным пользователям**_ получить доступ к маршруту. Функция обратного вызова может указать, что пользователь является _**анонимным**_, записав переменную состояния в `flask.g` или проверив, имеет ли **auth.current\_user()** значение `None`.

Обратите внимание, что когда предоставляется обратный вызов **verify\_password**, обратные вызовы **get\_password** и **hash\_password** не используются.

### get\_user\_roles(_roles\_callback_)

Если определено, эта функция обратного вызова будет вызываться платформой для получения ролей, назначенных данному пользователю. Функция обратного вызова принимает один аргумент — пользователя **user**, для которого запрашиваются роли. Пользовательский объект, переданный этой функции, будет возвращаться функцией **verify\_callback**. Функция должна возвращать роль или список ролей, принадлежащих пользователю. Пример:

```python
@auth.get_user_roles
def get_user_roles(user):
    return user.get_roles()
```

### get\_password(_password\_callback_)

_<mark style="color:orange;">Устарело</mark>_. Эта функция обратного вызова будет вызываться платформой для получения пароля для данного пользователя. Пример:

```python
@auth.get_password
def get_password(username):
    return db.get_user_password(username)
```

### hash\_password(_hash\_password\_callback_)

_<mark style="color:orange;">Устарело</mark>_. Если определено, эта функция обратного вызова будет вызываться платформой для применения пользовательского алгоритма хеширования к паролю, предоставленному клиентом. Если этот обратный вызов не предоставлен, пароль будет проверен без изменений. Обратный вызов может принимать один или два аргумента. Версия с одним аргументом получает пароль **password** для хэширования, а версия с двумя аргументами получает имя пользователя **username** и пароль **password** в указанном порядке. Пример обратного вызова с одним аргументом:

```python
@auth.hash_password
def hash_password(password):
    return md5(password).hexdigest()
```

Пример обратного вызова с двумя аргументами:

```python
@auth.hash_password
def hash_pw(username, password):
    salt = get_salt(username)
    return hash(password, salt)
```

### error\_handler(_error\_callback_)

Если определено, эта функция обратного вызова будет вызываться платформой, когда необходимо отправить ошибку аутентификации обратно клиенту. Функция может принимать один аргумент, код состояния ошибки (**status**), который может быть **401** (неправильные учетные данные) или **403** (правильные, но недостаточные учетные данные). Чтобы сохранить совместимость со старыми версиями этого пакета, функцию также можно определить без аргументов. Возвращаемое значение этой функции должно соответствовать любому принятому типу ответа в маршрутах Flask. Если этот обратный вызов не предоставлен, генерируется ответ об ошибке по умолчанию. Пример:

```python
@auth.error_handler
def auth_error(status):
    return "Access Denied", status
```

### login\_required(_view\_function\_callback_)

Эта функция обратного вызова будет вызываться при успешной аутентификации. Обычно это функция представления Flask. Пример:

```python
@app.route('/private')
@auth.login_required
def private_page():
    return "Only for authorized people!"
```

Необязательный аргумент роли **role** может быть задан для дальнейшего ограничения доступа по ролям. Пример:

```python
@app.route('/private')
@auth.login_required(role='admin')
def private_page():
    return "Only for admins!"
```

Необязательный аргумент **optional** может быть установлен в значение `True`, чтобы позволить маршруту выполняться также, когда аутентификация не включена в запрос, и в этом случае для `auth.current_user()` будет установлено значение `None`. Пример:

```python
@app.route('/private')
@auth.login_required(optional=True)
def private_page():
    user = auth.current_user()
    return "Hello {}!".format(user.name if user is not None else 'anonymous')
```

### current\_user()

Объект пользователя **user**, возвращаемый обратным вызовом **verify\_password** при успешной аутентификации. Если обратный вызов не возвращает ни одного пользователя, это устанавливается на имя пользователя **username**, переданное клиентом. Пример:

```python
@app.route('/')
@auth.login_required
def index():
    user = auth.current_user()
    return "Hello, {}!".format(user.name)
```

### username()

_<mark style="color:orange;">Устарело</mark>_. Функция представления, защищенная с помощью этого класса, может получить доступ к зарегистрированному имени пользователя с помощью этого метода. Пример:

```python
@app.route('/')
@auth.login_required
def index():
    return "Hello, {}!".format(auth.username())
```

## HTTPDigestAuth

## HTTPTokenAuth

## HTTPMultiAuth
