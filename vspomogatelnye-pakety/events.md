# Events

_Привнесение элегантности C# EventHandler в Python_

Концепция событий широко используется в библиотеках GUI и является основой для большинства реализаций шаблона проектирования MVC (модель, представление, контроллер). Еще одно известное использование событий — в стеках протоколов связи, где нижние уровни протоколов должны информировать верхние уровни о входящих данных и т.п. Вот удобный класс, который инкапсулирует ядро для подписки на события и запуска событий и выглядит как «естественная» часть языка.

Пакет был протестирован на Python 2.6, 2.7, 3.3 и 3.4.

## Использование

Язык C# предоставляет удобный способ объявлять, подписываться на события и запускать их. Технически событие представляет собой «слот», к которому могут быть прикреплены функции обратного вызова (обработчики событий) — процесс, называемый подпиской на событие. Чтобы подписаться на событие:

```python
>>> def something_changed(reason):
...     print "something changed because %s" % reason
...

>>> from events import Events
>>> events = Events()
>>> events.on_change += something_changed
```

Несколько функций обратного вызова могут подписаться на одно и то же событие. Когда событие запускается, последовательно вызываются все подключенные обработчики событий. Чтобы запустить событие, выполните вызов слота:

```python
>>> events.on_change('it had to happen')
something changed because it had to happen
```

Обычно экземпляры **Events** не будут свободно болтаться, как показано выше, а будут встроены в объекты модели, как здесь:

```python
class MyModel(object):
    def __init__(self):
        self.events = Events()
        ...
```

Точно так же объекты представления и контроллера будут основными подписчиками событий:

```python
class MyModelView(SomeWidget):
    def __init__(self, model):
        ...
        self.model = model
        model.events.on_change += self.display_value
        ...

    def display_value(self):
        ...
```

### Отмена подписки

Может наступить время, когда вы больше не захотите получать уведомления о событии. В этом случае вы отменяете подписку естественным аналогом `+=`, используя `-=`.

```python
# Мы больше не хотим получать уведомления,
# исключите нас из списка обратных вызовов событий
>>> events.on_change -= something_changed
```

Вы также можете отказаться от подписки по причинам управления памятью. Экземпляр **Events()** будет содержать ссылку **something\_changed**. Если это метод-член объекта, а время жизни экземпляра **Events()** больше, чем у этого объекта, он будет поддерживать его дольше, чем в обычном случае.

## Интроспекция

Классы **Events** и **\_EventSlot** обеспечивают некоторую поддержку самоанализа. Это полезно, например, для автоматической подписки на события на основе шаблонов имен методов.

```python
>>> from events import Events
>>> events = Events()
>>> print events
<events.events.Events object at 0x104e5d5f0>

>>> def changed():
...     print "something changed"
...

>>> def another_one():
...     print "something changed here too"
...

>>> def deleted():
...     print "something got deleted!"
...

>>> events.on_change += changed
>>> events.on_change += another_one
>>> events.on_delete += deleted
# два вида событий (on_change и on_delete)
>>> print len(events)
2

>>> for event in events:
...     print event.__name__
...
on_change
on_delete

>>> event = events.on_change
>>> print event
event 'on_change'
# сколько callback подписано на событие on_change ?
>>> print len(event)
2

>>> for handler in event:
...     print handler.__name__
...
changed
another_one

>>> print event[0]
<function changed at 0x104e5c230>

>>> print event[0].__name__
changed

>>> print len(events.on_delete)
1

>>> events.on_change()
something changed
somethind changed here too

>>> events.on_delete()
something got deleted!
```

## Имена события

Обратите внимание, что по умолчанию **Events** не проверяет, действительно ли событие, на которое вы подписываетесь, может быть запущено, если не определен атрибут класса **\_\_events\_\_**. Это может вызвать проблему, если имя события написано с небольшой ошибкой. Если это проблема, создайте подкласс **Events** и перечислите возможные события, например:

```python
class MyEvents(Events):
    __events__ = ('on_this', 'on_that', )

events = MyEvents()

# будет поднимать EventsException, так как `on_change` неизвестно в MyEvents:
events.on_change += changed
```

Вы также можете предварительно определить события для одного экземпляра **Events**, передав итератор конструктору.

```python
events = Events(('on_this', 'on_that'))

# будет поднимать EventsException, так как `on_change` неизвестно в MyEvents:
events.on_change += changed
```

Рекомендуется использовать метод конструктора для одноразовых случаев использования. Для более сложных вариантов использования рекомендуется создать подкласс **Events** и определить **\_\_events\_\_**.

Вы также можете использовать как метод конструктора, так и атрибут **\_\_events\_\_**, чтобы ограничить события для определенных экземпляров:

```python
DatabaseEvents(Events):
    __events__ = ('insert', 'update', 'delete', 'select')

audit_events = ('select')

AppDatabaseEvents = DatabaseEvents()

# известно только событие 'select' из DatabaseEvents
AuditDatabaseEvents = DatabaseEvents(audit_events)
```

## Установка

**Events** находится на PyPI, поэтому все, что вам нужно сделать, это:

```bash
pip install events
```

## Тестирование

Просто запустите:

```bash
python setup.py test
```

Пакет был протестирован на Python 2.6, Python 2.7 и Python 3.3.

## Исходный код

Исходный код доступен на [GitHub](https://github.com/pyeve/events).

## Атрибуция

На основе превосходного рецепта [Zoran Isailovski](http://code.activestate.com/recipes/410686/), Copyright (c) 2005.

## Уведомление об авторских правах

Это проект с открытым исходным кодом [Nicola Iarocci](http://nicolaiarocci.com). См. исходную [ЛИЦЕНЗИЮ](https://github.com/pyeve/events/blob/master/LICENSE) для получения дополнительной информации.
