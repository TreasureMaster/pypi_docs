# Одностраничные приложения Flask

**Flask** можно использовать для обслуживания одностраничных приложений (**SPA**), помещая статические файлы, созданные вашей интерфейсной платформой, во вложенную папку внутри вашего проекта. Вам также потребуется создать конечную точку для приема всей почты домена, которая направляет все запросы в ваш **SPA**.

В следующем примере показано, как обслуживать **SPA** вместе с API:

```python
from flask import Flask, jsonify

app = Flask(__name__, static_folder='app', static_url_path="/app")


@app.route("/heartbeat")
def heartbeat():
    return jsonify({"status": "healthy"})


@app.route('/', defaults={'path': ''})
@app.route('/<path:path>')
def catch_all(path):
    return app.send_static_file("index.html")
```
