Класс `Db` отвечает за подключение к базе данных, создание моделей и работу с конфигурацией БД.

## Базовая конфигурация

Минимальный пример (`conf.ini`):

```ini
[production]
database.adapter          = Pdo_Mysql
database.params.host      = 127.0.0.1
database.params.port      = 3306
database.params.dbname    = "project_com"
database.params.username  = "db_user"
database.params.password  = "db_pass"
```

Для PostgreSQL:

```ini
[production]
database.adapter          = Pdo_Pgsql
database.params.host      = 127.0.0.1
database.params.port      = 5432
database.params.dbname    = "project_com"
database.params.username  = "db_user"
database.params.password  = "db_pass"
database.schema           = public
```

## Получение соединения

В прикладном коде обычно используется `$this->db` (инициализируется ядром).

Создание отдельного подключения:

```php
/** @var Db $dbService */
$db = $dbService->newConnector('project_com', 'db_user', 'db_pass', '127.0.0.1:3306');
```

## Замечания

- Поддерживаются адаптеры `Pdo_Mysql` и `Pdo_Pgsql`.
- Подключения/модели модулей кэшируются в Registry.
- Часовой пояс и SQL mode могут применяться автоматически в зависимости от конфигурации.
