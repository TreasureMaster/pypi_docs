# Flask-Migrate

**Flask-Migrate** — это расширение, которое обрабатывает миграцию базы данных SQLAlchemy для приложений **Flask** с помощью **Alembic**. Операции с базой данных доступны через интерфейс командной строки **Flask**.

### Зачем использовать Flask-Migrate вместо Alembic напрямую?

**Flask-Migrate** — это расширение, которое правильно настраивает **Alembic** для работы с вашим приложением **Flask** и **Flask-SQLAlchemy**. Что касается фактической миграции базы данных, **Alembic** выполняет все операции, поэтому вы получаете точно такую же функциональность.

### Установка

Установите **Flask-Migrate** с помощью **pip**:

```bash
pip install Flask-Migrate
```

### Пример

Это пример приложения, которое обрабатывает миграцию базы данных через **Flask-Migrate**:

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///app.db'

db = SQLAlchemy(app)
migrate = Migrate(app, db)

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(128))
```

С помощью вышеуказанного приложения вы можете создать репозиторий миграции с помощью следующей команды:

```bash
$ flask db init
```

Это добавит папку _**migrations**_ в ваше приложение. Содержимое этой папки необходимо добавить в систему контроля версий вместе с другими вашими исходными файлами.

Затем вы можете создать первоначальную миграцию:

```bash
$ flask db migrate -m "Initial migration."
```

Сценарий переноса необходимо проверить и отредактировать, так как в настоящее время **Alembic** не обнаруживает каждое изменение, которое вы вносите в свои модели. В частности, в настоящее время **Alembic** не может обнаруживать изменения имени таблицы, изменения имени столбца или ограничения с анонимными именами. Подробное описание ограничений можно найти в [документации по автогенерации Alembic](http://alembic.zzzcomputing.com/en/latest/autogenerate.html#what-does-autogenerate-detect-and-what-does-it-not-detect). После завершения сценарий миграции также необходимо добавить в систему контроля версий.

Затем вы можете применить миграцию к базе данных:

```bash
$ flask db upgrade
```

Затем каждый раз при изменении моделей баз данных повторяйте команды **migrate** и **upgrade**.

Чтобы синхронизировать базу данных в другой системе, просто обновите папку _**migrations**_ из системы управления версиями и выполните команду **upgrade**.

Чтобы просмотреть все доступные команды, выполните следующую команду:

```bash
$ flask db --help
```

Обратите внимание, что сценарий приложения должен быть установлен в переменной среды **FLASK\_APP**, чтобы все вышеперечисленные команды работали, как того требует сценарий командной строки **flask**.

### Обратные вызовы конфигурации

Иногда приложениям необходимо динамически вставлять свои настройки в конфигурацию **Alembic**. Функция, декорированная обратным вызовом **configure**, будет вызываться после чтения конфигурации и до ее использования. Функция может изменить объект конфигурации или заменить его другим.

```python
@migrate.configure
def configure_alembic(config):
    # изменить объект конфигурации
    return config
```

Несколько обратных вызовов конфигурации можно определить, просто декорировав несколько функций. _**Порядок, в котором вызываются несколько обратных вызовов, не определен.**_

### Поддержка нескольких баз данных

**Flask-Migrate** может интегрироваться с функцией [binds](http://flask-sqlalchemy.pocoo.org/binds/) **Flask-SQLAlchemy**, что позволяет отслеживать миграции в несколько баз данных, связанных с приложением.

Чтобы создать репозиторий миграции с несколькими базами данных, добавьте в команду инициализации аргумент `--multidb` :

```bash
$ flask db init --multidb
```

С помощью этой команды репозиторий миграции будет настроен для отслеживания миграции в вашей основной базе данных и в любых дополнительных базах данных, определенных в параметре конфигурации **SQLALCHEMY\_BINDS**.

### Справочник по командам

**Flask-Migrate** предоставляет один класс под названием **Migrate**. Этот класс содержит всю функциональность расширения.

В следующем примере расширение инициализируется стандартным интерфейсом командной строки **Flask**:

```python
from flask_migrate import Migrate
migrate = Migrate(app, db)
```

Двумя аргументами **Migrate** являются экземпляр приложения и экземпляр базы данных **Flask-SQLAlchemy**. Конструктор **Migrate** также принимает дополнительные ключевые аргументы, которые передаются в метод **Alembic** `EnvironmentContext.configure()`. Стандартно для всех расширений **Flask**, **Flask-Migrate** также можно инициализировать с помощью метода **init\_app**:

```python
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate

db = SQLAlchemy()
migrate = Migrate()

def create_app():
     """Application-factory pattern"""
     ...
     ...
     db.init_app(app)
     migrate.init_app(app, db)
     ...
     ...
     return app
```

После инициализации расширения к параметрам командной строки будет добавлена группа **db** с несколькими подкомандами. Ниже приведен список доступных подкоманд:

* `flask db --help` - Показывает список доступных команд.
* `flask db list-templates` - Показывает список доступных шаблонов репозитория базы данных.
* `flask db init [--multidb] [--template TEMPLATE] [--package]` - Инициализирует поддержку миграции для приложения. Необязательный параметр `--multidb` включает миграцию для нескольких баз данных, настроенных как привязки [Flask-SQLAlchemy binds](http://flask-sqlalchemy.pocoo.org/binds/). Параметр `--template` позволяет вам явно выбрать шаблон репозитория базы данных либо из стандартных шаблонов, предоставляемых этим пакетом, либо из пользовательского шаблона, заданного как путь к каталогу шаблона. Параметр `--package` указывает **Alembic** добавить файлы `__init__.py` в каталоги миграции и версии.
* `flask db revision [--message MESSAGE] [--autogenerate] [--sql] [--head HEAD] [--splice] [--branch-label BRANCH_LABEL] [--version-path VERSION_PATH] [--rev-id REV_ID]` - Создает пустой скрипт ревизии. Сценарий нужно редактировать _**вручную**_ с изменениями **upgrade** и **downgrade**. Инструкции по написанию сценариев миграции см. в [документации Alembic](http://alembic.zzzcomputing.com/en/latest/index.html). Может быть включено необязательное сообщение **message** о миграции.
* `flask db migrate [--message MESSAGE] [--sql] [--head HEAD] [--splice] [--branch-label BRANCH_LABEL] [--version-path VERSION_PATH] [--rev-id REV_ID]` - Эквивалент `revision --autogenerate`. Сценарий миграции заполняется изменениями, обнаруженными автоматически. Сгенерированный сценарий необходимо просмотреть и отредактировать, так как не все типы изменений могут быть обнаружены автоматически. Эта команда не вносит никаких изменений в базу данных, а просто создает сценарий ревизии.
* `flask db edit <revision>` - Редактирует сценарий ревизии с помощью **$EDITOR**.
* `flask db upgrade [--sql] [--tag TAG] [--x-arg ARG] <revision>` - Обновляет базу данных. Если **revision** не указан, предполагается `"head"`.
* `flask db downgrade [--sql] [--tag TAG] [--x-arg ARG] <revision>` - Понижает базу данных. Если **revision** не указан, предполагается `-1`.
* `flask db stamp [--sql] [--tag TAG] <revision>` - Устанавливает ревизию в базе данных на ту, которая указана в качестве аргумента, без выполнения каких-либо миграций.
* `flask db current [--verbose]` - Показывает текущую версию базы данных.
* `flask db history [--rev-range REV_RANGE] [--verbose]` - Показывает список миграций. Если диапазон не указан, отображается вся история.
* `flask db show <revision>` - Показать ревизию, обозначенную данным символом.
* `flask db merge [--message MESSAGE] [--branch-label BRANCH_LABEL] [--rev-id REV_ID] <revisions>` - Объедините две ревизии вместе. Создает новый файл ревизии.
* `flask db heads [--verbose] [--resolve-dependencies]` - Показать текущие доступные головы **heads** в каталоге сценария ревизии.
* `flask db branches [--verbose]` - Показать текущие точки ветвления.

#### Примечания:

* Все команды также принимают параметр `--directory DIRECTORY`, указывающий на каталог, содержащий сценарии миграции. Если этот аргумент опущен, используется каталог **migrations**.
* Каталог по умолчанию также можно указать в качестве аргумента **directory** для конструктора **Migrate**.
* Параметр `--sql`, присутствующий в нескольких командах, выполняет миграцию в автономном (_offline_) режиме. Вместо выполнения команд базы данных операторы SQL, которые необходимо выполнить, они выводятся на консоль.
* Подробную документацию по этим командам можно найти на [странице справочника по командам Alembic](http://alembic.zzzcomputing.com/en/latest/api/commands.html).

### Справочник API

Доступ к командам, предоставляемым интерфейсом командной строки **Flask-Migrate**, также можно получить программно, импортировав функции из модуля **flask\_migrate**. Доступные функции:

* `init(directory='migrations', multidb=False)` - Инициализирует поддержку миграции для приложения.
* `revision(directory='migrations', message=None, autogenerate=False, sql=False, head='head', splice=False, branch_label=None, version_path=None, rev_id=None)` - Создает пустой скрипт ревизии.
* `migrate(directory='migrations', message=None, sql=False, head='head', splice=False, branch_label=None, version_path=None, rev_id=None)` - Создает сценарий автоматической ревизии.
* `edit(directory='migrations', revision='head')` - Отредактируйте скрипт(ы) редакции с помощью **$EDITOR**.
* `merge(directory='migrations', revisions='', message=None, branch_label=None, rev_id=None)` - Объединяет две ревизии вместе. Создает новый файл миграции.
* `upgrade(directory='migrations', revision='head', sql=False, tag=None)` - Обновляет базу данных.
* `downgrade(directory='migrations', revision='-1', sql=False, tag=None)` - Понижает базу данных.
* `show(directory='migrations', revision='head')` - Показать ревизию, обозначенную данным символом.
* `history(directory='migrations', rev_range=None, verbose=False)` - Показывает список миграций. Если диапазон не указан, отображается вся история.
* `heads(directory='migrations', verbose=False, resolve_dependencies=False)` - Показать текущие доступные головы **heads** в каталоге скриптов.
* `branches(directory='migrations', verbose=False)` - Показать текущие точки ветвления
* `current(directory='migrations', verbose=False, head_only=False)` - Показывает текущую версию базы данных.
* `stamp(directory='migrations', revision='head', sql=False, tag=None)` - Устанавливает ревизию в базе данных на ту, которая указана в качестве аргумента, без выполнения каких-либо миграций.

#### Примечания:

* Эти команды будут вызывать те же функции, что и из командной строки, включая вывод на терминал. Конфигурация регистрации процесса будет переопределена **Alembic** в соответствии с содержимым файла `alembic.ini`.
* Для большей гибкости сценариев вы также можете напрямую использовать API, предоставленный **Alembic**.
