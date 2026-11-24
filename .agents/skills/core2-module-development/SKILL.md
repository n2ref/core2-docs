---
name: core2-module-development
description: "Develop Core2 PHP framework modules (модули Core2): module layout, Mod*Controller action_* methods, Zend_Db table models, ModAjax save handlers, CLI, REST API, conf.ini and core interfaces. Use when creating, extending or reviewing a module under mod/<module>/v<version>/."
---

# Разработка модулей Core2

## Когда использовать

- Создание нового модуля или новой версии существующего.
- Правки контроллера, обработчика сохранения, CLI, REST API, моделей таблиц.
- Подключение возможностей ядра через интерфейсы (`Delete`, `Subscribe`, `Navigation` и др.).

Скил самодостаточен и применим к любой установке Core2. Сверяйте поведение с исходниками конкретного проекта: ядро в `core2/inc/`, код модуля в `mod/`. Версии Core2 могут отличаться, поэтому при расхождении источник истины — код проекта, а не этот текст.

## Структура модуля

Проектные модули лежат в `mod/<module>/v<version>/`, системные — в `core2/mod/<module>/v<version>/`. Каждая версия самодостаточна; одновременно активна только одна версия, указанная в таблице модулей. Признак «системный» задаётся в настройках модуля.

```text
mod/orders/v1.0.0/
├─ Api/                          # schema.json и классы API
├─ assets/                       # js, css, html
├─ classes/                      # прикладные PHP-классы модуля
├─ Model/                        # модели таблиц модуля
├─ ModAjax.php                   # сохранение форм
├─ ModOrdersApi.php              # REST-контроллер
├─ ModOrdersCli.php              # CLI/планировщик
├─ ModOrdersController.php       # контроллер интерфейса
├─ conf.ini                      # конфиг модуля
└─ composer.json                 # зависимости модуля
```

Имена файлов и классов:

| Файл | Класс | Назначение |
|---|---|---|
| `Mod<Name>Controller.php` | `Mod<Name>Controller` | интерфейс модуля, обязателен |
| `ModAjax.php` | `ModAjax` | обработка сохранения форм |
| `Mod<Name>Cli.php` | `Mod<Name>Cli` | CLI и задачи планировщика |
| `Mod<Name>Api.php` | `Mod<Name>Api` | REST API |
| `Model/<Table>.php` | `<Table>` | модель таблицы |

## Контроллер

- Наследуется от `Common`.
- Методы экранов: `action_<submodule>`, главная страница — `action_index`.
- Главная страница: `index.php#?module=orders`; субмодуль: `index.php#?module=orders&action=example`.
- Субмодуль должен быть зарегистрирован в таблице субмодулей, иначе метод недоступен.
- В docblock класса рекомендуется перечислять модели: `@property \Orders $dataOrders`.
- Публичные не-`action` методы можно вызывать из других модулей.

```php
/**
 * @property \Orders $dataOrders
 */
class ModOrdersController extends Common {

    public function action_index(): string {
        return 'text content';
    }
}
```

### Интерфейсы ядра

Реализуйте только нужные интерфейсы. Методы вызываются ядром в нужной точке жизненного цикла.

| Интерфейс | Обязательный метод | Назначение |
|---|---|---|
| `Delete` | `action_delete($resource, $id)` | кастомное удаление; `true` — сделано вручную, `false` — стандартная процедура |
| `Switches` | `action_switch(string $resource_name, string $field, string $id, string $value)` | переключатели в таблицах |
| `File` | `action_filehandler($context, $table, $id)` | выдача файлов для полей форм |
| `Navigation` | `navigationItems(\Core2\Navigation $nav)` | пункты главного меню |
| `TopCss` | `topCss(): array` | глобальные CSS |
| `TopJs` | `topJs(): array` | глобальные JS |
| `Subscribe` | `listen($module_id, $event, $data)` | подписка на события других модулей |
| `Event` | `check(): bool`, `dispatch()` | генерация модульных событий |
| `RestController` | `setVersion`, `setResource`, `setQuery`, `setBody`, `dispatch` | отдельные REST-контроллеры |
| `Sequence` | `sequence(string $resource_name, array $records): bool` | кастомная сортировка строк |

Интерфейсы находятся в `core2/inc/Interfaces`. Набор и сигнатуры могут отличаться между версиями Core2 — проверяйте исходники проекта.

## Модели таблиц

- Наследование: `\Zend_Db_Table_Abstract`.
- Обязательно указать `protected $_name = 'mod_orders_products';`.
- Имя класса/файла — имя таблицы без префикса `mod_` в CamelCase.
- Методы выборки: `getRowBy<Field>` / `getRowsBy<Field>`, например `getRowById`.
- Внутри модели работайте только с её таблицей.

```php
class OrdersProducts extends Zend_Db_Table_Abstract {

    protected $_name = 'mod_orders_products';

    public function getRowById(int $id): ?Zend_Db_Table_Row_Abstract {
        return $this->fetchRow($this->select()->where('id = ?', $id));
    }
}
```

## Обработчик сохранения (`ModAjax.php`)

- Наследуется от `ajaxFunc`; подключается через `require_once DOC_ROOT . "core2/inc/ajax.func.php";`.
- Публичные методы начинаются с `ax` и вызываются формой при сохранении.
- Сохраняемые данные — в `$data['control']`; служебные — `$data['class_id']`, `$data['class_refid']` и др.
- Ключи `$data['control']` должны совпадать с полями таблицы.

Валидация:

```php
if ($this->ajaxValidate($data, [
    'form_field1' => 'req',
    'form_field2' => 'req,int',
    'form_field3' => 'req,email',
])) {
    return $this->response;
}
```

Правила: `req`, `int`, `float`, `email`. Очистка — `$this->clearData($data['control'])`. Данные из сессии — `$this->getSessFormField($data['class_id'], 'refid')`.

Сохранение и ответ:

```php
$this->db->beginTransaction();
try {
    $row = $this->saveDataModel($this->modOrders->dataOrders, $data['control']);
    $this->response->script("CoreUI.notice.create('Сохранено');");
    $this->db->commit();
} catch (Exception $e) {
    $this->db->rollBack();
    $this->error[] = $e->getMessage();
}

$this->done($data);          // успех
// $this->displayError($data); // ошибка, после заполнения $this->error
return $this->response;
```

## CLI (`Mod<Name>Cli.php`)

- Наследуется от `\Core2\CommonCli`; подключается через `core2/inc/classes/CommonCli.php`.
- Публичный метод принимает параметры по порядку; docblock используется как название задачи в модуле `cron`.
- Запуск: `php core2/cli.php --module orders --action exampleMethod --param 123 --param "foo bar"`.
- Справка: `php core2/cli.php --help`.

## REST API (`Mod<Name>Api.php`)

- Наследуется от `CommonApi`; методы `action_<submodule>` / `action_index`.
- URL: `https://host.com/orders/index`, `https://host.com/orders/example`.
- Субмодуль должен быть зарегистрирован, иначе метод недоступен.
- Внутри — `\Core2\Routing\Router` и `$this->runRouter($router)`.

```php
public function action_index(): mixed {
    $router = new \Core2\Routing\Router('/orders/index', [
        ''                  => ['get' => [Api\Index::class, 'getOrders']],
        '/order'            => ['post' => [Api\Order::class, 'createOrder']],
        '/order/{order_id}' => [
            'get'    => [Api\Order::class, 'getOrder'],
            'patch'  => [Api\Order::class, 'patchOrder'],
            'delete' => [Api\Order::class, 'deleteOrder'],
        ],
    ]);

    return $this->runRouter($router);
}
```

Роуты описываются в `Api/schema.json` в формате OpenAPI и доступны в `core2/swagger/index.html` и `core2/scalar/index.html`.

## Конфиг модуля

`mod/<module>/v<version>/conf.ini`, секции окружений и наследование:

```ini
[production]
feature.enabled = true
api.timeout = 10

[project.com : production]
feature.enabled = false
```

Не храните секреты в репозитории; для критичных параметров добавляйте валидацию при инициализации.

## Установка и проверка

1. Разместить файлы в `mod/<module>/v<version>/` для проектного модуля или в `core2/mod/<module>/v<version>/` для системного (либо загрузить архив в `Admin -> Модули -> Доступные модули`).
2. При необходимости создать таблицы БД.
3. В `Admin -> Модули` создать запись: `module_id`, версия, системный/проектный, отображение в меню, доступ по умолчанию.
4. Проверить права ролей.
5. Smoke-check: список, форма, сохранение, удаление, API/CLI при наличии.

Минимум структуры: `Mod<Name>Controller.php`, `ModAjax.php`, `conf.ini`, `classes/`, `Model/`.

## Связанные скилы

- `core2-database-conventions` — схема и имена таблиц.
- `core2-forms` — построение форм `editTable`.
- `core2-ui-panel` — панель и табы интерфейса.
