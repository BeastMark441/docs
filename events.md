---
git: 46c2634ef5a4f15427c94a3157b626cf5bd3937f
---

# События (Events)

<a name="introduction"></a>
## Введение

События Laravel обеспечивают простую реализацию шаблона Наблюдатель, позволяя вам подписываться и отслеживать различные события, происходящие в вашем приложении. Классы событий обычно хранятся в каталоге `app/Events`, а их слушатели – в `app/Listeners`. Не беспокойтесь, если вы не видите эти каталоги в своем приложении, так как они будут созданы для вас, когда вы будете генерировать события и слушатели с помощью команд консоли Artisan.

События служат отличным способом разделения различных аспектов вашего приложения, поскольку одно событие может иметь несколько слушателей, которые не зависят друг от друга. Например, бывает необходимо отправлять уведомление Slack своему пользователю каждый раз, когда заказ будет отправлен. Вместо того чтобы связывать код обработки заказа с кодом уведомления Slack, вы можете вызвать событие `App\Events\OrderShipped`, которое слушатель может получить и использовать для отправки уведомления Slack.

<a name="registering-events-and-listeners"></a>
## Регистрация событий и слушателей

Поставщик `App\Providers\EventServiceProvider` Laravel – удобное место для регистрации всех слушателей событий вашего приложения. Свойство `$listen` содержит массив всех событий (ключей) и их слушателей (значений). Вы можете добавить в этот массив столько событий, сколько требуется вашему приложению. Например, добавим событие `OrderShipped`:

    use App\Events\OrderShipped;
    use App\Listeners\SendShipmentNotification;

    /**
     * Карта слушателей событий приложения.
     *
     * @var array<class-string, array<int, class-string>>
     */
    protected $listen = [
        OrderShipped::class => [
            SendShipmentNotification::class,
        ],
    ];

> [!NOTE]  
> Команда `event:list` используется для отображения списка всех событий и слушателей, зарегистрированных вашим приложением.

<a name="generating-events-and-listeners"></a>
### Генерация событий и слушателей

Конечно, вручную создавать файлы для каждого события и слушателя сложно. Вместо этого добавьте необходимые события и их слушатели в поставщике `EventServiceProvider`, затем, используйте команду `event:generate` [Artisan](artisan). Эта команда сгенерирует любые события или слушатели, перечисленные в поставщике `EventServiceProvider`, но которые еще не существуют:

```shell
php artisan event:generate
```

В качестве альтернативы вы можете использовать команды `make:event` и `make:listener` Artisan для генерации отдельных событий и слушателей:

```shell
php artisan make:event PodcastProcessed

php artisan make:listener SendPodcastNotification --event=PodcastProcessed
```

<a name="manually-registering-events"></a>
### Ручная регистрация событий

Обычно события должны регистрироваться через массив `$listen` поставщика `EventServiceProvider`; но вы также можете явно зарегистрировать слушателей событий на основе классов или замыканий в методе `boot` вашего `EventServiceProvider`:

    use App\Events\PodcastProcessed;
    use App\Listeners\SendPodcastNotification;
    use Illuminate\Support\Facades\Event;

    /**
     * Регистрация любых событий вашего приложения.
     */
    public function boot(): void
    {
        Event::listen(
            PodcastProcessed::class,
            SendPodcastNotification::class,
        );

        Event::listen(function (PodcastProcessed $event) {
            // ...
        });
    }

<a name="queuable-anonymous-event-listeners"></a>
#### Анонимные слушатели событий в очереди

При явной регистрации слушателей событий на основе замыкания вы можете обернуть замыкание слушателя в функцию `Illuminate\Events\queueable`, чтобы проинструктировать Laravel о выполнении слушателя с использованием [очереди](/docs/{{version}}/queues):

    use App\Events\PodcastProcessed;
    use function Illuminate\Events\queueable;
    use Illuminate\Support\Facades\Event;

    /**
     * Регистрация любых событий вашего приложения.
     */
    public function boot(): void
    {
        Event::listen(queueable(function (PodcastProcessed $event) {
            // ...
        }));
    }

Как и в случае с заданиями в очередях, вы можете использовать методы `onConnection`, `onQueue` и `delay` для детализации выполнения слушателя в очереди:

    Event::listen(queueable(function (PodcastProcessed $event) {
        // ...
    })->onConnection('redis')->onQueue('podcasts')->delay(now()->addSeconds(10)));

Если вы хотите обрабатывать сбои анонимного слушателя в очереди, то вы можете передать замыкание методу `catch` при определении слушателя `queueable`. Это замыкание получит экземпляр события и экземпляр `Throwable`, вызвавший сбой слушателя:

    use App\Events\PodcastProcessed;
    use function Illuminate\Events\queueable;
    use Illuminate\Support\Facades\Event;
    use Throwable;

    Event::listen(queueable(function (PodcastProcessed $event) {
        // ...
    })->catch(function (PodcastProcessed $event, Throwable $e) {
        // Событие в очереди завершилось неудачно ...
    }));

<a name="wildcard-event-listeners"></a>
#### Анонимные слушатели группы событий

Допускается использование метасимвола подстановки `*` при регистрации анонимных слушателей, что позволит вам перехватывать несколько событий, используя единый слушатель. Слушатель, зарегистрированный с помощью данного синтаксиса, получает имя текущего события в качестве первого аргумента и весь массив данных события в качестве второго аргумента:

    Event::listen('event.*', function (string $eventName, array $data) {
        // ...
    });

<a name="event-discovery"></a>
### Автообнаружение событий

Вместо того чтобы вручную регистрировать события и слушателей в массиве `$listen` поставщика `EventServiceProvider`, вы можете включить автоматическое обнаружение событий. Когда обнаружение событий включено, Laravel автоматически найдет и зарегистрирует ваши события и слушатели, сканируя каталог `Listeners` вашего приложения. Кроме того, любые явно определенные события, перечисленные в `EventServiceProvider`, по-прежнему будут зарегистрированы.

Laravel находит слушателей событий, сканируя классы слушателей с помощью рефлексии PHP.
Когда Laravel находит какой-либо метод класса слушателя, который начинается с `handle` или `__invoke`, Laravel зарегистрирует эти методы как слушатели событий для события, тип которого указан в сигнатуре метода:

    use App\Events\PodcastProcessed;

    class SendPodcastNotification
    {
        /**
         * Обработать переданное событие.
         */
        public function handle(PodcastProcessed $event): void
        {
            // ...
        }
    }

Обнаружение событий по умолчанию отключено, но вы можете включить его, переопределив метод `shouldDiscoverEvents` вашего `EventServiceProvider`:

    /**
     * Определить, должны ли автоматически обнаруживаться события и слушатели.
     */
    public function shouldDiscoverEvents(): bool
    {
        return true;
    }

По умолчанию будут сканироваться все слушатели в каталоге `app/Listeners` вашего приложения. Если вы хотите определить дополнительные каталоги для сканирования, вы можете переопределить метод `discoverEventsWithin` вашего `EventServiceProvider`:

    /**
     * Получить каталоги слушателей, которые следует использовать для обнаружения событий.
     *
     * @return array<int, string>
     */
    protected function discoverEventsWithin(): array
    {
        return [
            $this->app->path('Listeners'),
        ];
    }

<a name="event-discovery-in-production"></a>
#### Кеширование событий

В эксплуатационном окружении для фреймворка неэффективно сканировать всех ваших слушателей по каждому запросу. Следовательно, в процессе развертывания вы должны запустить команду `event:cache` Artisan, чтобы кешировать манифест всех событий и слушателей вашего приложения. Этот манифест будет использоваться фреймворком для ускорения процесса регистрации события. Команда `event:clear` используется для уничтожения кеша.

<a name="defining-events"></a>
## Определение событий

Класс событий – это, по сути, контейнер данных, который содержит информацию, относящуюся к событию. Например, предположим, что событие `App\Events\OrderShipped` получает объект [Eloquent ORM](/docs/{{version}}/eloquent):

    <?php

    namespace App\Events;

    use App\Models\Order;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Foundation\Events\Dispatchable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped
    {
        use Dispatchable, InteractsWithSockets, SerializesModels;

        /**
         * Создать новый экземпляр события.
         */
        public function __construct(
            public Order $order,
        ) {}
    }

Как видите, в этом классе событий нет логики. Это контейнер для экземпляра `App\Models\Order` заказа, который был выполнен. Трейт `SerializesModels`, используемый событием, будет изящно сериализовать любые модели Eloquent, если объект события сериализуется с использованием функции `serialize` PHP, например, при использовании [слушателей в очереди](#queued-event-listeners).

<a name="defining-listeners"></a>
## Определение слушателей

Затем, давайте посмотрим на слушателя для нашего примера события. Слушатели событий получают экземпляры событий в своем методе `handle`. Команды `event:generate` и `make:listener` Artisan автоматически импортируют соответствующий класс события и внедрят событие в методе `handle`. В методе `handle` вы можете выполнять любые действия, необходимые для ответа на событие:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;

    class SendShipmentNotification
    {
        /**
         * Создать слушателя событий.
         */
        public function __construct()
        {
            // ...
        }

        /**
         * Обработать событие.
         */
        public function handle(OrderShipped $event): void
        {
            // Доступ к заказу с помощью `$event->order` ...
        }
    }

> [!NOTE]   
> В конструкторе ваших слушателей событий могут быть объявлены любые необходимые типы зависимостей. Все слушатели событий разрешаются через [контейнер служб](/docs/{{version}}/container) Laravel, поэтому зависимости будут внедрены автоматически.

<a name="stopping-the-propagation-of-an-event"></a>
#### Остановка распространения события

По желанию можно остановить распространение события среди других слушателей. Вы можете сделать это, вернув `false` из метода `handle` вашего слушателя.

<a name="queued-event-listeners"></a>
## Слушатели событий в очереди

Слушатели в очереди могут быть полезны, если ваш слушатель собирается выполнять медленную задачу, такую как отправка электронной почты или выполнение HTTP-запроса. Перед использованием слушателей в очереди убедитесь, что вы [сконфигурировали очередь](/docs/{{version}}/queues) и запустили обработчик очереди на вашем сервере или в локальной среде разработки.

Чтобы указать, что слушатель должен быть поставлен в очередь, добавьте интерфейс `ShouldQueue` в класс слушателя. Слушатели, сгенерированные командами `event:generate` и `make:listener` Artisan, уже будут иметь этот интерфейс, импортируемый в текущее пространство имен, поэтому вы можете использовать его немедленно:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        // ...
    }

Это все! Теперь, когда отправляется событие, обрабатываемое этим слушателем, слушатель автоматически ставится в очередь диспетчером событий с использованием [системы очередей](/docs/{{version}}/queues) Laravel. Если при выполнении слушателя в очереди не возникает никаких исключений, задание в очереди будет автоматически удалено после завершения обработки.

<a name="customizing-the-queue-connection-queue-name"></a>
#### Настройка соединения очереди, имени, и времени задержки

Если вы хотите настроить соединение очереди, имя очереди или время задержки очереди для слушателя событий, то вы можете определить свойства `$connection`, `$queue`, или `$delay` в своем классе слушателя:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        /**
         * Имя соединения, на которое должно быть отправлено задание.
         *
         * @var string|null
         */
        public $connection = 'sqs';

        /**
         * Имя очереди, в которую должно быть отправлено задание.
         *
         * @var string|null
         */
        public $queue = 'listeners';

        /**
         * Время (в секундах) до обработки задания.
         *
         * @var int
         */
        public $delay = 60;
    }

Если вы хотите определить соединение очереди слушателя или имя очереди слушателя во время выполнения, вы можете определить методы `viaConnection`, `viaQueue` или `withDelay` слушателя:

    /**
     * Получить имя подключения очереди слушателя.
     */
    public function viaConnection(): string
    {
        return 'sqs';
    }

    /**
     * Получить имя очереди слушателя.
     */
    public function viaQueue(): string
    {
        return 'listeners';
    }

    /**
     * Получить количество секунд до того, как задача должна быть выполнена.
     */
    public function withDelay(OrderShipped $event): int
    {
        return $event->highPriority ? 0 : 60;
    }

<a name="conditionally-queueing-listeners"></a>
#### Условная отправка слушателей в очередь

Иногда требуется определить, следует ли ставить слушателя в очередь на основе некоторых данных, доступных только во время выполнения. Для этого к слушателю может быть добавлен метод `shouldQueue`, чтобы определить, следует ли поставить слушателя в очередь. Если метод `shouldQueue` возвращает `false`, то слушатель не будет выполнен:

    <?php

    namespace App\Listeners;

    use App\Events\OrderCreated;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class RewardGiftCard implements ShouldQueue
    {
        /**
         * Наградить покупателя подарочной картой.
         */
        public function handle(OrderCreated $event): void
        {
            // ...
        }

        /**
         * Определить, следует ли ставить слушателя в очередь.
         */
        public function shouldQueue(OrderCreated $event): bool
        {
            return $event->order->subtotal >= 5000;
        }
    }

<a name="manually-interacting-with-the-queue"></a>
### Взаимодействие с очередью вручную

Если вам нужно вручную получить доступ к методам `delete` и `release` базового задания в очереди слушателя, вы можете сделать это с помощью трейта `Illuminate\Queue\InteractsWithQueue`. Этот трейт по умолчанию импортируется в сгенерированные слушатели и обеспечивает доступ к этим методам:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Queue\InteractsWithQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;

        /**
         * Обработать событие.
         */
        public function handle(OrderShipped $event): void
        {
            if (true) {
                $this->release(30);
            }
        }
    }

<a name="queued-event-listeners-and-database-transactions"></a>
### Слушатели событий в очереди и транзакции базы данных

Когда слушатели в очереди отправляются в транзакциях базы данных, они могут быть обработаны очередью до того, как транзакция базы данных будет зафиксирована. Когда это происходит, любые обновления, внесенные вами в модели или записи базы данных во время транзакции базы данных, могут еще не быть отражены в базе данных. Кроме того, любые модели или записи базы данных, созданные в рамках транзакции, могут не существовать в базе данных. Если ваш слушатель зависит от этих моделей, могут возникнуть непредвиденные ошибки при обработке задания, которое отправляет поставленный в очередь слушатель.

Если опция `after_commit` вашего соединения с очередью установлена в значение `false`, то вы все равно можете указать, что конкретный слушатель в очереди должен быть выполнен после того, как все открытые транзакции в базе данных будут завершены, реализовав интерфейс `ShouldHandleEventsAfterCommit` в классе слушателя:

    <?php

    namespace App\Listeners;

    use Illuminate\Contracts\Events\ShouldHandleEventsAfterCommit;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Queue\InteractsWithQueue;

    class SendShipmentNotification implements ShouldQueue, ShouldHandleEventsAfterCommit
    {
        use InteractsWithQueue;
    }

> [!NOTE]   
> Чтобы узнать больше о том, как обойти эти проблемы, просмотрите документацию, касающуюся [заданий в очереди и транзакций базы данных](/docs/{{version}}/queues#jobs-and-database-transactions).

<a name="handling-failed-jobs"></a>
### Обработка невыполненных заданий

Иногда ваши слушатели событий в очереди могут дать сбой. Если слушатель в очереди превышает максимальное количество попыток, определенное вашим обработчиком очереди, для вашего слушателя будет вызван метод `failed`. Метод `failed` получает экземпляр события и `Throwable`, вызвавший сбой:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Queue\InteractsWithQueue;
    use Throwable;

    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;

        /**
         * Обработать событие.
         */
        public function handle(OrderShipped $event): void
        {
            // ...
        }

        /**
         * Обработать провал задания.
         */
        public function failed(OrderShipped $event, Throwable $exception): void
        {
            // ...
        }
    }

<a name="specifying-queued-listener-maximum-attempts"></a>
#### Указание максимального количества попыток слушателя в очереди

Если один из ваших слушателей в очереди обнаруживает ошибку, вы, вероятно, не хотите, чтобы он продолжал повторять попытки бесконечно. Таким образом, Laravel предлагает различные способы указать, сколько раз и как долго может выполняться попытка прослушивания.

Вы можете определить свойство `$tries` в своем классе слушателя, чтобы указать, сколько раз можно попытаться выполнить слушатель, прежде чем он будет считаться неудачным:

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Queue\InteractsWithQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;

        /**
         * Количество попыток слушателя в очереди.
         *
         * @var int
         */
        public $tries = 5;
    }

В качестве альтернативы определению того, сколько раз можно попытаться выполнить слушатель, прежде чем он потерпит неудачу, вы можете определить время, через которое слушатель больше не должен выполняться. Это позволяет попытаться выполнить прослушивание любое количество раз в течение заданного периода времени. Чтобы определить время, через которое больше не следует предпринимать попытки прослушивания, добавьте метод `retryUntil` в свой класс слушателя. Этот метод должен возвращать экземпляр `DateTime`:

    use DateTime;

    /**
     * Определить время, через которое слушатель должен отключиться.
     *
     * @return \DateTime
     */
    public function retryUntil(): DateTime
    {
        return now()->addMinutes(5);
    }

<a name="dispatching-events"></a>
## Отправка событий

Чтобы отправить событие, вы можете вызвать статический метод `dispatch` события. Этот метод доступен в событии с помощью трейта `Illuminate\Foundation\Events\Dispatchable`. Любые аргументы, переданные методу `dispatch`, будут переданы конструктору события:

    <?php

    namespace App\Http\Controllers;

    use App\Events\OrderShipped;
    use App\Http\Controllers\Controller;
    use App\Models\Order;
    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;

    class OrderShipmentController extends Controller
    {
        /**
         * Отправить заказ.
         */
        public function store(Request $request): RedirectResponse
        {
            $order = Order::findOrFail($request->order_id);

            // Логика отправки заказа ...

            OrderShipped::dispatch($order);

            return redirect('/orders');
        }
    }

Если вы хотите условно отправить событие, вы можете использовать методы `dispatchIf` и `dispatchUnless`:

```php
OrderShipped::dispatchIf($condition, $order);

OrderShipped::dispatchUnless($condition, $order);
```

> [!NOTE]    
> При тестировании может быть полезным утверждать, что определенные события были отправлены, не активируя их слушателей. В Laravel это легко сделать с помощью [встроенных средств тестирования](#testing).

<a name="dispatching-events-after-database-transactions"></a>
### Отправка событий после транзакций в базе данных

Иногда вам может потребоваться указать Laravel отправлять событие только после завершения активной транзакции в базе данных. Для этого вы можете реализовать интерфейс `ShouldDispatchAfterCommit` в классе события.

Этот интерфейс указывает Laravel не отправлять событие, пока текущая транзакция в базе данных не будет завершена. Если транзакция завершится с ошибкой, событие будет отброшено. Если в момент отправки события нет активной транзакции в базе данных, событие будет отправлено немедленно.

    <?php

    namespace App\Events;

    use App\Models\Order;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Contracts\Events\ShouldDispatchAfterCommit;
    use Illuminate\Foundation\Events\Dispatchable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped implements ShouldDispatchAfterCommit
    {
        use Dispatchable, InteractsWithSockets, SerializesModels;

        /**
         * Create a new event instance.
         */
        public function __construct(
            public Order $order,
        ) {}
    }

<a name="event-subscribers"></a>
## Подписчики событий

<a name="writing-event-subscribers"></a>
### Написание подписчиков на события

Подписчики событий – это классы, которые могут подписываться на несколько событий, что позволяет вам определять несколько обработчиков событий в одном классе. Подписчики должны определить метод `subscribe`, которому будет передан экземпляр диспетчера событий. Вы можете вызвать метод `listen` данного диспетчера для регистрации слушателей событий:

    <?php

    namespace App\Listeners;

    use Illuminate\Auth\Events\Login;
    use Illuminate\Auth\Events\Logout;

    class UserEventSubscriber
    {
        /**
         * Обработать событие входа пользователя в систему.
         */
        public function handleUserLogin(Login $event): void {}

        /**
         * Обработать событие выхода пользователя из системы.
         */
        public function handleUserLogout(Logout $event): void {}

        /**
         * Зарегистрировать слушателей для подписчика.
         *
         * @param  \Illuminate\Events\Dispatcher  $events
         * @return void
         */
        public function subscribe(Dispatcher $events): void
        {
            $events->listen(
                Login::class,
                [UserEventSubscriber::class, 'handleUserLogin']
            );

            $events->listen(
                Logout::class,
                [UserEventSubscriber::class, 'handleUserLogout']
            );
        }
    }

Если ваши методы слушателей событий определены в самом подписчике, вам может быть удобнее возвращать массив событий и имен методов из метода подписчика `subscribe`. Laravel автоматически определит имя класса подписчика при регистрации слушателей событий:

    <?php

    namespace App\Listeners;

    use Illuminate\Auth\Events\Login;
    use Illuminate\Auth\Events\Logout;
    use Illuminate\Events\Dispatcher;

    class UserEventSubscriber
    {
        /**
         * Обработать событие входа пользователя в систему.
         */
        public function handleUserLogin(Login $event): void {}

        /**
         * Обработать событие выхода пользователя из системы.
         */
        public function handleUserLogout(Logout $event): void {}

        /**
         * Register the listeners for the subscriber.
         */
        public function subscribe(Dispatcher $events): array
        {
            return [
                Login::class => 'handleUserLogin',
                Logout::class => 'handleUserLogout',
            ];
        }
    }

<a name="registering-event-subscribers"></a>
### Регистрация подписчиков на события

После написания подписчика вы готовы зарегистрировать его в диспетчере событий. Вы можете зарегистрировать подписчиков с помощью свойства `$subscribe` поставщика `EventServiceProvider`. Например, добавим подписчик `UserEventSubscriber` в список:

    <?php

    namespace App\Providers;

    use App\Listeners\UserEventSubscriber;
    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

    class EventServiceProvider extends ServiceProvider
    {
        /**
         * Карта слушателей событий приложения.
         *
         * @var array
         */
        protected $listen = [
            // ...
        ];

        /**
         * Классы подписчиков для регистрации.
         *
         * @var array
         */
        protected $subscribe = [
            UserEventSubscriber::class,
        ];
    }

<a name="testing"></a>
## Тестирование

При тестировании кода, который отправляет события, может потребоваться указать Laravel не выполнять фактически слушателей событий, так как код слушателей можно тестировать непосредственно и отдельно от кода, который отправляет соответствующее событие. Конечно, для тестирования самого слушателя вы можете создать экземпляр слушателя и вызвать метод `handle` напрямую в вашем тесте.

Используя метод `fake` фасада `Event`, вы можете предотвратить выполнение слушателей, выполнить код, который требуется протестировать, и затем утверждать, какие события были отправлены вашим приложением с помощью методов `assertDispatched`, `assertNotDispatched` и `assertNothingDispatched`:

```php
<?php

namespace Tests\Feature;

use App\Events\OrderFailedToShip;
use App\Events\OrderShipped;
use Illuminate\Support\Facades\Event;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * Test order shipping.
     */
    public function test_orders_can_be_shipped(): void
    {
        Event::fake();

        // Выполните процесс доставки заказа...

        // Утвердите, что событие было отправлено...
        Event::assertDispatched(OrderShipped::class);

        // Утвердите, что событие было отправлено дважды...
        Event::assertDispatched(OrderShipped::class, 2);

        // Утвердите, что событие не было отправлено...
        Event::assertNotDispatched(OrderFailedToShip::class);

        // Утвердите, что не было отправлено ни одного события...
        Event::assertNothingDispatched();
    }
}
```

Вы можете передать замыкание в методы `assertDispatched` или `assertNotDispatched`, чтобы утверждать, что было отправлено событие, которое проходит заданный "тест истинности". Если хотя бы одно событие было отправлено и прошло заданный тест истинности, то утверждение будет успешным:

```php
Event::assertDispatched(function (OrderShipped $event) use ($order) {
    return $event->order->id === $order->id;
});
```

Если вы хотите просто утвердить, что слушатель события слушает определенное событие, вы можете использовать метод `assertListening`:

```php
Event::assertListening(
    OrderShipped::class,
    SendShipmentNotification::class
);
```

> [!WARNING]
> После вызова `Event::fake()`, слушатели событий не будут выполнены. Поэтому, если ваши тесты используют фабрики моделей, которые зависят от событий, например, создание UUID во время события `creating` модели, вы должны вызвать `Event::fake()` **после** использования ваших фабрик.

### Подмена определенного набора событий

Если вы хотите подменить слушателей событий только для определенного набора событий, вы можете передать их в метод `fake` или `fakeFor`:

```php
/**
 * Test order process.
 */
public function test_orders_can_be_processed(): void
{
    Event::fake([
        OrderCreated::class,
    ]);

    $order = Order::factory()->create();

    Event::assertDispatched(OrderCreated::class);

    // Другие события отправляются как обычно...
    $order->update([...]);
}
```

Вы можете подменить все события, кроме указанных событий, используя метод `except`:

```php
Event::fake()->except([
    OrderCreated::class,
]);
```

### Подмена событий в ограниченной области видимости

Если вы хотите подменить слушателей событий только в определенной части вашего теста, вы можете использовать метод `fakeFor`:

```php
<?php

namespace Tests\Feature;

use App\Events\OrderCreated;
use App\Models\Order;
use Illuminate\Support\Facades\Event;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * Test order process.
     */
    public function test_orders_can_be_processed(): void
    {
        $order = Event::fakeFor(function () {
            $order = Order::factory()->create();

            Event::assertDispatched(OrderCreated::class);

            return $order;
        });

        // События отправляются как обычно, и наблюдатели будут выполнены...
        $order->update([...]);
    }
}
```
