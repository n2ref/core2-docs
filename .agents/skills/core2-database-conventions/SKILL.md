---
name: core2-database-conventions
description: "Core2 database naming and schema conventions (база данных Core2): table prefixes, id/foreign keys, is_*_sw booleans, DECIMAL, date_created/date_modify, indexes, foreign keys and file-table DDL. Use when designing or reviewing Core2 tables, migrations or Zend_Db models."
---

# Соглашения по базе данных Core2

## Когда использовать

- Проектирование новых таблиц модуля.
- Ревью DDL, имён полей, индексов и внешних ключей.
- Создание моделей `Zend_Db_Table_Abstract` и запросов к связанным таблицам.

Соглашения применимы к любой установке Core2. Перед изменением схемы проверяйте существующие таблицы, версию ядра и принятые в проекте миграции.

## Поля таблицы

1. Имена полей — только строчные латинские буквы, цифры и `_`.
2. У таблицы, за редкими исключениями, есть `id int unsigned NOT NULL AUTO_INCREMENT`.
3. Числовые поля-связи:
   - оканчиваются на `_id` (`user_id`);
   - по возможности идут в начале таблицы;
   - имеют простой индекс и foreign key.
4. Булевы поля:
   - имя `is_<something>_sw` (`is_active_sw`);
   - тип `enum('Y','N')`;
   - значение по умолчанию (`DEFAULT 'Y'` или `'N'`);
   - желательно `NOT NULL`, чтобы не появлялось третье состояние `NULL`.
5. Дробные числа — тип `DECIMAL`.
6. Служебные даты:
   - `date_created` — `timestamp NOT NULL DEFAULT current_timestamp()`;
   - `date_modify` — `timestamp NOT NULL DEFAULT current_timestamp() ON UPDATE current_timestamp()`;
   - рекомендуется располагать в конце таблицы.
7. Мягкое удаление — `is_deleted_sw enum('Y','N') NOT NULL DEFAULT 'N'`.
8. Активность — `is_active_sw enum('Y','N') NOT NULL DEFAULT 'Y'`.

## Имена таблиц

- Таблицы модуля начинаются с `mod_<module>`: `mod_orders`, `mod_orders_products`.
- Таблицы файлов оканчиваются на `_files`: `mod_orders_files`.
- Модель таблицы `mod_orders_products` — класс и файл `OrdersProducts` (CamelCase без `mod_`).

## Индексы

- Простой индекс называется так же, как поле.
- Составной индекс — имена полей через `_`: `user_id_role_id`.

## Foreign key

Всегда продумывайте `ON UPDATE` и `ON DELETE`. Для файловых таблиц обычно `ON DELETE CASCADE ON UPDATE CASCADE`.

## Таблицы с файлами

Рекомендуемая DDL-структура, на которую рассчитан встроенный функционал системы:

```sql
CREATE TABLE `mod_orders_files` (
   `id` int unsigned NOT NULL AUTO_INCREMENT,
   `content` longblob,
   `refid` int unsigned NOT NULL,
   `filename` varchar(255) NOT NULL,
   `filesize` int NOT NULL,
   `hash` varchar(128) NOT NULL,
   `type` varchar(255) DEFAULT NULL,
   `fieldid` varchar(255) DEFAULT NULL,
   `thumb` longblob,
   `storage` varchar(255) DEFAULT NULL,
   PRIMARY KEY (`id`),
   KEY `refid` (`refid`),
   CONSTRAINT `fk1_mod_orders_files` FOREIGN KEY (`refid`) REFERENCES `mod_orders` (`id`) ON DELETE CASCADE ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

## Пример таблицы модуля

```sql
CREATE TABLE `mod_orders` (
   `id` int unsigned NOT NULL AUTO_INCREMENT,
   `user_id` int unsigned NOT NULL,
   `name` varchar(255) NOT NULL,
   `amount` decimal(15,2) NOT NULL DEFAULT 0.00,
   `is_active_sw` enum('Y','N') NOT NULL DEFAULT 'Y',
   `is_deleted_sw` enum('Y','N') NOT NULL DEFAULT 'N',
   `date_created` timestamp NOT NULL DEFAULT current_timestamp(),
   `date_modify` timestamp NOT NULL DEFAULT current_timestamp() ON UPDATE current_timestamp(),
   PRIMARY KEY (`id`),
   KEY `user_id` (`user_id`),
   CONSTRAINT `fk1_mod_orders_user_id` FOREIGN KEY (`user_id`) REFERENCES `mod_users` (`id`) ON DELETE RESTRICT ON UPDATE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

## Связанные скилы

- `core2-module-development` — модели таблиц и структура модуля.
- `core2-forms` — поля формы, соответствующие полям таблицы.
