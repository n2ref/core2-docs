Системный модуль `cron` предназначен для запуска фоновых задач по расписанию внутри Core2.

> Речь идет о задачах в модуле `cron` Core2, а не о Linux crontab.

## Откуда берутся задачи

Задачи формируются из CLI-классов модулей:

- файл: `Mod<ModuleName>Cli.php`
- класс: `Mod<ModuleName>Cli`
- метод: публичный метод класса

Пример для модуля `orders`:

```php
require_once DOC_ROOT . 'core2/inc/classes/CommonCli.php';

class ModOrdersCli extends \Core2\CommonCli {

    /**
     * Пересчет агрегатов по заказам
     */
    public function recalcAggregates(): void {
        // ...
    }
}
```

Описание метода (PHPDoc) используется в интерфейсе планировщика как название/описание задачи,
поэтому рекомендуется писать короткий и понятный комментарий.

## Ручной запуск через CLI

```shell
$ php core2/cli.php --module orders --action recalcAggregates
```

С параметрами:

```shell
$ php core2/cli.php --module orders --action exampleMethod --param 123 --param "foo bar"
```

Параметры `--param` передаются в метод по порядку.

Подробная справка:

```shell
$ php core2/cli.php --help
```

## Практические рекомендации

- Делайте задачи идемпотентными (повторный запуск не должен ломать данные).
- Логируйте прогресс и ошибки.
- Для долгих задач добавляйте ограничение по времени/батчи.
- Явно валидируйте входные параметры CLI-методов.
