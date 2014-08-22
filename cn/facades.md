# Facades

- [简介](#introduction)
- [说明](#explanation)
- [实际用例](#practical-usage)
- [创建 Facades](#creating-facades)
- [模拟 Facades](#mocking-facades)
- [Facade Class Reference](#facade-class-reference)

<a name="introduction"></a>
## 简介

Facades 提供了一个“静态”接口到 [IoC 容器](/docs/ioc) 类 。Laravel 含有很多 facades，你可能不知道你在某些地方已经使用过它们了。Laravel "facades" serve as "static proxies" to underlying classes in the IoC container, providing the benefit of a terse, expressive syntax while maintaining more testability and flexibility than traditional static methods.

有时候， 你可能想为你的应用程序或包创建自己的 facades， 所以，让我们来讨论一下如何开发和使用这些类。


> **注意** 在深入 facades 之前，我们强烈建议你多了解一下 Laravel [Ioc 容器](/docs/ioc)。


<a name="explanation"></a>
## 说明

在 Laravel 应用程序中， facade 是提供从容器中访问对象方法的类。`Facade` 类实现了该机制。 

你的 facade 类只需要实现一个方法： `getFacadeAccesor` 。 `getFacadeAccessor` 方法的工作是定义如何从容器中取得对象。 `Facades` 基类构建了 `__callStatic()` 魔术方法来从 facade 延迟访问取得对象。

So, when you make a facade call like `Cache::get`, Laravel resolves the Cache manager class out of the IoC container and calls the `get` method on the class. In technical terms, Laravel Facades are a convenient syntax for using the Laravel IoC container as a service locator.

<a name="practical-usage"></a>
## 实际用例

在以下示例中，执行 Laravel 缓存系统， 查看该代码，我们可能会假定 `get` 静态方法是执行在 `Cache` 类。

In the example below, a call is made to the Laravel cache system. By glancing at this code, one might assume that the static method `get` is being called on the `Cache` class.

	$value = Cache::get('key');

然后，如果我们查看 `Illuminate\Support\Facades\Cache` 类， 你会发现该类没有任何 `get` 静态方法：

	class Cache extends Facade {

		/**
		 * Get the registered name of the component.
		 *
		 * @return string
		 */
		protected static function getFacadeAccessor() { return 'cache'; }

	}

Cache 类继承基本 `Facade` 类，并且定义了个 `getFacadeAccessor()` 方法。注意，该方法的工作是返回 IoC 绑定的名字。

当用户引用任何在 `Cache` facade 静态方法， Laravel 从 IoC 容器绑定中取得 `cache`，并且执行请求该对象方法（在该例子中为`get`）。

所以，我们 `Cache::get` 执行可以重写为：

	$value = $app->make('cache')->get('key');

<a name="creating-facades"></a>
## 创建 Facades

要为自己的应用程序或者包创建一个 facade 是非常简单的。你需要做三件事情：

- 一个 IoC 绑定。
- 一个 facade 类。
- 一个 facade 别名配置。

让我们来看个例子。这里，我们定义一个 `PaymentGateway\Payment` 类。

	namespace PaymentGateway;

	class Payment {

		public function process()
		{
			//
		}

	}

This class might live in your `app/models` directory, or any other directory that Composer knows how to auto-load.
	
我们需要能够在 IoC 容器中取得该类。所以，让我们增加一个绑定：

	App::bind('payment', function()
	{
		return new \PaymentGateway\Payment;
	});

最好注册该绑定的位置是创建一个新的名为 `PaymentServiceProvider` [服务提供器](/docs/ioc#service-providers)，并且将该绑定加入到 `register` 方法。接下来你就可以配置 Laravel `app/config/app.php` 配置文件来加载该服务提供器。

接下来，我们就可以创建我们的 facade 类：

	use Illuminate\Support\Facades\Facade;

	class Payment extends Facade {

		protected static function getFacadeAccessor() { return 'payment'; }

	}

最后，如果你想，我们可以为我们 facade 设置别名到 `app/config/app.php` 配置文件里的 `aliases` 数组。现在，我们能够调用 `Payment` 类实例的 `process` 方法。
	Payment::process();

<a name="mocking-facades"></a>
## 模拟 Facades

单元测试是 facades 工作的重要体现。事实上，可测试性是 facedes 存在的主要原因。要了解更多信息，查看文档[模拟 facades](/docs/testing#mocking-facades)部分。

<a name="facade-class-reference"></a>
## Facade Class Reference

Below you will find every facade and its underlying class. This is a useful tool for quickly digging into the API documentation for a given facade root. The [IoC binding](/docs/ioc) key is also included where applicable.

Facade  |  Class  |  IoC Binding
------------- | ------------- | -------------
App  |  [Illuminate\Foundation\Application](http://laravel.com/api/4.1/Illuminate/Foundation/Application.html)  | `app`
Artisan  |  [Illuminate\Console\Application](http://laravel.com/api/4.1/Illuminate/Console/Application.html)  |  `artisan`
Auth  |  [Illuminate\Auth\AuthManager](http://laravel.com/api/4.1/Illuminate/Auth/AuthManager.html)  |  `auth`
Auth (Instance)  |  [Illuminate\Auth\Guard](http://laravel.com/api/4.1/Illuminate/Auth/Guard.html)  |
Blade  |  [Illuminate\View\Compilers\BladeCompiler](http://laravel.com/api/4.1/Illuminate/View/Compilers/BladeCompiler.html)  |  `blade.compiler`
Cache  |  [Illuminate\Cache\Repository](http://laravel.com/api/4.1/Illuminate/Cache/Repository.html)  |  `cache`
Config  |  [Illuminate\Config\Repository](http://laravel.com/api/4.1/Illuminate/Config/Repository.html)  |  `config`
Cookie  |  [Illuminate\Cookie\CookieJar](http://laravel.com/api/4.1/Illuminate/Cookie/CookieJar.html)  |  `cookie`
Crypt  |  [Illuminate\Encryption\Encrypter](http://laravel.com/api/4.1/Illuminate/Encryption/Encrypter.html)  |  `encrypter`
DB  |  [Illuminate\Database\DatabaseManager](http://laravel.com/api/4.1/Illuminate/Database/DatabaseManager.html)  |  `db`
DB (Instance)  |  [Illuminate\Database\Connection](http://laravel.com/api/4.1/Illuminate/Database/Connection.html)  |
Event  |  [Illuminate\Events\Dispatcher](http://laravel.com/api/4.1/Illuminate/Events/Dispatcher.html)  |  `events`
File  |  [Illuminate\Filesystem\Filesystem](http://laravel.com/api/4.1/Illuminate/Filesystem/Filesystem.html)  |  `files`
Form  |  [Illuminate\Html\FormBuilder](http://laravel.com/api/4.1/Illuminate/Html/FormBuilder.html)  |  `form`
Hash  |  [Illuminate\Hashing\HasherInterface](http://laravel.com/api/4.1/Illuminate/Hashing/HasherInterface.html)  |  `hash`
HTML  |  [Illuminate\Html\HtmlBuilder](http://laravel.com/api/4.1/Illuminate/Html/HtmlBuilder.html)  |  `html`
Input  |  [Illuminate\Http\Request](http://laravel.com/api/4.1/Illuminate/Http/Request.html)  |  `request`
Lang  |  [Illuminate\Translation\Translator](http://laravel.com/api/4.1/Illuminate/Translation/Translator.html)  |  `translator`
Log  |  [Illuminate\Log\Writer](http://laravel.com/api/4.1/Illuminate/Log/Writer.html)  |  `log`
Mail  |  [Illuminate\Mail\Mailer](http://laravel.com/api/4.1/Illuminate/Mail/Mailer.html)  |  `mailer`
Paginator  |  [Illuminate\Pagination\Factory](http://laravel.com/api/4.1/Illuminate/Pagination/Factory.html)  |  `paginator`
Paginator (Instance)  |  [Illuminate\Pagination\Paginator](http://laravel.com/api/4.1/Illuminate/Pagination/Paginator.html)  |
Password  |  [Illuminate\Auth\Reminders\PasswordBroker](http://laravel.com/api/4.1/Illuminate/Auth/Reminders/PasswordBroker.html)  |  `auth.reminder`
Queue  |  [Illuminate\Queue\QueueManager](http://laravel.com/api/4.1/Illuminate/Queue/QueueManager.html)  |  `queue`
Queue (Instance) |  [Illuminate\Queue\QueueInterface](http://laravel.com/api/4.1/Illuminate/Queue/QueueInterface.html)  |
Queue (Base Class) |  [Illuminate\Queue\Queue](http://laravel.com/api/4.1/Illuminate/Queue/Queue.html)  |
Redirect  |  [Illuminate\Routing\Redirector](http://laravel.com/api/4.1/Illuminate/Routing/Redirector.html)  |  `redirect`
Redis  |  [Illuminate\Redis\Database](http://laravel.com/api/4.1/Illuminate/Redis/Database.html)  |  `redis`
Request  |  [Illuminate\Http\Request](http://laravel.com/api/4.1/Illuminate/Http/Request.html)  |  `request`
Response  |  [Illuminate\Support\Facades\Response](http://laravel.com/api/4.1/Illuminate/Support/Facades/Response.html)  |
Route  |  [Illuminate\Routing\Router](http://laravel.com/api/4.1/Illuminate/Routing/Router.html)  |  `router`
Schema  |  [Illuminate\Database\Schema\Blueprint](http://laravel.com/api/4.1/Illuminate/Database/Schema/Blueprint.html)  |
Session  |  [Illuminate\Session\SessionManager](http://laravel.com/api/4.1/Illuminate/Session/SessionManager.html)  |  `session`
Session (Instance)  |  [Illuminate\Session\Store](http://laravel.com/api/4.1/Illuminate/Session/Store.html)  |
SSH  |  [Illuminate\Remote\RemoteManager](http://laravel.com/api/4.1/Illuminate/Remote/RemoteManager.html)  |  `remote`
SSH (Instance)  |  [Illuminate\Remote\Connection](http://laravel.com/api/4.1/Illuminate/Remote/Connection.html)  |
URL  |  [Illuminate\Routing\UrlGenerator](http://laravel.com/api/4.1/Illuminate/Routing/UrlGenerator.html)  |  `url`
Validator  |  [Illuminate\Validation\Factory](http://laravel.com/api/4.1/Illuminate/Validation/Factory.html)  |  `validator`
Validator (Instance)  |  [Illuminate\Validation\Validator](http://laravel.com/api/4.1/Illuminate/Validation/Validator.html)
View  |  [Illuminate\View\Factory](http://laravel.com/api/4.1/Illuminate/View/Factory.html)  |  `view`
View (Instance)  |  [Illuminate\View\View](http://laravel.com/api/4.1/Illuminate/View/View.html)  |