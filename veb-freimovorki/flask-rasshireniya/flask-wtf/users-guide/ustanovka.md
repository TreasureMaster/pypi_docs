# Установка

В этой части документации описана установка **Flask-WTF**. Первым шагом к использованию любого программного пакета является его правильная установка.

### Распространение и pip

Установить **Flask-WTF** просто с помощью [pip](https://pip.pypa.io/en/stable/):

```bash
$ pip install Flask-WTF
```

или с помощью [easy\_install](https://pypi.org/project/setuptools/):

```bash
$ easy_install Flask-WTF
```

Но вам действительно [не следует этого делать](https://python-packaging-user-guide.readthedocs.io/en/latest/pip\_easy\_install/). (устарело ?)

### Получить код

**Flask-WTF** активно развивается на GitHub, где код [всегда доступен](https://github.com/lepture/flask-wtf).

Вы можете либо клонировать публичный репозиторий:

```bash
git clone git://github.com/lepture/flask-wtf.git
```

Загрузите [tar-архив](https://github.com/lepture/flask-wtf/tarball/master):

```bash
$ curl -OL https://github.com/lepture/flask-wtf/tarball/master
```

Или загрузите [zip-архив](https://github.com/lepture/flask-wtf/zipball/master):

```bash
$ curl -OL https://github.com/lepture/flask-wtf/zipball/master
```

Когда у вас есть копия исходного кода, вы можете встроить ее в свой пакет Python или легко установить в свои пакеты сайта:

```bash
$ python setup.py install
```
