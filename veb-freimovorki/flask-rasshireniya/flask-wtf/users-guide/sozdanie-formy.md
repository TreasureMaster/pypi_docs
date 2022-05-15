# Создание формы

### Безопасная форма

Без какой-либо конфигурации [**FlaskForm**](../api/interfeis-razrabotchika.md) будет безопасной формой сеанса с защитой csrf. Мы призываем вас ничего не делать.

Но если вы хотите отключить защиту csrf, вы можете пройти:

```python
form = FlaskForm(csrf_enabled=False)
```

Вы можете отключить его глобально - хотя на самом деле и не должны - с помощью конфигурации:

```python
WTF_CSRF_ENABLED = False
```

Чтобы сгенерировать токен csrf, у вас должен быть секретный ключ, обычно он совпадает с секретным ключом вашего приложения **Flask**. Если вы хотите использовать другой секретный ключ, настройте его:

```python
WTF_CSRF_SECRET_KEY = 'a random string'
```

### Загрузка файлов

****[**FileField**](../api/interfeis-razrabotchika.md), предоставляемый **Flask-WTF**, отличается от поля, предоставляемого **WTForms**. Он проверит, что файл не является пустым экземпляром [**FileStorage**](https://werkzeug.palletsprojects.com/en/1.0.x/datastructures/#werkzeug.datastructures.FileStorage), иначе данные будут `None`.

```python
from flask_wtf import FlaskForm
from flask_wtf.file import FileField, FileRequired
from werkzeug.utils import secure_filename

class PhotoForm(FlaskForm):
    photo = FileField(validators=[FileRequired()])

@app.route('/upload', methods=['GET', 'POST'])
def upload():
    if form.validate_on_submit():
        f = form.photo.data
        filename = secure_filename(f.filename)
        f.save(os.path.join(
            app.instance_path, 'photos', filename
        ))
        return redirect(url_for('index'))

    return render_template('upload.html', form=form)
```

Не забудьте установить для `enctype` HTML-формы значение `multipart/form-data`, иначе `request.files` будет пустым.

```markup
<form method="POST" enctype="multipart/form-data">
    ...
</form>
```

**Flask-WTF** обрабатывает передачу данных формы в форму за вас. Если вы передаете данные явно, помните, что `request.form` необходимо объединить с `request.files`, чтобы форма могла видеть данные файла.

```python
form = PhotoForm()
# эквивалентно:

from flask import request
from werkzeug.datastructures import CombinedMultiDict
form = PhotoForm(CombinedMultiDict((request.files, request.form)))
```

### Проверка

**Flask-WTF** поддерживает проверку загрузки файлов с помощью [FileRequired](../api/interfeis-razrabotchika.md) и [FileAllowed](../api/interfeis-razrabotchika.md). Их можно использовать как с классами **Flask-WTF**, так и с классами **WTForms** `FileField`.

[FileAllowed](../api/interfeis-razrabotchika.md) хорошо работает с **Flask-Uploads**.

```python
from flask_uploads import UploadSet, IMAGES
from flask_wtf import FlaskForm
from flask_wtf.file import FileField, FileAllowed, FileRequired

images = UploadSet('images', IMAGES)

class UploadForm(FlaskForm):
    upload = FileField('image', validators=[
        FileRequired(),
        FileAllowed(images, 'Images only!')
    ])
```

Его можно использовать без **Flask-Uploads**, передав расширения напрямую.

```python
class UploadForm(FlaskForm):
    upload = FileField('image', validators=[
        FileRequired(),
        FileAllowed(['jpg', 'png'], 'Images only!')
    ])
```

### Recaptcha

**Flask-WTF** также обеспечивает поддержку Recaptcha через **RecaptchaField**:

```python
from flask_wtf import FlaskForm, RecaptchaField
from wtforms import TextField

class SignupForm(FlaskForm):
    username = TextField('Username')
    recaptcha = RecaptchaField()
```

Это идет вместе с рядом настроек, которые вы должны реализовать.

| Название                | Необходимость | Описание                               |
| ----------------------- | ------------- | -------------------------------------- |
| RECAPTCHA\_PUBLIC\_KEY  |  **required** | Открытый ключ                          |
| RECAPTCHA\_PRIVATE\_KEY |  **required** | Закрытый ключ                          |
| RECAPTCHA\_API\_SERVER  |  **optional** | Укажите свой сервер API Recaptcha      |
| RECAPTCHA\_PARAMETERS   |  **optional** | Словарь параметров JavaScript (api.js) |
| RECAPTCHA\_DATA\_ATTRS  |  **optional** | Набор параметров атрибутов данных      |

Описание атрибутов параметра `RECAPTCHA_DATA_ATTRS` можно посмотреть [здесь](https://developers.google.com/recaptcha/docs/display).

Пример `RECAPTCHA_PARAMETERS` и `RECAPTCHA_DATA_ATTRS`:

```python
RECAPTCHA_PARAMETERS = {'hl': 'zh', 'render': 'explicit'}
RECAPTCHA_DATA_ATTRS = {'theme': 'dark'}
```

Для тестирования вашего приложения, если `app.testing` имеет значение `True`, поле **recaptcha** всегда будет действительным для вашего удобства.

И это легко настроить в шаблонах:

```python
<form action="/" method="post">
    {{ form.username }}
    {{ form.recaptcha }}
</form>
```

У нас есть для вас пример: [recaptcha@github](https://github.com/lepture/flask-wtf/tree/master/examples/recaptcha).
