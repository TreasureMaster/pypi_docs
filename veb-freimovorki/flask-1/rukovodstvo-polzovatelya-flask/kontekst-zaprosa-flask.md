# Контекст запроса Flask

Контекст запроса отслеживает данные уровня запроса во время запроса. Вместо того, чтобы передавать объект запроса каждой функции, которая выполняется во время запроса, вместо этого осуществляется доступ к прокси-серверам запроса [request](../api-dokumentaciya-flask/dannye-vkhodyashego-zaprosa-request.md#flask-request) и сеанса [session](../api-dokumentaciya-flask/sessii-flask.md#klass-flask-session).

Это похоже на [контекст приложения Application](kontekst-prilozheniya-flask.md), который отслеживает данные уровня приложения независимо от запроса. Соответствующий контекст приложения передается при передаче контекста запроса.

## Цель контекста

Когда приложение [Flask](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#klass-flask-flask) обрабатывает запрос, оно создает объект [Request](../api-dokumentaciya-flask/dannye-vkhodyashego-zaprosa-request.md#klass-flask-request) на основе среды, полученной от сервера **WSGI**. Поскольку рабочий процесс _**worker**_ (поток, процесс или сопрограмма в зависимости от сервера) обрабатывает только один запрос за раз, данные запроса могут считаться глобальными для этого _**worker**_ во время этого запроса. **Flask** использует для этого термин _**local context**_ ("локальный контекст").

**Flask** автоматически **подталкивает** контекст запроса при обработке запроса. Функции просмотра, обработчики ошибок и другие функции, которые выполняются во время запроса, будут иметь доступ к прокси-серверу [request](../api-dokumentaciya-flask/dannye-vkhodyashego-zaprosa-request.md#flask-request), который указывает на объект запроса для текущего запроса.

## Время жизни контекста

Когда приложение **Flask** начинает обрабатывать запрос, оно подталкивает контекст запроса, который также подталкивает [контекст приложения](kontekst-prilozheniya-flask.md). Когда запрос завершается, появляется контекст запроса, а затем контекст приложения.

Контекст уникален для каждого потока (или другого типа _**worker**_). [request](../api-dokumentaciya-flask/dannye-vkhodyashego-zaprosa-request.md#flask-request) не может быть передан другому потоку, другой поток будет иметь другой стек контекста и не будет знать о запросе, на который указывал родительский поток.

Локальные переменные контекста реализованы в **Werkzeug**. См. «[Локальные переменные контекста](https://werkzeug.palletsprojects.com/en/1.0.x/local/)» для получения дополнительной информации о том, как это работает внутри.

## Вставить контекст вручную

Если вы попытаетесь получить доступ к запросу [request](../api-dokumentaciya-flask/dannye-vkhodyashego-zaprosa-request.md#flask-request) или чему-либо, что его использует вне контекста запроса, вы получите следующее сообщение об ошибке:

```bash
RuntimeError: Working outside of request context.

Обычно это означает, что вы пытались использовать функции, которым
нужен активный HTTP-запрос. Обратитесь к документации по тестированию
для получения информации о том, как избежать этой проблемы.
```

Обычно это должно происходить только при тестировании кода, ожидающего активного запроса. Один из вариантов - использовать [тестовый клиент](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#test\_client) для имитации полного запроса. Или вы можете использовать [test\_request\_context ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#test\_request\_context) в блоке **with**, и все, что выполняется в блоке, будет иметь доступ к запросу [request](../api-dokumentaciya-flask/dannye-vkhodyashego-zaprosa-request.md#flask-request), заполненному вашими тестовыми данными.

```python
def generate_report(year):
    format = request.args.get('format')
    ...

with app.test_request_context(
        '/make_report/2017', data={'format': 'short'}):
    generate_report()
```

Если вы видите эту ошибку в другом месте вашего кода, не связанном с тестированием, это, скорее всего, указывает на то, что вам следует переместить этот код в функцию просмотра.

Для получения информации о том, как использовать контекст запроса из интерактивной оболочки Python, см. [Работа с оболочкой](rabota-flask-s-obolochkoi.md).

## Как работает контекст

Для обработки каждого запроса вызывается метод [Flask.wsgi\_app ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#wsgi\_app). Он управляет контекстами во время запроса. Внутри контексты запроса и приложения работают как стеки, [\_request\_ctx\_stack](../api-dokumentaciya-flask/poleznye-funkcii-flask.md#flask-\_request\_ctx\_stack) и [\_app\_ctx\_stack](../api-dokumentaciya-flask/poleznye-funkcii-flask.md#flask-\_app\_ctx\_stack). Когда контексты помещаются в стек, становятся доступными прокси-серверы, которые от них зависят, и указывают на информацию из верхнего контекста стека.

Когда начинается запрос, создается и отправляется [RequestContext](../api-dokumentaciya-flask/poleznye-funkcii-flask.md#klass-flask-ctx-requestcontext), который сначала создает и подталкивает [AppContext](../api-dokumentaciya-flask/poleznye-funkcii-flask.md#klass-flask-ctx-appcontext), если контекст для этого приложения еще не является верхним контекстом. Пока эти контексты передаются, прокси [current\_app](../api-dokumentaciya-flask/poleznye-funkcii-i-klassy-flask.md#flask-current\_app), [g](../api-dokumentaciya-flask/globalnyi-obekt-prilozheniya-flask.md#flask-g), [request](../api-dokumentaciya-flask/dannye-vkhodyashego-zaprosa-request.md#flask-request) и [session](../api-dokumentaciya-flask/sessii-flask.md#klass-flask-session) доступны для исходного потока, обрабатывающего запрос.

Поскольку контексты представляют собой стеки, другие контексты могут быть переданы для изменения прокси во время запроса. Хотя это не распространенный шаблон, его можно использовать в расширенных приложениях, например, для выполнения внутренних перенаправлений или связывания различных приложений вместе.

После отправки запроса и генерации и отправки ответа контекст запроса извлекается, а затем всплывает контекст приложения. Непосредственно перед их открытием выполняются функции [teardown\_request ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#teardown\_request) и [teardown\_appcontext ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#teardown\_appcontext). Они выполняются, даже если во время отправки возникло необработанное исключение.

## Обратные вызовы и ошибки

**Flask** отправляет запрос в несколько этапов, что может повлиять на запрос, ответ и способ обработки ошибок. Контексты активны на всех этих этапах.

[Blueprint](../api-dokumentaciya-flask/obekty-blueprint.md#klass-flask-blueprint) может добавлять обработчики для этих событий, специфичных для этого blueprint'а. Обработчики схемы будут запущены, если схеме принадлежит маршрут, соответствующий запросу.

1. Перед каждым запросом вызываются функции [before\_request ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#before\_request). Если одна из этих функций возвращает значение, другие функции пропускаются. Возвращаемое значение рассматривается как ответ, а функция просмотра не вызывается.
2. Если функция [before\_request ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#before\_request) не вернула ответ, вызывается функция просмотра для согласованного маршрута и возвращает ответ.
3. Возвращаемое значение представления преобразуется в фактический объект ответа и передается функциям [after\_request ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#after\_request). Каждая функция возвращает измененный или новый объект ответа.
4. После возврата ответа контексты всплывают, что вызывает функции [teardown\_request ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#teardown\_request) и [teardown\_appcontext ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#teardown\_appcontext). Эти функции вызываются, даже если необработанное исключение было вызвано в любой точке выше.

Если исключение возникает перед функциями удаления, **Flask** пытается сопоставить его с функцией [errorhandler ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#errorhandler) для обработки исключения и возврата ответа. Если обработчик ошибок не найден или сам обработчик вызывает исключение, **Flask** возвращает общий ответ `500 Internal Server Error`. Функции **teardown** по-прежнему вызываются, и им передается объект исключения.

Если включен режим отладки, необработанные исключения не преобразуются в ответ `500`, а вместо этого передаются на сервер **WSGI**. Это позволяет серверу разработки представить интерактивный отладчик с трассировкой.

### Обратные вызовы при teardown

Обратные вызовы **teardown** не зависят от отправки запроса и вместо этого вызываются контекстами, когда они выталкиваются. Функции вызываются, даже если во время отправки возникает необработанное исключение, а также для контекстов, помещенных вручную. Это означает, что нет гарантии, что какие-либо другие части отправки запроса будут выполнены раньше. Обязательно пишите эти функции таким образом, чтобы они не зависели от других обратных вызовов и не потерпели неудачу.

Во время тестирования может быть полезно отложить извлечение контекстов после завершения запроса, чтобы их данные могли быть доступны в тестовой функции. Используйте [test\_client ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#test\_client) как блок **with**, чтобы сохранить контексты до выхода из блока **with**.

```python
from flask import Flask, request

app = Flask(__name__)

@app.route('/')
def hello():
    print('during view')
    return 'Hello, World!'

@app.teardown_request
def show_teardown(exception):
    print('after with block')

with app.test_request_context():
    print('during with block')

# функции teardown вызываются после контекста с выходами из блока

with app.test_client() as client:
    client.get('/')
    # контексты не всплывают, даже если запрос завершился
    print(request.path)

# контексты всплывают, и функции разрыва вызываются после того,
# как клиент выходит из блока
```

### Сигналы

Если [signal\_available](../api-dokumentaciya-flask/signaly-flask.md#signals-signals\_available) истинно, отправляются следующие сигналы:

1. [request\_started](../api-dokumentaciya-flask/signaly-flask.md#flask-request\_started) отправляется до вызова функций [before\_request ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#before\_request).
2. [request\_finished](../api-dokumentaciya-flask/signaly-flask.md#flask-request\_finished) отправляется после вызова функций [after\_request ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#after\_request).
3. [got\_request\_exception](../api-dokumentaciya-flask/signaly-flask.md#flask-got\_request\_exception) отправляется, когда начинается обработка исключения, но до поиска или вызова [errorhandler ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#errorhandler).
4. [request\_tearing\_down](../api-dokumentaciya-flask/signaly-flask.md#flask-request\_tearing\_down) отправляется после вызова функции [teardown\_request ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#teardown\_request).

## Сохранение контекста при ошибке

В конце запроса контекст запроса выталкивается, и все данные, связанные с ним, уничтожаются. Если во время разработки возникает ошибка, полезно отложить уничтожение данных для целей отладки.

Когда сервер разработки работает в режиме **development** (для переменной среды **FLASK\_ENV** установлено значение `'development'`), ошибка и данные сохраняются и отображаются в интерактивном отладчике.

Этим поведением можно управлять с помощью конфигурации [PRESERVE\_CONTEXT\_ON\_EXCEPTION](obrabotka-konfiguracii-flask.md#preserve\_context\_on\_exception). Как описано выше, в среде разработки по умолчанию используется значение `True`.

Не включайте **PRESERVE\_CONTEXT\_ON\_EXCEPTION** в производственной среде, так как это приведет к утечке памяти в вашем приложении при исключениях.

## Примечания к прокси

Некоторые из объектов, предоставляемых **Flask**, являются прокси для других объектов. Доступ к прокси осуществляется одинаково для каждого рабочего потока, но они указывают на уникальный объект, связанный с каждым **worker** за кулисами, как описано на этой странице.

В большинстве случаев вам не нужно об этом заботиться, но есть некоторые исключения, когда полезно знать, что этот объект на самом деле является прокси:

* Прокси-объекты не могут подделывать свой тип как фактические типы объектов. Если вы хотите выполнить проверку экземпляра, вы должны сделать это на проксируемом объекте.
* Ссылка на проксируемый объект необходима в некоторых ситуациях, например при отправке [сигналов](signaly-flask.md) или данных в фоновый поток.

Если вам нужно получить доступ к базовому объекту, который проксируется, используйте метод Werkzeug [\_get\_current\_object ()](https://werkzeug.palletsprojects.com/en/1.0.x/local/#werkzeug.local.LocalProxy.\_get\_current\_object):

```python
app = current_app._get_current_object()
my_signal.send(app)
```
