# Справочник по API Flask-Nav

### &#x20;_class_ flask\_nav.Nav(_app=None_)

Расширение **Flask-Nav**.

#### Параметры:

* _**app**_ - Необязательное приложение **Flask** для инициализации.

#### Методы:

* &#x20;**init\_app(**_**app**_** )** - Инициализировать приложение. Параметры:
  * _**app**_ - приложение **Flask**.
* &#x20;**navigation(**_**id=None**_** )** - Декоратор функций для регистрации **navbar**. Функция удобства вызывает **register\_element ()** с идентификатором `id` и декорированную функцию как `elem`. Параметры:
  * _**id**_ - ID для передачи. Если `None`, используется имя декорируемой функции.
* &#x20;**register\_element(**_**id**_**, **_**elem**_**)** - Регистрирует навигационный элемент. Регистрирует данный навигационный элемент, делая его доступным по идентификатору `id`. Это означает, что внутри любого шаблона зарегистрированный элемент будет доступен как `nav.id`. Если `elem` является вызываемым, любая попытка получить его внутри шаблона вместо этого приведет к вызову `elem` и возвращению результата. Параметры:
  * _**id**_ - Идентификатор для регистрации элемента
  * _**elem**_ - Элемент, который должен быть зарегистрирован
* &#x20;**renderer(**_**id=None**_**, **_**force=True**_** )** - Декоратор классов для рендереров (Renderers). Декорированный класс будет добавлен в список средств визуализации, хранящийся в этом экземпляре, который будет зарегистрирован в приложении при инициализации приложения. Параметры:
  * _**id**_ - Идентификатор для средства визуализации, по умолчанию используется имя класса в верблюжьем стиле нотаций.
  * _**force**_ - Следует ли перезаписывать существующие рендереры.

### &#x20;_**function**_ flask\_nav.get\_renderer( _app_, _id_ )

Получить средство визуализации.

#### Параметры:

* _**app**_ - Приложение **Flask** для поиска идентификаторов
* _**id**_ - Строка идентификатора внутреннего рендерера для поиска

### &#x20;_function_ flask\_nav.register\_renderer( _app_, _id_, _renderer_, _force=True_ )

Регистрирует средство визуализации в приложении.

#### Параметры:

* _**app**_ - Приложение **Flask** для регистрации рендерера
* _**id**_ - Внутренняя строка идентификатора для рендерера
* _**renderer**_ - Рендерер для регистрации
* _**force**_ - Следует ли перезаписывать рендерер, если для идентификатора _**id**_ уже зарегистрирован другой.

### &#x20;_class_ flask\_nav.elements.Link( _text_, _dest_ )

Элемент, содержащий ссылку на пункт назначения и заголовок.

### &#x20;_class_ flask\_nav.elements.Navbar(_title_, _\*items_)

Навигационная панель верхнего уровня.

### &#x20;_class_ flask\_nav.elements.NavigationItem

Основа для всех элементов навигации. Каждый элемент в представлении навигации должен быть производным от этого класса.

#### Атрибуты:

* &#x20;**active **_**= False** -_ Указывает, представляет ли элемент текущий активный маршрут

#### Методы:

* &#x20;**render( **_**renderer=None**_**, **_**\*\*kwargs**_** )** - Визуализирует элемент навигации с помощью средства визуализации.
  * **параметр **_**renderer**_ - Объект, реализующий интерфейс **Renderer**.
  * **возвращает** - Безопасная для разметки строка с визуализированным результатом.

### &#x20;_class_ flask\_nav.elements.RawTag(_content_, _\*\*attribs_)

Элемент обычно выражается одним тегом HTML.

#### Параметры:

* _**title**_ - Текст внутри тега.
* _**attribs**_ - Атрибуты элемента.

### &#x20;_class_ flask\_nav.elements.Separator

Разделитель. Разделитель внутри главного навигационного меню или подгруппы **Subgroup**. Не все средства визуализации отображают их (или иногда только внутри подгрупп).

### &#x20;_class_ flask\_nav.elements.Subgroup(_title_, _\*items_)

Вложенная подструктура. Обычно используется для обозначения подменю.

#### Параметры:

* _**title**_ - Заголовок для отображения (т.е. при использовании раскрывающегося меню этот текст будет на кнопке).
* _**items**_ - Любое количество экземпляров **NavigationItem**, составляющих элемент навигации.

### &#x20;_class_ flask\_nav.elements.Text(_text_)

Текст ярлыка. Не текст `<label>`, но тем не менее текстовая метка. Точное представление зависит от средства визуализации, но, скорее всего, это что-то вроде `<span>`, `<div>`  или подобное.

### &#x20;_class_ flask\_nav.elements.View(_text_, _endpoint_, _\*\*kwargs_)

Внутренняя ссылка приложения. Конечная точка _**endpoint**_, _ **\*args** и **\*\*kwargs**_ передаются **url\_for ()** для получения ссылки.

#### Параметры:

* _**text**_ - Текст ссылки.
* _**endpoint**_ - Название представления.
* _**kwargs**_ - Дополнительные аргументы ключевого слова для **url\_for ().**

#### Методы:

* &#x20;**get\_url()** - Возвращает URL для этого элемента.
  * _**возвращает:**_ - Строка со ссылкой.

#### Атрибуты:

* &#x20;**ignore\_query **_**= True** -_ Следует ли учитывать аргументы запроса (`?Foo=bar&baz=1`) при определении, является ли представление **View** активным. По умолчанию аргументы запроса игнорируются. `"""`

### &#x20;_class_ flask\_nav.renderers.Renderer

Базовый интерфейс для средств визуализации навигации. Посещение узла должно возвращать строку или объект, который преобразуется в строку, содержащую HTML.

#### Методы:

* &#x20;**visit\_object(**_**node**_**)** - Резервный рендеринг для объектов. Если текущее приложение находится в режиме отладки (`flask.current_app.debug` имеет значение `True`), будет отображаться `<! - HTML-комментарий ->`, указывающий, в каком классе отсутствует функция посещения. Вне режима отладки возвращает пустую строку.

### &#x20;_class_ flask\_nav.renderers.SimpleRenderer(_\*\*kwargs_)

Очень простой рендерер HTML5. Визуализирует структуру навигации с помощью тегов `<nav>` и `<ul>`, которые можно стилизовать с помощью современного CSS.

#### Параметры:

* _**kwargs**_ - Дополнительные атрибуты для передачи корневому тегу `<nav>`.
