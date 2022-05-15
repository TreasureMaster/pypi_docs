# MarkupSafe

## Краткое описание

**MarkupSafe** экранирует символы, поэтому текст безопасно использовать в HTML и XML. Символы, имеющие особое значение, заменяются так, чтобы они отображались как фактические символы. Это смягчает атаки инъекций, а это означает, что ввод данных ненадежным пользователем может безопасно отображаться на странице.

Функция [escape ()](rabota-s-bezopasnym-tekstom.md#markupsafe-escape) экранирует текст и возвращает объект [Markup](rabota-s-bezopasnym-tekstom.md#class-markupsafe-markup). Объект больше не будет экранирован, но любой текст, который используется с ним, будет экранирован, гарантируя, что результат останется безопасным для использования в HTML.

```python
>>> from markupsafe import escape

>>> hello = escape('<em>Hello</em>')

>>> hello
Markup('&lt;em&gt;Hello&lt;/em&gt;')

>>> escape(hello)
Markup('&lt;em&gt;Hello&lt;/em&gt;')

>>> hello + ' <strong>World</strong>'
Markup('&lt;em&gt;Hello&lt;/em&gt; &lt;strong&gt;World&lt;/strong&gt;')
```

{% hint style="info" %}
В документации предполагается, что вы используете Python 3. Термины «**text**» и «**string**» относятся к классу [str](https://docs.python.org/3/library/stdtypes.html#str). В Python 2 это был бы класс **unicode**.
{% endhint %}

### Установка

Установите и обновите с помощью [pip](https://pip.pypa.io/en/stable/quickstart/):

```bash
pip install -U MarkupSafe
```

## Оглавление

* Работа с безопасным текстом
  * Необязательные значения
  * Преобразование объекта в строку
* HTML-представления
* Форматирование строк
  * Метод форматирования
  * Стиль форматирования printf
* Лицензия BSD-3-Clause
* Изменения
