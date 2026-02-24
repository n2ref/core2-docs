## Минимальные системные требования к серверу

* 1 CPU, 515 Mb RAM, 1Gb HDD
* PHP 7.4 или больше
* PHP расширения `zip`, `mbstring`, `curl`, `gd`, `intl`, `PDO`, `pdo_mysql`, `xml`, `fpm`
* MySQL или MariaDb база данных

## Установка компонентов на Linux (Debian / Ubuntu)

Выполните консольные команды для обновления системы
```shell
$ sudo apt update
$ sudo apt upgrade
```

Выполните установку php и требуемых расширений

```shell
$ sudo apt install php8.0 php8.0-{zip,mbstring,curl,gd,intl,mysql,xml,fpm}
```

Выполните установку nginx или apache2

Установка nginx

```shell
$ sudo apt install nginx
```

Установка apache2

```shell
$ sudo apt install apache2
```

Выполните установку Mysql или MariaDb.

Установка Mysql для 
[Debian 11](https://www.cloudbooklet.com/how-to-install-mysql-on-debian-11/), 
[Debian 10](https://www.digitalocean.com/community/tutorials/how-to-install-the-latest-mysql-on-debian-10),
[Ubuntu 22.04](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-22-04)


Установка MariaDb
```shell
$ sudo apt install mariadb-server
```



## Установка Core2
1) Скачайте архив c последней версией на [github.com](https://github.com/n2ref/core2/archive/refs/heads/master.zip)

2) В директории `DOCUMENT_ROOT` вашего проекта создайте папку с названием `core2` и разместите в нее содержимое архива

3) Создайте еще 2 папки `cache` и `logs` за пределами `DOCUMENT_ROOT` 

Например чтобы получилось вот такая структура 
```text
 .
 ├─ /var/www/project.com/
 │  ├─ htdocs/
 │  │  └─ core2/
 │  │
 │  ├─ cache/  
 │  └─ logs/  
 
```

4) Дайте права доступа на чтение и запись в папки `cache` и `logs` для пользователя www-data
```shell
$ chmod 755 /var/www/project.com/cache/
$ chmod 755 /var/www/project.com/logs/

$ chown -R www-data:www-data /var/www/project.com/cache/
$ chown -R www-data:www-data /var/www/project.com/logs/
```

5) В `DOCUMENT_ROOT` создайте файл `index.php` со следующем содержимым
```php
try {
  	require_once("core2/inc/classes/Error.php");
  	require_once("core2/inc/classes/Init.php");
 
  	$init = new Init();
  	$init->checkAuth();
 
  	echo $init->dispatch();
} catch (Exception $e) {
  	\Core2\Error::catchException($e);
}
```

6) Создайте новую базу данных с любым названием на вашем MySQL / MariaDb сервере. 
   В данном примере она будет названа `project_com`.  
```sql
CREATE SCHEMA project_com COLLATE utf8mb4_general_ci;
```

7) Скачайте и выполните этот [sql](https://github.com/n2ref/core2/blob/master/db.sql) скрипт, чтобы в вашей базе данных добавились необходимые таблицы.
   Для этого можно воспользоваться командной строкой.
```shell
$ mysql -u your_user_name -p project_com < db.sql
```

8) В директории рядом с файлом `index.php` создайте файл конфигурации `conf.ini` и поместите в него следующий текст.
   Замените значения для доступа к вашей базе данных, `project.com` замените на адрес вашего будущего проекта.
```ini
[production]
database.params.host      = localhost
database.params.port      = 3306
database.params.dbname    = "project_com"
database.params.username  = "your_user_name"
database.params.password  = "your_user_pass"

[project.com : production]
system.host       = project.com
system.theme.name = material
cache             = /var/www/project.com/cache
log.on            = true
log.path          = /var/www/project.com/logs/error.log
log.system.file   = /var/www/project.com/logs/access.log
log.system.writer = file
```

9) Скачайте последнюю версию Composer с сайта [getcomposer.org](https://getcomposer.org/download/) или по прямой [ссылке](https://getcomposer.org/download/latest-stable/composer.phar) 

10) Скопируйте скаченный файл `composer.phar` в папку `core2`

11) Откройте консоль и выполните команды для установки зависимых пакетов  
```shell
$ сd /var/www/project.com/htdocs/core2
$ php composer.phar update
```


## Настройка веб сервера

Для запуска приложения необходимо настроить http сервер.

### Рекомендуемые настройки Apache

1) В папке `/etc/apache2/sites-available/` необходимо добавить файл `project.com.conf`. Замените `project.com` на адрес вашего проекта. 

2) Поместите в созданный файл следующее содержимое. 
   Замените `/var/www/project.com`, `project.com` и `php8.0` на ваши актуальные значения.

```apacheconf
ServerName project.com
DocumentRoot "/var/www/project.com/htdocs"
ErrorLog /var/www/project.com/logs/error_apache.log

<Directory "/var/www/project.com/htdocs">
    Options +Indexes +FollowSymLinks
    DirectoryIndex index.php
    Order allow,deny
    Allow from all
    AllowOverride All
    Require all granted
</Directory>

<FilesMatch \.php$>
    SetHandler "proxy:unix:/var/run/php/php8.0-fpm.sock|fcgi://localhost"
</FilesMatch>
```

3) Выполните консольную команды для активации настройки
```shell
$ sudo a2ensite project.com
$ sudo systemctl restart apache2
```

### Рекомендуемые настройки Nginx

1) В папке `/etc/nginx/sites-available/` необходимо добавить файл `project.com.conf`. Замените `project.com` на адрес вашего проекта. 

2) Поместите в созданный файл следующее содержимое.
   Замените `/var/www/project.com`, `project.com` и `php8.0` на ваши актуальные значения.

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name project.com;
    root /var/www/project.com/htdocs;
    error_log /var/www/project.com/logs/errors_nginx.log warn;
    
    index index.php index.html;
    client_max_body_size 100M;

    location / {
        try_files $uri /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_buffers 8 128k;
        fastcgi_buffer_size 256k;
        fastcgi_buffering off;
        fastcgi_pass unix:/var/run/php/php8.0-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }

    location ~* (\.ini|\.log|\.conf)$ {
        deny all;
    }
}
```

3) Выполните консольные команды для активации настройки. Замените `project.com` на адрес вашего проекта.
```shell
$ sudo ln -s /etc/nginx/sites-available/project.com.conf /etc/nginx/sites-enabled/project.com.conf
$ sudo systemctl reload nginx
```

## Проверка установки

Если это доменное имя (`project_com`) не привязано к вашему серверу, то для проверки его работы вы можете добавить домен на своем компьютере в хостах.

**Для Windows**

1) Открыть файл `C:\Windows\System32\drivers\etc\hosts`

2) Добавить указанную строку в конец этого файла. Замените `192.168.10.10` на актуальный ip адрес вашего сервера.
```text
192.168.10.10 project.com
```

**Для Linux**

1) Открыть файл `etc/hosts`

2) Добавить указанную строку в конец этого файла. Замените `192.168.10.10` на актуальный ip адрес вашего сервера.
```text
192.168.10.10 project.com
```

---

После установки приложение будет доступно по следующему URL:

```text
http://project.com
```

В случае успеха вы должны будите увидеть страницу логина

Войдите в приложение используя логин и пароль по умолчанию `admin` `admin`. После входа обязательно смените пароль на другой.
