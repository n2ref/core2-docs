# Core2 Docs

Документация по проекту core2

[Online просмотр](https://shabuninil.github.io/core2-docs/index.html)


## Установка MkDocs

1) Перейдите на страницу приложения и выполните указанные действия для установки базовой версии.
   [MkDocs Installation](https://www.mkdocs.org/user-guide/installation/)
2) Установите несколько плагинов требуемых для сборки
Внешний вид [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)
```shell
$ pip install mkdocs-material
```
Плагин для отображения картинок на весь экран.
```shell
$ pip install mkdocs-glightbox
```

[Подробнее](https://squidfunk.github.io/mkdocs-material/getting-started)


## Запуск и сборка сайта документации

Для онлайн просмотра содержимого документации на своем компьютере выполните команду
и после этого перейдите на страницу [localhost:8000](http://localhost:8000)
```shell
$  python -m mkdocs serve
```

Для сборки сайта документации выполните команду
```shell
$  python -m mkdocs build 
```

После этого созданный сайт будет помещен в папку `/site` в корне проекта

Более подробно о командах можно узнать на странице приложения - [Command Line Interface](https://www.mkdocs.org/user-guide/cli/) 