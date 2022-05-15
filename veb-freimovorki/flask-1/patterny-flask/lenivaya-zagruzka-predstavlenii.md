# Ленивая загрузка представлений

**Flask** обычно используется с декораторами. Декораторы просты, и у вас есть URL-адрес прямо рядом с функцией, которая вызывается для этого конкретного URL-адреса. Однако у этого подхода есть обратная сторона: это означает, что весь ваш код, использующий декораторы, должен быть импортирован заранее, иначе **Flask** никогда не найдет вашу функцию.

Это может быть проблемой, если ваше приложение должно быстро импортировать. Возможно, это придется сделать в таких системах, как **Google App Engine** или других системах. Так что, если вы вдруг заметите, что ваше приложение переросло этот подход, вы можете вернуться к централизованному сопоставлению URL-адресов.

Система, которая позволяет иметь центральную карту URL-адресов, - это функция [add\_url\_rule ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#add\_url\_rule). Вместо использования декораторов у вас есть файл, который устанавливает приложение со всеми URL-адресами.

## Преобразование в централизованную карту URL-адресов

Представьте, что текущее приложение выглядит примерно так:

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def index():
    pass

@app.route('/user/<username>')
def user(username):
    pass
```

Тогда при централизованном подходе у вас будет один файл с представлениями (`views.py`), но без декоратора:

```python
def index():
    pass

def user(username):
    pass
```

И затем файл, который устанавливает приложение, которое сопоставляет функции с URL-адресами:

```python
from flask import Flask
from yourapplication import views
app = Flask(__name__)
app.add_url_rule('/', view_func=views.index)
app.add_url_rule('/user/<username>', view_func=views.user)
```

## Загрузка позже

Пока мы разделяем только представления и маршрутизацию, но модуль все еще загружен заранее. Хитрость заключается в том, чтобы фактически загрузить функцию просмотра по мере необходимости. Это может быть выполнено с помощью вспомогательного класса, который ведет себя так же, как функция, но внутренне импортирует реальную функцию при первом использовании:

```python
from werkzeug.utils import import_string, cached_property

class LazyView(object):

    def __init__(self, import_name):
        self.__module__, self.__name__ = import_name.rsplit('.', 1)
        self.import_name = import_name

    @cached_property
    def view(self):
        return import_string(self.import_name)

    def __call__(self, *args, **kwargs):
        return self.view(*args, **kwargs)
```

Здесь важно то, что `__module__` и `__name__` установлены правильно. Это используется **Flask** для внутренних целей, чтобы выяснить, как назвать правила URL-адресов, если вы сами не указываете имя для правила.

Затем вы можете определить свое центральное место для объединения представлений следующим образом:

```python
from flask import Flask
from yourapplication.helpers import LazyView
app = Flask(__name__)
app.add_url_rule('/',
                 view_func=LazyView('yourapplication.views.index'))
app.add_url_rule('/user/<username>',
                 view_func=LazyView('yourapplication.views.user'))
```

Вы можете дополнительно оптимизировать это с точки зрения количества нажатий клавиш, необходимых для написания этого, имея функцию, которая вызывает [add\_url\_rule ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#add\_url\_rule), добавляя префикс строки с именем проекта и точкой, а также путем обертывания **view\_func** в **LazyView** по мере необходимости.

```python
def url(import_name, url_rules=[], **options):
    view = LazyView('yourapplication.' + import_name)
    for url_rule in url_rules:
        app.add_url_rule(url_rule, view_func=view, **options)

# добавить единственный маршрут к просмотру индекса
url('views.index', ['/'])

# добавить два маршрута к одной конечной точке функции
url_rules = ['/user/','/user/<username>']
url('views.user', url_rules)
```

Следует иметь в виду, что обработчики запросов до и после должны находиться в файле, который импортирован заранее, чтобы правильно работать с первым запросом. То же самое касается любого оставшегося декоратора.
