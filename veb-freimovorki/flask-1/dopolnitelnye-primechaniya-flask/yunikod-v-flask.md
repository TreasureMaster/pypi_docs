# Юникод в Flask

Когда дело доходит до текста, **Flask**, как и **Jinja2** и **Werkzeug**, полностью основан на Unicode. Не только эти библиотеки, но и большинство веб-библиотек Python, которые работают с текстом. Если вы еще не знакомы с Unicode, вам, вероятно, следует прочитать «[Абсолютный минимум, который должен знать каждый разработчик программного обеспечения о Unicode и наборах символов](https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/)». Эта часть документации просто пытается охватить самые основы, чтобы у вас было приятное знакомство с вещами, связанными с Unicode.

## Автоматическое преобразование

Flask имеет несколько предположений о вашем приложении (которые вы, конечно, можете изменить), которые обеспечивают базовую и безболезненную поддержку Unicode:

* кодировка текста на вашем сайте - UTF-8
* внутри вы всегда будете использовать Unicode исключительно для текста, за исключением буквальных строк с точками символов ASCII.
* кодирование и декодирование происходит всякий раз, когда вы говорите по протоколу, который требует передачи байтов.

Так что это значит для тебя?

HTTP основан на байтах. Не только протокол, но и система, используемая для адресации документов на серверах (так называемые URI или URL-адреса). Однако HTML, который обычно передается поверх HTTP, поддерживает большое количество наборов символов, и какие из них используются, передаются в заголовке HTTP. Чтобы не усложнять этот процесс, **Flask** предполагает, что если вы отправляете Unicode, вы хотите, чтобы он был в кодировке UTF-8. **Flask** выполнит кодировку и настройку соответствующих заголовков за вас.

То же самое верно, если вы разговариваете с базами данных с помощью **SQLAlchemy** или аналогичной системы ORM. Некоторые базы данных имеют протокол, который уже передает Unicode, и если они этого не делают, **SQLAlchemy** или другой ваш ORM должен позаботиться об этом.

## Золотое правило

Итак, практическое правило: если вы не имеете дело с двоичными данными, работайте с Unicode. Что означает работа с Unicode в Python 2.x?

* пока вы используете только кодовые точки ASCII (в основном числа, некоторые специальные символы латинских букв без умляутов или чего-то особенного), вы можете использовать обычные строковые литералы (`'Hello World'`).
* если вам нужно что-то еще, кроме ASCII в строке, вы должны пометить эту строку как строку Unicode, добавив к ней строчную букву **u**. (как `u'Hänsel und Gretel'`)
* если вы используете в файлах Python символы, отличные от Unicode, вы должны указать Python, какую кодировку использует ваш файл. Опять же, для этой цели я рекомендую UTF-8. Чтобы сообщить интерпретатору свою кодировку, вы можете поместить `# -*- coding: utf-8 -*-` в первую или вторую строку исходного файла Python.
* **Jinja** настроен для декодирования файлов шаблонов из UTF-8. Поэтому убедитесь, что ваш редактор также сохранил файл как UTF-8.

## Самостоятельное кодирование и декодирование

Если вы говорите с файловой системой или чем-то, что на самом деле не основано на Unicode, вам нужно будет убедиться, что вы правильно декодируете при работе с интерфейсом Unicode. Так, например, если вы хотите загрузить файл в файловую систему и встроить его в шаблон **Jinja2**, вам придется декодировать его из кодировки этого файла. Здесь вступает в игру старая проблема, заключающаяся в том, что текстовые файлы не указывают свою кодировку. Так что сделайте себе одолжение и ограничьтесь UTF-8 и для текстовых файлов.

В любом случае. Чтобы загрузить такой файл в Unicode, вы можете использовать встроенный метод **str.decode ()**:

```python
def read_file(filename, charset='utf-8'):
    with open(filename, 'r') as f:
        return f.read().decode(charset)
```

Чтобы перейти от Unicode к определенной кодировке, такой как UTF-8, вы можете использовать метод **unicode.encode ()**:

```python
def write_file(filename, contents, charset='utf-8'):
    with open(filename, 'w') as f:
        f.write(contents.encode(charset))
```

## Конфигурирование редакторов

Большинство редакторов в настоящее время по умолчанию сохраняются как UTF-8, но если ваш редактор не настроен для этого, вы должны его изменить. Вот несколько распространенных способов сохранить ваш редактор как UTF-8:

* **Vim:** поместите `set enc = utf-8` в ваш файл `.vimrc`.
* **Emacs:** либо используйте файл **cookie** кодирования, либо поместите его в свой файл `.emacs`:

```
(prefer-coding-system 'utf-8)
(setq default-buffer-file-coding-system 'utf-8)
```

* **Notepad++:**
  * Зайдите в Settings -> Preferences…
  * Выберите вкладку «New Document/Default Directory».
  * Выберите «UTF-8 without BOM» в качестве кодировки.

Также рекомендуется использовать формат новой строки Unix, вы можете выбрать его на той же панели, но это не является обязательным требованием.
