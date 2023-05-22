# Использование benedict

## Основы

**benedict** является подклассом **dict**, поэтому его можно использовать как обычный словарь (вы можете просто преобразовать существующий **dict**).

```python
from benedict import benedict

# создать новый пустой экземпляр
d = benedict()

# или использовать существующий словарь
d = benedict(existing_dict)

# или создать из источника данных (путь к файлу, URL-адрес или строка данных)
# в поддерживаемом формате:
# Base64, CSV, JSON, TOML, XML, YAML, query-string
d = benedict("https://localhost:8000/data.json", format="json")

# или в представлении Django
params = benedict(request.GET.items())
page = params.get_int("page", 1)
```

## Keyattr my\_dict.x.y.z

Можно получить/установить элементы, используя **ключи в качестве атрибутов** (точечная запись).

```python
d = benedict(keyattr_dynamic=True) # по умолчанию False
d.profile.firstname = "Fabio"
d.profile.lastname = "Caccamo"
print(d) # -> { "profile":{ "firstname":"Fabio", "lastname":"Caccamo" } }
```

По умолчанию, если для **keyattr\_dynamic** явно не установлено значение `True`, эта функция работает только для получения/установки уже существующих элементов.

### Отключить функцию keyattr

Вы можете отключить функцию **keyattr**, передав параметр `keyattr_enabled=False` в конструкторе.

```python
d = benedict(existing_dict, keyattr_enabled=False) # по умолчанию True
```

или используя свойство `getter/setter`.

```python
d.keyattr_enabled = False
```

### Функциональность динамического keyattr

Вы можете включить функцию доступа к динамическим атрибутам, передав `keyattr_dynamic=True` в конструкторе.

```python
d = benedict(existing_dict, keyattr_dynamic=True) # по умолчанию False
```

или используя свойство `getter/setter`.

```python
d.keyattr_dynamic = True
```

{% hint style="info" %}
Предупреждение — даже если эта функция очень полезна, у нее есть некоторые очевидные ограничения: она работает только для строковых ключей, которые не защищены (не начинаются с **\_**) и которые не конфликтуют с именами поддерживаемых в настоящее время методов.
{% endhint %}

## Keylist my\_dict\["x", "y", "z"]

Везде, где используется **ключ**, можно также использовать **список (или кортеж) ключей**.

```python
d = benedict()

# установить значения по списку ключей
d["profile", "firstname"] = "Fabio"
d["profile", "lastname"] = "Caccamo"
print(d) # -> { "profile":{ "firstname":"Fabio", "lastname":"Caccamo" } }
print(d["profile"]) # -> { "firstname":"Fabio", "lastname":"Caccamo" }

# проверить, существует ли ключевой путь в словаре
print(["profile", "lastname"] in d) # -> True

# удалить значение по списку ключей
del d["profile", "lastname"]
print(d["profile"]) # -> { "firstname":"Fabio" }
```

## Keypath my\_dict\["xyz"]

`.` является разделителем пути по умолчанию.

Если вы создадите существующий **dict** и его ключи содержат разделитель пути к ключу, будет вызвано значение **ValueError**.

В этом случае вы должны использовать [пользовательский разделитель пути к ключу](ispolzovanie-benedict.md#polzovatelskii-razdelitel-keypath) или [отключить функцию пути к ключу](ispolzovanie-benedict.md#otklyuchit-funkciyu-keypath).

### Пользовательский разделитель keypath

Вы можете настроить разделитель пути к ключу, передав аргумент **keypath\_separator** в конструкторе.

Если вы передадите существующий **dict** в конструктор, а его ключи содержат разделитель пути к ключу, будет возбуждено исключение **Exception**.

```python
d = benedict(existing_dict, keypath_separator="/")
```

### Изменение разделителя keypath

Вы можете изменить **keypath\_separator** в любое время, используя свойство `getter/setter`.

Если какой-либо существующий ключ содержит новый **keypath\_separator**, будет возбуждено исключение **Exception**.

```python
d.keypath_separator = "/"
```

### Отключить функцию keypath

Вы можете отключить функцию **keypath**, передав параметр `keypath_separator=None` в конструкторе.

```python
d = benedict(existing_dict, keypath_separator=None)
```

или используя свойство `getter/setter`.

```python
d.keypath_separator = None
```

### Поддержка списка индексов

Поддерживаются списковые индексы, ключевые пути могут включать индексы (также отрицательные) с использованием `[n]` для очень быстрого выполнения любой операции:

```python
# Например, получить последние координаты местоположения первого результата:
loc = d["results[0].locations[-1].coordinates"]
lat = loc.get_decimal("latitude")
lng = loc.get_decimal("longitude")
```
