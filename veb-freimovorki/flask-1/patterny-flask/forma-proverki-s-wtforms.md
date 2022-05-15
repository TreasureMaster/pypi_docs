# Форма проверки с WTForms

Когда вам нужно работать с данными формы, отправленными из представления браузера, код быстро становится очень трудно читать. Существуют библиотеки, призванные упростить управление этим процессом. Один из них - это [WTForms](https://wtforms.readthedocs.io/en/2.3.x/), с которыми мы здесь и займемся. Если вы оказались в ситуации, когда у вас много форм, вы можете попробовать.

Когда вы работаете с **WTForms**, вы должны сначала определить свои формы как классы. Я рекомендую разбить приложение на несколько модулей ([большие приложения](bolshie-prilozheniya-flask.md)) для этого и добавить отдельный модуль для форм.

{% hint style="info" %}
**Получение максимальной отдачи от WTForms с расширением:**

Расширение [Flask-WTF](https://flask-wtf.readthedocs.io/en/stable/) расширяет этот шаблон и добавляет несколько небольших помощников, которые делают работу с формами и **Flask** более увлекательной. Вы можете получить его из [PyPI](https://pypi.org/project/Flask-WTF/).
{% endhint %}

## Формы

Это пример формы для типичной страницы регистрации:

```python
from wtforms import Form, BooleanField, StringField, PasswordField, validators

class RegistrationForm(Form):
    username = StringField('Username', [validators.Length(min=4, max=25)])
    email = StringField('Email Address', [validators.Length(min=6, max=35)])
    password = PasswordField('New Password', [
        validators.DataRequired(),
        validators.EqualTo('confirm', message='Passwords must match')
    ])
    confirm = PasswordField('Repeat Password')
    accept_tos = BooleanField('I accept the TOS', [validators.DataRequired()])
```

## В представлении

В функции просмотра использование этой формы выглядит так:

```python
@app.route('/register', methods=['GET', 'POST'])
def register():
    form = RegistrationForm(request.form)
    if request.method == 'POST' and form.validate():
        user = User(form.username.data, form.email.data,
                    form.password.data)
        db_session.add(user)
        flash('Thanks for registering')
        return redirect(url_for('login'))
    return render_template('register.html', form=form)
```

Обратите внимание, мы подразумеваем, что здесь представление использует SQLAlchemy ([SQLAlchemy во Flask](sqlalchemy-v-flask.md)), но это, конечно, не является обязательным требованием. При необходимости скорректируйте код.

То, что нужно запомнить:

1. создать форму из значения **form** запроса, если данные отправляются с помощью метода HTTP **POST**, и аргументов, если данные отправляются как **GET**.
2. для проверки данных вызовите метод **validate ()**, который вернет `True`, если данные подтвердятся, и `False` в противном случае.
3. для доступа к отдельным значениям из формы перейдите к `form.<NAME>.data`.

## Формы в шаблонах

Теперь о шаблоне. Когда вы передаете форму в шаблоны, вы можете легко отобразить их там. Посмотрите на следующий пример шаблона, чтобы увидеть, насколько это просто. **WTForms** уже делает за нас половину генерации форм. Чтобы сделать это еще лучше, мы можем написать макрос, который отображает поле с меткой и списком ошибок, если таковые имеются.

Вот пример шаблона `_formhelpers.html` с таким макросом:

```python
{% raw %}
{% macro render_field(field) %}
  <dt>{{ field.label }}
  <dd>{{ field(**kwargs)|safe }}
  {% if field.errors %}
    <ul class=errors>
    {% for error in field.errors %}
      <li>{{ error }}</li>
    {% endfor %}
    </ul>
  {% endif %}
  </dd>
{% endmacro %}
{% endraw %}
```

Этот макрос принимает несколько аргументов ключевого слова, которые передаются в функцию поля **WTForms**, которая отображает поле для нас. Аргументы ключевого слова будут вставлены как атрибуты HTML. Так, например, вы можете вызвать `render_field (form.username, class = 'username')`, чтобы добавить класс к элементу ввода. Обратите внимание, что **WTForms** возвращает стандартные строки Unicode Python, поэтому мы должны сообщить **Jinja2**, что эти данные уже экранированы в HTML с помощью фильтра `|safe`.

Вот шаблон `register.html` для функции, которую мы использовали выше, которая использует преимущества шаблона `_formhelpers.html`:

```python
{% raw %}
{% from "_formhelpers.html" import render_field %}
{% endraw %}
<form method=post>
  <dl>
    {{ render_field(form.username) }}
    {{ render_field(form.email) }}
    {{ render_field(form.password) }}
    {{ render_field(form.confirm) }}
    {{ render_field(form.accept_tos) }}
  </dl>
  <p><input type=submit value=Register>
</form>
```

Для получения дополнительной информации о **WTForms** перейдите на [сайт WTForms](https://wtforms.readthedocs.io/en/2.3.x/).
