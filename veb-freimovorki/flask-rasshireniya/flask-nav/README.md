# Flask-Nav

**Flask-Nav** - это расширение **Flask**, упрощающее создание элементов навигации в приложениях. Он предоставляет средства для выражения структуры навигации и различные способы их визуализации, что упрощает ее адаптацию для вашего приложения.

Мотивирующий пример:

```python
from flask import Flask, render_template
from flask_nav import Nav
from flask_nav.elements import *

nav = Nav()

# registers the "top" menubar
nav.register_element('top', Navbar(
    View('Widgits, Inc.', 'index'),
    View('Our Mission', 'about'),
    Subgroup(
        'Products',
        View('Wg240-Series', 'products', product='wg240'),
        View('Wg250-Series', 'products', product='wg250'),
        Separator(),
        Label('Discontinued Products'),
        View('Wg10X', 'products', product='wg10x'),
    ),
    Link('Tech Support', href='http://techsupport.invalid/widgits_inc'),
))


app = Flask(__name__)
# [...] (view definitions)

nav.init_app(app)
```

Вы можете найти небольшой исполняемый пример приложения в папке с примерами. Чтобы запустить его, установите [Flask-Appconfig](https://github.com/mbr/flask-appconfig) и выполните:

```bash
$ flask --app=example dev
```

Полную документацию можно найти на PyPI.

## Документация

### Начало работы

* Регистрация бара
* Рендеринг навигационной панели

### Дополнительные темы

* Рендереры (Renderers)
* Элементы (Elements)
* Динамическая конструкция

### Другое программное обеспечение

* Flask-Navigation
* Flask-Bootstrap

### Справочник по API
