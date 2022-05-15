# MarkupSafe работа с безопасным текстом

### markupsafe.escape()

Заменяет символы **&**, **<**, **>**, **'** и **"** в строке безопасными для HTML последовательностями. Используйте это, если вам нужно отобразить текст, который может содержать такие символы в HTML.

Если у объекта есть метод **\_\_html\_\_**, он вызывается, и предполагается, что возвращаемое значение уже безопасно для HTML.

**Параметры**: <mark style="color:red;">**s**</mark> - Объект, который нужно преобразовать в строку и экранировать.

**Возвращает**: Строка разметки [**Markup**](markupsafe-rabota-s-bezopasnym-tekstom.md#class-markupsafe.markup) **** с экранированным текстом.

### _class_ markupsafe.Markup(_base=''_, _encoding=None_, _errors='strict'_)

Строка, готовая к безопасной вставке в документ HTML или XML, либо потому, что она была экранирована, либо потому, что она была помечена как безопасная.

Передача объекта конструктору преобразует его в текст и обертывает, чтобы пометить его как безопасный без экранирования. Чтобы экранировать текст, используйте вместо этого метод класса [**escape()**](markupsafe-rabota-s-bezopasnym-tekstom.md#markupsafe.escape).

```python
>>> Markup("Hello, <em>World</em>!")
Markup('Hello, <em>World</em>!')
>>> Markup(42)
Markup('42')
>>> Markup.escape("Hello, <em>World</em>!")
Markup('Hello &lt;em&gt;World&lt;/em&gt;!')
```

Это реализует интерфейс **\_\_html \_\_()**, который используют некоторые фреймворки. Передача объекта, реализующего **\_\_html \_\_()**, обернет вывод этого метода, пометив его как безопасный.

```python
>>> class Foo:
...     def __html__(self):
...         return '<a href="/foo">foo</a>'
... 
>>> Markup(Foo())
Markup('<a href="/foo">foo</a>')
```

Это подкласс [str](https://docs.python.org/3/library/stdtypes.html#str). Он имеет те же методы, но избегает их аргументов и возвращает экземпляр разметки **Markup**.

```python
>>> Markup("<em>%s</em>") % ("foo & bar",)
Markup('<em>foo &amp; bar</em>')
>>> Markup("<em>Hello</em> ") + "<foo>"
Markup('<em>Hello</em> &lt;foo&gt;')
```

**Параметры**:

* **base** (_Any_) –
* **encoding** (_Optional\[_[_str_](https://docs.python.org/3/library/stdtypes.html#str)_]_) –
* **errors** ([_str_](https://docs.python.org/3/library/stdtypes.html#str)) –

Возвращаемый тип: [**Markup**](markupsafe-rabota-s-bezopasnym-tekstom.md#class-markupsafe.markup)****

### _classmethod_ escape(_s_)

Экранирует строку. Вызывает [**escape()**](markupsafe-rabota-s-bezopasnym-tekstom.md#markupsafe.escape) и гарантирует, что для подклассов будет возвращен правильный тип.

Параметры: **s** (_Any_) –

Возвращаемый тип: [markupsafe.Markup](markupsafe-rabota-s-bezopasnym-tekstom.md#class-markupsafe.markup-base-encoding-none-errors-strict)

### striptags()

****[**unescape()**](markupsafe-rabota-s-bezopasnym-tekstom.md#unescape) разметку, удаляет теги и нормализует пробелы до отдельных пробелов.

```python
>>> Markup("Main &raquo;        <em>About</em>").striptags()
'Main » About'
```

Возвращаемый тип: [**str**](https://docs.python.org/3/library/stdtypes.html#str)****

### unescape()

Преобразует экранированную разметку обратно в текстовую строку. При этом объекты HTML заменяются символами, которые они представляют.

```python
>>> Markup("Main &raquo; <em>About</em>").unescape()
'Main » <em>About</em>'
```

Возвращаемый тип: [**str**](https://docs.python.org/3/library/stdtypes.html#str)****

## Необязательные значения

### markupsafe.escape\_silent()

Подобно [escape()](markupsafe-rabota-s-bezopasnym-tekstom.md#markupsafe.escape), но обрабатывает **None** как пустую строку. Полезно с необязательными значениями, поскольку в противном случае вы получите строку **'None'**, когда значение равно **None**.

```python
>>> escape(None)
Markup('None')
>>> escape_silent(None)
Markup('')
```

## Конвертация объекта в строку

### markupsafe.soft\_str()

Преобразует объект в строку, если это еще не сделано. Это сохраняет строку разметки [Markup](markupsafe-rabota-s-bezopasnym-tekstom.md#class-markupsafe.markup-base-encoding-none-errors-strict), а не преобразует ее обратно в базовую строку, поэтому она все равно будет помечена как безопасная и больше не будет экранирована.

```python
>>> value = escape("<User 1>")
>>> value
Markup('&lt;User 1&gt;')
>>> escape(str(value))
Markup('&amp;lt;User 1&amp;gt;')
>>> escape(soft_str(value))
Markup('&lt;User 1&gt;')
```
