# Тестовые Client и CLI Runner

## Тестовый клиент

### Класс flask.testing.FlaskClient( _\*args_, _\*\*kwargs_ )

Работает как обычный тестовый клиент **Werkzeug**, но имеет некоторые знания о том, как работает **Flask**, чтобы отложить очистку стека контекста запроса до конца тела **with** при использовании в операторе **with**. Для получения общей информации о том, как использовать этот класс, обратитесь к [werkzeug.test.Client](https://werkzeug.palletsprojects.com/en/1.0.x/test/#werkzeug.test.Client).

_Изменено в версии 0.12:_ **app.test\_client ()** включает предустановленную среду по умолчанию, которая может быть установлена после создания экземпляра объекта `app.test_client ()` в `client.environ_base`.

Основное использование описано в главе «[Тестирование приложений Flask](../rukovodstvo-polzovatelya-flask/testirovanie-prilozhenii-flask.md)».

#### open( _\*args_, _\*\*kwargs_ )

Генерирует окружение из заданных аргументов, делает запрос к приложению, используя его, и возвращает ответ.

**Параметры:**

* **args** - Передается в **EnvironBuilder** для создания среды для запроса. Если передан единственный аргумент, это может быть существующий **EnvironBuilder** или словарь окружения.
* **buffered** - Преобразует итератор, возвращаемый приложением, в список. Если итератор имеет метод **close ()**, он вызывается автоматически.
* **follow\_redirects** - Делает дополнительные запросы для отслеживания перенаправлений HTTP, пока не будет возвращен статус отсутствия перенаправления. **TestResponse.history** перечисляет промежуточные ответы.

_Изменено в версии 2.0.0:_ **as\_tuple** устарел и будет удален в версии 2.1. Вместо этого используйте **TestResponse.request** и **request.environ**.

_Изменено в версии 0.5:_ если dict предоставляется как файл в dict для параметра **data**, тип содержимого должен называться **content\_type** вместо **mimetype**. Это изменение было внесено для согласованности с **werkzeug.FileWrapper**.

_Изменено в версии 0.5:_ Добавлен параметр **follow\_redirects**.

#### session\_transaction( _\*args_, _\*\*kwargs_ )

При использовании в сочетании с оператором **with** открывает транзакцию сеанса. Это можно использовать для изменения сеанса, который использует тестовый клиент. После выхода из блока **with** сеанс сохраняется.

```python
with client.session_transaction() as session:
    session['value'] = 42
```

Внутренне это реализуется путем прохождения временного контекста тестового запроса, и поскольку обработка сеанса может зависеть от переменных запроса, эта функция принимает те же аргументы, что и [test\_request\_context ()](obekt-prilozheniya-flask.md#test\_request\_context-args-kwargs), которые передаются напрямую.

## Тестирование CLI Runner

### Класс flask.testing.FlaskCliRunner( _app_, _\*\*kwargs_ )

[CliRunner](https://click.palletsprojects.com/en/7.x/api/#click.testing.CliRunner) для тестирования команд интерфейса командной строки приложения **Flask**. Обычно создается с помощью [test\_cli\_runner ()](obekt-prilozheniya-flask.md#test\_cli\_runner-kwargs). См. раздел «[Тестирование команд интерфейса командной строки](../rukovodstvo-polzovatelya-flask/testirovanie-prilozhenii-flask.md#testirovanie-cli-komand)».

#### invoke( _cli=None_, _args=None_, _\*\*kwargs_ )

Вызывает команду **CLI** в изолированной среде. См. [CliRunner.invoke](https://click.palletsprojects.com/en/7.x/api/#click.testing.CliRunner.invoke) для полной документации по методу. См. примеры в разделе «[Тестирование команд интерфейса командной строки](../rukovodstvo-polzovatelya-flask/testirovanie-prilozhenii-flask.md#testirovanie-cli-komand)».

Если аргумент _**obj**_ не указан, передает экземпляр [ScriptInfo](interfeis-komandnoi-stroki-flask.md#klass-flask-cli-scriptinfo), который знает, как загрузить тестируемое приложение **Flask**.

**Параметры:**

* **cli** - Вызываемый объект команды. По умолчанию это группа **cli** приложения.
* **args** - Список строк для вызова команды.

**Возвращает:**

* объект [Result](https://click.palletsprojects.com/en/7.x/api/#click.testing.Result) из **Click**
