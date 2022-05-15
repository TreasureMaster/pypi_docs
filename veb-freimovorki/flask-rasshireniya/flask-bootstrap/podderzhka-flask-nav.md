# Поддержка Flask-Nav

## Поддержка Flask-Nav

Расширение [Flask-Nav](https://pythonhosted.org/flask-nav/) позволяет легко создавать структуры навигации, и **Flask-Bootstrap** поставляется с совместимым с **Bootstrap** средством визуализации для них. После инициализации приложения **Flask-Bootstrap** зарегистрирует средство визуализации **Bootstrap** по умолчанию.

Например, рендеринг навигационной панели «просто работает»:

```python
{% raw %}
{% block navbar %}
{{nav.mynavbar.render()}}
{% endblock %}
{% endraw %}
```

и автоматически выдаст HTML, совместимый с **Bootstrap**. Минимальный пример создания рабочей панели навигации:

```python
from flask_nav import Nav
from flask_nav.elements import Navbar, View

nav = Nav()

@nav.navigation()
def mynavbar():
    return Navbar(
        'mysite',
        View('Home', 'index'),
    )

# ...

nav.init_app(app)
```

См. образец приложения для получения более подробного примера навигации.

### Рендерер BootstrapRenderer

Средство визуализации для Bootstrap-специфичного HTML (доступного как `flask_bootstrap.nav.BootstrapRenderer`) имеет несколько специфических функций. А именно, атрибут заголовка `title` любой панели навигации **Navbar** также может быть ссылкой **Link** или представлением **View**.

Заголовок `title`, если не `None`, будет отображаться с использованием классов бренда `brand` (подробности см. в [документации Bootstrap](https://getbootstrap.com/)), и если у него есть метод `get_url`, возвращаемое значение будет ссылкой на текст бренда.

### Настройка Navbar

Чтобы изменить вывод **BootstrapRenderer**, можно создать подкласс и зарегистрировать полученный дочерний класс в качестве другого средства визуализации. См. документацию [Flask-Nav](https://pythonhosted.org/flask-nav/) для получения [дополнительной информации](https://pythonhosted.org/flask-nav/advanced-topics.html#implementing-custom-renderers) по этой теме.
