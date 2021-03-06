# Описание модуля decorator

## Декораторы для людей

| Информация    | Описание                                                                                   |
| ------------- | ------------------------------------------------------------------------------------------ |
| Author        | Michele Simionato                                                                          |
| E-mail        |  [michele.simionato@gmail.com](mailto:michele.simionato@gmail.com)                         |
| Version       | 4.4.2 (2020-02-29)                                                                         |
| Supports      | Python 2.6, 2.7, 3.0, 3.1, 3.2, 3.3, 3.4, 3.5, 3.6, 3.7, 3.8                               |
| Download page | [http://pypi.python.org/pypi/decorator/4.4.2](http://pypi.python.org/pypi/decorator/4.4.2) |
| Installation  |  `pip install decorator`                                                                   |
| License       | BSD license                                                                                |

## Введение

Модулю **decorator** более десяти лет, но он все еще жив и здоров. Он используется несколькими фреймворками (IPython, scipy, authkit, pylons, pycuda, sugar, ...) и долгое время оставался стабильным. Это лучший вариант, если вы хотите согласованно сохранить сигнатуру оформленных функций во всех выпусках Python. Версия 4 полностью совместима с прошлым, за исключением одного: поддержка Python 2.4 и 2.5 была прекращена. Это решение позволило использовать единую базу кода как для Python 2.X, так и для Python 3.X. Это огромный бонус, поскольку я мог удалить более 2000 строк дублированной документации / тестов. Необходимость поддерживать отдельные документы для Python 2 и Python 3 фактически остановила любую разработку модуля на несколько лет. Более того, теперь просто распространять модуль как универсальное wheel, поскольку **2to3** больше не требуется. Поскольку Python 2.5 был выпущен много лет назад (в 2006 году), я счел разумным отказаться от его поддержки. Если вам нужна поддержка старых версий Python, используйте модуль декоратора версии 3.4.2. Текущая версия поддерживает все выпуски Python, начиная с 2.6.

## Что нового в версии 4

* **Новая документация** - Теперь существует единое руководство для всех версий Python, поэтому я воспользовался возможностью, чтобы пересмотреть документацию. Даже если вы являетесь давним пользователем, вы можете вернуться к документации, так как несколько примеров были улучшены.
* **Улучшения упаковки** - Код теперь также доступен в формате wheel. Интеграция с **setuptools** улучшилась, и вы также можете запускать тесты с помощью команды `python setup.py test`.
* **Изменения в коде** - Добавлена новая служебная функция `decorate (func, caller)`. Она выполняет ту же работу, что и более старый `decorator (caller, func)`. Старая функциональность устарела и больше не документирована, но пока доступна.
* **Множественная диспетчеризация** - Модуль **decorator** теперь включает реализацию общих функций (иногда называемых «множественными функциями диспетчеризации»). API разработан для имитации `functools.singledispatch` (добавлен в Python 3.4), но реализация намного проще. Более того, все задействованные декораторы сохраняют сигнатуру декорированных функций. На данный момент это существует в основном для демонстрации мощности модуля. В будущем он может быть улучшен / оптимизирован. В любом случае он очень короткий и компактный (менее 100 строк), поэтому вы можете извлечь его для собственного использования. Примите это как пищу для размышлений.
* **Сопрограммы Python 3.5** - Начиная с версии 4.1 появилась возможность декорировать сопрограммы, то есть функции, определенные с синтаксисом `async def`, и поддерживать проверку функции `inspect.iscoroutine`, работающую для декорированной функции.
* **Фабрики декораторов** - Начиная с версии 4.2 появилась возможность определять фабрики декораторов простым способом, что давно требовалось пользователями.

## Полезность декораторов

Декораторы Python - интересный пример того, почему синтаксический сахар имеет значение. В принципе, их введение в Python 2.4 ничего не изменило, поскольку они не предоставляли никаких новых функций, которых еще не было в языке. На практике их введение значительно изменило то, как мы структурируем наши программы на Python. Я считаю, что это изменение к лучшему, и декораторы - отличная идея, поскольку:

* декораторы помогают сократить шаблонный код;
* декораторы помогают разделить проблемы;
* декораторы повышают удобочитаемость и ремонтопригодность;
* декораторы явные.

Тем не менее, на данный момент для правильного написания собственных декораторов требуется некоторый опыт, и это не так просто, как могло бы быть. Например, типичные реализации декораторов включают вложенные функции, и все мы знаем, что плоские лучше, чем вложенные.

Цель модуля **decorator** - упростить использование декораторов для среднего программиста и популяризировать декораторы, показывая различные нетривиальные примеры. Конечно, как и все техники, декораторами можно злоупотреблять (я это видел), и вам не следует пытаться решить каждую проблему с помощью декоратора только потому, что вы можете.

Вы можете найти исходный код для всех примеров, обсуждаемых здесь, в файле `documentation.py`, который содержит документацию, которую вы читаете, в форме **doctests**.

## Определения

Технически говоря, любой объект Python, который можно вызвать с одним аргументом, можно использовать в качестве декоратора. Однако это определение слишком велико, чтобы быть действительно полезным. Общий класс декораторов удобнее разделить на два подкласса:

1. **сохраняющие сигнатуру декораторы**, вызываемые объекты, которые принимают функцию как вход и возвращают функцию как выход, _**с той же сигнатурой**_
2. **декораторы, изменяющие сигнатуру**, то есть декораторы, которые изменяют сигнатуру своей функции ввода, или декораторы, возвращающие невызываемые объекты

У декораторов с изменением подписи есть свое применение: например, в эту группу входят встроенные классы **staticmethod** и **classmethod**. Они принимают функции и возвращают объекты-дескрипторы, которые не являются ни функциями, ни вызываемыми объектами.

Тем не менее, декораторы, сохраняющие подпись, более распространены, и о них легче рассуждать. В частности, они могут быть составлены вместе, тогда как другие декораторы, как правило, не могут.

Написание сохраняющих сигнатуру декораторов с нуля не так очевидно, особенно если кто-то хочет определить правильные декораторы, которые могут принимать функции с любой сигнатурой. Простой пример прояснит проблему.

## Постановка задачи

Очень распространенный вариант использования декораторов - это запоминание функций. Декоратор **memoize** работает, кэшируя результат вызова функции в словаре, так что при следующем вызове функции с теми же входными параметрами результат извлекается из кеша и не вычисляется заново.

В [http://www.python.org/moin/PythonDecoratorLibrary](http://www.python.org/moin/PythonDecoratorLibrary) есть много реализаций **memoize**, но они не сохраняют сигнатуру. В последних версиях Python вы можете найти сложный декоратор **lru\_cache** в **functools** стандартной библиотеки. Здесь мне просто интересно привести пример.

Рассмотрим следующую простую реализацию (обратите внимание, что обычно невозможно правильно запоминать что-то, что зависит от нехешируемых аргументов):

```python
 def memoize_uw(func):
     func.cache = {}
 
     def memoize(*args, **kw):
         if kw:  # frozenset используется для обеспечения хешируемости
             key = args, frozenset(kw.items())
         else:
             key = args
         if key not in func.cache:
             func.cache[key] = func(*args, **kw)
         return func.cache[key]
     return functools.update_wrapper(memoize, func)
```

Здесь я использовал утилиту `functools.update_wrapper`, которая была добавлена в Python 2.5 для упрощения написания декораторов. (Раньше вам нужно было вручную скопировать атрибуты функции `__name__`, `__doc__`, `__module__` и `__dict__` в декорированную функцию).

Вот пример использования:

```python
 @memoize_uw
 def f1(x):
     "Смоделируйте долгое вычисление"
     time.sleep(1)
     return x
```

Это работает постольку, поскольку декоратор принимает функции с универсальными сигнатурами. К сожалению, это не декоратор, сохраняющий сигнатуру, поскольку **memoize\_uw** обычно возвращает функцию с сигнатурой, _**отличной от оригинала**_.

Рассмотрим, например, следующий случай:

```python
 @memoize_uw
 def f1(x):
     "Смоделируйте долгое вычисление"
     time.sleep(1)
     return x
```

Здесь исходная функция принимает единственный аргумент с именем **x**, но декорированная функция принимает любое количество аргументов и аргументов ключевого слова:

```python
>>> from decorator import getfullargspec
>>> print(getfullargspec(f1))
FullArgSpec(args=[], varargs='args', varkw='kw', defaults=None,
            kwonlyargs=[], kwonlydefaults=None, annotations={})
```

Это означает, что инструменты самоанализа (например, **pydoc**) будут давать ложную информацию о сигнатуре **f1** - если вы не используете Python 3.5. Это довольно плохо: **pydoc** сообщит вам, что функция принимает общую сигнатуру __ `*args, **kw`, но вызов функции с более чем одним аргументом вызывает ошибку:

```python
>>> f1(0, 1) 
Traceback (most recent call last):
   ...
TypeError: f1() takes exactly 1 positional argument (2 given)
```

Обратите внимание, что `inspect.getfullargspec` выдаст неправильную подпись даже в последней версии Python, то есть версии 3.6 на момент написания.

## Решение

Решение состоит в том, чтобы предоставить универсальную фабрику генераторов, которая скрывает сложность создания сохраняющих сигнатуру декораторов от прикладного программиста. Функция **decorate** в модуле **decorator** представляет собой такую фабрику:

```python
>>> from decorator import decorate
```

**decorate** принимает два аргумента:

1. вызывающая функция, описывающая функциональность декоратора, и
2. функция, которую нужно декорировать.

Вызывающая функция должна иметь сигнатуру `(f, *args, **kw)`, и она должна вызывать исходную функцию **f** с аргументами **args** и **kw**, реализуя желаемую возможность (в данном случае мемоизацию):

```python
 def _memoize(func, *args, **kw):
     if kw:  # frozenset используется для обеспечения хешируемости
         key = args, frozenset(kw.items())
     else:
         key = args
     cache = func.cache  # атрибут добавлен memoize
     if key not in cache:
         cache[key] = func(*args, **kw)
     return cache[key]
```

Теперь вы можете определить свой декоратор следующим образом:

```python
 def memoize(f):
     """
     Простая реализация Memoize.
     Она работает путем добавления словаря .cache к декорированной функции.
     Кеш будет расти бесконечно, поэтому вы обязаны очистить его, если это необходимо.
     """
     f.cache = {}
     return decorate(f, _memoize)
```

Отличие от подхода **memoize\_uw** к вложенным функциям заключается в том, что модуль декоратора заставляет вас поднять внутреннюю функцию на внешний уровень. Более того, вы вынуждены явно передавать функцию, которую хотите декорировать; замыканий нет.

Вот тест использования:

```python
>>> @memoize
... def heavy_computation():
...     time.sleep(2)
...     return "done"

>>> print(heavy_computation()) # первый раз это займет 2 секунды
done

>>> print(heavy_computation()) # во второй раз будет мгновенно
done
```

Подпись `heavy_computation` - это та, которую вы ожидаете:

```python
>>> print(getfullargspec(heavy_computation))
FullArgSpec(args=[], varargs=None, varkw=None, defaults=None, kwonlyargs=[],
kwonlydefaults=None, annotations={})
```

## Декоратор trace

Вот пример того, как определить простой декоратор **trace**, который печатает сообщение всякий раз, когда вызывается отслеживаемая функция:

```python
 def _trace(f, *args, **kw):
     kwstr = ', '.join('%r: %r' % (k, kw[k]) for k in sorted(kw))
     print("calling %s with args %s, {%s}" % (f.__name__, args, kwstr))
     return f(*args, **kw)
```

```python
 def trace(f):
     return decorate(f, _trace)
```

Вот пример использования:

```python
>>> @trace
... def f1(x):
...     pass
```

Немедленно проверить, что **f1** работает ...

```python
>>> f1(0)
calling f1 with args (0,), {}
```

... и если у него правильная подпись:

```python
>>> print(getfullargspec(f1))
FullArgSpec(args=['x'], varargs=None, varkw=None, defaults=None,
            kwonlyargs=[], kwonlydefaults=None, annotations={})
```

Декоратор работает с функциями любой сигнатуры:

```python
>>> @trace
... def f(x, y=1, z=2, *args, **kw):
...     pass

>>> f(0, 3)
calling f with args (0, 3, 2), {}

>>> print(getfullargspec(f))
FullArgSpec(args=['x', 'y', 'z'], varargs='args', varkw='kw',
            defaults=(1, 2), kwonlyargs=[], kwonlydefaults=None, annotations={})
```

## Аннотации функций

Python 3 представил концепцию [аннотаций функций](https://www.python.org/dev/peps/pep-3107/): возможность аннотировать подпись функции дополнительной информацией, хранящейся в словаре с именем `__annotations__`. Модуль **decorator** (начиная с версии 3.3) будет понимать и сохранять эти аннотации.

Вот пример:

```python
>>> @trace
... def f(x: 'the first argument', y: 'default argument'=1, z=2,
...       *args: 'varargs', **kw: 'kwargs'):
...     pass
```

Чтобы анализировать функции с помощью аннотаций, нужна утилита `inspect.getfullargspec` (введенная в Python 3, затем объявленная устаревшей в Python 3.5, а затем не поддерживаемая в Python 3.6):

```python
>>> from inspect import getfullargspec
>>> argspec = getfullargspec(f)
>>> argspec.args
['x', 'y', 'z']
>>> argspec.varargs
'args'
>>> argspec.varkw
'kw'
>>> argspec.defaults
(1, 2)
>>> argspec.kwonlyargs
[]
>>> argspec.kwonlydefaults
```

Вы можете проверить, что словарь `__annotations__` сохранен:

```python
>>> f.__annotations__ is f.__wrapped__.__annotations__
True
```

Здесь `f.__wrapped__` - это исходная функция без декорирования. Этот атрибут существует для согласованности с поведением `functools.update_wrapper`.

Другой атрибут, скопированный из исходной функции, - это полное имя `__qualname__`. Этот атрибут был введен в Python 3.3.

## decorator.decorator

Может оказаться утомительным писать каждый раз вызывающую функцию (как в приведенном выше примере `_trace`), а затем тривиальную оболочку (`def trace (f): return decorate (f, _trace)`). Не беспокойтесь! Модуль **decorator** предоставляет простой ярлык для преобразования вызывающей функции в декоратор, сохраняющий сигнатуру.

Это функция **decorator**:

```python
>>> from decorator import decorator
>>> print(decorator.__doc__)
decorator(caller) converts a caller function into a decorator
```

Функцию **decorator** можно использовать как декоратор, изменяющий сигнатуру, точно так же, как **classmethod** и **staticmethod**. Но **classmethod** и **staticmethod** возвращают общие объекты, которые нельзя вызвать. Вместо этого **decorator** возвращает декораторы, сохраняющие сигнатуру (то есть функции с одним аргументом).

Например, вы можете написать:

```python
>>> @decorator
... def trace(f, *args, **kw):
...     kwstr = ', '.join('%r: %r' % (k, kw[k]) for k in sorted(kw))
...     print("calling %s with args %s, {%s}" % (f.__name__, args, kwstr))
...     return f(*args, **kw)
```

И **trace** теперь декоратор!

```python
>>> trace 
<function trace at 0x...>
```

Вот пример использования:

```python
>>> @trace
... def func(): pass

>>> func()
calling func with args (), {}
```

## Фабрики декораторов

Функция **decorator** также может использоваться для определения фабрик декораторов, то есть функций, возвращающих декораторы. В общем, можно написать примерно так:

```python
def decfactory(param1, param2, ...):
    def caller(f, *args, **kw):
        return somefunc(f, param1, param2, .., *args, **kw)
    return decorator(caller)
```

Это вполне общий характер, но требует дополнительного уровня вложенности. По этой причине, начиная с версии 4.2, есть возможность создавать фабрики декораторов с использованием одного вызывающего объекта с аргументами по умолчанию, то есть записывая что-то вроде этого:

```python
def caller(f, param1=default1, param2=default2, ..., *args, **kw):
    return somefunc(f, param1, param2, *args, **kw)
decfactory = decorator(caller)
```

Обратите внимание, что этот упрощенный подход работает _**только с аргументами по умолчанию**_, т.е. `param1`, `param2` и т. д. должны иметь известные значения по умолчанию. Благодаря этому ограничению существует уникальный декоратор по умолчанию, то есть член семейства, который использует значения по умолчанию для всех параметров. Такой декоратор можно записать как `decfactory ()` без указания параметров; кроме того, в качестве ярлыка можно также опустить скобку, что очень востребовано пользователями. В течение многих лет я возражал против этой просьбы, поскольку наличие явных скобок для меня более ясно и менее волшебно; однако, как только эта функция вошла в декораторы стандартной библиотеки Python (я имею в виду [декоратор класса данных](https://www.python.org/dev/peps/pep-0557/)), я наконец сдался.

```python
 @decorator
 def blocking(f, msg='blocking', *args, **kw):
     if not hasattr(f, "thread"):  # нет потока
         def set_result():
             f.result = f(*args, **kw)
         f.thread = threading.Thread(None, set_result)
         f.thread.start()
         return msg
     elif f.thread.is_alive():
         return msg
     else:  # поток завершен, вернуть сохраненный результат
         del f.thread
         return f.result
```

Функции, отмеченные блокировкой, возвращают сообщение о занятости, если ресурс недоступен, и ожидаемый результат, если ресурс доступен. Например:

```python
>>> @blocking(msg="Please wait ...")
... def read_data():
...     time.sleep(3) # имитировать блокирующий ресурс
...     return "some data"

>>> print(read_data())  # данные пока недоступны
Please wait ...

>>> time.sleep(1)
>>> print(read_data())  # данные пока недоступны
Please wait ...

>>> time.sleep(1)
>>> print(read_data())  # данные пока недоступны
Please wait ...

>>> time.sleep(1.1)  # через 3,1 секунды данные становятся доступными
>>> print(read_data())
some data
```

Фабрики декораторов наиболее полезны для разработчиков фреймворков. Вот пример, который дает представление о том, как вы можете управлять разрешениями в фреймворке:

```python
 class Action(object):
     @restricted(user_class=User)
     def view(self):
         "Любой пользователь может просматривать объекты"
 
     @restricted(user_class=PowerUser)
     def insert(self):
         "Только опытные пользователи могут вставлять объекты"
 
     @restricted(user_class=Admin)
     def delete(self):
         "Только администратор может удалять объекты"
```

где **restricted** - это фабрика декораторов, определенная следующим образом

```python
 @decorator
 def restricted(func, user_class=User, *args, **kw):
     "Ограничить доступ к определенному классу пользователей"
     self = args[0]
     if isinstance(self.user, user_class):
         return func(*args, **kw)
     else:
         raise PermissionError(
             '%s does not have the permission to run %s!'
             % (self.user, func.__name__))
```

Обратите внимание, что если вы забудете использовать обозначение аргумента ключевого слова, т.е. если вы напишете `restricted (User)` вместо `restricted (user_class = User)`, вы получите сообщение об ошибке

```python
TypeError: You are decorating a non function: <class '__main__.User'>
```

Будь осторожен!
