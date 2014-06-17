# 配置

- [引言](#introduction)
- [环境配置](#environment-configuration)
- [供应者配置](#provider-configuration)
- [Protecting Sensitive 配置](#protecting-sensitive-configuration)
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

Notice that you do not have to specify _every_ option that is in the base 配置 file, but only the options you wish to override. The 环境 配置 files will "cascade" over the base files.

Next, we need to instruct the framework how to determine which 环境 it is running in. The default 环境 is always `production`. However, you may setup other 环境s within the `bootstrap/start.php` file at the root of your installation. In this file you will find an `$app->detect环境` call. The array passed to this method is used to determine the current 环境. You may add other 环境s and machine names to the array as needed.

    <?php

    $env = $app->detect环境(array(

        'local' => array('your-machine-name'),

    ));

In this example, 'local' is the name of the 环境 and 'your-machine-name' is the hostname of your server. On Linux and Mac, you may determine your hostname using the `hostname` terminal command.

If you need more flexible 环境 detection, you may pass a `Closure` to the `detect环境` method, allowing you to implement 环境 detection however you wish:

	$env = $app->detect环境(function()
	{
		return $_SERVER['MY_LARAVEL_ENV'];
	});

#### Accessing The Current Application 环境

You may access the current application 环境 via the `环境` method:

	$环境 = App::环境();

You may also pass arguments to the `环境` method to check if the 环境 matches a given value:

	if (App::环境('local'))
	{
		// The 环境 is local
	}

	if (App::环境('local', 'staging'))
	{
		// The 环境 is either local OR staging...
	}

<a name="provider-configuration"></a>
### 供应者 配置

When using 环境 配置, you may want to "append" 环境 [service 供应者s](/docs/ioc#service-供应者s) to your primary `app` 配置 file. However, if you try this, you will notice the 环境 `app` 供应者s are overriding the 供应者s in your primary `app` 配置 file. To force the 供应者s to be appended, use the `append_config` helper method in your 环境 `app` 配置 file:

	'供应者s' => append_config(array(
		'LocalOnlyService供应者',
	))

<a name="protecting-sensitive-configuration"></a>
## Protecting Sensitive 配置

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
