# Загрузка файлов и Flask

Ах да, старая добрая проблема с загрузкой файлов. Основная идея загрузки файлов на самом деле довольно проста. В основном это работает так:

1. Тег `<form>` помечается как `enctype=multipart/form-data`, и `<input type=file>` помещается в эту форму.
2. Приложение обращается к файлу из словаря **files** в объекте запроса.
3. используйте метод [save ()](https://werkzeug.palletsprojects.com/en/1.0.x/datastructures/#werkzeug.datastructures.FileStorage.save) файла, чтобы навсегда сохранить файл где-нибудь в файловой системе.

## Мягкое введение

Давайте начнем с очень простого приложения, которое загружает файл в определенную папку загрузки и отображает файл пользователю. Давайте посмотрим на код начальной загрузки для нашего приложения:

```python
import os
from flask import Flask, flash, request, redirect, url_for
from werkzeug.utils import secure_filename

UPLOAD_FOLDER = '/path/to/the/uploads'
ALLOWED_EXTENSIONS = {'txt', 'pdf', 'png', 'jpg', 'jpeg', 'gif'}

app = Flask(__name__)
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER
```

Итак, сначала нам нужна пара импорта. Большинство из них должно быть простым, **werkzeug.secure\_filename ()** объясняется немного позже. **UPLOAD\_FOLDER** - это место, где мы будем хранить загруженные файлы, а **ALLOWED\_EXTENSIONS** - это набор разрешенных расширений файлов.

Почему мы ограничиваем разрешенные расширения? Вероятно, вы не хотите, чтобы ваши пользователи могли загружать туда все, если сервер напрямую отправляет данные клиенту. Таким образом вы можете убедиться, что пользователи не могут загружать файлы HTML, которые могут вызвать проблемы с XSS (см. [Межсайтовый скриптинг (XSS)](../dopolnitelnye-primechaniya-flask/soobrazheniya-bezopasnosti-flask.md#mezhsaitovyi-skripting-xss)). Также убедитесь, что файлы `.php` запрещены, если сервер их выполняет, но у кого на сервере установлен **PHP**, верно? :)

Затем функции, которые проверяют, допустимо ли расширение, загружают файл и перенаправляют пользователя на URL-адрес загруженного файла:

```python
def allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS

@app.route('/', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        # проверьте, есть ли в почтовом запросе файловая часть
        if 'file' not in request.files:
            flash('No file part')
            return redirect(request.url)
        file = request.files['file']
        # если пользователь не выбирает файл,
        # браузер также отправляет пустую часть без имени файла
        if file.filename == '':
            flash('No selected file')
            return redirect(request.url)
        if file and allowed_file(file.filename):
            filename = secure_filename(file.filename)
            file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename))
            return redirect(url_for('uploaded_file',
                                    filename=filename))
    return '''
    <!doctype html>
    <title>Upload new File</title>
    <h1>Upload new File</h1>
    <form method=post enctype=multipart/form-data>
      <input type=file name=file>
      <input type=submit value=Upload>
    </form>
    '''
```

Так что же на самом деле делает эта функция [secure\_filename ()](https://werkzeug.palletsprojects.com/en/1.0.x/utils/#werkzeug.utils.secure\_filename)? Проблема в том, что существует принцип, называемый «никогда не доверяйте вводу пользователя». Это также верно для имени загруженного файла. Все отправленные данные формы могут быть подделаны, а имена файлов могут быть опасными. На данный момент просто помните: всегда используйте эту функцию для защиты имени файла перед сохранением его непосредственно в файловой системе.

{% hint style="info" %}
**Информация для профессионалов:**

Итак, вас интересует, что делает эта функция [secure\_filename ()](https://werkzeug.palletsprojects.com/en/1.0.x/utils/#werkzeug.utils.secure\_filename) и в чем проблема, если вы ее не используете? Представьте, что кто-то отправит вашему приложению в качестве имени файла следующую информацию:

```python
filename = "../../../../home/username/.bashrc"
```

Предполагая, что ряд **../** правильный, и вы присоедините его к **UPLOAD\_FOLDER**, пользователь может иметь возможность изменять файл в файловой системе сервера, который он или она не должны изменять. Это требует некоторых знаний о том, как выглядит приложение, но, поверьте, хакеры терпеливы :)

Теперь посмотрим, как работает эта функция:

```python
>>> secure_filename('../../../../home/username/.bashrc')
'home_username_.bashrc'
```
{% endhint %}

Теперь не хватает еще одного: обслуживания загруженных файлов. В **upload\_file ()** мы перенаправляем пользователя на `url_for ('uploaded_file', filename = filename)`, то есть `/uploads/filename`. Поэтому мы пишем функцию **uploaded\_file ()**, которая возвращает файл с таким именем. Начиная с `Flask 0.5` мы можем использовать функцию, которая делает это за нас:

```python
from flask import send_from_directory

@app.route('/uploads/<filename>')
def uploaded_file(filename):
    return send_from_directory(app.config['UPLOAD_FOLDER'],
                               filename)
```

В качестве альтернативы вы можете зарегистрировать **uploaded\_file** как правило **build\_only** и использовать **SharedDataMiddleware**. Это также работает со старыми версиями **Flask**:

```python
from werkzeug.middleware.shared_data import SharedDataMiddleware
app.add_url_rule('/uploads/<filename>', 'uploaded_file',
                 build_only=True)
app.wsgi_app = SharedDataMiddleware(app.wsgi_app, {
    '/uploads':  app.config['UPLOAD_FOLDER']
})
```

Если вы сейчас запустите приложение, все должно работать как положено.

## Улучшение загрузок

_Новое в версии 0.6_.

Так как именно **Flask** обрабатывает загрузки? Что ж, он будет хранить их в памяти веб-сервера, если файлы разумно малы, в противном случае во временном месте (как возвращается [tempfile.gettempdir ()](https://docs.python.org/3/library/tempfile.html#tempfile.gettempdir)). Но как указать максимальный размер файла, после которого загрузка прерывается? По умолчанию **Flask** с радостью принимает загрузку файлов в неограниченный объем памяти, но вы можете ограничить это, установив конфигурационный ключ **MAX\_CONTENT\_LENGTH**:

```python
from flask import Flask, Request

app = Flask(__name__)
app.config['MAX_CONTENT_LENGTH'] = 16 * 1024 * 1024
```

Приведенный выше код ограничивает максимально допустимую полезную нагрузку до 16 мегабайт. Если передается файл большего размера, **Flask** вызовет исключение [RequestEntityTooLarge](https://werkzeug.palletsprojects.com/en/1.0.x/exceptions/#werkzeug.exceptions.RequestEntityTooLarge).

{% hint style="info" %}
**Проблема сброса подключения:**

При использовании **локального сервера разработки** вы можете получить ошибку сброса соединения вместо ответа **413**. Вы получите правильный ответ о состоянии при запуске приложения с производственным сервером **WSGI**.
{% endhint %}

Эта функция была добавлена в `Flask 0.6`, но может быть реализована и в более старых версиях путем создания подкласса объекта запроса. Для получения дополнительной информации обратитесь к документации **Werkzeug** по работе с файлами.

## Индикаторы выполнения загрузки

Некоторое время назад у многих разработчиков возникла идея читать входящий файл небольшими порциями и сохранять ход загрузки в базе данных, чтобы иметь возможность опрашивать прогресс с помощью **JavaScript** от клиента. Короче говоря: клиент каждые 5 секунд спрашивает сервер, сколько он уже передал. Вы понимаете иронию? Клиент просит то, что он уже должен знать.

## Более простое решение

Теперь есть лучшие решения, которые работают быстрее и надежнее. Существуют библиотеки **JavaScript**, такие как [jQuery](https://jquery.com/), в которых есть плагины форм для облегчения построения индикатора выполнения.

Поскольку общий шаблон для загрузки файлов существует почти без изменений во всех приложениях, связанных с загрузкой, существует также расширение **Flask** под названием [Flask-Uploads](https://pythonhosted.org/Flask-Uploads/), которое реализует полноценный механизм загрузки с добавлением расширений в белый и черный список и т. д.
