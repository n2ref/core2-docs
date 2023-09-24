
Вся документация ведется в файлах формата [Markdown](https://ru.wikipedia.org/wiki/Markdown). 
Созданные файлы автоматически собираются в сайт с помощью приложения [MkDocs](https://www.mkdocs.org/)


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


## Настройка сайта документации

Основная информация о всех нюансах настройки расположена на сайте приложения:

- [Getting Started with MkDocs](https://www.mkdocs.org/getting-started/)
- [Writing your docs](https://www.mkdocs.org/user-guide/writing-your-docs/)
- [Getting started (Material)](https://squidfunk.github.io/mkdocs-material/creating-your-site/)

Все настройки проекта находятся в файле `/mkdocs.yml`  
Его основные параметры:

- **site_name** - Название проекта
- **site_url**  - Страница проекта
- **repo_name** - Название репозитория
- **repo_url**  - Страница репозитория
- **theme**     - Настройки темы
    - **name**     - [Имя темы](https://www.mkdocs.org/user-guide/choosing-your-theme/)
    - **features** - Список активных [функций](https://squidfunk.github.io/mkdocs-material/setup/setting-up-navigation/?h=navigation+sections) 
    - **palette**  - Тонкая настройка [внешнего вида](https://squidfunk.github.io/mkdocs-material/setup/changing-the-colors/) сайта (цвета, логотипы и тд.)
- **extra_javascript**    - Дополнительные `JS` скрипты добавленные на страницы
- **extra_css**           - Дополнительные `CSS` скрипты добавленные на страницы
- **plugins**             - Активные [плагины](https://squidfunk.github.io/mkdocs-material/reference/images/?h=glightbox#lightbox)
- **markdown_extensions** - Дополнительные [расширения](https://squidfunk.github.io/mkdocs-material/reference/content-tabs/#grouping-other-content-unordered-list) для парсинга markdown
- **nav**                 - Самая важная часть - [список страниц](https://www.mkdocs.org/getting-started/)
    - Все указанные в этом разделе пункты должны иметь название и путь до файла `.md`

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