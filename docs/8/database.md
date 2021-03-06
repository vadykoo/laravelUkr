# База даних: Початок роботи

[comment]: <> (-   [Вступ]&#40;#introduction&#41;)

[comment]: <> (    -   [Конфігурація]&#40;#configuration&#41;)

[comment]: <> (    -   [З’єднання для читання та запису]&#40;#read-and-write-connections&#41;)

[comment]: <> (    -   [Використання декількох підключень до бази даних]&#40;#using-multiple-database-connections&#41;)

[comment]: <> (-   [Запуск необроблених запитів SQL]&#40;#running-queries&#41;)

[comment]: <> (-   [Listening подій запитів]&#40;#listening-for-query-events&#41;)

[comment]: <> (-   [Транзакції з базами даних]&#40;#database-transactions&#41;)

[comment]: <> (-   [Підключення до CLI бази даних]&#40;#connecting-to-the-database-cli&#41;)

<a name="introduction"></a>

## Вступ

Laravel робить взаємодію з базами даних надзвичайно простою через різні бази даних, використовуючи або необроблений SQL,[вільний конструктор запитів](/docs/{{version}}/queries), і[Eloquent ОРМ](/docs/{{version}}/eloquent). В даний час Laravel підтримує чотири бази даних:

<div class="content-list" markdown="1">
- MySQL 5.6+ ([Version Policy](https://en.wikipedia.org/wiki/MySQL#Release_history))
- PostgreSQL 9.4+ ([Version Policy](https://www.postgresql.org/support/versioning/))
- SQLite 3.8.8+
- SQL Server 2017+ ([Version Policy](https://support.microsoft.com/en-us/lifecycle/search))
</div>

<a name="configuration"></a>

### Конфігурація

Конфігурація бази даних для вашої програми знаходиться за адресою`config/database.php`. У цьому файлі ви можете визначити всі свої підключення до бази даних, а також вказати, яке з’єднання слід використовувати за замовчуванням. У цьому файлі наведено приклади для більшості підтримуваних систем баз даних.

За замовчуванням зразок Laravel[конфігурація середовища](/docs/{{version}}/configuration#environment-configuration)готовий до використання з[ Homestead  Laravel](/docs/{{version}}/homestead), яка є зручною віртуальною машиною для розробки Laravel на вашій локальній машині. Ви можете змінювати цю конфігурацію відповідно до вашої локальної бази даних.

<a name="sqlite-configuration"></a>

#### Конфігурація SQLite

Після створення нової бази даних SQLite за допомогою команди, наприклад`touch database/database.sqlite`, Ви можете легко налаштувати змінні середовища, щоб вони вказували на нову базу даних, використовуючи абсолютний шлях до бази даних:

    DB_CONNECTION=sqlite
    DB_DATABASE=/absolute/path/to/database.sqlite

Щоб увімкнути обмеження зовнішнього ключа для з'єднань SQLite, слід встановити`DB_FOREIGN_KEYS`середовище змінна до`true`:

    DB_FOREIGN_KEYS=true

<a name="configuration-using-urls"></a>

#### Налаштування за допомогою URL-адрес

Як правило, з'єднання з базою даних конфігуруються з використанням декількох значень конфігурації, таких як`host`,`database`,`username`,`password`і т. д. Кожне із цих значень конфігурації має свою відповідну змінну середовища. Це означає, що під час налаштування інформації про підключення до бази даних на виробничому сервері вам потрібно керувати кількома змінними середовища.

Деякі постачальники керованих баз даних, такі як Heroku, надають єдину "URL-адресу" бази даних, яка містить всю інформацію про з'єднання для бази даних в одному рядку. Приклад URL-адреси бази даних може виглядати приблизно так:

    mysql://root:password@127.0.0.1/forge?charset=UTF-8

Ці URL-адреси, як правило, відповідають стандартним умовам схеми:

    driver://username:password@host:port/database?options

Для зручності Laravel підтримує ці URL-адреси як альтернативу конфігуруванню вашої бази даних за допомогою декількох параметрів конфігурації. Якщо`url`(або відповідний`DATABASE_URL`параметр конфігурації присутній, він буде використовуватися для вилучення підключення до бази даних та інформації про облікові дані.

<a name="read-and-write-connections"></a>

### З’єднання для читання та запису

Іноді може знадобитися використовувати одне підключення до бази даних для операторів SELECT, а інше - для операторів INSERT, UPDATE та DELETE. Laravel робить це вітряком, і належні зв’язки завжди будуть використовуватися незалежно від того, використовуєте ви необроблені запити, конструктор запитів або Eloquent ORM.

Щоб побачити, як слід налаштувати з'єднання для читання / запису, давайте розглянемо цей приклад:

    'mysql' => [
        'read' => [
            'host' => [
                '192.168.1.1',
                '196.168.1.2',
            ],
        ],
        'write' => [
            'host' => [
                '196.168.1.3',
            ],
        ],
        'sticky' => true,
        'driver' => 'mysql',
        'database' => 'database',
        'username' => 'root',
        'password' => '',
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_unicode_ci',
        'prefix' => '',
    ],

Зверніть увагу, що до масиву конфігурації додано три клавіші:`read`,`write`і`sticky`.`read`і`write`Ключі мають значення масиву, що містять один ключ:`host`. Решта опцій бази даних для`read`і`write`з'єднання будуть об'єднані з основними`mysql`масив.

Вам потрібно лише розмістити елементи в`read`і`write`масиви, якщо ви хочете замінити значення основного масиву. Отже, у цьому випадку`192.168.1.1`буде використовуватися як хост для з'єднання "читання", поки`192.168.1.3`буде використовуватися для з'єднання "писати". Повноваження бази даних, префікс, набір символів та всі інші параметри в основному`mysql`масив буде спільним для обох з'єднань.

<a name="the-sticky-option"></a>

#### `sticky`Варіант

`sticky`варіант є_за бажанням_ value that can be used to allow the immediate reading of records that have been written to the database during the current request cycle. If the `sticky`параметр увімкнено, і операція "запис" була виконана щодо бази даних протягом поточного циклу запиту, будь-які подальші операції "читання" використовуватимуть з'єднання "запис". Це гарантує, що будь-які дані, записані протягом циклу запиту, можуть бути негайно прочитані з бази даних під час того самого запиту. Ви самі вирішуєте, чи є це бажаною поведінкою для вашої програми.

<a name="using-multiple-database-connections"></a>

### Використання декількох підключень до бази даних

При використанні декількох з'єднань ви можете отримати доступ до кожного з'єднання через`connection`метод на`DB`фасад.`name`перейшов до`connection`метод повинен відповідати одному із з'єднань, перелічених у вашому`config/database.php`файл конфігурації:

    $users = DB::connection('foo')->select(...);

Ви також можете отримати доступ до необробленого базового екземпляра PDO за допомогою`getPdo`метод на екземплярі підключення:

    $pdo = DB::connection()->getPdo();

<a name="running-queries"></a>

## Запуск необроблених запитів SQL

Після налаштування підключення до бази даних ви можете запускати запити за допомогою`DB`фасад.`DB`facade надає методи для кожного типу запиту:`select`,`update`,`insert`,`delete`, і`statement`.

<a name="running-a-select-query"></a>

#### Запуск вибраного запиту

Щоб запустити базовий запит, ви можете використовувати`select`метод на`DB`фасад:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\DB;

    class UserController extends Controller
    {
        /**
         * Show a list of all of the application's users.
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::select('select * from users where active = ?', [1]);

            return view('user.index', ['users' => $users]);
        }
    }

Перший аргумент, переданий в`select`метод - це необроблений запит SQL, тоді як другий аргумент - це будь-які Binding параметрів, які потрібно прив’язати до запиту. Як правило, це значення`where`пункт обмеження. Прив'язка параметрів забезпечує захист від ін'єкції SQL.

`select`метод завжди повертає`array`результатів. Кожен результат у масиві буде PHP`stdClass`об'єкт, що дозволяє отримати доступ до значень результатів:

    foreach ($users as $user) {
        echo $user->name;
    }

<a name="using-named-bindings"></a>

#### Використання іменованих Bindings

Замість використання`?`щоб представити прив'язку параметрів, ви можете виконати запит, використовуючи іменовані прив'язки:

    $results = DB::select('select * from users where id = :id', ['id' => 1]);

<a name="running-an-insert-statement"></a>

#### Запуск оператора вставки

Щоб виконати файл`insert`ви можете використовувати`insert`метод на`DB`фасад. Люблю`select`, цей метод приймає необроблений запит SQL як перший аргумент, а прив'язки - як другий аргумент:

    DB::insert('insert into users (id, name) values (?, ?)', [1, 'Dayle']);

<a name="running-an-update-statement"></a>

#### Запуск заяви про оновлення

`update`метод слід використовувати для оновлення існуючих записів у базі даних. Буде повернуто кількість рядків, на які впливає оператор:

    $affected = DB::update('update users set votes = 100 where name = ?', ['John']);

<a name="running-a-delete-statement"></a>

#### Запуск оператора видалення

`delete`метод слід використовувати для видалення записів з бази даних. Люблю`update`, буде повернуто кількість уражених рядків:

    $deleted = DB::delete('delete from users');

<a name="running-a-general-statement"></a>

#### Запуск загальної заяви

Деякі оператори бази даних не повертають жодного значення. Для цих типів операцій ви можете використовувати`statement`метод на`DB`фасад:

    DB::statement('drop table users');

<a name="running-an-unprepared-statement"></a>

#### Запуск непідготовленої заяви

Іноді вам може знадобитися виконати оператор SQL без прив'язки будь-яких значень. Ви можете використовувати`DB`фасадні`unprepared`метод для досягнення цього:

    DB::unprepared('update users set votes = 100 where name = "Dries"');

> {note} Оскільки непідготовлені оператори не пов’язують параметри, вони можуть бути вразливими до введення SQL. Ви ніколи не повинні дозволяти контрольовані користувачем значення в непідготовленому операторі.

<a name="implicit-commits-in-transactions"></a>

#### Неявні коміти

При використанні`DB`фасадні`statement`і`unprepared`Методи в рамках транзакцій ви повинні бути обережними, щоб уникати Assertiors, які викликають[неявні коміти](https://dev.mysql.com/doc/refman/8.0/en/implicit-commit.html). Ці оператори змусять механізм бази даних опосередковано фіксувати всю транзакцію, залишаючи Laravel не в курсі рівня транзакцій бази даних. Прикладом такого твердження є створення таблиці бази даних:

    DB::unprepared('create table a (col varchar(1) null)');

Будь ласка, зверніться до посібника MySQL для[перелік усіх Assertiors](https://dev.mysql.com/doc/refman/8.0/en/implicit-commit.html)що запускають неявні коміти.

<a name="listening-for-query-events"></a>

## Listening подій запитів

Якщо ви хочете отримувати кожен SQL-запит, що виконується вашою програмою, ви можете використовувати`listen`метод. Цей метод корисний для реєстрації запитів або Debug. Ви можете зареєструвати прослуховувач запитів у[постачальник послуг](/docs/{{version}}/providers):

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\DB;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Register any application services.
         *
         * @return void
         */
        public function register()
        {
            //
        }

        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            DB::listen(function ($query) {
                // $query->sql
                // $query->bindings
                // $query->time
            });
        }
    }

<a name="database-transactions"></a>

## Транзакції з базами даних

Ви можете використовувати`transaction`метод на`DB`Facadeдля запуску набору операцій в рамках транзакції бази даних. Якщо всередині транзакції виникає виняток`Closure`, транзакція буде автоматично повернута назад. Якщо`Closure`Після успішного виконання транзакція буде автоматично здійснена. Вам не потрібно турбуватися про ручний відкат або фіксацію під час використання`transaction`метод:

    DB::transaction(function () {
        DB::table('users')->update(['votes' => 1]);

        DB::table('posts')->delete();
    });

<a name="handling-deadlocks"></a>

#### Виправлення тупикових ситуацій

`transaction`метод приймає необов’язковий другий аргумент, який визначає кількість повторних спроб транзакції при виникненні тупикової ситуації. Після того, як ці спроби будуть вичерпані, буде створено виняток:

    DB::transaction(function () {
        DB::table('users')->update(['votes' => 1]);

        DB::table('posts')->delete();
    }, 5);

<a name="manually-using-transactions"></a>

#### Використання транзакцій вручну

Якщо ви хочете розпочати транзакцію вручну і мати повний контроль над відкотами та комітами, ви можете використовувати`beginTransaction`метод на`DB`фасад:

    DB::beginTransaction();

Ви можете відмовити транзакцію через`rollBack`метод:

    DB::rollBack();

Нарешті, ви можете здійснити транзакцію через`commit`метод:

    DB::commit();

> {tip} The`DB`Фасадні методи транзакцій контролюють транзакції для обох[конструктор запитів](/docs/{{version}}/queries)і[Eloquent ОРМ](/docs/{{version}}/eloquent).

<a name="connecting-to-the-database-cli"></a>

## Підключення до CLI бази даних

Якщо ви хочете підключитися до CLI вашої бази даних, ви можете використовувати`db`Команда ремісників:

    php artisan db

Якщо потрібно, ви можете вказати ім'я підключення до бази даних для підключення до підключення до бази даних, яке не є за замовчуванням:

    php artisan db mysql
