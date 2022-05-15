# API Flask-Mail

## класс Mail

#### &#x20;_class_ flask\_mail.Mail(_app=None_)

Управляет обменом сообщениями электронной почты.

### Параметры конструктора

* _**app**_ - экземпляр **Flask**

### Методы Mail

* &#x20;_**send**_ ( _message_ ) - Отправляет один экземпляр сообщения. Если `TESTING` имеет значение `True`, сообщение фактически не будет отправлено. Параметры:
  * _**message**_ - экземпляр Message
* _**connect**_ () - Открывает соединение с почтовым хостом.
* &#x20;_**send\_message**_ (_\*args_, _\*\*kwargs_) - Ярлык для отправки (msg). Принимает те же аргументы, что и конструктор сообщений. _Добавлено в версии_: 0.3.5.

## класс Attachment

#### &#x20;_class_ flask\_mail.Attachment(_filename=None_, _content\_type=None_, _data=None_, _disposition=None_, _headers=None_)

Инкапсулирует информацию о прикрепленных файлах. _Добавлено в версии_: 0.3.5.

### Параметры конструктора

* _**filename**_ - имя вложенного файла
* _**content\_type**_ - тип mimetype файла
* _**data**_ - необработанные данные файла
* _**disposition**_ - Content-Disposition (если есть)

## класс Connection

#### &#x20;_class_ flask\_mail.Connection(_mail_)

Обрабатывает соединение с хостом.

### Методы Connection

* &#x20;_**send**_ (_message_, _envelope\_from=None_) - Проверяет и отправляет сообщение. Параметры:
  * _**message**_ - экземпляр Message
  * _**envelope\_from**_ - Адрес электронной почты, который будет использоваться в команде `MAIL FROM`.
* _**send\_message**_ (_\*args_, _\*\*kwargs_) - Ярлык для отправки (msg). Принимает те же аргументы, что и конструктор сообщений. _Добавлено в версии_: 0.3.5.

## класс Message

#### &#x20;_class_ flask\_mail.Message(_subject=''_, _recipients=None_, _body=None_, _html=None_, _sender=None_, _cc=None_, _bcc=None_, _attachments=None_, _reply\_to=None_, _date=None_, _charset=None_, _extra\_headers=None_, _mail\_options=None_, _rcpt\_options=None_)

Инкапсулирует сообщение электронной почты.

### Параметры конструктора

* _**subject**_ - заголовок темы электронного письма
* _**recipients**_ - список адресов электронной почты
* _**body**_ - тело сообщения в виде простого текста
* _**html**_ - тело сообщения в виде HTML текста
* _**sender**_ - адрес отправителя электронной почты или `MAIL_DEFAULT_SENDER` по умолчанию
* _**cc**_ - лист CC
* _**bcc**_ - лист BCC
* _**attachments**_ - список экземпляров вложений
* _**reply\_to**_ - обратный адрес
* _**date**_ - дата отправки
* _**charset**_ - набор символов сообщения
* _**extra\_headers**_ - словарь дополнительных заголовков для сообщения
* _**mail\_options**_ - список параметров ESMTP, которые будут использоваться в команде `MAIL FROM`
* _**rcpt\_options**_ - список параметров ESMTP, используемых в командах RCPT

### Методы Message

* &#x20;_**attach**_ (_filename=None_, _content\_type=None_, _data=None_, _disposition=None_, _headers=None_) - Добавляет вложение к сообщению. Параметры:
  * _**filename**_ - имя вложенного файла
  * _**content\_type**_ - тип mimetype файла
  * _**data**_ - необработанные данные файла
  * _**disposition**_ - Content-Disposition (если есть)
* &#x20;_**add\_recipient**_ (_recipient_) - Добавляет еще одного получателя к сообщению. Параметр:
  * _**recipient**_ - адрес электронной почты получателя.
