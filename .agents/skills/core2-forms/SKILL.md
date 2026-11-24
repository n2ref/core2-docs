---
name: core2-forms
description: "Build Core2 admin forms with editTable (формы Core2): addControl field types, groups, buttons, setTable/setData, save handlers, validation and success responses. Use when creating or editing module forms, field sets or save flow."
---

# Формы Core2 (`editTable`)

## Когда использовать

- Построение формы редактирования в контроллере модуля.
- Выбор типа поля (`text`, `select2`, `date`, `xfiles` и т.д.).
- Связка формы с обработчиком сохранения `ModAjax`.
- Настройка групп, кнопок, readonly и сообщений об успехе.

Класс формы: `core2/inc/classes/class.edit.php`; обработчик сохранения: `ModAjax.php` модуля и `core2/inc/ajax.func.php`. Набор типов и параметров может отличаться между версиями Core2 — сверяйтесь с исходниками конкретного проекта.

## Базовый каркас

```php
require_once DOC_ROOT . 'core2/inc/classes/class.edit.php';

$edit = new \editTable('storages_suppliers');

$edit->setTable('mod_storages_suppliers')
     ->setWidthLabels(200)
     ->setSaveHandler('saveSupplierData')
     ->setSuccessUrl('index.php?module=storages&suppliers')
     ->addSuccessNotice('Поставщик успешно сохранен')
     ->addSuccessScript('console.log("Сохранено");');

$edit->setData([
    'id'        => 123,
    'name'      => 'ООО Тест',
    'is_active' => 'Y',
]);

$edit->addControl('Название компании', $edit::TYPE_TEXT, '', '', '', true);
$edit->addControl('Активен',           $edit::TYPE_SWITCH);

echo $edit->render();
```

`addControl($name, $type, $in = "", $out = "", $default = "", $req = false)`:

- `$name` — заголовок поля;
- `$type` — строковый тип (`'text'`) или константа (`$edit::TYPE_TEXT`);
- `$in` — атрибуты/данные поля;
- `$out` — внешний HTML;
- `$default` — значение по умолчанию;
- `$req` — обязательность.

## Основные методы

| Метод | Назначение |
|---|---|
| `setTable(string $table)` | таблица БД формы |
| `setData(array $record)` | данные редактируемой записи |
| `addControl(...)` | добавить поле |
| `addGroup($name, $collapsed = false)` | сворачиваемая группа полей |
| `addButton($value, $action = '')` | стандартная кнопка |
| `addButtonSwitch($field_name, $value)` | переключатель Y/N |
| `addButtonCustom($html)` | произвольная кнопка |
| `setReadonly(bool)` | вся форма только для чтения |
| `setReadonlyFields(array)` / `addReadonlyFields(array)` / `clearReadonlyFields()` | управление readonly-полями |
| `setWidthLabels(string\|int)` | ширина подписей |
| `setSaveHandler(string)` | имя обработчика сохранения |
| `setSessFormField($id, $value)` | значение в сессию формы |
| `setLeaveChecking(bool)` | предупреждение при уходе со страницы |
| `setSuccessUrl(string)` | переход после сохранения |
| `addSuccessNotice(string)` | уведомление после сохранения |
| `addSuccessScript(string)` | JS после сохранения (`saveSuccess()` устарел) |
| `addParams($name, $value)` | доп. параметр формы |
| `setTemplate($html)` / `setTemplateControl($type, $file)` | кастомные шаблоны |
| `render(array $options = [])` | вернуть HTML формы |

## Типы полей

Тип передаётся строкой (`'text'`) или константой (`$edit::TYPE_TEXT`). Типовые имена типов (проверяйте в `class.edit.php`):

- текстовые: `text`, `textarea`, `password`, `fck`, `hidden`, `protect`, `custom`;
- числовые: `number`, `number_range`, `money`;
- даты и время: `date`, `datetime`, `date2`, `datetime2`, `datetime_local`, `date_week`, `date_month`, `time`, `daterange`;
- выбор: `select`, `select2`, `multiselect2`, `combobox`, `radio`, `radio2`, `checkbox`, `checkbox2`, `switch`, `tags`;
- списки: `list`, `list_hidden`, `multilist`, `multilist2`, `multilist3`, `dataset`;
- файлы: `file`, `xfile`, `xfiles`;
- связи и модальные окна: `link`, `search`, `modal`, `modal2`, `modal_list`;
- прочее: `color`, `coordinates`, `edit`.

Модификаторы типа: `_hidden`, `_auto`, `_default`, например `text_hidden`, `date_auto`.

Примеры:

```php
$edit->addControl('Поле', 'text');
$edit->addControl('Поле', 'select2');
$edit->addControl('Дата', 'date_auto');
$edit->addControl('Файлы', 'xfiles');
```

Для конкретных параметров `in`, `attr`, `readonly` и сохранения смотрите `core2/inc/classes/class.edit.php` и примеры использования в модулях проекта.

## Связка с сохранением

1. `setSaveHandler('saveSupplierData')` указывает обработчик формы.
2. Обработчик реализуется в `ModAjax.php` (методы начинаются с `ax`, класс наследует `ajaxFunc`).
3. Данные приходят в `$data['control']`; ключи должны совпадать с полями таблицы.
4. Валидация — `$this->ajaxValidate($data, ['field' => 'req,int'])`, правила: `req`, `int`, `float`, `email`.
5. Ответ — `$this->done($data)` при успехе, `$this->displayError($data)` после заполнения `$this->error`.

Подробный контракт обработчика — в скиле `core2-module-development`.

## Практика

- Держите поля формы в том же порядке, что и в задаче пользователя.
- Для обязательных полей используйте `$req = true`, не дублируйте проверку только на клиенте.
- Служебные значения (например `refid`) передавайте через `setSessFormField()`, а не скрытыми полями.
- После сохранения возвращайте понятное уведомление и, при необходимости, URL/скрипт обновления.

## Связанные скилы

- `core2-module-development` — контроллер, `ModAjax`, модели.
- `core2-database-conventions` — имена и типы полей таблиц.
- `core2-ui-panel` — размещение формы внутри панели.
