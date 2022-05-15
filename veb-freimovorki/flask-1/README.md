# Flask

Добро пожаловать в документацию **Flask**. Начните с установки, а затем получите обзор с помощью Quickstart. Существует также более подробное руководство, в котором показано, как создать небольшое, но полное приложение с помощью Flask. Общие шаблоны описаны в разделе Шаблоны для Flask. Остальные документы подробно описывают каждый компонент Flask с полной ссылкой в разделе API.

**Flask** зависит от механизма шаблонов [Jinja](https://palletsprojects.com/p/jinja/) и инструментария [Werkzeug](https://palletsprojects.com/p/werkzeug/) WSGI. Документацию для этих библиотек можно найти по адресу:

* [документация Jinja](../../shablonizatory/jinja/)
* [документация Werkzeug](https://werkzeug.palletsprojects.com/en/1.0.x/)

## Руководство пользователя

Эта часть документации, в основном прозаическая, начинается с некоторой справочной информации о **Flask**, а затем сосредотачивается на пошаговых инструкциях по веб-разработке с помощью **Flask**.

### [Предисловие](rukovodstvo-polzovatelya-flask/predislovie.md#predislovie)

* [Что значит «микро»?](rukovodstvo-polzovatelya-flask/predislovie.md#chto-znachit-mikro)
* [Конфигурация и условные обозначения](rukovodstvo-polzovatelya-flask/predislovie.md#konfiguraciya-i-uslovnye-oboznacheniya)
* [Рост вместе с Flask](rukovodstvo-polzovatelya-flask/predislovie.md#rost-vmeste-s-flask)

### [Предисловие для опытных программистов](rukovodstvo-polzovatelya-flask/predislovie.md#predislovie-dlya-opytnykh-programmistov)

* [Локальные потоки в Flask](rukovodstvo-polzovatelya-flask/predislovie.md#lokalnye-potoki-v-flask)
* [Разработка для Интернет с осторожностью](rukovodstvo-polzovatelya-flask/predislovie.md#razrabotka-dlya-internet-s-ostorozhnostyu)

### [Установка](rukovodstvo-polzovatelya-flask/ustanovka.md#ustanovka)

* [Версия Python](rukovodstvo-polzovatelya-flask/ustanovka.md#versiya-python)
* [Зависимости](rukovodstvo-polzovatelya-flask/ustanovka.md#zavisimosti)
* [Необязательные зависимости](rukovodstvo-polzovatelya-flask/ustanovka.md#neobyazatelnye-zavisimosti)
* [Виртуальные среды](rukovodstvo-polzovatelya-flask/ustanovka.md#virtualnye-sredy)
* [Создание среды](rukovodstvo-polzovatelya-flask/ustanovka.md#sozdanie-sredy)
* [Установка Flask](rukovodstvo-polzovatelya-flask/ustanovka.md#ustanovka-flask)
* [Установка virtualenv](rukovodstvo-polzovatelya-flask/ustanovka.md#ustanovka-virtualenv)

### [Быстрый старт](rukovodstvo-polzovatelya-flask/bystryi-start-flask.md#bystryi-start)

* [Минимальное приложение](rukovodstvo-polzovatelya-flask/bystryi-start-flask.md#minimalnoe-prilozhenie)
* [Что делать, если Сервер не запускается](rukovodstvo-polzovatelya-flask/bystryi-start-flask.md#chto-delat-esli-server-ne-zapuskaetsya)
* [Режим отладки](rukovodstvo-polzovatelya-flask/bystryi-start-flask.md#rezhim-otladki)
* [Маршрутизация](rukovodstvo-polzovatelya-flask/bystryi-start-flask.md#marshrutizaciya)
* [Правила для переменных маршрутов](rukovodstvo-polzovatelya-flask/bystryi-start-flask.md#pravila-dlya-peremennykh-marshrutov)
* [Уникальные URL-адреса / поведение при перенаправлении](rukovodstvo-polzovatelya-flask/bystryi-start-flask.md#unikalnye-url-adresa-povedenie-pri-perenapravlenii)
* [Создание URL](rukovodstvo-polzovatelya-flask/bystryi-start-flask.md#sozdanie-url)
* [HTTP-методы](rukovodstvo-polzovatelya-flask/bystryi-start-flask.md#http-metody)
* [Статические файлы](rukovodstvo-polzovatelya-flask/bystryi-start-flask.md#staticheskie-faily)
* [Шаблоны рендеринга](rukovodstvo-polzovatelya-flask/bystryi-start-flask.md#shablony-renderinga)
* [Доступ к данным запроса](rukovodstvo-polzovatelya-flask/bystryi-start-flask.md#dostup-k-dannym-zaprosa)
* [Локальные объекты контекста](rukovodstvo-polzovatelya-flask/bystryi-start-flask.md#kontekstnye-lokalnye-dannye)
* [Объект запроса](rukovodstvo-polzovatelya-flask/bystryi-start-flask.md#obekt-zaprosa)
* [Загрузка файлов](rukovodstvo-polzovatelya-flask/bystryi-start-flask.md#zagruzka-failov)
* [Cookies](rukovodstvo-polzovatelya-flask/bystryi-start-flask.md#cookies)
* [Перенаправления и ошибки](rukovodstvo-polzovatelya-flask/bystryi-start-flask.md#perenapravleniya-i-oshibki)
* [Об ответах](rukovodstvo-polzovatelya-flask/bystryi-start-flask.md#ob-otvetakh-response)
* [API с JSON](rukovodstvo-polzovatelya-flask/bystryi-start-flask.md#api-s-json)
* [Сессии](rukovodstvo-polzovatelya-flask/bystryi-start-flask.md#sessii)
* [Быстрая отправка сообщений (Flash)](rukovodstvo-polzovatelya-flask/bystryi-start-flask.md#bystraya-otpravka-soobshenii-flash)
* [Логирование](rukovodstvo-polzovatelya-flask/bystryi-start-flask.md#logirovanie)
* [Подключение к промежуточному программному обеспечению WSGI](rukovodstvo-polzovatelya-flask/bystryi-start-flask.md#podklyuchenie-k-promezhutochnomu-programmnomu-obespecheniyu-wsgi)
* [Использование расширений Flask](rukovodstvo-polzovatelya-flask/bystryi-start-flask.md#ispolzovanie-rasshirenii-flask)
* [Развертывание на веб-сервере](rukovodstvo-polzovatelya-flask/bystryi-start-flask.md#razvertyvanie-na-veb-servere)

### [Учебник Flask](rukovodstvo-polzovatelya-flask/uchebnik-flask.md)

* [Макет проекта](rukovodstvo-polzovatelya-flask/uchebnik-flask.md#maket-proekta)
* [Настройка приложения](rukovodstvo-polzovatelya-flask/uchebnik-flask.md#nastroika-prilozheniya)
* [Определение и доступ к базе данных](rukovodstvo-polzovatelya-flask/uchebnik-flask.md#opredelenie-i-dostup-k-baze-dannykh)
* [Схемы Blueprint и виды](rukovodstvo-polzovatelya-flask/uchebnik-flask.md#skhemy-blueprints-i-vidy-views)
* [Шаблоны](rukovodstvo-polzovatelya-flask/uchebnik-flask.md#shablony-templates)
* [Статические файлы](rukovodstvo-polzovatelya-flask/uchebnik-flask.md#staticheskie-faily)
* [Схема блога](rukovodstvo-polzovatelya-flask/uchebnik-flask.md#skhema-bloga)
* [Сделайте проект устанавливаемым](rukovodstvo-polzovatelya-flask/uchebnik-flask.md#sdelaite-proekt-ustanavlivaemym)
* [Тестовое покрытие](rukovodstvo-polzovatelya-flask/uchebnik-flask.md#testovoe-pokrytie)
* [Развертывание приложения](rukovodstvo-polzovatelya-flask/uchebnik-flask.md#razvertyvanie-prilozheniya)
* [Продолжайте развиваться!](rukovodstvo-polzovatelya-flask/uchebnik-flask.md#prodolzhaite-razvivatsya)

### Шаблоны

* Настройка Jinja
* Стандартный контекст
* Стандартные фильтры
* Управление автоматическим экранированием
* Регистрация фильтров
* Контекстные процессоры

### Тестирование приложений Flask

* Приложение
* Скелет тестирования
* Первый тест
* Ведение журнала входа и выхода
* Тестовое добавление сообщений
* Другие приемы тестирования
* Подделка ресурсов и контекста
* Сохранение контекста вокруг
* Доступ и изменение сеансов
* Тестирование JSON API
* Тестирование CLI команд

### Ошибки приложения

* Инструменты регистрации ошибок
* Обработчики ошибок
* Регистрация
* Обработка
* Универсальные обработчики исключений
* Необработанные исключения
* Ведение журнала

### Отладка ошибок приложений

* Если сомневаетесь, запускайте вручную
* Работа с отладчиками

### Ведение журнала

* Базовая конфигурация
* Конфигурация по умолчанию
* Удаление обработчика по умолчанию
* Электронная почта об ошибках администраторам
* Внедрение информации запроса
* Другие библиотеки
* Werkzeug
* Расширения Flask

### Обработка конфигурации

* Основы настройки
* Возможности среды и отладки
* Встроенные значения конфигурации
* Настройка из файлов
* Настройка из переменных среды
* Рекомендации по настройке
* Разработка / Производство
* Папки экземпляров

### Сигналы

* Подписка на сигналы
* Создание сигналов
* Отправка сигналов
* Сигналы и контекст запроса Flask
* Подписки на сигналы на основе декораторов
* Основные сигналы

### Подключаемые представления

* Основной принцип
* Подсказки по методам
* Диспетчеризация на основе метода
* Декорирование видов
* Представления методов для API

### Контекст приложения

* Цель контекста
* Время жизни контекста
* Вставить контекст вручную
* Хранение данных
* События и сигналы

### Контекст запроса

* Цель контекста
* Время жизни контекста
* Вставить контекст вручную
* Как работает контекст
* Обратные вызовы и ошибки
* Обратные вызовы при разборке
* Сигналы
* Сохранение контекста при ошибке
* Примечания к прокси

### Модульные приложения с чертежами

* Почему чертежи?
* Концепция чертежей
* Мой первый чертеж
* Регистрация чертежей
* Ресурсы для чертежей
* Папка ресурсов Blueprint
* Статические файлы
* Шаблоны
* Создание URL-адресов
* Обработчики ошибок

### Расширения

* Поиск расширений
* Использование расширений
* Разработка расширений

### Интерфейс командной строки CLI

* Обнаружение приложений
* Запуск сервера разработки
* Открытие оболочки
* Среды
* Смотрите дополнительные файлы с помощью Reloader
* Режим отладки
* Переменные среды из dotenv
* Настройка параметров команды
* Отключение dotenv
* Переменные окружения из virtualenv
* Пользовательские команды
* Регистрация команд с помощью Blueprints
* Контекст приложения
* Плагины
* Пользовательские скрипты
* Интеграция PyCharm

### Сервер разработки

* Командная строка
* В коде

### Работа с оболочкой

* Интерфейс командной строки
* Создание контекста запроса
* Стрельба до / после запроса
* Дальнейшее улучшение впечатлений от оболочки

### Паттерны для Flask

* Большие приложения
* Фабрики приложений
* Диспетчеризация приложений
* Реализация исключений API
* Использование процессоров URL
* Развертывание с помощью Setuptools
* Развертывание с помощью Fabric
* Использование SQLite 3 с Flask
* SQLAlchemy в Flask
* Загрузка файлов
* Кеширование
* Просмотр декораторов
* Проверка формы с помощью WTForms
* Наследование шаблона
* Сообщение мигает
* AJAX с jQuery
* Пользовательские страницы ошибок
* Ленивая загрузка просмотров
* MongoDB с MongoEngine
* Добавление значка **** favicon
* Потоковое содержимое
* Обратные вызовы отложенного запроса
* Добавление переопределений метода HTTP
* Запрос контрольных сумм содержимого
* Фоновые задачи Celery
* Создание подкласса Flask
* Одностраничные приложения

### Варианты развертывания

* Варианты работы с хостингами
* Самостоятельные варианты

### Стать большим

* Прочтите исходники.
* Хуки. Расширения.
* Подклассы.
* Оберните промежуточным ПО.
* Форки.
* Масштабируйте как профессионал.
* Обсуди с сообществом.

## Справочник по API

* Объект приложения
* Объекты чертежей
* Данные входящего запроса
* Объекты ответа
* Сессии
* Сессионный интерфейс
* Тестовый клиент
* Тест CLI команд
* Глобальные приложения
* Полезные функции и классы
* Выталкивание сообщений
* Поддержка JSON
* Тэгированный JSON
* Визуализация шаблона
* Конфигурация
* Помощники потоков
* Полезные внутренности
* Сигналы
* Представления на основе классов
* Регистрация маршрута URL
* Просмотр параметров функции
* Интерфейс командной строки

## Дополнительные замечания

* Дизайнерские решения в Flask
* HTML/XHTML FAQ
* Соображения безопасности
* Юникод в Flask
* Разработка расширений Flask
* Руководство по стилю Pocoo
* Обновление до более новых версий
* Журнал изменений
* Лицензия
* Как внести свой вклад в Flask
