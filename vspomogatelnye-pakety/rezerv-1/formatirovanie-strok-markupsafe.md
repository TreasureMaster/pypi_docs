# Форматирование строк MarkupSafe

Класс [Markup](rabota-s-bezopasnym-tekstom.md#class-markupsafe-markup) можно использовать как строку формата. Объекты, отформатированные в строку разметки, будут экранированы первыми.

## Метод форматирования

Метод **format** расширяет стандартное поведение [str.format ()](https://docs.python.org/3/library/stdtypes.html#str.format) для использования метода `__html_format__`.

1. Если у объекта есть метод `__html_format__`, он вызывается как замена для метода `__format__`. Ему передается спецификатор формата, если он задан. Метод должен возвращать строку или экземпляр [Markup](rabota-s-bezopasnym-tekstom.md#class-markupsafe-markup).
2. Если у объекта есть метод `__html__`, он вызывается. Если спецификатор формата был передан, а класс определил `__html__`, но не `__html_format__`, возникает ошибка **ValueError**.
3. В противном случае используется формат Python по умолчанию, и результат экранируется.

Например, чтобы реализовать **User**, который заключает свое _**name**_ в тег _**span**_ и добавляет ссылку при использовании спецификатора формата `'link'`:

```python
class User(object):
    def __init__(self, id, name):
        self.id = id
        self.name = name

    def __html_format__(self, format_spec):
        if format_spec == 'link':
            return Markup(
                '<a href="/user/{}">{}</a>'
            ).format(self.id, self.__html__())
        elif format_spec:
            raise ValueError('Invalid format spec')
        return self.__html__()

    def __html__(self):
        return Markup(
            '<span class="user">{0}</span>'
        ).format(self.name)
```

```python
>>> user = User(3, '<script>')
>>> escape(user)
Markup('<span class="user">&lt;script&gt;</span>')
>>> Markup('<p>User: {user:link}').format(user=user)
Markup('<p>User: <a href="/user/3"><span class="user">&lt;script&gt;</span></a>
```

См. документацию Python по [синтаксису строки формата](https://docs.python.org/3/library/string.html#formatstrings).

## Форматирование в стиле printf

Помимо экранирования, процентное форматирование не требует особого поведения.

```python
>>> user = User(3, '<script>')
>>> Markup('<a href="/user/%d">"%s</a>') % (user.id, user.name)
Markup('<a href="/user/3">&lt;script&gt;</a>')
```

См. документацию Python по [форматированию в стиле printf](https://docs.python.org/3/library/stdtypes.html#old-string-formatting).
