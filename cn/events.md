# 事件

- [基本用法](#basic-usage)
- [利用通配符匹配多个事件](#wildcard-listeners)
- [使用类作为监听器](#using-classes-as-listeners)
- [事件队列](#queued-events)
- [事件订阅者](#event-subscribers)

<a name="basic-usage"></a>
## 基本用法

Laravel中的`Event`类实现了简单的观察者模式，允许你在应用程序中订阅和监听事件。

#### 订阅事件

	Event::listen('auth.login', function($user)
	{
		$user->last_login = new DateTime;

		$user->save();
	});

#### 触发事件

	$event = Event::fire('auth.login', array($user));

#### 带有优先级的事件订阅

你也可以在订阅事件时指定优先级。优先级高的监听器将会被优先执行，相同优先级的将被按照订阅顺序执行。

	Event::listen('auth.login', 'LoginHandler', 10);

	Event::listen('auth.login', 'OtherHandler', 5);

#### 阻止事件传播

如果希望阻止事件传播到其他监听器，只需让监听器返回`false`即可实现：

	Event::listen('auth.login', function($event)
	{
		// Handle the event...

		return false;
	});

### 在哪儿注册事件

So, you know how to register events, but you may be wondering _where_ to register them. Don't worry, this is a common question. Unfortunately, it's a hard question to answer because you can register an event almost anywhere! But, here are some tips. Again, like most other bootstrapping code, you may register events in one of your `start` files such as `app/start/global.php`.

If your `start` files are getting too crowded, you could create a separate `app/events.php` file that is included from a `start` file. This is a simple solution that keeps your event registration cleanly separated from the rest of your bootstrapping. If you prefer a class based approach, you may register your events in a [service provider](/docs/ioc#service-providers). Since none of these approaches is inherently "correct", choose an approach you feel comfortable with based on the size of your application.

<a name="wildcard-listeners"></a>
## 利用通配符匹配多个事件

#### 注册多个事件的监听器

注册事件监听器时，可以使用星号匹配多个事件进行监听：

	Event::listen('foo.*', function($param)
	{
		// Handle the event...
	});

该监听器将会处理所有以`foo.`开始的事件。

You may use the `Event::firing` method to determine exactly which event was fired:

	Event::listen('foo.*', function($param)
	{
		if (Event::firing() == 'foo.bar')
		{
			//
		}
	});

<a name="using-classes-as-listeners"></a>
## 使用类作为监听器

在某些情况下，你可能希望使用类来代替闭包处理事件。类事件监听器（Class event listener）通过 [Laravel IoC 容器](/docs/ioc)实现，为监听器提供了强大的依赖注入的能力。

#### 注册一个类监事件听器

	Event::listen('auth.login', 'LoginHandler');

#### 定义一个事件监听器类

默认情况下，`LoginHandler`类的`handle`方法将被调用：

	class LoginHandler {

		public function handle($data)
		{
			//
		}

	}

#### 指定实现订阅的方法

如果你不想使用默认的`handle`方法，你可以在订阅时指定方法：

	Event::listen('auth.login', 'LoginHandler@onLogin');

<a name="queued-events"></a>
## 事件队列

#### 向事件队列中添加一个事件

使用`queue`和`flush`方法，可以将事件放入"队列"中，而不是立刻执行：

	Event::queue('foo', array($user));

#### 注册一个事件清除器

	Event::flusher('foo', function($user)
	{
		//
	});

最后，你可以使用`flush`方法清除掉队列中的所有事件：

	Event::flush('foo');

<a name="event-subscribers"></a>
## 事件订阅者

#### 定义一个事件订阅者

事件订阅者类可以在其自身内部订阅多个事件。并且事件订阅者应该定义`subscribe`方法，并接收一个事件调度器实例作为参数：

	class UserEventHandler {

		/**
		 * Handle user login events.
		 */
		public function onUserLogin($event)
		{
			//
		}

		/**
		 * Handle user logout events.
		 */
		public function onUserLogout($event)
		{
			//
		}

		/**
		 * Register the listeners for the subscriber.
		 *
		 * @param  Illuminate\Events\Dispatcher  $events
		 * @return array
		 */
		public function subscribe($events)
		{
			$events->listen('auth.login', 'UserEventHandler@onUserLogin');

			$events->listen('auth.logout', 'UserEventHandler@onUserLogout');
		}

	}

#### 注册一个事件订阅者

一旦定义了事件订阅者，就可以使用`Event`类进行注册。

	$subscriber = new UserEventHandler;

	Event::subscribe($subscriber);

You may also use the [Laravel IoC container](/docs/ioc) to resolve your subscriber. To do so, simply pass the name of your subscriber to the `subscribe` method:

	Event::subscribe('UserEventHandler');

