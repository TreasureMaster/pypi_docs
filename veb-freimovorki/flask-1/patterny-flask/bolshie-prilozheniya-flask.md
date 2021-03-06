# Большие приложения Flask

Представьте себе простую структуру приложения Flask, которая выглядит так:

```bash
/yourapplication
    yourapplication.py
    /static
        style.css
    /templates
        layout.html
        index.html
        login.html
        ...
```

Хотя это нормально для небольших приложений, для больших приложений рекомендуется использовать пакет вместо модуля. [Учебник](../rukovodstvo-polzovatelya-flask/uchebnik-flask.md) построен с использованием шаблона пакета, см. [пример кода](https://github.com/pallets/flask/tree/1.1.2/examples/tutorial).

## Простые пакеты

Чтобы преобразовать его в более крупный, просто создайте новую папку **yourapplication** внутри существующей и переместите все под ней. Затем переименуйте `yourapplication.py` в `__init__.py`. (Не забудьте сначала удалить все файлы `.pyc`, иначе все, скорее всего, сломается)

У вас должно получиться что-то вроде этого:

```bash
/yourapplication
    /yourapplication
        __init__.py
        /static
            style.css
        /templates
            layout.html
            index.html
            login.html
            ...
```

Но как теперь запустить приложение? Нативный `python yourapplication/__init__.py` работать не будет. Скажем так, Python не хочет, чтобы модули в пакетах были файлом запуска. Но это не большая проблема, просто добавьте новый файл с именем `setup.py` рядом с внутренней папкой `yourapplication` со следующим содержимым:

```python
from setuptools import setup

setup(
    name='yourapplication',
    packages=['yourapplication'],
    include_package_data=True,
    install_requires=[
        'flask',
    ],
)
```

Чтобы запустить приложение, вам необходимо экспортировать переменную среды, которая сообщает **Flask**, где найти экземпляр приложения:

```bash
$ export FLASK_APP=yourapplication
```

Если вы находитесь за пределами каталога проекта, обязательно укажите точный путь к каталогу вашего приложения. Точно так же вы можете включить такие функции разработки:

```bash
$ export FLASK_ENV=development
```

Чтобы установить и запустить приложение, вам необходимо выполнить следующие команды:

```bash
$ pip install -e .
$ flask run
```

Что мы получили от этого? Теперь мы можем немного реструктурировать приложение на несколько модулей. Единственное, что вам нужно запомнить, это следующий быстрый контрольный список:

1. создание объекта приложения **Flask** должно быть в файле `__init__.py`. Таким образом, каждый модуль может безопасно импортировать его, а переменная `__name__` будет преобразована в правильный пакет.
2. все функции просмотра (с декоратором [route ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#route) наверху) должны быть импортированы в файл `__init__.py`. Не сам объект, а модуль, в котором он находится. Импортируйте модуль представления **после создания объекта приложения**.

Вот пример `__init__.py`:

```python
from flask import Flask
app = Flask(__name__)

import yourapplication.views
```

А вот как будет выглядеть `views.py`:

```python
from yourapplication import app

@app.route('/')
def index():
    return 'Hello World!'
```

У вас должно получиться что-то вроде этого:

```bash
/yourapplication
    setup.py
    /yourapplication
        __init__.py
        views.py
        /static
            style.css
        /templates
            layout.html
            index.html
            login.html
            ...
```

{% hint style="info" %}
**Циклический импорт:**

Каждый программист Python ненавидит их, но мы просто добавили: циклический импорт (это когда два модуля зависят друг от друга. В этом случае **views.py** зависит от **\_\_init\_\_.py**). Имейте в виду, что это в целом плохая идея, но в данном случае это нормально. Причина этого в том, что мы фактически не используем представления в **\_\_init\_\_.py**, а просто обеспечиваем импорт модуля, а делаем это в нижней части файла.

Есть еще некоторые проблемы с этим подходом, но если вы хотите использовать декораторы, этого не избежать. Ознакомьтесь с разделом «[Стать большим](../rukovodstvo-polzovatelya-flask/eshe-bolshe-o-flask.md)», чтобы узнать, как с этим справиться.
{% endhint %}

## Работа с Blueprints

Если у вас есть более крупные приложения, рекомендуется разделить их на более мелкие группы, где каждая группа реализуется с помощью _**blueprint**_. Для мягкого введения в эту тему обратитесь к главе документации [Модульные приложения с Blueprints](../rukovodstvo-polzovatelya-flask/modulnye-prilozheniya-s-blueprints.md).
