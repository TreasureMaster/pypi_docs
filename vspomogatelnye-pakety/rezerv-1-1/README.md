# decorator

## Декораторы для людей

Цель модуля **decorator** - упростить определение декораторов функций и фабрик декораторов, сохраняющих сигнатуру. Он также включает в себя реализацию множественной отправки и другие тонкости (пожалуйста, проверьте документацию). Он выпущен под лицензией BSD, состоящей из двух пунктов, то есть в основном вы можете делать с ним все, что хотите, но я не несу ответственности.

### Установка

Если вы ленивы, просто выполняйте код

```bash
$ pip install decorator
```

который установит в вашу систему только модуль.

Если вы предпочитаете установить полный дистрибутив из исходного кода, включая документацию, клонируйте [репозиторий GitHub](https://github.com/micheles/decorator) или загрузите [tarball](https://pypi.org/project/decorator/#files), распакуйте его и запустите

```bash
$ pip install .
```

в основном каталоге, возможно, как суперпользователь.

### Тестирование

Если у вас установлен исходный код, вы можете запускать тесты с помощью

```bash
$ python src/tests/test.py -v
```

или (если у вас установлен **setuptools**)

```bash
$ python setup.py test
```

Обратите внимание, что у вас могут возникнуть проблемы, если в вашей системе есть более старая версия модуля **decorator**; в таком случае удалите старую версию. Безопасно даже копировать модуль `decorator.py` поверх существующего, так как мы долгое время сохраняли обратную совместимость.

### Репозитарий

Проект размещен на GitHub. Вы можете посмотреть исходник здесь:

[https://github.com/micheles/decorator](https://github.com/micheles/decorator)

### Документация

Документация перенесена в [https://github.com/micheles/decorator/blob/master/docs/documentation.md](https://github.com/micheles/decorator/blob/master/docs/documentation.md)

Оттуда вы можете получить PDF-версию, просто используя функцию печати вашего браузера.

Вот документация к предыдущим версиям модуля:

* [https://github.com/micheles/decorator/blob/4.3.2/docs/tests.documentation.rst](https://github.com/micheles/decorator/blob/4.3.2/docs/tests.documentation.rst)
* [https://github.com/micheles/decorator/blob/4.2.1/docs/tests.documentation.rst](https://github.com/micheles/decorator/blob/4.2.1/docs/tests.documentation.rst)
* [https://github.com/micheles/decorator/blob/4.1.2/docs/tests.documentation.rst](https://github.com/micheles/decorator/blob/4.1.2/docs/tests.documentation.rst)
* [https://github.com/micheles/decorator/blob/4.0.0/documentation.rst](https://github.com/micheles/decorator/blob/4.0.0/documentation.rst)
* [https://github.com/micheles/decorator/blob/3.4.2/documentation.rst](https://github.com/micheles/decorator/blob/3.4.2/documentation.rst)

### Для нетерпеливых

Вот пример того, как определить семейство декораторов, отслеживающих медленные операции:

```python
from decorator import decorator

@decorator
def warn_slow(func, timelimit=60, *args, **kw):
    t0 = time.time()
    result = func(*args, **kw)
    dt = time.time() - t0
    if dt > timelimit:
        logging.warn('%s took %d seconds', func.__name__, dt)
    else:
        logging.info('%s took %d seconds', func.__name__, dt)
    return result

@warn_slow  # предупредить, если это займет больше 1 минуты
def preprocess_input_files(inputdir, tempdir):
    ...

@warn_slow(timelimit=600)  # предупредить, если это займет больше 10 минут
def run_calculation(tempdir, outdir):
    ...
```

Наслаждайся!
