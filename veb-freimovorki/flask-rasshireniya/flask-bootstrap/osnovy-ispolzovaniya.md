# Основы использования

### Основы использования

Чтобы начать работу, первым делом необходимо импортировать и загрузить расширение:

```python
from flask import Flask
from flask_bootstrap import Bootstrap

def create_app():
  app = Flask(__name__)
  Bootstrap(app)

  return app

# do something with app...
```

После загрузки новые шаблоны доступны для создания в ваших шаблонах.

### Пример приложения

Если вы хотите взглянуть на небольшой образец приложения, попробуйте [просмотреть его на github](https://github.com/mbr/flask-bootstrap/tree/master/sample\_app).

### Шаблоны

Создать новый шаблон на основе Bootstrap очень просто:

```python
{% raw %}
{% extends "bootstrap/base.html" %}
{% block title %}This is an example page{% endblock %}

{% block navbar %}
<div class="navbar navbar-fixed-top">
  <!-- ... -->
</div>
{% endblock %}

{% block content %}
  <h1>Hello, Bootstrap</h1>
{% endblock %}
{% endraw %}
```

Все, что вы делаете в дочерних шаблонах, основано на блоках. Некоторые блоки (например, **title**, **navbar** или **content**) являются «вспомогательными блоками». Строго говоря, в них нет необходимости, но они добавлены для экономии усилий при вводе текста.

Очень мощная функция - это функция `super ()` в [Jinja2](../../../shablonizatory/jinja/). Это дает вам возможность изменять блоки вместо их замены.

### Доступные блоки

| Имя блока       | Родительский блок | Назначение                                              |
| --------------- | ----------------- | ------------------------------------------------------- |
| _doc_           |                   | Самый внешний блок                                      |
| _html_          | _doc_             | Содержит полное содержимое тега `<html>`                |
| _html\_attribs_ | _doc_             | Атрибуты тега HTML                                      |
| _head_          | _doc_             | Содержит полное содержимое тега `<head>`                |
| _body_          | _doc_             | Содержит полное содержимое тега `<body>`                |
| _body\_attribs_ | _body_            | Атрибуты тега body                                      |
| _**title**_     | _head_            | Содержит полное содержимое тега `<title>`               |
| _**styles**_    | _head_            | Содержит все теги `<link>` в стиле CSS внутри заголовка |
| _metas_         | _head_            | Содержит все теги `<meta>` внутри head                  |
| _**navbar**_    | _body_            | Пустой блок прямо над содержимым                        |
| _**content**_   | _body_            | Блок комфорта внутри корпуса. Положи сюда вещи.         |
| _**scripts**_   | _body_            | Содержит все теги `<script>` в конце body               |

### Примеры

Добавление собственного файла CSS:

```python
{% raw %}
{% block styles %}
{{super()}}
<link rel="stylesheet"
      href="{{url_for('.static', filename='mystyle.css')}}">
{% endblock %}
{% endraw %}
```

Пользовательский Javascript загружается до кода JavaScript Bootstrap:

```python
{% raw %}
{% block scripts %}
<script src="{{url_for('.static', filename='myscripts.js')}}"></script>
{{super()}}
{% endblock %}
{% endraw %}
```

Добавление атрибута `lang = "en"` к тегу `<html>`:

```python
{% raw %}
{% block html_attribs %} lang="en"{% endblock %}
{% endraw %}
```

### Статические ресурсы

URL-адрес `bootstrap.static` доступен для ссылки на ресурсы Bootstrap, но обычно в этом нет необходимости. Немного лучше использовать фильтр шаблона `bootstrap_find_resource`, который учитывает настройки CDN.

Текущая система ресурсов подробно описана по [поддержке CDN](podderzhka-cdn.md).
