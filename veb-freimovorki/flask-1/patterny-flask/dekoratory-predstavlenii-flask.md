# Декораторы представлений Flask

В Python есть действительно интересная функция, называемая декораторами функций. Это позволяет создавать действительно полезные вещи для веб-приложений. Поскольку каждое представление во **Flask** - это функция, декораторы можно использовать для добавления дополнительных функций к одной или нескольким функциям. Декоратор [route ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#route) - это тот, который вы, вероятно, уже использовали. Но есть варианты использования для реализации собственного декоратора. Например, представьте, что у вас есть представление, которое должно использоваться только людьми, которые вошли в систему. Если пользователь переходит на сайт и не вошел в систему, он должен быть перенаправлен на страницу входа. Это хороший пример использования, где декоратор - отличное решение.

## Декоратор Login Required

Итак, давайте реализуем такой декоратор. Декоратор - это функция, которая обертывает и заменяет другую функцию. Поскольку исходная функция заменяется, вам нужно не забыть скопировать информацию исходной функции в новую функцию. Используйте [functools.wraps ()](https://docs.python.org/3/library/functools.html#functools.wraps), чтобы сделать это за вас.

В этом примере предполагается, что страница входа называется `'login'` и что текущий пользователь хранится в `g.user` и имеет значение `None`, если никто не вошел в систему.

```python
from functools import wraps
from flask import g, request, redirect, url_for

def login_required(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        if g.user is None:
            return redirect(url_for('login', next=request.url))
        return f(*args, **kwargs)
    return decorated_function
```

Чтобы использовать декоратор, примените его как самый внутренний декоратор к функции просмотра. При применении дополнительных декораторов всегда помните, что декоратор [route ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#route) является самым внешним.

```python
@app.route('/secret_page')
@login_required
def secret_page():
    pass
```

{% hint style="info" %}
Значение **next** будет существовать в **request.args** после запроса **GET** для страницы входа. Вам нужно будет передать его при отправке запроса **POST** из формы входа. Вы можете сделать это с помощью скрытого тега ввода, а затем получить его из **request.form** при входе пользователя в систему.

```markup
<input type="hidden" value="{{ request.args.get('next', '') }}"/>
```
{% endhint %}

## Декоратор кеширования

Представьте, что у вас есть функция просмотра, которая выполняет дорогостоящие вычисления, и из-за этого вы хотите кэшировать сгенерированные результаты на определенное время. Для этого подойдет декоратор. Мы предполагаем, что вы настроили кеш, как описано в разделе «[Кэширование](keshirovanie-flask.md)».

Вот пример функции кеширования. Он генерирует ключ кеша из определенного префикса (фактически, строки формата) и текущего пути запроса. Обратите внимание, что мы используем функцию, которая сначала создает декоратор, который затем украшает функцию. Ужасно звучит? К сожалению, это немного сложнее, но код все же должен быть простым для чтения.

Декорированная функция будет работать следующим образом

1. получить уникальный ключ кеша для текущего запроса на основе текущего пути.
2. получить значение этого ключа из кеша. Если кеш что-то вернул, мы вернем это значение.
3. в противном случае вызывается исходная функция, а возвращаемое значение сохраняется в кеше в течение заданного времени ожидания (по умолчанию 5 минут).

Вот код:

```python
from functools import wraps
from flask import request

def cached(timeout=5 * 60, key='view/%s'):
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            cache_key = key % request.path
            rv = cache.get(cache_key)
            if rv is not None:
                return rv
            rv = f(*args, **kwargs)
            cache.set(cache_key, rv, timeout=timeout)
            return rv
        return decorated_function
    return decorator
```

Обратите внимание, что это предполагает, что экземпляр объекта **cache** доступен, см. [Кэширование](keshirovanie-flask.md) для получения дополнительной информации.

## Декоратор шаблона

Обычный шаблон, изобретенный ребятами из **TurboGears** недавно, - это декоратор шаблонов. Идея этого декоратора заключается в том, что вы возвращаете словарь со значениями, переданными в шаблон из функции просмотра, и шаблон автоматически отображается. При этом следующие три примера делают то же самое:

```python
@app.route('/')
def index():
    return render_template('index.html', value=42)

@app.route('/')
@templated('index.html')
def index():
    return dict(value=42)

@app.route('/')
@templated()
def index():
    return dict(value=42)
```

Как видите, если имя шаблона не указано, он будет использовать конечную точку карты URL с точками, преобразованными в косую черту + `'.html'`. В противном случае используется указанное имя шаблона. Когда декорированная функция возвращается, возвращенный словарь передается в функцию отрисовки шаблона. Если возвращается `None`, предполагается пустой словарь, если возвращается что-то еще, кроме словаря, мы возвращаем его из функции без изменений. Таким образом, вы по-прежнему можете использовать функцию перенаправления или возвращать простые строки.

Вот код этого декоратора:

```python
from functools import wraps
from flask import request, render_template

def templated(template=None):
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            template_name = template
            if template_name is None:
                template_name = request.endpoint \
                    .replace('.', '/') + '.html'
            ctx = f(*args, **kwargs)
            if ctx is None:
                ctx = {}
            elif not isinstance(ctx, dict):
                return ctx
            return render_template(template_name, **ctx)
        return decorated_function
    return decorator
```

## Декоратор конечной точки

Если вы хотите использовать систему маршрутизации **Werkzeug** для большей гибкости, вам необходимо сопоставить конечную точку, как определено в [Rule](https://werkzeug.palletsprojects.com/en/1.0.x/routing/#werkzeug.routing.Rule), с функцией просмотра. Это возможно с помощью этого декоратора. Например:

```python
from flask import Flask
from werkzeug.routing import Rule

app = Flask(__name__)
app.url_map.add(Rule('/', endpoint='index'))

@app.endpoint('index')
def my_index():
    return "Hello world"
```
