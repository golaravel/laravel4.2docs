# 安全

- [配置](#configuration)
- [保存密码](#storing-passwords)
- [验证用户](#authenticating-users)
- [手动登陆用户](#manually)
- [保护路由](#protecting-routes)
- [HTTP基本验证](#http-basic-authentication)
- [密码提示和重置](#password-reminders-and-reset)
- [加密](#encryption)
- [Authentication Drivers](#authentication-drivers)

<a name="configuration"></a>
## 配置

Laravel 旨在让验证实现简单。事实上，大部分都已经配置好了。验证配置文件位于 `app/config/auth.php`，该文件包含了一些文档说明非常齐全的配置选项，通过它们可以调整用户验证的具体形式。

一般情况下，Laravel在`app/models`目录下包含有一个`User` 模型，通常被作为默认的Eloquent验证驱动所使用。请注意在构建该数据库结构模型时确保密码字段至少能容纳60个字符。

如果你的应用中不使用 Eloquent，你可以使用 Laravel 查询构造器来使用 `数据库` 验证驱动。

> **Note:** Before getting started, make sure that your `users` (or equivalent) table contains a nullable, string `remember_token` column of 100 characters. This column will be used to store a token for "remember me" sessions being maintained by your application.

<a name="storing-passwords"></a>
## 保存密码

Laravel `Hash` 类提供了可靠的<a href='http://en.wikipedia.org/wiki/Bcrypt'>Bcrypt</a>散列算法：

#### 使用Bcrypt散列密码

	$password = Hash::make('secret');

#### 验证散列密码

	if (Hash::check('secret', $hashedPassword))
	{
		// 密码匹配...
	}

#### 检查密码是否需要重新散列

	if (Hash::needsRehash($hashed))
	{
		$hashed = Hash::make('secret');
	}

<a name="authenticating-users"></a>
## 用户验证

要在应用程序中登陆用户，使用 `Auth::attempt` 方法。

	if (Auth::attempt(array('email' => $email, 'password' => $password)))
	{
		return Redirect::intended('dashboard');
	}

注意 `email` 不是必需选项，这里仅作为示例。你应该使用任意数据库字段名来作为"用户名"。  `Redirect::intended` 方法会将用户请求重定向至被用户验证过滤器拦截之前用户试图访问URL中去。可以给该方法提供个回退URI，用于在访问目的无效时，重定向到该URI。

在调用 `attempt` 方法时，`auth.attempt` [事件](/docs/events) 将会触发。如果验证成功以及用户登陆了，`auth.login` 事件也会被触发。

#### 判断用户是否已经验证

要在应用程序中判断用户是否已经登陆，可以使用 `check` 方法：

	if (Auth::check())
	{
		// 用户已经登陆...
	}

#### 验证用户并且“记住”她们
	
如果你想在应用中提供“记住我”功能，你可以传递true作为第二个参数传递给attempt方法，应用程序将会无期限地保持用户验证状态（除非手动退出）：	

	if (Auth::attempt(array('email' => $email, 'password' => $password), true))
	{
		// 用户状态永久保存...
	}

**注意：** 如果 `attempt` 方法返回 `true`， 用户就已经成功登陆应用程序了。

#### 确定用户是否是通过“Remember”得到的授权

如果你在应用中“记住”了用户的登录状态，你可以使用 `viaRemember` 方法来确定用户是否正在使用“remember me”这个 cookie 来获得授权：

	if (Auth::viaRemember())
	{
		//
	}

#### 指定其它条件验证用户

你也可以在用户验证查询中加入其它条件：

    if (Auth::attempt(array('email' => $email, 'password' => $password, 'active' => 1)))
    {
        // 用户激动状态，没有暂停，并且存在。
    }

> **注意：** 为了增强安全性，防止会话固定，用户的 SESSION ID 会在每次验证后重新生成。

#### 查看登陆用户

一旦用户验证通过了，就可以查看 User 模型/记录：

	$email = Auth::user()->email;
    
To retrieve the authenticated user's ID, you may use the `id` method:

	$id = Auth::id();

要简单使用用户ID来登陆应用程序，可以使用 `loginUsingId` 方法：

	Auth::loginUsingId(1);

#### 验证用户但不登陆
    
`validate` 方法可以在不登录应用程序的情况下来验证用户的信息：

	if (Auth::validate($credentials))
	{
		//
	}

#### 登陆用户做单次请求

也可以使用 `once` 方法将一个用户登录到系统中做一个单次请求。这样的方式不会使用Session或Cookie来进行状态保持。

	if (Auth::once($credentials))
	{
		//
	}

#### 应用程序中注销用户登陆状态

	Auth::logout();

<a name="manually"></a>
## 手动登陆用户

如果想登陆一个存在的用户实例，只需要简单使用`login`方法并传入该实例即可：

	$user = User::find(1);

	Auth::login($user);

这与使用 `attempt` 方法验证用户登陆是一样的。


<a name="protecting-routes"></a>
## 保护路由

路由过滤器可以保证只有通过了验证的用户才能访问指定路由。Laravel 默认提供了 `auth` 过滤器，它定义在 `app/filters.php`文件。

#### 保护路由

	Route::get('profile', array('before' => 'auth', function()
	{
		// 只有验证用户可以访问……
	}));

### 跨站请求伪造（CSRF）保护

Laravel 提供便捷的方法来避免应用程序受到跨站伪造请求的攻击。

#### 表单中插入 CSRF Token

    <input type="hidden" name="_token" value="<?php echo csrf_token(); ?>">

#### 提供表单时验证 CSRF Token

    Route::post('register', array('before' => 'csrf', function()
    {
        return 'You gave a valid CSRF token!';
    }));

<a name="http-basic-authentication"></a>
## HTTP 基本验证

HTTP基本验证可以在不设定专门的登录页面的情况下便捷的对应用程序中的用户进行验证。要使用该功能，在路由中附加 `auth.filter` 过滤器：

#### HTTP 基本验证来保护路由

	Route::get('profile', array('before' => 'auth.basic', function()
	{
		// 只有验证用户可以访问……
	}));

默认情况下，`basic` 过滤器验证时会使用用户记录里的 `email` 字段。如果你想使用其他字段，你可以通过传递该字段名字给 `basic` 方法作为第一个参数：

	Route::filter('auth.basic', function()
	{
		return Auth::basic('username');
	});
    
#### 设置无状态的 HTTP 基本过滤器
    
你也可以使用 HTTP 基本验证时不将用户cookie标识写入 session，对 API 验证极其有用。要实现该功能，请定义一个返回 `onceBasic` 方法的过滤器：

	Route::filter('basic.once', function()
	{
		return Auth::onceBasic();
	});

如果你正在使用PHP FastCGI，那么HTTP 基本验证在默认情况下是不会正常工作的。应该将下面的代码加入到`.htaccess`文件中：

	RewriteCond %{HTTP:Authorization} ^(.+)$
	RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="password-reminders-and-reset"></a>
## 密码提示和重置

### Model & Table

大多web应用程序提供了让用户重置密码功能。与其让你在每个应用程序中重复去实现该功能，Laravel 提供非常方便的方法来发送密码提示以及执行密码重置。在开始前，请确认你的 `User` 模型实现了 `Illuminate\Auth\Reminders\RemindableInterface` 接口。 当然，框架中的默认的 `User` 模型已经实现了该接口。

#### 实现 RemindableInterface 接口

	use Illuminate\Auth\Reminders\RemindableTrait;
	use Illuminate\Auth\Reminders\RemindableInterface;

	class User extends Eloquent implements RemindableInterface {

		use RemindableTrait;

	}

#### 为提示表建立迁移

接下来，要创建一个表来存储密码重置 token。要为该表生成迁移，简单执行 Artisan 命令 `auth::reminders`：

	php artisan auth:reminders-table

	php artisan migrate

### Password Reminder Controller

Now we're ready to generate the password reminder controller. To automatically generate a controller, you may use the `auth:reminders-controller` Artisan command, which will create a `RemindersController.php` file in your `app/controllers` directory.

	php artisan auth:reminders-controller

The generated controller will already have a `getRemind` method that handles showing your password reminder form. All you need to do is create a `password.remind` [view](/docs/responses#views). This view should have a basic form with an `email` field. The form should POST to the `RemindersController@postRemind` action.

A simple form on the `password.remind` view might look like this:

	<form action="{{ action('RemindersController@postRemind') }}" method="POST">
		<input type="email" name="email">
		<input type="submit" value="Send Reminder">
	</form>

In addition to `getRemind`, the generated controller will already have a `postRemind` method that handles sending the password reminder e-mails to your users. This method expects the `email` field to be present in the `POST` variables. If the reminder e-mail is successfully sent to the user, a `status` message will be flashed to the session. If the reminder fails, an `error` message will be flashed instead.

Within the `postRemind` controller method you may modify the message instance before it is sent to the user:

	Password::remind(Input::only('email'), function($message)
	{
		$message->subject('Password Reminder');
	});

Your user will receive an e-mail with a link that points to the `getReset` method of the controller. The password reminder token, which is used to identify a given password reminder attempt, will also be passed to the controller method. The action is already configured to return a `password.reset` view which you should build. The `token` will be passed to the view, and you should place this token in a hidden form field named `token`. In addition to the `token`, your password reset form should contain `email`, `password`, and `password_confirmation` fields. The form should POST to the `RemindersController@postReset` method.

A simple form on the `password.reset` view might look like this:

	<form action="{{ action('RemindersController@postReset') }}" method="POST">
		<input type="hidden" name="token" value="{{ $token }}">
		<input type="email" name="email">
		<input type="password" name="password">
		<input type="password" name="password_confirmation">
		<input type="submit" value="Reset Password">
	</form>

Finally, the `postReset` method is responsible for actually changing the password in storage. In this controller action, the Closure passed to the `Password::reset` method sets the `password` attribute on the `User` and calls the `save` method. Of course, this Closure is assuming your `User` model is an [Eloquent model](/docs/eloquent); however, you are free to change this Closure as needed to be compatible with your application's database storage system.

If the password is successfully reset, the user will be redirected to the root of your application. Again, you are free to change this redirect URL. If the password reset fails, the user will be redirect back to the reset form, and an `error` message will be flashed to the session.

### 密码的验证规则

默认情况下 `Password::reset` 方法将会验证这个密码的长度 >= 6。你可以使用 `Password::validator` 方法，在它接收的匿名函数中自定义验证规则。在这个匿名函数中你可以做任何你想要的密码验证。请注意，你不需要验证密码是否匹配，因为这将由框架自动完成。

	Password::validator(function($credentials)
	{
		return strlen($credentials['password']) >= 6;
	});

> **注意：** 默认情况下，密码重置令牌的过期时间为1小时。你可以在 `app/config/auth.php` 文件的 `reminder.expire` 配置项中进行修改。

<a name="encryption"></a>
## 加密

Laravel通过 mcrypt PHP 的扩展提供了强大的AES 加密组件：

#### 加密

	$encrypted = Crypt::encrypt('secret');

> **注意:** 确认在 `app/config/app.php` 文件设置了一个16、24或32随机字符给 `key` 项。否则，加密的值是不安全的。

#### 解密

	$decrypted = Crypt::decrypt($encryptedValue);

#### 设置 Cipher 和 Mode

你也可以设置在encrypter中使用的 cipher 和 mode：

	Crypt::setMode('crt');

	Crypt::setCipher($cipher);

<a name="authentication-drivers"></a>
## Authentication Drivers

Laravel offers the `database` and `eloquent` authentication drivers out of the box. For more information about adding additional authentication drivers, check out the [Authentication extension documentation](/docs/extending#authentication).
