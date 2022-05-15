# Установка tox

## Краткая информация об установке

* &#x20;**Pythons**: CPython 2.7 и 3.5 или новее, Jython-2.5.1, pypy-1.9ff
* **Операционные системы**: Linux, Windows, OSX, Unix
* **Требования для установщика**: [setuptools](https://pypi.org/project/setuptools/)
* **Лицензия**: лицензия MIT
* **Репозиторий git**: [https://github.com/tox-dev/tox](https://github.com/tox-dev/tox)

## Установка с помощью pip

Используйте следующую команду:

```bash
pip install tox
```

Саму **tox** можно установить в среду [virtualenv](https://pypi.org/project/virtualenv/).

## Установить из клона

Проконсультируйтесь со страницей GitHub, как клонировать репозиторий git: [https://github.com/tox-dev/tox](https://github.com/tox-dev/tox)

а затем установите в своей среде что-то вроде:

```bash
$ cd <path/to/clone>
$ pip install .
```

или установите его с [возможностью редактирования](https://pip.pypa.io/en/stable/reference/pip\_install/#editable-installs), если вы хотите, чтобы изменения кода распространялись автоматически:

```bash
$ cd <path/to/clone>
$ pip install --editable .
```

чтобы вы могли вносить изменения и отправлять исправления.

## \[Linux / macOS] Установка через диспетчер пакетов

Вы также можете найти пакет **tox** для многих дистрибутивов Linux и Homebrew для macOs - обычно под названием **python-tox** или просто **tox**. Однако имейте в виду, что есть и другие проекты под тем же именем (наиболее заметно [клиент безопасного чата](https://tox.chat/), не связанный с этим проектом), поэтому убедитесь, что вы установили правильный пакет.
