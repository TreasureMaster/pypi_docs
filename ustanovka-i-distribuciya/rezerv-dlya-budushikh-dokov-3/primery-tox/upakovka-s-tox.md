# Упаковка с tox

Хотя можно использовать **tox** для разработки и тестирования приложений, одним из наиболее популярных способов его использования является помощь создателям библиотек. Библиотеки необходимо сначала упаковать, чтобы затем их можно было установить в виртуальной среде для тестирования. Чтобы помочь с этой интеграцией, применяются [PEP-517](https://www.python.org/dev/peps/pep-0517/) и [PEP-518](https://www.python.org/dev/peps/pep-0518/). Это означает, что по умолчанию **tox** будет строить исходный код из исходных деревьев. Перед запуском тестовых команд **pip** используется для установки исходного кода внутри среды сборки.

Для создания исходного дистрибутива существует множество инструментов, а с **PEP-517** и **PEP-518** вы можете легко использовать свой любимый с **tox**. Исторически **tox** поддерживал только **setuptools** и всегда использовал среду хоста **tox** для создания исходного дистрибутива из исходного дерева. Это по-прежнему поведение по умолчанию. Чтобы отказаться от этого поведения, вам необходимо установить для изолированных сборок значение `true`.

## setuptools

Используя файл `pyproject.toml` в корневой папке (вместе с `setup.py`), можно указать требования к сборке.

```bash
# pyproject.toml
[build-system]
requires = [
    "setuptools >= 35.0.2",
    "setuptools_scm >= 2.0.0, <3"
]
build-backend = "setuptools.build_meta"
```

```bash
# tox.ini
[tox]
isolated_build = True
```

## flit

[flit](https://flit.readthedocs.io/en/latest/) требует **Python 3**, однако сгенерированный исходный код можно установить под **python 2**. Кроме того, для него не требуется файл `setup.py`, поскольку эта информация также добавляется в файл `pyproject.toml`.

```bash
# pyproject.toml
[build-system]
requires = ["flit_core >=2,<4"]
build-backend = "flit_core.buildapi"

[tool.flit.metadata]
module = "package_toml_flit"
author = "Happy Harry"
author-email = "happy@harry.com"
home-page = "https://github.com/happy-harry/is"
```

```bash
# tox.ini
[tox]
isolated_build = True
```

## poetry

[poetry](https://python-poetry.org/) требует **Python 3**, однако сгенерированный исходный код можно установить под **python 2**. Кроме того, он не требует файла `setup.py`, поскольку эта информация также добавляется в файл `pyproject.toml`.

```bash
# pyproject.toml
[build-system]
requires = ["poetry_core>=1.0.0"]
build-backend = "poetry.core.masonry.api"

[tool.poetry]
name = "package_toml_poetry"
version = "0.1.0"
description = ""
authors = ["Name <email@email.com>"]
```

```bash
# tox.ini
[tox]
isolated_build = True

[tox:.package]
# обратите внимание, что tox будет использовать ту же версию python,
# что и тот, в котором tox установлен для упаковки, поэтому,
# если это не python 3, вы можете потребовать данную версию python
#  для среды упаковки через ключ basepython
basepython = python3
```
