---
name: core2-ui-panel
description: "Build Core2 admin UI with the Panel class (панель Core2): titles, content, tabs, tab types/positions, controls and rendering. Use when laying out a module screen, tabs or panel toolbar."
---

# Панель Core2 (`Panel`)

## Когда использовать

- Нужен экран модуля с заголовком, подзаголовком, кнопкой «Назад».
- Нужны табы (обычные, pills, steps, боковые).
- Нужны кнопки/произвольные контролы в шапке панели.

Класс панели: `core2/inc/classes/Panel.php`. Путь и набор методов могут отличаться между версиями Core2 — проверяйте исходники конкретного проекта.

## Простая панель

```php
require_once DOC_ROOT . 'core2/inc/classes/Panel.php';

$panel = new \Panel();
$panel->setTitle('Конфигурация');
$panel->setContent('html content');

echo $panel->render();
```

## Панель с табами

```php
$base_url = 'index.php?module=your_mod&action=index';
$panel    = new \Panel('tab');

$panel->setTitle('Конфигурация', 'Описание', $base_url);

$panel->addTab('Настройки системы',        'system',   $base_url);
$panel->addTab('Дополнительные параметры', 'extra',    $base_url);
$panel->addTab('Персональные настройки',   'personal', $base_url);

$content = [];

switch ($panel->getActiveTab()) {
    case 'system':   $content[] = 'tab 1 html'; break;
    case 'extra':    $content[] = 'tab 2 html'; break;
    case 'personal': $content[] = 'tab 3 html'; break;
}

$panel->setContent(implode('', $content));

echo $panel->render();
```

Идентификатор панели (`new \Panel('tab')`) обязателен при наличии табов. Активный таб определяется query-параметром `&tab=<id>` через `getActiveTab()`.

## Вид и положение табов

```php
$panel->setTypeTabs($panel::TYPE_PILLS);   // или TYPE_STEPS
$panel->setPosition($panel::POSITION_LEFT); // TOP, LEFT, RIGHT, BOTTOM
$panel->setWidthTabs(240);                  // только для боковых табов
$panel->addDivider();                       // разделитель, только для боковых
```

## Контролы в шапке

```php
$panel->addControls('button_1', '<i class="fa fa-print"></i> Печать', 'onclick_btn1_func()');
$panel->addControls('button_2', 'Экспорт', 'onclick_btn2_func()');
$panel->addControlsCustom('<b>Custom html control</b>');
```

`addControls($id, $content, $js)` — кнопка; `$content` может содержать HTML. `addControlsCustom($html)` — произвольный HTML.

## Практика

- Собирайте контент активного таба в массив и склеивайте через `implode('', $content)`.
- Для боковых табов при длинных подписях задавайте `setWidthTabs()`.
- Панель — только layout: бизнес-логику держите в контроллере/сервисах, данные передавайте в HTML.
- Формы внутри панели стройте через `editTable` (см. `core2-forms`).

## Связанные скилы

- `core2-module-development` — `action_*` методы, куда встраивается панель.
- `core2-forms` — форма редактирования.
