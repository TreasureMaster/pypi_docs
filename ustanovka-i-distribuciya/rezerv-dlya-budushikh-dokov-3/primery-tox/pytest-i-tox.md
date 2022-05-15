# pytest и tox

Выполнение [pytest](https://docs.pytest.org/en/stable/) легко интегрировать с **tox**. Если вы столкнулись с проблемами, проверьте, указаны ли они как [известные проблемы](pytest-i-tox.md#izvestnye-problemy-i-ogranicheniya), и/или воспользуйтесь [каналами поддержки](../podderzhka-tox.md).

## Базовый пример

Предполагая следующий макет:

```bash
tox.ini      # см. содержание ниже
setup.py     # классический файл distutils/setuptools - setup.py
```

и следующее содержимое `tox.ini`:

```bash
[tox]
envlist = py35,py36

[testenv]
deps = pytest               # Пакет PYPI, предоставляющий pytest
commands = pytest {posargs} # заменить позиционными аргументами tox
```

Теперь вы можете вызвать **tox** в каталоге, в котором находится ваш `tox.ini`. **tox** упакует ваш проект в **sdist**, создаст две среды **virtualenv** с интерпретаторами **python3.5** и **python3.6** соответственно, а затем запустит указанную тестовую команду в каждой из них.

## Расширенный пример: измените каталог перед тестированием и используйте per-virtualenv tempdir

Предполагая следующий макет:

```bash
tox.ini      # см. содержание ниже
setup.py     # классический файл distutils/setuptools - setup.py
tests        # каталог, содержащий тесты
```

и следующее содержимое `tox.ini`:

```bash
[tox]
envlist = py35,py36

[testenv]
changedir = tests
deps = pytest
# измените pytest tempdir и добавьте posargs из командной строки
commands = pytest --basetemp="{envtmpdir}" {posargs}
```

вы можете вызвать **tox** в каталоге, в котором находится ваш `tox.ini`. В отличие от предыдущего примера, команда **pytest** будет выполняться с текущим рабочим каталогом, установленным на **tests**, а тестовый запуск будет использовать временный каталог **per-virtualenv**.

## Использование нескольких процессоров для тестовых прогонов

**pytest** поддерживает распространение тестов по нескольким процессам и хостам через плагин **pytest-xdist**. Вот пример конфигурации, позволяющей **tox** использовать эту функцию:

```bash
[testenv]
deps = pytest-xdist
changedir = tests
# использовать три подпроцесса
commands = pytest --basetemp="{envtmpdir}"  \
                  --confcutdir=..         \
                  -n 3                    \
                  {posargs}
```

## Известные проблемы и ограничения

**Слишком длинные имена файлов**. Вы можете столкнуться со «слишком длинными именами файлов» для временно созданных файлов при запуске **pytest**. Старайтесь не использовать параметр «`–basetemp`».

Версия **installed-versus-checkout**. **pytest** собирает тестовые модули в файловой системе, а затем пытается импортировать их под их [полностью подходящим именем](https://docs.pytest.org/en/latest/goodpractices.html#test-package-name). Это означает, что если ваши тестовые файлы можно импортировать откуда-то, тогда ваш вызов **pytest** может закончить импортированием пакета из каталога проверки, а не из установленного пакета.

Эта проблема может характеризоваться сообщениями об ошибках **pytest** _**test-collection**_ в средах python 3.x, которые выглядят следующим образом:

```bash
import file mismatch:
imported module 'myproj.foo.tests.test_foo' has this __file__ attribute:
  /home/myuser/repos/myproj/build/lib/myproj/foo/tests/test_foo.py
which is not the same as the test file we want to collect:
  /home/myuser/repos/myproj/myproj/foo/tests/test_foo.py
HINT: remove __pycache__ / .pyc files and/or use a unique basename for your test file modules
```

Есть несколько способов предотвратить это.

С установленными тестами (пакеты тестов известны как `setup.py`) безопасным и явным вариантом является указание явного пути `{envsitepackagesdir}/mypkg` к **pytest**. В качестве альтернативы можно использовать **changedir**, чтобы извлеченные файлы находились за пределами пути импорта, а затем передать `--pyargs mypkg` в **pytest**.

Для тестов, которые не будут установлены, самый простой способ запустить их для установленного пакета - избегать файлов `__init__.py` в тестовых каталогах; **pytest** по-прежнему найдет и импортирует их, добавив их родительский каталог в `sys.path`, но они не будут скопированы в другие места или найдены системой импорта Python за пределами **pytest**.
