# Потоковое содержимое Flask

Иногда вы хотите отправить клиенту огромное количество данных, намного больше, чем вы хотите сохранить в памяти. Однако, когда вы генерируете данные «на лету», как вы отправляете их обратно клиенту без обращения к файловой системе?

Ответ заключается в использовании генераторов и прямых ответов.

## Базовое использование

Это базовая функция просмотра, которая на лету генерирует множество данных **CSV**. Уловка состоит в том, чтобы иметь внутреннюю функцию, которая использует генератор для генерации данных, а затем вызывает эту функцию и передает ее объекту ответа:

```python
from flask import Response

@app.route('/large.csv')
def generate_large_csv():
    def generate():
        for row in iter_all_rows():
            yield ','.join(row) + '\n'
    return Response(generate(), mimetype='text/csv')
```

Каждое выражение **yield** напрямую отправляется в браузер. Однако обратите внимание, что некоторые промежуточные программы **WSGI** могут нарушать потоковую передачу, поэтому будьте осторожны в средах отладки с профилировщиками и другими функциями, которые вы могли включить.

## Потоки из шаблонов

Шаблонизатор **Jinja2** также поддерживает рендеринг шаблонов по частям. **Flask** не предоставляет эту функциональность напрямую, потому что это довольно необычно, но вы легко можете сделать это самостоятельно:

```python
from flask import Response

def stream_template(template_name, **context):
    app.update_template_context(context)
    t = app.jinja_env.get_template(template_name)
    rv = t.stream(context)
    rv.enable_buffering(5)
    return rv

@app.route('/my-large-page.html')
def render_large_template():
    rows = iter_all_rows()
    return Response(stream_template('the_template.html', rows=rows))
```

Уловка здесь заключается в том, чтобы получить объект шаблона из среды **Jinja2** в приложении и вызвать [stream ()](https://jinja.palletsprojects.com/en/2.11.x/api/#jinja2.Template.stream) вместо [render ()](https://jinja.palletsprojects.com/en/2.11.x/api/#jinja2.Template.render), который возвращает объект потока вместо строки. Поскольку мы обходим функции рендеринга шаблона **Flask** и используем сам объект шаблона, мы должны сами обновить контекст рендеринга, вызвав [update\_template\_context ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#update\_template\_context). Затем шаблон оценивается по мере повторения потока. Так как каждый раз, когда вы выполняете **yield**, сервер будет сбрасывать контент клиенту, вы можете захотеть буферизовать несколько элементов в шаблоне, что вы можете сделать с помощью `rv.enable_buffering (size)`. **5** - это нормальное значение по умолчанию.

## Потоки с контекстом

_Новое в версии 0.9_.

Обратите внимание, что при потоковой передаче данных контекст запроса уже отсутствует в момент выполнения функции. `Flask 0.9` предоставляет вам помощника, который может поддерживать контекст запроса во время выполнения генератора:

```python
from flask import stream_with_context, request, Response

@app.route('/stream')
def streamed_response():
    def generate():
        yield 'Hello '
        yield request.args['name']
        yield '!'
    return Response(stream_with_context(generate()))
```

Без функции [stream\_with\_context ()](../api-dokumentaciya-flask/pomoshniki-potoka-flask.md#flask-stream\_with\_context) в этот момент вы получите [RuntimeError](https://docs.python.org/3/library/exceptions.html#RuntimeError).
