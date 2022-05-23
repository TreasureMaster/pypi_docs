# Flask-Migrate

**Flask-Migrate** — это расширение, которое обрабатывает миграцию базы данных SQLAlchemy для приложений Flask с помощью **Alembic**. Операции с базой данных доступны через интерфейс командной строки Flask.

## Зачем использовать Flask-Migrate вместо Alembic напрямую?

**Flask-Migrate** — это расширение, которое правильно настраивает **Alembic** для работы с вашим приложением Flask и Flask-SQLAlchemy. Что касается фактической миграции базы данных, **Alembic** выполняет все операции, поэтому вы получаете точно такую же функциональность.

## Установка

Установите Flask-Migrate с помощью **pip**:

```bash
pip install Flask-Migrate
```

## Пример

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

Затем <mark style="color:orange;">**каждый раз при изменении моделей баз данных**</mark> повторяйте команды **migrate** и **upgrade**.

Чтобы <mark style="color:orange;">**синхронизировать базу данных в другой системе**</mark>, просто обновите папку _**migrations**_ из системы управления версиями и выполните команду **upgrade**.

Чтобы просмотреть все доступные команды, выполните следующую команду:

```bash
$ flask db --help
```

Обратите внимание, что сценарий приложения должен быть установлен в переменной среды **FLASK\_APP**, чтобы все вышеперечисленные команды работали, как того требует сценарий командной строки **flask**.

## Обратные вызовы конфигурации

Иногда приложениям необходимо динамически вставлять свои настройки в конфигурацию **Alembic**. Функция, декорированная обратным вызовом **configure**, будет вызываться после чтения конфигурации и до ее использования. Функция может изменить объект конфигурации или заменить его другим.

```python
@migrate.configure
def configure_alembic(config):
    # изменить объект конфигурации
    return config
```

Несколько обратных вызовов конфигурации можно определить, просто декорировав несколько функций. Порядок, в котором вызываются несколько обратных вызовов, не определен.

## Поддержка нескольких баз данных

**Flask-Migrate** может интегрироваться с функцией [binds](flask-sqlalchemy/rukovodstvo-polzovatelya-flask-sqlalchemy/neskolko-baz-dannykh-s-privyazkami.md) Flask-SQLAlchemy, что позволяет отслеживать миграции в несколько баз данных, связанных с приложением.

Чтобы создать репозиторий миграции с несколькими базами данных, нужно добавить аргумент `--multidb` в команду инициализации:

```bash
$ flask db init --multidb
```

С помощью этой команды репозиторий миграции будет настроен для отслеживания миграции в вашей основной базе данных и в любых дополнительных базах данных, определенных в параметре конфигурации **SQLALCHEMY\_BINDS**.

## Справочник по командам

**Flask-Migrate** предоставляет один класс под названием **Migrate**. Этот класс содержит всю функциональность расширения.

В следующем примере расширение инициализируется стандартным интерфейсом командной строки Flask:

```python
from flask_migrate import Migrate
migrate = Migrate(app, db)
```

Двумя аргументами **Migrate** являются экземпляр приложения и экземпляр базы данных **Flask-SQLAlchemy**. Конструктор **Migrate** также принимает дополнительные ключевые аргументы, которые передаются в метод Alembic `EnvironmentContext.configure()`. Стандартно для всех расширений Flask, **Flask-Migrate** также можно инициализировать с помощью метода **init\_app**:

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

### flask db --help

Показывает список доступных команд.

### flask db list-templates

Показывает список доступных шаблонов репозитория базы данных.

### flask db init

#### flask db init \[--multidb] \[--template TEMPLATE] \[--package]

Инициализирует поддержку миграции для приложения. Необязательный параметр **--multidb** включает миграцию для нескольких баз данных, настроенных как [привязки Flask-SQLAlchemy](flask-sqlalchemy/rukovodstvo-polzovatelya-flask-sqlalchemy/neskolko-baz-dannykh-s-privyazkami.md). Параметр **--template** позволяет вам _**явно выбрать шаблон**_ репозитория базы данных либо из стандартных шаблонов, предоставляемых этим пакетом, либо из пользовательского шаблона, заданного как путь к каталогу шаблона. Параметр **--package** указывает **Alembic** добавить файлы **\_\_init\_\_.py** в каталоги _**migrations**_ и _**versions**_.

### flask db revision

#### flask db revision \[--message MESSAGE] \[--autogenerate] \[--sql] \[--head HEAD] \[--splice] \[--branch-label BRANCH\_LABEL] \[--version-path VERSION\_PATH] \[--rev-id REV\_ID]

Создает пустой скрипт ревизии. Сценарий нужно редактировать вручную с изменениями обновления **upgrade** и понижения **downgrade**. Инструкции по написанию сценариев миграции см. в [документации Alembic](http://alembic.zzzcomputing.com/en/latest/index.html). Может быть включено необязательное сообщение о миграции.

### flask db migrate

#### flask db migrate \[--message MESSAGE] \[--sql] \[--head HEAD] \[--splice] \[--branch-label BRANCH\_LABEL] \[--version-path VERSION\_PATH] \[--rev-id REV\_ID]

Эквивалент `revision --autogenerate`. Сценарий миграции заполняется изменениями, обнаруженными автоматически. Сгенерированный сценарий необходимо просмотреть и отредактировать, так как не все типы изменений могут быть обнаружены автоматически. Эта команда не вносит никаких изменений в базу данных, а просто создает сценарий ревизии.

### flask db edit

#### flask db edit \<revision>

Отредактируйте сценарий ревизии с помощью **$EDITOR**.

### flask db upgrade

#### flask db upgrade \[--sql] \[--tag TAG] \[--x-arg ARG] \<revision>

Обновляет базу данных. Если **revision** не указан, предполагается `"head"`.

### flask db downgrade

#### flask db downgrade \[--sql] \[--tag TAG] \[--x-arg ARG] \<revision>

Понижает базу данных. Если **revision** не указана, предполагается `-1`.

### flask db stamp

#### flask db stamp \[--sql] \[--tag TAG] \<revision>

Устанавливает ревизию в базе данных на ту, которая указана в качестве аргумента, без выполнения каких-либо миграций.

### flask db current

#### flask db current \[--verbose]

Показывает текущую версию базы данных.

### flask db history

#### flask db history \[--rev-range REV\_RANGE] \[--verbose]

Показывает список миграций. Если диапазон не указан, отображается вся история.

### flask db show

#### flask db show \<revision>

Показать ревизию, обозначенную данным символом.

### flask db merge

#### flask db merge \[--message MESSAGE] \[--branch-label BRANCH\_LABEL] \[--rev-id REV\_ID] \<revisions>

Объедините две ревизии вместе. Создает новый файл ревизии.

### flask db heads

#### flask db heads \[--verbose] \[--resolve-dependencies]

Показать текущие доступные головы (_heads_) в каталоге сценария ревизии.

### flask db branches

#### flask db branches \[--verbose]

Показать текущие точки ветвления.

{% hint style="info" %}
* Все команды также принимают параметр **--directory DIRECTORY**, указывающий на каталог, содержащий сценарии миграции. Если этот аргумент опущен, используется каталог _**migrations**_.
* Каталог по умолчанию также можно указать в качестве аргумента **direcory** для конструктора **Migrate**.
* Параметр **--sql**, присутствующий в нескольких командах, выполняет миграцию в автономном режиме. Вместо выполнения команд базы данных операторы SQL, которые необходимо выполнить, выводятся на консоль.
* Подробную документацию по этим командам можно найти на [странице справочника по командам Alembic](http://alembic.zzzcomputing.com/en/latest/api/commands.html).
{% endhint %}

## Справочник API

Доступ к командам, предоставляемым интерфейсом командной строки **Flask-Migrate**, также можно получить программно, импортировав функции из модуля **flask\_migrate**. Доступные функции:

### init

#### init(directory='migrations', multidb=False)

Инициализирует поддержку миграции для приложения.

### revision

#### revision(directory='migrations', message=None, autogenerate=False, sql=False, head='head', splice=False, branch\_label=None, version\_path=None, rev\_id=None)

Создает пустой скрипт ревизии.

### migrate

#### migrate(directory='migrations', message=None, sql=False, head='head', splice=False, branch\_label=None, version\_path=None, rev\_id=None)

Создает сценарий автоматической ревизии.

### edit

#### edit(directory='migrations', revision='head')

Отредактируйте скрипт(ы) редакции с помощью **$EDITOR**.

### merge

#### merge(directory='migrations', revisions='', message=None, branch\_label=None, rev\_id=None)

Объедините две ревизии вместе. Создает новый файл миграции.

### upgrade

#### upgrade(directory='migrations', revision='head', sql=False, tag=None)

Обновляет базу данных.

### downgrade

#### downgrade(directory='migrations', revision='-1', sql=False, tag=None)

Понижает базу данных.

### show

#### show(directory='migrations', revision='head')

Показать ревизию, обозначенную данным символом.

### history

#### history(directory='migrations', rev\_range=None, verbose=False)

Показывает список миграций. Если диапазон не указан, отображается вся история.

### heads

#### heads(directory='migrations', verbose=False, resolve\_dependencies=False)

Показать текущие доступные головы в каталоге скриптов.

### branches

#### branches(directory='migrations', verbose=False)

Показать текущие точки ветвления

### current

#### current(directory='migrations', verbose=False, head\_only=False)

Показывает текущую версию базы данных.

### stamp

#### stamp(directory='migrations', revision='head', sql=False, tag=None)

Устанавливает ревизию в базе данных на ту, которая указана в качестве аргумента, без выполнения каких-либо миграций.

{% hint style="info" %}
* Эти команды будут вызывать те же функции, что и из командной строки, включая вывод на терминал. Конфигурация регистрации процесса будет переопределена **Alembic** в соответствии с содержимым файла **alembic.ini**.
* Для большей гибкости сценариев вы также можете напрямую использовать API, предоставленный **Alembic**.
{% endhint %}
