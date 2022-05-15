# Поддержка CDN

**Flask-Bootstrap** поддерживает доставку через CDN или локальные ресурсы, настраиваемые во время выполнения. После инициализации **Flask-Bootstrap** сохранит в вашем приложении словарь с именем `yourapp.extensions ['bootstrap'] ['cdns']`, который сопоставляет имена с экземплярами объекта **CDN**.

Вы также можете использовать **bootstrap\_find\_resource ()** в своих шаблонах при использовании других ресурсов, которые могут быть доступны в CDN. CDN могут быть добавлены путем добавления новых записей в упомянутый выше словарь.

### &#x20;_class_ flask\_bootstrap.CDN

Базовый класс для объектов CDN.

#### Методы flask\_bootstrap.CDN:

#### get\_resource\_url(_filename_)

Возвращает URL ресурса для имени файла.

### &#x20;_class_ flask\_bootstrap.StaticCDN(_static\_endpoint='static'_, _rev=False_)

CDN, который обслуживает контент из локального приложения.

#### Параметры:

* _**static\_endpoint**_ - Конечная точка (endpoint) для использования.
* _**rev**_ - Если `True`, выполняется **BOOTSTRAP\_QUERYSTRING\_REVVING**.

### &#x20;_class_ flask\_bootstrap.WebCDN( _baseurl_ )

Обслуживает файлы из Интернета.

#### Параметры:

* _**baseurl**_ - Базовый **url**. Имена файлов просто добавляются к этому URL-адресу.

### &#x20;flask\_bootstrap.bootstrap\_find\_resource(_filename_, _cdn_, _use\_minified=None_, _local=True_)

Функция поиска ресурсов, также доступна в шаблонах.

Пытается найти ресурс, принудительно использует SSL в зависимости от настроек **BOOTSTRAP\_CDN\_FORCE\_SSL**.

#### Параметры:

* _**filename**_ - Файл, для которого нужно найти URL.
* _**cdn**_ - Имя используемой CDN.
* _**use\_minified**_ - Если установлено значение `True`/`False`, использовать/не использовать минимизированный. Если `None`, учитывается `BOOTSTRAP_USE_MINIFIED`.
* _**local**_ - Если `True`, использует `local-CDN`, когда включен **BOOTSTRAP\_SERVE\_LOCAL**. Если `False`, вместо этого используется **static-CDN**.

#### Возвращает:

* _**URL**_
