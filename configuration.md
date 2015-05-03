# Configuration

- [Introdução](#introduction)
- [Depois da Instalação](#after-installation)
- [Acessando Valores de Configuração](#accessing-configuration-values)
- [Configuração de Ambiente](#environment-configuration)
- [Configuração de Caching](#configuration-caching)
- [Modo de Manutenção](#maintenance-mode)
- [URLs Amigáveis](#pretty-urls)

<a name="introduction"></a>
## Introdução

Todos os arquivos de configuração para o framework Laravel são armazenados no diretório `config`. Cada opção é documentada, então sinta-se livre para dar uma olhada nos arquivos
e se famialirizar com as opções disponíves pra você.

<a name="after-installation"></a>
## Depois Da Instalação

### Nomeando sua aplicação

Depois de instalar Laravel, você pode querer nomear a sua aplicação. Por padrão, o diretório `app` tem o namespace `App`, e é carregado automaticamente pelo Composer usando a [PSR-4 autoloading standard](http://www.php-fig.org/psr/psr-4/). Contudo, você pode mudar o namespace para o mesmo da sua aplicação, que você pode facilmente fazer por meio do comando Artisan `app:name`.

Por exemplo, se sua aplicação é nomeada "Horsefly", você pode executar o seguinte comando a partir do diretório root da sua instalação:

	php artisan app:name Horsefly

Renomear sua aplicação é totalmente opcional, e você é livre para manter o namespace `App` se você desejar.

### Outras Configurações

Laravel precisa de poucas configurações de fora da caixa. Você é livre para começar a desenvolver! No entanto, você pode desejar revisar o arquivo `config/app.php` e sua documentação. Nela, contém várias opções como as de `timezone` (fuso-horário) e `locale` (localização) que você pode desejar mudar de acordo com a sua localização.


Uma vez que o Laravel é instalado, você deve tambeḿ [configure your local environment](/docs/{{version}}/configuration#environment-configuration).

> **Nota:** Você nunca deve ter a configuração `app.debug` definida como "true" no ambiente de produção da sua aplicação.

<a name="permissions"></a>
### Permissões

Laravel pode requerer um conjunto de permissões para ser configurado: pastas dentro do diretório `storage` requerem permissões de acesso a escrita no servidor. 

<a name="accessing-configuration-values"></a>
## Acessando Valores de Configuração

Você pode facilmente acessar os valores de configurações usando a fachada `Config`:

	$value = Config::get('app.timezone');

	Config::set('app.timezone', 'America/Chicago');

Você pode também usar o função helper `config`:

	$value = config('app.timezone');

<a name="environment-configuration"></a>
## Configuração de Ambiente

Na maioria das vezes, é util ter diferentes valores de configurações baseados nos ambientes em que a aplicação está sendo executada. Por examplo, você pode desejar usar drivers de cache diferentes localmente do que você usa no servidor de produção. Isto é fácil, usando o ambiente baseado na configuração.  

Para isso ser fácil, Laravel utiliza a biblioteca PHP feita por Vance Lucar [DotEnv](https://github.com/vlucas/phpdotenv). Em uma instalação pura de laravel, o diretório rais da sua aplicação irá conter o arquivo `.env.example`. Se você instalar o Laravel via Composer, este arquvivo irá automaticamente ser renomeado por `.env`. Caso contrário, você deve renomear o arquvio manualmente. 

Todas as variáveis listadas neste arquvio serão carregadas na superglobal PHP `$_ENV` quando sua aplicação receber a requisição. Você pode usar o helper `env` para recuperar os valores das variáveis. Na verdade, se você revisar os arquivos de configuração do Laravel, você irá notar que várias das opções já usam este Helper!

Sinta-se à vontade para modificar as suas variáveis de ambiente como for necessário para o seu servidor local, como também para seu ambiente de produção. No entanto, seu arquivo `.env` não deve ser commitado para sistema de controle de versionamento, desde que cada desenvolvedor / servidor que está usando sua aplicação possa requerer diferentes configurações de ambiente. 

Se você estiver desenvolvendo com um time, você pode querer continuar incluindo o arquivo `.env.example` na sua aplicação. Ao colocar valores place-holder no exemplo de arquivo de configuração, outros desenvolvedor no seu time podem claramente ver que o as variáveis de ambeinte são necessárias para a execução da sua aplicação. 

#### Accessing The Current Application Environment

You may access the current application environment via the `environment` method on the `Application` instance:

	$environment = $app->environment();

You may also pass arguments to the `environment` method to check if the environment matches a given value:

	if ($app->environment('local'))
	{
		// The environment is local
	}

	if ($app->environment('local', 'staging'))
	{
		// The environment is either local OR staging...
	}

To obtain an instance of the application, resolve the `Illuminate\Contracts\Foundation\Application` contract via the [service container](/docs/{{version}}/container). Of course, if you are within a [service provider](/docs/{{version}}/providers), the application instance is available via the `$this->app` instance variable.

An application instance may also be accessed via the `app` helper or the `App` facade:

	$environment = app()->environment();

	$environment = App::environment();

<a name="configuration-caching"></a>
## Configuration Caching

To give your application a little speed boost, you may cache all of your configuration files into a single file using the `config:cache` Artisan command. This will combine all of the configuration options for your application into a single file which can be loaded quickly by the framework.

You should typically run the `config:cache` command as part of your deployment routine.

<a name="maintenance-mode"></a>
## Maintenance Mode

When your application is in maintenance mode, a custom view will be displayed for all requests into your application. This makes it easy to "disable" your application while it is updating or when you are performing maintenance. A maintenance mode check is included in the default middleware stack for your application. If the application is in maintenance mode, an `HttpException` will be thrown with a status code of 503.

To enable maintenance mode, simply execute the `down` Artisan command:

	php artisan down

To disable maintenance mode, use the `up` command:

	php artisan up

### Maintenance Mode Response Template

The default template for maintenance mode responses is located in `resources/views/errors/503.blade.php`.

### Maintenance Mode & Queues

While your application is in maintenance mode, no [queued jobs](/docs/{{version}}/queues) will be handled. The jobs will continue to be handled as normal once the application is out of maintenance mode.

<a name="pretty-urls"></a>
## Pretty URLs

### Apache

The framework ships with a `public/.htaccess` file that is used to allow URLs without `index.php`. If you use Apache to serve your Laravel application, be sure to enable the `mod_rewrite` module.

If the `.htaccess` file that ships with Laravel does not work with your Apache installation, try this one:

	Options +FollowSymLinks
	RewriteEngine On

	RewriteCond %{REQUEST_FILENAME} !-d
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteRule ^ index.php [L]

If your web host doesn't allow the `FollowSymlinks` option, try replacing it with `Options +SymLinksIfOwnerMatch`.

### Nginx

On Nginx, the following directive in your site configuration will allow "pretty" URLs:

	location / {
		try_files $uri $uri/ /index.php?$query_string;
	}

Of course, when using [Homestead](/docs/{{version}}/homestead), pretty URLs will be configured automatically.
