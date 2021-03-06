# Сессии Flask

## Сессии Flask

Если вы установили [Flask.secret\_key](obekt-prilozheniya-flask.md#secret\_key) (или настроили его из [SECRET\_KEY](../rukovodstvo-polzovatelya-flask/obrabotka-konfiguracii-flask.md#secret\_key)), вы можете использовать сеансы в приложениях **Flask**. Сеанс позволяет запоминать информацию от одного запроса к другому. **Flask** делает это с помощью подписанного файла **cookie**. Пользователь может просматривать содержимое сеанса, но не может изменять его, если не знает секретный ключ, поэтому обязательно установите для него что-то сложное и неразличимое.

Для доступа к текущему сеансу вы можете использовать объект **session**:

### Класс flask.session

Объект сеанса работает почти так же, как обычный словарь, с той разницей, что он отслеживает изменения.

Это прокси. См. [примечания о прокси](../rukovodstvo-polzovatelya-flask/kontekst-zaprosa-flask.md#primechaniya-k-proksi) для получения дополнительной информации.

Интересны следующие атрибуты:

#### new

`True`, если сеанс новый, в противном случае - `False`.

#### modified

`True`, если объект сеанса обнаружил модификацию. Имейте в виду, что модификации изменяемых структур не подбираются автоматически, в этой ситуации вам нужно явно установить для атрибута значение `True` самостоятельно. Вот пример:

```python
# это изменение не принимается, потому что
# изменяемый объект (здесь список) изменен.
session['objects'].append(42)
# так что отметьте его как модифицированный самостоятельно
session.modified = True
```

#### permanent

Если установлено значение `True`, сеанс живет в течение [**permanent\_session\_lifetime**](obekt-prilozheniya-flask.md#permanent\_session\_lifetime) секунд. По умолчанию 31 день. Если установлено значение `False` (по умолчанию), сеанс будет удален, когда пользователь закроет браузер.

## Интерфейс сессии

_Новое в версии 0.8_.

Интерфейс сеанса предоставляет простой способ заменить реализацию сеанса, которую использует **Flask**.

### Класс flask.sessions.SessionInterface

Базовый интерфейс, который вы должны реализовать, чтобы заменить интерфейс сеанса по умолчанию, который использует реализацию _**securecookie**_ **Werkzeug**. Единственные методы, которые вам нужно реализовать, - это open\_session () и save\_session (), у остальных есть полезные значения по умолчанию, которые вам не нужно менять.

Объект сеанса, возвращаемый методом open\_session (), должен предоставлять интерфейс, подобный словарю, а также свойства и методы из SessionMixin. Мы рекомендуем просто создать подкласс dict и добавить этот миксин:

```python
class Session(dict, SessionMixin):
    pass
```

Если open\_session () возвращает `None`, **Flask** вызовет make\_null\_session () для создания сеанса, который действует как замена, если поддержка сеанса не может работать из-за невыполнения некоторых требований. Созданный по умолчанию класс NullSession будет жаловаться на то, что секретный ключ не был установлен.

Чтобы заменить интерфейс сеанса в приложении, все, что вам нужно сделать, это назначить flask.Flask.session\_interface:

```python
app = Flask(__name__)
app.session_interface = MySessionInterface()
```

#### get\_cookie\_domain( _app_ )

Возвращает домен, который должен быть установлен для файла **cookie** сеанса.

Использует **SESSION\_COOKIE\_DOMAIN**, если он настроен, в противном случае возвращается к обнаружению домена на основе **SERVER\_NAME**.

После обнаружения (или если не установлен вообще), **SESSION\_COOKIE\_DOMAIN** обновляется, чтобы избежать повторного запуска логики.

#### get\_cookie\_httponly( _app_ )

Возвращает `True`, если файл **cookie** сеанса должен быть _**httponly**_. В настоящее время это просто возвращает значение переменной конфигурации **SESSION\_COOKIE\_HTTPONLY**.

#### get\_cookie\_path( _app_ )

Возвращает путь, для которого файл **cookie** должен быть действительным. Реализация по умолчанию использует значение из переменной конфигурации **SESSION\_COOKIE\_PATH**, если она установлена, и возвращается к **APPLICATION\_ROOT** или использует `/`, если это `None`.

#### get\_cookie\_samesite( _app_ )

Верните `'Strict'` или `'Lax'`, если файл **cookie** должен использовать атрибут **SameSite**. В настоящее время это просто возвращает значение параметра [SESSION\_COOKIE\_SAMESITE](../rukovodstvo-polzovatelya-flask/obrabotka-konfiguracii-flask.md#session\_cookie\_samesite).

#### get\_cookie\_secure( _app_ )

Возвращает `True`, если файл **cookie** должен быть безопасным. В настоящее время это просто возвращает значение параметра **SESSION\_COOKIE\_SECURE**.

#### get\_expiration\_time( _app_, _session_ )

Вспомогательный метод, который возвращает дату истечения срока действия сеанса или `None`, если сеанс связан с сеансом браузера. Реализация по умолчанию возвращает **now** + **permanent\_session\_lifetime**, настроенное в приложении.

#### is\_null\_session( _obj_ )

Проверяет, является ли данный объект нулевым сеансом. Нулевые сеансы не требуется сохранять.

Это проверяет, является ли объект экземпляром [null\_session\_class](sessii-flask.md#null\_session\_class) по умолчанию.

#### make\_null\_session( _app_ )

Создает нулевой сеанс, который действует как замещающий объект, если реальная поддержка сеанса не может быть загружена из-за ошибки конфигурации. Это в основном помогает пользователю, потому что задача нулевого сеанса - по-прежнему поддерживать поиск без жалоб, но на изменения отвечает полезное сообщение об ошибке с указанием того, что не удалось.

По умолчанию это создает экземпляр [null\_session\_class](sessii-flask.md#null\_session\_class).

#### null\_session\_class

[make\_null\_session ()](sessii-flask.md#make\_null\_session-app) будет искать здесь класс, который должен быть создан при запросе нулевого сеанса. Точно так же метод [is\_null\_session ()](sessii-flask.md#is\_null\_session-obj) выполнит проверку типов для этого типа.

псевдоним [NullSession](sessii-flask.md#klass-flask-sessions-nullsession)

#### open\_session( _app_, _request_ )

Этот метод должен быть реализован и должен либо возвращать `None` в случае сбоя загрузки из-за ошибки конфигурации, либо экземпляр объекта сеанса, который реализует словарь, такой как интерфейс + методы и атрибуты в [SessionMixin](sessii-flask.md#klass-flask-sessions-sessionmixin).

#### pickle\_based _= False_

_Новое в версии 0.10_.

Флаг, который указывает, основан ли интерфейс сеанса на **pickle**. Это может использоваться расширениями **Flask** для принятия решения о том, как работать с объектом сеанса.

#### save\_session( _app_, _session_, _response_ )

Это вызывается для фактических сеансов, возвращаемых [open\_session ()](sessii-flask.md#open\_session-app-request) в конце запроса. Он по-прежнему вызывается в контексте запроса, поэтому, если вам абсолютно необходим доступ к запросу, вы можете это сделать.

#### should\_set\_cookie( _app_, _session_ )

_Новое в версии 0.11_.

Используется серверными модулями сеанса, чтобы определить, должен ли быть установлен заголовок **Set-Cookie** для этого файла **cookie** сеанса для этого ответа. Если сеанс был изменен, **cookie** устанавливается. Если сеанс постоянный и конфигурация **SESSION\_REFRESH\_EACH\_REQUEST** истинна, **cookie** всегда устанавливается.

Эта проверка обычно пропускается, если сеанс был удален.

### Класс flask.session.SecureCookieSessionInterface

Интерфейс сеанса по умолчанию, который хранит сеансы в подписанных файлах **cookie** через модуль **itsdangerous**.

#### (_static_) digest\_method()

хеш-функция, используемая для подписи. По умолчанию **sha1**

#### key\_derivation _= 'hmac'_

имя его **itsdangerous** поддерживаемого производного ключа. По умолчанию это **hmac**.

#### open\_session( _app_, _request_ )

Этот метод должен быть реализован и должен либо возвращать `None` в случае сбоя загрузки из-за ошибки конфигурации, либо экземпляр объекта сеанса, который реализует словарь, такой как интерфейс + методы и атрибуты в [SessionMixin](sessii-flask.md#klass-flask-sessions-sessionmixin).

#### salt _= 'cookie-session'_

соль, которая должна применяться поверх секретного ключа для подписи сеансов на основе файлов **cookie**.

#### save\_session( _app_, _session_, _response_ )

Это вызывается для фактических сеансов, возвращаемых [open\_session ()](sessii-flask.md#open\_session-app-request-1) в конце запроса. Он по-прежнему вызывается в контексте запроса, поэтому, если вам абсолютно необходим доступ к запросу, вы можете это сделать.

#### serializer _= \<flask.json.tag.TaggedJSONSerializer object>_

Сериализатор Python для полезной нагрузки. По умолчанию используется компактный сериализатор, производный от **JSON**, с поддержкой некоторых дополнительных типов Python, таких как объекты **datetime** или кортежи.

#### session\_class

псевдоним [SecureCookieSession](sessii-flask.md#klass-flask-sessions-securecookiesession)

### Класс flask.sessions.SecureCookieSession( _initial=None_ )

Базовый класс для сеансов на основе подписанных файлов **cookie**.

Этот бэкэнд сеанса установит [modified](sessii-flask.md#modified-1) и [accessed](sessii-flask.md#accessed) атрибуты. Он не может надежно отслеживать, является ли сеанс новым (или пустым), поэтому **new** остается жестко закодированным на `False`.

#### accessed _= False_

заголовок, который позволяет кэшировать прокси для кеширования разных страниц для разных пользователей.

#### get( _key_, _default=None_ )

Возвращает значение ключа, если ключ находится в словаре, иначе по умолчанию.

#### modified _= False_

При изменении данных устанавливается значение `True`. Отслеживается только сам словарь сеанса; если сеанс содержит изменяемые данные (например, вложенный dict), тогда при изменении этих данных необходимо вручную установить значение `True`. Файл **cookie** сеанса будет записан в ответ, только если он имеет значение `True`.

#### setdefault( _key_, _default=None_ )

Вставляет ключ со значением по умолчанию, если ключа нет в словаре.

Возвращает значение ключа, если ключ находится в словаре, иначе по умолчанию.

### Класс flask.sessions.NullSession( _initial=None_ )

Класс, используемый для создания более приятных сообщений об ошибках, если сеансы недоступны. Будет по-прежнему разрешен доступ только для чтения к пустому сеансу, но не удастся установить.

### Класс flask.sessions.SessionMixin

Расширяет базовый словарь с атрибутами сеанса.

#### accessed _= True_

Некоторые реализации могут определять, когда данные сеанса читаются или записываются, и устанавливать это, когда это происходит. По умолчанию миксин жестко задан как `True`.

#### modified _= True_

Некоторые реализации могут обнаруживать изменения в сеансе и устанавливать их, когда это происходит. По умолчанию миксин жестко задан как `True`.

#### (_property_) permanent

Это отражает ключ `'_permanent'` в dict.

{% hint style="info" %}
Ключ конфигурации **PERMANENT\_SESSION\_LIFETIME** также может быть целым числом, начиная с **Flask 0.8**. Либо модифицируйте это самостоятельно, либо используйте атрибут [permanent\_session\_lifetime](obekt-prilozheniya-flask.md#permanent\_session\_lifetime) в приложении, который автоматически преобразует результат в целое число.
{% endhint %}
