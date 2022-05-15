# Родные типы Python

Среда по умолчанию **Environment** отображает шаблоны в строки. С помощью **NativeEnvironment** при рендеринге шаблона создается собственный тип Python. Это полезно, если вы используете **Jinja** вне контекста создания текстовых файлов. Например, в вашем коде может быть промежуточный этап, на котором пользователи могут использовать шаблоны для определения значений, которые затем будут переданы в традиционную строковую среду.

## Примеры

Добавление двух значений приводит к целому числу, а не к строке с числом:

```python
>>> env = NativeEnvironment()
>>> t = env.from_string('{{ x + y }}')
>>> result = t.render(x=4, y=2)
>>> print(result)
6
>>> print(type(result))
int
```

Синтаксис списка рендеринга создает список:

```python
>>> t = env.from_string('[{% raw %}
{% for item in data %}{{ item + 1 }},{% endfor %}
{% endraw %}]')
>>> result = t.render(data=range(5))
>>> print(result)
[1, 2, 3, 4, 5]
>>> print(type(result))
list
```

При рендеринге чего-то, что не похоже на литерал Python, получается строка:

```python
>>> t = env.from_string('{{ x }} * {{ y }}')
>>> result = t.render(x=4, y=2)
>>> print(result)
4 * 2
>>> print(type(result))
str
```

Визуализация объекта Python создает этот объект, пока он является единственным узлом:

```python
>>> class Foo:
...     def __init__(self, value):
...         self.value = value
...
>>> result = env.from_string('{{ x }}').render(x=Foo(15))
>>> print(type(result).__name__)
Foo
>>> print(result.value)
15
```

## API типов Python

### класс nativetypes.NativeEnvironment

#### &#x20;_class_ jinja2.nativetypes.NativeEnvironment( \[ _options_ ] )

Среда, которая отображает шаблоны в собственные типы Python.

### класс nativetypes.NativeTemplate

#### &#x20;_class_ jinja2.nativetypes.NativeTemplate( \[ _options_ ] )

* &#x20;**render**(_\*args_, _\*\*kwargs_) - Визуализирует шаблон для создания собственного типа Python. Если результатом является единственный узел, возвращается его значение. В противном случае узлы объединяются в виде строк. Если результат можно проанализировать с помощью [ast.literal\_eval ()](https://docs.python.org/3/library/ast.html#ast.literal\_eval), возвращается проанализированное значение. В противном случае возвращается строка.
