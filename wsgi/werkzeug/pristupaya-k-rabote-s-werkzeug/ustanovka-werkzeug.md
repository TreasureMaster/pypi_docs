# Установка Werkzeug

## Версия Python

Мы рекомендуем использовать последнюю версию Python 3. **Werkzeug** поддерживает Python 3.5 и новее, а также Python 2.7.

## Зависимости

**Werkzeug** не имеет прямых зависимостей.

### Необязательные зависимости

Эти дистрибутивы не будут установлены автоматически. **Werkzeug** обнаружит и использует их, если вы их установите.

* [SimpleJSON](https://simplejson.readthedocs.io/en/latest/) - это быстрая реализация **JSON**, совместимая с модулем **json** Python. Если он установлен, он предпочтительнее для операций с **JSON**.
* [Click](https://pypi.org/project/click/) обеспечивает ведение журнала запросов при использовании сервера разработки.
* [Watchdog](https://pypi.org/project/watchdog/) обеспечивает более быструю и эффективную перезагрузку для сервера разработки.

## Виртуальные среды

Используйте виртуальную среду для управления зависимостями вашего проекта как в разработке, так и в производстве.

Какую проблему решает виртуальная среда? Чем больше у вас проектов Python, тем больше вероятность, что вам придется работать с разными версиями библиотек Python или даже с самим Python. Новые версии библиотек для одного проекта могут нарушить совместимость в другом проекте.

Виртуальные среды - это независимые группы библиотек Python, по одной для каждого проекта. Пакеты, установленные для одного проекта, не повлияют на другие проекты или пакеты операционной системы.

Python 3 поставляется в комплекте с модулем [venv](https://docs.python.org/3/library/venv.html#module-venv) для создания виртуальных сред. Если вы используете современную версию Python, можете перейти к следующему разделу.

Если вы используете Python 2, см. сначала [установите virtualenv](ustanovka-werkzeug.md#ustanovka-virtualenv).

### Создание виртуальной среды

Создайте папку проекта и папку **venv** внутри:

```bash
mkdir myproject
cd myproject
python3 -m venv venv
```

В Windows:

```bash
py -3 -m venv venv
```

Если вам нужно установить **virtualenv**, потому что вы используете более старую версию Python, используйте вместо этого следующую команду:

```bash
virtualenv venv
```

В Windows:

```bash
\Python27\Scripts\virtualenv.exe venv
```

### Активация виртуальной среды

Прежде чем приступить к работе над своим проектом, активируйте соответствующую среду:

```bash
. venv/bin/activate
```

В Windows:

```bash
venv\Scripts\activate
```

Приглашение вашей оболочки изменится, чтобы отобразить имя активированной среды.

## Установка Werkzeug

В активированной среде используйте следующую команду для установки Werkzeug:

```bash
pip install Werkzeug
```

### Жизнь на краю

Если вы хотите работать с последним кодом **Werkzeug** до его выпуска, установите или обновите код из основной ветки:

```bash
pip install -U https://github.com/pallets/werkzeug/archive/master.tar.gz
```

## Установка virtualenv

Если вы используете Python 2, модуль **venv** недоступен. Вместо этого установите [virtualenv](https://virtualenv.pypa.io/en/latest/).

В Linux **virtualenv** предоставляется вашим менеджером пакетов:

```bash
# Debian, Ubuntu
sudo apt-get install python-virtualenv

# CentOS, Fedora
sudo yum install python-virtualenv

# Arch
sudo pacman -S python-virtualenv
```

Если у вас Mac OS X или Windows, скачайте [get-pip.py](https://bootstrap.pypa.io/get-pip.py), а затем:

```bash
sudo python2 Downloads/get-pip.py
sudo python2 -m pip install virtualenv
```

В Windows от имени администратора:

```bash
\Python27\python.exe Downloads\get-pip.py
\Python27\python.exe -m pip install virtualenv
```

Теперь вы можете продолжить [создание среды](ustanovka-werkzeug.md#sozdanie-virtualnoi-sredy).
