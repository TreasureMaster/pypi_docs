# Фабрики приложения Flask

Если вы уже используете пакеты и _**blueprints**_ для своего приложения ([модульные приложения с Blueprints](../rukovodstvo-polzovatelya-flask/modulnye-prilozheniya-s-blueprints.md)), есть несколько действительно хороших способов дальнейшего улучшения опыта. Распространенным паттерном является создание объекта приложения при импорте _**blueprint**_. Но если вы переместите создание этого объекта в функцию, вы сможете позже создать несколько экземпляров этого приложения.

Так зачем вам это делать?

1. Тестирование. У вас могут быть экземпляры приложения с разными настройками для проверки каждого случая.
2. Несколько экземпляров. Представьте, что вы хотите запустить разные версии одного и того же приложения. Конечно, у вас может быть несколько экземпляров с разными конфигурациями, настроенными на вашем веб-сервере, но если вы используете фабрики, у вас может быть несколько экземпляров одного и того же приложения, запущенного в одном процессе приложения, что может быть удобно.

Итак, как бы вы тогда это реализовали?

## Базовые фабрики

Идея состоит в том, чтобы настроить приложение в функции. Так:

```python
def create_app(config_filename):
    app = Flask(__name__)
    app.config.from_pyfile(config_filename)

    from yourapplication.model import db
    db.init_app(app)

    from yourapplication.views.admin import admin
    from yourapplication.views.frontend import frontend
    app.register_blueprint(admin)
    app.register_blueprint(frontend)

    return app
```

Обратной стороной является то, что вы не можете использовать объект приложения в _**blueprints**_ во время импорта. Однако вы можете использовать его из запроса. Как получить доступ к приложению с **config**? Используйте [current\_app](../api-dokumentaciya-flask/poleznye-funkcii-i-klassy-flask.md#flask-current\_app):

```python
from flask import current_app, Blueprint, render_template
admin = Blueprint('admin', __name__, url_prefix='/admin')

@admin.route('/')
def index():
    return render_template(current_app.config['INDEX_TEMPLATE'])
```

Здесь мы ищем имя шаблона в **config**.

## Фабрики и расширения

Желательно создавать свои расширения и фабрики приложений, чтобы объект расширения изначально не привязывался к приложению.

Используя, например, [Flask-SQLAlchemy](https://flask-sqlalchemy.palletsprojects.com/en/2.x/), вы не должны делать что-то в этом роде:

```python
def create_app(config_filename):
    app = Flask(__name__)
    app.config.from_pyfile(config_filename)

    db = SQLAlchemy(app)
```

Но, скорее, в `model.py` (или аналоге):

```python
db = SQLAlchemy()
```

и в вашем `application.py` (или эквивалентном):

```python
def create_app(config_filename):
    app = Flask(__name__)
    app.config.from_pyfile(config_filename)

    from yourapplication.model import db
    db.init_app(app)
```

При использовании этого шаблона проектирования в объекте расширения не сохраняется состояние, зависящее от приложения, поэтому один объект расширения может использоваться для нескольких приложений. Для получения дополнительной информации о дизайне расширений обратитесь к [Flask Extension Development](../dopolnitelnye-primechaniya-flask/razrabotka-rasshirenii-flask.md).

## Использование приложений

Чтобы запустить такое приложение, вы можете использовать команду **flask**:

```bash
$ export FLASK_APP=myapp
$ flask run
```

**Flask** автоматически обнаружит фабрику (**create\_app** или **make\_app**) в **myapp**. Вы также можете передать аргументы фабрике следующим образом:

```bash
$ export FLASK_APP="myapp:create_app('dev')"
$ flask run
```

Затем в **myapp** вызывается фабрика **create\_app** со строкой `«dev»` в качестве аргумента. Подробнее см. [Интерфейс командной строки](../rukovodstvo-polzovatelya-flask/interfeis-komandnoi-stroki-flask.md).

## Улучшения фабрики

Приведенная выше функция фабрика не очень умна, но вы можете ее улучшить. Следующие изменения легко реализовать:

1. Обеспечьте возможность передачи значений конфигурации для модульных тестов, чтобы вам не приходилось создавать файлы конфигурации в файловой системе.
2. Вызов функции из _**blueprint**_ при настройке приложения, чтобы у вас было место для изменения атрибутов приложения (например, подключение обработчиков запросов до / после и т. д.)
3. При необходимости добавьте промежуточное ПО WSGI при создании приложения.
