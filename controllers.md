# HTTP Controllers

- [Introdução](#introduction)
- [Básico de Controladores](#basic-controllers)
- [Controller Middleware](#controller-middleware)
- [Controllers Ímplicitos](#implicit-controllers)
- [RESTful Resource Controllers](#restful-resource-controllers)
- [Injeção de Dependências & Controllers](#dependency-injection-and-controllers)
- [Caching de Rotas](#route-caching)

<a name="introduction"></a>
## Introdução

Ao invés de definir todos a lógica das suas requisições em um único arquivo `routes.php`, você pode desejar organizar este comportamente usando classes Controllers ou Controlador. Controladores podem agrupar solicitações HTPP relacionadas a manipulação lógica de uma classe. Controladores são normalmente localizados no diretório `app/Http/Controllers`.

<a name="basic-controllers"></a>
## Básico de Controladores

Aqui vai um exemplo de uma classe básica de controlador:

	<?php namespace App\Http\Controllers;

	use App\Http\Controllers\Controller;

	class UserController extends Controller {

		/**
		 * Show the profile for the given user.
		 *
		 * @param  int  $id
		 * @return Response
		 */
		public function showProfile($id)
		{
			return view('user.profile', ['user' => User::findOrFail($id)]);
		}

	}

Nós pode encaminhar para a ação do controlador assim:

	Route::get('user/{id}', 'UserController@showProfile');

> **Nota:** Todos os controladores devem herdar da classe controladora base. 

#### Controladores & Namespaces

Isto é muito importante para notar que nos não precisavamos especificar o namespace do controlador por completo, apenas uma parte do nome da classe, apenas uma parte do nome da classe que vem depois `App\Http\Controllers` do namespace "root"(raíz). Por padrão, o `RouteServiceProvider` irá carregar o arquivo `routes.php` dentro do grupo de rota contendo o controlador de namespace (root)raiz. 

Se você escolher aninhar ou organizar seus controladores usando namespaces do PHP no diretório `App\Http\Controllers`, simplemene use o nome específico da classe relativa ao namespace raíz `App\Http\Controllers`. Então, se namespace completo do seu controlador forApp\Http\Controllers\Photos\AdminController`, você deve registrar a rota como:

	Route::get('foo', 'Photos\AdminController@method');

#### Nomeando as Rotas de Controladores

Como rotas Closure, você pode especificar nomes nas rotas dos controladores. 

	Route::get('foo', ['uses' => 'FooController@method', 'as' => 'name']);

#### URLs Para Ações do Controller

Para gerar uma URL para uma ação do controlador, user o método helper `action`.

	$url = action('App\Http\Controllers\FooController@method');

Se você desejar gerar uma URL para uma ação do controlador enquanto estiver usando apenas uma porção do nome da classe relativa ao namespace do seu controlador, registre o namespace do controlador raíz com o gerador de URL.

	URL::setRootControllerNamespace('App\Http\Controllers');

	$url = action('FooController@method');

Você pode acessar o nome da ação do controlador que está sendo executada usando o método `currentRouteAction`:

	$action = Route::currentRouteAction();

<a name="controller-middleware"></a>
## Controller Middleware

[Middleware](/docs/{{version}}/middleware) pode ser especificado nas rotas dos controladores assim:

	Route::get('profile', [
		'middleware' => 'auth',
		'uses' => 'UserController@showProfile'
	]);

Além disso, você pode especificar middleware dentro do método construtor do seu controlador: 

	class UserController extends Controller {

		/**
		 * Instantiate a new UserController instance.
		 */
		public function __construct()
		{
			$this->middleware('auth');

			$this->middleware('log', ['only' => ['fooAction', 'barAction']]);

			$this->middleware('subscribed', ['except' => ['fooAction', 'barAction']]);
		}

	}

<a name="implicit-controllers"></a>
## Controladores Implícitos

Laravel permite que você facilmente defina uma rota única para manipular ações em um controlador. Primeiro, defina a rota usando o método `Route::controller`:

	Route::controller('users', 'UserController');

O método `controller` aceita dois argumentos. O primeiro é a URI base que o controlador manipula, enquando o segundo é o nome da classe do controlador. Em seguida, apenas adicione o método em seu controlador, prefixado com o verbo HTTP que o mesmo responde a:

	class UserController extends BaseController {

		public function getIndex()
		{
			//
		}

		public function postProfile()
		{
			//
		}

		public function anyLogin()
		{
			//
		}

	}

Os métodos `index` irão responder para a URI raíz tratada pelo controlador, que neste caso, é `users`.

Se o sua ação do controlador contém múltiplas palavras, você pode acessar a ação usando a sintax "dash" na URI. Por exemplo, a  seguinte ação do nosso controladro `UserController` deve responder a URI `users/admin-profile`: 

	public function getAdminProfile() {}

#### Assinando Nomes as Rotas

Se você quiser "nomear" alguma das rotas no seu controlador, você pode passar um terceiro argumento ao método `controller`:  

	Route::controller('users', 'UserController', [
		'anyLogin' => 'user.login',
	]);

<a name="restful-resource-controllers"></a>
## Controladores RESTful Resource 

Controladores Resource dão facicilidades para se desenvolver controladores RESTful em volta do resouce. Por exemplo, você pode criar um controlador que lida com requisições HTTP sobre "photos"(fotos) armazenadas pela sua aplicação. Usando o comando Artisan `make:controller`, nos podemos rapidamente criar tal controlador. 

	php artisan make:controller PhotoController

Em seguida, nos registramos uma rota resourceful(do tipo resource) para o controlador:

	Route::resource('photo', 'PhotoController');

Esta declaração única de rota cria múltiplas rotas para ligar com uma variedade de ações RESTful no resource de "photo". Da mesma forma, o controlador gerado já irá ter os métodos prontos para cada ação, incluindo notas informando a você as URIs e verbos que elas manipulam.

#### Ações Manipuladas Pelo Controlador Resource

Verb      | Path                  | Action       | Route Name
----------|-----------------------|--------------|---------------------
GET       | /photo                | index        | photo.index
GET       | /photo/create         | create       | photo.create
POST      | /photo                | store        | photo.store
GET       | /photo/{photo}        | show         | photo.show
GET       | /photo/{photo}/edit   | edit         | photo.edit
PUT/PATCH | /photo/{photo}        | update       | photo.update
DELETE    | /photo/{photo}        | destroy      | photo.destroy

#### Customizing Resource Routes

Additionally, you may specify only a subset of actions to handle on the route:

	Route::resource('photo', 'PhotoController',
					['only' => ['index', 'show']]);

	Route::resource('photo', 'PhotoController',
					['except' => ['create', 'store', 'update', 'destroy']]);

By default, all resource controller actions have a route name; however, you can override these names by passing a `names` array with your options:

	Route::resource('photo', 'PhotoController',
					['names' => ['create' => 'photo.build']]);

#### Handling Nested Resource Controllers

To "nest" resource controllers, use "dot" notation in your route declaration:

	Route::resource('photos.comments', 'PhotoCommentController');

This route will register a "nested" resource that may be accessed with URLs like the following: `photos/{photos}/comments/{comments}`.

	class PhotoCommentController extends Controller {

		/**
		 * Show the specified photo comment.
		 *
		 * @param  int  $photoId
		 * @param  int  $commentId
		 * @return Response
		 */
		public function show($photoId, $commentId)
		{
			//
		}

	}

#### Adding Additional Routes To Resource Controllers

If it becomes necessary to add additional routes to a resource controller beyond the default resource routes, you should define those routes before your call to `Route::resource`:

	Route::get('photos/popular', 'PhotoController@method');

	Route::resource('photos', 'PhotoController');

<a name="dependency-injection-and-controllers"></a>
## Dependency Injection & Controllers

#### Constructor Injection

The Laravel [service container](/docs/{{version}}/container) is used to resolve all Laravel controllers. As a result, you are able to type-hint any dependencies your controller may need in its constructor:

	<?php namespace App\Http\Controllers;

	use Illuminate\Routing\Controller;
	use App\Repositories\UserRepository;

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

	}

Of course, you may also type-hint any [Laravel contract](/docs/{{version}}/contracts). If the container can resolve it, you can type-hint it.

#### Method Injection

In addition to constructor injection, you may also type-hint dependencies on your controller's methods. For example, let's type-hint the `Request` instance on one of our methods:

	<?php namespace App\Http\Controllers;

	use Illuminate\Http\Request;
	use Illuminate\Routing\Controller;

	class UserController extends Controller {

		/**
		 * Store a new user.
		 *
		 * @param  Request  $request
		 * @return Response
		 */
		public function store(Request $request)
		{
			$name = $request->input('name');

			//
		}

	}

If your controller method is also expecting input from a route parameter, simply list your route arguments after your other dependencies:

	<?php namespace App\Http\Controllers;

	use Illuminate\Http\Request;
	use Illuminate\Routing\Controller;

	class UserController extends Controller {

		/**
		 * Store a new user.
		 *
		 * @param  Request  $request
		 * @param  int  $id
		 * @return Response
		 */
		public function update(Request $request, $id)
		{
			//
		}

	}

> **Note:** Method injection is fully compatible with [model binding](/docs/{{version}}/routing#route-model-binding). The container will intelligently determine which arguments are model bound and which arguments should be injected.

<a name="route-caching"></a>
## Route Caching

If your application is exclusively using controller routes, you may take advantage of Laravel's route cache. Using the route cache will drastically decrease the amount of time it take to register all of your application's routes. In some cases, your route registration may even be up to 100x faster! To generate a route cache, just execute the `route:cache` Artisan command:

	php artisan route:cache

That's all there is to it! Your cached routes file will now be used instead of your `app/Http/routes.php` file. Remember, if you add any new routes you will need to generate a fresh route cache. Because of this, you may wish to only run the `route:cache` command during your project's deployment.

To remove the cached routes file without generating a new cache, use the `route:clear` command:

	php artisan route:clear
