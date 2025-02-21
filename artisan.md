---
git: 46c2634ef5a4f15427c94a3157b626cf5bd3937f
---

# Консоль Artisan

<a name="introduction"></a>
## Введение

Artisan – это интерфейс командной строки, входящий в состав Laravel. Он предлагает ряд полезных команд, которые помогут при создании приложения. Для просмотра списка всех доступных команд Artisan можно использовать команду `list`:

```shell
php artisan list
```

Каждая команда также включает в себя экран «справки», который отображает и описывает доступные аргументы и параметры команды. Чтобы просмотреть экран справки, используйте `help` перед именем команды:

```shell
php artisan help migrate
```

<a name="laravel-sail"></a>
#### Laravel Sail

Если вы используете [Laravel Sail](/docs/{{version}}/sail) в качестве локальной среды разработки, не забудьте использовать командную строку `sail` для вызова команд Artisan. Sail выполнит ваши команды Artisan в контейнерах Docker вашего приложения:

```shell
./vendor/bin/sail artisan list
```

<a name="tinker"></a>
### Tinker (REPL)

Laravel Tinker – это мощный REPL для фреймворка Laravel, основанный на пакете [PsySH](https://github.com/bobthecow/psysh).

<a name="installation"></a>
#### Установка

Все приложения Laravel по умолчанию включают Tinker. Однако вы можете установить Tinker с помощью Composer, если вы ранее удалили его из своего приложения:


```shell
composer require laravel/tinker
```

> [!NOTE]
> Ищете графический интерфейс для взаимодействия с приложением Laravel? Зацените [Tinkerwell](https://tinkerwell.app)!

<a name="usage"></a>
#### Использование

Tinker позволяет взаимодействовать полностью со всем приложением Laravel из командной строки, включая модели Eloquent, задачи, события и многое другое. Чтобы войти в среду Tinker, выполните команду `tinker` Artisan:

```shell
php artisan tinker
```

Вы можете опубликовать конфигурационный файл Tinker с помощью команды `vendor:publish`:

```shell
php artisan vendor:publish --provider="Laravel\Tinker\TinkerServiceProvider"
```

> [!WARNING]
> Глобальный помощник `dispatch` и метод `dispatch` класса `Dispatchable` зависят от "garbage collection" для помещения задания в очередь. Следовательно, при использовании Tinker вы должны использовать `Bus::dispatch` или `Queue::push` для отправки заданий.

<a name="command-allow-list"></a>
#### Список разрешенных команд

Tinker использует список «разрешенных» команд, которые разрешено запускать Artisan в её среде. По умолчанию вы можете запускать команды `clear-compiled`, `down`, `env`, `inspire`, `migrate`, `optimize` и `up`. Для добавления в этот список больше команд, добавьте их в массив `commands` конфигурационного файла `config/tinker.php`:

    'commands' => [
        // App\Console\Commands\ExampleCommand::class,
    ],

<a name="classes-that-should-not-be-aliased"></a>
#### Черный список псевдонимов

Как правило, Tinker автоматически создает псевдонимы классов, когда вы взаимодействуете с ними в Tinker. Тем не менее вы можете запретить такое поведение для некоторых классов, перечислив их в массиве `dont_alias` конфигурационного файла `config/tinker.php`:

    'dont_alias' => [
        App\Models\User::class,
    ],

<a name="writing-commands"></a>
## Написание команд

В дополнение к командам Artisan, вы можете создавать пользовательские команды. Команды обычно хранятся в каталоге `app/Console/Commands`; однако вы можете выбрать другое месторасположение, если эти команды могут быть загружены менеджером Composer.

<a name="generating-commands"></a>
### Генерация команд

Чтобы сгенерировать новую команду, используйте команду `make:command` [Artisan](artisan). Эта команда поместит новый класс команды в каталог `app/Console/Commands` вашего приложения. Если этот каталог не существует в вашем приложении, то Laravel предварительно создаст его:

```shell
php artisan make:command SendEmails
```

<a name="command-structure"></a>
### Структура команды

После создания команды следует заполнить свойства класса `$signature` и `$description`. Эти свойства будут отображаться на экране при использовании команды `list`. Свойство `$signature` также позволяет [определять вводимые данные](#defining-input-expectations). Метод `handle` будет вызываться при выполнении команды. Вы можете разместить логику команды в этом методе.

Давайте рассмотрим пример команды. Обратите внимание, что мы можем запросить любые необходимые зависимости в методе `handle` команды. [Контейнер служб](/docs/{{version}}/container) Laravel автоматически внедрит все зависимости, типы которых объявлены в этом методе:

    <?php

    namespace App\Console\Commands;

    use App\Models\User;
    use App\Support\DripEmailer;
    use Illuminate\Console\Command;

    class SendEmails extends Command
    {
        /**
         * Имя и сигнатура консольной команды.
         *
         * @var string
         */
        protected $signature = 'mail:send {user}';

        /**
         * Описание консольной команды.
         *
         * @var string
         */
        protected $description = 'Send a marketing email to a user';


        /**
         * Выполнить консольную команду.
         */
        public function handle(DripEmailer $drip): void
        {
            $drip->send(User::find($this->argument('user')));
        }
    }

> [!NOTE]
> Хорошей практикой повторного использования кода считается создание «простых» консольных команд с делегированием своих задач службам приложения. В приведенном примере мы внедряем класс службы для выполнения «затратной» отправки электронных писем.

<a name="closure-commands"></a>
### Анонимные команды

Анонимные команды обеспечивают альтернативу определению консольных команд в виде классов. Точно так же, как замыкания маршрутов являются альтернативой контроллерам. В рамках метода `commands` файла `app/Console/Kernel.php` Laravel загружает файл `routes/console.php`:

    /**
     * Зарегистрировать команды, основанные на анонимных функциях.
     */
    protected function commands(): void
    {
        require base_path('routes/console.php');
    }

Этот файл не определяет маршруты HTTP, он определяет точки входа (маршруты) для консольных команд в приложении. В этом файле с помощью метода `Artisan::command` можно определить все анонимные консольные команды. Метод `command` принимает два аргумента: [сигнатуру команды](#defining-input-expectations) и замыкание, которое получает аргументы и параметры команды:

    Artisan::command('mail:send {user}', function (string $user) {
        $this->info("Sending email to: {$user}!");
    });

Замыкание привязано к базовому экземпляру команды, поэтому у вас есть полный доступ ко всем вспомогательным методам, к которым вы обычно можете обращаться в команде, созданной с помощью класса.

<a name="type-hinting-dependencies"></a>
#### Типизация зависимостей

Помимо получения аргументов и параметров, замыкание анонимной команды также принимает дополнительные зависимости из [контейнера служб](/docs/{{version}}/container), необходимые для внедрения:

    use App\Models\User;
    use App\Support\DripEmailer;

    Artisan::command('mail:send {user}', function (DripEmailer $drip, string $user) {
        $drip->send(User::find($user));
    });

<a name="closure-command-descriptions"></a>
#### Описания анонимных команд

При определении анонимных команд, можно использовать метод `purpose` для добавления описания команды. Это описание будет отображаться при запуске команд `php artisan list` и `php artisan help`:

    Artisan::command('mail:send {user}', function (string $user) {
        // ...
    })->purpose('Send a marketing email to a user');

<a name="isolatable-commands"></a>
### Изолированные команды

> [!WARNING]
> Для использования этой функции ваше приложение должно использовать `memcached`, `redis`, `dynamodb`, `database`, `file` или `array` в качестве кэш-драйвера по умолчанию. Кроме того, все серверы должны обмениваться данными с одним и тем же центральным сервером кэша.

Иногда вам может потребоваться, чтобы одновременно мог выполняться только один экземпляр команды. Для этого вы можете реализовать интерфейс `Illuminate\Contracts\Console\Isolatable` в вашем классе команды:

     <?php
    
    namespace App\Console\Commands;

    use Illuminate\Console\Command;
    use Illuminate\Contracts\Console\Isolatable;

    class SendEmails extends Command implements Isolatable
    {
        // ...
    }


Когда команда отмечена как `Isolatable`, Laravel автоматически добавит опцию `--isolated` к команде. Когда команда вызывается с этой опцией, Laravel гарантирует, что никакие другие экземпляры этой команды в данный момент не выполняются. Laravel достигает этого, пытаясь получить блокировку с использованием кэш-драйвера по умолчанию вашего приложения. Если другие экземпляры команды выполняются, команда не будет выполнена; однако команда все равно завершится с кодом успешного завершения:

```shell
php artisan mail:send 1 --isolated
```

Если вы хотите указать код статуса завершения, который команда должна вернуть, если она не может выполниться, вы можете предоставить его с помощью опции `isolated`:

```shell
php artisan mail:send 1 --isolated=12
```

<a name="lock-id"></a>
#### Идентификатор блокировки (Lock ID)

По умолчанию Laravel использует имя команды для генерации строкового ключа, который используется для получения блокировки в кэше вашего приложения. Однако вы можете настроить этот ключ, определив метод `isolatableId` в вашем классе команды Artisan, что позволяет вам интегрировать аргументы или опции команды в ключ:

```php
/**
 * Get the isolatable ID for the command.
 */
public function isolatableId(): string
{
    return $this->argument('user');
}
```

<a name="lock-expiration-time"></a>
#### Срок действия блокировки

По умолчанию изоляционные блокировки истекают после завершения команды. Или, если команда прервана и не может быть завершена, срок действия блокировки истечет через один час. Однако вы можете настроить время действия блокировки, определив метод `isolationLockExpiresAt` в вашей команде:

```php
use DateTimeInterface;
use DateInterval;

/**
 * Determine when an isolation lock expires for the command.
 */
public function isolationLockExpiresAt(): DateTimeInterface|DateInterval
{
    return now()->addMinutes(5);
}
```

<a name="defining-input-expectations"></a>
## Определение вводимых данных

При написании консольных команд обычно происходит сбор данных, получаемых от пользователя, с помощью аргументов или параметров. Laravel позволяет очень удобно определять входные данные, которые вы ожидаете от пользователя, используя свойство `$signature` команды. Свойство `$signature` позволяет определить имя, аргументы и параметры команды в едином выразительном синтаксисе, схожим с синтаксисом маршрутов.

<a name="arguments"></a>
### Аргументы

Все предоставленные пользователем аргументы и параметры заключаются в фигурные скобки. В следующем примере команда определяет один обязательный аргумент `user`:

    /**
     * Имя и сигнатура консольной команды.
     *
     * @var string
     */
    protected $signature = 'mail:send {user}';

По желанию можно сделать аргументы необязательными или определить значения по умолчанию:

    // Необязательный аргумент ...
    'mail:send {user?}'

    // Необязательный аргумент с заданным по умолчанию значением ...
    'mail:send {user=foo}'

<a name="options"></a>
### Параметры

Параметры, как и аргументы, являются разновидностью пользовательского ввода. Параметры должны иметь префикс в виде двух дефисов (`--`), при использовании их в командной строке. Существует два типа параметров: получающие значение, и те, которые его не получают. Параметры, которые не получают значение, служат логическими «переключателями». Давайте рассмотрим пример такого варианта:

    /**
     * Имя и сигнатура консольной команды.
     *
     * @var string
     */
    protected $signature = 'mail:send {user} {--queue}';

В этом примере при вызове команды Artisan может быть указан переключатель `--queue`. Если переключатель `--queue` передан, то значение этого параметра будет `true`. В противном случае значение будет `false`:

```shell
php artisan mail:send 1 --queue
```

<a name="options-with-values"></a>
#### Параметры со значениями

Давайте рассмотрим параметр, ожидающий значение. Если пользователь должен указать значение для параметра, то добавьте суффикс `=` к имени параметра:

    /**
     * Имя и сигнатура консольной команды.
     *
     * @var string
     */
    protected $signature = 'mail:send {user} {--queue=}';

В этом примере пользователь может передать значение для параметра. Если параметр не указан при вызове команды, то его значение будет `null`:

```shell
php artisan mail:send 1 --queue=default
```

Параметру можно присвоить значение по умолчанию, указав его после имени. Если значение параметра не передано пользователем, то будет использовано значение по умолчанию:

    'mail:send {user} {--queue=default}'

<a name="option-shortcuts"></a>
#### Псевдонимы параметров

Чтобы назначить псевдоним при определении параметра, вы можете указать его перед именем параметра и использовать символ разделителя `|` для отделения псевдонима от полного имени параметра:

    'mail:send {user} {--Q|queue}'

При вызове команды в терминале, псевдонимы параметров должны иметь префикс с одним дефисом, и символ `=` не должен использоваться при указании значения параметра:

```shell
php artisan mail:send 1 -Qdefault
```

<a name="input-arrays"></a>
### Массивы данных

Чтобы определить, что аргументы или параметры ожидают массив данных, используйте метасимвол `*`. Во-первых, давайте рассмотрим пример, в котором описывается аргумент как массив данных:

    'mail:send {user*}'

При вызове этого метода аргументы `user` могут передаваться по порядку в командную строку. Например, следующая команда установит значение `user` как `1` и `2`:

```shell
php artisan mail:send 1 2
```

Метасимвол `*` можно комбинировать с необязательным определением аргумента, чтобы разрешить ноль или более экземпляров аргумента:

    'mail:send {user?*}'

<a name="option-arrays"></a>
#### Параметр со множеством значений

При определении параметра, ожидающего множество значений, каждое значение передаваемого команде параметра должно иметь префикс с именем параметра:

    'mail:send {--id=*}'

Такую команду можно вызвать, передав несколько аргументов `--id`:

```shell
php artisan mail:send --id=1 --id=2
```

<a name="input-descriptions"></a>
### Описания вводимых данных

Вы можете назначить описания входным аргументам и параметрам, отделив имя аргумента от описания с помощью двоеточия. Если вам нужно немного больше места для определения вашей команды, то распределите определение на несколько строк:

    /**
     * Имя и сигнатура консольной команды.
     *
     * @var string
     */
    protected $signature = 'mail:send
                            {user : The ID of the user}
                            {--queue : Whether the job should be queued}';

<a name="prompting-for-missing-input"></a>
### Запрос отсутствующего ввода

Если ваша команда содержит обязательные аргументы, пользователь получит сообщение об ошибке, если они не были предоставлены. В качестве альтернативы, вы можете настроить вашу команду так, чтобы автоматически запрашивать пользователя при отсутствии необходимых аргументов, реализовав интерфейс `PromptsForMissingInput`:

    <?php

    namespace App\Console\Commands;

    use Illuminate\Console\Command;
    use Illuminate\Contracts\Console\PromptsForMissingInput;

    class SendEmails extends Command implements PromptsForMissingInput
    {
        /**
         * Имя и сигнатура консольной команды.
         *
         * @var string
         */
        protected $signature = 'mail:send {user}';

        // ...
    }

Если Laravel должен получить обязательный аргумент от пользователя, он автоматически запросит у пользователя этот аргумент, формулируя вопрос разумно с использованием имени или описания аргумента. Если вы хотите настроить вопрос, используемый для получения обязательного аргумента, реализуйте метод `promptForMissingArgumentsUsing`, возвращающий массив вопросов с ключами соответствующими именам аргументов:

    /**
     * Prompt for missing input arguments using the returned questions.
     *
     * @return array
     */
    protected function promptForMissingArgumentsUsing()
    {
        return [
            'user' => 'Which user ID should receive the mail?',
        ];
    }

Вы также можете указать текст заполнителя, используя кортеж, содержащий вопрос и заполнитель:

    return [
        'user' => ['Which user ID should receive the mail?', 'E.g. 123'],
    ];


Если вы хотите полностью контролировать запрос, вы можете предоставить замыкание, которое будет запрашивать пользователя и возвращать его ответ:

    use App\Models\User;
    use function Laravel\Prompts\search;

    // ...

    return [
        'user' => fn () => search(
            label: 'Search for a user:',
            placeholder: 'E.g. Taylor Otwell',
            options: fn ($value) => strlen($value) > 0
                ? User::where('name', 'like', "%{$value}%")->pluck('name', 'id')->all()
                : []
        ),
    ];

> [!NOTE]
> Подробная документация по [Laravel Prompts](/docs/{{version}}/prompts) содержит дополнительную информацию о доступных запросах и их использовании.


Если вы хотите запросить у пользователя выбор или ввод опций, вы можете включить подсказки в метод `handle` вашей команды. Однако, если вы хотите запрашивать у пользователя только тогда, когда ему было автоматически предложено ввести отсутствующие аргументы, вы можете реализовать метод `afterPromptingForMissingArguments`:

    use Symfony\Component\Console\Input\InputInterface;
    use Symfony\Component\Console\Output\OutputInterface;
    use function Laravel\Prompts\confirm;

    // ...

    /**
     * Выполнить действия после запроса пользователя относительно отсутствующих аргументов
     *
     * @param  \Symfony\Component\Console\Input\InputInterface  $input
     * @param  \Symfony\Component\Console\Output\OutputInterface  $output
     * @return void
     */
    protected function afterPromptingForMissingArguments(InputInterface $input, OutputInterface $output)
    {
        $input->setOption('queue', confirm(
            label: 'Would you like to queue the mail?',
            default: $this->option('queue')
        ));
    }

<a name="command-io"></a>
## Ввод/вывод команды

<a name="retrieving-input"></a>
### Получение входных данных

Во время выполнения команды вам, вероятно, потребуется получить доступ к значениям аргументов и параметров, принятых командой. Для этого вы можете использовать методы `argument` и `option`. Если аргумент или параметр не существует, то будет возвращено значение `null`.

    /**
     * Выполнить консольную команду.
     */
    public function handle(): void
    {
        $userId = $this->argument('user');

        //
    }

Если вам нужно получить все аргументы в виде массива, вызовите метод `arguments`:

    $arguments = $this->arguments();

Параметры могут быть получены так же легко, как и аргументы, используя метод `option`. Чтобы получить все параметры в виде массива, вызовите метод `options`:

    // Получение определенного параметра ...
    $queueName = $this->option('queue');

    // Получение всех параметров в виде массива ...
    $options = $this->options();

<a name="prompting-for-input"></a>
### Запрос для ввода данных
> [!NOTE]
> [Laravel Prompts](/docs/{{version}}/prompts)  это PHP-пакет для добавления красивых и удобных форм в ваше консольное приложение с функциями, аналогичными браузеру, включая текст заполнителя и проверку данных.

Помимо отображения вывода, вы можете попросить пользователя предоставить данные во время выполнения вашей команды. Метод `ask` отобразит пользователю указанный вопрос, примет его ввод, а затем вернет эти данные, полученные от пользователя, обратно в команду:

    /**
     * Выполнить консольную команду.
     */
    public function handle(): void
    {
        $name = $this->ask('What is your name?');

        // ...
    }


Метод `ask` также принимает необязательный второй аргумент, который определяет значение по умолчанию, возвращаемое, если пользователь не предоставил ввод:

    $name = $this->ask('What is your name?', 'Taylor');

Метод `secret` похож на `ask`, но ввод пользователя не будет виден ему в консоли при вводе. Этот метод полезен при запросе конфиденциальной информации, например, пароля:

    $password = $this->secret('What is the password?');

<a name="asking-for-confirmation"></a>
#### Запрос подтверждения

Если вам нужно получить от пользователя простое подтверждение «yes or no», то вы можете использовать метод `confirm`. По умолчанию этот метод возвращает значение `false`. Однако, если пользователь вводит `y` или `yes` в ответ на запрос, то метод возвращает `true`.

    if ($this->confirm('Do you wish to continue?')) {
        // ...
    }

По желанию можно указать, что запрос подтверждения должен по умолчанию возвращать `true`, передав `true` в качестве второго аргумента метода `confirm`:

    if ($this->confirm('Do you wish to continue?', true)) {
        // ...
    }

<a name="auto-completion"></a>
#### Автозавершение

Метод `anticipate` используется для автоматического завершения возможных вариантов. Пользователь по-прежнему может дать любой ответ, независимо от подсказок автозавершения:

    $name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);

В качестве альтернативы, вы можете передать замыкание в качестве второго аргумента метода `anticipate`. Замыкание будет вызываться каждый раз, когда пользователь вводит символ. Замыкание должно принимать строковый параметр, содержащий введенные пользователем данные, и возвращать массив вариантов для автозавершения:

    $name = $this->anticipate('What is your address?', function (string $input) {
        // Вернуть варианты для автоматического завершения ...
    });

<a name="multiple-choice-questions"></a>
#### Вопросы с множественным выбором

Если нужно предоставить пользователю предопределенный набор вариантов для выбора при задании вопроса, то используйте метод `choice`. Вы можете установить индекс массива для возвращаемого по умолчанию значения, если не выбран ни один из вариантов, передав индекс в качестве третьего аргумента метода:

    $name = $this->choice(
        'What is your name?',
        ['Taylor', 'Dayle'],
        $defaultIndex
    );

Кроме того, метод `choice` принимает необязательные четвертый и пятый аргументы для определения максимального количества попыток выбора действительного ответа и того, разрешен ли множественный выбор:

    $name = $this->choice(
        'What is your name?',
        ['Taylor', 'Dayle'],
        $defaultIndex,
        $maxAttempts = null,
        $allowMultipleSelections = false
    );

<a name="writing-output"></a>
### Вывод данных

Чтобы вывести в консоль, используйте методы `line`, `info`, `comment`, `question`, `warn` и `error`. Каждый из этих методов будет использовать соответствующие ANSI-цвета. Например, давайте покажем пользователю некоторую общую информацию. Обычно метод `info` отображается в консоли в виде зеленого текста:

    /**
     * Выполнить консольную команду.
     */
    public function handle(): void
    {
        // ...

        $this->info('The command was successful!');
    }

Для отображения сообщения об ошибке используйте метод `error`. Текст сообщения об ошибке обычно отображается красным цветом:

    $this->error('Something went wrong!');

Вы можете использовать метод `line` для отображения простого неокрашенного текста:

    $this->line('Display this on the screen');

Вы можете использовать метод `newLine` для отображения пустой строки:

    // Вывести одну пустую строку ...
    $this->newLine();

    // Вывести три пустые строки ...
    $this->newLine(3);

<a name="tables"></a>
#### Таблицы

Метод `table` упрощает корректное форматирование нескольких строк / столбцов данных. Все, что вам нужно сделать, это указать имена столбцов и данные для таблицы, и Laravel автоматически рассчитает подходящую ширину и высоту таблицы:
<!--  -->

    use App\Models\User;

    $this->table(
        ['Name', 'Email'],
        User::all(['name', 'email'])->toArray()
    );

<a name="progress-bars"></a>
#### Индикаторы выполнения

Для длительно выполняемых задач было бы полезно показать индикатор выполнения, информирующий пользователя о том, насколько завершена задача. Используя метод `withProgressBar`, Laravel будет отображать индикатор выполнения и продвигать его для каждой итерации на заданное повторяемое значение:

    use App\Models\User;

    $users = $this->withProgressBar(User::all(), function (User $user) {
        $this->performTask($user);
    });

Иногда может потребоваться больший контроль над продвижением индикатора выполнения. Сначала определите общее количество шагов, через которые будет проходить процесс. Затем продвигайте индикатор выполнения после обработки каждого элемента:

    $users = App\Models\User::all();

    $bar = $this->output->createProgressBar(count($users));

    $bar->start();

    foreach ($users as $user) {
        $this->performTask($user);

        $bar->advance();
    }

    $bar->finish();

> [!NOTE]
> Для получения дополнительной информации ознакомьтесь с [разделом документации компонента Symfony Progress Bar](https://symfony.com/doc/current/components/console/helpers/progressbar.html).

<a name="registering-commands"></a>
## Регистрация команд

Все ваши консольные команды должны быть зарегистрированы в классе `App\Console\Kernel`, который является «ядром консоли» вашего приложения. Внутри метода `commands` этого класса вы увидите вызов метода `load` ядра. Метод `load` просканирует каталог `app/Console/Commands` и автоматически зарегистрирует каждую содержащуюся в нем команду в Artisan. Фактически, вы можете делать дополнительные вызовы метода `load` для сканирования других каталогов на наличие команд Artisan:

    /**
     * Зарегистрировать команды приложения.
     */
    protected function commands(): void
    {
        $this->load(__DIR__.'/Commands');
        $this->load(__DIR__.'/../Domain/Orders/Commands');

        // ...
    }

Вы можете самостоятельно зарегистрировать команды, добавив название класса команды в свойство `$commands` класса `App\Console\Kernel`. Если это свойство еще не определено в вашем ядре, вы должны определить его вручную. При загрузке Artisan, все команды, перечисленные в этом свойстве будут доступны в [контейнере служб](/docs/{{version}}/container) и зарегистрированы в Artisan:

    protected $commands = [
        Commands\SendEmails::class
    ];

<a name="programmatically-executing-commands"></a>
## Программное выполнение команд

По желанию можно выполнить команду Artisan за пределами CLI. Например, вы можете запустить команду Artisan в маршруте или контроллере. Для этого можно использовать метод `call` фасада `Artisan`. Метод `call` принимает в качестве первого аргумента либо имя сигнатуры команды, либо имя класса, а в качестве второго – массив параметров команды. Будет возвращен код выхода / возврата:

    use Illuminate\Support\Facades\Artisan;

    Route::post('/user/{user}/mail', function (string $user) {
        $exitCode = Artisan::call('mail:send', [
            'user' => $user, '--queue' => 'default'
        ]);

        //...
    });

Кроме того, вы можете передать методу `call` команду полностью в виде строки:

    Artisan::call('mail:send 1 --queue=default');

<a name="passing-array-values"></a>
#### Передача массива значений

Если ваша команда определяет параметр, который принимает массив, то вы можете передать массив значений этому параметру:

    use Illuminate\Support\Facades\Artisan;

    Route::post('/mail', function () {
        $exitCode = Artisan::call('mail:send', [
            '--id' => [5, 13]
        ]);
    });

<a name="passing-boolean-values"></a>
#### Передача значений логического типа

Если необходимо указать значение параметра, который не принимает строковые значения, например флаг `--force` в команде `migrate:refresh`, то вы должны передать `true` или `false` как значение параметра:

    $exitCode = Artisan::call('migrate:refresh', [
        '--force' => true,
    ]);

<a name="queueing-artisan-commands"></a>
#### Очереди команд Artisan

Используя метод `queue` фасада `Artisan`, вы можете даже поставить команды Artisan в очередь, чтобы они обрабатывались в фоновом режиме [обработчиком очереди](/docs/{{version}}/queues). Перед использованием этого метода убедитесь, что вы настроили очереди и был запущен слушатель очереди:

    use Illuminate\Support\Facades\Artisan;

    Route::post('/user/{user}/mail', function (string $user) {
        Artisan::queue('mail:send', [
            'user' => $user, '--queue' => 'default'
        ]);

        //...
    });

Используя методы `onConnection` и `onQueue`, вы также можете указать соединение или очередь, в которую должна быть отправлена команда Artisan:

    Artisan::queue('mail:send', [
        'user' => 1, '--queue' => 'default'
    ])->onConnection('redis')->onQueue('commands');

<a name="calling-commands-from-other-commands"></a>
### Вызов команд из других команд

По желанию можно вызвать другие команды из существующей команды Artisan. Вы можете сделать это с помощью метода `call`. Метод `call` принимает имя команды и массив аргументов / параметров команды:

    /**
     * Выполнить консольную команду.
     */
    public function handle(): void
    {
        $this->call('mail:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //...
    }

Если вы хотите вызвать другую консольную команду в тихом режиме, то используйте метод `callSilently`. Метод `callSilently` имеет ту же сигнатуру, что и метод `call`:

    $this->callSilently('mail:send', [
        'user' => 1, '--queue' => 'default'
    ]);

<a name="signal-handling"></a>
## Обработка сигналов

Как вы, возможно, знаете, операционные системы позволяют отправлять сигналы запущенным процессам. Например, сигнал `SIGTERM` используется операционными системами для запроса программе о завершении выполнения. Если вы хотите прослушивать сигналы в ваших консольных командах Artisan и выполнять код при их возникновении, вы можете использовать метод `trap`:

    /**
     * Выполнить консольную команду.
     */
    public function handle(): void
    {
        $this->trap(SIGTERM, fn () => $this->shouldKeepRunning = false);

        while ($this->shouldKeepRunning) {
            // ...
        }
    }


Для прослушивания нескольких сигналов сразу, вы можете предоставить массив сигналов методу `trap`:

    $this->trap([SIGTERM, SIGQUIT], function (int $signal) {
        $this->shouldKeepRunning = false;

        dump($signal); // SIGTERM / SIGQUIT
    });

<a name="stub-customization"></a>
## Настройка заготовок команд (stubs)

Команды `make` консоли Artisan используются для создания различных классов, таких как контроллеры, задания, миграции и тесты. Эти классы создаются с помощью файлов «заготовок», которые заполняются значениями на основе ваших входных данных. Однако, иногда может потребоваться внести небольшие изменения в файлы, создаваемые с помощью Artisan. Для этого можно использовать команду `stub:publish`, чтобы опубликовать наиболее распространенные заготовки для их дальнейшего изменения:

```shell
php artisan stub:publish
```

Опубликованные заготовки будут расположены в каталоге `stubs` корня вашего приложения. Любые изменения, внесенные вами в эти заготовки, будут учтены при создании соответствующих классов с помощью команд `make` Artisan.

<a name="events"></a>
## События

Artisan запускает три события при выполнении команд: `Illuminate\Console\Events\ArtisanStarting`, `Illuminate\Console\Events\CommandStarting`, и `Illuminate\Console\Events\CommandFinished`. Событие `ArtisanStarting` выполняется сразу после запуска Artisan. Затем событие `CommandStarting` выполняется непосредственно перед запуском команды. Наконец, событие `CommandFinished` выполняется после завершения команды.
