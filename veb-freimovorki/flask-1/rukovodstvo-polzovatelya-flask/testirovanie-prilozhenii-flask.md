# Тестирование приложений Flask

&#x20;"**Something that is untested is broken."**

Происхождение этой цитаты неизвестно, и хотя она не совсем верна, она также недалеко от истины. Непроверенные приложения затрудняют улучшение существующего кода, а разработчики непроверенных приложений обычно становятся параноиками. Если в приложении есть автоматизированные тесты, вы можете безопасно вносить изменения и сразу узнавать, что что-то сломается.

**Flask** предоставляет способ тестирования вашего приложения, открывая тестовый клиент **Werkzeug** [Client](https://werkzeug.palletsprojects.com/en/1.0.x/test/#werkzeug.test.Client) и обрабатывая локальные переменные контекста за вас. Затем вы можете использовать это с вашим любимым тестовым решением.

В этой документации мы будем использовать пакет [pytest](https://docs.pytest.org/en/stable/) в качестве базовой структуры для наших тестов. Вы можете установить его с помощью **pip**, например:

```bash
$ pip install pytest
```

## Приложение

Во-первых, нам нужно приложение для тестирования; мы будем использовать приложение из [руководства](uchebnik-flask.md). Если у вас еще нет этого приложения, возьмите исходный код из [примеров](https://github.com/pallets/flask/tree/1.1.2/examples/tutorial).

## Каркас тестирования

Начнем с добавления каталога тестов в корень приложения. Затем создайте файл Python для хранения наших тестов (`test_flaskr.py`). Когда мы форматируем имя файла как `test_*.py`, оно будет автоматически обнаружено **pytest**.

Затем мы создаем [фикстуру pytest](https://docs.pytest.org/en/latest/fixture.html) с именем **client ()**, который настраивает приложение для тестирования и инициализирует новую базу данных:

```python
import os
import tempfile

import pytest

from flaskr import flaskr


@pytest.fixture
def client():
    db_fd, flaskr.app.config['DATABASE'] = tempfile.mkstemp()
    flaskr.app.config['TESTING'] = True

    with flaskr.app.test_client() as client:
        with flaskr.app.app_context():
            flaskr.init_db()
        yield client

    os.close(db_fd)
    os.unlink(flaskr.app.config['DATABASE'])
```

Эта фикстура **client** будет вызываться в каждом отдельном тесте. Это дает нам простой интерфейс к приложению, где мы можем запускать тестовые запросы к приложению. Клиент также будет отслеживать файлы cookie для нас.

Во время настройки активируется флаг конфигурации **TESTING**. Это отключает перехват ошибок во время обработки запроса, чтобы вы могли лучше получать отчеты об ошибках при выполнении тестовых запросов к приложению.

Поскольку **SQLite3** основан на файловой системе, мы можем легко использовать модуль [tempfile](https://docs.python.org/3/library/tempfile.html#module-tempfile) для создания временной базы данных и ее инициализации. Функция [mkstemp ()](https://docs.python.org/3/library/tempfile.html#tempfile.mkstemp) делает для нас две вещи: возвращает дескриптор файла низкого уровня и случайное имя файла, последнее мы используем в качестве имени базы данных. Нам просто нужно сохранить _**db\_fd**_, чтобы мы могли использовать функцию [os.close ()](https://docs.python.org/3/library/os.html#os.close) для закрытия файла.

Чтобы удалить базу данных после теста, фикстура закрывает файл и удаляет его из файловой системы.

Если мы сейчас запустим набор тестов, мы должны увидеть следующий результат:

```bash
$ pytest

================ test session starts ================
rootdir: ./flask/examples/flaskr, inifile: setup.cfg
collected 0 items

=========== no tests ran in 0.07 seconds ============
```

Несмотря на то, что оно не проводило никаких реальных тестов, мы уже знаем, что наше приложение **flaskr** синтаксически корректно, иначе импорт бы возбудил исключение.

## Первый тест

Пришло время приступить к тестированию функциональности приложения. Давайте проверим, что приложение показывает «Здесь пока нет записей», если мы обращаемся к корню приложения (`/`). Для этого мы добавляем новую тестовую функцию в `test_flaskr.py`, например:

```bash
def test_empty_db(client):
    """Начните с пустой базы данных."""

    rv = client.get('/')
    assert b'No entries here so far' in rv.data
```

Обратите внимание, что наши тестовые функции начинаются со слова **test**; это позволяет **pytest** автоматически идентифицировать функцию как запускаемый тест.

Используя `client.get`, мы можем отправить HTTP-запрос **GET** в приложение с заданным путем. Возвращаемое значение будет объектом [**response\_class**](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#response\_class). Теперь мы можем использовать атрибут **data** для проверки возвращаемого значения (в виде строки) из приложения. В этом случае мы гарантируем, что `«No entries here so far»` является частью вывода.

Запустите его снова, и вы должны увидеть один проходящий тест:

```bash
$ pytest -v

================ test session starts ================
rootdir: ./flask/examples/flaskr, inifile: setup.cfg
collected 1 items

tests/test_flaskr.py::test_empty_db PASSED

============= 1 passed in 0.10 seconds ==============
```

## Вход и выход

Большая часть функций нашего приложения доступна только для администратора, поэтому нам нужен способ входа и выхода нашего тестового клиента в приложение. Для этого мы запускаем несколько запросов на страницы входа и выхода с необходимыми данными формы (имя пользователя и пароль). И поскольку страницы входа и выхода перенаправляют, мы говорим клиенту **follow\_redirects**.

Добавьте в файл `test_flaskr.py` следующие две функции:

```python
def login(client, username, password):
    return client.post('/login', data=dict(
        username=username,
        password=password
    ), follow_redirects=True)


def logout(client):
    return client.get('/logout', follow_redirects=True)
```

Теперь мы можем легко проверить, работает ли вход и выход и что он не работает с неверными учетными данными. Добавьте эту новую тестовую функцию:

```python
def test_login_logout(client):
    """Убедитесь, что вход и выход из системы работают."""

    rv = login(client, flaskr.app.config['USERNAME'], flaskr.app.config['PASSWORD'])
    assert b'You were logged in' in rv.data

    rv = logout(client)
    assert b'You were logged out' in rv.data

    rv = login(client, flaskr.app.config['USERNAME'] + 'x', flaskr.app.config['PASSWORD'])
    assert b'Invalid username' in rv.data

    rv = login(client, flaskr.app.config['USERNAME'], flaskr.app.config['PASSWORD'] + 'x')
    assert b'Invalid password' in rv.data
```

## Тестирование добавления сообщения

Мы также должны проверить, работает ли добавление сообщений. Добавьте новую тестовую функцию вроде этого:

```python
def test_messages(client):
    """Проверьте, работают ли сообщения."""

    login(client, flaskr.app.config['USERNAME'], flaskr.app.config['PASSWORD'])
    rv = client.post('/add', data=dict(
        title='<Hello>',
        text='<strong>HTML</strong> allowed here'
    ), follow_redirects=True)
    assert b'No entries here so far' not in rv.data
    assert b'&lt;Hello&gt;' in rv.data
    assert b'<strong>HTML</strong> allowed here' in rv.data
```

Здесь мы проверяем, разрешен ли HTML в тексте, но не в заголовке, что является предполагаемым поведением.

Выполнение этого должно дать нам три проходящих теста:

```bash
$ pytest -v

================ test session starts ================
rootdir: ./flask/examples/flaskr, inifile: setup.cfg
collected 3 items

tests/test_flaskr.py::test_empty_db PASSED
tests/test_flaskr.py::test_login_logout PASSED
tests/test_flaskr.py::test_messages PASSED

============= 3 passed in 0.23 seconds ==============
```

## Другие приемы тестирования

Помимо использования тестового клиента, как показано выше, существует также метод [test\_request\_context ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#test\_request\_context), который можно использовать в сочетании с оператором **with** для временной активации контекста запроса. С его помощью вы можете получить доступ к объектам [request](../api-dokumentaciya-flask/dannye-vkhodyashego-zaprosa-request.md#flask-request), [g](../api-dokumentaciya-flask/globalnyi-obekt-prilozheniya-flask.md#flask-g) и [session](../api-dokumentaciya-flask/sessii-flask.md#klass-flask-session), как в функциях просмотра. Вот полный пример, демонстрирующий этот подход:

```python
import flask

app = flask.Flask(__name__)

with app.test_request_context('/?name=Peter'):
    assert flask.request.path == '/'
    assert flask.request.args['name'] == 'Peter'
```

Все другие объекты, привязанные к контексту, могут использоваться таким же образом.

Если вы хотите протестировать свое приложение с различными конфигурациями, но, похоже, это не лучший способ сделать это, рассмотрите возможность перехода на фабрики приложений (см. [Фабрики приложений](../patterny-flask/#fabriki-prilozheniya)).

Однако обратите внимание, что если вы используете контекст тестового запроса, функции [before\_request ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#before\_request) и [after\_request ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#after\_request) не вызываются автоматически. Однако функции [teardown\_request ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#teardown\_request) действительно выполняются, когда контекст тестового запроса выходит из блока **with**. Если вы хотите, чтобы также вызывались функции [before\_request ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#before\_request), вам нужно вызвать [preprocess\_request ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#preprocessor\_request) самостоятельно:

```python
app = flask.Flask(__name__)

with app.test_request_context('/?name=Peter'):
    app.preprocess_request()
    ...
```

Это может быть необходимо для открытия соединений с базой данных или чего-то подобного в зависимости от того, как было разработано ваше приложение.

Если вы хотите вызвать функции [after\_request ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#after\_request), вам необходимо вызвать [process\_response ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#process\_response), который, однако, требует, чтобы вы передали ему объект ответа **Response**:

```python
app = flask.Flask(__name__)

with app.test_request_context('/?name=Peter'):
    resp = Response('...')
    resp = app.process_response(resp)
    ...
```

В общем, это менее полезно, потому что с этого момента вы можете напрямую начать использовать тестовый клиент.

## Поддельные ресурсы и контекст

_Новое в версии 0.10_.

Очень распространенный шаблон - хранить информацию об авторизации пользователя и соединениях с базой данных в контексте приложения или в объекте [flask.g](../api-dokumentaciya-flask/globalnyi-obekt-prilozheniya-flask.md#flask-g). Общий шаблон для этого - поместить объект туда при первом использовании, а затем удалить его при разборке. Представьте себе, например, этот код, чтобы получить текущего пользователя:

```python
def get_user():
    user = getattr(g, 'user', None)
    if user is None:
        user = fetch_current_user_from_database()
        g.user = user
    return user
```

Для теста было бы неплохо переопределить этого пользователя извне, не изменяя код. Это можно сделать, подключив сигнал [flask.appcontext\_pushed](../api-dokumentaciya-flask/signaly-flask.md#flask-appcontext\_pushed):

```python
from contextlib import contextmanager
from flask import appcontext_pushed, g

@contextmanager
def user_set(app, user):
    def handler(sender, **kwargs):
        g.user = user
    with appcontext_pushed.connected_to(handler, app):
        yield
```

А потом использовать:

```python
from flask import json, jsonify

@app.route('/users/me')
def users_me():
    return jsonify(username=g.user.username)

with user_set(app, my_user):
    with app.test_client() as c:
        resp = c.get('/users/me')
        data = json.loads(resp.data)
        self.assert_equal(data['username'], my_user.username)
```

## Сохранение окружающего контекста

_New in version 0.4_.

Иногда полезно инициировать регулярный запрос, но при этом сохранить контекст еще немного, чтобы можно было провести дополнительный самоанализ. В `Flask 0.4` это возможно с помощью [test\_client ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#test\_client) с блоком **with**:

```python
app = flask.Flask(__name__)

with app.test_client() as c:
    rv = c.get('/?tequila=42')
    assert request.args['tequila'] == '42'
```

Если бы вы использовали только [test\_client ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#test\_client) без блока **with**, утверждение завершилось бы ошибкой, потому что запрос больше не доступен (потому что вы пытаетесь использовать его вне фактического запроса).

## Доступ и изменение сеансов

_Новое в версии 0.8_.

Иногда может быть очень полезно получить доступ к сеансам или изменить их из тестового клиента. Обычно для этого есть два пути. Если вы просто хотите убедиться, что для определенных ключей в сеансе установлены определенные значения, вы можете просто сохранить контекст и получить доступ к [flask.session](../api-dokumentaciya-flask/sessii-flask.md#klass-flask-session):

```python
with app.test_client() as c:
    rv = c.get('/')
    assert flask.session['foo'] == 42
```

Однако это не позволяет также изменить сеанс или получить доступ к сеансу до того, как запрос был запущен. Начиная с `Flask 0.8`, мы предоставляем так называемую «транзакцию сеанса», которая имитирует соответствующие вызовы для открытия сеанса в контексте тестового клиента и его изменения. В конце транзакции сеанс сохраняется и готов к использованию тестовым клиентом. Это работает независимо от используемой серверной части сеанса:

```python
with app.test_client() as c:
    with c.session_transaction() as sess:
        sess['a_key'] = 'a value'

    # как только это будет достигнуто, сеанс был сохранен
    # и готов к использованию клиентом
    c.get(...)
```

Обратите внимание, что в этом случае вы должны использовать объект **sess** вместо прокси [flask.session](../api-dokumentaciya-flask/sessii-flask.md#klass-flask-session). Однако сам объект будет предоставлять тот же интерфейс.

## Тестирование API JSON

_Новое в версии 1.0_.

**Flask** отлично поддерживает **JSON** и является популярным выбором для создания API-интерфейсов **JSON**. Делать запросы с данными **JSON** и изучать данные **JSON** в ответах очень удобно:

```python
from flask import request, jsonify

@app.route('/api/auth')
def auth():
    json_data = request.get_json()
    email = json_data['email']
    password = json_data['password']
    return jsonify(token=generate_token(email, password))

with app.test_client() as c:
    rv = c.post('/api/auth', json={
        'email': 'flask@example.com', 'password': 'secret'
    })
    json_data = rv.get_json()
    assert verify_token(email, json_data['token'])
```

Передача аргумента _**json**_ в тестовых клиентских методах устанавливает данные запроса в сериализованный объект **JSON** и устанавливает тип содержимого как `application/json`. Вы можете получить данные **JSON** из запроса или ответа с помощью **get\_json**.

## Тестирование CLI команд

**Click** поставляется с [утилитами для тестирования](https://click.palletsprojects.com/en/7.x/testing/) ваших команд **CLI**. [CliRunner](https://click.palletsprojects.com/en/7.x/api/#click.testing.CliRunner) выполняет команды изолированно и фиксирует вывод в объекте [Result](https://click.palletsprojects.com/en/7.x/api/#click.testing.Result).

**Flask** предоставляет [test\_cli\_runner ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#test\_cli\_runner) для создания [FlaskCliRunner](../api-dokumentaciya-flask/testovye-client-i-cli-runner.md#klass-flask-testing-flaskclirunner), который автоматически передает приложение **Flask** в интерфейс командной строки. Используйте его метод [invoke ()](../api-dokumentaciya-flask/testovye-client-i-cli-runner.md#invoke) для вызова команд так же, как они вызывались бы из командной строки.

```python
import click

@app.cli.command('hello')
@click.option('--name', default='World')
def hello_command(name):
    click.echo(f'Hello, {name}!')

def test_hello():
    runner = app.test_cli_runner()

    # вызвать команду напрямую
    result = runner.invoke(hello_command, ['--name', 'Flask'])
    assert 'Hello, Flask' in result.output

    # или по имени
    result = runner.invoke(args=['hello'])
    assert 'World' in result.output
```

В приведенном выше примере вызов команды по имени полезен, поскольку он проверяет, правильно ли была зарегистрирована команда в приложении.

Если вы хотите проверить, как ваша команда анализирует параметры, не выполняя команду, используйте ее метод [make\_context ()](https://click.palletsprojects.com/en/7.x/api/#click.BaseCommand.make\_context). Это полезно для тестирования сложных правил проверки и настраиваемых типов.

```python
def upper(ctx, param, value):
    if value is not None:
        return value.upper()

@app.cli.command('hello')
@click.option('--name', default='World', callback=upper)
def hello_command(name):
    click.echo(f'Hello, {name}!')

def test_hello_params():
    context = hello_command.make_context('hello', ['--name', 'flask'])
    assert context.params['name'] == 'FLASK'
```
