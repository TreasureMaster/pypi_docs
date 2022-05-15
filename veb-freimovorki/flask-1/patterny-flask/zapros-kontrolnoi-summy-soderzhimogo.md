# Запрос контрольной суммы содержимого

Различные фрагменты кода могут использовать данные запроса и предварительно обрабатывать их. Например, данные **JSON** попадают в уже прочитанный и обработанный объект запроса, данные формы также попадают туда, но проходят другой путь кода. Это кажется неудобным, если вы хотите вычислить контрольную сумму данных входящего запроса. Иногда это необходимо для некоторых API.

К счастью, это очень просто изменить, обернув входной поток.

В следующем примере вычисляется контрольная сумма **SHA1** входящих данных по мере их чтения и сохраняется в среде **WSGI**:

```python
import hashlib

class ChecksumCalcStream(object):

    def __init__(self, stream):
        self._stream = stream
        self._hash = hashlib.sha1()

    def read(self, bytes):
        rv = self._stream.read(bytes)
        self._hash.update(rv)
        return rv

    def readline(self, size_hint):
        rv = self._stream.readline(size_hint)
        self._hash.update(rv)
        return rv

def generate_checksum(request):
    env = request.environ
    stream = ChecksumCalcStream(env['wsgi.input'])
    env['wsgi.input'] = stream
    return stream._hash
```

Чтобы использовать это, все, что вам нужно сделать, это подключить вычислительный поток до того, как запрос начнет потреблять данные. (Например: будьте осторожны при доступе к `request.form` или чему-либо подобному. Например, `before_request_handlers` должны быть осторожны, чтобы не получить к нему доступ).

```python
@app.route('/special-api', methods=['POST'])
def special_api():
    hash = generate_checksum(request)
    # Доступ к этому анализирует входной поток
    files = request.files
    # На этом этапе хеш полностью построен.
    checksum = hash.hexdigest()
    return 'Hash was: %s' % checksum
```
