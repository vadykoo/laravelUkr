# Розгортання

[comment]: <> (-   [Вступ]&#40;#introduction&#41;)

[comment]: <> (-   [Конфігурація сервера]&#40;#server-configuration&#41;)

[comment]: <> (    -   [Nginx]&#40;#nginx&#41;)

[comment]: <> (-   [Оптимізація]&#40;#optimization&#41;)

[comment]: <> (    -   [Оптимізація автозавантажувача]&#40;#autoloader-optimization&#41;)

[comment]: <> (    -   [Оптимізація завантаження конфігурації]&#40;#optimizing-configuration-loading&#41;)

[comment]: <> (    -   [Оптимізація завантаження маршруту]&#40;#optimizing-route-loading&#41;)

[comment]: <> (    -   [Оптимізація завантаження View]&#40;#optimizing-view-loading&#41;)

[comment]: <> (-   [Розгортання з Forge / Vapor]&#40;#deploying-with-forge-or-vapor&#41;)

<a name="introduction"></a>

## Вступ

Коли ви будете готові до розгортання програми Laravel у виробничій системі, є кілька важливих речей, які ви можете зробити, щоб забезпечити максимально ефективну роботу своєї програми. У цьому документі ми розглянемо декілька чудових вихідних моментів, щоб переконатися, що програма Laravel правильно розгорнута.

<a name="server-configuration"></a>

## Конфігурація сервера

<a name="nginx"></a>

### Nginx

Якщо ви розгортаєте свою програму на сервері, на якому запущений Nginx, ви можете використовувати наступний файл конфігурації як вихідну точку для налаштування веб-сервера. Швидше за все, цей файл потрібно буде налаштувати залежно від конфігурації вашого сервера. Якщо вам потрібна допомога в управлінні сервером, розгляньте можливість використання такої послуги, як[Кузня Laravel](https://forge.laravel.com):

    server {
        listen 80;
        server_name example.com;
        root /srv/example.com/public;

        add_header X-Frame-Options "SAMEORIGIN";
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Content-Type-Options "nosniff";

        index index.php;

        charset utf-8;

        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }

        location = /favicon.ico { access_log off; log_not_found off; }
        location = /robots.txt  { access_log off; log_not_found off; }

        error_page 404 /index.php;

        location ~ \.php$ {
            fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
            fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
            include fastcgi_params;
        }

        location ~ /\.(?!well-known).* {
            deny all;
        }
    }

<a name="optimization"></a>

## Оптимізація

<a name="autoloader-optimization"></a>

### Оптимізація автозавантажувача

Розгортаючи у виробничій версії, переконайтеся, що ви оптимізуєте карту автозавантажувача класу Composer, щоб Composer швидко знайшов відповідний файл для завантаження для даного класу:

    composer install --optimize-autoloader --no-dev

> {tip} Окрім оптимізації автозавантажувача, ви завжди повинні обов’язково включати файл`composer.lock`файл у сховищі керування джерелом вашого проекту. Залежності вашого проекту можна встановити набагато швидше, коли a`composer.lock`файл присутній.

<a name="optimizing-configuration-loading"></a>

### Оптимізація завантаження конфігурації

Розгортаючи вашу програму до робочої версії, ви повинні переконатися, що ви запустили`config:cache`Команда ремісників під час процесу розгортання:

    php artisan config:cache

Ця команда поєднає всі конфігураційні файли Laravel в один кешований файл, що значно зменшує кількість поїздок, які фреймворк повинен здійснити до файлової системи під час завантаження значень конфігурації.

> {note} Якщо ви виконаєте файл`config:cache`команди під час процесу розгортання, ви повинні бути впевнені, що телефонуєте лише на`env`функція з файлів конфігурації. Після кешування конфігурації файл`.env`файл не буде завантажений, і всі дзвінки до`env`функція для`.env`повертаються змінні`null`.

<a name="optimizing-route-loading"></a>

### Оптимізація завантаження маршруту

Якщо ви створюєте великий додаток з багатьма маршрутами, переконайтеся, що у вас запущено`route:cache`Команда ремісників під час процесу розгортання:

    php artisan route:cache

Ця команда зменшує всі ваші реєстрації маршрутів в один виклик методу в кешованому файлі, покращуючи ефективність реєстрації маршруту при реєстрації сотень маршрутів.

<a name="optimizing-view-loading"></a>

### Оптимізація завантаження View

Розгортаючи вашу програму до робочої версії, ви повинні переконатися, що ви запустили`view:cache`Команда ремісників під час процесу розгортання:

    php artisan view:cache

Ця команда попередньо компілює всі ваші View Blade, щоб вони не компілювалися на вимогу, покращуючи продуктивність кожного запиту, який повертає View.

<a name="deploying-with-forge-or-vapor"></a>

## Розгортання з Forge / Vapor

Якщо ви ще не зовсім готові керувати власною конфігурацією сервера або вам не зручно налаштовувати всі різні сервіси, необхідні для запуску надійної програми Laravel,[Кузня Laravel](https://forge.laravel.com)є чудовою альтернативою.

Laravel Forge може створювати сервери на різних постачальниках інфраструктури, таких як DigitalOcean, Linode, AWS та ін. Крім того, Forge встановлює та керує всіма інструментами, необхідними для створення надійних додатків Laravel, таких як Nginx, MySQL, Redis, Memcached, Beanstalk та ін.

<a name="laravel-vapor"></a>

#### Пара Laravel

Якщо ви хочете повністю безсерверну платформу розгортання з автоматичним масштабуванням, налаштовану на Laravel, відвідайте[Пара Laravel](https://vapor.laravel.com). Laravel Vapor - це безсерверна платформа розгортання для Laravel, що працює на базі AWS. Запустіть свою інфраструктуру Laravel на Vapor і полюбіть масштабовану простоту безсерверності. Laravel Vapor налаштований творцями Laravel для безперебійної роботи з фреймворком, щоб ви могли продовжувати писати свої програми Laravel точно так, як звикли.
