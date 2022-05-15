# Быстрый старт Click

Вы можете получить библиотеку прямо из PyPI:

```bash
pip install click
```

Настоятельно рекомендуется установка в [virtualenv](bystryi-start-click.md#virtualenv).

## virtualenv

**virtualenv**, вероятно, то, что вы хотите использовать для разработки приложений **Click**.

Какую проблему решает **virtualenv**? Скорее всего, вы захотите использовать его для других проектов, помимо вашего скрипта **Click**. Но чем больше у вас проектов, тем больше вероятность, что вы будете работать с разными версиями самого Python или, по крайней мере, с разными версиями библиотек Python. Посмотрим правде в глаза: довольно часто библиотеки нарушают обратную совместимость, и маловероятно, что какое-либо серьезное приложение будет иметь нулевые зависимости. Итак, что вы будете делать, если у двух или более ваших проектов есть конфликтующие зависимости?

**Virtualenv** спешат на помощь! **Virtualenv** позволяет устанавливать несколько одновременно установленных Python, по одной для каждого проекта. На самом деле он не устанавливает отдельные копии Python, но предоставляет разумный способ изолировать различные среды проекта. Посмотрим, как работает **virtualenv**.

Если вы используете Mac OS X или Linux, скорее всего, вам подойдет одна из следующих двух команд:

```bash
$ sudo easy_install virtualenv
```

или даже лучше:

```bash
$ pip install virtualenv --user
```

Один из них, вероятно, установит **virtualenv** в вашей системе. Может, дело даже в вашем диспетчере пакетов. Если вы используете Ubuntu, попробуйте:

```bash
$ sudo apt-get install python-virtualenv
```

Если вы работаете в Windows (или ни один из вышеперечисленных методов не сработал), вы должны сначала установить **pip**. Для получения дополнительной информации об этом см. [Установка pip](https://pip.pypa.io/en/latest/installing/). После его установки запустите команду **pip** сверху, но без префикса **sudo**.

После того, как вы установили **virtualenv**, просто запустите оболочку и создайте свою собственную среду. Обычно я создаю папку проекта и папку **venv** внутри:

```bash
$ mkdir myproject
$ cd myproject
$ virtualenv venv
New python executable in venv/bin/python
Installing setuptools, pip............done.
```

Теперь, когда вы хотите работать над проектом, вам нужно только активировать соответствующую среду. В OS X и Linux сделайте следующее:

```bash
$ . venv/bin/activate
```

Если вы пользователь Windows, вам подойдет следующая команда:

```bash
$ venv\scripts\activate
```

В любом случае, теперь вы должны использовать свой **virtualenv** (обратите внимание, как приглашение вашей оболочки изменилось, чтобы показать активную среду).

А если вы хотите вернуться в реальный мир, используйте следующую команду:

```bash
$ deactivate
```

После этого приглашение вашей оболочки должно быть таким же знакомым, как и раньше.

А теперь пойдем дальше. Введите следующую команду, чтобы активировать **Click** в вашем виртуальном окружении:

```bash
$ pip install Click
```

Несколько секунд спустя, и все готово.

## Скринкаст и примеры

Доступен скринкаст, показывающий базовый API **Click** и способ создания с его помощью простых приложений. Он также исследует, как создавать команды с помощью подкоманд.

* [Создание приложений командной строки с помощью Click](https://www.youtube.com/watch?v=kNke39OZ2k0)

Примеры приложений **Click** можно найти в документации, а также в репозитории GitHub вместе с файлами readme:

* **inout:** [Файловый ввод и вывод](https://github.com/pallets/click/tree/master/examples/inout)
* **naval:** [Порт docopt военно-морской пример](https://github.com/pallets/click/tree/master/examples/naval)
* **aliases:** [Пример псевдонима команды](https://github.com/pallets/click/tree/master/examples/aliases)
* **repo:** [Интерфейс командной строки, подобный Git / Mercurial](https://github.com/pallets/click/tree/master/examples/repo)
* **complex:** [Сложный пример с загрузкой плагина](https://github.com/pallets/click/tree/master/examples/complex)
* **validation:** [Пример проверки специальных параметров](https://github.com/pallets/click/tree/master/examples/validation)
* **colors:** [Поддержка цвета Colorama ANSI](https://github.com/pallets/click/tree/master/examples/colors)
* **termui:** [Демонстрация функций пользовательского интерфейса терминала](https://github.com/pallets/click/tree/master/examples/termui)
* **imagepipe:** [Демонстрация объединения нескольких команд](https://github.com/pallets/click/tree/master/examples/imagepipe)

## Основные понятия - создание команды

**Click** основан на объявлении команд через декораторы. Внутри есть интерфейс без декоратора для сложных случаев использования, но он не рекомендуется для высокоуровневого использования.

Функция становится инструментом командной строки **Click**, декорируя ее с помощью [click.command ()](../api-click/dekoratory-click-api.md#click-command). В самом простом случае, если просто декорировать функцию этим декоратором, она превратится в вызываемый скрипт:

```python
import click

@click.command()
def hello():
    click.echo('Hello World!')
```

Происходит то, что декоратор преобразует функцию в команду [Command](../api-click/komandy-click-api.md#class-click-command), которая затем может быть вызвана:

```python
if __name__ == '__main__':
    hello()
```

И как это выглядит:

```bash
$ python hello.py
Hello World!
```

И соответствующая страница помощи:

```bash
$ python hello.py --help
Usage: hello.py [OPTIONS]

Options:
  --help  Show this message and exit.
```

## Вывод echo

Почему в этом примере используется [echo ()](../api-click/utility-click-api.md#click-echo) вместо обычной функции [print ()](https://docs.python.org/3/library/functions.html#print)? Ответ на этот вопрос заключается в том, что **Click** пытается поддерживать как Python 2, так и Python 3 одинаково и быть очень надежным даже при неправильной настройке среды. **Click** хочет быть работоспособным хотя бы на базовом уровне, даже если все полностью сломано.

Это означает, что функция [echo ()](../api-click/utility-click-api.md#click-echo) применяет некоторую коррекцию ошибок в случае неправильной настройки терминала вместо того, чтобы умереть с [UnicodeError](https://docs.python.org/3/library/exceptions.html#UnicodeError).

В качестве дополнительного преимущества, начиная с `Click 2.0`, функция **echo** также хорошо поддерживает цвета ANSI. Он автоматически удаляет коды ANSI, если выходной поток является файлом, и если поддерживается **colorama**, цвета ANSI также будут работать в Windows. Обратите внимание, что в Python 2 функция [echo ()](../api-click/utility-click-api.md#click-echo) не анализирует информацию о цветовом коде из массивов байтов. Для получения дополнительной информации см. [Цвета ANSI](utility-click.md#cveta-ansi).

Если вам это не нужно, вы также можете использовать конструкцию / функцию **print ()**.

## Команды вложенности

Команды могут быть присоединены к другим командам типа [Group](../api-click/komandy-click-api.md#class-click-group). Это позволяет произвольно размещать скрипты. В качестве примера приведем скрипт, реализующий две команды для управления базами данных:

```python
@click.group()
def cli():
    pass

@click.command()
def initdb():
    click.echo('Initialized the database')

@click.command()
def dropdb():
    click.echo('Dropped the database')

cli.add_command(initdb)
cli.add_command(dropdb)
```

Как видите, декоратор [group ()](../api-click/dekoratory-click-api.md#click-group) работает как декоратор [command ()](../api-click/dekoratory-click-api.md#click-command), но вместо этого создает объект [Group](../api-click/komandy-click-api.md#class-click-group), которому можно дать несколько подкоманд, которые можно присоединить с помощью [Group.add\_command ()](../api-click/komandy-click-api.md#add\_command).

Для простых скриптов также возможно автоматически присоединить и создать команду, используя вместо этого декоратор [Group.command ()](../api-click/komandy-click-api.md#command). Вместо этого приведенный выше сценарий можно записать так:

```python
@click.group()
def cli():
    pass

@cli.command()
def initdb():
    click.echo('Initialized the database')

@cli.command()
def dropdb():
    click.echo('Dropped the database')
```

Затем вы должны вызвать группу в точках входа в **setuptools** или других вызовах:

```python
if __name__ == '__main__':
    cli()
```

## Добавление параметров

Чтобы добавить параметры, используйте декораторы [option ()](../api-click/dekoratory-click-api.md#click-option) и [argument ()](../api-click/dekoratory-click-api.md#click-argument):

```python
@click.command()
@click.option('--count', default=1, help='number of greetings')
@click.argument('name')
def hello(count, name):
    for x in range(count):
        click.echo('Hello %s!' % name)
```

На что это похоже:

```bash
$ python hello.py --help
Usage: hello.py [OPTIONS] NAME

Options:
  --count INTEGER  number of greetings
  --help           Show this message and exit.
```

## Переход к setuptools

В коде, который вы написали до сих пор, есть блок в конце файла, который выглядит следующим образом: `if __name__ == '__main__':`. Традиционно так выглядит автономный файл Python. С помощью **Click** вы можете продолжать это делать, но есть способы лучше с помощью инструментов настройки.

Для этого есть две основные (и многие другие) причины:

Во-первых, **setuptools** автоматически генерирует исполняемые оболочки для Windows, поэтому ваши утилиты командной строки работают и в Windows.

Вторая причина заключается в том, что сценарии **setuptools** работают с **virtualenv** в Unix без необходимости активации **virtualenv**. Это очень полезная концепция, которая позволяет объединить ваши скрипты со всеми требованиями в файл **virtualenv**.

**Click** идеально подходит для работы с этим, и на самом деле остальная часть документации предполагает, что вы пишете приложения с помощью **setuptools**.

Я настоятельно рекомендую ознакомиться с главой «Интеграция с Setuptools», прежде чем читать остальные, поскольку в примерах предполагается, что вы будете использовать **setuptools**.
