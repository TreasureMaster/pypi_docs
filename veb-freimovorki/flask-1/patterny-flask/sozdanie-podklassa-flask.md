# Создание подкласса Flask

Класс [Flask](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#klass-flask-flask) предназначен для создания подклассов.

Например, вы можете захотеть переопределить способ обработки параметров запроса, чтобы сохранить их порядок:

```python
from flask import Flask, Request
from werkzeug.datastructures import ImmutableOrderedMultiDict
class MyRequest(Request):
    """Подкласс Request для переопределения хранилища параметров запроса"""
    parameter_storage_class = ImmutableOrderedMultiDict
class MyFlask(Flask):
    """Подкласс Flask с использованием настраиваемого класса Request"""
    request_class = MyRequest
```

Это рекомендуемый подход для переопределения или расширения внутренней функциональности **Flask**.
