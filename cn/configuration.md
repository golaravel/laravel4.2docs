# 配置

- [引言](#introduction)
- [环境配置](#environment-configuration)
- [供应者配置](#provider-configuration)
- [敏感信息保护配置](#protecting-sensitive-configuration)
- [维护模式](#maintenance-mode)

<a name="introduction"></a>
## 引言

Laravel 框架的所有配置文件都存储于 `app/config` 目录。每个文件中的每个选项都做了详细的记录，以便随时翻阅文件，熟悉提供给你的选项。

有时你需要在程序执行阶段访问配置的值。你可以使用 `Config` 类：

#### 访问一个配置的值

	Config::get('app.timezone');

你也可以指定一个默认值，如果配置选项不存在它将被返回：

	$timezone = Config::get('app.timezone', 'UTC');

#### 设置一个配置的值

注意“点”语法风格可以用于访问不同文件里的值，你也可以在程序执行阶段设置配置的值：

	Config::set('database.default', 'sqlite');

在程序执行阶段设置的配置的值仅作用于当前请求，并不会延续到后续请求中。

<a name="environment-configuration"></a>
## 环境配置

基于应用程序的运行环境拥有不同配置值通常是非常有益的。例如，相对于生产服务器你希望在你本地开发设备上使用一个不同的缓存驱动。使用基于环境的配置可以很容易的做到这点。

在 `config` 目录里简单的创建一个与你的环境同名的文件夹，比如 `local`。接着，创建你希望在这个环境中指定选项被覆盖的配置文件。例如，在 local 环境下覆盖缓存驱动，你将要在 `app/config/local` 里创建一个 `cache.php` 文件并包含以下内容：

	<?php

	return array(

		'driver' => 'file',

	);

> **注意：** 不要使用 'testing' 作为环境名称。这是为单元测试预留的。

注意，你不必指定基础配置文件中的 _每一个_ 选项，仅需指定你希望覆盖的选项。环境配置文件将 "叠加" 在基础配置文件之上。

接着，我们需要告知框架如何判定自己运行于哪个环境中。默认的环境始终是 `production`。然而，你可以在安装程序的根目录下 `bootstrap/start.php` 文件中设置其他环境。在这个文件中你将找到一个 `$app->detectEnvironment` 的调用。向这个方法传入的数组用于确定当前环境。必要时你可以增加其他环境和机器名。

    <?php

    $env = $app->detectEnvironment(array(

        'local' => array('your-machine-name'),

    ));

在这个例子中，'local' 是环境名称而 'your-machine-name' 是你本地服务器的主机名。在 Linux 和 Mac 上，你可以使用 `hostname` 终端命令来确定你的主机名。

如果你需要更灵活的环境检测，你可以通过向 `detectEnvironment` 方法传入一个 `匿名函数`，这允许你按照自己的方式执行环境检测：

	$env = $app->detectEnvironment(function()
	{
		return $_SERVER['MY_LARAVEL_ENV'];
	});

#### 访问当前的应用环境

你可以通过 `environment` 方法访问当前的应用环境：

	$environment = App::environment();

你也可以通过向 `environment` 方法传递参数来检测环境是否与给定的值匹配：

	if (App::environment('local'))
	{
		// 当前为 local 运行环境
	}

	if (App::environment('local', 'staging'))
	{
		// 当前为 local 或 staging 运行环境
	}

<a name="provider-configuration"></a>
### 供应者配置

当使用环境配置，你可能想要 "追加" 环境 [服务供应者](/docs/ioc#service-providers) 到你的基础 `app` 配置文件中。然而，如果你尝试这么做，你需要注意这个环境 `app` 供应者将会完全覆盖你的基础 `app` 配置文件中的值。要强制追加供应者，需要在你的环境 `app` 配置文件中使用 `append_config` 辅助函数：

	'providers' => append_config(array(
		'LocalOnlyServiceProvider',
	))

<a name="protecting-sensitive-configuration"></a>
## 敏感信息保护配置

For "real" applications, it is advisable to keep all of your sensitive 配置 out of your 配置 files. Things such as database passwords, Stripe API keys, and encryption keys should be kept out of your 配置 files whenever possible. So, where should we place them? Thankfully, Laravel provides a very simple solution to protecting these types of 配置 items using "dot" files.

First, [configure your application](/docs/配置#环境-配置) to recognize your machine as being in the `local` 环境. Next, create a `.env.local.php` file within the root of your project, which is usually the same directory that contains your `composer.json` file. The `.env.local.php` should return an array of key-value pairs, much like a typical Laravel 配置 file:

	<?php

	return array(

		'TEST_STRIPE_KEY' => 'super-secret-sauce',

	);

All of the key-value pairs returned by this file will automatically be available via the `$_ENV` and `$_SERVER` PHP "superglobals". You may now reference these globals from within your 配置 files:

	'key' => $_ENV['TEST_STRIPE_KEY']

Be sure to add the `.env.local.php` file to your `.gitignore` file. This will allow other developers on your team to create their own local 环境 配置, as well as hide your sensitive 配置 items from source control.

Now, on your production server, create a `.env.php` file in your project root that contains the corresponding values for your production 环境. Like the `.env.local.php` file, the production `.env.php` file should never be included in source control.

> **Note:** You may create a file for each 环境 supported by your application. For example, the `development` 环境 will load the `.env.development.php` file if it exists.

<a name="maintenance-mode"></a>
## 维护模式

When your application is in 维护模式, a custom view will be displayed for all routes into your application. This makes it easy to "disable" your application while it is updating or when you are performing maintenance. A call to the `App::down` method is already present in your `app/start/global.php` file. The response from this method will be sent to users when your application is in 维护模式.

To enable 维护模式, simply execute the `down` Artisan command:

	php artisan down

To disable 维护模式, use the `up` command:

	php artisan up

To show a custom view when your application is in 维护模式, you may add something like the following to your application's `app/start/global.php` file:

	App::down(function()
	{
		return Response::view('maintenance', array(), 503);
	});

If the Closure passed to the `down` method returns `NULL`, 维护模式 will be ignored for that request.

### 维护模式 & Queues

While your application is in 维护模式, no [queue jobs](/docs/queues) will be handled. The jobs will continue to be handled as normal once the application is out of 维护模式.
