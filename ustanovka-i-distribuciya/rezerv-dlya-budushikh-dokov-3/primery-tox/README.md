# Примеры использования tox

* [Базовое использование](bazovoe-ispolzovanie-tox.md)
  * Простая tox.ini / среда по умолчанию
  * pyproject.toml tox унаследованный ini
  * Указание платформы
  * Разрешение команд, отличных от virtualenv
  * Зависимости от файла requirements.txt или определения ограничений
  * Использование другого URL-адреса PyPI по умолчанию
  * Установка зависимостей с нескольких серверов PyPI
  * Дальнейшая настройка установки
  * Принудительное воссоздание виртуальных сред
  * Передача переменных среды
  * Установка переменных окружения
  * Особое обращение с PYTHONHASHSEED
  * Интеграция с командой «setup.py test»
  * Игнорирование кода выхода команды
  * Сжатие матрицы зависимостей
  * Использование генеративных имен разделов
  * Запретить символические ссылки в virtualenv
  * Параллельный режим
  * автоматическая подготовка tox
* Упаковка
  * setuptools
  * flit
  * poetry
* pytest и tox
  * Базовый пример
  * Расширенный пример: измените каталог перед тестированием и используйте per-virtualenv tempdir
  * Использование нескольких процессоров для тестовых прогонов
  * Известные проблемы и ограничения
* unittest2, discover and tox
  * Запуск модульных тестов с помощью «discover»
  * Запуск тестов unittest2 и sphinx за один раз
* nose и tox
  * Пример базового nosetests
  * Еще примеры?
* Создание документации
  * sphinx
  * mkdocs
* Общие советы и рекомендации
  * Интерактивная передача позиционных аргументов
  * Изменения и отслеживание зависимостей
  * Выбор одной или нескольких сред для запуска тестов
  * Доступ к артефактам пакета между несколькими запусками tox
  * basepython по умолчанию, переопределение
  * Избегайте дорогостоящих sdist
  * Общие сведения о кодах выхода InvocationError
* Использование tox с сервером интеграции Jenkins
  * Использование заданий с несколькими конфигурациями Jenkins
  * нулевая установка для агентов
  * Интеграция проверок документации «сфинкс» в работу Jenkins
  * Доступ к артефактам пакетов между заданиями Jenkins
  * Как избежать ошибки «слишком длинный путь» при использовании длинных строк shebang
  * Параллельное выполнение среды tox
* Среда разработки
  * Создание сред разработки с использованием параметра --devenv
  * Создание сред разработки с использованием конфигурации
  * Пример 1: Базовый сценарий
  * Пример 2: более сложный сценарий
* Спецификация платформы
  * Базовый мультиплатформенный пример
