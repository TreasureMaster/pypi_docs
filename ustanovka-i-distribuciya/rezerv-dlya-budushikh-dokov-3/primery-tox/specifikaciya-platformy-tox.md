# Спецификация платформы tox

## Базовый мультиплатформенный пример

Предполагая следующий макет:

```bash
tox.ini      # см. содержание ниже
setup.py     # классический файл distutils/setuptools - setup.py
```

и следующее содержимое `tox.ini`:

```bash
[tox]
# поддержка спецификации платформы доступна с версии 2.0
minversion = 2.0
envlist = py{27,36}-{mylinux,mymacos,mywindows}

[testenv]
# среда будет пропущена, если регулярное выражение не соответствует строке sys.platform
platform = mylinux: linux
           mymacos: darwin
           mywindows: win32

# вы можете указать зависимости и их версии на основе сред, отфильтрованных платформой
deps = mylinux,mymacos: py==1.4.32
       mywindows: py==1.4.30

# после вызова tox вас встретят в соответствии с вашей платформой
commands=
   mylinux: python -c 'print("Hello, Linus!")'
   mymacos: python -c 'print("Hello, Steve!")'
   mywindows: python -c 'print("Hello, Bill!")'
```

вы можете вызвать **tox** в каталоге, в котором находится ваш `tox.ini`. **tox** создает две среды **virtualenv** с интерпретаторами `python2.7` и `python3.6` соответственно, а затем запускает указанную команду в соответствии с платформой, на которой вы вызываете **tox**.
