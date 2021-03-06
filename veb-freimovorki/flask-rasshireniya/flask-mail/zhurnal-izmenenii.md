# Журнал изменений

Здесь вы можете увидеть полный список изменений между выпусками **Flask-Mail**.

## Версия 0.9.1

Дата выпуска: 28 сентября 2014 г.

* Добавлена опция принудительного прикрепления файлов ASCII
* Исправлена функция _**force\_text**_
* Исправлена некоторая поддержка Python 3 в отношении политики электронной почты
* Добавлена поддержка параметров ESMTP.
* Исправлены различные проблемы с Юникодом, связанные с вложениями сообщений, темами и адресами электронной почты.

## Версия 0.9.0

Дата выпуска: 14 июня 2013 г.

* Добавлена начальная поддержка Python 3

## Версия 0.8.2

Дата выпуска 11 апреля 2013 г.

* Удалено раздражающее сообщение о случайной печати.

## Версия 0.8.1

Дата выпуска: 4 апреля 2013 г.

* Исправлена ошибка с новым объектом состояния

## Версия 0.8.0

Дата выпуска: 3 апреля 2013 г.

* Исправлена ошибка с дублированием получателей
* Изменены параметры конфигурации, чтобы было меньше путаницы
* Общий API очищен, поскольку все происходило в нескольких разных местах

## Версия 0.7.6

Дата выпуска: 11 марта 2013 г.

* Исправлена ошибка, из-за которой поля **cc** и **bcc** не были списками

## Версия 0.7.5

Дата выпуска: 3 марта 2013 г.

* Исправлена ошибка с символами, отличными от ascii, в адресе электронной почты
* Значение конфигурации `MAIL_FAIL_SILENTLY` по умолчанию равно `False`
* Заголовок **bcc** больше не устанавливается, поскольку некоторые почтовые серверы пересылают его получателю.

## Версия 0.7.4

Релиз 20 ноября 2012 г.

* Разрешает отправку сообщений без тела

## Версия 0.7.3

Выпущено 27 сентября 2012 г.

* Добавлен _**extra\_headers**_ в класс сообщений **Message**

## Версия 0.7.2

Выпущено 16 сентября 2012 г.

* Добавлен метод `__str__` в класс сообщений
* Добавлен параметр набора символов сообщения, который по умолчанию - utf-8

## Версия 0.7.1

Выпущено 5 сентября 2012 г.

* Указаны заголовки даты и идентификатора **ID** сообщения

## Версия 0.7 и предыдущие

Первоначальная разработка Дэна Джейкоба и Рона ДюПлена. Раньше журнала изменений не было.
