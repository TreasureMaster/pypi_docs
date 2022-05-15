# Ускоренный курс

Итак, вы сломали костяшки пальцев и начали работать над тем замечательным веб-приложением на Python, которое хотите написать. Вы написали несколько страниц, и, наконец, вам нужно заняться этой отвратительной задачей: обработкой и проверкой ввода формы. Войдите в **WTForms**.

Но зачем мне еще один фреймворк? Что ж, в некоторых фреймворках веб-приложений используется подход связывания моделей баз данных с обработкой форм. Хотя это может быть удобно для очень простых представлений создания/обновления, не все формы, которые вам нужны, можно сопоставить непосредственно с моделью базы данных. Или, может быть, вы уже используете общую структуру обработки форм, но хотите настроить создание HTML этих полей формы и определить собственную проверку.

С помощью **WTForms** HTML вашего поля формы может быть сгенерирован для вас, но мы позволяем вам настраивать его в ваших шаблонах. Это позволяет сохранить разделение кода и представления и исключить эти беспорядочные параметры из кода Python. Поскольку мы стремимся к слабой взаимосвязи, вы также сможете сделать это в любом движке шаблонов, который вам нравится.

### Скачать / Установка

**WTForms** доступен через PyPI. Установите его с помощью pip:

```python
pip install WTForms
```

### Ключевые понятия

* ****[**Forms**](../api/forms-formy.md) - это основной контейнер **WTForms**. Формы представляют собой набор полей, к которым можно получить доступ в стиле словаря или стиля атрибута.
* ****[**Fields**](../api/fields-polya.md) - делают большую часть тяжелой работы. Каждое поле представляет тип данных, и поле обрабатывает ввод формы для принуждения к этому типу данных. Например, **IntegerField** и **StringField** представляют два разных типа данных. Поля содержат ряд полезных свойств, таких как метка, описание и список ошибок проверки, в дополнение к данным, содержащимся в поле.
* Каждое поле имеет экземпляр виджета **Widget**. Задача виджета - визуализировать HTML-представление этого поля. Экземпляры виджетов могут быть указаны для каждого поля, но каждое поле имеет один по умолчанию, что имеет смысл. Некоторые поля просто удобны, например [**TextAreaField**](../api/fields-polya.md#osnovnye-polya) - это просто [**StringField**](../api/fields-polya.md#osnovnye-polya), а виджетом по умолчанию является [**TextArea**](../api/widgets-vidzhety.md#vstroennye-vidzhety).
* Чтобы указать правила проверки, поля содержат список валидаторов [**Validators**](../api/validators-validatory.md).

### Начало работы

Давайте приступим к делу и определим нашу первую форму:

```python
from wtforms import Form, BooleanField, StringField, validators

class RegistrationForm(Form):
    username     = StringField('Username', [validators.Length(min=4, max=25)])
    email        = StringField('Email Address', [validators.Length(min=6, max=35)])
    accept_rules = BooleanField('I accept the site rules', [validators.InputRequired()])
```

Когда вы создаете форму, вы определяете поля способом, аналогичным тому, как многие ORM определяют свои столбцы: путем определения переменных класса, которые являются экземплярами полей.

Поскольку формы являются обычными классами Python, вы можете легко расширить их, как и следовало ожидать:

```python
class ProfileForm(Form):
    birthday  = DateTimeField('Your Birthday', format='%m/%d/%y')
    signature = TextAreaField('Forum Signature')

class AdminProfileForm(ProfileForm):
    username = StringField('Username', [validators.Length(max=40)])
    level    = IntegerField('User Level', [validators.NumberRange(min=0, max=10)])
```

Посредством создания подклассов **AdminProfileForm** получает все поля, уже определенные в **ProfileForm**. Это позволяет вам легко использовать общие подмножества полей между формами, как в примере выше, где мы добавляем в **ProfileForm** поля только для администратора.

### Использование форм

Использовать форму так же просто, как создать ее экземпляр. Рассмотрим следующий django-подобный вид, используя **RegistrationForm**, который мы определили ранее:

```python
def register(request):
    form = RegistrationForm(request.POST)
    if request.method == 'POST' and form.validate():
        user = User()
        user.username = form.username.data
        user.email = form.email.data
        user.save()
        redirect('register')
    return render_response('register.html', form=form)
```

Сначала мы создаем экземпляр формы, предоставляя ей любые данные, доступные в `request.POST`. Затем мы проверяем, сделан ли запрос с помощью POST, и если да, мы проверяем форму и проверяем, принял ли пользователь правила. В случае успеха мы создаем нового пользователя, назначаем ему данные из проверенной формы и сохраняем его.

### Редактирование существующих объектов

Наш предыдущий пример регистрации показал, как принимать ввод и проверять его на наличие новых записей, но что, если мы хотим отредактировать существующий объект? Легко:

```python
def edit_profile(request):
    user = request.current_user
    form = ProfileForm(request.POST, user)
    if request.method == 'POST' and form.validate():
        form.populate_obj(user)
        user.save()
        redirect('edit_profile')
    return render_response('edit_profile.html', form=form)
```

Здесь мы создаем экземпляр формы, предоставляя форме как `request.POST`, так и объект пользователя. Таким образом, форма получит все данные, которых нет в данных публикации, от объекта пользователя _**user**_.

Мы также используем метод [_**populate\_obj**_](../api/forms-formy.md#klass-form-form) формы, чтобы повторно заполнить объект пользователя содержимым проверенной формы. Этот метод предоставляется для удобства, когда имена полей совпадают с именами объекта, который вы предоставляете с данными. Обычно вам нужно назначать значения вручную, но для этого простого случая это идеально. Это также может быть полезно для CRUD и административных форм.

### Изучение в консоли

Формы **WTForms** - это очень простые объекты-контейнеры, и, возможно, самый простой способ узнать, что вам доступно в форме, - это поиграться с формой в консоли:

```python
>>> from wtforms import Form, StringField, validators
>>> class UsernameForm(Form):
...     username = StringField('Username', [validators.Length(min=5)], default=u'test')
...
>>> form = UsernameForm()
>>> form['username']
<wtforms.fields.StringField object at 0x827eccc>
>>> form.username.data
u'test'
>>> form.validate()
False
>>> form.errors
{'username': [u'Field must be at least 5 characters long.']}
```

Мы обнаружили, что когда вы создаете форму, она содержит экземпляры всех полей, к которым можно получить доступ через стиль словаря или стиль атрибута. У этих полей есть свои свойства, как и у окружающей формы.

Когда мы проверяем форму, она возвращает `False`, что означает, что по крайней мере один валидатор не удовлетворен. `form.errors` предоставит вам сводку всех ошибок.

```python
>>> form2 = UsernameForm(username=u'Robert')
>>> form2.data
{'username': u'Robert'}
>>> form2.validate()
True
```

На этот раз мы передали новое значение для имени пользователя при создании экземпляра **UserForm**, и этого было достаточно для проверки формы.

### Как формы получают данные

Помимо предоставления данных с использованием первых двух аргументов (_**formdata**_ и _**obj**_), вы можете передавать аргументы ключевого слова для заполнения формы. Обратите внимание, что некоторые имена зарезервированы: _**formdata**_, _**obj**_ и _**prefix**_.

Данные формы _**formdata**_ имеют приоритет над _**obj**_, который сам по себе имеет приоритет над аргументами ключевого слова. Например:

```python
def change_username(request):
    user = request.current_user
    form = ChangeUsernameForm(request.POST, user, username='silly')
    if request.method == 'POST' and form.validate():
        user.username = form.username.data
        user.save()
        return redirect('change_username')
    return render_response('change_username.html', form=form)
```

Хотя на практике вы почти никогда не используете все три метода вместе, он иллюстрирует, как **WTForms** ищет поле имени пользователя _**username**_:

1. Если форма была отправлена (`request.POST` не пустой), будет обработан ввод формы. Даже если не было ввода формы, в частности, для этого поля, если существует ввод формы любого вида, то мы обработаем ввод формы.
2. Если не было ввода формы, будет сделано следующее по порядку:
   1. Проверяет, есть ли у пользователя _**user**_ атрибут с именем _**username**_.
   2. Проверяет, был ли указан аргумент ключевого слова с именем пользователя _**username**_.
   3. Наконец, если все остальное не удается, используется значение по умолчанию, указанное в поле, если оно есть.

### Валидаторы

Валидация в **WTForms** выполняется путем предоставления поля с набором валидаторов, запускаемых, когда содержимое формы проверяется. Вы предоставляете их через второй аргумент конструктора поля, валидаторы _**validators**_:

```python
class ChangeEmailForm(Form):
    email = StringField('Email', [validators.Length(min=6, max=120), validators.Email()])
```

Вы можете указать любое количество валидаторов для поля. Как правило, вы хотите предоставить собственное сообщение об ошибке:

```python
class ChangeEmailForm(Form):
    email = StringField('Email', [
        validators.Length(min=6, message=_(u'Little short for an email address?')),
        validators.Email(message=_(u'That\'s not a valid email address.'))
    ])
```

Обычно предпочтительно предоставлять свои собственные сообщения, поскольку сообщения по умолчанию по необходимости являются общими. Это также способ предоставления локализованных сообщений об ошибках.

Список всех встроенных валидаторов см. в [справочнике по валидаторам](../api/validators-validatory.md).

### Рендеринг полей

Визуализировать поле так же просто, как преобразовать его в строку:

```python
>>> from wtforms import Form, StringField
>>> class SimpleForm(Form):
...   content = StringField('content')
...
>>> form = SimpleForm(content='foobar')
>>> str(form.content)
'<input id="content" name="content" type="text" value="foobar" />'
>>> unicode(form.content)
u'<input id="content" name="content" type="text" value="foobar" />'
```

Однако реальная мощь исходит от рендеринга поля с помощью метода `__call__()`. Вызывая поле, вы можете предоставить аргументы ключевого слова, которые будут вставлены как атрибуты html в вывод:

```python
>>> form.content(style="width: 200px;", class_="bar")
u'<input class="bar" id="content" name="content" style="width: 200px;" '
    'type="text" value="foobar" />'
```

Теперь давайте применим эту возможность для рендеринга формы в шаблоне **Jinja**. Во-первых, наша форма:

```python
class LoginForm(Form):
    username = StringField('Username')
    password = PasswordField('Password')

form = LoginForm()
```

И шаблон:

```python
<form method="POST" action="/login">
    <div>{{ form.username.label }}: {{ form.username(class="css_class") }}</div>
    <div>{{ form.password.label }}: {{ form.password() }}</div>
</form>
```

В качестве альтернативы, если вы используете шаблоны **Django**, вы можете использовать тег шаблона _**form\_field**_, который мы предоставляем в нашем расширении **Django**, когда вы хотите передать аргументы ключевого слова:

```python
{% raw %}
{% load wtforms %}
<form method="POST" action="/login">
    <div>
        {{ form.username.label }}:
        {% form_field form.username class="css_class" %}
{% endraw %}
    </div>
    <div>
        {{ form.password.label }}:
        {{ form.password }}
    </div>
</form>
```

Оба они выведут:

```markup
<form method="POST" action="/login">
    <div>
        <label for="username">Username</label>:
        <input class="css_class" id="username" name="username" type="text" value="" />
    </div>
    <div>
        <label for="password">Password</label>:
        <input id="password" name="password" type="password" value="" />
    </div>
</form>
```

**WTForms** не зависит от механизма шаблонов и будет работать со всем, что разрешает доступ к атрибутам, приведение строк и/или вызовы функций. Тег шаблона _**form\_field**_ предоставляется для удобства, так как вы не можете передавать аргументы в шаблонах **Django**.

### Отображение ошибок

Теперь, когда у нас есть шаблон для нашей формы, давайте добавим сообщения об ошибках:

```python
<form method="POST" action="/login">
    <div>{{ form.username.label }}: {{ form.username(class="css_class") }}</div>
    {% raw %}
{% if form.username.errors %}
        <ul class="errors">
            {% for error in form.username.errors %}<li>{{ error }}</li>{% endfor %}
        </ul>
    {% endif %}

    <div>{{ form.password.label }}: {{ form.password() }}</div>
    {% if form.password.errors %}
        <ul class="errors">
            {% for error in form.password.errors %}<li>{{ error }}</li>{% endfor %}
        </ul>
    {% endif %}
{% endraw %}
</form>
```

Если вы предпочитаете один большой список ошибок вверху, это тоже легко:

```python
{% raw %}
{% if form.errors %}
    <ul class="errors">
        {% for field_name, field_errors in form.errors|dictsort if field_errors %}
            {% for error in field_errors %}
                <li>{{ form[field_name].label }}: {{ error }}</li>
            {% endfor %}
        {% endfor %}
    </ul>
{% endif %}
{% endraw %}
```

Поскольку обработка ошибок может стать довольно многословным делом, предпочтительно использовать макросы **Jinja** (или аналогичные), чтобы уменьшить количество шаблонов в ваших шаблонах. ([пример](reshenie-konkretnykh-problem.md#rendering-oshibok))

### Пользовательские валидаторы

Есть два способа предоставить собственные валидаторы. Определив настраиваемый валидатор и используя его в поле:

```python
from wtforms.validators import ValidationError

def is_42(form, field):
    if field.data != 42:
        raise ValidationError('Must be 42')

class FourtyTwoForm(Form):
    num = IntegerField('Number', [is_42])
```

Или предоставив валидатор для конкретного поля в форме:

```python
class FourtyTwoForm(Form):
    num = IntegerField('Number')

    def validate_num(form, field):
        if field.data != 42:
            raise ValidationError(u'Must be 42')
```

Для более сложных валидаторов, которые принимают параметры, проверьте раздел [Пользовательские валидаторы](../api/validators-validatory.md#polzovatelskie-validatory).

### Следующие шаги

Ускоренный курс только что бегло рассмотрел, как вы можете начать использовать **WTForms** для обработки ввода и проверки формы в вашем приложении. Для получения дополнительной информации проверьте следующее:

* В документации **WTForms** есть [документация по API](../api/) для всей библиотеки.
* [Решение конкретных проблем](reshenie-konkretnykh-problem.md) может помочь вам решить конкретные проблемы интеграции с **WTForms** и другими фреймворками.
