# Api

## Api

#### _class_ flask\_restful.Api(_app=None_, _prefix=''_, _default\_mediatype='application/json'_, _decorators=None_, _catch\_all\_404s=False_, _serve\_challenge\_on\_401=False_, _url\_part\_order='bae'_, _errors=None_)

Основная точка входа для приложения. Вам нужно инициализировать его с помощью приложения **Flask**:

```python
>>> app = Flask(__name__)
>>> api = restful.Api(app)
```

Кроме того, вы можете использовать [init\_app()](api.md#init\_app) для установки приложения **Flask** после его создания.

#### Параметры:

* **app** (_flask.Flask_ или _flask.Blueprint_) - объект приложения Flask
* **prefix** (_str_) - Укажите перед всеми маршрутами значение, например v1 или 2010-04-01.
* **default\_mediatype** (_str_) - media-тип по умолчанию для возврата
* **decorators** (_list_) - Декораторы для прикрепления к каждому ресурсу
* **catch\_all\_404s** (_bool_) - Используйте [handle\_error()](api.md#handle\_error) для обработки ошибок **404** во всем приложении.
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

#### error\_router(_original\_handler_, _e_)

Эта функция решает, произошла ли ошибка в конечной точке flask-restful или нет. Если это произошло в конечной точке flask-restful, наш обработчик будет отправлен. Если это произошло в несвязанном представлении, будет отправлен исходный обработчик ошибок приложения. В случае, если ошибка произошла в конечной точке flask-restful, но локальный обработчик не может разрешить ситуацию, маршрутизатор в крайнем случае вернется к **original\_handler**.

#### Параметры:

* **original\_handler** (_function_) - оригинальный обработчик ошибок Flask для приложения
* **e** (_Exception_) - исключение, возникающее при обработке запроса

### handle\_error()

#### handle\_error(_e_)

Обработчик ошибок для API преобразует возникшее исключение в ответ Flask с соответствующим кодом состояния HTTP и телом.

#### Параметры:

* **e** (_Exception_) - поднятый объект Exception

### init\_app()

#### init\_app(_app_)

Инициализируйте этот класс с данным приложением [flask.Flask](https://flask.palletsprojects.com/en/1.1.x/api/#flask.Flask) или объектом [flask.Blueprint](https://flask.palletsprojects.com/en/1.1.x/api/#flask.Blueprint).

#### Параметры:

* **app** (_flask.Blueprint_) - приложение Flask или объект чертежа blueprint

Примеры:

```python
api = Api()
api.add_resource(...)
api.init_app(app)
```

### make\_response()

#### make\_response(_data_, _\*args_, _\*\*kwargs_)

Ищет преобразователь представления для запрошенного типа мультимедиа, вызывая преобразователь для создания объекта ответа. По умолчанию используется значение **default\_mediatype**, если для запрошенного типа носителя не найдено ни одного преобразователя. Если **default\_mediatype** имеет значение `None`, ответ **406 Not Acceptable** будет отправлен в соответствии с RFC 2616, раздел 14.1.

#### Параметры:

* **data** - Объект Python, содержащий данные ответа, которые необходимо преобразовать

### mediatypes()

Возвращает список запрошенных медиатипов, отправленных в заголовке **Accept**.

### mediatypes\_method()

Возвращает метод, который возвращает список медиатипов

### output()

#### output(_resource_)

Обертывает ресурс (как функцию представления Flask) для случаев, когда ресурс не возвращает объект ответа напрямую.

#### Параметры:

* **resource** - Ресурс как функция view Flask

### owns\_endpoint()

#### owns\_endpoint(_endpoint_)

Проверяет, принадлежит ли имя конечной точки (не путь) этому API. Принимает во внимание часть имени **Blueprint** имени конечной точки.

#### Параметры:

* **endpoint** - Имя проверяемой конечной точки

**Возвращает**: bool

### representation()

#### representation(_mediatype_)

Позволяет объявить дополнительные преобразователи представления для API. Преобразователи — это функции, которые должны быть декорированы этим методом, передавая медиатип, который представляет преобразователь. Преобразователю передаются три аргумента:

* Данные, которые должны быть представлены в теле ответа
* Код состояния http
* Словарь заголовков

Преобразователь должен преобразовать данные соответствующим образом для типа носителя и вернуть объект ответа Flask.

Например:

```python
@api.representation('application/xml')
def xml(data, code, headers):
    resp = make_response(convert_data_to_xml(data), code)
    resp.headers.extend(headers)
    return resp
```

### resource()

#### resource(_\*urls_, _\*\*kwargs_)

Обертывает класс <mark style="color:red;">Resource</mark>, добавляя его в API. Параметры такие же, как у [add\_resource()](api.md#add\_resource).

Пример:

```python
app = Flask(__name__)
api = restful.Api(app)

@api.resource('/foo')
class Foo(Resource):
    def get(self):
        return 'Hello, World!'
```

### unauthorized()

#### unauthorized(_response_)

Получив ответ, измените его, чтобы запросить учетные данные

### url\_for()

#### url\_for(_resource_, _\*\*values_)

Создает URL-адрес для данного ресурса.

Работает как [flask.url\_for()](https://flask.palletsprojects.com/en/1.1.x/api/#flask.url\_for).

## Resource

#### _class_ flask\_restful.Resource

Представляет абстрактный ресурс **RESTful**. Конкретные ресурсы должны расширяться из этого класса и предоставлять методы для каждого поддерживаемого метода HTTP. Если ресурс вызывается с помощью неподдерживаемого HTTP-метода, API вернет ответ со статусом **405 Method Not Allowed**. В противном случае вызывается соответствующий метод и передаются все аргументы из правила URL, используемого при добавлении ресурса в экземпляр API. Подробности смотрите в [add\_resource()](api.md#add\_resource).

### dispatch\_request()

#### dispatch\_request(_\*args_, _\*\*kwargs_)

Подклассы должны переопределить этот метод, чтобы реализовать фактический код функции представления. Этот метод вызывается со всеми аргументами из правила URL.
