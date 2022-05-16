# ReqParse

## ReqParse

#### _class_ reqparse.RequestParser(_argument\_class=\<class 'reqparse.Argument'>_, _namespace\_class=\<class 'reqparse.Namespace'>_, _trim=False_, _bundle\_errors=False_)

Позволяет добавлять и анализировать несколько аргументов в контексте одного запроса. Пример:

```python
from flask_restful import reqparse

parser = reqparse.RequestParser()
parser.add_argument('foo')
parser.add_argument('int_bar', type=int)
args = parser.parse_args()
```

#### Параметры:

* **trim** (_bool_) - Если включено, обрезает пробелы во всех аргументах в этом синтаксическом анализаторе.
* **bundle\_errors** (_bool_) - Если включено, не прерывать работу при возникновении первой ошибки, возвращать dict с именем аргумента и сообщением об ошибке, которое должно быть объединено, и возвращать все ошибки проверки.

### add\_argument()

#### add\_argument(_\*args_, _\*\*kwargs_)

Добавляет аргумент для анализа.

Принимает либо один экземпляр [Argument](reqparse.md#argument), либо аргументы для передачи в конструктор [Argument](reqparse.md#argument).

См. конструктор [Argument](reqparse.md#argument) для документации по доступным опциям.

## copy()

Создает копию этого **RequestParser** с тем же набором аргументов.

### parse\_args()

#### parse\_args(_req=None_, _strict=False_, _http\_error\_code=400_)

Анализирует все аргументы из предоставленного запроса и возвращает результаты в виде пространства имен.

#### Параметры:

* **req** - Может использоваться для перезаписи запроса от Flask
* **strict** - если **req** включает аргументы, которых нет в синтаксическом анализаторе, сгенерировать исключение **400 BadRequest**
* **http\_error\_code** - использовать собственный код ошибки для `flask_restful.abort()`

### remove\_argument()

#### remove\_argument(_name_)

Удаляет аргумент, соответствующий данному имени.

### replace\_argument()

#### replace\_argument(_name_, _\*args_, _\*\*kwargs_)

Заменяет аргумент, соответствующий данному имени, новой версией.

## Argument

#### _class_ reqparse.Argument(_name_, _default=None_, _dest=None_, _required=False_, _ignore=False_, _type=\<function \<lambda>>_, _location=('json'_, _'values')_, _choices=()_, _action='store'_, _help=None_, _operators=('='_, _)_, _case\_sensitive=True_, _store\_missing=True_, _trim=False_, _nullable=True_)

#### Параметры:

* **name** - Либо имя, либо список строк параметров, например: `foo` или `-f`, `-foo`.
* **default** - Значение, полученное, если аргумент отсутствует в запросе.
* **dest** - Имя атрибута, добавляемого к объекту, возвращаемому функцией [parse\_args()](reqparse.md#parse\_args).
* **required** (_bool_) - Может ли аргумент быть опущен (только необязательные параметры).
* **action** - Основной тип действия, которое необходимо предпринять, когда этот аргумент встречается в запросе. Допустимые варианты: `«store»` и `«append»`.
* **ignore** - Игнорировать ли случаи, когда аргумент не удается преобразовать в тип
* **type** - Тип, в который должен быть преобразован аргумент запроса. Если тип вызывает исключение, сообщение об ошибке будет возвращено в ответе. По умолчанию используется **unicode** в python2 и [str](https://docs.python.org/3/library/stdtypes.html#str) в python3.
* **location** - Атрибуты объекта [flask.Request](https://flask.palletsprojects.com/en/1.1.x/api/#flask.Request) для получения аргументов (например, _**headers**_, _**args**_ и т. д.); могут быть итератором. Последний элемент в списке имеет приоритет в результирующем наборе.
* **choices** - Контейнер допустимых значений аргумента.
* **help** - Краткое описание аргумента, возвращаемого в ответе, когда аргумент недействителен. Опционально может содержать токен интерполяции `«{error_msg}»`, который будет заменен текстом ошибки, вызванной преобразователем типов.
* **case\_sensitive** (_bool_) - Чувствительны ли значения аргументов в запросе к регистру или нет (при этом все значения будут преобразованы в нижний регистр)
* **store\_missing** (_bool_) - Следует ли сохранять значение аргументов по умолчанию, если аргумент отсутствует в запросе.
* **trim** (_bool_) - Если включено, обрезает пробелы вокруг аргумента.
* **nullable** (_bool_) - Если включено, разрешает null значение в аргументе.

### \_\_init\_\_()

#### \_\_init\_\_(_name_, _default=None_, _dest=None_, _required=False_, _ignore=False_, _type=\<function \<lambda>>_, _location=('json'_, _'values')_, _choices=()_, _action='store'_, _help=None_, _operators=('='_, _)_, _case\_sensitive=True_, _store\_missing=True_, _trim=False_, _nullable=True_)

Инициализировать self. См. `help(type(self))` для точной сигнатуры.

### handle\_validation\_error()

#### handle\_validation\_error(_error_, _bundle\_errors_)

Вызывается, когда возникает ошибка при синтаксическом анализе. Прерывает запрос со статусом **400** и сообщением об ошибке

#### Параметры:

* **error** - ошибка, которая была поднята
* **bundle\_errors** - не прерывать работу при возникновении первой ошибки, возвращать dict с именем аргумента и сообщением об ошибке, которое будет объединено

### parse()

#### parse(_request_, _bundle\_errors=False_)

Анализирует значения аргументов из запроса, преобразуя их в соответствии с типом аргумента.

#### Параметры:

* **request** - Объект запроса Flask для анализа аргументов
* **bundle\_errors** - не прерывать работу при возникновении первой ошибки, возвращать dict с именем аргумента и сообщением об ошибке, которое будет объединено

### source()

#### source(_request_)

Извлекает значения из запроса в указанном месте `:param request:` Объект запроса Flask для анализа аргументов.
