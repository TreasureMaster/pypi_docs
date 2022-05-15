# unittest2, discover и tox

## Запуск модульных тестов с помощью «discover»

Проект [discover](https://pypi.org/project/discover/) позволяет вам обнаруживать и запускать модульные тесты, которые вы можете легко интегрировать в прогон **tox**. В качестве примера выполните проверку [Pygments](https://pypi.org/project/Pygments/):

```bash
hg clone https://bitbucket.org/birkenfeld/pygments-main
```

и добавьте к нему следующий `tox.ini`:

```bash
[tox]
envlist = py27,py35,py36

[testenv]
changedir = tests
commands = discover
deps = discover
```

Если вы сейчас вызовете **tox**, вы увидите создание трех виртуальных сред и выполнение **unittest-run** в каждой из них.

## Запуск тестов unittest2 и sphinx за один раз

[Майкл Форд](http://www.voidspace.org.uk/) предоставил файл `tox.ini`, который позволяет вам запускать все тесты для его фиктивного [mock](https://pypi.org/project/mock/) проекта, включая некоторые **doctests** на основе **sphinx**. Если вы проверите этот репозиторий с помощью:

```bash
git clone https://github.com/testing-cabal/mock.git
```

То увидите, что есть файл `tox.ini`, который выглядит так:

```bash
[tox]
envlist = py27,py35,py36,py37

[testenv]
deps = unittest2
commands = unit2 discover []

[testenv:py36]
commands =
    unit2 discover []
    sphinx-build -b doctest docs html
    sphinx-build docs html
deps =
    unittest2
    sphinx

[testenv:py27]
commands =
    unit2 discover []
    sphinx-build -b doctest docs html
    sphinx-build docs html
deps =
    unittest2
    sphinx
```

mock использует [unittest2](https://pypi.org/project/unittest2/) для запуска тестов. Вызов **tox** запускает обнаружение тестов путем выполнения команды `unit2 discover` на `Python 2.7`, `3.5`, `3.6` и `3.7` соответственно. Против `Python3.6` и `Python2.7` он дополнительно будет запускать **doctests**, опосредованные **sphinx**. Если сборка документов не удалась из-за ошибки **reST** или какой-либо из тестов не прошел, об этом будет сообщено с помощью прогона **tox**.

Скобки `[]` в командах обеспечивают [замену интерактивной оболочки](../specifikaciya-konfiguracii-tox.md#zamesheniya-interaktivnoi-obolochki), что означает, что вы можете, например, тип:

```bash
tox -- -f -s SOMEPATH
```

который в конечном итоге вызовет:

```bash
unit2 discover -f -s SOMEPATH
```

в каждой из сред. Это позволяет вам настроить обнаружение тестов в ваших прогонах **tox**.
