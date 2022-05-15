# Поддержка системы сборки

## Что это?

Упаковка Python прошла [долгий путь](https://www.bernat.tech/pep-517-518/).

Традиционный способ упаковки модулей Python с помощью **setuptools** использует функцию `setup ()` в скрипте `setup.py`. Такие команды, как `python setup.py bdist` или `python setup.py bdist_wheel`, создают пакет распространения, а `python setup.py install` устанавливает дистрибутив. Этот интерфейс затрудняет выбор других инструментов упаковки без капитального ремонта. Поскольку сценарии `setup.py` допускают произвольное выполнение, оказалось сложно обеспечить надежное взаимодействие с пользователем в разных средах и истории.

Поэтому на помощь пришел [PEP 517](https://www.python.org/dev/peps/pep-0517/), который определил новый стандарт для упаковки и распространения модулей Python. Под PEP 517:

* файл `pyproject.toml` используется для указания того, какую программу использовать для создания дистрибутива.
* Затем две функции, предоставляемые программой, `build_wheel (directory: str)` и `build_sdist (directory: str)`, создают пакет распространения в указанном каталоге. Программа может использовать собственный сценарий конфигурации или расширять файл `.toml`.
* Наконец, `pip install *.whl` или `pip install *.tar.gz` выполняет фактическую установку. Если доступен `*.whl`, **pip** продолжит копирование файлов в каталог `site-packages`. Если нет, **pip** просмотрит `pyproject.toml` и решит, какую программу использовать для «сборки из исходного кода» (по умолчанию это **setuptools**).

Благодаря этому стандарту переключение между упаковочными инструментами становится намного проще. **build\_meta** реализует поддержку системы сборки **setuptools**.

## Как это использовать?

Начиная с пакета, который вы хотите распространить. Вам потребуются исходные скрипты, файл `pyproject.toml` и файл `setup.cfg`:

```bash
~/meowpkg/
    pyproject.toml
    setup.cfg
    meowpkg/__init__.py
```

Файл `pyproject.toml` требуется для указания системы сборки (т. е. того, что используется для упаковки ваших скриптов и установки из исходного кода). Чтобы использовать его с инструментами настройки, содержимое будет следующим:

```bash
[build-system]
requires = ["setuptools", "wheel"]
build-backend = "setuptools.build_meta"
```

Используйте [декларативную конфигурацию](rukovodstvo-polzovatelya-setuptools/nastroika-setup-s-pomoshyu-failov-setup.cfg.md#nastroika-setup-s-pomoshyu-failov-setup-cfg) **setuptools**, чтобы указать информацию о пакете:

```bash
[metadata]
name = meowpkg
version = 0.0.1
description = a package that meows

[options]
packages = find:
```

Теперь сгенерируйте дистрибутив. Для сборки пакета используйте модуль [PyPA build](https://pypa-build.readthedocs.io/en/latest/):

```bash
$ pip install -q build
$ python -m build
```

И вот готово! Затем можно распространить и установить файл `.whl` и `.tar.gz`:

```bash
dist/
    meowpkg-0.0.1.whl
    meowpkg-0.0.1.tar.gz

$ pip install dist/meowpkg-0.0.1.whl
```

или:

```bash
$ pip install dist/meowpkg-0.0.1.tar.gz
```
