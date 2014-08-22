# Session

- [配置](#configuration)
- [Session 用法](#session-usage)
- [闪存数据](#flash-data)
- [数据库 Sessions](#database-sessions)
- [Session 启动](#session-drivers)

<a name="configuration"></a>
## 配置

因为HTTP协议本身是无状态的，session提供了一种保存用户请求信息的途径。Laravel框架可以使用多种session后端驱动，并且提供了清晰、统一的API支持。框架内置支持一些比较流行的后端驱动如[Memcached](http://memcached.org)、 [Redis](http://redis.io)和数据库。

session的配置被存放在 `app/config/session.php` 文件中。请务必查看一下这个文件中那些带有注释的配置选项。Laravel默认使用`基于文件（file）`的session驱动，它可以在大多数应用中良好地工作。

#### Reserved Keys

The Laravel framework uses the `flash` session key internally, so you should not add an item to the session by that name.

<a name="session-usage"></a>
## Session 用法

#### 储存一个Session变量

	Session::put('key', 'value');

#### Push A Value Onto An Array Session Value

	Session::push('user.teams', 'developers');

#### 读取一个Session变量

	$value = Session::get('key');

#### 读取一个Session变量或者返回默认值

	$value = Session::get('key', 'default');

	$value = Session::get('key', function() { return 'default'; });
    
#### 读取一个变量并遗忘它

	$value = Session::pull('key', 'default');

#### 读取所有Session数据

	$data = Session::all();

#### 检查一个Session变量是否存在

	if (Session::has('users'))
	{
		//
	}

#### 删除一个Session变量

	Session::forget('key');

#### 删除所有Session变量

	Session::flush();

#### 重置 Session ID

	Session::regenerate();

<a name="flash-data"></a>
## 闪存数据

有时，你也许希望将信息暂存在session中到下一次请求为止，你可以使用 `Session::flash` 方法达到这个目的：

	Session::flash('key', 'value');

#### 刷新当前闪存数据，延长其过期时间到下一个请求

	Session::reflash();

#### 刷新一部分闪存数据

	Session::keep(array('username', 'email'));

<a name="database-sessions"></a>
## 数据库 Sessions

当使用 `数据库（database）` session驱动时，你需要设置一张存储session数据的表。下面的例子展示了使用 `Schema` 声明新建一张表：

	Schema::create('sessions', function($table)
	{
		$table->string('id')->unique();
		$table->text('payload');
		$table->integer('last_activity');
	});

当然，你也可以使用Artisan命令 `session:table` 来创建这张表：

	php artisan session:table

	composer dump-autoload

	php artisan migrate

<a name="session-drivers"></a>
## Session Drivers

The session "driver" defines where session data will be stored for each request. Laravel ships with several great drivers out of the box:

- `file` - sessions will be stored in `app/storage/sessions`.
- `cookie` - sessions will be stored in secure, encrypted cookies.
- `database` - sessions will be stored in a database used by your application.
- `memcached` / `redis` - sessions will be stored in one of these fast, cached based stores.
- `array` - sessions will be stored in a simple PHP array and will not be persisted across requests.

> **Note:** The array driver is typically used for running [unit tests](/docs/testing), so no session data will be persisted.