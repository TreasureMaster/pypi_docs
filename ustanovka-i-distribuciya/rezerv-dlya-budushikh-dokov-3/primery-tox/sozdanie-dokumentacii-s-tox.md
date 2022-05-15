# Создание документации с tox

Можно создать документацию по проекту с помощью самого **tox**. Преимущество этого пути заключается в том, что теперь создание документации может быть частью CI, и всякий раз, когда какие-либо валидации/проверки/операции завершаются неудачно при создании документации, вы поймаете это в **tox**.

## Sphinx

Нет необходимости использовать загадочный файл **make** для создания документации **sphinx**. Можно использовать **tox**, чтобы гарантировать, что все нужные зависимости доступны в виртуальной среде, и даже указать версию python, необходимую для выполнения сборки. Например, если файловая структура **sphinx** находится в папке **doc**, следующая конфигурация сгенерирует документацию в `{toxworkdir} / docs_out` и распечатает ссылку на созданную документацию:

```bash
[testenv:docs]
description = invoke sphinx-build to build the HTML docs
basepython = python3.7
deps = sphinx >= 1.7.5, < 2
commands = sphinx-build -d "{toxworkdir}/docs_doctree" doc "{toxworkdir}/docs_out" --color -W -bhtml {posargs}
           python -c 'import pathlib; print("documentation available under file://\{0\}".format(pathlib.Path(r"{toxworkdir}") / "docs_out" / "index.html"))'
```

Обратите внимание, здесь мы говорим, что нам также требуется `python 3.7`, что позволяет нам использовать f-строки в **sphinx** `conf.py`. Теперь можно указать отдельную тестовую среду, которая будет проверять правильность ссылок.

## mkdocs

Определите одну среду для написания/генерации документации, а другую - для ее развертывания. Используйте логику подстановки конфигурации, чтобы избежать многократного определения зависимостей:

```bash
[testenv:docs]
description = Run a development server for working on documentation
basepython = python3.7
deps = mkdocs >= 1.7.5, < 2
       mkdocs-material
commands = mkdocs build --clean
           python -c 'print("###### Starting local server. Press Control+C to stop server ######")'
           mkdocs serve -a localhost:8080

[testenv:docs-deploy]
description = built fresh docs and deploy them
deps = {[testenv:docs]deps}
basepython = {[testenv:docs]basepython}
commands = mkdocs gh-deploy --clean
```
