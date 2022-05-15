# Теги YAML и типы Python

В следующей таблице описывается, как узлы с разными тегами преобразуются в объекты Python.

| Тег YAML                       | Тип Python                        |
| ------------------------------ | --------------------------------- |
| _Стандартные YAML теги_        |                                   |
| !!null                         | None                              |
| !!bool                         | bool                              |
| !!int                          | int или long (int в Python 3)     |
| !!float                        | float                             |
| !!binary                       | str (bytes в Python 3)            |
| !!timestamp                    | datetime.datetime                 |
| !!omap, !!pairs                | list или pairs                    |
| !!set                          | set                               |
| !!str                          | str или unicode (str в Python 3)  |
| !!seq                          | list                              |
| !!map                          | dict                              |
| _Специфичные теги Python_      |                                   |
| !!python/none                  | None                              |
| !!python/bool                  | bool                              |
| !!python/bytes                 | (bytes в Python 3)                |
| !!python/str                   | str (str в Python 3)              |
| !!python/unicode               | unicode (str в Python 3)          |
| !!python/int                   | int                               |
| !!python/long                  | long (int в Python 3)             |
| !!python/float                 | float                             |
| !!python/complex               | complex                           |
| !!python/list                  | list                              |
| !!python/tuple                 | tuple                             |
| !!python/dict                  | dict                              |
| _Комплексные теги Python_      |                                   |
| !!python/name:module.name      | module.name                       |
| !!python/module:package.module | package.name                      |
| !!python/object:module.cls     | экземпляр module.cls              |
| !!python/object/new:module.cls | экземпляр module.cls              |
| !!python/object/apply:module.f | значение f(...)                   |

### Преобразование строк (только Python 2)

Есть четыре тега, которые преобразуются в значения **str** и **unicode**: <mark style="color:red;">!!str</mark>, <mark style="color:red;">!!binary</mark>, <mark style="color:red;">!!python/str</mark> и <mark style="color:red;">!!python/unicode</mark>.

Скаляры с тегами <mark style="color:red;">!!str</mark> преобразуются в объекты **str**, если их значение - **ASCII**. В противном случае он преобразуется в **unicode**. Cкаляры с двоичными тегами <mark style="color:red;">!!binary</mark> преобразуются в объекты **str**, значение которых декодируется с использованием кодировки **base64**. Скаляры <mark style="color:red;">!!python/str</mark> преобразуются в объекты **str**, закодированные в кодировке **utf-8**. Скаляры <mark style="color:red;">!!python/unicode</mark> преобразуются в объекты **unicode**.

И наоборот, объект **str** преобразуется в

1. скаляр <mark style="color:red;">!!str</mark>, если его значение - **ASCII**.
2. скаляр <mark style="color:red;">!!python/str</mark>, если его значение является правильной последовательностью **utf-8**.
3. скаляр <mark style="color:red;">!!binary</mark> в противном случае.

Объект Unicode преобразуется в

1. скаляр <mark style="color:red;">!!python/unicode</mark>, если его значение равно **ASCII**.
2. <mark style="color:red;">!!str</mark> скаляр в противном случае.

### Преобразование строк (только Python 3)

В Python 3 объекты **str** преобразуются в скаляры <mark style="color:red;">!!str</mark>, а объекты **bytes** - в скаляры <mark style="color:red;">!!binary</mark>. По соображениям совместимости теги <mark style="color:red;">!!python/str</mark> и <mark style="color:red;">!!python/unicode</mark> по-прежнему поддерживаются и преобразуются в объекты **str**.

### Имена и модули

Чтобы представлять статические объекты Python, такие как функции или классы, вам необходимо использовать сложный тег <mark style="color:red;">!!python/name</mark>. Например, функцию **yaml.dump** можно представить как

```yaml
!!python/name:yaml.dump
```

Точно так же модули представлены с помощью тега <mark style="color:red;">!!python/module</mark>:

```yaml
!!python/module:yaml
```

### Объекты

Любой обрабатываемый объект можно сериализовать с помощью тега <mark style="color:red;">!!python/object</mark>:

```yaml
!!python/object:module.Class { attribute: value, ... }
```

Для поддержки протокола **pickle** предусмотрены две дополнительные формы тега <mark style="color:red;">!!python/object</mark>:

```yaml
!!python/object/new:module.Class
args: [argument, ...]
kwds: {key: value, ...}
state: ...
listitems: [item, ...]
dictitems: [key: value, ...]

!!python/object/apply:module.function
args: [argument, ...]
kwds: {key: value, ...}
state: ...
listitems: [item, ...]
dictitems: [key: value, ...]
```

Если только поле **args** не пусто, указанные выше записи можно сократить:

```yaml
!!python/object/new:module.Class [argument, ...]

!!python/object/apply:module.function [argument, ...]
```
