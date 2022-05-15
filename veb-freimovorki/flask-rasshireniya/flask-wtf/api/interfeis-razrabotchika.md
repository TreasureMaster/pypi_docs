# Интерфейс разработчика

### Формы и поля

#### &#x20;_class_ flask\_wtf.FlaskForm(_formdata=\<object object>_, _\*\*kwargs_)

Подкласс **WTForms** [Form](../../../../spisok-paketov/veb-freimvorki/wtforms/api/forms-formy.md#klass-form-form), специфичный для **Flask**.

Если данные формы _**formdata**_ не указаны, будет использоваться `flask.request.form` и `flask.request.files`. Явно передайте `formdata = None`, чтобы предотвратить это.

* &#x20;**hidden\_tag**(_\*fields_) - Рендерит скрытые поля формы за один вызов. Поле считается скрытым, если в нем используется виджет **HiddenInput**. Если указаны поля _**fileds**_, отображает только указанные скрытые поля. Если передана строка, отобразите поле с этим именем, если оно существует.

_Изменено в версии 0.13:_ больше не переносит ввод в скрытый **div**. Это действительный HTML5.

_Изменено в версии 0.13:_ пропускает переданные поля, которые не скрыты. Пропускает переданные имена, которых не существует.

* &#x20;**is\_submitted**() - Обрабатывает отправленную форму, если есть активный запрос и используется метод POST, PUT, PATCH или DELETE.
* &#x20;**validate\_on\_submit**() - Вызывает [validate ()](../../../../spisok-paketov/veb-freimvorki/wtforms/api/forms-formy.md), только если форма отправлена. Это ярлык для `form.is_submitted ()` и `form.validate ()`.

#### &#x20;_class_ flask\_wtf.Form(_..._)

_Не рекомендуется с версии 0.13:_ переименован в **FlaskForm**.

#### &#x20;_class_ flask\_wtf.RecaptchaField(_label=''_, _validators=None_, _\*\*kwargs_)

#### &#x20;_class_ flask\_wtf.Recaptcha(_message=None_)

Проверяет ReCaptcha.

#### &#x20;_class_ flask\_wtf.RecaptchaWidget

#### &#x20;_class_ flask\_wtf.file.FileField(_label=None_, _validators=None_, _filters=()_, _description=''_, _id=None_, _default=None_, _widget=None_, _render\_kw=None_, _\_form=None_, _\_name=None_, _\_prefix=''_, _\_translations=None_, _\_meta=None_)

Подкласс [wtforms.fields.FileField](../../../../spisok-paketov/veb-freimvorki/wtforms/api/fields-polya.md#osnovnye-polya), поддерживающий **Werkzeug**.

* &#x20;**has\_file**() - Возвращает `True`, если _**self.data**_ является объектом [**FileStorage**](https://werkzeug.palletsprojects.com/en/1.0.x/datastructures/#werkzeug.datastructures.FileStorage).

_Не рекомендуется, начиная с версии 0.14.1:_ данные больше не устанавливаются, если вход не является непустым **FileStorage**. Вместо этого проверьте, что _**form.data**_ не `None`.

#### &#x20;_class_ flask\_wtf.file.FileAllowed(_upload\_set_, _message=None_)

Проверяет, разрешен ли загруженный файл заданным списком расширений или набором **Flask-Uploads** [UploadSet](https://flask-uploads.readthedocs.io/en/latest/). Параметры:

* _**upload\_set**_ - Список расширений или **UploadSet**
* _**message**_** -** Сообщение об ошибке

Вы также можете использовать синоним `file_allowed`.

#### &#x20;_class_ **flask\_wtf.file.FileRequired**(_message=None_)

Проверяет, что данные являются объектом **Werkzeug** [FileStorage](https://werkzeug.palletsprojects.com/en/1.0.x/datastructures/#werkzeug.datastructures.FileStorage). Параметры:

* _**message**_** -** Сообщение об ошибке

Вы также можете использовать синоним `file_required`.

### CSRF защита

#### &#x20;_class_ flask\_wtf.csrf.CSRFProtect(_app=None_)

Включите глобальную защиту CSRF для приложения **Flask**.

```python
app = Flask(__name__)
csrf = CsrfProtect(app)
```

Проверяет поле `csrf_token`, отправленное с формами, или заголовок `X-CSRFToken`, отправленный с запросами **JavaScript**. Визуализирует токен в шаблонах с помощью `{{ csrf_token() }}`.

См. [документацию по защите от CSRF](../users-guide/csrf-zashita.md).

* &#x20;**error\_handler**(_view_) - Регистрирует функцию, которая будет генерировать ответ на ошибки CSRF.

_Не рекомендуется с версии 0.14:_ используйте вместо нее стандартную систему ошибок Flask с `@app.errorhandler (CSRFError)`. Это будет удалено в версии 1.0.

В функцию будет передан один аргумент, _**reason**_. По умолчанию это вызовет ошибку **CSRFError**.

```python
@csrf.error_handler
def csrf_error(reason):
    return render_template('error.html', reason=reason)
```

По историческим причинам функция может либо вернуть ответ, либо вызвать исключение с помощью [flask.abort ()](https://flask.palletsprojects.com/en/1.1.x/api/#flask.abort).

* &#x20;**exempt**(_view_) - Отметьте представление или схему для исключения из защиты CSRF.

```python
@app.route('/some-view', methods=['POST'])
@csrf.exempt
def some_view():
    ...
```

```python
bp = Blueprint(...)
csrf.exempt(bp)
```

#### &#x20;_class_ flask\_wtf.csrf.CsrfProtect(_..._)

_Не рекомендуется с версии 0.14:_ переименовано в **CSRFProtect**.

#### &#x20;_class_ flask\_wtf.csrf.CSRFError(_description=None_, _response=None_)

Возбуждается, если клиент отправляет недопустимые данные CSRF с запросом. По умолчанию генерирует ответ 400 **Bad Request** с указанием причины сбоя. Настройте ответ, зарегистрировав обработчик с помощью [flask.Flask.errorhandler ()](https://flask.palletsprojects.com/en/1.1.x/api/#flask.Flask.errorhandler).

#### &#x20;flask\_wtf.csrf.generate\_csrf(_secret\_key=None_, _token\_key=None_)

Создает токен CSRF. Токен кэшируется для запроса, поэтому несколько вызовов этой функции будут генерировать один и тот же токен. Во время тестирования может быть полезно получить доступ к подписанному токену в `g.csrf_token` и необработанному токену в `session ['csrf_token']`. Параметры:

* _**secret\_key**_ - Используется для надежной подписи токена. По умолчанию - `WTF_CSRF_SECRET_KEY` или `SECRET_KEY`.
* _**token\_key**_ - Ключ, где токен хранится в сеансе для сравнения. По умолчанию - `WTF_CSRF_FIELD_NAME` или `'csrf_token'`.

#### &#x20;flask\_wtf.csrf.validate\_csrf(_data_, _secret\_key=None_, _time\_limit=None_, _token\_key=None_)

Проверьте, являются ли указанные данные действительным токеном CSRF. Это сравнивает данный подписанный токен с тем, который хранится в сеансе. Параметры:

* _**data**_ - Подписанный токен CSRF для проверки.
* _**secret\_key**_ - Используется для надежной подписи токена. По умолчанию - `WTF_CSRF_SECRET_KEY` или `SECRET_KEY`.
* _**time\_limit**_ - Количество секунд, в течение которых токен действителен. По умолчанию - `WTF_CSRF_TIME_LIMIT` или 3600 секунд (60 минут).
* _**token\_key**_ - Ключ, где токен хранится в сеансе для сравнения. По умолчанию - `WTF_CSRF_FIELD_NAME` или `'csrf_token'`.

Возбуждает: _**ValidationError**_ - Содержит причину сбоя проверки.

_Изменено в версии 0.14:_ вызывает _**ValidationError**_ с конкретным сообщением об ошибке вместо возврата `True` или `False`.
