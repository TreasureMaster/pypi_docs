# Представления Flask, основанные на классах

_Новое в версии 0.7_.

## Класс flask.views.View

Альтернативный способ использования функций просмотра. Подкласс должен реализовать [dispatch\_request ()](predstavleniya-flask-osnovannye-na-klassakh.md#dispatch\_request), который вызывается с аргументами представления из системы маршрутизации URL. Если предоставляются [methods](predstavleniya-flask-osnovannye-na-klassakh.md#methods), их не нужно явно передавать методу [add\_url\_rule ()](obekt-prilozheniya-flask.md#add\_url\_rule-rule-endpoint-none-view\_func-none-provide\_automatic\_options-none-options):

```python
class MyView(View):
    methods = ['GET']

    def dispatch_request(self, name):
        return 'Hello %s!' % name

app.add_url_rule('/hello/<name>', view_func=MyView.as_view('myview'))
```

Если вы хотите декорировать подключаемое представление, вам придется либо сделать это при создании функции представления (путем обертывания возвращаемого значения [as\_view ()](predstavleniya-flask-osnovannye-na-klassakh.md#classmethod-as\_view)), либо вы можете использовать атрибут [decorators](predstavleniya-flask-osnovannye-na-klassakh.md#decorators):

```python
class SecretView(View):
    methods = ['GET']
    decorators = [superuser_required]

    def dispatch_request(self):
        ...
```

Декораторы, хранящиеся в списке декораторов, применяются один за другим при создании функции просмотра. Обратите внимание, что вы _**не можете**_ использовать декораторы на основе классов, поскольку они будут украшать класс представления, а не сгенерированную функцию представления!

### _(classmethod)_ as\_view( _name_, _\*class\_args_, _\*\*class\_kwargs_ )

Преобразует класс в фактическую функцию просмотра, которую можно использовать с системой маршрутизации. Внутренне это генерирует функцию на лету, которая будет создавать экземпляр [View](predstavleniya-flask-osnovannye-na-klassakh.md#klass-flask-views-view) для каждого запроса и вызывать для него метод [dispatch\_request ()](predstavleniya-flask-osnovannye-na-klassakh.md#dispatch\_request).

Аргументы, переданные в **as\_view ()**, передаются конструктору класса.

### decorators _= ()_

_Новое в версии 0.8_.

Канонический способ декорировать представления на основе классов - декорировать возвращаемое значение [as\_view ()](predstavleniya-flask-osnovannye-na-klassakh.md#classmethod-as\_view-name-class\_args-class\_kwargs). Однако, это перемещает части логики из объявления класса в место, где он подключен к системе маршрутизации.

Вы можете разместить один или несколько декораторов в этом списке, и всякий раз, когда создается функция просмотра, результат автоматически оформляется.

### dispatch\_request()

Подклассы должны переопределить этот метод, чтобы реализовать фактический код функции просмотра. Этот метод вызывается со всеми аргументами из правила URL.

### methods _= None_

Список методов, которые может обрабатывать это представление.

### provide\_automatic\_options _= None_

Установка этого параметра отключает или принудительно включает автоматическую обработку параметров.

## Класс flask.views.MethodView

Представление на основе классов, которое отправляет методы запроса соответствующим методам класса. Например, если вы реализуете метод **get**, он будет использоваться для обработки запросов **GET**.

```python
class CounterAPI(MethodView):
    def get(self):
        return session.get('counter', 0)

    def post(self):
        session['counter'] = session.get('counter', 0) + 1
        return 'OK'

app.add_url_rule('/counter', view_func=CounterAPI.as_view('counter'))
```

### dispatch\_request( _\*args_, _\*\*kwargs_ )

Подклассы должны переопределить этот метод, чтобы реализовать фактический код функции просмотра. Этот метод вызывается со всеми аргументами из правила URL.
