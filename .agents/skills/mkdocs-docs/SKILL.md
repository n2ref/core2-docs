---
name: mkdocs-docs
description: "Maintain the core2-docs repository (документация Core2, MkDocs Material): add pages, edit mkdocs.yml nav, use Russian Markdown/tabs, run mkdocs serve/build, avoid editing generated site/. Use only for changes under docs/ or mkdocs.yml in this repository."
---

# Документация Core2 (MkDocs)

> Область применения: только репозиторий `core2-docs` (текущая папка). Скил не предназначен для переноса в другие проекты.

## Когда использовать

- Добавление или правка страницы в `docs/`.
- Изменение меню/настроек в `mkdocs.yml`.
- Локальный просмотр или сборка сайта документации.
- Проверка ссылок, картинок и структуры разделов.

Источники: `README.md`, `docs/other/docs.md`, `mkdocs.yml`.

## Структура репозитория

```text
.
├─ mkdocs.yml          # конфиг сайта и nav
├─ docs/               # исходники страниц
│  ├─ index.md
│  ├─ setup/
│  ├─ framework/
│  ├─ modules/
│  ├─ best_practice/
│  ├─ other/
│  └─ assets/          # css/js/картинки
└─ site/               # собранный сайт, генерируется mkdocs, в git не хранится
```

`docs/` — единственный источник контента. Не редактируйте `site/` вручную: он пересоздаётся при сборке и игнорируется git.

## Установка и запуск

```shell
pip install mkdocs-material
pip install mkdocs-glightbox

python -m mkdocs serve   # http://localhost:8000
python -m mkdocs build   # результат в site/
```

Тема — Material, язык `ru`; плагины: `search`, `offline`, `glightbox`; расширения: `pymdownx.superfences`, `pymdownx.tabbed` (`alternate_style`).

## Добавление страницы

1. Создайте файл в нужном разделе `docs/`, например `docs/modules/queue.md`.
2. Добавьте пункт в `nav` в `mkdocs.yml`. Путь указывается относительно `docs/`.
3. Подписи меню — на русском, как в существующих пунктах.
4. Выполните `python -m mkdocs build` и убедитесь, что сборка прошла без ошибок.

Пример `nav`:

```yaml
nav:
  - Введение: index.md
  - Структура модулей:
    - Обзор: modules/review.md
    - Очередь: modules/queue.md
```

Правила:

- Файлы и папки — строчные, `snake_case` (`datetime_local.md`, `users_roles.md`).
- Каждая страница, добавленная в `nav`, должна существовать; битый путь ломает/портит навигацию.
- Новый раздел — новая папка в `docs/` плюс группа в `nav`.

## Оформление страниц

- Текст документации — на русском.
- Первый заголовок страницы — `# Название` (кроме `index.md`, где исторически `##`).
- Блоки кода — с указанием языка: `php`, `sql`, `ini`, `shell`, `text`, `yaml`, `json`.
- Внутренние ссылки — относительные пути между `.md`-файлами.
- Картинки — рядом со страницей (например `docs/framework/classes/img/panel_example.png`), вставка `![alt](img/file.png)`; glightbox открывает их на весь экран.
- Не используйте расширения Markdown, которых нет в `markdown_extensions` (например admonition не подключён).

### Табы

Используется `pymdownx.tabbed` с `alternate_style`. Содержимое таба — с отступом 4 пробела:

````markdown
=== "Пример"

    ```php
    echo 'ok';
    ```

=== "С комментариями"

    ```php
    // комментарий
    echo 'ok';
    ```
````

## Проверка перед завершением

- [ ] Страница лежит в правильной папке `docs/`.
- [ ] Пункт добавлен в `nav` (если страница новая).
- [ ] Команды, классы и пути соответствуют реальному коду Core2, а не догадкам.
- [ ] Нет ссылок на несуществующие файлы/картинки.
- [ ] `python -m mkdocs build` завершается успешно.
- [ ] `site/` не добавлен в коммит.

## Связанные скилы

- `core2-module-development`, `core2-database-conventions`, `core2-forms`, `core2-ui-panel` — предметное содержание страниц.
