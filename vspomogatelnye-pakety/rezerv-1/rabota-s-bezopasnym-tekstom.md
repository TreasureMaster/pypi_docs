# Работа с безопасным текстом

## Работа с безопасным текстом

### markupsafe.escape( _s_ ) → markup

Преобразует символы `&`, `<`, `>`, `‘` и `”` в строке _**s**_ в безопасные для HTML последовательности. Используйте это, если вам нужно отобразить текст, который может содержать такие символы в HTML. Помечает возвращаемое значение как строку _**markup**_.

### (_class_) markupsafe.Markup

Строка, готовая к безопасной вставке в документ HTML или XML, либо потому, что она была экранирована, либо потому, что она была помечена как безопасная.

Передача объекта конструктору преобразует его в текст и обертывает, чтобы пометить его как безопасный без экранирования. Чтобы экранировать текст, используйте вместо этого метод класса [escape ()](rabota-s-bezopasnym-tekstom.md#markupsafe-escape-s-markup).

```python
>>> Markup('Hello, <em>World</em>!')
Markup('Hello, <em>World</em>!')

>>> Markup(42)
Markup('42')

>>> Markup.escape('Hello, <em>World</em>!')
Markup('Hello &lt;em&gt;World&lt;/em&gt;!')
```

Это реализует интерфейс `__html__()`, который используют некоторые фреймворки. Передача объекта, реализующего `__html__()`, обернет вывод этого метода, пометив его как безопасный.

```python
>>> class Foo:
...    def __html__(self):
...       return '<a href="/foo">foo</a>'
...
>>> Markup(Foo())
Markup('<a href="/foo">foo</a>')
```

Это подкласс текстового типа (**str** в Python 3, **unicode** в Python 2). Он имеет те же методы, что и этот тип, но все методы экранируют свои аргументы и возвращают экземпляр **Markup**.

```python
>>> Markup('<em>%s</em>') % 'foo & bar'
Markup('<em>foo &amp; bar</em>')

>>> Markup('<em>Hello</em> ') + '<foo>'
Markup('<em>Hello</em> &lt;foo&gt;')
```

#### (classmethod) escape( _s_ )

Экранирует строку. Вызывает [escape ()](rabota-s-bezopasnym-tekstom.md#markupsafe-escape-s-markup) и гарантирует, что для подклассов будет возвращен правильный тип.

#### striptags()

[unescape ()](rabota-s-bezopasnym-tekstom.md#unescape) разметку, удаление тегов и нормализацию пробелов до отдельных пробелов.

```python
>>> Markup('Main &raquo;        <em>About</em>').striptags()
'Main » About'
```

#### unescape()

Преобразует экранированную разметку обратно в текстовую строку. При этом объекты HTML заменяются символами, которые они представляют.

```python
>>> Markup('Main &raquo; <em>About</em>').unescape()
'Main » <em>About</em>'
```

## Необязательные значения

### markupsafe.escape\_silent( _s_ ) → markup

Как **escape**, но преобразует `None` в пустую строку.

## Преобразование объекта в строку

### markupsafe.soft\_unicode( _object_ ) → string

Делает строковый **Unicode**, если это еще не сделано. Таким образом, строка разметки не преобразуется обратно в **Unicode**.
