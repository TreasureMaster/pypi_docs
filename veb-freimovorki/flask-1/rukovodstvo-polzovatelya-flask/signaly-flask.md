# Сигналы Flask

_Новое в версии 0.6_.

Начиная с `Flask 0.6`, в **Flask** есть встроенная поддержка сигнализации. Эта поддержка обеспечивается отличной библиотекой мигалок и будет изящно откатиться, если она недоступна.

Что такое сигналы? Сигналы помогают разделить приложения, отправляя уведомления, когда действия происходят где-то в другом месте базовой структуры или других расширений **Flask**. Короче говоря, сигналы позволяют определенным отправителям уведомлять подписчиков о том, что что-то произошло.

**Flask** поставляется с парой сигналов, а другие расширения могут предоставить больше. Также имейте в виду, что сигналы предназначены для уведомления подписчиков и не должны побуждать подписчиков изменять данные. Вы заметите, что есть сигналы, которые, похоже, делают то же самое, что и некоторые из встроенных декораторов (например: [request\_started](../api-dokumentaciya-flask/signaly-flask.md#flask-request\_started) очень похож на [before\_request ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#before\_request)). Однако есть различия в том, как они работают. Например, основной обработчик **before\_request ()** выполняется в определенном порядке и может преждевременно прервать запрос, вернув ответ. Напротив, все обработчики сигналов выполняются в неопределенном порядке и не изменяют никаких данных.

Большим преимуществом сигналов перед обработчиками является то, что вы можете безопасно подписаться на них всего за доли секунды. Эти временные подписки полезны, например, для модульного тестирования. Допустим, вы хотите знать, какие шаблоны были отрисованы как часть запроса: сигналы позволяют сделать именно это.

## Подписка на сигналы

Чтобы подписаться на сигнал, вы можете использовать метод сигнала [connect ()](https://pythonhosted.org/blinker/index.html#blinker.base.Signal.connect). Первый аргумент - это функция, которая должна вызываться при передаче сигнала, второй необязательный аргумент указывает отправителя. Чтобы отказаться от подписки на сигнал, вы можете использовать метод [disconnect ()](https://pythonhosted.org/blinker/index.html#blinker.base.Signal.disconnect).

Для всех основных сигналов **Flask** отправителем является приложение, которое отправило сигнал. Когда вы подписываетесь на сигнал, обязательно укажите отправителя, если вы действительно не хотите прослушивать сигналы от всех приложений. Это особенно верно, если вы разрабатываете расширение.

Например, вот вспомогательный диспетчер контекста, который можно использовать в модульном тесте, чтобы определить, какие шаблоны были отрисованы и какие переменные были переданы в шаблон:

```python
from flask import template_rendered
from contextlib import contextmanager

@contextmanager
def captured_templates(app):
    recorded = []
    def record(sender, template, context, **extra):
        recorded.append((template, context))
    template_rendered.connect(record, app)
    try:
        yield recorded
    finally:
        template_rendered.disconnect(record, app)
```

Теперь это можно легко связать с тестовым клиентом:

```python
with captured_templates(app) as templates:
    rv = app.test_client().get('/')
    assert rv.status_code == 200
    assert len(templates) == 1
    template, context = templates[0]
    assert template.name == 'index.html'
    assert len(context['items']) == 10
```

Обязательно подпишитесь с дополнительным `**extra` аргументом, чтобы ваши вызовы не завершились ошибкой, если **Flask** вводит новые аргументы в сигналы.

Вся отрисовка шаблона в коде, выдаваемом приложением приложения в теле блока **with**, теперь будет записана в переменной **templates**. Каждый раз, когда шаблон отображается, к нему добавляются объект шаблона, а также контекст.

Кроме того, существует удобный вспомогательный метод ([connected\_to ()](https://pythonhosted.org/blinker/index.html#blinker.base.Signal.connected\_to)), который позволяет вам временно подписать функцию на сигнал с помощью диспетчера контекста самостоятельно. Поскольку возвращаемое значение диспетчера контекста не может быть указано таким образом, вы должны передать список в качестве аргумента:

```python
from flask import template_rendered

def captured_templates(app, recorded, **extra):
    def record(sender, template, context):
        recorded.append((template, context))
    return template_rendered.connected_to(record, app)
```

Приведенный выше пример будет выглядеть так:

```python
templates = []
with captured_templates(app, templates, **extra):
    ...
    template, context = templates[0]
```

{% hint style="info" %}
**Изменения API Blinker:**

Метод [connected\_to ()](https://pythonhosted.org/blinker/index.html#blinker.base.Signal.connected\_to) появился в **Blinker** с версией **1.1**.
{% endhint %}

## Создание сигналов

Если вы хотите использовать сигналы в своем собственном приложении, вы можете напрямую использовать библиотеку **blinker**. Чаще всего используются именованные сигналы в пользовательском пространстве имен [Namespace](https://pythonhosted.org/blinker/index.html#blinker.base.Namespace). В большинстве случаев это рекомендуется:

```python
from blinker import Namespace
my_signals = Namespace()
```

Теперь вы можете создавать такие новые сигналы:

```python
model_saved = my_signals.signal('model-saved')
```

Имя сигнала здесь делает его уникальным, а также упрощает отладку. Вы можете получить доступ к имени сигнала с помощью атрибута [**name**](https://pythonhosted.org/blinker/index.html#blinker.base.NamedSignal.name).

{% hint style="info" %}
**Для разработчиков расширений:**

Если вы пишете расширение **Flask** и хотите изящно выполнить деградацию из-за отсутствующих установок **blinker**, вы можете сделать это с помощью класса [flask.signals.Namespace](../api-dokumentaciya-flask/signaly-flask.md#klass-signals-namespace).
{% endhint %}

## Отправка сигналов

Если вы хотите послать сигнал, вы можете сделать это, вызвав метод [send ()](https://pythonhosted.org/blinker/index.html#blinker.base.Signal.send). Он принимает отправителя в качестве первого аргумента и, возможно, некоторые аргументы ключевого слова, которые передаются подписчикам сигнала:

```python
class Model(object):
    ...

    def save(self):
        model_saved.send(self)
```

Старайтесь всегда выбирать хорошего отправителя. Если у вас есть класс, излучающий сигнал, передайте **self** как отправителя. Если вы испускаете сигнал от случайной функции, вы можете передать `current_app._get_current_object ()` как отправитель.

{% hint style="info" %}
**Передача прокси в качестве отправителя:**

Никогда не передавайте [current\_app](../api-dokumentaciya-flask/poleznye-funkcii-i-klassy-flask.md#flask-current\_app) в качестве отправителя сигнала. Вместо этого используйте **current\_app.\_get\_current\_object ()**. Причина в том, что **current\_app** - это прокси, а не реальный объект приложения.
{% endhint %}

## Сигналы и контекст запроса Flask

Сигналы полностью поддерживают [контекст запроса Request](kontekst-zaprosa-flask.md) при получении сигналов. Локальные контекстные переменные постоянно доступны между [request\_started](../api-dokumentaciya-flask/signaly-flask.md#flask-request\_started) и [request\_finished](../api-dokumentaciya-flask/signaly-flask.md#flask-request\_finished), поэтому вы можете полагаться на [flask.g](../api-dokumentaciya-flask/globalnyi-obekt-prilozheniya-flask.md#flask-g) и другие по мере необходимости. Обратите внимание на ограничения, описанные в разделах «[Отправка сигналов](signaly-flask.md#otpravka-signalov)» и «Сигнал [request\_tearing\_down](../api-dokumentaciya-flask/signaly-flask.md#flask-request\_tearing\_down)».

## Подписки на сигналы на основе декораторов

В `Blinker 1.1` вы также можете легко подписаться на сигналы с помощью нового декоратора **connect\_via ()**:

```python
from flask import template_rendered

@template_rendered.connect_via(app)
def when_template_rendered(sender, template, context, **extra):
    print 'Template %s is rendered with %s' % (template.name, context)
```

## Основные сигналы

Взгляните на [Сигналы](../api-dokumentaciya-flask/signaly-flask.md), чтобы получить список всех встроенных сигналов.
