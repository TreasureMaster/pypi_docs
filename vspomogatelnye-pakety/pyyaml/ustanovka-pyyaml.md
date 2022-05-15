# Установка PyYAML

Простая установка:

```bash
pip install pyyaml
```

Для установки из исходников загрузите исходный пакет PyYAML-5.1.tar.gz и распакуйте его. Переходим в каталог PyYAML-5.1 и запускаем:

```bash
$ python setup.py install
```

Если вы хотите использовать привязки LibYAML, которые намного быстрее, чем чистая версия Python, вам необходимо загрузить и установить [LibYAML](https://pyyaml.org/wiki/LibYAML). Затем вы можете построить и установить привязки, выполнив

```bash
$ python setup.py --with-libyaml install
```

Чтобы использовать синтаксический анализатор и эмиттер на основе LibYAML, используйте классы **CParser** и **CEmitter**. Например,

```python
from yaml import load, dump
try:
    from yaml import CLoader as Loader, CDumper as Dumper
except ImportError:
    from yaml import Loader, Dumper

# ...

data = load(stream, Loader=Loader)

# ...

output = dump(data, Dumper=Dumper)
```

Обратите внимание, что есть некоторые тонкие (но не очень существенные) различия между синтаксическими анализаторами и эмиттерами на основе чистого Python и LibYAML.
