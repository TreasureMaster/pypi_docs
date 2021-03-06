# Становление Flask большим

Вот ваши варианты при расширении кодовой базы или масштабировании вашего приложения.

## Прочтите исходники

**Flask** начал частично с того, чтобы продемонстрировать, как создать собственный фреймворк на основе существующих широко используемых инструментов **Werkzeug** (**WSGI**) и **Jinja** (создание шаблонов), и по мере его развития он стал полезен широкой аудитории. По мере того, как вы расширяете свою кодовую базу, не используйте **Flask** просто - поймите это. Прочтите источник. Код **Flask** написан для чтения; его документация опубликована, поэтому вы можете использовать его внутренние API. **Flask** придерживается документированных API-интерфейсов в вышестоящих библиотеках и документирует свои внутренние утилиты, чтобы вы могли найти точки привязки, необходимые для вашего проекта.

## Хук. Расширение

Документация [API](../api-dokumentaciya-flask/) полна доступных переопределений, точек привязки и сигналов [Signals](signaly-flask.md). Вы можете предоставить собственные классы для таких вещей, как объекты запроса и ответа. Погрузитесь глубже в API, которые вы используете, и поищите настройки, которые доступны из коробки в выпуске **Flask**. Найдите способы реорганизации вашего проекта в набор утилит и расширений **Flask**. Изучите множество [расширений](rasshireniya-flask.md) в сообществе и поищите шаблоны для создания собственных расширений, если вы не найдете нужных инструментов.

## Подклассы

Класс [Flask](../api-dokumentaciya-flask/obekt-prilozheniya-flask.md#klass-flask-flask) имеет множество методов, предназначенных для создания подклассов. Вы можете быстро добавить или настроить поведение, создав подкласс **Flask** (см. документацию по связанным методам) и используя этот подкласс везде, где вы создаете экземпляр класса приложения. Это хорошо работает с [фабриками приложений](../patterny-flask/fabriki-prilozheniya-flask.md). См. пример в разделе [Создание подклассов Flask](../patterny-flask/sozdanie-podklassa-flask.md).

## Обернуть промежуточным ПО

В главе «[Диспетчеризация приложений](../patterny-flask/dispetcherizaciya-prilozhenii-flask.md)» подробно показано, как применять промежуточное ПО. Вы можете ввести промежуточное ПО **WSGI**, чтобы обернуть ваши экземпляры **Flask** и внести исправления и изменения на уровне между вашим приложением **Flask** и вашим HTTP-сервером. **Werkzeug** включает несколько [промежуточных программ](https://werkzeug.palletsprojects.com/en/1.0.x/middleware/).

## Форк

Если ни один из перечисленных выше вариантов не работает, разветвите **Flask**. Большая часть кода **Flask** находится в **Werkzeug** и **Jinja2**. Эти библиотеки выполняют большую часть работы. **Flask** - это просто паста, которая склеивает их вместе. Для каждого проекта есть момент, когда базовая структура мешает (из-за предположений исходных разработчиков). Это естественно, потому что, если бы это было не так, фреймворк был бы очень сложной системой для начала, что привело бы к крутой кривой обучения и большому разочарованию пользователей.

Это не уникально для **Flask**. Многие люди используют исправленные и модифицированные версии своих фреймворков для устранения недостатков. Эта идея также отражена в лицензии **Flask**. Вам не нужно вносить какие-либо изменения, если вы решите изменить структуру.

Обратной стороной разветвления, конечно же, является то, что расширения **Flask**, скорее всего, сломаются, потому что новый фреймворк имеет другое имя импорта. Более того, в зависимости от количества изменений интеграция предшествующих изменений может быть сложным процессом. Поэтому разветвление должно быть крайней мерой.

## Масштабируйте как профессионал

Для многих веб-приложений сложность кода является меньшей проблемой, чем масштабирование для ожидаемого количества пользователей или вводов данных. **Flask** сам по себе ограничен только с точки зрения масштабирования кодом вашего приложения, хранилищем данных, которое вы хотите использовать, а также реализацией Python и веб-сервером, на котором вы работаете.

Хорошее масштабирование означает, например, что если вы удвоите количество серверов, вы получите примерно вдвое большую производительность. Плохое масштабирование означает, что если вы добавите новый сервер, приложение не будет работать лучше или даже не будет поддерживать второй сервер.

В отношении масштабирования во **Flask** есть только один ограничивающий фактор - это контекстные локальные прокси. Они зависят от контекста, который во **Flask** определяется как поток, процесс или **greenlet**. Если ваш сервер использует какой-либо параллелизм, который не основан на потоках или **greenlet**, **Flask** больше не сможет поддерживать эти глобальные прокси. Однако большинство серверов используют потоки, **greenlet** или отдельные процессы для достижения параллелизма, и все эти методы хорошо поддерживаются базовой библиотекой **Werkzeug**.

## Обсуди с сообществом

Разработчики **Flask** делают фреймворк доступным для пользователей с большими и маленькими базами кода. Если вы обнаружите препятствие на своем пути, вызванное **Flask**, не стесняйтесь обращаться к разработчикам в списке рассылки или на сервере **Discord**. Лучший способ для разработчиков **Flask** и расширений **Flask** улучшить инструменты для более крупных приложений - это получить отзывы пользователей.
