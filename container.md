# Service Container

- [Introdução](#introduction)
- [Uso Básico](#basic-usage)
- [Ligação de Interfaces para Implementações](#binding-interfaces-to-implementations)
- [Ligações Contextuais](#contextual-binding)
- [Tagging](#tagging)
- [Aplicações Práticas](#practical-applications)
- [Container de Eventos](#container-events)

<a name="introduction"></a>
## Introdução

O serviço Laravel container é uma ferramenta poderosa para gerenciar dependencia de classes. Injeção de dependeência é uma palavra bonita que essencialmente que dizer: Dependencia de classes são injetadas na classe via método contrutor ou, em alguns casos, via métodos "setter".

Vamos da uma olhada neste exemplo:

	<?php namespace App\Handlers\Commands;

	use App\User;
	use App\Commands\PurchasePodcastCommand;
	use Illuminate\Contracts\Mail\Mailer;

	class PurchasePodcastHandler {

		/**
		 * The mailer implementation.
		 */
		protected $mailer;

		/**
		 * Create a new instance.
		 *
		 * @param  Mailer  $mailer
		 * @return void
		 */
		public function __construct(Mailer $mailer)
		{
			$this->mailer = $mailer;
		}

		/**
		 * Purchase a podcast.
		 *
		 * @param  PurchasePodcastCommand  $command
		 * @return void
		 */
		public function handle(PurchasePodcastCommand $command)
		{
			//
		}

	}

Neste exemplo, o manipulador de comando `PurchasePodcast` precisar enviar e-mail quando um podcast for comprado. Então, nos iremos **injetar** o serviço que é hábil a enviar e-mails. Desde que o serviço é injetado, nos podemos facilmente trocar isto por uma outra implementação. Nos também podemos facilmente "mockar", ou criar uma implementação dummy do enviador de e-mail quando estivermos testando a nossa aplicação.

Um profundo entendimento do container de serviços do Laravel é essencial para criar uma poderosa e robusta aplicação, bem como para contribuir para o próprio núcleo Laravel. 

<a name="basic-usage"></a>
## Uso Básico

### Bindings

Quase todas as bindings(ligações) do seu container de serviço serão registratos nos [provedores de serviço](/docs/{{version}}/providers), então todos estes exemplos demostrarão o uso do container neste contexto. No entando, se você precisar de uma instância do container em um outro lugar da sua aplicação, como uma factory, você pode tipar o contrato `Illuminate\Contracts\Container\Container`  e a instância do container irá ser injetada para você. Alternativamente, você pode usar a fachada `App` para acessar o container. 

#### Registrando Um Resolvedor Básico 

Dentro de um provedor de serviço, você sempre tem acesso ao container via a variável de instância `$this->app`. 

Existem várias maneiras de o container de serviços possa registrar dependências, incluindo Closure callbacks e binding(ligações) de interfaces para implementações. Primeiro, nos vamos explorar Closure callbacks. Um resolver Closure é registrado em um container com a chave (normalmente o nome da classe) e a Closure que retorna algum valor:

	$this->app->bind('FooBar', function($app)
	{
		return new FooBar($app['SomethingElse']);
	});

#### Registrando um Singleton

Algumas vezes, você pode desejar bing(ligar) algumas coisa ao container que deve apenas ser resolvido uma vez, e a mesma instância pode ser retornada em chamadas subsequentes para o container. 

	$this->app->singleton('FooBar', function($app)
	{
		return new FooBar($app['SomethingElse']);
	});

#### Binding(Ligar) Uma Instância Existente ao Container. 

Você pode também bind um objeto intância de objeto existente no container usando o método `instance`. A instância dada sempre será retornada em chamadas subsequentes para o container:

	$fooBar = new FooBar(new SomethingElse);

	$this->app->instance('FooBar', $fooBar);

### Resolvendo

Há vários modos de se resolver alguma coisa fora do container. Primeiro, você pode usar o método `make`:

	$fooBar = $this->app->make('FooBar');

Segundo, você pode usar "array access"(acesso array) no container, desde que isto implemente `ArrayAccess` interface PHP's:

	$fooBar = $this->app['FooBar'];

Por fim, mas o mais importante, você pode simplesmente "type-hint"(tipar) a dependência no méotodo contrutor da classe que é resolvido pelo container, incluindo controladores, listeners de eventos, queue jobs, filtros, e mais. O container irá automaticamente injetar as dependências. 

	<?php namespace App\Http\Controllers;

	use Illuminate\Routing\Controller;
	use App\Users\Repository as UserRepository;

	class UserController extends Controller {

		/**
		 * The user repository instance.
		 */
		protected $users;

		/**
		 * Create a new controller instance.
		 *
		 * @param  UserRepository  $users
		 * @return void
		 */
		public function __construct(UserRepository $users)
		{
			$this->users = $users;
		}

		/**
		 * Show the user with the given ID.
		 *
		 * @param  int  $id
		 * @return Response
		 */
		public function show($id)
		{
			//
		}

	}

<a name="binding-interfaces-to-implementations"></a>
## Binding Interfaces Para Implementações 

### Injetando Dependências Concretas

Uma funcioanalidade muito poderosa do container de serviços é a abilidade de bind(ligar) uma interface a uma implementação dada. Por exemplo, talvez nossa aplicação se integra com o webservice [Pusher](https://pusher.com) para mandar e receber eventos em tempo-real. Se nos estivermos usando o PHP SDK do Pusher's, nos podemos injetar uma instância do cliente Pusher a uma classe. 

	<?php namespace App\Handlers\Commands;

	use App\Commands\CreateOrder;
	use Pusher\Client as PusherClient;

	class CreateOrderHandler {

		/**
		 * The Pusher SDK client instance.
		 */
		protected $pusher;

		/**
		 * Create a new order handler instance.
		 *
		 * @param  PusherClient  $pusher
		 * @return void
		 */
		public function __construct(PusherClient $pusher)
		{
			$this->pusher = $pusher;
		}

		/**
		 * Execute the given command.
		 *
		 * @param  CreateOrder  $command
		 * @return void
		 */
		public function execute(CreateOrder $command)
		{
			//
		}

	}
Neste exemplo, é bom que estajamos injetando dependência de classes, no entanto, nos firmemente acoplanos ao SDK do Pusher. Se o método do pusher SDK, os métodos mudam ou nos decidimos trocar para um novo evento,no iremos precisar mudar o código `CreateOrderHandler`.

### Program To An Interface

In order to "insulate" the `CreateOrderHandler` against changes to event pushing, we could define an `EventPusher` interface and a `PusherEventPusher` implementation:

	<?php namespace App\Contracts;

	interface EventPusher {

		/**
		 * Push a new event to all clients.
		 *
		 * @param  string  $event
		 * @param  array  $data
		 * @return void
		 */
		public function push($event, array $data);

	}

Once we have coded our `PusherEventPusher` implementation of this interface, we can register it with the service container like so:

	$this->app->bind('App\Contracts\EventPusher', 'App\Services\PusherEventPusher');

This tells the container that it should inject the `PusherEventPusher` when a class needs an implementation of `EventPusher`. Now we can type-hint the `EventPusher` interface in our constructor:

		/**
		 * Create a new order handler instance.
		 *
		 * @param  EventPusher  $pusher
		 * @return void
		 */
		public function __construct(EventPusher $pusher)
		{
			$this->pusher = $pusher;
		}

<a name="contextual-binding"></a>
## Contextual Binding

Sometimes you may have two classes that utilize the same interface, but you wish to inject different implementations into each class. For example, when our system receives a new Order, we may want to send an event via [PubNub](http://www.pubnub.com/) rather than Pusher. Laravel provides a simple, fluent interface for defining this behavior:

	$this->app->when('App\Handlers\Commands\CreateOrderHandler')
	          ->needs('App\Contracts\EventPusher')
	          ->give('App\Services\PubNubEventPusher');

<a name="tagging"></a>
## Tagging

Occasionally, you may need to resolve all of a certain "category" of binding. For example, perhaps you are building a report aggregator that receives an array of many different `Report` interface implementations. After registering the `Report` implementations, you can assign them a tag using the `tag` method:

	$this->app->bind('SpeedReport', function()
	{
		//
	});

	$this->app->bind('MemoryReport', function()
	{
		//
	});

	$this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');

Once the services have been tagged, you may easily resolve them all via the `tagged` method:

	$this->app->bind('ReportAggregator', function($app)
	{
		return new ReportAggregator($app->tagged('reports'));
	});

<a name="practical-applications"></a>
## Practical Applications

Laravel provides several opportunities to use the service container to increase the flexibility and testability of your application. One primary example is when resolving controllers. All controllers are resolved through the service container, meaning you can type-hint dependencies in a controller constructor, and they will automatically be injected.

	<?php namespace App\Http\Controllers;

	use Illuminate\Routing\Controller;
	use App\Repositories\OrderRepository;

	class OrdersController extends Controller {

		/**
		 * The order repository instance.
		 */
		protected $orders;

		/**
		 * Create a controller instance.
		 *
		 * @param  OrderRepository  $orders
		 * @return void
		 */
		public function __construct(OrderRepository $orders)
		{
			$this->orders = $orders;
		}

		/**
		 * Show all of the orders.
		 *
		 * @return Response
		 */
		public function index()
		{
			$orders = $this->orders->all();

			return view('orders', ['orders' => $orders]);
		}

	}

In this example, the `OrderRepository` class will automatically be injected into the controller. This means that a "mock" `OrderRepository` may be bound into the container when [unit testing](/docs/{{version}}/testing), allowing for painless stubbing of database layer interaction.

#### Other Examples Of Container Usage

Of course, as mentioned above, controllers are not the only classes Laravel resolves via the service container. You may also type-hint dependencies on route Closures, filters, queue jobs, event listeners, and more. For examples of using the service container in these contexts, please refer to their documentation.

<a name="container-events"></a>
## Container Events

#### Registering A Resolving Listener

The container fires an event each time it resolves an object. You may listen to this event using the `resolving` method:

	$this->app->resolving(function($object, $app)
	{
		// Called when container resolves object of any type...
	});

	$this->app->resolving(function(FooBar $fooBar, $app)
	{
		// Called when container resolves objects of type "FooBar"...
	});

The object being resolved will be passed to the callback.
