# Подключаемые представления Flask

_Новое в версии 0.7_.

`Flask 0.7` представляет подключаемые представления, вдохновленные общими представлениями **Django**, которые основаны на классах, а не на функциях. Основная цель состоит в том, чтобы вы могли заменять части реализаций и, таким образом, иметь настраиваемые подключаемые представления.

## Базовые принципы

Предположим, у вас есть функция, которая загружает список объектов из базы данных и отображает их в шаблоне:

```python
@app.route('/users/')
def show_users(page):
    users = User.query.all()
    return render_template('users.html', users=users)
```

Это просто и гибко, но если вы хотите предоставить это представление в общем виде, которое может быть адаптировано к другим моделям и шаблонам, вам может потребоваться большая гибкость. Здесь и появляются подключаемые представления на основе классов. В качестве первого шага к преобразованию этого в представление на основе классов вы должны сделать следующее:

```python
from flask.views import View

class ShowUsers(View):

    def dispatch_request(self):
        users = User.query.all()
        return render_template('users.html', objects=users)

app.add_url_rule('/users/', view_func=ShowUsers.as_view('show_users'))
```

Как видите, вам нужно создать подкласс [flask.views.View](../api-dokumentaciya-flask/predstavleniya-flask-osnovannye-na-klassakh.md#klass-flask-views-view) и реализовать [dispatch\_request ()](../api-dokumentaciya-flask/predstavleniya-flask-osnovannye-na-klassakh.md#dispatch\_request). Затем мы должны преобразовать этот класс в фактическую функцию просмотра с помощью метода класса [as\_view ()](../api-dokumentaciya-flask/predstavleniya-flask-osnovannye-na-klassakh.md#classmethod-as\_view). Строка, которую вы передаете этой функции, - это имя конечной точки, которую затем будет иметь представление. Но это само по себе бесполезно, поэтому давайте немного реорганизуем код:

```python
from flask.views import View

class ListView(View):

    def get_template_name(self):
        raise NotImplementedError()

    def render_template(self, context):
        return render_template(self.get_template_name(), **context)

    def dispatch_request(self):
        context = {'objects': self.get_objects()}
        return self.render_template(context)

class UserView(ListView):

    def get_template_name(self):
        return 'users.html'

    def get_objects(self):
        return User.query.all()
```

Это, конечно, не очень полезно для такого небольшого примера, но достаточно хорошо, чтобы объяснить основной принцип. Когда у вас есть классовое представление, возникает вопрос, на что указывает **self**. Это работает так: всякий раз, когда отправляется запрос, создается новый экземпляр класса и вызывается метод [dispatch\_request ()](../api-dokumentaciya-flask/predstavleniya-flask-osnovannye-na-klassakh.md#dispatch\_request) с параметрами из правила URL. Сам класс создается с параметрами, переданными в функцию [as\_view ()](../api-dokumentaciya-flask/predstavleniya-flask-osnovannye-na-klassakh.md#classmethod-as\_view). Например, вы можете написать такой класс:

```python
class RenderTemplateView(View):
    def __init__(self, template_name):
        self.template_name = template_name
    def dispatch_request(self):
        return render_template(self.template_name)
```

И тогда вы можете зарегистрировать это так:

```python
app.add_url_rule('/about', view_func=RenderTemplateView.as_view(
    'about_page', template_name='about.html'))
```

## Подсказки метода

Подключаемые представления прикрепляются к приложению, как обычная функция, с помощью метода [route ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#route) или, лучше, [add\_url\_rule ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#add\_url\_rule). Однако это также означает, что вам нужно будет указать имена HTTP-методов, поддерживаемых представлением, когда вы его прикрепите. Чтобы переместить эту информацию в класс, вы можете предоставить атрибут [methods](../api-dokumentaciya-flask/predstavleniya-flask-osnovannye-na-klassakh.md#methods), который содержит эту информацию:

```python
class MyView(View):
    methods = ['GET', 'POST']

    def dispatch_request(self):
        if request.method == 'POST':
            ...
        ...

app.add_url_rule('/myview', view_func=MyView.as_view('myview'))
```

## Диспетчеризация на основе метода

Для **RESTful API** особенно полезно выполнять разные функции для каждого метода HTTP. Это легко сделать с помощью [flask.views.MethodView](../api-dokumentaciya-flask/predstavleniya-flask-osnovannye-na-klassakh.md#klass-flask-views-methodview). Каждый метод HTTP сопоставляется с функцией с тем же именем (только в нижнем регистре):

```python
from flask.views import MethodView

class UserAPI(MethodView):

    def get(self):
        users = User.query.all()
        ...

    def post(self):
        user = User.from_form_data(request.form)
        ...

app.add_url_rule('/users/', view_func=UserAPI.as_view('users'))
```

Таким образом, вам также не нужно указывать атрибут [methods](../api-dokumentaciya-flask/predstavleniya-flask-osnovannye-na-klassakh.md#methods). Он автоматически устанавливается на основе методов, определенных в классе.

## Декорирование представлений

Поскольку сам класс представления не является функцией представления, добавляемой в систему маршрутизации, не имеет особого смысла декорировать сам класс. Вместо этого вам либо нужно вручную декорировать возвращаемое значение [as\_view ()](../api-dokumentaciya-flask/predstavleniya-flask-osnovannye-na-klassakh.md#classmethod-as\_view):

```python
def user_required(f):
    """Проверяет, вошел ли пользователь в систему, или выдает ошибку 401."""
    def decorator(*args, **kwargs):
        if not g.user:
            abort(401)
        return f(*args, **kwargs)
    return decorator

view = user_required(UserAPI.as_view('users'))
app.add_url_rule('/users/', view_func=view)
```

Начиная с `Flask 0.8` существует также альтернативный способ, в котором вы можете указать список декораторов для применения в объявлении класса:

```python
class UserAPI(MethodView):
    decorators = [user_required]
```

Из-за неявного **self** с точки зрения вызывающего, вы не можете использовать обычные декораторы представления для отдельных методов представления, однако имейте это в виду.

## Представления методов для API

Веб-API-интерфейсы часто очень тесно взаимодействуют с HTTP-командами, поэтому имеет смысл реализовать такой API-интерфейс на основе [MethodView](../api-dokumentaciya-flask/predstavleniya-flask-osnovannye-na-klassakh.md#klass-flask-views-methodview). Тем не менее, вы заметите, что API потребует разные правила URL, которые большую часть времени обращаются к одному и тому же представлению метода. Например, представьте, что вы открываете объект пользователя в сети:

| URL              | Метод  | Описание                         |
| ---------------- | ------ | -------------------------------- |
| **/users/**      | GET    | Выдает список всех пользователей |
| **/users/**      | POST   | Создает нового пользователя      |
| **/users/\<id>** | GET    | Показывает одного пользователя   |
| **/users/\<id>** | POST   | Обновляет одного пользователя    |
| **/users/\<id>** | DELETE | Удаляет одного пользователя      |

Итак, как бы вы это сделали с помощью [MethodView](../api-dokumentaciya-flask/predstavleniya-flask-osnovannye-na-klassakh.md#klass-flask-views-methodview)? Уловка состоит в том, чтобы воспользоваться тем фактом, что вы можете предоставить несколько правил для одного и того же представления.

Предположим, что представление будет выглядеть так:

```python
class UserAPI(MethodView):

    def get(self, user_id):
        if user_id is None:
            # вернуть список пользователей
            pass
        else:
            # показать одного пользователя
            pass

    def post(self):
        # создать нового пользователя
        pass

    def delete(self, user_id):
        # удалить одного пользователя
        pass

    def put(self, user_id):
        # обновить одного пользователя
        pass
```

Так как же нам связать это с системой маршрутизации? Добавив два правила и явно указав методы для каждого:

```python
user_view = UserAPI.as_view('user_api')
app.add_url_rule('/users/', defaults={'user_id': None},
                 view_func=user_view, methods=['GET',])
app.add_url_rule('/users/', view_func=user_view, methods=['POST',])
app.add_url_rule('/users/<int:user_id>', view_func=user_view,
                 methods=['GET', 'PUT', 'DELETE'])
```

Если у вас много похожих API, вы можете реорганизовать этот регистрационный код:

```python
def register_api(view, endpoint, url, pk='id', pk_type='int'):
    view_func = view.as_view(endpoint)
    app.add_url_rule(url, defaults={pk: None},
                     view_func=view_func, methods=['GET',])
    app.add_url_rule(url, view_func=view_func, methods=['POST',])
    app.add_url_rule('%s<%s:%s>' % (url, pk_type, pk), view_func=view_func,
                     methods=['GET', 'PUT', 'DELETE'])

register_api(UserAPI, 'user_api', '/users/', pk='user_id')
```
