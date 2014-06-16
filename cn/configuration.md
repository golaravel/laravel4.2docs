# 配置

- [引言](#introduction)
- [环境 配置](#environment-configuration)
- [供应者 配置](#provider-configuration)
- [Protecting Sensitive 配置](#protecting-sensitive-configuration)
- [维护模式](#maintenance-mode)

<a name="introduction"></a>
## 引言

Laravel 框架的所有配置文件都存储于 `app/config` 目录。每个文件中的每个选项Each option in every file is documented, so feel free to look through the files and get familiar with the options available to you.

Sometimes you may need to access 配置 values at run-time. You may do so using the `Config` class:

#### Accessing A 配置 Value

	Config::get('app.timezone');

You may also specify a default value to return if the 配置 option does not exist:

	$timezone = Config::get('app.timezone', 'UTC');

#### Setting A 配置 Value

Notice that "dot" style syntax may be used to access values in the various files. You may also set 配置 values at run-time:

	Config::set('database.default', 'sqlite');

配置 values that are set at run-time are only set for the current request, and will not be carried over to subsequent requests.

<a name="environment-configuration"></a>
## 环境配置

It is often helpful to have different 配置 values based on the 环境 the application is running in. For example, you may wish to use a different cache driver on your local development machine than on the production server. It is easy to accomplish this using 环境 based 配置.

Simply create a folder within the `config` directory that matches your 环境 name, such as `local`. Next, create the 配置 files you wish to override and specify the options for that 环境. For example, to override the cache driver for the local 环境, you would create a `cache.php` file in `app/config/local` with the following content:

	<?php

	return array(

		'driver' => 'file',

	);

> **Note:** Do not use 'testing' as an 环境 name. This is reserved for unit testing.

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
