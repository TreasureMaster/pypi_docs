# CSRF защита

Любое представление, использующее [FlaskForm](../api/interfeis-razrabotchika.md) для обработки запроса, уже получает защиту CSRF. Если у вас есть представления, которые не используют **FlaskForm** или не выполняют запросы **AJAX**, используйте предоставленное расширение CSRF для защиты этих запросов.

### Настройка

Чтобы включить защиту CSRF глобально для приложения **Flask**, зарегистрируйте расширение **CSRFProtect**.

```python
from flask_wtf.csrf import CSRFProtect

csrf = CSRFProtect(app)
```

Как и другие расширения **Flask**, вы можете применять его лениво:

```python
csrf = CSRFProtect()

def create_app():
    app = Flask(__name__)
    csrf.init_app(app)
```

{% hint style="info" %}
**Примечание:**

Для защиты CSRF требуется секретный ключ для надежной подписи токена. По умолчанию это будет использовать `SECRET_KEY` приложения **Flask**. Если вы хотите использовать отдельный токен, вы можете установить `WTF_CSRF_SECRET_KEY`.
{% endhint %}

### HTML формы

При использовании **FlaskForm** визуализируйте поле CSRF формы как обычно.

```markup
<form method="post">
    {{ form.csrf_token }}
</form>
```

Если шаблон не использует **FlaskForm**, отобразите скрытый ввод с токеном в форме.

```markup
<form method="post">
    <input type="hidden" name="csrf_token" value="{{ csrf_token() }}"/>
</form>
```

### Запросы JavaScript

При отправке запроса **AJAX** добавьте к нему заголовок **X-CSRFToken**. Например, в **jQuery** вы можете настроить все запросы на отправку токена.

```javascript
<script type="text/javascript">
    var csrf_token = "{{ csrf_token() }}";

    $.ajaxSetup({
        beforeSend: function(xhr, settings) {
            if (!/^(GET|HEAD|OPTIONS|TRACE)$/i.test(settings.type) && !this.crossDomain) {
                xhr.setRequestHeader("X-CSRFToken", csrf_token);
            }
        }
    });
</script>
```

### Настройка ответа ошибки

Если проверка CSRF не удалась, возникнет ошибка **CSRFError**. По умолчанию возвращается ответ с причиной сбоя и кодом 400. Вы можете настроить ответ на ошибку, используя обработчик ошибок **Flask** `errorhandler()`.

```python
from flask_wtf.csrf import CSRFError

@app.errorhandler(CSRFError)
def handle_csrf_error(e):
    return render_template('csrf_error.html', reason=e.description), 400
```

### Исключить просмотры из-под защиты

Мы настоятельно рекомендуем вам защитить все свои отображения с помощью CSRF. Но при необходимости вы можете исключить некоторые представления с помощью декоратора.

```python
@app.route('/foo', methods=('GET', 'POST'))
@csrf.exempt
def my_handler():
    # ...
    return 'ok'
```

Вы можете исключить все виды макета.

```python
csrf.exempt(account_blueprint)
```

Вы можете отключить защиту CSRF во всех представлениях по умолчанию, установив для `WTF_CSRF_CHECK_DEFAULT` значение `False` и выборочно вызывая `protect ()` только тогда, когда вам нужно. Это также позволяет выполнять некоторую предварительную обработку запросов перед проверкой токена CSRF.

```python
@app.before_request
def check_csrf():
    if not is_oauth(request):
        csrf.protect()
```
