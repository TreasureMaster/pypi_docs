# Объекты Blueprint

## Класс flask.Blueprint( _name_, _import\_name_, _static\_folder=None_, _static\_url\_path=None_, _template\_folder=None_, _url\_prefix=None_, _subdomain=None_, _url\_defaults=None_, _root\_path=None_, _cli\_group=\<object object>_ )

_Новое в версии 0.7_.

Представляет схему _**blueprint**_, набор маршрутов и другие функции, связанные с приложением, которые можно позже зарегистрировать в реальном приложении.

_**blueprint**_ - это объект, который позволяет определять функции приложения без предварительного запроса объекта приложения. Он использует те же декораторы, что и [Flask](obekt-prilozheniya-flask.md#klass-flask-flask-import\_name-static\_url\_path-none-static\_folder-static-static\_host-none-host\_matching-false-subdomain\_matching-false-template\_folder-templates-instance\_path-none-instance\_relative\_config-false-root\_path-none), но устраняет необходимость в приложении, записывая их для последующей регистрации.

Декорирование функции схемой _**blueprint**_ создает отложенную функцию, которая вызывается с помощью [BlueprintSetupState](poleznye-funkcii-flask.md#klass-flask-blueprints-blueprintsetupstate), когда _**blueprint**_ регистрируется в приложении.

См. [Модульные приложения с Blueprints](../rukovodstvo-polzovatelya-flask/modulnye-prilozheniya-s-blueprints.md) для получения дополнительной информации.

_Изменено в версии 1.1.0:_ в _**blueprints**_ есть группа **cli** для регистрации вложенных команд CLI. Параметр _**cli\_group**_ управляет именем группы в команде **flask**.

### Параметры flask.Blueprint

* **name** - Название схемы _**blueprint**_. Будет добавлено к каждому имени конечной точки.
* **import\_name** - Имя пакета _**blueprints**_, обычно `__name__`. Это помогает найти **root\_path** для схемы _**blueprint**_.
* **static\_folder** - Папка со статическими файлами, которые должны обслуживаться статическим маршрутом _**blueprint**_. Путь определяется относительно корневого пути _**blueprint**_. По умолчанию статические файлы **Blueprint** отключены.
* **static\_url\_path -** URL-адрес для обслуживания статических файлов. По умолчанию _**static\_folder**_. Если в схеме не указан _**url\_prefix**_, статический маршрут приложения будет иметь приоритет, а статические файлы схемы _**blueprint**_ будут недоступны.
* **template\_folder -** Папка с шаблонами, которую нужно добавить в путь поиска шаблонов приложения. Путь определяется относительно корневого пути _**blueprint**_. По умолчанию шаблоны **Blueprint** отключены. Шаблоны **Blueprint** имеют более низкий приоритет, чем шаблоны в папке шаблонов приложения.
* **url\_prefix -** Путь, добавляемый ко всем URL-адресам проекта, чтобы отличать их от остальных маршрутов приложения.
* **subdomain -** Поддомен, на котором будут совпадать маршруты _**blueprints**_ по умолчанию.
* **url\_defaults -** Набор значений по умолчанию, которые маршруты _**blueprints**_ будут получать по умолчанию.
* **root\_path -** По умолчанию _**blueprint**_ будет автоматически основан на _**import\_name**_. В определенных ситуациях это автоматическое обнаружение может дать сбой, поэтому путь можно указать вручную.

### add\_app\_template\_filter( _f_, _name=None_ )

Регистрирует настраиваемый шаблон фильтра, доступный для всего приложения. Как [Flask.add\_template\_filter ()](obekt-prilozheniya-flask.md#add\_template\_filter-f-name-none), но для схемы _**blueprint**_. Работает точно так же, как декоратор [app\_template\_filter ()](obekty-blueprint.md#app\_template\_filter).

**Параметры**: _**name**_ - необязательное имя фильтра, иначе будет использовано имя функции.

### add\_app\_template\_global( _f_, _name=None_ )

_Новое в версии 0.10_.

Регистрирует  глобальный настраиваемый шаблон, доступный для всего приложения. Как [Flask.add\_template\_global ()](obekt-prilozheniya-flask.md#add\_template\_global-f-name-none), но для _**blueprint**_. Работает точно так же, как декоратор [app\_template\_global ()](obekty-blueprint.md#app\_template\_global).

**Параметры**: _**name**_ - необязательное имя глобального шаблона, иначе будет использовано имя функции.

### add\_app\_template\_test( _f_, _name=None_ )

_Новое в версии 0.10_.

Регистрирует собственный шаблон теста, доступный для всего приложения. Как [Flask.add\_template\_test ()](obekt-prilozheniya-flask.md#add\_template\_test-f-name-none), но для _**blueprint**_. Работает точно так же, как декоратор [app\_template\_test ()](obekty-blueprint.md#app\_template\_test).

**Параметры**: _**name**_ - необязательное имя теста, иначе будет использовано имя функции.

### add\_url\_rule( _rule_, _endpoint=None_, _view\_func=None_, _\*\*options_ )

Как [Flask.add\_url\_rule ()](obekt-prilozheniya-flask.md#add\_url\_rule-rule-endpoint-none-view\_func-none-provide\_automatic\_options-none-options), но для _**blueprint**_. Конечная точка для функции [url\_for ()](poleznye-funkcii-i-klassy-flask.md#flask-url\_for-endpoint-values) имеет префикс с именем _**blueprint**_.

### after\_app\_request( _f_ )

Как [Flask.after\_request ()](obekt-prilozheniya-flask.md#after\_request-f), но для _**blueprint**_. Такая функция выполняется после каждого запроса, даже если она находится вне _**blueprint**_.

### after\_request( _f_ )

Как [Flask.after\_request ()](obekt-prilozheniya-flask.md#after\_request-f), но для _**blueprint**_. Эта функция выполняется только после каждого запроса, который обрабатывается функцией этого _**blueprint**_.

### app\_context\_processor( _f_ )

Как [Flask.context\_processor ()](obekt-prilozheniya-flask.md#context\_processor-f), но для _**blueprint**_. Такая функция выполняется при каждом запросе, даже если она находится вне _**blueprint**_.

### app\_errorhandler( _code_ )

Как [Flask.errorhandler ()](obekt-prilozheniya-flask.md#errorhandler-code\_or\_exception), но для _**blueprint**_. Этот обработчик используется для всех запросов, даже если они не входят в _**blueprint**_.

### app\_template\_filter( _name=None_ )

Регистрирует настраиваемый шаблон фильтра, доступный для всего приложения. Как [Flask.template\_filter ()](obekt-prilozheniya-flask.md#template\_filter-name-none), но для _**blueprint**_.

**Параметры**: _**name**_ - необязательное имя фильтра, иначе будет использовано имя функции.

### app\_template\_global( _name=None_ )

_Новое в версии 0.10_.

Регистрирует глобальный настраиваемый шаблон, доступный для всего приложения. Как [Flask.template\_global ()](obekt-prilozheniya-flask.md#template\_global-name-none), но для _**blueprint**_.

**Параметры**: _**name**_ - необязательное имя глобального шаблона, иначе будет использовано имя функции.

### app\_template\_test( _name=None_ )

_Новое в версии 0.10_.

Регистрирует собственный шаблон теста, доступный для всего приложения. Как [Flask.template\_test ()](obekt-prilozheniya-flask.md#template\_test-name-none), но для _**blueprint**_.

**Параметры**: _**name**_ - необязательное имя теста, иначе будет использовано имя функции.

### app\_url\_defaults( _f_ )

То же, что и [url\_defaults ()](obekty-blueprint.md#url\_defaults), но для всего приложения.

### app\_url\_value\_preprocessor( _f_  )

То же, что [url\_value\_preprocessor ()](obekty-blueprint.md#url\_value\_preprocessor), но для всего приложения.

### before\_app\_first\_request( _f_ )

Как [Flask.before\_first\_request ()](obekt-prilozheniya-flask.md#before\_first\_request-f). Такая функция выполняется перед первым запросом к приложению.

### before\_app\_request( _f_ )

Как [Flask.before\_request ()](obekt-prilozheniya-flask.md#before\_request-f). Такая функция выполняется перед каждым запросом, даже если не входит в _**blueprint**_.

### before\_request( _f_ )

Как [Flask.before\_request ()](obekt-prilozheniya-flask.md#before\_request-f), но для _**blueprint**_. Эта функция выполняется только перед каждым запросом, который обрабатывается функцией этого _**blueprint**_.

### context\_processor( _f_ )

Как [Flask.context\_processor ()](obekt-prilozheniya-flask.md#context\_processor-f), но для _**blueprint**_. Эта функция выполняется только для запросов, обрабатываемых _**blueprint**_.

### endpoint( _endpoint_ )

Как [Flask.endpoint ()](obekt-prilozheniya-flask.md#endpoint-endpoint), но для _**blueprint**_. Это не префикс конечной точки с именем схемы _**blueprint**_, это должно быть сделано явно пользователем этого метода. Если конечная точка имеет префикс. он будет зарегистрирован в текущем проекте, в противном случае это конечная точка, независимая от приложения.

### errorhandler(_code\_or\_exception_ )

Регистрирует обработчик ошибок, который становится активным только для этого _**blueprint**_. Имейте в виду, что маршрутизация не происходит локально для схемы _**blueprint**_, поэтому обработчик ошибок для **404** обычно не обрабатывается схемой _**blueprint**_, если только она не вызывается внутри функции представления. Другой особый случай - это внутренняя ошибка сервера **500**, которая всегда просматривается из приложения.

В противном случае работает как декоратор [errorhandler ()](obekt-prilozheniya-flask.md#errorhandler-code\_or\_exception) объекта [Flask](obekt-prilozheniya-flask.md#klass-flask-flask-import\_name-static\_url\_path-none-static\_folder-static-static\_host-none-host\_matching-false-subdomain\_matching-false-template\_folder-templates-instance\_path-none-instance\_relative\_config-false-root\_path-none).

### get\_send\_file\_max\_age( _filename_ )

_Новое в версии 0.9_.

Предоставляет _**cache\_timeout**_ по умолчанию для функций [send\_file ()](poleznye-funkcii-i-klassy-flask.md#flask-send\_file-filename\_or\_fp-mimetype-none-as\_attachment-false-attachment\_filename-none-add\_etags-true-cache\_timeout-none-conditional-false-last\_modified-none).

По умолчанию эта функция возвращает **SEND\_FILE\_MAX\_AGE\_DEFAULT** из конфигурации [current\_app](poleznye-funkcii-i-klassy-flask.md#flask-current\_app).

Статические файловые функции, такие как [send\_from\_directory ()](poleznye-funkcii-i-klassy-flask.md#flask-send\_from\_directory-directory-filename-options), используют эту функцию, а [send\_file ()](poleznye-funkcii-i-klassy-flask.md#flask-send\_file-filename\_or\_fp-mimetype-none-as\_attachment-false-attachment\_filename-none-add\_etags-true-cache\_timeout-none-conditional-false-last\_modified-none) вызывает эту функцию в [current\_app](poleznye-funkcii-i-klassy-flask.md#flask-current\_app), когда заданное _**cache\_timeout**_ равно `None`. Если в **send\_file ()** задано _**cache\_timeout**_, используется этот тайм-аут; в противном случае вызывается этот метод.

Это позволяет подклассам изменять поведение при отправке файлов на основе имени файла. Например, чтобы установить таймаут кеширования для файлов `.js` равным 60 секундам:

```python
class MyFlask(flask.Flask):
    def get_send_file_max_age(self, name):
        if name.lower().endswith('.js'):
            return 60
        return flask.Flask.get_send_file_max_age(self, name)
```

### _(property)_ has\_static\_folder

_Новое в версии 0.5_.

Это `True`, если в контейнере объекта, привязанного к пакету, есть папка для статических файлов.

### import\_name _= None_

Имя пакета или модуля, которому принадлежит это приложение. Не изменяйте это значение, если оно установлено конструктором.

### jinja\_loader

_Новое в версии 0.5_.

Загрузчик **Jinja** для этого связанного с пакетом объекта.

### json\_decoder _= None_

_**Blueprint**_ использования локального класса декодера **JSON**. Установите значение `None`, чтобы использовать **json\_decoder** приложения.

### json\_encoder _= None_

_**Blueprint**_ использования локального класса декодера **JSON**. Установите значение `None`, чтобы использовать **json\_encoder** приложения.

### make\_setup\_state( _app_, _options_, _first\_registration=False_ )

Создает экземпляр объекта [BlueprintSetupState ()](poleznye-funkcii-flask.md#klass-flask-blueprints-blueprintsetupstate), который позже передается функциям обратного вызова регистра. Подклассы могут переопределить это, чтобы вернуть подкласс состояния настройки.

### open\_resource( _resource_, _mode='rb'_ )

Открывает ресурс из папки ресурсов приложения. Чтобы увидеть, как это работает, рассмотрим следующую структуру папок:

```bash
/myapplication.py
/schema.sql
/static
    /style.css
/templates
    /layout.html
    /index.html
```

Если вы хотите открыть файл `schema.sql`, сделайте следующее:

```python
with app.open_resource('schema.sql') as f:
    contents = f.read()
    do_something_with(contents)
```

**Параметры:**

* **resource** - название ресурса. Для доступа к ресурсам в подпапках используйте косую черту в качестве разделителя.
* **mode** - Открывает файл в этом режиме. Поддерживается только чтение, допустимые значения - `«r»` (или `«rt»`) и `«rb»`.

### record( _func_ )

Регистрирует функцию, вызываемую при регистрации схемы _**blueprint**_ в приложении. Эта функция вызывается с состоянием в качестве аргумента, возвращаемого методом [make\_setup\_state ()](obekty-blueprint.md#make\_setup\_space-app-options-first\_registration-false).

### record\_once( _func_ )

Работает как [record ()](obekty-blueprint.md#record-func), но объединяет функцию в другую функцию, которая гарантирует, что функция вызывается только один раз. Если _**blueprint**_ второй раз регистрируется в приложении, переданная функция не вызывается.

### register( _app_, _options_, _first\_registration=False_ )

Вызывается [Flask.register\_blueprint ()](obekt-prilozheniya-flask.md#register\_blueprint-blueprint-options) для регистрации всех представлений и обратных вызовов, зарегистрированных в схеме _**blueprint**_, в приложении. Создает [BlueprintSetupState](poleznye-funkcii-flask.md#klass-flask-blueprints-blueprintsetupstate) и вызывает с ним обратный вызов каждого [record ()](obekty-blueprint.md#record-func).

**Параметры:**

* **app** - Приложение, в котором регистрируется этот _**blueprint**_.
* **options** - Аргументы ключевого слова пересылаются из [register\_blueprint ()](obekt-prilozheniya-flask.md#register\_blueprint-blueprint-options).
* **first\_registration** - Впервые ли этот _**blueprint**_ был зарегистрирован в приложении.

### register\_error\_handler( _code\_or\_exception_, _f_ )

_Новое в версии 0.11_.

Версия функции присоединения ошибок [errorhandler ()](obekty-blueprint.md#errorhandler-code\_or\_exception) без декоратора, аналогичная функции [register\_error\_handler ()](obekt-prilozheniya-flask.md#register\_error\_handler-code\_or\_exception-f) для всего приложения объекта [Flask](obekt-prilozheniya-flask.md#klass-flask-flask-import\_name-static\_url\_path-none-static\_folder-static-static\_host-none-host\_matching-false-subdomain\_matching-false-template\_folder-templates-instance\_path-none-instance\_relative\_config-false-root\_path-none), но для обработчиков ошибок, ограниченных этой схемой _**blueprint**_.

### root\_path _= None_

Абсолютный путь к пакету в файловой системе. Используется для поиска ресурсов, содержащихся в пакете.

### route( _rule_, _\*\*options_ )

Как [Flask.route ()](obekt-prilozheniya-flask.md#route-rule-options), но для _**blueprint**_. Конечная точка для функции [url\_for ()](poleznye-funkcii-i-klassy-flask.md#flask-url\_for-endpoint-values) имеет префикс с именем схемы _**blueprint**_.

### send\_static\_file( _filename_ )

_Новое в версии 0.5_.

Функция, используемая внутри для отправки статических файлов из статической папки в браузер.

### _(property)_ static\_folder

Абсолютный путь к настроенной статической папке.

### _(property)_ static\_url\_path

Префикс URL-адреса, с которого будет доступен статический маршрут.

Если он не был настроен во время инициализации, он является производным от [static\_folder](obekty-blueprint.md#property-static\_folder).

### teardown\_app\_request( _f_ )

Как [Flask.teardown\_request ()](obekt-prilozheniya-flask.md#teardown\_request-f), но для _**blueprint**_. Такая функция выполняется при _**teardown**_ каждого запроса, даже если он находится вне _**blueprint**_.

### teardown\_request( _f_ )

Как [Flask.teardown\_request ()](obekt-prilozheniya-flask.md#teardown\_request-f), но для _**blueprint**_. Эта функция выполняется только при отключении запросов, обрабатываемых функцией этого _**blueprint**_. Функции _**teardown**_ запроса выполняются при открытии контекста запроса, даже если фактический запрос не выполнялся.

### template\_folder _= None_

Расположение файлов шаблона, которые нужно добавить в поиск по шаблону. `None`, если не нужно добавлять шаблоны.

### url\_defaults( _f_ )

Функция обратного вызова для значений URL по умолчанию для этого _**blueprint**_. Он вызывается с конечной точкой и значениями и должен обновлять переданные значения.

### url\_value\_preprocessor( _f_ )

Регистрирует функцию как препроцессор значения URL для этого _**blueprint**_. Он вызывается перед вызовом функций просмотра и может изменять предоставленные значения URL.
