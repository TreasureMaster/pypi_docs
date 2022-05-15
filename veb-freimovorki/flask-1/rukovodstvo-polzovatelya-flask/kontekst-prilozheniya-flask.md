# Контекст приложения Flask

Контекст приложения отслеживает данные уровня приложения во время запроса, команды CLI или других действий. Вместо того, чтобы передавать приложение каждой функции, вместо этого осуществляется доступ к прокси [current\_app](../api-dokumentaciya-flask/poleznye-funkcii-i-klassy-flask.md#flask-current\_app) и [g](../api-dokumentaciya-flask/globalnyi-obekt-prilozheniya-flask.md#flask-g).

Это похоже на [контекст запроса Request](kontekst-zaprosa-flask.md), который отслеживает данные уровня запроса во время запроса. Соответствующий контекст приложения передается при передаче контекста запроса.

## Цель контекста

Объект приложения [Flask](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#klass-flask-flask) имеет атрибуты, такие как [config](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#config), которые полезны для доступа в представлениях и [командах CLI](interfeis-komandnoi-stroki-flask.md). Однако при импорте экземпляра приложения **app** в модули вашего проекта часто возникают проблемы с циклическим импортом. При использовании [паттерна фабрики приложений](../patterny-flask/#fabriki-prilozheniya) или написании [многоразовых схем blueprints](modulnye-prilozheniya-s-blueprints.md) или [расширений](rasshireniya-flask.md) экземпляра приложения **app** импорта вообще не будет.

**Flask** решает эту проблему с помощью **контекста приложения**. Вместо того, чтобы ссылаться на приложение **app** напрямую, вы используете прокси [current\_app](../api-dokumentaciya-flask/poleznye-funkcii-i-klassy-flask.md#flask-current\_app), который указывает на приложение, обрабатывающее текущую активность.

**Flask** автоматически подталкивает (**pushes**) контекст приложения при обработке запроса. Функции просмотра, обработчики ошибок и другие функции, которые выполняются во время запроса, будут иметь доступ к [current\_app](../api-dokumentaciya-flask/poleznye-funkcii-i-klassy-flask.md#flask-current\_app).

**Flask** также автоматически подталкивает контекст приложения при запуске команд CLI, зарегистрированных в `Flask.cli`, с помощью `@app.cli.command()`.

## Время жизни контекста

Контекст приложения создается и уничтожается по мере необходимости. Когда приложение **Flask** начинает обрабатывать запрос, оно подталкивает контекст приложения и [контекст запроса](kontekst-zaprosa-flask.md). Когда запрос завершается, появляется контекст запроса, а затем контекст приложения. Обычно у контекста приложения такое же время жизни, как у запроса.

См. «[Контекст запроса](kontekst-zaprosa-flask.md)» для получения дополнительной информации о том, как работают контексты и полный жизненный цикл запроса.

## Вставить контекст вручную

Если вы попытаетесь получить доступ к [current\_app](../api-dokumentaciya-flask/poleznye-funkcii-i-klassy-flask.md#flask-current\_app) или чему-либо, что его использует вне контекста приложения, вы получите следующее сообщение об ошибке:

```bash
RuntimeError: Working outside of application context.

Обычно это означает, что вы пытались использовать функции, которым
необходимо каким-то образом взаимодействовать с текущим объектом приложения.
Чтобы решить эту проблему, настройте контекст приложения с помощью app.app_context().
```

Если вы видите эту ошибку при настройке приложения, например, при инициализации расширения, вы можете отправить контекст вручную, поскольку у вас есть прямой доступ к приложению **app**. Используйте [app\_context ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#app\_context) в блоке **with**, и все, что выполняется в блоке, будет иметь доступ к [current\_app](../api-dokumentaciya-flask/poleznye-funkcii-i-klassy-flask.md#flask-current\_app).

```python
def create_app():
    app = Flask(__name__)

    with app.app_context():
        init_db()

    return app
```

Если вы видите эту ошибку где-то еще в вашем коде, не связанном с настройкой приложения, это, скорее всего, означает, что вам следует переместить этот код в функцию просмотра или команду CLI.

## Хранение данных

Контекст приложения - хорошее место для хранения общих данных во время запроса или команды CLI. **Flask** предоставляет для этой цели данный [объект g](../api-dokumentaciya-flask/globalnyi-obekt-prilozheniya-flask.md#flask-g). Это простой объект пространства имен, имеющий то же время жизни, что и контекст приложения.

{% hint style="info" %}
Имя **g** означает «глобальный», но это относится к данным, являющимся глобальными **в контексте**. Данные **g** теряются после завершения контекста, и это не подходящее место для хранения данных между запросами. Используйте [session](../api-dokumentaciya-flask/sessii-flask.md#klass-flask-session) или базу данных для хранения данных между запросами.
{% endhint %}

Обычно [g](../api-dokumentaciya-flask/globalnyi-obekt-prilozheniya-flask.md#flask-g) используется для управления ресурсами во время запроса.

1. `get_X ()` создает ресурс **X**, если он не существует, кэшируя его как `g.X`.
2. `teardown_X ()` закрывает или иным образом освобождает ресурс, если он существует. Он зарегистрирован как обработчик [teardown\_appcontext ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#teardown\_appcontext).

Например, вы можете управлять подключением к базе данных, используя этот шаблон:

```python
from flask import g

def get_db():
    if 'db' not in g:
        g.db = connect_to_database()

    return g.db

@app.teardown_appcontext
def teardown_db(exception):
    db = g.pop('db', None)

    if db is not None:
        db.close()
```

Во время запроса каждый вызов `get_db ()` будет возвращать одно и то же соединение, и оно будет автоматически закрыто в конце запроса.

Вы можете использовать [LocalProxy](https://werkzeug.palletsprojects.com/en/1.0.x/local/#werkzeug.local.LocalProxy), чтобы сделать новый контекст локальным из `get_db ()`:

```python
from werkzeug.local import LocalProxy
db = LocalProxy(get_db)
```

Доступ к **db** вызовет **get\_db** внутренне, так же, как работает [current\_app](../api-dokumentaciya-flask/poleznye-funkcii-i-klassy-flask.md#flask-current\_app).

Если вы пишете расширение, [g](../api-dokumentaciya-flask/globalnyi-obekt-prilozheniya-flask.md#flask-g) следует зарезервировать для пользовательского кода. Вы можете хранить внутренние данные в самом контексте, но обязательно используйте достаточно уникальное имя. Доступ к текущему контексту осуществляется с помощью [\_app\_ctx\_stack.top](../api-dokumentaciya-flask/poleznye-funkcii-flask.md#flask-\_app\_ctx\_stack). Для получения дополнительной информации см. [Разработка расширений Flask](../dopolnitelnye-primechaniya-flask/razrabotka-rasshirenii-flask.md).

## События и сигналы

Приложение будет вызывать функции, зарегистрированные с помощью [teardown\_appcontext ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#teardown\_appcontext), когда контекст приложения открывается.

Если [signal\_available](../api-dokumentaciya-flask/signaly-flask.md#signals-signals\_available) истинно, отправляются следующие сигналы: [appcontext\_pushing](../api-dokumentaciya-flask/signaly-flask.md#flask-appcontext\_pushed), [appcontext\_tearing\_down](../api-dokumentaciya-flask/signaly-flask.md#flask-appcontext\_tearing\_down) и [appcontext\_popped](../api-dokumentaciya-flask/signaly-flask.md#flask-appcontext\_popped).
