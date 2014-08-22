# 单元测试

- [简介](#introduction)
- [定义 & 运行测试](#defining-and-running-tests)
- [测试环境](#test-environment)
- [测试中执行路由](#calling-routes-from-tests)
- [模拟Facades](#mocking-facades)
- [框架断言](#framework-assertions)
- [辅助函数](#helper-methods)
- [Refreshing The Application](#refreshing-the-application)

<a name="introduction"></a>
## 简介

Laravel 自建了单元测试。事实上，我们支持使用自带的 PHPUnit 进行测试，并且已经为你的应用程序配置好了 `phpunit.xml` 文件。除了 PHPUnit，Laravel 也使用了 Symfony HttpKernel，DomCrawler 和 BrowserKit 组件，允许你在测试中检查和操作视图以及模拟浏览器行为。

在 `app/tests` 目录下已经提供了一个测试示例文件。在安装了一个全新的 Laravel 应用程序之后，只需要简单的在命令行中执行 `phpunit`， 即可运行你的测试。


<a name="defining-and-running-tests"></a>
## 定义 & 运行测试

要创建一个测试用例，只需要简单的在 `app/tests` 目录创建一个新的测试文件。测试类需要继承 `TestCase` 。然后你可以像使用 PHPUnit 一样来定义这些测试函数。

#### 一个测试类示例

	class FooTest extends TestCase {

		public function testSomethingIsTrue()
		{
			$this->assertTrue(true);
		}

	}

你可以在终端执行 `phpunit` 命令来运行应用程序中的所有测试。

> **注意:** 如果你定义了自己的 `setUp` 函数，请确保调用了 `parent::setUp `。

<a name="test-environment"></a>
## 测试环境

在运行单元测试时，Laravel 将自动设置环境配置为 `testing` 。并且，Laravel 在测试环境中包含了 `session` 和 `cache` 配置文件。测试环境中，这两个驱动都被设置为空 `array`，意味着在测试过程中，不会将 session 或者 cache 持久化。必要时，你也可以创建其他的测试环境。

<a name="calling-routes-from-tests"></a>
## 测试中执行路由

#### 测试中执行路由

在测试时可以很简单的使用 `call` 函数来执行一个路由：

	$response = $this->call('GET', 'user/profile');

	$response = $this->call($method, $uri, $parameters, $files, $server, $content);

你可以检查 `Illuminate\Http\Response` 对象:

	$this->assertEquals('Hello World', $response->getContent());
    
#### 测试中执行控制器

你也可以在测试中执行控制器：

	$response = $this->action('GET', 'HomeController@index');

	$response = $this->action('GET', 'UserController@profile', array('user' => 1));

`getContent` 函数将会返回和响应相同的字符串内容。如果路由返回 `视图（View）` ， 你可以使用 `original` 属性来访问：

	$view = $response->original;

	$this->assertEquals('John', $view['name']);

要执行一个 HTTPS 路由，可以使用 `callSecure` 函数：

	$response = $this->callSecure('GET', 'foo/bar');

> **注意：** 在测试环境中，路由过滤器将被禁用。如果需要启用它们, 请在你的测试中添加 `Route::enableFilters()` 。

### DOM Crawler

你也可以执行一个路由并且接受一个 DOM Crawler 实例来检查内容：

	$crawler = $this->client->request('GET', '/');

	$this->assertTrue($this->client->getResponse()->isOk());

	$this->assertCount(1, $crawler->filter('h1:contains("Hello World!")'));

如需获取更多关于 DOM Crawler 的信息，请参见文档 [official documentation](http://symfony.com/doc/master/components/dom_crawler.html)。

<a name="mocking-facades"></a>
## 模拟Facades

在测试时，你可能经常需要模拟执行一个 Laravel 静态 facade。例如，参考如下控制器的动作：

	public function getIndex()
	{
		Event::fire('foo', array('name' => 'Dayle'));

		return 'All done!';
	}

我们可以通过在 facade 使用 `shouldReceive` 函数来模拟执行 `Event` 类, 它将返回一个 [Mockery](https://github.com/padraic/mockery) 模拟的实例。



#### 模拟一个Facade

	public function testGetIndex()
	{
		Event::shouldReceive('fire')->once()->with('foo', array('name' => 'Dayle'));

		$this->call('GET', '/');
	}

> **注意：** 你不能模拟 `Request` 的 facade。替代的方式为，在运行你的测试时，将你预期的输入传给 `call` 函数。

<a name="framework-assertions"></a>
## 框架断言

Laravel 提供了一些 `断言（assert）` 函数来让测试更加容易：

#### 断言响应为OK

	public function testMethod()
	{
		$this->call('GET', '/');

		$this->assertResponseOk();
	}

#### 断言响应状态码

	$this->assertResponseStatus(403);

#### 断言响应为重定向

	$this->assertRedirectedTo('foo');

	$this->assertRedirectedToRoute('route.name');

	$this->assertRedirectedToAction('Controller@method');

#### 断言视图含有的一些数据

	public function testMethod()
	{
		$this->call('GET', '/');

		$this->assertViewHas('name');
		$this->assertViewHas('age', $value);
	}

#### 断言 Session 中含有的一些数据

	public function testMethod()
	{
		$this->call('GET', '/');

		$this->assertSessionHas('name');
		$this->assertSessionHas('age', $value);
	}

#### Asserting The Session Has Errors

    public function testMethod()
    {
        $this->call('GET', '/');

        $this->assertSessionHasErrors();

        // Asserting the session has errors for a given key...
        $this->assertSessionHasErrors('name');

        // Asserting the session has errors for several keys...
        $this->assertSessionHasErrors(array('name', 'age'));
    }

#### Asserting Old Input Has Some Data

	public function testMethod()
	{
		$this->call('GET', '/');

		$this->assertHasOldInput();
	}

<a name="helper-methods"></a>
## 辅助函数

`TestCase` 类包含一些辅助函数来让应用程序测试更加容易。

#### Setting And Flushing Sessions From Tests

	$this->session(['foo' => 'bar']);

	$this->flushSession();

#### 设置当前通过验证的用户

你可以使用 `be` 函数来设置当前通过验证的用户：

	$user = new User(array('name' => 'John'));

	$this->be($user);

你可以在测试中使用 `seed` 函数来重新填充你的数据库：

#### 测试中重新填充数据库

	$this->seed();

	$this->seed($connection);

如需获取更多关于创建数据填充的信息，请参见文档 [迁移 & 数据填充](/docs/migrations#database-seeding)。

<a name="refreshing-the-application"></a>
## Refreshing The Application

As you may already know, you can access your Laravel `Application` / IoC Container via `$this->app` from any test method. This Application instance is refreshed for each test class. If you wish to manually force the Application to be refreshed for a given method, you may use the `refreshApplication` method from your test method. This will reset any extra bindings, such as mocks, that have been placed in the IoC container since the test case started running.
