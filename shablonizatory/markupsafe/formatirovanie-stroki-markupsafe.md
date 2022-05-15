# Форматирование строки MarkupSafe

Класс [Markup ](markupsafe-rabota-s-bezopasnym-tekstom.md#class-markupsafe.markup-base-encoding-none-errors-strict)можно использовать как строку формата. Объекты, отформатированные в строку разметки, будут экранированы первыми.

### Метод format

Метод форматирования <mark style="color:red;">**format**</mark> расширяет стандартное поведение [str.format()](https://docs.python.org/3/library/stdtypes.html#str.format) для использования метода **\_\_html\_format\_\_**.

1. Если у объекта есть метод **\_\_html\_format\_\_**, он вызывается как замена для метода **\_\_format\_\_**. Ему передается спецификатор формата, если он задан. Метод должен возвращать строку или экземпляр разметки [Markup](markupsafe-rabota-s-bezopasnym-tekstom.md#class-markupsafe.markup-base-encoding-none-errors-strict).
2. Если у объекта есть метод **\_\_html\_\_**, он вызывается. Если спецификатор формата был передан, а класс определил **\_\_html\_\_**, но не **\_\_html\_format\_\_**, возникает ошибка **ValueError**.
3. В противном случае используется метод форматирования **format()** Python по умолчанию, и результат экранируется.

Например, чтобы реализовать **User**, который заключает свое имя **name** в тег **span** и добавляет ссылку при использовании спецификатора формата **«link»**:

```python
class User(object):
    def __init__(self, id, name):
        self.id = id
        self.name = name

    def __html_format__(self, format_spec):
        if format_spec == "link":
            return Markup(
                '<a href="/user/{}">{}</a>'
            ).format(self.id, self.__html__())
        elif format_spec:
            raise ValueError("Invalid format spec")
        return self.__html__()

    def __html__(self):
        return Markup(
            '<span class="user">{0}</span>'
        ).format(self.name)
```

```python
>>> user = User(3, "<script>")
>>> escape(user)
Markup('<span class="user">&lt;script&gt;</span>')
>>> Markup("<p>User: {user:link}").format(user=user)
Markup('<p>User: <a href="/user/3"><span class="user">&lt;script&gt;</span></a>
```

См. документацию Python по [синтаксису строки формата](https://docs.python.org/3/library/string.html#formatstrings).

### Форматирование в стиле printf

Помимо экранирования, процентное форматирование не требует особого поведения.

```python
>>> user = User(3, "<script>")
>>> Markup('<a href="/user/%d">%s</a>') % (user.id, user.name)
Markup('<a href="/user/3">&lt;script&gt;</a>')
```

См. документацию Python по [форматированию в стиле printf](https://docs.python.org/3/library/stdtypes.html#old-string-formatting).
