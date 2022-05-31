# Flask-ApiExceptions

**Flask-ApiExceptions** — это расширение Flask, предоставляющее базовые функции для сериализации неперехваченных исключений в виде ответов HTTP для REST API на основе JSON.

## Установка

Вы можете установить это расширение с помощью **pip**:

```bash
$ pip install flask_apiexceptions
```

Или вы можете клонировать репозиторий:

```bash
$ git clone https://github.com/jperras/flask_apiexceptions.git
```

## Запуск тестов

[Tox](https://pypi.python.org/pypi/tox) используется для запуска тестов, которые написаны с использованием [PyTest](https://docs.pytest.org/en/latest/). Чтобы запустить их, клонируйте репозиторий (указанный выше), убедитесь, что **tox** установлен и доступен, и запустите:

```bash
$ cd path/to/flask_apiexceptions
$ tox
```

## Использование

### Обработка пользовательского класса исключений

## Использование объектов ApiException и ApiError
