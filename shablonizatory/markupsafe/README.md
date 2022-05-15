# MarkupSafe

**MarkupSafe** экранирует символы, поэтому текст безопасно использовать в HTML и XML. Символы, имеющие особое значение, заменяются так, чтобы они отображались как фактические символы. Это смягчает атаки инъекций, что означает, что вводимые пользователем данные, которым не доверяют, могут безопасно отображаться на странице.

Функция **escape()** экранирует текст и возвращает объект **Markup**. Объект больше не будет экранирован, но любой текст, который используется с ним, будет экранирован, гарантируя, что результат останется безопасным для использования в HTML.

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

Установите и обновите с помощью [pip](https://pip.pypa.io/en/stable/getting-started/):

```bash
pip install -U MarkupSafe
```

### Содержание

* [Работа с безопасным текстом](markupsafe-rabota-s-bezopasnym-tekstom.md)
  * [Необязательные значения](markupsafe-rabota-s-bezopasnym-tekstom.md#neobyazatelnye-znacheniya)
  * [Конвертация объекта в строку](markupsafe-rabota-s-bezopasnym-tekstom.md#konvertaciya-obekta-v-stroku)
* [HTML-представления](html-predstavleniya-markupsafe.md)
* [Форматирование строки](formatirovanie-stroki-markupsafe.md)
  * [Метод format](formatirovanie-stroki-markupsafe.md#metod-format)
  * [Форматирование в стиле printf](formatirovanie-stroki-markupsafe.md#formatirovanie-v-stile-printf)
* Лицензия BSD-3-Clause
* Изменения
