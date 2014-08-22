# SSH(Secure Shell)

- [配置](#configuration)
- [基本用法](#basic-usage)
- [任务](#tasks)
- [SFTP 下载](#sftp-downloads)
- [SFTP上传](#sftp-uploads)
- [显示远程日志的末尾几行](#tailing-remote-logs)
- [Envoy Task Runner](#envoy-task-runner)

<a name="configuration"></a>
## 配置

Laravel 为 SSH 登录远程服务器并运行命令提供了一种简单的方式，允许你轻松创建运行在远程服务器上的 Artisan 任务。`SSH` 门面类(facade)提供了连接远程服务器和运行命令的访问入口。

配置文件位于 `app/config/remote.php`，该文件包含了配置远程连接所需的所有选项。`connections` 数组包含了一个以名称作为键的服务器列表。只需将 `connections` 数组中的凭证信息填好，就可以开始运行远程任务了。注意 `SSH` 可以使用密码或 SSH 密钥进行身份验证。

> **Note:** Need to easily run a variety of tasks on your remote server? Check out the [Envoy task runner](#envoy-task-runner)!

<a name="basic-usage"></a>
## 基本用法

#### 在默认服务器上运行命令

使用 `SSH::run` 方法在 `default` 远程服务器连接上运行命令：

	SSH::run(array(
		'cd /var/www',
		'git pull origin master',
	));

#### 在指定连接上运行命令

或者，你可以使用 `into` 方法在指定的服务器连接上运行命令：

	SSH::into('staging')->run(array(
		'cd /var/www',
		'git pull origin master',
	));

#### 捕获命令输出结果

你可以通过向 `run` 方法传递一个闭包来捕获远程命令的“实时”输出结果：

	SSH::run($commands, function($line)
	{
		echo $line.PHP_EOL;
	});

## 任务
<a name="tasks"></a>

如果需要定义一组经常放在一起运行的命令，你可以使用 `define` 方法来定义一个任务( `task` )：

	SSH::into('staging')->define('deploy', array(
		'cd /var/www',
		'git pull origin master',
		'php artisan migrate',
	));

任务一旦创建，就可以使用 `task` 来执行：

	SSH::into('staging')->task('deploy', function($line)
	{
		echo $line.PHP_EOL;
	});

<a name="sftp-downloads"></a>
## SFTP Downloads

The `SSH` class includes a simple way to download files using the `get` and `getString` methods:

	SSH::into('staging')->get($remotePath, $localPath);

	$contents = SSH::into('staging')->getString($remotePath);

<a name="sftp-uploads"></a>
## SFTP 上传

`SSH` 类的 `put` 和 `putString` 方法提供了一种简单的上传方式，用于将文件或字符串上传到服务器：

	SSH::into('staging')->put($localFile, $remotePath);

	SSH::into('staging')->putString($remotePath, 'Foo');

<a name="tailing-remote-logs"></a>
## 显示远程日志的末尾几行

Laravel 提供了一个有用的命令用来查看任何远程连接服务器上的 `laravel.log` 文件的末尾几行内容。只需使用 Artisan 命令 `tail` 并指定你想要查看日志的远程连接的名称即可：

	php artisan tail staging

	php artisan tail staging --path=/path/to/log.file

<a name="envoy-task-runner"></a>
## Envoy Task Runner

- [Installation](#envoy-installation)
- [Running Tasks](#envoy-running-tasks)
- [Multiple Servers](#envoy-multiple-servers)
- [Parallel Execution](#envoy-parallel-execution)
- [Task Macros](#envoy-task-macros)
- [Notifications](#envoy-notifications)
- [Updating Envoy](#envoy-updating-envoy)

Laravel Envoy provides a clean, minimal syntax for defining common tasks you run on your remote servers. Using a [Blade](/docs/templates#blade-templating) style syntax, you can easily setup tasks for deployment, Artisan commands, and more.

> **Note:** Envoy requires PHP version 5.4 or greater, and only runs on Mac / Linux operating systems.

<a name="envoy-installation"></a>
### Installation

First, install Envoy using the Composer `global` command:

	composer global require "laravel/envoy=~1.0"

Make sure to place the `~/.composer/vendor/bin` directory in your PATH so the `envoy` executable is found when you run the `envoy` command in your terminal.

Next, create an `Envoy.blade.php` file in the root of your project. Here's an example to get you started:

	@servers(['web' => '192.168.1.1'])

	@task('foo', ['on' => 'web'])
		ls -la
	@endtask

As you can see, an array of `@servers` is defined at the top of the file. You can reference these servers in the `on` option of your task declarations. Within your `@task` declarations you should place the Bash code that will be run on your server when the task is executed.

The `init` command may be used to easily create a stub Envoy file:

	envoy init user@192.168.1.1

<a name="envoy-running-tasks"></a>
### Running Tasks

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

<a name="envoy-multiple-servers"></a>
### Multiple Servers

You may easily run a task across multiple servers. Simply list the servers in the task declaration:

	@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

	@task('deploy', ['on' => ['web-1', 'web-2']])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

By default, the task will be executed on each server serially. Meaning, the task will finish running on the first server before proceeding to execute on the next server.

<a name="envoy-parallel-execution"></a>
### Parallel Execution

If you would like to run a task across multiple servers in parallel, simply add the `parallel` option to your task declaration:

	@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

	@task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

<a name="envoy-task-macros"></a>
### Task Macros

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
### Notifications

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
		@slack('team', 'token', 'channel')
	@endafter

<a name="envoy-updating-envoy"></a>
### Updating Envoy

To update Envoy, simply run the `self-update` command:

	envoy self-update

If your Envoy installation is in `/usr/local/bin`, you may need to use `sudo`:

	sudo envoy self-update
