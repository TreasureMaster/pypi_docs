# Создание пакета

## Основы: создание и распространение дистрибутивов

Если у вас есть несколько полезных [модулей](glossarii.md#module) Python, которые, по вашему мнению, могут принести пользу другим, но не знаете, как их упаковывать и распространять, тогда этот короткий документ для вас. По его окончании вы станете участником [индекса пакетов Python (PyPI)](publikaciya-paketa.md#the-python-package-index-pypi).

Для более подробного ознакомления с упаковкой более крупного проекта см. [этот пример](https://the-hitchhikers-guide-to-packaging.readthedocs.io/en/latest/example.html).

Давайте начнем.

### Предпосылка

Предположим, вы написали пару модулей, которые помогут вам следить за своим полотенцем (`location.py` и `utils.py`), и хотите поделиться ими. Первое, что нужно сделать, это придумать для них название проекта в стиле **CamelCase**. Давайте перейдем к «**TowelStuff**», так как он кажется подходящим и еще не использовался в [индексе пакетов Python (PyPI)](publikaciya-paketa.md#the-python-package-index-pypi).

### Организация структуры файлов и каталогов

«**TowelStuff**» будет названием нашего проекта, а также названием нашего [дистрибутива](glossarii.md#distribution). Мы также должны придумать имя пакета, в котором будут размещаться наши модули (чтобы избежать конфликтов имен с другими модулями). В этом примере есть только один [пакет](glossarii.md#package), поэтому давайте повторно используем название проекта и выберем «towelstuff». Сделайте макет каталога вашего проекта (описанного ниже) таким:

```bash
TowelStuff/
    bin/
    CHANGES.txt
    docs/
    LICENSE.txt
    MANIFEST.in
    README.txt
    setup.py
    towelstuff/
        __init__.py
        location.py
        utils.py
        test/
            __init__.py
            test_location.py
            test_utils.py
```

Вот что вам следует сделать для каждого из перечисленных выше:

* Поместите в `bin/` все написанные вами сценарии, которые используют ваш пакет **towelstuff** и которые, по вашему мнению, будут полезны для ваших пользователей. Если у вас их нет, удалите каталог `bin/`.
* На данный момент файл `CHANGES.txt` должен содержать только:

```bash
v<version>, <date> -- Initial release.
```

поскольку это ваша самая первая версия (номер версии будет описан ниже), и нет никаких изменений, о которых следует сообщить.

* В каталоге `docs/` должны содержаться все документы по дизайну, примечания по реализации, ответы на часто задаваемые вопросы или любые другие написанные вами документы. На данный момент придерживайтесь простых текстовых файлов с расширением «`.txt`». Этому автору (JohnMG) нравится использовать [Pandoc Markdown](https://pandoc.org/), но многие питонщики используют [reStructuredText](glossarii.md#restructuredtext).
* Файл `LICENSE.txt` часто является просто копией/вставкой выбранной вами лицензии. Мы рекомендуем использовать широко используемые лицензии, такие как GPL, BSD или MIT.
* Файл `MANIFEST.in` должен содержать следующее:

```bash
include *.txt
recursive-include docs *.txt
```

* Файл `README.txt` должен быть написан на [reST](glossarii.md#restructuredtext), чтобы PyPI мог использовать его для создания страницы PyPI вашего проекта. Вот 10-секундное вступление к **reST**, с которого вы можете начать:

```bash
===========
Towel Stuff
===========

Towel Stuff provides such and such and so and so. You might find
it most useful for tasks involving <x> and also <y>. Typical usage
often looks like this::

    #!/usr/bin/env python

    from towelstuff import location
    from towelstuff import utils

    if utils.has_towel():
        print "Your towel is located:", location.where_is_my_towel()

(Note the double-colon and 4-space indent formatting above.)

Paragraphs are separated by blank lines. *Italics*, **bold**,
and `monospace` look like this.


A Section
=========

Lists look like this:

* First

* Second. Can be multiple lines
  but must be indented properly.

A Sub-Section
-------------

Numbered lists look like you'd expect:

1. hi there

2. must be going

Urls are http://like.this and links can be
written `like this <http://www.example.com/foo/bar>`_.
```

Вы также можете подумать о добавлении разделов «Соавторы» и/или «Спасибо также», чтобы перечислить имена людей, которые помогли.

Кстати, чтобы увидеть, как выглядит приведенный выше `README.txt` в формате `html`, см. [Проект TowelStuff в PyPI](https://pypi.org/project/TowelStuff/).

* `setup.py` - создайте этот файл и сделайте его таким:

```python
from distutils.core import setup

setup(
    name='TowelStuff',
    version='0.1.0',
    author='J. Random Hacker',
    author_email='jrh@example.com',
    packages=['towelstuff', 'towelstuff.test'],
    scripts=['bin/stowe-towels.py','bin/wash-towels.py'],
    url='http://pypi.python.org/pypi/TowelStuff/',
    license='LICENSE.txt',
    description='Useful towel-related stuff.',
    long_description=open('README.txt').read(),
    install_requires=[
        "Django >= 1.1.1",
        "caldav == 0.1.4",
    ],
)
```

но, конечно, замените **towelstuff** своими именами проектов и пакетов. Дополнительные сведения о выборе номеров версий см. В разделе «Управление версиями», но «`0.1.0`» отлично подойдет для первого выпуска (используется стандартное соглашение о нумерации «`major.minor.micro`»).

Используйте аргумент _**install\_requires**_, чтобы автоматически устанавливать зависимости, когда ваш пакет будет установлен, и включать информацию о зависимостях (чтобы инструменты управления пакетами, такие как **pip**, могли использовать эту информацию). Требуется строка или список строк, содержащих спецификаторы требований.

Синтаксис состоит из имени PyPI проекта, за которым может следовать список спецификаторов версий, разделенных запятыми. Современные инструменты упаковки реализуют синтаксис спецификаторов версии, описанный в [PEP 345](https://www.python.org/dev/peps/pep-0345/#version-specifiers), и разрешают сравнение версий в соответствии с [PEP 386](https://www.python.org/dev/peps/pep-0386/).

Если у вас нет сценариев для распространения (и, следовательно, нет каталога `bin/`), вы можете удалить указанную выше строку, которая начинается со слова «_**scripts**_».

* Внутри каталога **towelstuff** `__init__.py` может быть пустым. Точно так же внутри `towelstuff/test` этот `__init__.py` также может быть пустым. Если у вас еще нет написанных тестов, вы можете оставить два других файла модуля в файле `towelstuff/test` пустыми. При написании тестов используйте стандартный модуль **unittest**.

В нашем примере **TowelStuff** не зависит от других дистрибутивов (это зависит только от того, что уже есть в стандартной библиотеке Python). Чтобы указать зависимости от других дистрибутивов, см. более подробный [пример Project](https://the-hitchhikers-guide-to-packaging.readthedocs.io/en/latest/example.html).

### Создание вашего дистрибутивного файла

Создайте свой дистрибутивный файл так:

```bash
$ cd path/to/TowelStuff
$ python setup.py sdist
```

Выполнение этой последней команды создаст файл **MANIFEST** в каталоге вашего проекта, а также каталог `dist/` и `build/`. Внутри этого каталога `dist/` находится дистрибутив, который вы будете загружать в PyPI. В нашем случае файл дистрибутива будет называться `TowelStuff-0.1.0.tgz`. Не стесняйтесь копаться в каталоге `dist/`, чтобы посмотреть свой дистрибутив.

### Загрузка вашего дистрибутивного файла

Перед загрузкой вам сначала необходимо создать учетную запись на [http://pypi.python.org/pypi](http://pypi.python.org/pypi). После этого зарегистрируйте свой дистрибутив в PyPI следующим образом:

```bash
$ cd path/to/TowelStuff
$ python setup.py register
```

Используйте существующий логин (вариант №1). Вам будет предложено сохранить данные для входа в систему для будущего использования (с чем я согласен). Затем загрузите дистрибутив:

```bash
$ python setup.py sdist upload
```

Это последний раз соберет дистрибутив, а затем загрузит его.

Спасибо за ваш вклад!

### Обновление вашего дистрибутива

В будущем, после того как вы обновите свой дистрибутив и захотите выпустить новый выпуск:

1. увеличьте номер версии в файле `setup.py`,
2. обновите файл `CHANGES.txt`,
3. при необходимости обновите разделы «**Авторы**» и «**Спасибо также**» вашего файла `README.txt`.
4. снова запустите `python setup.py sdist upload`.

### Точки входа

Точки входа - это функция **setuptools/distribute**, которая действительно удобна в одном конкретном случае: зарегистрируйте что-то под определенным ключом в пакете A, который может запросить пакет B.

Сам **distribute** использует это. Если вы правильно упаковываете свой проект, вы, вероятно, использовали точку входа _**console\_scripts**_:

```python
setup(name='zest.releaser',
      ...
      entry_points={
          'console_scripts':
              ['release = zest.releaser.release:main',
               'prerelease = zest.releaser.prerelease:main',
               ]}
      )
```

_**console\_scripts**_ - это точка входа, которую ищет **setuptools**. Он ищет все точки входа, зарегистрированные под именем _**console\_scripts**_, и использует эту информацию для создания скриптов. В приведенном выше примере это будет скрипт `bin/release`, который запускает метод `main ()` в `zest/releaser/release.py`.

Вы можете использовать это для своего собственного механизма расширения. Для [zest.releaser](https://pypi.org/project/zest.releaser/) мне понадобился какой-то механизм расширения. Я хотел иметь возможность делать дополнительные вещи во время **prerelease/release/postrelease**.

* Загрузка внешней библиотеки **javascript** в пакет, который не может храниться в репозитории **svn (zope)** напрямую из-за проблем с лицензированием. То есть, перед упаковкой и выпуском. Автоматически, чтобы вы не забыли об этом.
* Загрузка `version.cfg` в `scp://somewhere/kgs/ourmainproduct-version.cfg` после выпуска версии для использования в качестве так называемого «[известного хорошего набора](glossarii.md#known-good-set-kgs)» (KGS).
* Возможно изменение значений (например, сообщения о фиксации) внутри самого `zest.releaser` во время выпуска. (Время от времени я получаю запросы на изменение: «Эй, можешь ли ты сделать настраиваемыми x и y»). Итак, теперь каждый этап `zest.releaser` (предварительный выпуск, релиз, пострелиз) разделен на два: этап расчета и этап «выполнения». Результаты первого этапа сохраняются в словаре **dict**, который используется на втором этапе. И вы можете зарегистрировать точку входа, которой передается этот **dict**, чтобы вы могли ее изменить. Подробнее см. [Документацию по точке входа zest.releaser](https://zestreleaser.readthedocs.io/en/latest/).

Точка входа для `zest.releaser` настраивается в файле `setup.py` следующим образом:

```python
entry_points={
    'console_scripts':
        ['myscript = my.package.scripts:main'],
    'zest.releaser.prereleaser.middle':
        ['dosomething = my.package.some:some_entrypoint, ]
}
```

Замените _**prereleaser**_ и _**middle**_ в `zest.releaser.prereleaser.middle` на **prerelease/release/postrelease** и **before/middle/after** там, где это необходимо. (Для этого конкретного примера `zest.releaser`).

Теперь, как использовать это в своей программе? Лучший способ - показать быстрый пример из `zest.releaser`, где мы запрашиваем и используем одну из наших точек входа:

```python
import pkg_resources

...
def run_entry_point(data):
    # Примечание: данные специфичны для zest.releaser:
    # мы хотим передать что-то в группу плагинов = 'zest.releaser.prerelease.middle'

    for entrypoint in pkg_resources.iter_entry_points(group=group):
        # Возьмите функцию, которая является фактическим плагином.
        plugin = entrypoint.load()
        # Вызов плагина
        plugin(data)
```

Итак: довольно легкий и простой способ разрешить другим пакетам регистрировать то, что вы хотите знать. Дополнительные плагины, дополнительные методы рендеринга, дополнительные функции, которые вы хотите зарегистрировать в своем веб-приложении, и т. д.

## Упаковка для конкретной операционной системы (ОС)

### Общие рекомендации по упаковке для Unix

### Общие рекомендации по упаковке для Windows
