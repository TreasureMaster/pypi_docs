# Оглавление Flask ?

Эта часть документации охватывает все интерфейсы Flask. Для частей, где Flask зависит от внешних библиотек, мы документируем наиболее важные прямо здесь и даем ссылки на каноническую документацию.

### Объект приложения

* class flask.Flask

### Объекты макета

* class flask.Blueprint

### Данные входящего запроса

* class flask.Request
* flask.request

### Объекты ответа

* class flask.Response

### Сессии

* class flask.session

### Интерфейс сессии

* class flask.sessions.SessionInterface
* class flask.sessions.SecureCookieSessionInterface
* class flask.sessions.SecureCookieSession
* class flask.sessions.NullSession
* class flask.sessions.SessionMixin

### Тестовый клиент

* class flask.testing.FlaskClient

### Запуск тестирования CLI

* class flask.testing.FlaskCliRunner

### Глобальные атрибуты приложения

* flask.g
* class flask.ctx.\_\_AppCtxGlobals

### Полезные функции и классы

* flask.current\_app
* flask.has\_request_\__context ()
* flask.copy\_current_\__request\_context ()
* flask.has\_app_\__context ()
* flask.url\_for ()
* flask.abort ()
* flask.redirect ()
* flask.make\_response ()
* flask.after\_this_\__request ()
* flask.send\_file ()
* flask.send\_from_\__directory ()
* flask.safe\_join ()
* flask.escape ()
* class flask.Markup

### Передача сообщения

* flask.flush ()
* flask.get\_flashed_\__messages ()

### Поддержка JSON

* flask.json.jsonify ()
* flask.json.dumps ()
* flask.json.dump ()
* flask.json.loads ()
* flask.json.load ()
* class flask.json.JSONDecoder

### JSON с тегами

* class flask.json.tag.TaggedJSONSerializer
* class flask.json.tag.JSONTag

### Рендеринг шаблонов

* flask.render\_template ()
* flask.render\_template_\__string ()
* flask.get\_template_\__attribute ()

### Конфигурация

* class flask.Config

### Потоковые помощники

* flask.stream\_with_\__context ()

### Полезные свойства

* class flask.ctx.RequestContext
* flask.\_request_\__ctx\_stack
* class flask.ctx.AppContext
* flask.\_app_\__ctx\_stack
* class flask.blueprints.BlueprintSetupState

### Сигналы

* signals.signals\_available
* flask.template\_rendered
* flask.before\_render_\__template
* flask.request\_started
* flask.request\_finished
* flask.got\_request_\__exception
* flask.request\_tearing_\__down
* flask.appcontext\_tearing_\__down
* flask.appcontext\_pushed
* flask.appcontext\_popped
* flask.message\_flashed
* class signals.Namespace

### Основанные на классах вьюхи

* class flask.views.View
* class flask.views.MethodView

### Регистрации маршрутов URL

### Опции функций вьюх

### Интерфейс командной строки

* class flask.cli.FlaskGroup
* class flask.cli.AppGroup
* class flask.cli.ScriptInfo
* flask.cli.load\_dotenv ()
* flask.cli.with\_appcontext ()
* flask.cli.pass\_script_\__info ()
* flask.cli.run\_command
* flask.cli.shell\_command
