# HTML-представления MarkupSafe

Во многих фреймворках, если класс реализует метод **\_\_html\_\_**, он будет использоваться для получения представления объекта в HTML. Функция [escape()](markupsafe-rabota-s-bezopasnym-tekstom.md#markupsafe.escape) и класс [Markup](markupsafe-rabota-s-bezopasnym-tekstom.md#class-markupsafe.markup-base-encoding-none-errors-strict) в **MarkupSafe** понимают и реализуют этот метод. Если у объекта есть метод **\_\_html\_\_**, он будет вызываться вместо преобразования объекта в строку, и результат будет считаться безопасным и не экранированным.

Например, класс **Image** может автоматически генерировать тег **\<img>**:

```python
class Image:
    def __init__(self, url):
        self.url = url

    def __html__(self):
        return f'<img src="{self.url}">'
```

```python
>>> img = Image("/static/logo.png")
>>> Markup(img)
Markup('<img src="/static/logo.png">')
```

Поскольку это позволяет избежать экранирования, вам нужно быть осторожным при использовании данных, предоставленных пользователем, в выводе. Например, отображаемое имя пользователя все равно следует экранировать:

```python
class User:
    def __init__(self, id, name):
        self.id = id
        self.name = name

    def __html__(self):
        return f'<a href="/user/{self.id}">{escape(self.name)}</a>'
```

```python
>>> user = User(3, "<script>")
>>> escape(user)
Markup('<a href="/users/3">&lt;script&gt;</a>')
```
