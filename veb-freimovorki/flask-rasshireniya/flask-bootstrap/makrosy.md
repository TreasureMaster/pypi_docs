# Макросы

## Макросы (Macros)

**Flask-Bootstrap** содержит макросы, которые облегчают вашу жизнь. Их нужно импортировать, как в этом примере:

```python
{% raw %}
{% extends "bootstrap/base.html" %}
{% import "bootstrap/wtf.html" as wtf %}
{% endraw %}
```

Это позволит импортировать макросы `wtf.html` с префиксом имени `wtf` (они обсуждаются ниже в разделе [поддержки WTForms](podderzhka-wtforms.md)).

В дополнение к небольшим макросам на этой странице также доступна широкая поддержка других библиотек; подробности см. в [поддержке WTForms](podderzhka-wtforms.md) и в [поддержке Flask-SQLAlchemy](podderzhka-flask-sqlalchemy.md).

## Исправления (Fixes)

Исправления кроссбраузерности (особенно для Internet Explorer < 9) обычно включены, но не поставляются с **Flask-Bootstrap**. Вы можете скачать [html5shiv](https://raw.githubusercontent.com/aFarkas/html5shiv/master/dist/html5shiv.min.js) и [Respond.js](https://raw.githubusercontent.com/scottjehl/Respond/master/dest/respond.min.js), поместить их в статическую папку своих приложений и включить их, как в этом примере:

```python
{% raw %}
{% import "bootstrap/fixes.html" as fixes %}
{% block head %}
  {{super()}}
  {{fixes.ie8()}}
{% endblock %}
{% endraw %}
```

Хотя скрипты не включены, ссылки на них в CDN есть, поэтому, если вы не используете **BOOTSTRAP\_SERVE\_LOCAL**, они будут работать из коробки. См. раздел [поддержка CDN](podderzhka-cdn.md) для получения дополнительных сведений о том, как доставка CDN работает с **Flask-Bootstrap**.

## Аналитика Google

API Google Analytics в последнее время изменился довольно быстро, в настоящее время [analytics.js](https://developers.google.com/analytics/devguides/collection/analyticsjs/) лучше всего поддерживается с помощью макроса `uanalytics (id, options = 'auto')`:

```python
{% raw %}
{% import "bootstrap/google.html" as google %}

{% block scripts %}
  {{super()}}
  {{google.uanalytics('U-XXXX-YY')}}
{% endblock %}
{% endraw %}
```

Можно передать параметры в вызов функции js `ga ()`, например, чтобы использовать функцию [User ID](https://developers.google.com/analytics/devguides/collection/analyticsjs/cookies-user-id):

```python
{{google.uanalytics('U-XXXX-YY', {'userId': 'myUser'})}}
```

Если вы хотите, чтобы учетную запись аналитики можно было настраивать извне, вы можете использовать что-то вроде этого:

```python
{{google.uanalytics(config['GOOGLE_ANALYTICS_ID'])}}
```

{% hint style="info" %}
**Примечание:**

Убедитесь, что вы, по крайней мере, правильно псевдомизируете идентификаторы пользователей.
{% endhint %}

Официально устаревшая поддержка API [ga.js](https://developers.google.com/analytics/devguides/collection/gajs/) также доступна через одноименную макрос `analytics (account)`:

```python
{{google.analytics(account=config['GOOGLE_ANALYTICS_ID'])}}
```

## Утилиты

Несколько дополнительных шаблонных макросов доступны в файле `bootstrap/utils.html`. Как и макросы форм, они предназначены для ускорения разработки приложений до тех пор, пока они не будут заменены пользовательскими решениями в более зрелых приложениях.

### функция flashed\_messages ()

#### &#x20;flashed\_messages(_messages=None_, _container=True_, _transform=..._, _default\_category=None_, _dismissible=False_)

Отображает сообщения **Flask** `flash ()`. Сопоставляет часто используемые категории с немного необычными классами начальной загрузки css (например, `error -> danger`).

#### Параметры:

* _**messages**_ - Список сообщений. Если не указан, будет использовать `get_flashed_messages ()` для их получения.
* _**container**_ - Если `true`, выводит полный элемент `<div class="container">`, в противном случае - только сообщения, каждое из которых заключено в `<div>`.
* _**transform**_ - Словарь сопоставлений категорий. Будет выполняться поиск без учета регистра. По умолчанию сопоставляет все имена уровней логирования Python с классами начальной загрузки CSS.
* _**default\_category**_ - Если у категории нет карты преобразований, она передается без изменений. Если задана категория `default_category`, она заменяется этим.
* _**dismissible**_ - Если `true`, выводит кнопку, чтобы закрыть предупреждение. Для полностью работоспособных и запрещенных предупреждений необходимо использовать плагин JavaScript предупреждений.

Обратите внимание, что для правильной работы этой функции мигающие сообщения должны быть отнесены к допустимой категории предупреждений начальной загрузки (одна из следующих: **success**, **info**, **warning**, **danger**).

#### Пример:

```python
flash('Operation failed', 'danger')
```

Версии **Flask-Bootstrap** до 3.3.5.7 не экранировали содержимое `flashed_messages`, чтобы можно было использовать HTML. Это поведение изменилось, теперь предпочтительным способом использования HTML внутри сообщений является использование обертки разметки **Markup**:

```python
from flask import flash
from markupsafe import Markup

# ...

flash(Markup('Flashed message with <b>bold</b> statements'), 'success')

user_name = '<b>ad username'
flash(Markup('<u>You</u> are our favorite user, <i>'
             + user_name
             + Markup('</i>!'),
     'danger')
```

### функция icon ()

#### &#x20;icon(_type_, _extra\_classes_, _\*\*kwargs_)

Отображает **Glyphicon** в элементе `<span>`.

#### Параметры:

* _**messages**_ - Краткое название значка, например `remove`.
* _**extra\_classes**_ - Список дополнительных классов, добавляемых к атрибуту класса.
* _**kwargs**_ - Дополнительные атрибуты html.

### функция form\_button ()

#### &#x20;form\_button(_url_, _content_, _method='post'_, _class='btn-link'_, _\*\*kwargs_)

Отображает кнопку/ссылку, заключенную в форму.

#### Параметры:

* _**url**_ - Конечная точка (endpoint) для отправки.
* _**content**_ - Внутреннее содержимое элемента кнопки.
* _**method**_ - method-атрибут окружающей формы.
* _**class**_ - class-атрибут элемента кнопки.
* _**kwargs**_ - Дополнительные атрибуты html для элемента кнопки.

Небольшой удобный метод создания таких вещей, как кнопки удаления, без использования запросов **GET**. Пример:

```python
{{form_button(url_for('remove_entry', id=entry_id),
              icon('remove') + ' Remove entry')}}
```
