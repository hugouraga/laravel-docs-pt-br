# Extending The Framework

- [Gerenciadores & Factories(Fábricas)](#managers-and-factories)
- [Cache](#cache)
- [Session](#session)
- [Autenticação](#authentication)
- [Service Container Based Extension](#container-based-extension)

<a name="managers-and-factories"></a>
## Gerenciadores & Factories(Fábricas)

Laravel tem várias classes `gerenciadoras` que gerenciam a criação de drivers baseados-em-componentes. Estes incluem o cache, seção, autenticação, e componentes de queue(fila). A classe gerenciadora é responsável pela criação de uma implementação de driver específico baseado na configuração da aplicação. Por exemplo, a classe `CacheManager` pode criar APC, Mencached, Arquivos, e várias outras implementações de drivers de cache.

Cada um dos gerenciadores incluem um método `extend` que pode ser usado para facilmente a injetar de novas funcionalidades de resolução de drivers no gerenciador. Nos iremos cobrir cada desses gerenciadores abaixo, com exemplos de como injetar um driver de suporte customizado em cada um deles.   

>**Nota:** Pegue um momento para explorar várias classes `gerenciadoras` que vem com o Laravel, como o `CacheManager` e a `SessionManager`. Lendo por essas classes dará um entendimento completo a mais de como Laravel trabalha por baixo dos panos. Todas as classes gerenciadoras extendem da classe base  `Illuminate\Support\Manager, que fornece algumas coisas úteis, funcionalidade comum para cada gerenciador. 

<a name="cache"></a>
## Cache

Para extender da instalações do cache do laravel, nos iremos usar o método `extend` no `CacheManager`, que é usado para vincular um driver resolvedor customizado a um gerenciador, e é comum a todas as classes gerenciadoras. Por exemplo, para o registrar um novo driver cache chamado "mongo", nos podemos fazer o seguinte: 

	Cache::extend('mongo', function($app)
	{
		return Cache::repository(new MongoStore);
	});

O primeiro argumento passado para o método `extend` é o nome do driver. Isto irá corresponder a sua opção `driver` no seu arquivo de configuração `config/cache.php`. O segundo argumento é um Closure que deve retornar uma instância de `Illuminate\Cache\Repository`. A Closure será passada à uma instância de `$app`, que é uma instância de `Illuminate\Foundation\Application` e um container de serviços.

A chamada para `Cache::extend` pode ser realizada no método `boot` do padrão `App\Providers\AppServiceProvider` que vem em instalações nativas de aplicações Larave, ou você pode criar seu próprio fornecedor de serviço para alojar a extenção - apenas não se esqueça de registrar o fornecedor, no array provider no arquivo `config/app.php`.

Para criar nosso driver cache customizado, nos primeiro precisamos implementar o contrato `Illuminate\Contracts\Cache\Store`. Então, nossa implementação de cache MongoDB deve parecer com isto:

	class MongoStore implements Illuminate\Contracts\Cache\Store {

		public function get($key) {}
		public function put($key, $value, $minutes) {}
		public function increment($key, $value = 1) {}
		public function decrement($key, $value = 1) {}
		public function forever($key, $value) {}
		public function forget($key) {}
		public function flush() {}

	}

Nos apenas precisamos implementar cada um destes métodos usando uma conexão MongoDB. Uma vez que nossa implementação é completa, nos podemos finalizar o registro do nosso driver customizado. 

	Cache::extend('mongo', function($app)
	{
		return Cache::repository(new MongoStore);
	});

Se você estiver pensativo sobre aonde alocar o código do seu driver de cache customizado, considere fazer com que o mesmo esteja disponível no Packagist! OU, você pode criar o namespace `Extensions` (Extenções) dentro do seu diretório `app`. No entanto, tenha em mente que o Laravel não tem uma estrutura de aplicação rígida e você é livre para organizar a sua aplicação de acordo com suas preferências. 

<a name="session"></a>
## Sessões 

Extender do Laravel com um driver de sessão customizado é tão fácil quando extender do sistema de cache. Novamente, nos iremos usar o método `extend` para registrar nosso código customizado.

	Session::extend('mongo', function($app)
	{
		// Return implementation of SessionHandlerInterface
	});

### Onde Extender A Sessão

Você deve alocar seu código de extenção da sessão no método `boot` do seu `AppServiceProvider`.

### Escrevendo A extensão Session (Sessão)

Note que o nosso driver se sessão customizado deve implementar a `SessionHandlerInterface`. Esta interface contém apenas alguns simples métodos que nos precisamos implementar. Uma implementação MongoDB encurtada deve parece com algo assim:
	
	class MongoHandler implements SessionHandlerInterface {

		public function open($savePath, $sessionName) {}
		public function close() {}
		public function read($sessionId) {}
		public function write($sessionId, $data) {}
		public function destroy($sessionId) {}
		public function gc($lifetime) {}

	}

Desde que estes métodos não são tão compreensíveis a leitura quanto os da cache `StoreInterface`, vamos rapidamenta explicar o que cada um destes métodos faz:

- O método `open` deve tipicamente ser usado em um arquivo baseado nos sistemas de armazenamento de sessão. Desde que o Laravel entrega o driver da sessão com um `arquivo`, você quase nunca precisará colocar qualquer coisa neste método. Você pode deixar isto em um stub vazio. Isto é pelo simples fato de um design de interface pobre (que nos iremos discutir posteriormente) que o PHP requere que nos façamos a implementação deste método.  
- O método `close`, como o método `open`, pode também ser usualmente desconsiderado. Para a maioria dos drivers, isto não é necessário.
- O método `read` deve retornar a versão string dos dados da sessão associados com o `$sessionId` dado. Não há necessidado para qualquer serealizaççao ou outra codificação quando se esta recuperando ou armazenando dados de sessão no seu driver, já que o Laravel fará a serialização para você.
- O método `write` deve escrever a string `$data` dada associada com o `$sessionId` para algum sistema de armazenamento de persistênte, tais como MongoDB, Dynamo, etc.
- O método `destroy` deve remover os dados associados com o `$sessionId` do armazenamento persistente.
- O método `gc` deve destroir todos os dados da sessão que é mais velho do que `$lifetime`(tempo-de-vida) dado, que é um timestamp UNIX. Para sistemas auto-expiração como Memcached e Redis, este método pode ser deixado vazio. 

Uma vez que a `SessionHandlerInterface` tiver sido implementada, nos estaremos prontos para registrar isto com o gerenciador de Sessão:

	Session::extend('mongo', function($app)
	{
		return new MongoHandler;
	});

Uma vez que o driver de sessão tiver sido registrado, nos podemos usar o driver `mongo` nos seu arquivo de configuração `config/session.php`.

> **Nota:** Lembre-se, se você escrever um manipulador de sessão customizado, compartilhe isso no Packagist!

<a name="authentication"></a>
## Autenticação

Autenticação pode ser extendida do mesma forma que as instalações de cache e sessão. Novamente, no iremos usar o método `extend` que nós nos tornamos familiarizados com: 

	Auth::extend('riak', function($app)
	{
		// Return implementation of Illuminate\Contracts\Auth\UserProvider
	});


As implementações de `UserProvider` são apenas responsáveis por buscar a implementação `Illuminate\Contracts\Auth\Authenticatable` fora de um sistema de armazenamento persistente, como MySQL, Riak, etc. Estas duas interfaces permitem o mecanismo de autenticação do Laravel continuar funcionando independentemente da como os dados do usuário é armazenado ou que tipo de classe é usado para reprensentar isto.

Vamos dar uma olhada no contrato `UserProvider`:

	interface UserProvider {

		public function retrieveById($identifier);
		public function retrieveByToken($identifier, $token);
		public function updateRememberToken(Authenticatable $user, $token);
		public function retrieveByCredentials(array $credentials);
		public function validateCredentials(Authenticatable $user, array $credentials);

	}

The `retrieveById` function typically receives a numeric key representing the user, such as an auto-incrementing ID from a MySQL database. The `Authenticatable` implementation matching the ID should be retrieved and returned by the method.

The `retrieveByToken` function retrieves a user by their unique `$identifier` and "remember me" `$token`, stored in a field `remember_token`. As with the previous method, the `Authenticatable` implementation should be returned.

The `updateRememberToken` method updates the `$user` field `remember_token` with the new `$token`. The new token can be either a fresh token, assigned on successful "remember me" login attempt, or a null when user is logged out.

The `retrieveByCredentials` method receives the array of credentials passed to the `Auth::attempt` method when attempting to sign into an application. The method should then "query" the underlying persistent storage for the user matching those credentials. Typically, this method will run a query with a "where" condition on `$credentials['username']`. The method should then return an implementation of `UserInterface`. **This method should not attempt to do any password validation or authentication.**

The `validateCredentials` method should compare the given `$user` with the `$credentials` to authenticate the user. For example, this method might compare the `$user->getAuthPassword()` string to a `Hash::make` of `$credentials['password']`. This method should only validate the user's credentials and return boolean.

Now that we have explored each of the methods on the `UserProvider`, let's take a look at the `Authenticatable`. Remember, the provider should return implementations of this interface from the `retrieveById` and `retrieveByCredentials` methods:

	interface Authenticatable {

		public function getAuthIdentifier();
		public function getAuthPassword();
		public function getRememberToken();
		public function setRememberToken($value);
		public function getRememberTokenName();

	}

This interface is simple. The `getAuthIdentifier` method should return the "primary key" of the user. In a MySQL back-end, again, this would be the auto-incrementing primary key. The `getAuthPassword` should return the user's hashed password. This interface allows the authentication system to work with any User class, regardless of what ORM or storage abstraction layer you are using. By default, Laravel includes a `User` class in the `app` directory which implements this interface, so you may consult this class for an implementation example.

Finally, once we have implemented the `UserProvider`, we are ready to register our extension with the `Auth` facade:

	Auth::extend('riak', function($app)
	{
		return new RiakUserProvider($app['riak.connection']);
	});

After you have registered the driver with the `extend` method, you switch to the new driver in your `config/auth.php` configuration file.

<a name="container-based-extension"></a>
## Service Container Based Extension

Almost every service provider included with the Laravel framework binds objects into the service container. You can find a list of your application's service providers in the `config/app.php` configuration file. As you have time, you should skim through each of these provider's source code. By doing so, you will gain a much better understanding of what each provider adds to the framework, as well as what keys are used to bind various services into the service container.

For example, the `HashServiceProvider` binds a `hash` key into the service container, which resolves into a `Illuminate\Hashing\BcryptHasher` instance. You can easily extend and override this class within your own application by overriding this binding. For example:

	<?php namespace App\Providers;

	class SnappyHashProvider extends \Illuminate\Hashing\HashServiceProvider {

		public function boot()
		{
			parent::boot();
					
			$this->app->bindShared('hash', function()
			{
				return new \Snappy\Hashing\ScryptHasher;
			});
		}

	}

Note that this class extends the `HashServiceProvider`, not the default `ServiceProvider` base class. Once you have extended the service provider, swap out the `HashServiceProvider` in your `config/app.php` configuration file with the name of your extended provider.

This is the general method of extending any core class that is bound in the container. Essentially every core class is bound in the container in this fashion, and can be overridden. Again, reading through the included framework service providers will familiarize you with where various classes are bound into the container, and what keys they are bound by. This is a great way to learn more about how Laravel is put together.
