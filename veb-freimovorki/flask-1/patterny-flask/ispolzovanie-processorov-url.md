# Использование процессоров URL

_Новое в версии 0.7_.

Во `Flask 0.7` представлена концепция обработчиков URL. Идея состоит в том, что у вас может быть куча ресурсов с общими частями в URL, которые вы не всегда явно хотите предоставлять. Например, у вас может быть несколько URL-адресов с кодом языка, но вы не хотите, чтобы вам приходилось обрабатывать его в каждой отдельной функции самостоятельно.

Обработчики URL-адресов особенно полезны в сочетании с _**blueprints**_. Здесь мы будем обрабатывать как обработчики URL-адресов для конкретных приложений, так и особенности чертежей.

## Интернационализированные URL-адреса приложений

Рассмотрим такое приложение:

```python
from flask import Flask, g

app = Flask(__name__)

@app.route('/<lang_code>/')
def index(lang_code):
    g.lang_code = lang_code
    ...

@app.route('/<lang_code>/about')
def about(lang_code):
    g.lang_code = lang_code
    ...
```

Это очень много повторений, так как вам нужно самостоятельно обрабатывать настройку языкового кода для объекта [g](../api-dokumentaciya-flask/globalnyi-obekt-prilozheniya-flask.md#flask-g) в каждой отдельной функции. Конечно, для упрощения этого можно использовать декоратор, но если вы хотите сгенерировать URL-адреса от одной функции к другой, вам все равно придется явно указывать языковой код, что может раздражать.

Для последнего здесь на помощь приходят функции [url\_defaults ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#url\_defaults). Они могут автоматически вставлять значения в вызов [url\_for ()](../api-dokumentaciya-flask/poleznye-funkcii-i-klassy-flask.md#flask-url\_for). Приведенный ниже код проверяет, нет ли кода языка еще в словаре значений URL и требуется ли конечной точке значение с именем `'lang_code'`:

```python
@app.url_defaults
def add_language_code(endpoint, values):
    if 'lang_code' in values or not g.lang_code:
        return
    if app.url_map.is_endpoint_expecting(endpoint, 'lang_code'):
        values['lang_code'] = g.lang_code
```

Метод Werkzeug [is\_endpoint\_expecting ()](https://werkzeug.palletsprojects.com/en/1.0.x/routing/#werkzeug.routing.Map.is\_endpoint\_expecting) карты URL-адресов может использоваться, чтобы выяснить, имеет ли смысл предоставлять языковой код для данной конечной точки.

Обратной этой функции является [url\_value\_preprocessor ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#url\_value\_preprocessor). Они выполняются сразу после сопоставления запроса и могут выполнять код на основе значений URL. Идея состоит в том, что они извлекают информацию из словаря значений и помещают ее в другое место:

```python
@app.url_value_preprocessor
def pull_lang_code(endpoint, values):
    g.lang_code = values.pop('lang_code', None)
```

Таким образом, вам больше не нужно назначать _**lang\_code**_ для [g](../api-dokumentaciya-flask/globalnyi-obekt-prilozheniya-flask.md#flask-g) в каждой функции. Вы можете еще больше улучшить это, написав свой собственный декоратор, который добавляет к URL-адресам код языка, но более красивым решением является использование схемы _**blueprint**_. После того, как `'lang_code'` будет извлечен из словаря значений и больше не будет перенаправлен в функцию просмотра, уменьшив код до следующего:

```python
from flask import Flask, g

app = Flask(__name__)

@app.url_defaults
def add_language_code(endpoint, values):
    if 'lang_code' in values or not g.lang_code:
        return
    if app.url_map.is_endpoint_expecting(endpoint, 'lang_code'):
        values['lang_code'] = g.lang_code

@app.url_value_preprocessor
def pull_lang_code(endpoint, values):
    g.lang_code = values.pop('lang_code', None)

@app.route('/<lang_code>/')
def index():
    ...

@app.route('/<lang_code>/about')
def about():
    ...
```

## Интернационализированные URL-адреса Blueprint

Поскольку схемы _**blueprints**_ могут автоматически добавлять префикс всех URL-адресов с помощью общей строки, это легко сделать автоматически для каждой функции. Кроме того, в схемах _**blueprints**_ могут быть обработчики URL-адресов для каждого чертежа, что устраняет большую часть логики из функции [url\_defaults ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#url\_defaults), поскольку ей больше не нужно проверять, действительно ли URL-адрес интересует параметр `'lang_code'`:

```python
from flask import Blueprint, g

bp = Blueprint('frontend', __name__, url_prefix='/<lang_code>')

@bp.url_defaults
def add_language_code(endpoint, values):
    values.setdefault('lang_code', g.lang_code)

@bp.url_value_preprocessor
def pull_lang_code(endpoint, values):
    g.lang_code = values.pop('lang_code')

@bp.route('/')
def index():
    ...

@bp.route('/about')
def about():
    ...
```
