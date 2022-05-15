# Flask-Session

Добро пожаловать в документацию **Flask-Session**. Flask-Session - это расширение для Flask, которое добавляет поддержку серверного сеанса в ваше приложение. Требуется Flask 0.8 или новее, если вы используете старую версию, проверьте Поддержка старых и новых сессий.

Если вы не знакомы с Flask, я настоятельно рекомендую вам попробовать. Flask - это микрофреймворк для Python, с которым действительно интересно работать. Если вы хотите погрузиться в его документацию, воспользуйтесь следующими ссылками:

[Документация Flask](https://flask.palletsprojects.com/en/2.0.x/)

## Установка

Установите расширение с помощью следующей команды:

```bash
$ easy_install Flask-Session
```

или, альтернативно, если у вас установлен pip:

```bash
$ pip install Flask-Session
```

## Быстрый старт

Flask-Session действительно прост в использовании.

В основном для обычного использования одного приложения Flask все, что вам нужно сделать, это создать свое приложение Flask, загрузить выбранную конфигурацию и затем создать объект Session, передав ему приложение.

Экземпляр Session не используется для прямого доступа, вы всегда должны использовать flask.session:

```python
from flask import Flask, session
from flask_session import Session

app = Flask(__name__)
# Более подробную информацию см. В разделе «Конфигурация».
SESSION_TYPE = 'redis'
app.config.from_object(__name__)
Session(app)

@app.route('/set/')
def set():
    session['key'] = 'value'
    return 'ok'

@app.route('/get/')
def get():
    return session.get('key', 'not set')
```

Вы также можете настроить свое приложение позже, используя метод init\_app ():

```python
sess = Session()
sess.init_app(app)
```

## Конфигурация

Для Flask-Session существуют следующие значения конфигурации. Flask-Session загружает эти значения из конфигурации вашего приложения Flask, поэтому вы должны сначала настроить свое приложение, прежде чем передавать его во Flask-Session. Обратите внимание, что эти значения не могут быть изменены после применения init\_app, поэтому не изменяйте их во время выполнения.

Мы не предоставляем что-то вроде SESSION\_REDIS\_HOST и SESSION\_REDIS\_PORT, если вы хотите использовать RedisSessionInterface, вам следует настроить SESSION\_REDIS на свой собственный экземпляр redis.Redis. Это дает вам больше гибкости, например, возможно, вы хотите использовать один и тот же экземпляр redis.Redis для целей кеширования, тогда вам не нужно хранить два экземпляра redis.Redis в одном процессе.

Следующие значения конфигурации являются встроенными значениями конфигурации в самом Flask, которые связаны с сеансом. **Все они понимаются Flask-Session, например, вы должны использовать PERMANENT\_SESSION\_LIFETIME для управления временем жизни сеанса.**

| Ключ                             | Описание                                                                                                                                      |
| -------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| **SESSION\_COOKIE\_NAME**        | имя файла cookie сеанса                                                                                                                       |
| **SESSION\_COOKIE\_DOMAIN**      | домен для файла cookie сеанса. Если он не установлен, cookie будет действителен для всех поддоменов SERVER\_NAME.                             |
| **SESSION\_COOKIE\_PATH**        | путь для файла cookie сеанса. Если он не установлен, cookie будет действителен для всего APPLICATION\_ROOT или если он не установлен для '/'. |
| **SESSION\_COOKIE\_HTTPONLY**    | контролирует, должен ли cookie быть установлен с флагом httponly. По умолчанию True.                                                          |
| **SESSION\_COOKIE\_SECURE**      | контролирует, должен ли cookie быть установлен с безопасным флагом. По умолчанию False.                                                       |
| **PERMANENT\_SESSION\_LIFETIME** | время жизни постоянного сеанса как объект datetime.timedelta. Начиная с Flask 0.8 это также может быть целым числом, представляющим секунды.  |

Список ключей конфигурации, также понимаемых расширением:

| Ключ                           | Описание                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **SESSION\_TYPE**              | <p>Указывает, какой тип интерфейса сеанса использовать. Встроенные типы сессий:</p><p><strong>null</strong>: NullSessionInterface (по умолчанию)</p><p><strong>redis</strong>: RedisSessionInterface</p><p><strong>memcached</strong>: MemcachedSessionInterface</p><p><strong>filesystem</strong>: FileSystemSessionInterface</p><p><strong>mongodb</strong>: MongoDBSessionInterface</p><p><strong>sqlalchemy</strong>: SqlAlchemySessionInterface</p> |
| **SESSION\_PERMANENT**         | Независимо от того, используется ли постоянный сеанс или нет, по умолчанию установлено значение True                                                                                                                                                                                                                                                                                                                                                     |
| **SESSION\_USE\_SIGNER**       | Независимо от того, подписываете ли сеансовый cookie sid или нет, если установлено значение True, вы должны установить flask.Flask.secret\_key, по умолчанию - False                                                                                                                                                                                                                                                                                     |
| **SESSION\_KEY\_PREFIX**       | Префикс, добавляемый перед всеми сеансовыми ключами. Это позволяет использовать один и тот же сервер внутреннего хранилища для разных приложений, по умолчанию «session:»                                                                                                                                                                                                                                                                                |
| **SESSION\_REDIS**             | Экземпляр redis.Redis, по умолчанию подключается к 127.0.0.1:6379                                                                                                                                                                                                                                                                                                                                                                                        |
| **SESSION\_MEMCACHED**         | Экземпляр memcache.Client, по умолчанию подключается к 127.0.0.1:11211                                                                                                                                                                                                                                                                                                                                                                                   |
| **SESSION\_FILE\_DIR**         | Каталог, в котором хранятся файлы сеанса. По умолчанию используется каталог flask\_session в текущем рабочем каталоге.                                                                                                                                                                                                                                                                                                                                   |
| **SESSION\_FILE\_THRESHOLD**   | Максимальное количество элементов, которые сеанс хранит до того, как он начнет удалять некоторые, по умолчанию 500.                                                                                                                                                                                                                                                                                                                                      |
| **SESSION\_FILE\_MODE**        | Файловый режим, который требуется для файлов сеанса, по умолчанию 0600                                                                                                                                                                                                                                                                                                                                                                                   |
| **SESSION\_MONGODB**           | Экземпляр pymongo.MongoClient, по умолчанию подключается к 127.0.0.1:27017                                                                                                                                                                                                                                                                                                                                                                               |
| **SESSION\_MONGODB\_DB**       | База данных MongoDB, которую вы хотите использовать, по умолчанию «flask\_session»                                                                                                                                                                                                                                                                                                                                                                       |
| **SESSION\_MONGODB\_COLLECT**  | Коллекция MongoDB, которую вы хотите использовать, по умолчанию «сеансы»                                                                                                                                                                                                                                                                                                                                                                                 |
| **SESSION\_SQLALCHEMY**        | Экземпляр flask\_sqlalchemy.SQLAlchemy, URI подключения к базе данных которого настроен с помощью параметра SQLALCHEMY\_DATABASE\_URI                                                                                                                                                                                                                                                                                                                    |
| **SESSION\_SQLALCHEMY\_TABLE** | Имя таблицы SQL, которую вы хотите использовать, по умолчанию «sessions».                                                                                                                                                                                                                                                                                                                                                                                |

В основном вам нужно только настроить SESSION\_TYPE.

{% hint style="info" %}
По умолчанию все ненулевые сеансы в Flask-Session являются постоянными.
{% endhint %}

_Новое в версии 0.2:_  **SESSION\_TYPE**: **sqlalchemy**, **SESSION\_USE\_SIGNER**

## Встроенные интерфейсы сеанса

#### NullSessionInterface

Если вы не настроите другой **SESSION\_TYPE**, он будет использоваться для создания более приятных сообщений об ошибках. Разрешит доступ только для чтения к пустому сеансу, но не удастся установить.

#### RedisSessionInterface

Использует хранилище ключей и значений Redis в качестве серверной части сеанса. (требуется [redis-py](https://github.com/andymccurdy/redis-py))

Соответствующие значения конфигурации: **SESSION\_REDIS**

#### MemcachedSessionInterface

Использует Memcached как серверную часть сеанса. (требуется [pylibmc](http://sendapatch.se/projects/pylibmc/) или [memcache](https://github.com/linsomniac/python-memcached))

Соответствующие значения конфигурации: **SESSION\_MEMCACHED**

#### FileSystemSessionInterface

Использует cachelib.file.FileSystemCache как серверную часть сеанса.

Соответствующие значения конфигурации: **SESSION\_FILE\_DIR, SESSION\_FILE\_THRESHOLD, SESSION\_FILE\_MODE**

#### MongoDBSessionInterface

Использует MongoDB как серверную часть сеанса. (требуется [pymongo](https://docs.mongodb.com/drivers/pymongo/))

Соответствующие значения конфигурации: **SESSION\_MONGODB, SESSION\_MONGODB\_DB, SESSION\_MONGODB\_COLLECT**

#### SqlAlchemySessionInterface

_Новое в версии 0.2._

Использует SQLAlchemy как серверную часть сеанса. (Требуется [Flask-SQLAlchemy](https://flask-sqlalchemy.palletsprojects.com/en/2.x/))

Соответствующие значения конфигурации: **SESSION\_SQLALCHEMY, SESSION\_SQLALCHEMY\_TABLE**

## API

### &#x20;_class_ flask\_session.Session(_app=None_)

Этот класс используется для добавления сеанса на стороне сервера в одно или несколько приложений Flask.

Есть два режима использования. Один из них - инициализировать экземпляр очень специфическим приложением Flask:

```python
app = Flask(__name__)
Session(app)
```

Вторая возможность - создать объект один раз и настроить приложение позже:

```python
sess = Session()

def create_app():
    app = Flask(__name__)
    sess.init_app(app)
    return app
```

По умолчанию Flask-Session будет использовать NullSessionInterface, вам действительно следует настроить свое приложение для использования другого SessionInterface.

{% hint style="info" %}
Вы не можете использовать экземпляр Session напрямую, Session просто меняет атрибут session\_interface в ваших приложениях Flask.
{% endhint %}

#### &#x20;init\_app(_app_)

Используется для настройки сеанса для вашего объекта приложения.

#### Параметры:

* **app** - объект приложения Flask с правильной конфигурацией.

### &#x20;_class_ flask\_session.sessions.ServerSideSession(_initial=None_, _sid=None_, _permanent=None_)

Базовый класс для сеансов на стороне сервера.

* **sid** - Идентификатор сеанса, внутри мы используем **uuid.uuid4 ()** для генерации одного идентификатора сеанса. Вы можете получить к нему доступ с помощью **session.sid**.

### &#x20;_class_ flask\_session.NullSessionInterface

Используется для открытия экземпляра **flask.sessions.NullSession**.

### &#x20;_class_ flask\_session.RedisSessionInterface(_redis_, _key\_prefix_, _use\_signer=False_, _permanent=True_)

Использует хранилище ключей и значений Redis в качестве серверной части сеанса.

_Новое в версии 0.2:_ добавлен параметр **use\_signer**.

**Параметры:**

* **redis** - Экземпляр redis.Redis.
* **key\_prefix** - Префикс, который добавляется ко всем ключам хранилища Redis.
* **use\_signer** - Подписывать или нет файл cookie идентификатора сеанса.
* **permanent** - Независимо от того, использовать ли постоянный сеанс или нет.

### &#x20;_class_ flask\_session.MemcachedSessionInterface(_client_, _key\_prefix_, _use\_signer=False_, _permanent=True_)

Интерфейс сеанса, использующий memcached в качестве бэкэнда.

_Новое в версии 0.2:_ добавлен параметр **use\_signer**.

Параметры:

* **client** - Экземпляр memcache.Client.
* **key\_prefix** - Префикс, который добавляется ко всем ключам хранилища Memcached.
* **use\_signer** - Подписывать или нет файл cookie идентификатора сеанса.
* **permanent** - Независимо от того, использовать ли постоянный сеанс или нет.

### &#x20;_class_ flask\_session.FileSystemSessionInterface(_cache\_dir_, _threshold_, _mode_, _key\_prefix_, _use\_signer=False_, _permanent=True_)

Использует cachelib.file.FileSystemCache как серверную часть сеанса.

_Новое в версии 0.2:_ добавлен параметр **use\_signer**.

**Параметры**:

* **cache\_dir** - каталог, в котором хранятся файлы сеанса.
* **threshold** - максимальное количество элементов, которые сеанс хранит до того, как он начнет удалять некоторые из них.
* **mode** - файловый режим, который требуется для файлов сеанса, по умолчанию 0600
* **key\_prefix** - Префикс, который добавляется к ключам хранилища FileSystemCache.
* **use\_signer** - Подписывать или нет файл cookie идентификатора сеанса.
* **permanent** - Независимо от того, использовать ли постоянный сеанс или нет.

### &#x20;_class_ flask\_session.MongoDBSessionInterface(_client_, _db_, _collection_, _key\_prefix_, _use\_signer=False_, _permanent=True_)

Интерфейс сеанса, использующий mongodb в качестве бэкэнда.

_Новое в версии 0.2:_ добавлен параметр **use\_signer**.

**Параметры**:

* **client** - Экземпляр pymongo.MongoClient.
* **db** - База данных, которую вы хотите использовать.
* **collection** - Коллекция, которую вы хотите использовать.
* **key\_prefix** - Префикс, который добавляется ко всем ключам хранилища MongoDB.
* **use\_signer** - Подписывать или нет файл cookie идентификатора сеанса.
* **permanent** - Независимо от того, использовать ли постоянный сеанс или нет.

### &#x20;_class_ flask\_session.SqlAlchemySessionInterface(_app_, _db_, _table_, _key\_prefix_, _use\_signer=False_, _permanent=True_)

Использует Flask-SQLAlchemy из приложения Flask в качестве серверной части сеанса.

_Новое в версии 0.2_.

* **app** - Экземпляр приложения Flask.
* **db** - Экземпляр Flask-SQLAlchemy.
* **table** - Имя таблицы, которое вы хотите использовать.
* **key\_prefix** - Префикс, который добавляется ко всем ключам хранилища.
* **use\_signer** - Подписывать или нет файл cookie идентификатора сеанса.
* **permanent** - Независимо от того, использовать ли постоянный сеанс или нет.

## Журнал изменений

### Версия 0.1

Первый общедоступный предварительный выпуск.

### Версия 0.1.1

Исправлена ошибка InvalidDocument backend MongoDB.

### Версия 0.2

* Добавлен SqlAlchemySessionInterface.
* Добавлена поддержка подписи идентификатора сеанса cookie.
* Различные исправления.

### Версия 0.2.1

Исправлена ошибка подписи.

### Версия 0.2.2

Добавлена поддержка непостоянной сессии.

### Версия 0.2.3

* Исправлена ошибка подписи в Python 3.x
* Исправлен сбой MongoDBSessionInterface в Python 3.x
* Исправлен сбой SqlAlchemySessionInterface в Python 3.x
* Фиксированная поддержка StrictRedis

### Версия 0.3

* SqlAlchemySessionInterface теперь использует тип LargeBinary для хранения данных
* Исправлен метод удаления MongoDBSessionInterface, который не найден
* Исправлена ошибка TypeError при получении store\_id с помощью подписавшего

### Версия 0.3.1

* SqlAlchemySessionInterface теперь использует VARCHAR (255) для хранения идентификатора сеанса
* SqlAlchemySessionInterface больше не запускает db.create\_all

### Версия 0.3.2

Изменен werkzeug.contrib.cache на cachelib

### Версия 0.4.0

Добавлена поддержка SESSION\_COOKIE\_SAMESITE
