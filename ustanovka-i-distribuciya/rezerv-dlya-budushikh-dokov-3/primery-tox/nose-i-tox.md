# nose и tox

[nosetests](https://pypi.org/project/nose/) легко интегрировать с **tox**. Для начала вот простая конфигурация `tox.ini` для настройки вашего проекта для работы с **nose**:

## Пример базового nosetests

Предполагая следующий макет:

```bash
tox.ini      # см. содержание ниже
setup.py     # классический файл distutils/setuptools - setup.py
```

и следующее содержимое `tox.ini`:

```bash
[testenv]
deps = nose
# ``{posargs}`` будет заменен позиционными аргументами из командной строки
commands = nosetests {posargs}
```

вы можете вызвать **tox** в каталоге, в котором находится ваш `tox.ini`. **tox** будет **sdist-package**, чтобы ваш проект создал две среды **virtualenv** с интерпретаторами `python2.7` и `python3.6` соответственно, а затем запустит указанную тестовую команду.

## Еще примеры?

Также вы можете проверить [Общие советы и рекомендации](obshie-sovety-po-tox.md) и [Создать документацию](sozdanie-dokumentacii-s-tox.md).
