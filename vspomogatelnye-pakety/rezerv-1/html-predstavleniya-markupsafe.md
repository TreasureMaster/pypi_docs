# HTML-представления MarkupSafe

Во многих фреймворках, если класс реализует метод `__html__`, он будет использоваться для получения представления объекта в HTML. Функция [escape ()](rabota-s-bezopasnym-tekstom.md#markupsafe-escape-s-markup) и класс [Markup](rabota-s-bezopasnym-tekstom.md#class-markupsafe-markup) в **MarkupSafe** понимают и реализуют этот метод. Если у объекта есть метод `__html__`, он будет вызываться вместо преобразования объекта в строку, и результат будет считаться безопасным и не экранированным.

Например, класс **Image** может автоматически генерировать тег `<img>`:

```python
class Image:
    def __init__(self, url):
        self.url = url

    def __html__(self):
        return '<img src="%s">' % self.url
```

```python
>>> img = Image('/static/logo.png')
>>> Markup(img)
Markup('<img src="/static/logo.png">')
```

Поскольку это позволяет избежать экранирования, вам нужно быть осторожным при использовании данных, предоставленных пользователем, в выводе. Например, отображаемое имя пользователя по-прежнему должно быть экранировано:

```python
class User:
    def __init__(self, id, name):
        self.id = id
        self.name = name

    def __html__(self):
        return '<a href="/user/{}">{}</a>'.format(
            self.id, escape(self.name)
        )
```

```python
>>> user = User(3, '<script>')
>>> escape(user)
Markup('<a href="/users/3">&lt;script&gt;</a>')
```
