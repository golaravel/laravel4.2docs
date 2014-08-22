# IoC 容器

- [简介](#introduction)
- [基本用例](#basic-usage)
- [Where To Register Bindings](#where-to-register)
- [自动解析](#automatic-resolution)
- [实际用例](#practical-usage)
- [服务提供器](#service-providers)
- [容器事件](#container-events)

<a name="introduction"></a>
## 简介

Laravel 控制器反转容器是一个强大的工具来处理类依赖关系。依赖注入是一个不用硬代码处理类依赖关系的方法。依赖关系是在运行时注入的，允许处理依赖时具有更大的灵活性。

理解 Laravel IoC 容器是构建强大应用程序所必要的，也有助于 Laravel 核心本身。

<a name="basic-usage"></a>
## 基本用例

#### 绑定一个类型到容器

IoC 容器有两种方法来解决依赖关系：通过闭包回调或者自动解析。首先，我们来探究一下闭包回调。首先，需要绑定一个“类型”到容器中：

	App::bind('foo', function($app)
	{
		return new FooBar;
	});

#### 从容器中取得一个类型

	$value = App::make('foo');

当执行 `App::make` 方法，闭包函数被执行并返回结果。

#### 绑定一个”共享“类型到容器

有时，你只想将绑定到容器的类型只应该取得一次，然后接下来从容器中取得的都应该是相同实例：

	App::singleton('foo', function()
	{
		return new FooBar;
	});

#### 绑定一个已经存在的类型实例到容器

你也可以使用 `instance`方法 将一个已经存在的类型绑定到容器中：

	$foo = new Foo;

	App::instance('foo', $foo);

<a name="where-to-register"></a>
## Where To Register Bindings

IoC bindings, like event handlers or route filters, generally fall under the title of "bootstrap code". In other words, they prepare your application to actually handle requests, and usually need to be executed before a route or controller is actually called. Like most other bootstrap code, the `start` files are always an option for registering IoC bindings. Alternatively, you could create an `app/ioc.php` (filename does not matter) file and require that file from your `start` file.

If your application has a very large number of IoC bindings, or you simply wish to organize your IoC bindings in separate files by category, you may register your bindings in a [service provider](#service-providers).

<a name="automatic-resolution"></a>
## 自动解析

#### 取得一个类

IoC 容器在许多场景下足够强大来取得类而不需要任何配置。例如：

	class FooBar {

		public function __construct(Baz $baz)
		{
			$this->baz = $baz;
		}

	}

	$fooBar = App::make('FooBar');

注意我们虽然没有在容器中注册FooBar类，容器仍然可以取得该类，甚至自动注入 `Baz` 依赖！

当某个类型没有绑定到容器，将使用 PHP 的反射工具来检查类和读取构造器的类型提示。使用这些信息，容器可以自动构建类实例。

然而，在某些情况下，一个类可能依赖某个接口实现，而不是一个 “具体的类”。当在这种情况下，`App::bind` 方法必须通知容器注入该实现哪个接口：

#### 绑定一个接口给实现类

	App::bind('UserRepositoryInterface', 'DbUserRepository');

现在考虑以下控制器：

	class UserController extends BaseController {

		public function __construct(UserRepositoryInterface $users)
		{
			$this->users = $users;
		}

	}

由于我们将 `UserRepositoryInterface` 绑定了具体类，`DbUserRepository` 在该控制器创建时将自动注入到该控制器。

<a name="practical-usage"></a>
## 实际用例

Laravel 提供了几个方法使用 IoC 容器增强应用程序可扩展性和可测试性。一个主要的例子是取得控制器。所有控制器都通过 IoC 容器取得，意味着可以解决控制器构造方法类型提示的依赖关系，它们将自动被注入。

#### 控制器类型提示依赖关系

	class OrderController extends BaseController {

		public function __construct(OrderRepository $orders)
		{
			$this->orders = $orders;
		}

		public function getIndex()
		{
			$all = $this->orders->all();

			return View::make('orders', compact('all'));
		}

	}

在这个例子中，`OrderRepository` 将会自动注入到控制器。意味着当 [单元测试](/docs/testing) 模拟请求时，`OrderRepository` 将会绑定到容器以及注入到控制器中，允许无痛与数据库层交互。

#### IoC 使用的其他例子

[过滤器](/docs/routing#route-filters), [composers](/docs/responses#view-composers), 和 [事件处理器](/docs/events#using-classes-as-listeners) 也通过 IoC 容器来取得。当注册了它们，仅简单提供要使用的类名：

	Route::filter('foo', 'FooFilter');

	View::composer('foo', 'FooComposer');

	Event::listen('foo', 'FooHandler');

<a name="service-providers"></a>
## 服务提供器

服务器提供器是将一组相关 IoC 注册到单一路径的有效方法。将它们看做是一种引导组件的方法。在服务器提供器里，你可以注册自定义的验证驱动器，使用 IoC 容器注册应用程序仓库类，甚至是自定义 Artisan 命令。

事实上，大多数核心 Laravel 组件 包含服务提供器。应用程序所有注册在服务提供器的均列在 `app/config/app.php` 配置文件的 `providers` 数组中。

#### 定义一个服务提供器


要创建服务提供器，简单的继承 `Illuminate\Support\ServiceProvider` 类并且定义一个 `register` 方法：
	use Illuminate\Support\ServiceProvider;

	class FooServiceProvider extends ServiceProvider {

		public function register()
		{
			$this->app->bind('foo', function()
			{
				return new Foo;
			});
		}

	}

注意在 `register` 方法，应用程序通过 `$this->app` 属性访问 IoC 容器。一旦你已经创建了提供器并且想将它注册到应用程序中， 只需简单的放入 `app` 配置文件里 `providers` 数组中。

#### 运行时注册服务提供器

也可以使用 `App::register` 方法在运行时注册服务提供器：

	App::register('FooServiceProvider');

<a name="container-events"></a>
## 容器事件

#### 注册获取事件监听者

容器在每次获取对象时都触发一个时间。你可以通过使用 `resolving` 方法来监听该事件：

	App::resolvingAny(function($object)
	{
		//
	});

	App::resolving('foo', function($foo)
	{
		//
	});

注意获取的对象将会传入回调函数中。
