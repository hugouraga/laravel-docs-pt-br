# Envoy Task Runner

- [Introdução](#introduction)
- [Instalação](#envoy-installation)
- [Executando Tarefas](#envoy-running-tasks)
- [Múltiplos Servidores](#envoy-multiple-servers)
- [Parallel Execution](#envoy-parallel-execution)
- [Macros de Tarefas](#envoy-task-macros)
- [Notificações](#envoy-notifications)
- [Atualizando o Envoy](#envoy-updating-envoy)

<a name="introduction"></a>
## Introdução

[Laravel Envoy](https://github.com/laravel/envoy) fornece uma limpa, sintax minimalista por definição tarefas comuns que são executadas nos seus servidores remotos. Usando o estilo de sintax do Blade, você pode facilmente configurar tarefas para deploy, comandos Artisan, e mais.

> **Nota:** Envoy requere que a versão do PHP seja 5.4 ou superior, e apenas funciona nos sistemas operacionais Mac/Linux.

<a name="envoy-installation"></a>
## Instalação

Primeiro, instale Envoy usando o comando `global` do Composer:

	composer global require "laravel/envoy=~1.0"

Assegure-se de alocar o diretório `~/.composer/vendor/bin` na sua variável de ambiente PATH para que então o comando `envoy` seja encontrado quando você executar o comando no terminal.

Em seguida, crie um arquivo blade `Envoy.blade.php` no diretório raíz do seu projeto. Aqui vai um exemplo de como vocẽ pode começar:

	@servers(['web' => '192.168.1.1'])

	@task('foo', ['on' => 'web'])
		ls -la
	@endtask

Como você pode ver, o array de `@servers` é definido no topo do arquivo. Você pode referenciar estes servidores na opção `on` das suas declarações de tarefas. Dentro das suas declarações `@task` você deve alocar o código Bash que irá executar no seu servidor quando a tarefa é executada.

O comando `init` pode ser usado para criar facilmente um arquivo Stub do Envoy:

	envoy init user@192.168.1.1

<a name="envoy-running-tasks"></a>
## Executando Tarefas

To run a task, use the `run` command of your Envoy installation:

	envoy run foo

If needed, you may pass variables into the Envoy file using command line switches:

	envoy run deploy --branch=master

You may use the options via the Blade syntax you are used to:

	@servers(['web' => '192.168.1.1'])

	@task('deploy', ['on' => 'web'])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

#### Bootstrapping

You may use the ```@setup``` directive to declare variables and do general PHP work inside the Envoy file:

	@setup
		$now = new DateTime();

		$environment = isset($env) ? $env : "testing";
	@endsetup

You may also use ```@include``` to include any PHP files:

	@include('vendor/autoload.php');

#### Confirming Tasks Before Running

If you would like to be prompted for confirmation before running a given task on your servers, you may use the `confirm` directive:

	@task('deploy', ['on' => 'web', 'confirm' => true])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

<a name="envoy-multiple-servers"></a>
## Múltiplos Servidores

You may easily run a task across multiple servers. Simply list the servers in the task declaration:

	@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

	@task('deploy', ['on' => ['web-1', 'web-2']])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

By default, the task will be executed on each server serially. Meaning, the task will finish running on the first server before proceeding to execute on the next server.

<a name="envoy-parallel-execution"></a>
## Execução Paralela

If you would like to run a task across multiple servers in parallel, simply add the `parallel` option to your task declaration:

	@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

	@task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

<a name="envoy-task-macros"></a>
## Macros de Tarefas

Macros allow you to define a set of tasks to be run in sequence using a single command. For instance:

	@servers(['web' => '192.168.1.1'])

	@macro('deploy')
		foo
		bar
	@endmacro

	@task('foo')
		echo "HELLO"
	@endtask

	@task('bar')
		echo "WORLD"
	@endtask

The `deploy` macro can now be run via a single, simple command:

	envoy run deploy

<a name="envoy-notifications"></a>
<a name="envoy-hipchat-notifications"></a>
## Notificações

#### HipChat

After running a task, you may send a notification to your team's HipChat room using the simple `@hipchat` directive:

	@servers(['web' => '192.168.1.1'])

	@task('foo', ['on' => 'web'])
		ls -la
	@endtask

	@after
		@hipchat('token', 'room', 'Envoy')
	@endafter

You can also specify a custom message to the hipchat room. Any variables declared in ```@setup``` or included with ```@include``` will be available for use in the message:

	@after
		@hipchat('token', 'room', 'Envoy', "$task ran on [$environment]")
	@endafter

This is an amazingly simple way to keep your team notified of the tasks being run on the server.

#### Slack

The following syntax may be used to send a notification to [Slack](https://slack.com):

	@after
		@slack('hook', 'channel', 'message')
	@endafter

You may retrieve your webhook URL by creating an `Incoming WebHooks` integration on Slack's website. The `hook` argument should be the entire webhook URL provided by the Incoming Webhooks Slack Integration. For example:

	https://hooks.slack.com/services/ZZZZZZZZZ/YYYYYYYYY/XXXXXXXXXXXXXXX

You may provide one of the following for the channel argument:

- To send the notification to a channel: `#channel`
- To send the notification to a user: `@user`

If no `channel` argument is provided the default channel will be used.

> Note: Slack notifications will only be sent if all tasks complete successfully.

<a name="envoy-updating-envoy"></a>
## Atualizando o Envoy

To update Envoy, simply use Composer:

	composer global update

