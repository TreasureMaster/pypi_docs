# Поддержка WTForms

## Поддержка WTForms

Шаблон `bootstrap/wtf.html` содержит макросы, которые помогут вам быстро выводить формы. Однако **Flask-WTF** не является зависимостью от **Flask-Bootstrap** и должен быть установлен явно. API **Flask-WTF** сильно изменился за последние несколько версий, **Flask-Bootstrap** в настоящее время разрабатывается для **Flask-WTF** версии 0.9.2.

Самый простой способ - использовать их как вспомогательные средства для создания формы вручную:

```python
<form class="form form-horizontal" method="post" role="form">
  {{ form.hidden_tag() }}
  {{ wtf.form_errors(form, hiddens="only") }}

  {{ wtf.form_field(form.field1) }}
  {{ wtf.form_field(form.field2) }}
</form>
```

Однако часто вам просто нужно быстро подготовить форму и не требовать интенсивной тонкой настройки:

```python
{{ wtf.quick_form(form) }}
```

## Справка макросов формы

### функция quick\_form ()

#### &#x20;quick\_form(_form_, _action="."_, _method="post"_, _extra\_classes=None_, _role="form"_, _form\_type="basic"_, _horizontal\_columns=('lg'_, _2_, _10)_, _enctype=None_, _button\_map={}_, _id=""_)

Выводит разметку **Bootstrap** для полной формы [**Flask-WTF**](https://flask-wtf.readthedocs.io/en/latest/).

#### Параметры:

* _**form**_ - Форма для вывода.
* _**action**_&#x20;
* _**method**_ - Атрибут метода `<form>`.
* _**extra\_classes**_ - Классы, которые нужно добавить в `<form>`.
* _**role**_ - Атрибут **role** формы `<form>`.
* _**form\_type**_ - Один из основных `basic`, линейных `inline` или горизонтальных `horizontal`. См. [документацию Bootstrap](https://getbootstrap.com/) для получения подробной информации о различных макетах форм.
* _**horizontal\_columns**_ - При использовании горизонтального макета макет формируется следующим образом. Должен быть кортежем из трех элементов (`column-type`, `left-column-size`, `right-column-size`).
* _**enctype**_ -  атрибут **enctype** форму `<form>`. Если `None`, будет автоматически установлено значение `multipart/form-data`, если в форме присутствует **FileField**.
* _**button\_map**_ - Словарь, сопоставляющий имена полей кнопок с такими именами, как `primary`, `danger` или `success`. Кнопки, у которых нет **button\_map**, будут использовать тип кнопки по умолчанию.
* _**id**_ - Атрибут **id** формы `<form>`.

### функция form\_errors ()

#### &#x20;form\_errors(_form_, _hiddens=True_)

Отображает абзацы, содержащие сообщения об ошибках формы. Обычно это используется только для вывода ошибок формы скрытых полей, так как другие присоединяются к полям формы.

#### Параметры:

* _**form**_ - Форма, ошибки которой должны быть отображены.
* _**hiddens**_ - Если `True`, также отображать ошибки скрытых полей. Если `"only"`, визуализируются только скрытые ошибки.

### функция form\_field ()

#### &#x20;form\_field(_field_, _form\_type="basic"_, _horizontal\_columns=('lg'_, _2_, _10)_, _button\_map={}_)

Отображает одно поле формы с окружающими элементами. Используется в основном **quick\_form**.
