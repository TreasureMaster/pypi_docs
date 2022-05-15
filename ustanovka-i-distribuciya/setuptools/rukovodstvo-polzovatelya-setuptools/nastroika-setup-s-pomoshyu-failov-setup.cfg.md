# Настройка setup () с помощью файлов setup.cfg

## Настройка setup () с помощью файлов setup.cfg

{% hint style="info" %}
**ПРИМЕЧАНИЕ:**

Новое в версии 30.3.0 (8 декабря 2016 г.).
{% endhint %}

{% hint style="warning" %}
**ВАЖНО:**

Если требуется совместимость с устаревшими сборками (то есть с теми, которые не используют API сборки [PEP 517](https://www.python.org/dev/peps/pep-0517/)), файл **setup.py**, содержащий вызов функции **setup ()**, по-прежнему требуется, даже если ваша конфигурация находится в **setup.cfg**.
{% endhint %}

**Setuptools** позволяет использовать файлы конфигурации (обычно `setup.cfg`) для определения метаданных пакета и других параметров, которые обычно предоставляются функции **setup ()** (декларативная конфигурация).

Такой подход не только позволяет сценарии автоматизации, но и в некоторых случаях сокращает шаблонный код.

{% hint style="info" %}
**ЗАМЕЧАНИЕ:**

Эта реализация имеет ограниченную совместимость с разделами **setup.cfg**, подобными **distutils2**, которые используются пакетами **pbr** и **d2to1**.

А именно: поддерживаются только ключи, относящиеся к метаданным из раздела метаданных (кроме файла-описания); ключи из файлов, **entry\_points** и **backwards\_compat** не поддерживаются.
{% endhint %}

```bash
[metadata]
name = my_package
version = attr: src.VERSION
description = My package description
long_description = file: README.rst, CHANGELOG.rst, LICENSE.rst
keywords = one, two
license = BSD 3-Clause License
classifiers =
    Framework :: Django
    License :: OSI Approved :: BSD License
    Programming Language :: Python :: 3
    Programming Language :: Python :: 3.5

[options]
zip_safe = False
include_package_data = True
packages = find:
scripts =
    bin/first.py
    bin/second.py
install_requires =
    requests
    importlib; python_version == "2.6"

[options.package_data]
* = *.txt, *.rst
hello = *.msg

[options.extras_require]
pdf = ReportLab>=1.2; RXP
rest = docutils>=0.3; pack ==1.1, ==1.3

[options.packages.find]
exclude =
    src.subpackage1
    src.subpackage2

[options.data_files]
/etc/my_package =
    site.d/00_default.conf
    host.d/00_default.conf
data = data/img/logo.png, data/svg/icon.svg
```

Метаданные и параметры устанавливаются в одноименных разделах конфигурации.

* Ключи такие же, как аргументы ключевого слова, которые передаются функции **setup ()**.
* Сложные значения могут быть записаны через запятую или помещены по одному в строке в зависших значениях конфигурации. Следующие варианты эквивалентны:

```bash
[metadata]
keywords = one, two

[metadata]
keywords =
    one
    two
```

* В некоторых случаях сложные значения могут быть представлены в специальных подразделах для ясности.
* Некоторые ключи позволяют использовать директивы `file:`, `attr:`, `find:` и `find_namespace:` для охвата распространенных сценариев использования.
* Неизвестные ключи игнорируются.

### Использование макета src/

Одна обычно используемая конфигурация пакета имеет весь исходный код модуля в подкаталоге (часто называемом макетом `src/` ), например:

```bash
├── src
│   └── mypackage
│       ├── __init__.py
│       └── mod1.py
├── setup.py
└── setup.cfg
```

Вы можете настроить файл `setup.cfg` для автоматического поиска всех ваших пакетов в подкаталоге следующим образом:

```bash
# Этот пример содержит только необходимые параметры для src-layout,
# настройте остальную часть файла, как описано выше.

[options]
package_dir=
    =src
packages=find:

[options.packages.find]
where=src
```

### Указание значений

Некоторые значения обрабатываются как простые строки, некоторые допускают больше логики.

Используемые имена типов ниже:

* **str** - простая строка
* **list-comma** - висячий список или строка значений, разделенных запятыми
* **list-semi** - висячий список или строка значений, разделенных точкой с запятой
* **bool** - `True` - это `1`, yes, true
* **dict** - список-запятая, где ключи отделены от значений знаком `=`
* **section** - значения считываются из специального (под) раздела

Специальные директивы:

* **attr:** - Значение считывается из атрибута модуля. `attr:` поддерживает вызываемые и итерируемые объекты; неподдерживаемые типы приводятся с помощью `str ()`. Чтобы поддержать общий случай буквального значения, присвоенного переменной в модуле, содержащем (прямо или косвенно) сторонний импорт, `attr:` сначала пытается прочитать значение из модуля, исследуя AST модуля. Если это не удается, `attr:` возвращается к импорту модуля.
* **file:** - Значение считывается из списка файлов и затем объединяется

{% hint style="info" %}
**ЗАМЕЧАНИЕ:**

Директива **file:** изолирована в песочнице и не будет достигать ничего за пределами каталога, содержащего **setup.py**.
{% endhint %}

### Метаданные

{% hint style="info" %}
**ПРИМЕЧАНИЕ:**

Приведенные ниже псевдонимы поддерживаются из соображений совместимости, но их использование не рекомендуется.
{% endhint %}

| Ключ                             | Псевдоним        | Тип               | Минимальная версия | Сноски |
| -------------------------------- | ---------------- | ----------------- | ------------------ | ------ |
| name                             |                  | str               |                    |        |
| version                          |                  | attr:, file:, str | 39.2.0             | 1.     |
| url                              | home-page        | str               |                    |        |
| download\_url                    | download-url     | str               |                    |        |
| project\_urls                    |                  | dict              | 38.3.0             |        |
| author                           |                  | str               |                    |        |
| author\_email                    | author-email     | str               |                    |        |
| maintainer                       |                  | str               |                    |        |
| maintainer\_email                | maintainer-email | str               |                    |        |
| classifiers                      | classifier       | file:, list-comma |                    |        |
| license                          |                  | str               |                    |        |
| license\_file                    |                  | str               |                    |        |
| license\_files                   |                  | list-comma        |                    |        |
| description                      | summary          | file:, str        |                    |        |
| long\_description                | long-description | file:, str        |                    |        |
| long\_description\_content\_type |                  | str               | 38.6.0             |        |
| keywords                         |                  | list-comma        |                    |        |
| platforms                        | platform         | list-comma        |                    |        |
| provides                         |                  | list-comma        |                    |        |
| requires                         |                  | list-comma        |                    |        |
| obsoletes                        |                  | list-comma        |                    |        |

{% hint style="info" %}
**ЗАМЕЧАНИЕ:**

Версия, загруженная с помощью директивы **file:**, должна соответствовать **PEP 440**. В такой файл легко случайно поместить что-то, кроме допустимой строки версии, поэтому в этом случае проверка будет более строгой.
{% endhint %}

Сноски:

1. Атрибут файла **version** поддерживается только с 39.2.0.

### Параметры

| Ключ                       | Тип                                 | Минимальная версия | Сноски |
| -------------------------- | ----------------------------------- | ------------------ | ------ |
| zip\_safe                  | bool                                |                    |        |
| setup\_requires            | list-semi                           |                    |        |
| install\_requires          | list-semi                           |                    |        |
| extras\_require            | section                             |                    |        |
| python\_requires           | str                                 |                    |        |
| entry\_points              | file:, section                      |                    |        |
| use\_2to3                  | bool                                |                    |        |
| use\_2to3\_fixers          | list-comma                          |                    |        |
| use\_2to3\_exclude\_fixers | list-comma                          |                    |        |
| convert\_2to3\_doctests    | list-comma                          |                    |        |
| scripts                    | list-comma                          |                    |        |
| eager\_resources           | list-comma                          |                    |        |
| dependency\_links          | list-comma                          |                    |        |
| tests\_require             | list-semi                           |                    |        |
| include\_package\_data     | bool                                |                    |        |
| packages                   | find:, find\_namespace:, list-comma |                    |        |
| package\_dir               | dict                                |                    |        |
| package\_data              | list-comma                          |                    | 1.     |
| exclude\_package\_data     | list-comma                          |                    |        |
| namespace\_packages        | section                             |                    |        |
| py\_modules                | section                             |                    |        |
| data\_files                | dict                                | 40.6.0             |        |

{% hint style="info" %}
ПРИМЕЧАНИЕ:

**packages** - директива **find:** и **find\_namespace:** может быть дополнительно настроена в специальном подразделе **options.packages.find**. Этот подраздел принимает те же ключи, что и **setuptools.find\_packages** и функция **setuptools.find\_namespace\_packages:** _**where**_, _**include**_ и _**exclude**_.

**find\_namespace** - директива **find\_namespace:** поддерживается начиная с **Python>=3.3**.
{% endhint %}

Сноски:

1. В секции **package\_data** ключ, названный одной звездочкой (`*`), относится ко всем пакетам вместо пустой строки, используемой в `setup.py`.
