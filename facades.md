# Фасади

[comment]: <> (-   [Вступ]&#40;#introduction&#41;)

[comment]: <> (-   [Коли використовувати фасади]&#40;#when-to-use-facades&#41;)

[comment]: <> (    -   [Фасади Vs. Ін’єкція залежності]&#40;#facades-vs-dependency-injection&#41;)

[comment]: <> (    -   [Фасади Vs. Допоміжні функції]&#40;#facades-vs-helper-functions&#41;)

[comment]: <> (-   [Як працюють фасади]&#40;#how-facades-work&#41;)

[comment]: <> (-   [Фасади в режимі реального часу]&#40;#real-time-facades&#41;)

[comment]: <> (-   [Class Reference фасадів]&#40;#facade-class-reference&#41;)

<a name="introduction"></a>

## Вступ

Фасади забезпечують "статичний" інтерфейс для класів, доступних у програмі[службовий контейнер](/docs/{{version}}/container). Судна Laravel мають безліч фасадів, що забезпечують доступ майже до всіх функцій Laravel. Фасади Laravel служать "статичними проксі" основних класів у службовому контейнері, забезпечуючи перевагу стислого, виразного синтаксису, зберігаючи при цьому більшу перевірочність і гнучкість, ніж традиційні статичні методи.

Усі фасади Laravel визначені в`Illuminate\Support\Facades`простір імен. Отже, ми можемо легко отримати доступ до фасаду так:

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

У документації Laravel багато прикладів використовуватимуть фасади, щоб продемонструвати різні особливості фреймворку.

<a name="when-to-use-facades"></a>

## Коли використовувати фасади

Фасади мають багато переваг. Вони забезпечують стислий запам’ятовуваний синтаксис, що дозволяє використовувати функції Laravel, не пам’ятаючи довгих назв класів, які потрібно вводити або конфігурувати вручну. Крім того, завдяки унікальному використанню динамічних методів PHP, їх легко перевірити.

Однак слід дотримуватися певної обережності при використанні фасадів. Основною небезпекою фасадів є повзучість класу. Оскільки фасади настільки прості у використанні і не потребують впорскування, можна легко дозволити вашим класам продовжувати рости та використовувати багато фасадів в одному класі. Використовуючи ін’єкцію залежностей, цей потенціал пом’якшується за допомогою візуального зворотного зв’язку, який великий конструктор дає вам, що ваш клас стає занадто великим. Тож, використовуючи фасади, звертайте особливу увагу на розмір вашого класу, щоб сфера його відповідальності залишалася вузькою.

> {tip} При створенні стороннього пакету, який взаємодіє з Laravel, краще вводити[Контракти Laravel](/docs/{{version}}/contracts)замість використання фасадів. Оскільки пакети створюються за межами самої Laravel, ви не матимете доступу до помічників тестування фасадів Laravel.

<a name="facades-vs-dependency-injection"></a>

### Фасади проти Ін’єкція залежності
<<<<<<< HEAD

Однією з основних переваг введення залежностей є можливість обміну реалізаціями введеного класу. Це корисно під час тестування, оскільки ви можете ввести макет або заглушку та стверджувати, що на заглушці були викликані різні методи.

=======

Однією з основних переваг введення залежностей є можливість обміну реалізаціями введеного класу. Це корисно під час тестування, оскільки ви можете ввести макет або заглушку та стверджувати, що на заглушці були викликані різні методи.

>>>>>>> 2684557fed6a6d955a2a144612b53ef3b2ac155e
Як правило, неможливо знущатись чи заглушати дійсно статичний метод класу. Однак, оскільки фасади використовують динамічні методи для виклику методів проксі до об'єктів, вирішених із контейнера служби, ми фактично можемо тестувати фасади так само, як і тестований ін'єкційний клас. Наприклад, враховуючи такий маршрут:

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

Ми можемо написати наступний тест, щоб переконатися, що`Cache::get`метод був викликаний з аргументом, який ми очікували:

    use Illuminate\Support\Facades\Cache;

    /**
     * A basic functional test example.
     *
     * @return void
     */
    public function testBasicExample()
    {
        Cache::shouldReceive('get')
             ->with('key')
             ->andReturn('value');

        $response = $this->get('/cache');

        $response->assertSee('value');
    }

<a name="facades-vs-helper-functions"></a>

### Фасади проти Допоміжні функції

На додаток до фасадів, Laravel включає в себе безліч "допоміжних" функцій, які можуть виконувати загальні завдання, такі як генерація поглядів, активація подій, відправлення завдань або надсилання відповідей HTTP. Багато з цих допоміжних функцій виконують ту саму функцію, що і відповідний фасад. Наприклад, цей фасадний виклик та допоміжний виклик еквівалентні:

    return View::make('profile');

    return view('profile');

Практичної різниці між фасадами та допоміжними функціями немає абсолютно. Використовуючи допоміжні функції, ви все одно можете протестувати їх точно так само, як і відповідний фасад. Наприклад, враховуючи такий маршрут:

    Route::get('/cache', function () {
        return cache('key');
    });

Під капотом`cache`помічник збирається зателефонувати`get`метод для класу, що лежить в основі`Cache`фасад. Отже, незважаючи на те, що ми використовуємо допоміжну функцію, ми можемо написати наступний тест, щоб переконатися, що метод був викликаний з аргументом, який ми очікували:

    use Illuminate\Support\Facades\Cache;

    /**
     * A basic functional test example.
     *
     * @return void
     */
    public function testBasicExample()
    {
        Cache::shouldReceive('get')
             ->with('key')
             ->andReturn('value');

        $response = $this->get('/cache');

        $response->assertSee('value');
    }

<a name="how-facades-work"></a>

## Як працюють фасади
<<<<<<< HEAD

У програмі Laravel Facade- це клас, який забезпечує доступ до об’єкта з контейнера. Машина, яка робить цю роботу, знаходиться в`Facade`клас. Фасади Laravel та будь-які спеціальні фасади, які ви створюєте, розширять основу`Illuminate\Support\Facades\Facade`клас.

=======

У програмі Laravel Facade- це клас, який забезпечує доступ до об’єкта з контейнера. Машина, яка робить цю роботу, знаходиться в`Facade`клас. Фасади Laravel та будь-які спеціальні фасади, які ви створюєте, розширять основу`Illuminate\Support\Facades\Facade`клас.

>>>>>>> 2684557fed6a6d955a2a144612b53ef3b2ac155e
`Facade`базовий клас використовує`__callStatic()`magic-метод для відкладання викликів з вашого фасаду до об'єкта, вирішеного з контейнера. У наведеному нижче прикладі здійснюється виклик кеш-системи Laravel. Поглянувши на цей код, можна припустити, що статичний метод`get`викликається на`Cache`клас:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            $user = Cache::get('user:'.$id);

            return view('profile', ['user' => $user]);
        }
    }

Зверніть увагу, що вгорі файлу ми "імпортуємо" файл`Cache`фасад. Цей Facadeслужить проксі для доступу до базової реалізації`Illuminate\Contracts\Cache\Factory`інтерфейс. Будь-які дзвінки, які ми робимо за допомогою фасаду, будуть передані базовому екземпляру кеш-служби Laravel.

Якщо ми подивимось на це`Illuminate\Support\Facades\Cache`клас, ви побачите, що не існує статичного методу`get`:

    class Cache extends Facade
    {
        /**
         * Get the registered name of the component.
         *
         * @return string
         */
        protected static function getFacadeAccessor() { return 'cache'; }
    }

Натомість,`Cache`Facadeпродовжує основу`Facade`клас і визначає метод`getFacadeAccessor()`. Завдання цього методу - повернути ім’я Binding контейнера служби. Коли користувач посилається на будь-який статичний метод на`Cache`фасаду, Laravel вирішує`cache`прив'язка з[службовий контейнер](/docs/{{version}}/container)і запускає запитаний метод (у цьому випадку`get`) проти цього об’єкта.

<a name="real-time-facades"></a>

## Фасади в режимі реального часу

Використовуючи фасади в режимі реального часу, ви можете поводитися з будь-яким класом у вашій програмі так, ніби це фасад. Щоб проілюструвати, як це можна використовувати, давайте розглянемо альтернативу. Наприклад, припустимо наш`Podcast`модель має`publish`метод. Однак, щоб опублікувати подкаст, нам потрібно ввести a`Publisher`примірник:

    <?php

    namespace App\Models;

    use App\Contracts\Publisher;
    use Illuminate\Database\Eloquent\Model;

    class Podcast extends Model
    {
        /**
         * Publish the podcast.
         *
         * @param  Publisher  $publisher
         * @return void
         */
        public function publish(Publisher $publisher)
        {
            $this->update(['publishing' => now()]);

            $publisher->publish($this);
        }
    }

Введення реалізації видавця в метод дозволяє нам легко протестувати метод ізольовано, оскільки ми можемо знущатись над введеним видавцем. Однак це вимагає від нас, щоб ми завжди передавали екземпляр видавця кожного разу, коли ми телефонуємо до`publish`метод. Використовуючи фасади в режимі реального часу, ми можемо підтримувати однакову перевірочність, не вимагаючи явного проходження а`Publisher`екземпляр. Щоб сформувати Facadeу реальному часі, додайте до простору імен імпортованого класу префікс`Facades`:

    <?php

    namespace App\Models;

    use Facades\App\Contracts\Publisher;
    use Illuminate\Database\Eloquent\Model;

    class Podcast extends Model
    {
        /**
         * Publish the podcast.
         *
         * @return void
         */
        public function publish()
        {
            $this->update(['publishing' => now()]);

            Publisher::publish($this);
        }
    }

Коли використовується Facadeу реальному часі, реалізація видавця буде вирішена з контейнера служби за допомогою частини інтерфейсу або імені класу, що з'являється після`Facades`префікс. Під час тестування ми можемо використовувати вбудовані помічники фасадного тестування Laravel для глузування над цим викликом методу:

    <?php

    namespace Tests\Feature;

    use App\Models\Podcast;
    use Facades\App\Contracts\Publisher;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Tests\TestCase;

    class PodcastTest extends TestCase
    {
        use RefreshDatabase;

        /**
         * A test example.
         *
         * @return void
         */
        public function test_podcast_can_be_published()
        {
            $podcast = Podcast::factory()->create();

            Publisher::shouldReceive('publish')->once()->with($podcast);

            $podcast->publish();
        }
    }

<a name="facade-class-reference"></a>

## Class Reference фасадів

Facade  |  Class  |  Service Container Binding
------------- | ------------- | -------------
App  |  [Illuminate\Foundation\Application](https://laravel.com/api/{{version}}/Illuminate/Foundation/Application.html)  |  `app`
Artisan  |  [Illuminate\Contracts\Console\Kernel](https://laravel.com/api/{{version}}/Illuminate/Contracts/Console/Kernel.html)  |  `artisan`
Auth  |  [Illuminate\Auth\AuthManager](https://laravel.com/api/{{version}}/Illuminate/Auth/AuthManager.html)  |  `auth`
Auth (Instance)  |  [Illuminate\Contracts\Auth\Guard](https://laravel.com/api/{{version}}/Illuminate/Contracts/Auth/Guard.html)  |  `auth.driver`
Blade  |  [Illuminate\View\Compilers\BladeCompiler](https://laravel.com/api/{{version}}/Illuminate/View/Compilers/BladeCompiler.html)  |  `blade.compiler`
Broadcast  |  [Illuminate\Contracts\Broadcasting\Factory](https://laravel.com/api/{{version}}/Illuminate/Contracts/Broadcasting/Factory.html)  |  &nbsp;
Broadcast (Instance)  |  [Illuminate\Contracts\Broadcasting\Broadcaster](https://laravel.com/api/{{version}}/Illuminate/Contracts/Broadcasting/Broadcaster.html)  |  &nbsp;
Bus  |  [Illuminate\Contracts\Bus\Dispatcher](https://laravel.com/api/{{version}}/Illuminate/Contracts/Bus/Dispatcher.html)  |  &nbsp;
Cache  |  [Illuminate\Cache\CacheManager](https://laravel.com/api/{{version}}/Illuminate/Cache/CacheManager.html)  |  `cache`
Cache (Instance)  |  [Illuminate\Cache\Repository](https://laravel.com/api/{{version}}/Illuminate/Cache/Repository.html)  |  `cache.store`
Config  |  [Illuminate\Config\Repository](https://laravel.com/api/{{version}}/Illuminate/Config/Repository.html)  |  `config`
Cookie  |  [Illuminate\Cookie\CookieJar](https://laravel.com/api/{{version}}/Illuminate/Cookie/CookieJar.html)  |  `cookie`
Crypt  |  [Illuminate\Encryption\Encrypter](https://laravel.com/api/{{version}}/Illuminate/Encryption/Encrypter.html)  |  `encrypter`
DB  |  [Illuminate\Database\DatabaseManager](https://laravel.com/api/{{version}}/Illuminate/Database/DatabaseManager.html)  |  `db`
DB (Instance)  |  [Illuminate\Database\Connection](https://laravel.com/api/{{version}}/Illuminate/Database/Connection.html)  |  `db.connection`
Event  |  [Illuminate\Events\Dispatcher](https://laravel.com/api/{{version}}/Illuminate/Events/Dispatcher.html)  |  `events`
File  |  [Illuminate\Filesystem\Filesystem](https://laravel.com/api/{{version}}/Illuminate/Filesystem/Filesystem.html)  |  `files`
Gate  |  [Illuminate\Contracts\Auth\Access\Gate](https://laravel.com/api/{{version}}/Illuminate/Contracts/Auth/Access/Gate.html)  |  &nbsp;
Hash  |  [Illuminate\Contracts\Hashing\Hasher](https://laravel.com/api/{{version}}/Illuminate/Contracts/Hashing/Hasher.html)  |  `hash`
Http  |  [Illuminate\Http\Client\Factory](https://laravel.com/api/{{version}}/Illuminate/Http/Client/Factory.html)  |  &nbsp;
Lang  |  [Illuminate\Translation\Translator](https://laravel.com/api/{{version}}/Illuminate/Translation/Translator.html)  |  `translator`
Log  |  [Illuminate\Log\LogManager](https://laravel.com/api/{{version}}/Illuminate/Log/LogManager.html)  |  `log`
Mail  |  [Illuminate\Mail\Mailer](https://laravel.com/api/{{version}}/Illuminate/Mail/Mailer.html)  |  `mailer`
Notification  |  [Illuminate\Notifications\ChannelManager](https://laravel.com/api/{{version}}/Illuminate/Notifications/ChannelManager.html)  |  &nbsp;
Password  |  [Illuminate\Auth\Passwords\PasswordBrokerManager](https://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBrokerManager.html)  |  `auth.password`
Password (Instance)  |  [Illuminate\Auth\Passwords\PasswordBroker](https://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBroker.html)  |  `auth.password.broker`
Queue  |  [Illuminate\Queue\QueueManager](https://laravel.com/api/{{version}}/Illuminate/Queue/QueueManager.html)  |  `queue`
Queue (Instance)  |  [Illuminate\Contracts\Queue\Queue](https://laravel.com/api/{{version}}/Illuminate/Contracts/Queue/Queue.html)  |  `queue.connection`
Queue (Base Class)  |  [Illuminate\Queue\Queue](https://laravel.com/api/{{version}}/Illuminate/Queue/Queue.html)  |  &nbsp;
Redirect  |  [Illuminate\Routing\Redirector](https://laravel.com/api/{{version}}/Illuminate/Routing/Redirector.html)  |  `redirect`
Redis  |  [Illuminate\Redis\RedisManager](https://laravel.com/api/{{version}}/Illuminate/Redis/RedisManager.html)  |  `redis`
Redis (Instance)  |  [Illuminate\Redis\Connections\Connection](https://laravel.com/api/{{version}}/Illuminate/Redis/Connections/Connection.html)  |  `redis.connection`
Request  |  [Illuminate\Http\Request](https://laravel.com/api/{{version}}/Illuminate/Http/Request.html)  |  `request`
Response  |  [Illuminate\Contracts\Routing\ResponseFactory](https://laravel.com/api/{{version}}/Illuminate/Contracts/Routing/ResponseFactory.html)  |  &nbsp;
Response (Instance)  |  [Illuminate\Http\Response](https://laravel.com/api/{{version}}/Illuminate/Http/Response.html)  |  &nbsp;
Route  |  [Illuminate\Routing\Router](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html)  |  `router`
Schema  |  [Illuminate\Database\Schema\Builder](https://laravel.com/api/{{version}}/Illuminate/Database/Schema/Builder.html)  |  &nbsp;
Session  |  [Illuminate\Session\SessionManager](https://laravel.com/api/{{version}}/Illuminate/Session/SessionManager.html)  |  `session`
Session (Instance)  |  [Illuminate\Session\Store](https://laravel.com/api/{{version}}/Illuminate/Session/Store.html)  |  `session.store`
Storage  |  [Illuminate\Filesystem\FilesystemManager](https://laravel.com/api/{{version}}/Illuminate/Filesystem/FilesystemManager.html)  |  `filesystem`
Storage (Instance)  |  [Illuminate\Contracts\Filesystem\Filesystem](https://laravel.com/api/{{version}}/Illuminate/Contracts/Filesystem/Filesystem.html)  |  `filesystem.disk`
URL  |  [Illuminate\Routing\UrlGenerator](https://laravel.com/api/{{version}}/Illuminate/Routing/UrlGenerator.html)  |  `url`
Validator  |  [Illuminate\Validation\Factory](https://laravel.com/api/{{version}}/Illuminate/Validation/Factory.html)  |  `validator`
Validator (Instance)  |  [Illuminate\Validation\Validator](https://laravel.com/api/{{version}}/Illuminate/Validation/Validator.html)  |  &nbsp;
View  |  [Illuminate\View\Factory](https://laravel.com/api/{{version}}/Illuminate/View/Factory.html)  |  `view`
View (Instance)  |  [Illuminate\View\View](https://laravel.com/api/{{version}}/Illuminate/View/View.html)  |  &nbsp;
