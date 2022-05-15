# Api

## Api

#### _class_ flask\_restful.Api(_app=None_, _prefix=''_, _default\_mediatype='application/json'_, _decorators=None_, _catch\_all\_404s=False_, _serve\_challenge\_on\_401=False_, _url\_part\_order='bae'_, _errors=None_)

Основная точка входа для приложения. Вам нужно инициализировать его с помощью приложения **Flask**:

```python
>>> app = Flask(__name__)
>>> api = restful.Api(app)
```

Кроме того, вы можете использовать <mark style="color:red;">init\_app()</mark> для установки приложения **Flask** после его создания.

#### Параметры:

* **app** (_flask.Flask_ или _flask.Blueprint_) - объект приложения Flask
* **prefix** (_str_) - Укажите перед всеми маршрутами значение, например v1 или 2010-04-01.
* **default\_mediatype** (_str_) - media-тип по умолчанию для возврата
* **decorators** (_list_) - Декораторы для прикрепления к каждому ресурсу
* **catch\_all\_404s** (_bool_) - Используйте <mark style="color:red;">handle\_error()</mark> для обработки ошибок **404** во всем приложении.
* **serve\_challenge\_on\_401** - Отправлять ли ответ на вызов клиентам при получении **401**. Обычно это приводит к появлению всплывающего окна с именем пользователя и паролем в веб-браузерах.
* **url\_part\_order** - Строка, управляющая порядком объединения частей URL-адреса при создании полного URL-адреса. `«b»` — префикс схемы (или регистрации схемы), `«a»` — префикс API, а `«e»` — компонент пути, с которым добавляется конечная точка.
* **errors** - Словарь для определения пользовательского ответа для каждого исключения или ошибки, возникшей во время запроса.

### add\_resource()

#### add\_resource(_resource_, _\*urls_, _\*\*kwargs_)

Добавляет ресурс в API.

#### Параметры:

* **resource** (_Type\[Resource]_) - имя класса вашего ресурса
* **urls** (_str_) - один или несколько маршрутов URL-адресов для соответствия ресурсу, применяются стандартные правила маршрутизации Flask. Любые переменные URL будут переданы методу ресурсов в качестве аргументов.
* **endpoint** (_str_) - имя конечной точки (по умолчанию **Resource.\_\_name\_\_.lower()**) Может использоваться для ссылки на этот маршрут в полях <mark style="color:red;">fields.Url</mark>
* **resource\_class\_args** (_tuple_) - args для пересылки конструктору ресурса.
* **resource\_class\_kwargs** (_dict_) - kwargs для пересылки конструктору ресурса.

Дополнительные аргументы ключевого слова, не указанные выше, будут переданы как есть в [flask.Flask.add\_url\_rule()](https://flask.palletsprojects.com/en/1.1.x/api/#flask.Flask.add\_url\_rule).

Примеры:

```python
api.add_resource(HelloWorld, '/', '/hello')
api.add_resource(Foo, '/foo', endpoint="foo")
api.add_resource(FooSpecial, '/special/foo', endpoint="foo")
```

### **error\_router()**

#### `error_router`(_original\_handler_, _e_)
