# Обратные вызовы отложенного запроса

Один из принципов проектирования **Flask** заключается в том, что объекты ответа создаются и передаются по цепочке потенциальных обратных вызовов, которые могут их изменять или заменять. Когда начинается обработка запроса, объекта ответа еще нет. Он создается по мере необходимости либо функцией просмотра, либо каким-либо другим компонентом в системе.

Что произойдет, если вы захотите изменить ответ в точке, где ответ еще не существует? Типичным примером этого может быть обратный вызов [before\_request ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#before\_request), который хочет установить файл **cookie** для объекта ответа.

Один из способов - избежать этой ситуации. Очень часто это возможно. Например, вы можете попробовать переместить эту логику в обратный вызов [after\_request ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#after\_request). Однако иногда перенос кода туда делает его более сложным или неудобным для размышлений.

В качестве альтернативы вы можете использовать [after\_this\_request ()](../api-dokumentaciya-flask/poleznye-funkcii-i-klassy-flask.md#flask-after\_this\_request) для регистрации обратных вызовов, которые будут выполняться только после текущего запроса. Таким образом, вы можете отложить выполнение кода из любого места приложения на основе текущего запроса.

В любой момент во время запроса мы можем зарегистрировать функцию, которая будет вызываться в конце запроса. Например, вы можете запомнить текущий язык пользователя в файле **cookie** с помощью обратного вызова [before\_request ()](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#before\_request):

```python
from flask import request, after_this_request

@app.before_request
def detect_user_language():
    language = request.cookies.get('user_lang')

    if language is None:
        language = guess_language_from_request()

        # когда есть ответ, установите cookie с языком
        @after_this_request
        def remember_language(response):
            response.set_cookie('user_lang', language)
            return response

    g.language = language
```
