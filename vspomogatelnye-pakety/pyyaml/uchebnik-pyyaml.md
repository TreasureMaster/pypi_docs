# Учебник PyYAML

Начните с импорта пакета **yaml**.

```python
>>> import yaml
```

## Загрузка YAML

{% hint style="warning" %}
Вызвать **yaml.load** с любыми данными, полученными из ненадежного источника, небезопасно! **yaml.load** так же мощен, как **pickle.load**, и поэтому может вызывать любую функцию Python. Однако проверьте функцию **yaml.safe\_load**.
{% endhint %}

Функция **yaml.load** преобразует документ YAML в объект Python.

```python
>>> yaml.load("""
... - Hesperiidae
... - Papilionidae
... - Apatelodidae
... - Epiplemidae
... """)

['Hesperiidae', 'Papilionidae', 'Apatelodidae', 'Epiplemidae']
```

**yaml.load** принимает байтовую строку, строку Unicode, объект открытого двоичного файла или объект открытого текстового файла. Строка байтов или файл должны быть закодированы в кодировке <mark style="color:red;">**utf-8**</mark>, <mark style="color:red;">**utf-16-be**</mark> или <mark style="color:red;">**utf-16-le**</mark>. **yaml.load** определяет кодировку, проверяя последовательность BOM (метка порядка байтов) в начале строки/файла. Если спецификация отсутствует, предполагается кодировка <mark style="color:red;">**utf-8**</mark>.

**yaml.load** возвращает объект Python.

```python
>>> yaml.load(u"""
... hello: Привет!
... """)    # В Python 3 не используйте префикс 'u'

{'hello': u'\u041f\u0440\u0438\u0432\u0435\u0442!'}

>>> stream = file('document.yaml', 'r')    # 'document.yaml' содержит один документ YAML.
>>> yaml.load(stream)
[...]    # Объект Python, соответствующий документу.
```

Если строка или файл содержат несколько документов, вы можете загрузить их все с помощью функции **yaml.load\_all**.

```python
>>> documents = """
... ---
... name: The Set of Gauntlets 'Pauraegen'
... description: >
...     A set of handgear with sparks that crackle
...     across its knuckleguards.
... ---
... name: The Set of Gauntlets 'Paurnen'
... description: >
...   A set of gauntlets that gives off a foul,
...   acrid odour yet remains untarnished.
... ---
... name: The Set of Gauntlets 'Paurnimmen'
... description: >
...   A set of handgear, freezing with unnatural cold.
... """

>>> for data in yaml.load_all(documents):
...     print data

{'description': 'A set of handgear with sparks that crackle across its knuckleguards.\n',
'name': "The Set of Gauntlets 'Pauraegen'"}
{'description': 'A set of gauntlets that gives off a foul, acrid odour yet remains untarnished.\n',
'name': "The Set of Gauntlets 'Paurnen'"}
{'description': 'A set of handgear, freezing with unnatural cold.\n',
'name': "The Set of Gauntlets 'Paurnimmen'"}
```

PyYAML позволяет создавать объект Python любого типа.

```python
>>> yaml.load("""
... none: [~, null]
... bool: [true, false, on, off]
... int: 42
... float: 3.14159
... list: [LITE, RES_ACID, SUS_DEXT]
... dict: {hp: 13, sp: 5}
... """)

{'none': [None, None], 'int': 42, 'float': 3.1415899999999999,
'list': ['LITE', 'RES_ACID', 'SUS_DEXT'], 'dict': {'hp': 13, 'sp': 5},
'bool': [True, False, True, False]}
```

Даже экземпляры классов Python можно создать с помощью тега **!!python/object**.

```python
>>> class Hero:
...     def __init__(self, name, hp, sp):
...         self.name = name
...         self.hp = hp
...         self.sp = sp
...     def __repr__(self):
...         return "%s(name=%r, hp=%r, sp=%r)" % (
...             self.__class__.__name__, self.name, self.hp, self.sp)

>>> yaml.load("""
... !!python/object:__main__.Hero
... name: Welthyr Syxgon
... hp: 1200
... sp: 0
... """)

Hero(name='Welthyr Syxgon', hp=1200, sp=0)
```

Обратите внимание, что возможность создания произвольного объекта Python может быть опасной, если вы получаете документ YAML из ненадежного источника, такого как Интернет. Функция **yaml.safe\_load** ограничивает эту возможность простыми объектами Python, такими как целые числа или списки.

Объект python может быть помечен как безопасный и, таким образом, распознается **yaml.safe\_load**. Для этого унаследуйте его от **yaml.YAMLObject** (как описано в разделе _Конструкторы, репрезентаторы, преобразователи_) и явно установите его свойство класса **yaml\_loader** на **yaml.SafeLoader**.

## Дамп YAML

Функция **yaml.dump** принимает объект Python и создает документ YAML.

```python
>>> print yaml.dump({'name': 'Silenthand Olleander', 'race': 'Human',
... 'traits': ['ONE_HAND', 'ONE_EYE']})

name: Silenthand Olleander
race: Human
traits: [ONE_HAND, ONE_EYE]
```

**yaml.dump** принимает второй необязательный аргумент, который должен быть открытым текстовым или двоичным файлом. В этом случае **yaml.dump** запишет созданный документ YAML в файл. В противном случае **yaml.dump** возвращает созданный документ.

```python
>>> stream = file('document.yaml', 'w')
>>> yaml.dump(data, stream)    # Пишет представление данных YAML в 'document.yaml'.
>>> print yaml.dump(data)      # Выведите документ на экран.
```

Если вам нужно выгрузить несколько документов YAML в один поток, используйте функцию **yaml.dump\_all**. **yaml.dump\_all** принимает список или генератор, производящий объекты Python для сериализации в документ YAML. Второй необязательный аргумент - открытый файл.

```python
>>> print yaml.dump([1,2,3], explicit_start=True)
--- [1, 2, 3]

>>> print yaml.dump_all([1,2,3], explicit_start=True)
--- 1
--- 2
--- 3
```

Вы даже можете сбрасывать экземпляры классов Python.

```python
>>> class Hero:
...     def __init__(self, name, hp, sp):
...         self.name = name
...         self.hp = hp
...         self.sp = sp
...     def __repr__(self):
...         return "%s(name=%r, hp=%r, sp=%r)" % (
...             self.__class__.__name__, self.name, self.hp, self.sp)

>>> print yaml.dump(Hero("Galain Ysseleg", hp=-3, sp=2))

!!python/object:__main__.Hero {hp: -3, name: Galain Ysseleg, sp: 2}
```

**yaml.dump** поддерживает ряд аргументов ключевого слова, которые определяют детали форматирования для эмиттера. Например, вы можете установить предпочтительное назначение и ширину, использовать канонический формат YAML или принудительно использовать предпочтительный стиль для скаляров и коллекций.

```python
>>> print yaml.dump(range(50))
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22,
  23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42,
  43, 44, 45, 46, 47, 48, 49]

>>> print yaml.dump(range(50), width=50, indent=4)
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15,
    16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27,
    28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39,
    40, 41, 42, 43, 44, 45, 46, 47, 48, 49]

>>> print yaml.dump(range(5), canonical=True)
---
!!seq [
  !!int "0",
  !!int "1",
  !!int "2",
  !!int "3",
  !!int "4",
]

>>> print yaml.dump(range(5), default_flow_style=False)
- 0
- 1
- 2
- 3
- 4

>>> print yaml.dump(range(5), default_flow_style=True, default_style='"')
[!!int "0", !!int "1", !!int "2", !!int "3", !!int "4"]
```

## Конструкторы, репрезентаторы, преобразователи

Вы можете определить свои собственные теги для конкретного приложения. Самый простой способ сделать это - определить подкласс **yaml.YAMLObject**:

```python
>>> class Monster(yaml.YAMLObject):
...     yaml_tag = u'!Monster'
...     def __init__(self, name, hp, ac, attacks):
...         self.name = name
...         self.hp = hp
...         self.ac = ac
...         self.attacks = attacks
...     def __repr__(self):
...         return "%s(name=%r, hp=%r, ac=%r, attacks=%r)" % (
...             self.__class__.__name__, self.name, self.hp, self.ac, self.attacks)
```

Приведенного выше определения достаточно для автоматической загрузки и выгрузки объектов Monster:

```python
>>> yaml.load("""
... --- !Monster
... name: Cave spider
... hp: [2,6]    # 2d6
... ac: 16
... attacks: [BITE, HURT]
... """)

Monster(name='Cave spider', hp=[2, 6], ac=16, attacks=['BITE', 'HURT'])

>>> print yaml.dump(Monster(
...     name='Cave lizard', hp=[3,6], ac=16, attacks=['BITE','HURT']))

!Monster
ac: 16
attacks: [BITE, HURT]
hp: [3, 6]
name: Cave lizard
```

**yaml.YAMLObject** использует магию метакласса для регистрации конструктора, который преобразует узел YAML в экземпляр класса, и представителя, который сериализует экземпляр класса в узел YAML.

Если вы не хотите использовать метаклассы, вы можете зарегистрировать свои конструкторы и репрезентаторы с помощью функций **yaml.add\_constructor** и **yaml.add\_presenter**. Например, вы можете добавить конструктор и представитель для следующего класса Dice:

```python
>>> class Dice(tuple):
...     def __new__(cls, a, b):
...         return tuple.__new__(cls, [a, b])
...     def __repr__(self):
...         return "Dice(%s,%s)" % self

>>> print Dice(3,6)
Dice(3,6)
```

Представление по умолчанию для объектов Dice некрасиво:

```python
>>> print yaml.dump(Dice(3,6))

!!python/object/new:__main__.Dice
- !!python/tuple [3, 6]
```

Предположим, вы хотите, чтобы объект Dice был представлен как AdB в YAML:

```python
>>> print yaml.dump(Dice(3,6))

3d6
```

Сначала мы определяем средство представления, которое преобразует объект **dice** в скалярный узел с тегом **!Dice**, затем мы его регистрируем.

```python
>>> def dice_representer(dumper, data):
...     return dumper.represent_scalar(u'!dice', u'%sd%s' % data)

>>> yaml.add_representer(Dice, dice_representer)
```

Теперь вы можете выгрузить экземпляр объекта Dice:

```python
>>> print yaml.dump({'gold': Dice(10,6)})
{gold: !dice '10d6'}
```

Давайте добавим код для создания объекта Dice:

```python
>>> def dice_constructor(loader, node):
...     value = loader.construct_scalar(node)
...     a, b = map(int, value.split('d'))
...     return Dice(a, b)

>>> yaml.add_constructor(u'!dice', dice_constructor)
```

Затем вы также можете загрузить объект Dice:

```python
>>> print yaml.load("""
... initial hit points: !dice 8d4
... """)

{'initial hit points': Dice(8,4)}
```

Возможно, вы не захотите указывать везде тег **!Dice**. Есть способ научить PyYAML, что любой немаркированный простой скаляр, который выглядит как XdY, имеет неявный тег **!Dice**. Используйте **add\_implicit\_resolver**:

```python
>>> import re
>>> pattern = re.compile(r'^\d+d\d+$')
>>> yaml.add_implicit_resolver(u'!dice', pattern)
```

Теперь вам не нужно указывать тег для определения объекта **Dice**:

```python
>>> print yaml.dump({'treasure': Dice(10,20)})

{treasure: 10d20}

>>> print yaml.load("""
... damage: 5d10
... """)

{'damage': Dice(5,10)}
```
