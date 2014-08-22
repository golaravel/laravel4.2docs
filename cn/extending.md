# 对框架进行扩展

- [介绍](#introduction)
- [管理者 & 工厂](#managers-and-factories)
- [Where To Extend](#where-to-extend)
- [缓存](#cache)
- [Session会话](#session)
- [用户验证](#authentication)
- [基于IoC的扩展](#ioc-based-extension)
- [请求(Request)相关扩展](#request-extension)

<a name="introduction"></a>
## 介绍
Laravel 为你提供了很多可扩展的地方， 通过它们你可以定制框架中一些核心组件的行为，甚至对这些核心组件进行全部替换。 例如， 哈希组件是在 "HaserInterface" 接口中被定义的，你可以根据应用程序的需求来选择是否实现它。 你也可以扩展 "Request" 对象来添加专属你的"帮助"方法。你甚至可以添加全新的 用户验证，缓存以及会话(Session) 驱动！

Laravel 的组件通常以两种方式进行扩展:在IoC容器中绑定新的实现，或者为一个扩展注册一个 "Manager" 类，第二种方式运用了设计模式中"工厂"的理念。在这一章中，我们会探索扩展框架的各种方法以及查看一些具体的实现代码。

> **注意:** 
请记住，Laravel 组件的扩展通常使用以下两种方法的其中一种实现的:IoC绑定和 "Manager" 类。 Manager类充当实现工厂模式的作用，它负责缓存、会话(Session)等基本驱动的实例化工作。

<a name="managers-and-factories"></a>
## 管理者 & 工厂

Laravel 有几个 "Manager" 类 ，用来管理一些基本驱动组件的创建工作。 包括 缓存、会话(Session)、用户验证以及队列组件。"Manager"类负责根据应用程序中相关配置来创建一个驱动实现。例如，"CacheManager"类可以创建 <a href="http://www.php.net/apc">APC</a>、Memcached、File 以及其他各种缓存驱动的实现。
每一个Manager类都包含有一个"extend"方法，通过它可以轻松的将新驱动中的解决方案和功能注入到Manager中。下面，我们会通过列举一些向Manager中注入定制驱动的例子来依次对它们进行说明。


> **注意:**
花一些时间来探索Laravel中各种不同的"Manager"类，例如，"CacheManager"以及"SessionManager"。通过阅读这些类，可以让你对Laravel的底层实现有一个更加彻底的了解。 所有的"Manager"类都继承了"Illuminate\Support\Manager"基类， 这个基类为每一个"Manager"类提供了一些有用的，常见的功能。
 
<a name="where-to-extend"></a>
## Where To Extend

This documentation covers how to extend a variety of Laravel's components, but you may be wondering where to place your extension code. Like most other bootstrapping code, you are free to place some extensions in your `start` files. Cache and Auth extensions are good candidates for this approach. Other extensions, like `Session`, must be placed in the `register` method of a service provider since they are needed very early in the request life-cycle.

<a name="cache"></a>
## 缓存

要扩展Laravel的缓存机制，我们需要使用"CacheManager"的"extend"方法来为"manager"绑定一个定制的驱动解析器，这个方法在所有的"manager"中都是通用的。例如，注册一个新的名叫"mongo"的缓存驱动，我们需要做一下操作:


	Cache::extend('mongo', function($app)
	{
		// Return Illuminate\Cache\Repository instance...
	});

传入"extend"方法中的第一个参数是这个驱动的名字。这个会与你在"app/config/cache.php"配置文件中的"driver"选项相对应。第二个参数是一个返回"Illuminate\Cache\Repository"实例的闭包。 这闭包需要传入一个"$app"实例， 它是"Illuminate\Foundation\Application" 的一个实例，并且是一个IoC容器。

为了创建我们定制的缓存驱动，我们首先应该实现"Illuminate\Cache\StoreInterface"接口。 因此，我们的 MongDB 缓存的实现应该是这样的:

	class MongoStore implements Illuminate\Cache\StoreInterface {

		public function get($key) {}
		public function put($key, $value, $minutes) {}
		public function increment($key, $value = 1) {}
		public function decrement($key, $value = 1) {}
		public function forever($key, $value) {}
		public function forget($key) {}
		public function flush() {}

	}

我们要用一个MongoDB连接来实现所有这些方法。一旦完成了这些实现,我们就完成了定制驱动的注册。

	use Illuminate\Cache\Repository;

	Cache::extend('mongo', function($app)
	{
		return new Repository(new MongoStore);
	});

正如你在上面的例子中所看到的，你会在创建定制缓存驱动时使用到"Illuminate\Cache\Repository"基类。通常情况下，不需要自己创建"Repository"类。

如果你不知道将定制的缓存驱动代码放在哪里，那么可以考虑将它们放在<a href="https://packagist.org/">Packagist</a>中，或者你可以在应用程序的主目录下创建一个"Extension"子目录。例如，如果你的应用程序叫"Snappy"，你可以将你的缓存扩展放在"app/Snappy/Extensions/MongoStore.php"中。 请记住，Laravel对于应用程序的结构没有严格的限制，你可以自由地根据自己的选择来组织你的应用程序结构。


> **注意:** 
当你不知道将代码放在哪里时，请回想一下"service provider" 。  我们已经讨论过，利用"service provider"来组织你的框架扩展是组织代码的最好方式。


<a name="session"></a>
## 会话 Session

为Laravel扩展一个定制的Session驱动跟扩展一个缓存系统一样简单。同样，我们需要用 "extend" 方法来注册定制的驱动:

	Session::extend('mongo', function($app)
	{
		// Return implementation of SessionHandlerInterface
	});

### Where To Extend The Session

Session extensions need to be registered differently than other extensions like Cache and Auth. Since sessions are started very early in the request-lifecycle, registering the extensions in a `start` file will happen be too late. Instead, a [service provider](/docs/ioc#service-providers) will be needed. You should place your session extension code in the `register` method of your service provider, and the provider should be placed **below** the default `Illuminate\Session\SessionServiceProvider` in the `providers` configuration array.

### Writing The Session Extension

请注意，定制的缓存驱动需要实现"SessionHandlerInterface"接口。这个接口包含在在PHP5.4+core中。如果你正在使用PHP5.3，Laravel会帮你定义这个接口，这使得你的系统可以向前兼容。我们只需要实现这个接口中的一些简单的方法。 下面是一个简化的MongoDB实现:

	class MongoHandler implements SessionHandlerInterface {

		public function open($savePath, $sessionName) {}
		public function close() {}
		public function read($sessionId) {}
		public function write($sessionId, $data) {}
		public function destroy($sessionId) {}
		public function gc($lifetime) {}

	}

上面这些方法不像在"StoreInterface"缓存接口中的方法一样让人容易理解。下面让我们快速的了解一下每一个方法的功能:

- "open"方法通常在基于文件的Session存储系统中使用。因为Laravel自带了PHP文件存储session的驱动。因此，你不需要在这个方法中添加任何实现。事实上 PHP强制要求我们实现这个不需要添加任何代码的方法，这实在是一个糟糕的接口设计(在后面的内容中会讨论这一点)。
-"close"方法也像"open"方法一样，通常是可以被忽略的。大多数驱动不需要这个方法。
-"read"方法会返回一个与传入的"$sessionId"相关联的字符串形式的Session数据。将驱动中的Session数据取出时，我们不需要做任何的序列化和转码的工作。因为Laravel会帮你处理这些。
-"write"方法是将与"$sessionId"相关联的"$data"字符串写入到一些持久化存储系统中。例如: MongoDB，Dynamo 等等。
-"destroy"方法将与"$sessionId"相关的数据从持久化系统中移除。
-"gc"方法会删除超过传入"$lifetime"(一个UNIX 时间戳)的所有Session数据。对于像Memcached和Redis这一类（self-expired）自动管理超期的系统，"gc"方法内不应该有任何代码。

实现"SessionHandlerInterface"后，我们可以将它注册到Session管理中。

	Session::extend('mongo', function($app)
	{
		return new MongoHandler;
	});

session驱动成功注册后，我们就可以使用"app/config/session.php"配置文件中的"mongo"驱动了。


> **注意:** 如果你是在编写一个定制的session处理程序，请在<a href="https://packagist.org/">Packagist</a>中分享!

<a name="authentication"></a>
## 用户验证

用户验证的扩展方法和之前讨论过的缓存和session组件的扩展方法很相似。
同样，我们会使用这个我们已经渐渐熟悉的"extend"方法。

	Auth::extend('riak', function($app)
	{
		// Return implementation of Illuminate\Auth\UserProviderInterface
	});

"UserProviderInterface"的实现仅仅是负责从如 MySQL、Riak等 持久化存储系统中将"UserInterface"实现获取出来。这两个接口使得Laravel的用户验证机制在不同的用户数据存储方法以及使用不同的类来展示的情况下都可以保持正常的功能。

让我们看一下"UserProviderInterface"接口：

	interface UserProviderInterface {

		public function retrieveById($identifier);
		public function retrieveByCredentials(array $credentials);
		public function validateCredentials(UserInterface $user, array $credentials);

	}

"retrieveById"方法通常接收一个代表用户的数字形式的键，例如在MySQL数据库中自增的ID字段值。"UserInterface"实现会通过这个方法来比对接收和返回的ID。

"retrieveByCredentials"方法接收在试图登录应用程序时传入"Auth:attempt"方法的一个认证信息数组。然后，"retrieveByCredentials"方法会从底层的持久化存储系统中查询与传入身份信息相符合的用户。通常情况下，"retrieveByCredentials"方法会根据"$credentials['username']"来设定"where"条件来进行查询。
 **"retrieveByCredentials"方法不应该试图做任何密码的确认和验证操作。**

"validateCredentials" 方法会将传入的"$user"用户和"$credentials"身份信息进行对比并验证这个用户。例如，这个方法会将"$user->getAuthPassword()"返回的字符串和经过"Hash::make"方法哈希处理的"$credentials['password']"进行对比。

现在，我们已经讨论过"UserProviderInterface"中的所有方法，让我们来看一看"UserInterface"接口。请记住，"provider"必须在"retrieveById"和"retrieveByCredentials"方法中返回对于这个接口的实现：

	interface UserInterface {

		public function getAuthIdentifier();
		public function getAuthPassword();

	}

这个接口非常简单。"getAuthIdentifier"返回这个用户的"主键",在MySQL后台中，这个主键就是自增的primary key。"getAuthPassword"方法返回这个用户进过hash加密后的密码。这个接口可以让用户验证系统在任何User类的基础上正常工作，不管你是在使用哪种ORM或存储抽象层。默认情况下，Laravel在"app/models"目录下会有一个实现了这个接口的"User"类，因此，你可以参照这个作一些实现。

最后，完成了"UserProviderInterface"实现后，我们可以将"Auth" 扩展注册到用户认证管理中:

	Auth::extend('riak', function($app)
	{
		return new RiakUserProvider($app['riak.connection']);
	});

当你通过extend方法注册了这个驱动后，你可以将当前的用户验证驱动切换为"app/config/auth.php"配置文件中的新驱动。


<a name="ioc-based-extension"></a>
## 基于IoC的扩展

几乎所有的service提供者包括Laravel框架都会将对象绑定在IoC容器中，你可以在"app/config/app.php"配置文件中找到关于你应用程序中的所有service提供者。如果有时间，可以浏览这些service提供者的源代码。通过浏览这些细节，你可以更深刻的了解这些service提供者在框架中添加了什么内容，以及IoC容器利用了哪些键来绑定这些service提供者。

例如，"HashServiceProvider"在IoC容器中绑定了一个"hash"键，这个键对应一个"Illuminate\Hashing\BcryptHasher"实例。通过重写这个IoC绑定，你可以轻松的在你的应用程序中扩展以及重写这个类。例如：

	class SnappyHashProvider extends Illuminate\Hashing\HashServiceProvider {

		public function boot()
		{
			App::bindShared('hash', function()
			{
				return new Snappy\Hashing\ScryptHasher;
			});

			parent::boot();
		}

	}
    
请注意，这个类继承的是 "HashServiceProvider" 类，而不是默认的"ServiceProvider"基类。完成service提供者的扩展之后，将在配置文件"app/config/app.php"中的"HashServiceProvider"替换成经过扩展的service提供者。

这是为扩展绑定在容器中的核心类而通常使用的方法。基本上每一个核心类都是通过这样的方式绑定在容器中，并且可以对它们进行重写。再一次提醒，详细阅读框架中所包含的service提供者会使你更加清楚这些类绑定在容器中的什么地方，以及它们与什么Key（键值）所绑定着。这是学习Laravel究竟是如何组织的最好的方法。


<a name="request-extension"></a>
## Request（请求）扩展

由于"Request"是框架中非常基础性的组件，而且它在Request请求周期中很早就被实例化了，因此扩展"Request"类与前面所讲的例子有一些不同。

首先，用上述的方法来扩展这个类：


	<?php namespace QuickBill\Extensions;

	class Request extends \Illuminate\Http\Request {

		// Custom, helpful methods here...

	}

完成了对这个类的扩展之后，打开"bootstrap/start.php"文件。当每一个request请求到达应用程序时，"bootstrap/start.php"文件是第一个被引入的。请注意。创建Laravel "$app"实例是第一个被执行的活动：

	$app = new \Illuminate\Foundation\Application;

当一个新的应用程序实例被创建后，它将会创建一个新的"Illuminate\Http\Request"实例并通过"request"键将它绑定在IoC容器中。因此，我们需要通过一种方法来指定一个定制的类作为"默认"的请求类型。对吗？ 令人兴奋的是，应用程序实例中的"requestClass"方法正是用来做这个工作的 ! 因此，我们可以在"bootstrap/start.php" 文件的最顶端添加这一行：

	use Illuminate\Foundation\Application;

	Application::requestClass('QuickBill\Extensions\Request');

当你完成了指定定制的"Request"类之后，Laravel 在任何时候创建"Request"实例，都会使用这个类。Laravel 方便的让你在任何时候都拥有一个自己定制的"Request"类的实例，即使在"unit tests"中也是一样！

