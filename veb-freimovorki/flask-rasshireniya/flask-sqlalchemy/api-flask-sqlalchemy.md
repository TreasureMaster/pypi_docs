# API Flask-SQLAlchemy

## Конфигурация

### SQLAlchemy

#### _class_ flask\_sqlalchemy.SQLAlchemy(_app=None_, _use\_native\_unicode=True_, _session\_options=None_, _metadata=None_, _query\_class=\<class 'flask\_sqlalchemy.BaseQuery'>_, _model\_class=\<class 'flask\_sqlalchemy.model.Model'>_, _engine\_options=None_)

Этот класс используется для управления интеграцией SQLAlchemy с одним или несколькими приложениями Flask. В зависимости от того, как вы инициализируете объект, его можно использовать сразу или при необходимости подключить к приложению Flask.

Есть два режима использования, которые работают очень похоже. Один из них привязывает экземпляр к очень конкретному приложению Flask:

```python
app = Flask(__name__)
db = SQLAlchemy(app)
```

Второй вариант — создать объект один раз, а затем настроить приложение для его поддержки:

```python
db = SQLAlchemy()

def create_app():
    app = Flask(__name__)
    db.init_app(app)
    return app
```

Разница между ними заключается в том, что в первом случае такие методы, как [create\_all()](api-flask-sqlalchemy.md#create\_all-bind-\_\_all\_\_-app-none) и [drop\_all()](api-flask-sqlalchemy.md#drop\_all-bind-\_\_all\_\_-app-none), будут работать постоянно, а во втором случае должен существовать [flask.Flask.app\_context()](https://flask.palletsprojects.com/en/2.0.x/api/#flask.Flask.app\_context).

По умолчанию Flask-SQLAlchemy применяет некоторые настройки, специфичные для серверной части, чтобы улучшить работу с ними.

Начиная с **SQLAlchemy 0.6**, SQLAlchemy будет проверять библиотеку на предмет собственной поддержки Unicode. Если он обнаружит юникод, он позволит библиотеке обработать это, в противном случае сделает это сам. Иногда это обнаружение может дать сбой, и в этом случае вы можете установить для **use\_native\_unicode** (или ключа конфигурации `SQLALCHEMY_NATIVE_UNICODE`) значение `False`. Обратите внимание, что ключ конфигурации переопределяет значение, которое вы передаете конструктору. Прямая поддержка **use\_native\_unicode** и `SQLALCHEMY_NATIVE_UNICODE` устарела с версии 2.4 и будет удалена в версии 3.0. Вместо этого можно использовать **engine\_options** и `SQLALCHEMY_ENGINE_OPTIONS`.

Этот класс также обеспечивает доступ ко всем функциям и классам SQLAlchemy из модулей **sqlalchemy** и **sqlalchemy.orm**. Таким образом, вы можете объявить модели следующим образом:

```python
class User(db.Model):
    username = db.Column(db.String(80), unique=True)
    pw_hash = db.Column(db.String(80))
```

Вы по-прежнему можете использовать **sqlalchemy** и **sqlalchemy.orm** напрямую, но обратите внимание, что настройки Flask-SQLAlchemy доступны только через экземпляр этого класса [SQLAlchemy](api-flask-sqlalchemy.md#sqlalchemy). Классами запросов по умолчанию являются [BaseQuery](api-flask-sqlalchemy.md#basequery) для `db.Query`, `db.Model.query_class` и `query_class` по умолчанию для `db.relationship` и `db.backref`. Если вы используете эти интерфейсы напрямую через **sqlalchemy** и **sqlalchemy.orm**, классом запроса по умолчанию будет класс **sqlalchemy**.

{% hint style="info" %}
**Тщательно проверяйте типы**

Не выполняйте проверки типа или экземпляра **instance** для **db.Table**, который эмулирует поведение **Table**, но не является классом. **db.Table** предоставляет интерфейс таблицы **Table**, но это функция, которая позволяет опускать метаданные.
{% endhint %}

Параметр **session\_options**, если он предоставлен, представляет собой набор параметров, которые необходимо передать конструктору сеанса. См. [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) для стандартных опций.

Параметр **engine\_options**, если он указан, представляет собой набор параметров, которые необходимо передать для создания движка. См. [create\_engine()](https://docs.sqlalchemy.org/en/14/core/engines.html#sqlalchemy.create\_engine) для стандартных опций. Значения, указанные здесь, будут объединены и переопределяют все, что установлено в переменной конфигурации `SQLALCHEMY_ENGINE_OPTIONS` или иным образом установлено этой библиотекой.

_Новое в версии 0.10_: Добавлен параметр **session\_options**.

_Новое в версии 0.16_: **scopefunc** теперь принимается в **session\_options**. Это позволяет указать пользовательскую функцию, которая будет определять область действия сеанса SQLAlchemy.

_Новое в версии 2.1_: Добавлен параметр **metadata**. Это позволяет устанавливать пользовательские соглашения об именах среди других нетривиальных вещей.

Параметр **query\_class** был добавлен, чтобы разрешить настройку класса запроса вместо значения по умолчанию [BaseQuery](api-flask-sqlalchemy.md#basequery).

Был добавлен параметр **model\_class**, который позволяет использовать пользовательский класс модели вместо [Model](api-flask-sqlalchemy.md#model).

_Изменено в версии 2.1_: используйте один и тот же класс запроса в сеансе **session**, **Model.query** и **Query**.

_Новое в версии 2.4_: Добавлен параметр **engine\_options**.

_Изменено в версии 2.4_: Параметр **use\_native\_unicode** <mark style="color:orange;">устарел</mark>.

_Изменено в версии 2.4.3_: `COMMIT_ON_TEARDOWN` <mark style="color:orange;">устарел</mark> и будет удален в версии 3.1. Вместо этого вызовите `db.session.commit()` напрямую.

### Query_= None_

Класс запросов по умолчанию, используемый **Model.query** и другими запросами. Настройте это, передав **query\_class** в [SQLAlchemy()](api-flask-sqlalchemy.md#sqlalchemy). По умолчанию используется [BaseQuery](api-flask-sqlalchemy.md#basequery).

### apply\_driver\_hacks(_app_, _sa\_url_, _options_)

Этот метод вызывается перед созданием движка и используется для внедрения хаков, специфичных для драйвера, в параметры. Параметр **options** представляет собой словарь аргументов ключевого слова, которые затем будут использоваться для вызова функции [sqlalchemy.create\_engine()](https://docs.sqlalchemy.org/en/14/core/engines.html#sqlalchemy.create\_engine).

Реализация по умолчанию обеспечивает более разумные значения по умолчанию для таких вещей, как размеры пула для MySQL и sqlite. Также он вводит параметр `SQLALCHEMY_NATIVE_UNICODE`.

_Изменено в версии 2.5_: Возвращает `(sa_url, options)`. SQLAlchemy 1.4 сделал URL-адрес неизменяемым, поэтому любые изменения в нем теперь должны передаваться обратно исходному вызывающему объекту.

### apply\_pool\_defaults(_app_, _options_)

_Изменено в версии 2.5_: возвращает параметры **options** как словарь для согласованности с [apply\_driver\_hacks()](api-flask-sqlalchemy.md#apply\_driver\_hacks-app-sa\_url-options).

### create\_all(_bind='\_\_all\_\_'_, _app=None_)

Создает все таблицы.

_Изменено в версии 0.12_: Добавлены параметры

### create\_engine(_sa\_url_, _engine\_opts_)

Переопределите этот метод, чтобы окончательно решить, как создается механизм SQLAlchemy.

В большинстве случаев вы захотите использовать переменную конфигурации `«SQLALCHEMY_ENGINE_OPTIONS»` или установить **engine\_options** для [SQLAlchemy()](api-flask-sqlalchemy.md#sqlalchemy).

### create\_scoped\_session(_options=None_)

Создайте [scoped\_session](https://docs.sqlalchemy.org/en/14/orm/contextual.html#sqlalchemy.orm.scoping.scoped\_session) на фабрике из [create\_session()](api-flask-sqlalchemy.md#create\_session-options).

Дополнительный ключ `«scopefunc»` может быть установлен в словаре параметров **options**, чтобы указать пользовательскую функцию области. Если он не указан, используется идентификатор стека контекста приложения Flask. Это гарантирует, что сеансы будут создаваться и удаляться вместе с циклом запроса/ответа, и в большинстве случаев это будет нормально.

#### Параметры:

* **options** - словарь аргументов ключевого слова, переданных классу сеанса в **create\_session**

### create\_session(_options_)

Создайте фабрику сеансов, используемую функцией [create\_scoped\_session()](api-flask-sqlalchemy.md#create\_scoped\_session-options-none).

Фабрика **должна** возвращать объект, который SQLAlchemy распознает как сеанс, иначе регистрация событий сеанса может вызвать исключение.

Допустимые фабрики включают класс [Session](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.Session) или [sessionmaker](https://docs.sqlalchemy.org/en/14/orm/session\_api.html#sqlalchemy.orm.sessionmaker).

Реализация по умолчанию создает **sessionmaker** для [SignallingSession](api-flask-sqlalchemy.md#signallingsession).

#### Параметры:

* **options** - словарь аргументов ключевого слова, переданных классу сеанса

### drop\_all(_bind='\_\_all\_\_'_, _app=None_)

Удаляет все таблицы.

_Изменено в версии 0.12_: Добавлены параметры

### engine

Дает доступ к движку. Если конфигурация базы данных привязана к определенному приложению (инициализированному приложением), это всегда будет возвращать соединение с базой данных. Однако если используется текущее приложение, это может привести к ошибке [RuntimeError](https://docs.python.org/3/library/exceptions.html#RuntimeError), если в данный момент ни одно приложение не активно.

### get\_app(_reference\_app=None_)

Вспомогательный метод, реализующий логику поиска приложения.

### get\_binds(_app=None_)

Возвращает словарь с отображением `table->engine`.

Это подходит для использования `sessionmaker(binds=db.get_binds(app))`.

### get\_engine(_app=None_, _bind=None_)

Возвращает конкретный движок БД.

### get\_tables\_for\_bind(_bind=None_)

Возвращает список всех таблиц, релевантных для привязки.

### init\_app(_app_)

Этот обратный вызов можно использовать для инициализации приложения для использования с этой настройкой базы данных. Никогда не используйте базу данных в контексте приложения, не инициализированного таким образом, иначе произойдет утечка соединений.

### make\_connector(_app=None_, _bind=None_)

Создает **connector** для заданного состояния и привязки.

### make\_declarative\_base(_model_, _metadata=None_)

Создает декларативную базу, от которой будут наследоваться все модели.

#### Параметры:

* **model** - класс базовой модели (или кортеж базовых классов) для передачи в **declarative\_base()**. Или класс, возвращенный из **declarative\_base**, и в этом случае новый базовый класс не создается.
* **metadata** - Экземпляр **MetaData** для использования или `None` для использования SQLAlchemy по умолчанию.

### metadata

Метаданные, связанные с **db.Model**.

### reflect(_bind='\_\_all\_\_'_, _app=None_)

Отражает таблицы из базы данных.

_Изменено в версии 0.12_: Добавлены параметры

## Модели

### Model

#### _class_ flask\_sqlalchemy.Model

Базовый класс для декларативной базовой модели SQLAlchemy.

Чтобы определить модели, создайте подкласс **db.Model**, а не этот класс. Чтобы настроить **db.Model**, создайте подкласс и передайте его как **model\_class** в [SQLAlchemy](api-flask-sqlalchemy.md#sqlalchemy).

### \_\_bind\_key\_\_

Опционально объявляет привязку для использования. `None` относится к привязке по умолчанию. Дополнительные сведения см. в разделе [Несколько баз данных с привязками](rukovodstvo-polzovatelya-flask-sqlalchemy/neskolko-baz-dannykh-s-privyazkami.md).

### \_\_tablename\_\_

Имя таблицы в базе данных. Это требуется для SQLAlchemy; однако Flask-SQLAlchemy установит его автоматически, если в модели определен первичный ключ. Если **\_\_table\_\_** или **\_\_tablename\_\_** заданы явно, они будут использоваться вместо них.

### BaseQuery

#### _class_ flask\_sqlalchemy.BaseQuery(_entities_, _session=None_)

Подкласс SQLAlchemy [Query](https://docs.sqlalchemy.org/en/14/orm/query.html#sqlalchemy.orm.Query) с удобными методами для запросов в веб-приложении.

Это объект запроса **query** по умолчанию, используемый для моделей и представленный как [Query](api-flask-sqlalchemy.md#query-none). Переопределите класс запроса для отдельной модели, создав подкласс и установив **query\_class**.

### first\_or\_404(_description=None_)

Подобно **first()**, но прерывается с ошибкой 404, если не найдено, вместо возврата `None`.

### get\_or\_404(_ident_, _description=None_)

Подобно **get()**, но прерывается с ошибкой 404, если не найдено, вместо возврата `None`.

### paginate(_page=None_, _per\_page=None_, _error\_out=True_, _max\_per\_page=None_)

Возвращает элементы **per\_page** со страницы **page**.

Если **page** или **per\_page** имеют значение `None`, они будут извлечены из запроса. Если указано значение **max\_per\_page**, **per\_page** будет ограничено этим значением. Если запроса нет или их нет в запросе, по умолчанию они равны 1 и 20 соответственно.

Когда **error\_out** имеет значение `True` (по умолчанию), следующие правила вызовут ответ 404:

* Элементы не найдены, и **page** не 1.
* **page** меньше 1 или **per\_page** имеет отрицательное значение.
* **page** или **per\_page** не являются целыми числами.

Когда **error\_out** имеет значение `False`, значения **page** и **per\_page** по умолчанию равны 1 и 20 соответственно.

Возвращает объект [Pagination](api-flask-sqlalchemy.md#pagination).

## Сессии

### SignallingSession

#### _class_ flask\_sqlalchemy.SignallingSession(_db_, _autocommit=False_, _autoflush=True_, _\*\*options_)

Сеанс сигнализации — это сеанс по умолчанию, который использует Flask-SQLAlchemy. Он расширяет систему сеансов по умолчанию за счет выбора привязки и отслеживания изменений.

Если вы хотите использовать другой сеанс, вы можете переопределить функцию [SQLAlchemy.create\_session()](api-flask-sqlalchemy.md#create\_session-options).

_Новое в версии 2.0_.

_Новое в версии 2.1_: добавлена опция привязки, которая позволяет присоединить сеанс к внешней транзакции.

### get\_bind(_mapper=None_, _clause=None_)

Возвращает механизм или соединение для данной модели или таблицы, используя ключ **\_\_bind\_key\_\_**, если он установлен.

## Утилиты

### Pagination

#### _class_ flask\_sqlalchemy.Pagination(_query_, _page_, _per\_page_, _total_, _items_)

Внутренний вспомогательный класс, возвращаемый [BaseQuery.paginate()](api-flask-sqlalchemy.md#paginate-page-none-per\_page-none-error\_out-true-max\_per\_page-none). Вы также можете создать его из любого другого объекта запроса SQLAlchemy, если вы работаете с другими библиотеками. Кроме того, в качестве объекта запроса можно передать `None`, и в этом случае функции [prev()](api-flask-sqlalchemy.md#prev-error\_out-false) и [next()](api-flask-sqlalchemy.md#next-error\_out-false) больше не будут работать.

### has\_next

`True`, если следующая страница существует.

### has\_prev

`True`, если предыдущая страница существует

### items _= None_

элементы для текущей страницы

### iter\_pages(_left\_edge=2_, _left\_current=2_, _right\_current=5_, _right\_edge=2_)

Перебирает номера страниц в разбиении на страницы. Четыре параметра контролируют пороги количества чисел, которые должны быть получены из сторон. Номера пропущенных страниц представлены как `None`. Вот как вы можете отобразить такую разбивку на страницы в шаблонах:

```python
{% raw %}
{% macro render_pagination(pagination, endpoint) %}
  <div class=pagination>
  {%- for page in pagination.iter_pages() %}
    {% if page %}
      {% if page != pagination.page %}
        <a href="{{ url_for(endpoint, page=page) }}">{{ page }}</a>
      {% else %}
        <strong>{{ page }}</strong>
      {% endif %}
    {% else %}
      <span class=ellipsis>…</span>
    {% endif %}
  {%- endfor %}
  </div>
{% endmacro %}
{% endraw %}
```

### next(_error\_out=False_)

Возвращает объект [Pagination](api-flask-sqlalchemy.md#pagination) для следующей страницы.

### next\_num

Номер следующей страницы

### page _= None_

текущий номер страницы (1 проиндексирована)

### pages

Общее количество страниц

### per\_page _= None_

количество элементов, отображаемых на странице.

### prev(_error\_out=False_)

Возвращает объект [Pagination](api-flask-sqlalchemy.md#pagination) для предыдущей страницы.

### prev\_num

Номер предыдущей страницы.

### query _= None_

неограниченный объект запроса, который использовался для создания этого объекта разбивки на страницы.

### total _= None_

общее количество элементов, соответствующих запросу

### get\_debug\_queries()

#### flask\_sqlalchemy.get\_debug\_queries()

В режиме отладки Flask-SQLAlchemy будет регистрировать все SQL-запросы, отправленные в базу данных. Эта информация доступна до конца запроса, что позволяет легко убедиться, что сгенерированный SQL соответствует ожидаемому при ошибках или модульном тестировании. Если вы не хотите включать режим DEBUG для своих юнит-тестов, вы также можете включить запись запросов, установив для переменной конфигурации `«SQLALCHEMY_RECORD_QUERIES»` значение `True`. Это автоматически включается, если Flask находится в режиме тестирования.

Возвращаемое значение будет списком именованных кортежей со следующими атрибутами:

* **statement** - проблемный оператор SQL
* **parameters** - Параметры оператора SQL
* **start\_time / end\_time** - Время начала запроса / получения результатов. Пожалуйста, имейте в виду, что используемая функция таймера зависит от вашей платформы. Эти значения полезны только для сортировки или сравнения. Они не обязательно представляют абсолютную отметку времени.
* **duration** - Время выполнения запроса в секундах
* **context** - Строка, дающая приблизительную оценку того, где в вашем приложении был выдан запрос. Точный формат не определен, поэтому не пытайтесь восстановить имя файла или имя функции.
