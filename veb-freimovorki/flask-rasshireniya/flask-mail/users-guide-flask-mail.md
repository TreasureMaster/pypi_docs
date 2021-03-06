# User's Guide Flask-Mail

Расширение Flask-Mail предоставляет простой интерфейс для настройки SMTP с вашим приложением Flask и для отправки сообщений из ваших представлений и сценариев.

## Установка Flask-Mail

Установите с помощью **pip** и **easy\_install**:

```python
pip install Flask-Mail
```

или загрузите последнюю версию из системы контроля версий:

```bash
git clone https://github.com/mattupstate/flask-mail.git
cd flask-mail
python setup.py install
```

Если вы используете **virtualenv**, предполагается, что вы устанавливаете **flask-mail** в том же **virtualenv**, что и ваше приложение (приложения) **Flask**.

## Конфигурирование Flask-Mail

**Flask-Mail** настраивается через стандартный API конфигурации **Flask**. Это доступные параметры (каждая из них описана позже в документации):

* **MAIL\_SERVER** : по умолчанию **‘localhost’**
* **MAIL\_PORT** : по умолчанию **25**
* **MAIL\_USE\_TLS** : по умолчанию **False**
* **MAIL\_USE\_SSL** : по умолчанию **False**
* **MAIL\_DEBUG** : по умолчанию **app.debug**
* **MAIL\_USERNAME** : по умолчанию **None**
* **MAIL\_PASSWORD** : по умолчанию **None**
* **MAIL\_DEFAULT\_SENDER** : по умолчанию **None**
* **MAIL\_MAX\_EMAILS** : по умолчанию **None**
* **MAIL\_SUPPRESS\_SEND** : по умолчанию **app.testing**
* **MAIL\_ASCII\_ATTACHMENTS** : по умолчанию **False**

Кроме того, **Flask-Mail** использует стандартную конфигурацию **Flask** `TESTING` в модульных тестах (см. ниже).

Электронная почта управляется через экземпляр `Mail`:

```python
from flask import Flask
from flask_mail import Mail

app = Flask(__name__)
mail = Mail(app)
```

В этом случае все электронные письма отправляются с использованием значений конфигурации приложения, которые были переданы конструктору класса **Mail**.

В качестве альтернативы вы можете настроить свой экземпляр **Mail** позже во время настройки, используя метод _**init\_app**_:

```python
mail = Mail()

app = Flask(__name__)
mail.init_app(app)
```

В этом случае электронные письма будут отправляться с использованием значений конфигурации из глобального контекста `current_app` **Flask**. Это полезно, если в одном процессе запущено несколько приложений, но с разными параметрами конфигурации.

## Отправка сообщений

Чтобы отправить сообщение, сначала создайте экземпляр сообщения **Message**:

```python
from flask_mail import Message

@app.route("/")
def index():

    msg = Message("Hello",
                  sender="from@example.com",
                  recipients=["to@example.com"])
```

Вы можете установить адреса электронной почты получателя сразу или индивидуально:

```python
msg.recipients = ["you@example.com"]
msg.add_recipient("somebodyelse@example.com")
```

Если вы установили **`MAIL_DEFAULT_SENDER`**, вам не нужно явно указывать отправителя сообщения, так как он будет использовать это значение конфигурации по умолчанию:

```python
msg = Message("Hello",
              recipients=["to@example.com"])
```

Если отправитель `sender` - двухэлементный кортеж, он будет разделен на имя и адрес:

```python
msg = Message("Hello",
              sender=("Me", "me@example.com"))

assert msg.sender == "Me <me@example.com>"
```

Сообщение может содержать текст и/или HTML:

```python
msg.body = "testing"
msg.html = "<b>testing</b>"
```

Наконец, чтобы отправить сообщение, вы используете экземпляр **Mail**, настроенный с вашим приложением **Flask**:

```python
mail.send(msg)
```

## Массовые рассылки

Обычно в веб-приложении вы отправляете одно или два электронных письма за запрос. В определенных ситуациях вы можете захотеть отправить, возможно, десятки или сотни электронных писем одним пакетом - возможно, с помощью внешнего процесса, такого как сценарий командной строки или **cronjob**.

В этом случае вы поступаете немного иначе:

```python
with mail.connect() as conn:
    for user in users:
        message = '...'
        subject = "hello, %s" % user.name
        msg = Message(recipients=[user.email],
                      body=message,
                      subject=subject)

        conn.send(msg)
```

Соединение с вашим почтовым хостом остается активным и закрывается автоматически после отправки всех сообщений.

Некоторые почтовые серверы устанавливают ограничение на количество писем, отправляемых за одно соединение. Вы можете установить максимальное количество писем для отправки перед повторным подключением, указав параметр **`MAIL_MAX_EMAILS`**.

## Вложения

Добавить вложения просто:

```python
with app.open_resource("image.png") as fp:
    msg.attach("image.png", "image/png", fp.read())
```

См. подробности в [API](api-flask-mail.md).

Если **`MAIL_ASCII_ATTACHMENTS`** установлено в `True`, имена файлов будут преобразованы в эквивалент ASCII. Это может быть полезно при использовании почтового ретранслятора, который изменяет содержимое почты и портит спецификацию **Content-Disposition**, когда имена файлов имеют кодировку UTF-8. Преобразование в ASCII - это базовое удаление символов, отличных от ASCII. Это должно быть хорошо для любого символа Юникода, который может быть разложен NFKD на один или несколько символов ASCII. Если вам нужна латинизация/транслитерация (например, ß → ss), ваше приложение должно это сделать и передать правильную строку ASCII.

## Модульные тесты и подавление писем

Когда вы отправляете сообщения в модульных тестах или в среде разработки, полезно иметь возможность подавить отправку электронной почты.

Если для параметра **`TESTING`** установлено значение `True`, электронные письма будут подавлены. Вызов `send ()` для ваших сообщений не приведет к фактической отправке каких-либо сообщений.

В качестве альтернативы вне тестовой среды вы можете установить для **`MAIL_SUPPRESS_SEND`** значение `False`. Это будет иметь такой же эффект.

Однако при написании модульных тестов все же полезно отслеживать электронные письма, которые были бы отправлены.

Чтобы отслеживать отправленные электронные письма, используйте метод _**record\_messages**_:

```python
with mail.record_messages() as outbox:

    mail.send_message(subject='testing',
                      body='test',
                      recipients=emails)

    assert len(outbox) == 1
    assert outbox[0].subject == "testing"
```

**outbox** - это список исходящих отправленных экземпляров сообщений **Message**.

Чтобы этот метод работал, необходимо установить пакет **blinker**.

Обратите внимание, что старый способ добавления исходящих **outbox** сообщений к объекту _**g**_ теперь не рекомендуется.

## Внедрение заголовка

Чтобы предотвратить [внедрение заголовка](http://nyphp.org/PHundamentals/8\_Preventing-Email-Header-Injection) (header injection), попытки отправить сообщение с новой строкой в теме, адресах отправителя или получателя приведут к ошибке **`BadHeaderError`**.

## Сигнальная поддержка

_Новое в версии 0.4_.

**Flask-Mail** теперь обеспечивает поддержку сигнализации через сигнал `email_dispatched`. Он отправляется всякий раз, когда отправляется электронное письмо (даже если электронное письмо фактически не отправлено, то есть в тестовой среде).

Функция, подключающаяся к сигналу `email_dispatched`, принимает экземпляр **Message** в качестве первого аргумента и экземпляр приложения **Flask** в качестве необязательного аргумента:

```python
def log_message(message, app):
    app.logger.debug(message.subject)

email_dispatched.connect(log_message)
```
