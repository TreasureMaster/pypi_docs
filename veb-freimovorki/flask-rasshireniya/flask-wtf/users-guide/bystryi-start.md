# Быстрый старт

Хотите начать? Эта страница дает хорошее представление о **Flask-WTF**. Предполагается, что у вас уже установлен **Flask-WTF**. Если вы этого не сделаете, перейдите в раздел «[Установка](ustanovka.md)».

### Создание форм

**Flask-WTF** обеспечивает интеграцию вашего приложения **Flask** с **WTForms**. Например:

```python
from flask_wtf import FlaskForm
from wtforms import StringField
from wtforms.validators import DataRequired

class MyForm(FlaskForm):
    name = StringField('name', validators=[DataRequired()])
```

{% hint style="info" %}
**Примечание:**

Начиная с версии 0.9.0, **Flask-WTF** ничего не импортирует из [**wtforms**](../../../../spisok-paketov/veb-freimvorki/wtforms/), вам нужно импортировать поля из **wtforms**.
{% endhint %}

Кроме того, автоматически создается скрытое поле токена CSRF. Вы можете отобразить это в своем шаблоне:

```python
<form method="POST" action="/">
    {{ form.csrf_token }}
    {{ form.name.label }} {{ form.name(size=20) }}
    <input type="submit" value="Go">
</form>
```

Если ваша форма имеет несколько скрытых полей, вы можете отобразить их в одном блоке с помощью [hidden\_tag ()](../api/interfeis-razrabotchika.md).

```python
<form method="POST" action="/">
    {{ form.hidden_tag() }}
    {{ form.name.label }} {{ form.name(size=20) }}
    <input type="submit" value="Go">
</form>
```

### Проверка форм

Проверка запроса в ваших обработчиках представления:

```python
@app.route('/submit', methods=('GET', 'POST'))
def submit():
    form = MyForm()
    if form.validate_on_submit():
        return redirect('/success')
    return render_template('submit.html', form=form)
```

Обратите внимание, что вам не нужно передавать `request.form` в **Flask-WTF**; он загрузится автоматически. И удобство `validate_on_submit()` проверит, является ли это запрос POST и действителен ли он.

Перейдите в раздел «[Создание форм](sozdanie-formy.md)», чтобы получить дополнительные навыки.
