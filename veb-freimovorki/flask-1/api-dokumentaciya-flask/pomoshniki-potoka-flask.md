# Помощники потока Flask

### flask.stream\_with\_context( _generator\_or\_function_ )

_Новое в версии 0.9_.

Контексты запроса исчезают, когда ответ запускается на сервере. Это сделано из соображений эффективности и для уменьшения вероятности возникновения утечек памяти из-за плохо написанного промежуточного программного обеспечения **WSGI**. Обратной стороной является то, что при использовании потоковых ответов генератор больше не может получить доступ к информации, связанной с запросом.

Однако эта функция может помочь вам сохранить контекст дольше:

```python
from flask import stream_with_context, request, Response

@app.route('/stream')
def streamed_response():
    @stream_with_context
    def generate():
        yield 'Hello '
        yield request.args['name']
        yield '!'
    return Response(generate())
```

В качестве альтернативы его также можно использовать с определенным генератором:

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
