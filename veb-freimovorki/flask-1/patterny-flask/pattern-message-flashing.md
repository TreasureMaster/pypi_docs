# Паттерн Message Flashing

Хорошие приложения и пользовательские интерфейсы основаны на обратной связи. Если пользователь не получит достаточной обратной связи, он, вероятно, в конечном итоге возненавидит приложение. **Flask** предоставляет действительно простой способ дать обратную связь пользователю с помощью системы **flashing**. Система **flashing** в основном позволяет записать сообщение в конце запроса и получить к нему доступ при следующем запросе и только при следующем запросе. Обычно это сочетается с шаблоном макета, который делает это. Обратите внимание, что браузеры, а иногда и веб-серверы устанавливают ограничение на размер файлов **cookie**. Это означает, что сообщения **flashing**, которые слишком велики для файлов **cookie** сеанса, приводят к тому, что **flashing** сообщений не выполняется без уведомления.

## Простой Flashing

Итак, вот полный пример:

```python
from flask import Flask, flash, redirect, render_template, \
     request, url_for

app = Flask(__name__)
app.secret_key = b'_5#y2L"F4Q8z\n\xec]/'

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/login', methods=['GET', 'POST'])
def login():
    error = None
    if request.method == 'POST':
        if request.form['username'] != 'admin' or \
                request.form['password'] != 'secret':
            error = 'Invalid credentials'
        else:
            flash('You were successfully logged in')
            return redirect(url_for('index'))
    return render_template('login.html', error=error)
```

А вот шаблон `layout.html`, который творит чудеса:

```markup
<!doctype html>
<title>My Application</title>
{% raw %}
{% with messages = get_flashed_messages() %}
  {% if messages %}
    <ul class=flashes>
    {% for message in messages %}
      <li>{{ message }}</li>
    {% endfor %}
    </ul>
  {% endif %}
{% endwith %}
{% block body %}{% endblock %}
{% endraw %}
```

Вот шаблон `index.html`, унаследованный от `layout.html`:

```markup
{% raw %}
{% extends "layout.html" %}
{% block body %}
  <h1>Overview</h1>
  <p>Do you want to <a href="{{ url_for('login') }}">log in?</a>
{% endblock %}
{% endraw %}
```

А вот шаблон `login.html`, который также наследуется от `layout.html`:

```markup
{% raw %}
{% extends "layout.html" %}
{% block body %}
  <h1>Login</h1>
  {% if error %}
    <p class=error><strong>Error:</strong> {{ error }}
  {% endif %}
  <form method=post>
    <dl>
      <dt>Username:
      <dd><input type=text name=username value="{{
          request.form.username }}">
      <dt>Password:
      <dd><input type=password name=password>
    </dl>
    <p><input type=submit value=Login>
  </form>
{% endblock %}
{% endraw %}
```

## Flashing с категориями

_Новое в версии 0.3_.

Также можно указать категории при отображении сообщения. Если ничего не указано, по умолчанию используется категория `'message'`. Можно использовать альтернативные категории, чтобы дать пользователю лучшую обратную связь. Например, сообщения об ошибках могут отображаться на красном фоне.

Чтобы высветить сообщение с другой категорией, просто используйте второй аргумент функции [flash ()](../api-dokumentaciya-flask/message-flashing.md#flask-flash):

```python
flash(u'Invalid password provided', 'error')
```

Затем внутри шаблона вы должны указать функции [get\_flashed\_messages ()](../api-dokumentaciya-flask/message-flashing.md#flask-get\_flashed\_messages), чтобы она также возвращала категории. В этой ситуации цикл выглядит немного иначе:

```markup
{% raw %}
{% with messages = get_flashed_messages(with_categories=true) %}
  {% if messages %}
    <ul class=flashes>
    {% for category, message in messages %}
      <li class="{{ category }}">{{ message }}</li>
    {% endfor %}
    </ul>
  {% endif %}
{% endwith %}
{% endraw %}
```

Это всего лишь один пример того, как отображать эти мигающие сообщения. Можно также использовать категорию, чтобы добавить к сообщению префикс, например `<strong> Error: </strong>`.

## Фильтрация Flash сообщений

_Новое в версии 0.9_.

При желании вы можете передать список категорий, который фильтрует результаты [get\_flashed\_messages ()](../api-dokumentaciya-flask/message-flashing.md#flask-get\_flashed\_messages). Это полезно, если вы хотите отобразить каждую категорию в отдельном блоке.

```markup
{% raw %}
{% with errors = get_flashed_messages(category_filter=["error"]) %}
{% if errors %}
<div class="alert-message block-message error">
  <a class="close" href="#">×</a>
  <ul>
    {%- for msg in errors %}
    <li>{{ msg }}</li>
    {% endfor -%}
  </ul>
</div>
{% endif %}
{% endwith %}
{% endraw %}
```
