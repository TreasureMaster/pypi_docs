# Widgets (виджеты)

Виджеты - это классы, целью которых является отображение поля в его пригодном для использования представлении, обычно XHTML. Когда вызывается поле, поведение по умолчанию - делегировать визуализацию его виджету. Эта абстракция предоставляется для того, чтобы можно было легко создавать виджеты для настройки отображения существующих полей.

{% hint style="info" %}
**Примечание.** Все встроенные виджеты вернутся после рендеринга «HTML-safe» подкласса строки Unicode, который многие фреймворки шаблонов (Jinja, Mako, Genshi) распознают как не требующие автоматического экранирования.
{% endhint %}

## Встроенные виджеты

* &#x20;_class_ **wtforms.widgets.ListWidget**(_html\_tag='ul'_, _prefix\_label=True_) - Отображает список полей в виде списка `<ul>` или `<ol>`. Это используется для полей, которые инкапсулируют многие внутренние поля как подполя. Виджет попытается перебрать поле, чтобы получить доступ к подполям, и вызвать их для их визуализации. Если установлен _**prefix\_label**_, метка подполя печатается перед полем, в противном случае - после. Последний полезен для итерации радио `type="radio"` или флажков `type="checkbox"`.
* &#x20;_class_ **wtforms.widgets.TableWidget**(_with\_table\_tag=True_) - Отображает список полей как набор строк таблицы с парами `th/td`. Если _**with\_table\_tag**_ имеет значение `True`, то вокруг строк помещается закрывающий тег `<table>`. Скрытые поля не будут отображаться в строке, вместо этого поле будет помещено в следующую строку таблицы, чтобы обеспечить достоверность XHTML. Скрытые поля в конце списка полей появятся за пределами таблицы.
* &#x20;_class_ **wtforms.widgets.Input**(_input\_type=None_) - Визуализирует базовое поле `<input>`. Это используется в качестве основы для большинства других полей ввода. По умолчанию метод `_value ()` будет вызываться для связанного поля для предоставления атрибута `value=` HTML.
* &#x20;_class_ **wtforms.widgets.TextInput** - Визуализирует ввод текста в одну строку.
* &#x20;_class_ **wtforms.widgets.PasswordInput**(_hide\_value=True_) - Визуализирует ввод пароля. В целях безопасности это поле по умолчанию не воспроизводит значение при отправке формы. Чтобы значение было заполнено, установите _**hide\_value**_ на `False`.
* &#x20;_class_ **wtforms.widgets.HiddenInput -** Визуализирует скрытый ввод.
* &#x20;_class_ **wtforms.widgets.CheckboxInput -** Устанавливает флажок checkbox. Атрибут HTML `"checked"` устанавливается, если данные поля не являются ложными.
* &#x20;_class_ **wtforms.widgets.FileInput -** Отображение ввода средства выбора файла. Параметры:
  * _**mulitple**_ - разрешить выбор нескольких файлов.
* &#x20;_class_ **wtforms.widgets.SubmitInput -** Отображает кнопку отправки. Метка поля используется как текст кнопки отправки, а не данные в поле.
* &#x20;_class_ **wtforms.widgets.TextArea -** Отображает многострочную текстовую область. Строки _**rows**_ и столбцы _**cols**_ должны передаваться как аргументы ключевого слова при рендеринге.
*   &#x20;_class_ **wtforms.widgets.Select**(_multiple=False_) - Отображает поле выбора. Если _**multiple**_ равно `True`, то свойство _**size**_ должно быть указано при рендеринге, чтобы поле было пригодным для работы. Поле должно предоставлять метод `iter_choices ()`, который будет вызывать&#x20;

    виджет при рендеринге; этот метод должен возвращать кортежи `(value, label, selected)`.

## HTML5 виджеты

* &#x20;_class_ **wtforms.widgets.html5.ColorInput**(_input\_type=None_) - Отображает ввод с типом «color».
* &#x20;_class_ **wtforms.widgets.html5.DateTimeInput**(_input\_type=None_) - Отображает ввод с типом «datetime».
* &#x20;_class_ **wtforms.widgets.html5.DateTimeLocalInput**(_input\_type=None_) - Отображает ввод с типом «datetime-local».
* &#x20;_class_ **wtforms.widgets.html5.DateInput**(_input\_type=None_) - Отображает ввод с типом «date».
* &#x20;_class_ **wtforms.widgets.html5.EmailInput**(_input\_type=None_) - Отображает ввод с типом «email».
* &#x20;_class_ **wtforms.widgets.html5.MonthInput**(_input\_type=None_) - Отображает ввод с типом «month».
* &#x20;_class_ **wtforms.widgets.html5.NumberInput**(_step=None_, _min=None_, _max=None_) - Отображает ввод с типом «number».
* &#x20;_class_ **wtforms.widgets.html5.RangeInput**(_step=None_) - Отображает ввод с типом «range».
* &#x20;_class_ **wtforms.widgets.html5.SearchInput**(_input\_type=None_) - Отображает ввод с типом «search».
* &#x20;_class_ **wtforms.widgets.html5.TelInput**(_input\_type=None_) - Отображает ввод с типом «tel».
* &#x20;_class_ **wtforms.widgets.html5.TimeInput**(_input\_type=None_) - Отображает ввод с типом «time».
* &#x20;_class_ **wtforms.widgets.html5.URLInput**(_input\_type=None_) - Отображает ввод с типом «url».
* &#x20;_class_ **wtforms.widgets.html5.WeekInput**(_input\_type=None_) - Отображает ввод с типом «week».

## Утилиты для создания виджетов

Эти утилиты используются в виджетах **WTForms** для рендеринга HTML, а также для работы со структурами шаблонов HTML. Их также можно импортировать для использования в создании пользовательских виджетов.

* &#x20;**wtforms.widgets.html\_params**(_\*\*kwargs_) - Генерирует синтаксис атрибута HTML из введенных аргументов ключевого слова.

Выходное значение сортируется по переданным ключам, чтобы обеспечить согласованный вывод каждый раз, когда эта функция вызывается с теми же параметрами. Из-за частого использования обычно зарезервированных ключевых слов _**class**_ и _**for**_ их суффикс с подчеркиванием позволит их использовать.

Чтобы упростить использование атрибутов `data-` и `aria-`, если имя атрибута начинается с _**data\_** _ или _ **aria\_**_, то каждое подчеркивание будет заменено дефисом в сгенерированном атрибуте.

```python
>>> html_params(data_attr='user.name', aria_labeledby='name')
'data-attr="user.name" aria-labeledby="name"'
```

Кроме того, значения `True` и `False` особенные:

* `attr=True` генерирует компактный HTML-вывод логического атрибута, например `checked=True` будет генерировать просто `checked`.
* `attr=False` будет проигнорирован и не будет выводиться.

```python
>>> html_params(name='text1', id='f', class_='text')
'class="text" id="f" name="text1"'

>>> html_params(checked=True, readonly=False, name="text1", abc="hello")
'abc="hello" checked name="text1"'
```

**WTForms** использует **MarkupSafe** для экранирования небезопасных символов HTML перед рендерингом. Вы можете пометить строку с помощью **markupsafe.Markup**, чтобы указать, что ее не следует экранировать.

## Пользовательские виджеты

Виджеты, как и валидаторы, предоставляют простой вызываемый контракт. При необходимости виджеты могут принимать аргументы настройки через конструктор. Когда поле вызывается или печатается, оно вызывает виджет с самим собой в качестве первого аргумента, а затем любые дополнительные аргументы, передаваемые вызывающему объекту в качестве ключевых слов. Передача поля выполняется таким образом, чтобы один экземпляр виджета мог использоваться во многих экземплярах поля.

Давайте посмотрим на виджет, который отображает текстовое поле с дополнительным классом, если есть ошибки:

```python
class MyTextInput(TextInput):
    def __init__(self, error_class=u'has_errors'):
        super(MyTextInput, self).__init__()
        self.error_class = error_class

    def __call__(self, field, **kwargs):
        if field.errors:
            c = kwargs.pop('class', '') or kwargs.pop('class_', '')
            kwargs['class'] = u'%s %s' % (self.error_class, c)
        return super(MyTextInput, self).__call__(field, **kwargs)
```

В приведенном выше примере мы расширили поведение существующего виджета **TextInput**, добавив при необходимости класс CSS. Однако виджеты не обязательно должны расширяться от существующего виджета и даже не должны быть классом. Например, вот виджет, который отображает **SelectMultipleField** как набор флажков:

```python
def select_multi_checkbox(field, ul_class='', **kwargs):
    kwargs.setdefault('type', 'checkbox')
    field_id = kwargs.pop('id', field.id)
    html = [u'<ul %s>' % html_params(id=field_id, class_=ul_class)]
    for value, label, checked in field.iter_choices():
        choice_id = u'%s-%s' % (field_id, value)
        options = dict(kwargs, name=field.name, value=value, id=choice_id)
        if checked:
            options['checked'] = 'checked'
        html.append(u'<li><input %s /> ' % html_params(**options))
        html.append(u'<label for="%s">%s</label></li>' % (field_id, label))
    html.append(u'</ul>')
    return u''.join(html)
```
