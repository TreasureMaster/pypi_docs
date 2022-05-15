# API

Здесь будет краткое описание и оглавление по API WTForms.

## [Forms (формы)](forms-formy.md)

### класс **wtforms.form.Form** - базовый класс формы

* [Параметры конструктора **Form**](forms-formy.md#konstruktor)****
  * _**formdata**_ - принятые данные HTML формы
  * _**obj**_ - объект для заполнения полей формы, если _**formdata**_ отсутствует
  * _**prefix**_ - префикс для полей формы
  * _**data**_ - словарь данных для заполнения формы, если _**formdata**_ и _**obj**_ отсутствуют
  * _**meta**_ - словарь значений для переопределения атрибутов в мета-экземпляре формы
  * _**\*\*kwargs**_ - ключевые слова используются, если _**formdata**_ пуста, а в объекте _**obj**_ нет данных
* [Свойства **Form**](forms-formy.md#svoistva)****
  * _**data**_ - словарь с данными каждого поля
  * _**errors**_ - словарь с ошибками для каждого поля
  * _**meta**_ - объект с различными параметрами конфигурации для настройки формы
* [Методы **Form**](forms-formy.md#metody)****
  * _**validate()**_ - проверяет форму, вызывая _**validate**_ для каждого поля и возвращает True/False&#x20;
  * _**populate\_obj( obj )**_ - заполняет атрибуты переданного объекта _**obj**_ данными формы
  * _**\_\_iter\_\_()**_ - итерирует поля формы в порядке создания (н-р, в цикле)
  * _**\_\_contains\_\_(name)**_ - возвращает True, если поле _**name**_ является членом этой формы
  * _**\_get\_translations()**_ - возвращает объект с методами _**gettext**_ и _**ngettext**_ (устарело)

### класс wtforms.form.BaseForm - базовый низкоуровневый класс формы

* [Параметры конструктора **BaseForm**](forms-formy.md#konstruktor-1)****
  * _**fields**_ - словарь или кортеж частично построенных полей
  * _**prefix**_ - префикс значений полей
  * _**meta**_ - мета-экземпляр для конфигурации и настройки **WTForms**
* [Свойства **BaseForm**](forms-formy.md#svoistva-1)****
  * _**data**_ - см. **Form.data**
  * _**errors**_ - см. **Form.errors**
* [Методы **BaseForm**](forms-formy.md#metody-1)****
  * _**process()**_ - обрабатывает форму и данные объекта, аргументы _**kwargs**_ для каждого поля
  * _**validate()**_ - проверяет форму, вызывая _**validate**_ для каждого поля и возвращает True/False
  * _**\_\_iter\_\_()**_ -&#x20;
  * _**\_\_contains\_\_()**_ -&#x20;
  * _**\_\_getitem\_\_()**_ -&#x20;
  * _**\_\_setitem\_\_()**_ -&#x20;
  * _**\_\_delitem\_\_()**_ -&#x20;

## [Fields (поля)](fields-polya.md)

### класс wtforms.fields.Field - базовый класс поля

* [Параметры конструктора **Field**](fields-polya.md#konstruktor)****
  * _**label**_ - метка `<label>` поля
  * _**validators**_ - список валидаторов поля, вызываемых _**validate()**_
  * _**filters**_ - последовательность фильтров для входных данных, вызываемых _**process()**_
  * _**description**_ - описание поля (обычно для текста справки)
  * _**id**_ - HTML идентификатор поля `<input>` ; задается формой
  * _**default**_ - значение по умолчанию, назначаемое полю
  * _**widget**_ - виджет визуализации поля
  * _**render\_kw(dict)**_ - словарь _**dict**_ атрибутов поля `<input>`
  * _**\_form**_ - форма, содержащая это поле
  * _**\_name**_ - имя этого поля во внешней форме
  * _**\_prefix**_ - префикс имени формы
  * _**\_translations**_ - объект переводов сообщений
  * _**\_meta**_ - экземпляр **meta** из формы
* [Проверка поля **Field**](fields-polya.md#proverka)****
  * _**validate()**_ - проверяет поле и возвращает True или False
  * _**pre\_validate()**_ - проверка, выполняемая раньше других валидаторов
  * _**post\_validate()**_ - проверка, выполняемая после всех валидаторов
  * _**errors**_ - список ошибок поля
* [Доступ к данным поля **Field** и их обработка](fields-polya.md#dostup-k-dannym-i-ikh-obrabotka)
  * _**process()**_ - обработка входных данных (_**process\_data**_, _**process\_formdata**_, фильтры)
  * _**process\_data()**_ - обрабатывает данные Python, примененные к этому полю
  * _**process\_formdata()**_ - обработка данных, полученных из HTML-формы
  * _**data**_ - результирующее значение вызова любого из методов **process\_\*\***
  * _**raw\_data**_ - значения, полученные из оболочки _**formdata**_
  * _**object\_data**_ - данные, переданные из _**obj**_ или _**kwargs**_ в исходном виде
* [Рендеринг поля **Field**](fields-polya.md#rendering)****
  * _**\_\_call\_\_()**_ - отображает поле как HTML, делегируя рендеринг **meta.render\_field()**
  * _**\_\_html\_\_()**_ - возвращает HTML представление поля
* [Переводы сообщений поля](fields-polya.md#perevody-soobshenii)
  * _**gettext()**_ - получает перевод данного сообщения
  * _**ngettext()**_ - получает перевод сообщений из множественной формы
* [Свойства **Field**](fields-polya.md#svoistva)****
  * _**name**_ - имя HTML-вида этого поля с префиксом, если он определен
  * _**short\_name**_ - имя этого поля без префикса
  * _**id**_ - HTML-идентификатор поля (по умолчанию совпадает с именем поля)
  * _**label**_ - экземпляр метки поля **Label**, возвращающий `<label>`
  * _**default**_ - то, что передано конструктору поля по умолчанию (или None)
  * _**description**_ - описание, переданное конструкторы поля
  * _**errors**_ - список ошибок этого поля
  * _**process\_errors**_ - ошибки, полученные при обработке ввода
  * _**widget**_ - виджет, используемый для визуализации поля
  * _**type**_ - тип поля в виде строки
  * _**flags**_ - объект, содержащий логические флаги, установленные полем или валидатором
  * _**meta**_ - экземпляр метаобъекта, **Form.meta**
  * _**filters**_ - список фильтров, переданных конструкторы поля
* [Базовые поля **fields**](fields-polya.md#osnovnye-polya)****
  * **BooleanField** - представляет собой поле `<input type="checkbox">`
  * **DateField** - текстовое поле, хранящее `datetime.date`
  * **DateTimeField** - текстовое поле, хранящее `datetime.datetime`
  * **DecimalField** - текстовое поле, хранящее данные `decimal.Decimal`
  * **FileField** - отображает поле загрузки файла
  * **MultipleFileField** - **FileField**, который позволяет выбирать несколько файлов
  * **FloatField** - текстовое поле с данными типов с плавающей точкой
  * **IntegerField** - текстовое поле с целочисленными данными
  * **RadioField** - переключатели типа `<input type="radio">`
  * **SelectField** - поля выбора `<select>`
  * **SelectMultipleField** - **SelectField** с возможностью выбора нескольких вариантов
  * **SubmitField** - кнопка отправки формы `<input type="submit">`
  * **StringField** - базовое поле для многих остальных полей вида `<input type="text">`
* [Удобные поля](fields-polya.md#polya-dlya-udobstva)
  * **HiddenField** - скрытое поле **StringField**, отображаемое как `<input type="hidden">`
  * **PasswordField** - поле **StringField** для ввода пароля вида `<input type="password">`
  * **TextAreaField** - поле для ввода многострочного текста вида `<textarea>`
* [Вложенные поля](fields-polya.md#vlozhennye-polya)
  * **FormField** - представляет форму как поле в другой форме
  * **FieldList** - инкапсулирует несколько экземпляров полей в виде списка
* [Дополнительные классы](fields-polya.md#dopolnitelnye-klassy-pomoshi)
  * Flags - набор логических флагов в качестве атрибутов
  * Label - класс для свойства полей `<label>`
* [Поля HTML5](fields-polya.md#polya-html5)

## [Validators (валидаторы)](validators-validatory.md)

* [Исключения **validators**](validators-validatory.md#isklyucheniya-validators)****
  * **ValidationError** - возникает, когда валидатор не может проверить свой ввод
  * **StopValidation** - останавливает цепочку проверки
* [Встроенные валидаторы](validators-validatory.md#vstroennye-validatory)
  * ****[**DataRequired**](validators-validatory.md#klass-validators-datarequired) - проверяет, является ли атрибут данных поля истинным True
  * ****[**Email**](validators-validatory.md#klass-validators-email) - проверяет адрес электронной почты
  * ****[**EqualTo**](validators-validatory.md#klass-validators-equalto) - сравнивает значения двух полей
  * ****[**InputRequired**](validators-validatory.md#klass-validators-inputrequired) - проверяет, чтобы ввод был предоставлен для данного поля
  * ****[**IPAddress**](validators-validatory.md#klass-validators-ipaddress) - проверяет IP-адрес
  * ****[**Length**](validators-validatory.md#klass-validators-length) - проверяет длину строки
  * ****[**MacAddress**](validators-validatory.md#klass-validators-macaddress) - проверяет MAC-адрес
  * ****[**NumberRange**](validators-validatory.md#klass-validators-numberrange) проверяет попадание числа в заданный диапазон
  * ****[**Optional**](validators-validatory.md#klass-validators-optional) - разрешает пустой ввод, останавливает продолжение цепочки проверки
  * ****[**Regexp**](validators-validatory.md#klass-validators-regexp) - проверяет поле на соответствие регулярному выражению
  * ****[**URL**](validators-validatory.md#klass-validators-url) - проверка адреса URL
  * ****[**UUID**](validators-validatory.md#klass-validators-uuid) - проверяет UUID
  * ****[**AnyOf**](validators-validatory.md#klass-validators-anyof) - сравнивает входные данные с последовательностью допустимых данных
  * ****[**NoneOf**](validators-validatory.md#klass-validators-noneof) - сравнивает входные данные с последовательностью недопустимых данных
* [Пользовательские валидаторы](validators-validatory.md#polzovatelskie-validatory)
* [Установка флагов на поле](validators-validatory.md#ustanovka-flagov-na-pole-s-validatorami)

## [Widgets (виджеты)](widgets-vidzhety.md)

* [Встроенные виджеты](widgets-vidzhety.md#vstroennye-vidzhety)
  * **ListWidget** - отображает список полей в виде списка `<ol>` или `<ul>`
  * **TableWidget** - отображает список полей в виде строк таблицы `<th>` и `<td>`
  * **Input** - отображает базовое поле `<input>`
  * **TextInput** - отображает ввод текста в одну строку
  * **PasswordInput** - отображает ввод пароля
  * **HiddenInput** - отображает скрытый ввод
  * **CheckboxInput** - отображает флажки checkbox
  * **FileInput** - отображает ввод средства выбора файла
  * **SubmitInput** - отображает кнопку отправки
  * **TextArea** - отображает многострочную текстовую область
  * **Select** - отображает поле выбора `<select>`
* [HTML5 виджеты](widgets-vidzhety.md#html5-vidzhety)
* [Утилиты для создания виджетов](widgets-vidzhety.md#utility-dlya-sozdaniya-vidzhetov)
* [Пользовательские виджеты](widgets-vidzhety.md#polzovatelskie-vidzhety)
