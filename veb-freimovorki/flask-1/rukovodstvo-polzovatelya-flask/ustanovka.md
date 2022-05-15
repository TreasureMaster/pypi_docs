# Установка Flask

## Установка

### Версия Python

Мы рекомендуем использовать последнюю версию **Python 3**. **Flask** поддерживает Python 3.5 и новее, Python 2.7 и PyPy.

### Зависимости

Эти дистрибутивы будут установлены автоматически при установке **Flask**.

* [Werkzeug](https://palletsprojects.com/p/werkzeug/) реализует **WSGI**, стандартный интерфейс **Python** между приложениями и серверами.
* [Jinja](../../../shablonizatory/jinja/) - это язык шаблонов, который отображает страницы, которые обслуживает ваше приложение.
* [MarkupSafe](https://palletsprojects.com/p/markupsafe/) поставляется с **Jinja**. Он избегает ненадежного ввода при рендеринге шаблонов, чтобы избежать атак путем внедрения.
* [ItsDangerous](https://palletsprojects.com/p/itsdangerous/) надежно подписывает данные, чтобы гарантировать их целостность. Это используется для защиты файла **cookie** сеанса **Flask**.
* [Click](https://palletsprojects.com/p/click/) - это платформа для написания приложений командной строки. Он предоставляет команду **flask** и позволяет добавлять собственные команды управления.

### Необязательные зависимости

Эти дистрибутивы не будут установлены автоматически. **Flask** обнаружит и использует их, если вы их установите.

* [Blinker](https://pythonhosted.org/blinker/) поддерживает [Signals](https://flask.palletsprojects.com/en/1.1.x/signals/#signals).
* [SimpleJSON](https://simplejson.readthedocs.io/en/latest/) - это быстрая реализация **JSON**, совместимая с модулем **json** **Python**. Если он установлен, он предпочтительнее для операций с **JSON**.
* [python-dotenv](https://github.com/theskumar/python-dotenv#readme) включает поддержку переменных среды из **dotenv** при выполнении команд **flask**.
* [Watchdog](https://pythonhosted.org/watchdog/) обеспечивает более быстрый и эффективный перезагрузчик для сервера разработки.

### Виртуальные среды

Используйте виртуальную среду для управления зависимостями вашего проекта как в процессе разработки, так и в производстве.

Какую проблему решает виртуальная среда? Чем больше у вас проектов **Python**, тем больше вероятность, что вам придется работать с разными версиями библиотек **Python** или даже с самим **Python**. Новые версии библиотек для одного проекта могут нарушить совместимость в другом проекте.

Виртуальные среды - это независимые группы библиотек **Python**, по одной для каждого проекта. Пакеты, установленные для одного проекта, не повлияют на другие проекты или пакеты операционной системы.

**Python 3** поставляется в комплекте с модулем [**venv**](https://docs.python.org/3/library/venv.html#module-venv) для создания виртуальных сред. Если вы используете современную версию **Python**, вы можете перейти к следующему разделу.

Если вы используете **Python 2**, см. установку virtualenv.

### Создание среды

Создайте папку проекта и папку `venv` внутри:

```bash
$ mkdir myproject
$ cd myproject
$ python3 -m venv venv
```

В **Windows**:

```bash
$ py -3 -m venv venv
```

Если вам нужно установить **virtualenv**, потому что вы используете **Python 2**, используйте вместо этого следующую команду:

```bash
$ python2 -m virtualenv venv
```

В **Windows**:

```bash
> \Python27\Scripts\virtualenv.exe venv
```

#### Активируйте среду

Перед тем как приступить к работе над своим проектом, активируйте соответствующую среду:

```bash
$ . venv/bin/activate
```

В **Windows**:

```bash
> venv\Scripts\activate
```

Приглашение вашей оболочки изменится, чтобы показать имя активированной среды.

### Установка Flask

В активированной среде используйте следующую команду для установки **Flask**:

```bash
$ pip install Flask
```

**Flask** установлен. Ознакомьтесь с кратким руководством или перейдите к обзору документации.

#### Жизнь на краю

Если вы хотите работать с последним кодом **Flask** до его выпуска, установите или обновите код из основной ветки:

```bash
$ pip install -U https://github.com/pallets/flask/archive/master.tar.gz
```

### Установка virtualenv

Если вы используете **Python 2**, модуль **venv** недоступен. Вместо этого установите [virtualenv](https://virtualenv.pypa.io/en/latest/).

В **Linux** **virtualenv** предоставляется вашим менеджером пакетов:

```bash
# Debian, Ubuntu
$ sudo apt-get install python-virtualenv

# CentOS, Fedora
$ sudo yum install python-virtualenv

# Arch
$ sudo pacman -S python-virtualenv
```

Если у вас **Mac OS X** или **Windows**, скачайте [get-pip.py](https://bootstrap.pypa.io/get-pip.py), а затем:

```bash
$ sudo python2 Downloads/get-pip.py
$ sudo python2 -m pip install virtualenv
```

В **Windows** от имени администратора:

```bash
> \Python27\python.exe Downloads\get-pip.py
> \Python27\python.exe -m pip install virtualenv
```

Теперь вы можете вернуться выше и создать среду.
