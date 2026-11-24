---
name: core2-testing
description: "Write and review Core2 PHP framework module tests (тестирование модулей Core2): Tests folder layout, PHPUnit bootstrap/config, integration API tests over HTTP, PSR-7 helpers, fixtures, cleanup, skip rules and CI. Use when adding or reviewing tests for a module under mod/<module>/v<version>/Tests."
---

# Тестирование модулей Core2

## Когда использовать

- Нужно добавить тесты в модуль Core2.
- Нужно проверить или отревьюить существующий набор тестов.
- Нужно настроить запуск тестов локально или в CI.
- Нужно покрыть тестами REST-метод, CLI-команду или бизнес-логику модуля.

Скил самодостаточен и применим к любой установке Core2. Перед работой сверяйтесь с кодом проекта:
версии ядра, PHPUnit и набор зависимостей могут отличаться. Источник истины — код проекта.

## Базовые принципы

- Тесты лежат внутри модуля, в папке `Tests`, и версионируются вместе с кодом.
- Основной тип тестов — интеграционные: запрос к запущенному приложению по HTTP плюс проверка базы.
- Один тестовый класс описывает один REST-метод или одну логическую группу.
- Название теста отражает сценарий и ожидаемый результат.
- Тест не зависит от порядка запуска и от данных других тестов.
- Тест сам готовит данные и удаляет созданное после себя.
- При отсутствии подходящих данных тест пропускается, а не падает.
- Проверяются две вещи отдельно: ответ API (код, тело) и состояние базы.
- Вспомогательный код выносится в `Tests/Api/Support` и переиспользуется.

Не делайте тесты, которые проходят только на конкретной базе или только при ручной подготовке данных.

## Структура папки тестов

```text
Tests/
├─ bootstrap.php          # настройки, окружение, автозагрузчики
├─ phpunit.xml            # конфигурация PHPUnit
└─ Api/
   ├─ <Группа>/
   │  └─ <Метод>Test.php  # тесты одного REST-метода
   └─ Support/
      ├─ BaseApiTest.php  # общий предок тестов
      ├─ ApiClient.php    # HTTP-клиент
      ├─ SimpleResponse.php
      └─ SimpleStream.php
```

- Папки внутри `Api` повторяют структуру субмодулей.
- Имя файла: название проверяемого метода плюс суффикс `Test`.
- Все тесты из `Api` подключаются одной тестовой группой в `phpunit.xml`:

```xml
<testsuites>
    <testsuite name="Api Tests">
        <directory>./Api</directory>
    </testsuite>
</testsuites>
```

## Настройка окружения

Приоритет параметров:

1. переменные окружения;
2. значения из `phpunit.xml`;
3. секция `[production]` файла `conf.ini` в корне проекта.

| Параметр          | Назначение                                                     |
|-------------------|----------------------------------------------------------------|
| `API_BASE_URL`    | Базовый адрес приложения, например `http://example.local`      |
| `APIKEY`          | API-ключ, передается в заголовке `Core2-Apikey`                |
| `SERVER_NAME`     | Имя хоста, если не задан `API_BASE_URL`                        |
| `SERVER_PROTOCOL` | Протокол (`http` или `https`), если не задан `API_BASE_URL`    |

Пример `conf.ini`:

```ini
[production]
tests.api.server_name = example.local
tests.api.base_url    = http://example.local
tests.api.apikey      = <API-ключ>
```

`bootstrap.php` вычисляет итоговые значения, пробрасывает их в окружение, подключает
`core2/bootstrap.php` и автозагрузчик модуля. Если `APIKEY` не задан, тесты помечаются
как пропущенные.

## Пример bootstrap.php

```php
<?php

/**
 * Настройки тестов берутся по приоритету:
 *   1. переменные окружения (env);
 *   2. переменные из phpunit.xml (PHPUnit применяет их до bootstrap);
 *   3. секция [production] файла conf.ini в корне проекта.
 *
 * В conf.ini, только в секции [production], можно задать:
 *   tests.api.server_name = example.local
 *   tests.api.base_url    = http://example.local
 *   tests.api.apikey      = <API-ключ>
 */

$config = [
    'SERVER_NAME'     => getenv('SERVER_NAME') ?: ($_SERVER['SERVER_NAME'] ?? null),
    'SERVER_PROTOCOL' => getenv('SERVER_PROTOCOL') ?: ($_SERVER['SERVER_PROTOCOL'] ?? null),
    'API_BASE_URL'    => getenv('API_BASE_URL') ?: null,
    'APIKEY'          => getenv('APIKEY') ?: null,
];

// Путь до conf.ini зависит от глубины вложенности модуля.
$conf_file = dirname(__DIR__, 4) . '/conf.ini';
$conf_prod = [];

if (is_file($conf_file)) {
    $sections  = parse_ini_file($conf_file, true, INI_SCANNER_RAW) ?: [];
    $conf_prod = is_array($sections['production'] ?? null) ? $sections['production'] : [];
}

$config['SERVER_NAME']     = $config['SERVER_NAME']     ?: ($conf_prod['tests.api.server_name'] ?? null) ?: 'dev';
$config['SERVER_PROTOCOL'] = $config['SERVER_PROTOCOL'] ?: (! empty($conf_prod['tests.api.https']) ? 'https' : null) ?: 'http';
$config['API_BASE_URL']    = $config['API_BASE_URL']    ?: ($conf_prod['tests.api.base_url'] ?? null);
$config['APIKEY']          = $config['APIKEY']          ?: ($conf_prod['tests.api.apikey'] ?? null);
$config['API_BASE_URL']    = $config['API_BASE_URL']    ?: "{$config['SERVER_PROTOCOL']}://{$config['SERVER_NAME']}";

// Пробрасываем итоговые значения в окружение, чтобы их видели
// core2/bootstrap.php и базовый класс тестов.
foreach ($config as $name => $value) {
    if ($value === null || $value === '') {
        continue;
    }

    putenv("{$name}={$value}");
    $_ENV[$name] = $value;
}

$_SERVER['SERVER_NAME']     = $config['SERVER_NAME'];
$_SERVER['SERVER_PROTOCOL'] = $config['SERVER_PROTOCOL'];

require_once __DIR__ . '/../../../../core2/bootstrap.php';
require_once __DIR__ . '/../autoload.php';
```

Что важно в примере:

- `force="false"` в `phpunit.xml` нужен, чтобы переменные окружения имели приоритет над значениями файла.
- `API_BASE_URL` собирается из `SERVER_PROTOCOL` и `SERVER_NAME`, если не задан явно.
- Пустые значения не пробрасываются в окружение.
- Относительные пути до `core2/bootstrap.php` и `autoload.php` зависят от расположения модуля —
  проверьте их для конкретного проекта.

## Запуск

Скрипт в `composer.json` модуля:

```json
{
    "scripts": {
        "test-api": "../../../core2/vendor/phpunit/phpunit/phpunit -c ./Tests"
    }
}
```

```shell
composer run test-api
../../../core2/vendor/phpunit/phpunit/phpunit -c ./Tests
phpunit -c ./Tests Tests/Api/<Группа>/<Метод>Test.php
phpunit -c ./Tests --filter <Сценарий>
```

Если CI проверяет только синтаксис PHP, интеграционные тесты запускаются вручную.

## Базовый класс тестов

Общий предок собирает адрес и ключ, создает HTTP-клиент и дает доступ к базе.

```php
abstract class BaseApiTest extends TestCase {

    protected ApiClient $api;
    protected \Zend_Db_Adapter_Abstract $db;

    protected function setUp(): void {
        $base_url = getenv('API_BASE_URL') ?: $this->buildBaseUrlFromHost();
        $apikey   = getenv('APIKEY');

        if (empty($apikey)) {
            $this->markTestSkipped('Не указана настройка APIKEY');
        }

        $this->api = new ApiClient($base_url, ['Core2-Apikey' => $apikey]);
        $this->db  = (new \Core2\Db())->db;
    }

    protected function assertResponseStatusCodeSame(int $expected, SimpleResponse $response): void {
        $this->assertSame($expected, $response->getStatusCode(), $response->toString());
    }
}
```

Требования к базовому классу:

- доступ к базе только для подготовки данных и проверок;
- общие утверждения (код ответа, JSON-тело) — в базовом классе;
- никакой бизнес-логики конкретного модуля.

## Скелет тестового класса

```php
final class <Метод>Test extends BaseApiTest {

    private const URI = '/<submodule>/<method>/%s';
    private array $created_ids = [];

    protected function tearDown(): void {
        if ($this->created_ids !== []) {
            $this->db->delete('<table>', $this->db->quoteInto('id IN (?)', $this->created_ids));
            $this->created_ids = [];
        }
        parent::tearDown();
    }

    public function testSuccessfulCallReturnsExpectedResult(): void {
        $fixture = $this->findFixture();

        if ( ! $fixture) {
            $this->markTestSkipped('Нет данных для сценария');
        }

        $response = $this->api->request('POST', sprintf(self::URI, $fixture['id']));

        $this->assertResponseStatusCodeSame(200, $response);

        $data = json_decode((string)$response->getBody(), true, 512, JSON_THROW_ON_ERROR);

        $this->assertArrayHasKey('id', $data);
        $this->assertGreaterThan(0, $data['id']);

        $row = $this->findById((int)$data['id']);
        $this->assertNotNull($row);
        $this->assertSame($fixture['id'], $row['source_id']);

        $this->created_ids[] = (int)$row['id'];
    }
}
```

Правила именования тестов — по смыслу сценария, например:

- `testSuccessfulCallReturnsExpectedResult`;
- `testCreatedRecordIsFilledFromSource`;
- `testRepeatedCallDoesNotCreateDuplicate`;
- `testExistingRecordIsReturned`;
- `testUnknownIdReturnsError`;
- `testInvalidParameterReturnsNotFound`;
- `testWrongHttpMethodReturnsNotFound`;
- `testMissingApiKeyReturnsUnauthorized`.

## Типовые сценарии

| Сценарий                         | Что проверяется                                                         |
|----------------------------------|--------------------------------------------------------------------------|
| Успешный вызов                   | Ожидаемый код ответа и структура тела                                    |
| Создание сущности                | Идентификатор в ответе, запись в базе с ожидаемыми полями                |
| Повторный вызов                  | Нет дубля, возвращается тот же результат                                 |
| Вызов для существующей сущности  | Возвращается существующая запись, количество записей не меняется         |
| Неизвестный идентификатор        | Ошибка с ожидаемым кодом и сообщением                                    |
| Недопустимые символы в параметре | Маршрут не совпадает, возвращается ошибка                                |
| Неверный HTTP-метод              | Возвращается ошибка, действие не выполняется                             |
| Нет прав или ключа               | Возвращается ошибка авторизации                                          |

## Данные, фикстуры и очистка

- Ищите данные запросом по условиям, а не по фиксированному идентификатору.
- Для «неизвестного» значения генерируйте уникальное значение, которого заведомо нет в базе.
- Созданные записи запоминайте и удаляйте в `tearDown()`.
- Удаляйте по сохраненным идентификаторам, а не по широкому условию.
- Не оставляйте мусор в базе, даже если утверждение упало.
- Если подходящей записи нет — `markTestSkipped`, а не `assert`.

```php
private function findFixture(): ?array {
    $row = $this->db->fetchRow("
        SELECT t.*
        FROM <table> AS t
        LEFT JOIN <related_table> AS r ON r.<key> = t.<key>
        WHERE t.<key> IS NOT NULL
          AND r.id IS NULL
        LIMIT 1
    ");

    return $row ?: null;
}
```

## Вспомогательные классы

- **`ApiClient`** — минимальный HTTP-клиент на `file_get_contents()` с `stream_context_create()`.
  Принимает метод, URI, query, заголовки и тело; массив тела кодируется в JSON; возвращает
  `SimpleResponse`. `ignore_errors => true`, чтобы получать ответы с кодами 4xx/5xx, а не исключения.
- **`SimpleResponse`** — неизменяемая реализация `Psr\Http\Message\ResponseInterface`; хранит код,
  заголовки и тело; `toString()` форматирует JSON и обрезает длинное тело.
- **`SimpleStream`** — реализация `Psr\Http\Message\StreamInterface` для тела ответа.

Не подключайте внешние HTTP-клиенты, если в проекте уже есть такой набор. Расширяйте существующие
классы, а не создавайте параллельные.

## Чек-лист нового теста

- [ ] Класс лежит в `Tests/Api/<Группа>` и назван по проверяемому методу.
- [ ] Сценарий отражен в названии тестового метода.
- [ ] Проверяются и ответ API, и состояние базы.
- [ ] Данные готовятся внутри теста.
- [ ] При отсутствии данных тест пропускается.
- [ ] Созданные записи удаляются в `tearDown()`.
- [ ] Тест проходит и в общем наборе, и отдельно.
- [ ] Тест не требует ручной подготовки базы.

## Частые ошибки

- Тест проверяет только код `200` и не смотрит в базу.
- Фикстуры берутся по фиксированному `id`, которого может не быть на другом стенде.
- Тест создает записи и не удаляет их.
- Проверка ответа через исключение: клиент падает на `400`, хотя тест ждет `400`.
- Тест зависит от порядка запуска других тестов.
- В тест попадает бизнес-логика модуля вместо вызова API.

## Связанные скилы

- `core2-module-development` — структура модуля и REST-контроллеры.
- `core2-database-conventions` — имена таблиц и полей для запросов и фикстур.
- `mkdocs-docs` — страница документации о тестировании в `docs/other/tests.md`.
