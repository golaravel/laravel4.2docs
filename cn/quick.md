# Laravel 快速入门

- [安装](#installation)
- [路由](#routing)
- [创建视图](#creating-a-view)
- [创建迁移](#creating-a-migration)
- [Eloquent ORM](#eloquent-orm)
- [显示数据](#displaying-data)

<a name="installation"></a>
## 安装

### 通过 Laravel Installer

首先，下载[Laravel installer PHAR 归档]。为了方便，重名文件为`laravel`并且移动到`/usr/local/bin`目录。一旦安装完成，使用简单的`laravel new`命令将会在特定目录下创建一个全新的laravel环境。例如，`laravel new blog`将创建`blog`目录，里面包含有所有laravel安装及依赖。该方法比通过Composer安装要快。

### 通过 Composer

Laravel使用[Composer](http://getcomposer.org)执行安装和管理依赖。如果你还没有安装，从[安装 Composer](http://getcomposer.org/doc/00-intro.md)开始吧。

现在你可以在终端使用如下命令来安装Laravel:

	composer create-project laravel/laravel your-project-name --prefer-dist
	
该命令将下载以及安装全新的Laravel拷贝，存放在`your-project-name`目录。

如果你喜欢的话，你还可以手动拷贝[Laravel Github 仓库](https://github.com/laravel/laravel/archive/master.zip)来代替安装。接着在手动创建项目根目录下执行`composer install`命令。该命令将会下载并安装框架依赖。

### 权限

安装完Laravel，你需要确保web服务器具有`app/storage`目录写入权限。查看[安装](/docs/installation)文档了解配置细节。


### Serving Laravel

Typically, you may use a web server such as Apache or Nginx to serve your Laravel applications. If you are on PHP 5.4+ and would like to use PHP's built-in development server, you may use the `serve` Artisan command:

	php artisan serve

<a name="directories"></a>
### 目录结构

安装完框架后，你需要熟悉一下该项目的目录结构。`app` 文件夹包含了一些例如 `views` ，`controllers` 和 `models` 目录。 程序中大部分代码将要存放这些目录下。你也可以查看一下 `app/config` 文件夹里一些配置项目。

<a name="routing"></a>
## 路由

我们开始创建我们第一个路由。在 Laravel，简单路由的方法是闭包。打开 `app/routes.php` 文件加入如下代码:

	Route::get('users', function()
	{
		return 'Users!';
	});

现在，你在 web 浏览器输入 `/users`，你应该会看到 `Users!` 输出。真棒！已经创建了你第一个路由。

路由也可以赋予控制器类。例如：

	Route::get('users', 'UserController@getIndex');

该路由告知框架 `/users` 路由请求应该调用 `UserController` 类的 `getIndex` 方法。要查看更多关于路由控制器信息，查看 [控制器文档](/docs/controllers) 。


<a name="creating-a-view"></a>
## 创建视图

接下来，我们要创建视图来显示我们用户数据。视图以HTML代码存放在 `app/views` 文件夹。我们将存放两个视图文件到该文件夹：`layout.blade.php` 和 `users.blade.php`。首先，让我们先创建 `layout.blade.php` 文件：

	<html>
		<body>
			<h1>Laravel Quickstart</h1>

			@yield('content')
		</body>
	</html>

接着， 我们创建 `users.blade.php` 视图：

	@extends('layout')

	@section('content')
		Users!
	@stop

这里的语法可能让你感到陌生。因为我们使用的是 Laravel 模板系统：Blade。Blade 非常快，因为仅使用了少量的正则表达式来为你的模板编译成原始PHP代码。Blade提供强大的功能，例如模板继承，还有一些常用的PHP控制结构语法糖，例如 `if` 和 `for`。 查看 [Blade 文档](/docs/templates) 了解更多。

现在我们有了我们视图，让我们返回 `/users` 路由。我们用视图来替代返回 `Users!`:

	Route::get('users', function()
	{
		return View::make('users');
	});

漂亮！现在你成功创建了继承至layout的视图。接下来，让我们开始数据库层。


<a name="creating-a-migration"></a>
## 创建迁移

要创建表来保存我们数据，我们将使用 Laravel 迁移系统。迁移描述数据库的改变，这让分享给他们团队成员非常简单。

首先，我们配置数据库连接。你可以在 `app/config/database.php` 文件配置所有数据库连接信息。默认，Laravel 被配置为使用 SQLite，并且一个 SQLite 数据库存放在 `app/database` 目录。你可以将数据库配置文件的 `driver` 选项修改为 `mysql` 并且配置 `mysql` 连接信息。

接下来，要创建迁移，我们将使用 [Artisan CLI](/docs/artisan)。在项目根目录中，在终端中执行以下命令：

	php artisan migrate:make create_users_table

然后，找到生成的迁移文件 `app/database/migrations` 目录。该文件包含了一个包含两个方法： `up` 和 `down` 的类。在 `up` 方法，你要指名数据库表的修改，在 `down` 方法中你只需要移除它。

让我们定义如下迁移：

	public function up()
	{
		Schema::create('users', function($table)
		{
			$table->increments('id');
			$table->string('email')->unique();
			$table->string('name');
			$table->timestamps();
		});
	}

	public function down()
	{
		Schema::drop('users');
	}

然后，我们在项目根目录中使用终端运行 `migrate` 命令来执行迁移：

	php artisan migrate

如果你想回滚迁移，你可以执行 `migrate:rollback` 命令。现在我们已经有了数据库表，让我们让添加一些数据！

<a name="eloquent-orm"></a>
## Eloquent ORM

Laravel 提供非常棒的 ORM：Eloquent。如果你使用过 Ruby on Rails 框架，你会发现 Eloquent 很相似，因为它遵循数据库交互的 ActiveRecord ORM 风格。

首先，让我们来定义个模型。ELoquent 模型可以用来查询相关数据表，以及表内的某一行。别着急，我们很快会谈及！模型通常存放在 `app/models` 目录。让我们在该目录定义个 `User.php` 模型，如：

	class User extends Eloquent {}

注意我们并没有告诉 Eloquent 使用哪个表。Eloquent 有多种约定， 一个是使用模型的复数形式作为模型的数据库表。非常方便！

使用你喜欢的数据库管理工具，插入几行数据到 `users` 表，我们将使用 Eloquent 取得它们并传递到视图中。

现在我们修改我们 `/users` 路由如下：

	Route::get('users', function()
	{
		$users = User::all();

		return View::make('users')->with('users', $users);
	});

让我们来看看该路由。首先，`User` 模型的 `all` 方法将会从 `users` 表中取得所有记录。接下来，我们通过 `with` 方法将这些记录传递到视图。`with` 方法接受一个键和一个值，那么该值就可以在视图中使用了。

激动啊。现在我们准备将用户显示在我们视图！

<a name="displaying-data"></a>
## 显示数据

现在我们视图中已经可以访问 `users` 类，我们可以如下显示它们：

	@extends('layout')

	@section('content')
		@foreach($users as $user)
			<p>{{ $user->name }}</p>
		@endforeach
	@stop

你可以发现没有找到 `echo` 语句。当使用 Blade 时，你可以使用两个花括号来输出数据。非常简单，你现在应该可以通过 `/users` 路由来查看到用户姓名作为响应输出。

这仅仅是开始。在本系列教程中，你已经了解了 Laravel 基础部分，但是还有更让人兴奋的东西要学。继续阅读该文档并且深入[Eloquent](/docs/eloquent)和[Blade](/docs/templates)这些强大的特性。或者你对[队列](/docs/queues) 和 [单元测试](/docs/testing) 感兴趣。或许是你想了解[IoC Container](/docs/ioc), 选择权在于你!