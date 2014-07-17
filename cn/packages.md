# Package Development

- [简介](#简介)
- [创建包](#创建包)
- [包结构](#包结构)
- [Service Providers](#service-providers)
- [Deferred Providers](#deferred-providers)
- [Package Conventions](#package-conventions)
- [Development Workflow](#development-workflow)
- [Package Routing](#package-routing)
- [Package Configuration](#package-configuration)
- [Package Views](#package-views)
- [Package Migrations](#package-migrations)
- [Package Assets](#package-assets)
- [Publishing Packages](#publishing-packages)

<a name="introduction"></a>
## 简介

Packages are the primary way of adding functionality to Laravel. Packages might be anything from a great way to work with dates like [Carbon](https://github.com/briannesbitt/Carbon), or an entire BDD testing framework like [Behat](https://github.com/Behat/Behat).

包是向Laravel中添加功能最重要的途径。它通过一种强大的方式几乎可以包含任意功能，比如处理日期的扩展包[Carbon](https://github.com/briannesbitt/Carbon)，完整的[BDD](http://baike.baidu.com/view/1384794.htm)测试框架扩展包[Behat](https://github.com/Behat/Behat).

Of course, there are different types of packages. Some packages are stand-alone, meaning they work with any framework, not just Laravel. Both Carbon and Behat are examples of stand-alone packages. Any of these packages may be used with Laravel by simply requesting them in your `composer.json` file.

当然，还有很多不同类型的包。有些包是独立的，这意味着它们可以在任何框架中工作，而不仅仅是Laravel。上面提到的Carbon和Behat就是独立的包。要在Laravel中使用这些包只需要在`composer.json`文件中指明。

On the other hand, other packages are specifically intended for use with Laravel. In previous versions of Laravel, these types of packages were called "bundles". These packages may have routes, controllers, views, configuration, and migrations specifically intended to enhance a Laravel application. As no special process is needed to develop stand-alone packages, this guide primarily covers the development of those that are Laravel specific.

另一方面，有些包仅支持Laravel。在上一个Laravel版本中，这些类型的包我们称为"bundles"。这些包可以增强Laravel应用的路由、控制器、视图、配置和迁移。由于开发独立的包不需要专门的过程，因此，本手册主要涵盖针对Laravel开发独立的包。

All Laravel packages are distributed via [Packagist](http://packagist.org) and [Composer](http://getcomposer.org), so learning about these wonderful PHP package distribution tools is essential.

所有Laravel包都是通过[Packagist](http://packagist.org)和[Composer](http://getcomposer.org)发布的，因此很有必要学习这些功能强大的PHP包发布管理工具。

<a name="creating-a-package"></a>
## 创建包

The easiest way to create a new package for use with Laravel is the `workbench` Artisan command. First, you will need to set a few options in the `app/config/workbench.php` file. In that file, you will find a `name` and `email` option. These values will be used to generate a `composer.json` file for your new package. Once you have supplied those values, you are ready to build a workbench package!

为Laravel创建一个包的最简单方式是使用Artisan的`workbench`命令。首先，你需要在`app/confg/workbench.php`文件中配置一些参数。在该文件中，你会看到`name`和`email`两个参数，这些值是用来为您新创建的包生成`composer.json`文件的。一旦你提供了这些值，就可以开始构建一个新包了！

#### 执行Artisan的workbench命令

	php artisan workbench vendor/package --resources

The vendor name is a way to distinguish your package from other packages of the same name from different authors. For example, if I (Taylor Otwell) were to create a new package named "Zapper", the vendor name could be `Taylor` while the package name would be `Zapper`. By default, the workbench will create framework agnostic packages; however, the `resources` command tells the workbench to generate the package with Laravel specific directories such as `migrations`, `views`, `config`, etc.

厂商名称（vendor name）是用来区分不同作者构建了相同名字的包。例如，如果我（Taylor Otwell）创建了个名为"Zapper"的包，厂商名就可以叫做`Taylor`，包名可以叫做`Zapper`。默认情况下，workbench命令建的包是不依赖任何框架的；不过，`resources`命令将会告诉workbench生成关联Laravel的一些特殊目录，例如migrations、views、config等。

Once the `workbench` command has been executed, your package will be available within the `workbench` directory of your Laravel installation. Next, you should register the `ServiceProvider` that was created for your package. You may register the provider by adding it to the `providers` array in the `app/config/app.php` file. This will instruct Laravel to load your package when your application starts. Service providers use a `[Package]ServiceProvider` naming convention. So, using the example above, you would add `Taylor\Zapper\ZapperServiceProvider` to the `providers` array.

一旦执行了workbench命令，新创建的包就会出现在Laravel安装目录下的workbench目录中。接下来就应该为你创建的包注册`ServiceProvider`了。你可以通过在`app/config/app.php`文件里的`provides`数组中添加该包。这将通知Laravel在应用程序开始启动时加载该包。服务提供者（Service providers）使用`[Package]ServiceProvider`样式的命名方式。所以，以上案例中，你需要将`Taylor\Zapper\ZapperServiceProvider`添加到providers数组。

Once the provider has been registered, you are ready to start developing your package! However, before diving in, you may wish to review the sections below to get more familiar with the package structure and development workflow.

一旦注册了provider，你就可以开始写代码了！然而，在此之前，建议你查看以下部分来了解更多关于包结构和开发流程的知识。

> **Note:** If your service provider cannot be found, run the `php artisan dump-autoload` command from your application's root directory

> **注意:** 如果你的Service providers提示无法找到, 在项目根目录执行`php artisan dump-autoload` 

<a name="package-structure"></a>
## 包结构

When using the `workbench` command, your package will be setup with conventions that allow the package to integrate well with other parts of the Laravel framework:

执行`workbench`命令之后，你的包结构将被初始化，并能够与laravel框架完美融合：


#### 包的基本目录结构

	/src
		/Vendor
			/Package
				PackageServiceProvider.php
		/config
		/lang
		/migrations
		/views
	/tests
	/public

Let's explore this structure further. The `src/Vendor/Package` directory is the home of all of your package's classes, including the `ServiceProvider`. The `config`, `lang`, `migrations`, and `views` directories, as you might guess, contain the corresponding resources for your package. Packages may have any of these resources, just like "regular" applications.

让我们来深入了解该结构。`src/Vendor/Package`目录是所有`class`的主目录，包括`ServiceProvider`。`config`、`lang`、`migrations`和`views`目录，就如你所猜测，包含了你创建包种相应的资源。包可以包含这些资源中的任意几个，就像一个"regular"的应用。

<a name="service-providers"></a>
## 服务提供器

Service providers are simply bootstrap classes for packages. By default, they contain two methods: `boot` and `register`. Within these methods you may do anything you like: include a routes file, register bindings in the IoC container, attach to events, or anything else you wish to do.

服务提供器只是包的引导类。默认情况下，他们包含两个方法：`boot`和`register`。你可以在这些方法内做任何事情，例如：包含路由文件、注册IoC容器的绑定、监听事件或者任何想做的事情。

The `register` method is called immediately when the service provider is registered, while the `boot` command is only called right before a request is routed. So, if actions in your service provider rely on another service provider already being registered, or you are overriding services bound by another provider, you should use the `boot` method.

`register`方法在服务提供器注册时被立即调用，而boot方法仅在请求被路由到之前调用。因此，如果服务提供器中的动作（action）依赖另一个已经注册的服务提供器，或者你正在覆盖另一个服务提供其绑定的服务，就应该使用boot方法。

When creating a package using the `workbench`, the `boot` command will already contain one action:

当使用`workbench`命令创建包时，`boot`方法已经包含了如下的动作：

	$this->package('vendor/package');

This method allows Laravel to know how to properly load the views, configuration, and other resources for your application. In general, there should be no need for you to change this line of code, as it will setup the package using the workbench conventions.

该方法告诉Laravel如何为应用程序加载视图、配置或其他资源。通常情况下，你没有必要改变这行代码，因为它会根据workbench的默认约定将包设置好的。

By default, after registering a package, its resources will be available using the "package" half of `vendor/package`. However, you may pass a second argument into the `package` method to override this behavior. For example:

默认情况下，一旦注册了一个包，那么它的资源可以通过"package"方法在`vendor/package`中找到。你也可以向`package` 方法中传入第二个参数来重写这个方法。例如

	//向 `package` 方法中传入一个自定义的命名空间
	$this->package('vendor/package', 'custom-namespace');

	//现在，这个包的资源现在可以通过这个自定义的命名空间来访问
	$view = View::make('custom-namespace::foo');

There is not a "default location" for service provider classes. You may put them anywhere you like, perhaps organizing them in a `Providers` namespace within your `app` directory. The file may be placed anywhere, as long as Composer's [auto-loading facilities](http://getcomposer.org/doc/01-basic-usage.md#autoloading) know how to load the class.

Laravel并没有为service provider提供“默认”的存放地点。您可以根据自己的喜好，将它们放置在任何地方，您也可以将它们统一组织在一个`Providers`命名空间里，并放置在应用的`app`目录下。这些文件可以被放置在任何地方，只需保证Composer的[自动加载](http://getcomposer.org/doc/01-basic-usage.md#autoloading)组件知道如何加载这些类。

If you have changed the location of your package's resources, such as configuration files or views, you should pass a third argument to the `package` method which specifies the location of your resources:

如果你改变了你的包得资源的位置，比如配置文件或者视图，你需要在`package`函数中传递第三个参数，指定你的资源位置

	$this->package('vendor/package', null, '/path/to/resources');

<a name="deferred-providers"></a>
## Deferred Providers
## 延迟加载服务提供器

If you are writing a service provider that does not register any resources such as configuration or views, you may choose to make your provider "deferred". A deferred service provider is only loaded and registered when one of the services it provides is actually needed by the application IoC container. If none of the provider's services are needed for a given request cycle, the provider is never loaded.

如果你写了一个服务提供器，但没有注册任何资源，比如配置文件或视图等，你可以选择"延迟"加载你的提供器。延迟加载的服务提供器在这个服务真正被IoC容器需要的时候才会被加载和注册。如果这个服务提供器没有被当前的请求路由需要，那么这个提供器永远不会被加载

To defer the execution of your service provider, set the `defer` property on the provider to `true`:

如果想延时载入你的服务提供器，只需要在提供器重设置`defer`属性为`true`:

	protected $defer = true;

Next you should override the `provides` method from the base `Illuminate\Support\ServiceProvider` class and return an array of all of the bindings that your provider adds to the IoC container. For example, if your provider registers `package.service` and `package.another-service` in the IoC container, your `provides` method should look like this:

接下来你需要重写继承自基类`Illuminate\Support\ServiceProvider`的`provides`方法，并返回绑定了你的提供器的IoC容器中类型的数组集合。例如，你的提供器在IoC容器注册了`package.service` 和 `package.another-service`两个类型，你的`provides`的方法看起来应该是这个样子：

	public function provides()
	{
		return array('package.service', 'package.another-service');
	}

<a name="package-conventions"></a>
## 包约定

When utilizing resources from a package, such as configuration items or views, a double-colon syntax will generally be used:

要使用包中的资源，例如配置或视图，需要用双冒号语法：

#### 从包中载入视图

	return View::make('package::view.name');

#### 获取包的某个配置项

	return Config::get('package::group.option');

> **Note:** If your package contains migrations, consider prefixing the migration name with your package name to avoid potential class name conflicts with other packages.

> **注意:** 如果你包中包含迁移，请为迁移名（migration name）添加包名作为前缀，以避免与其他包中的类名冲突。


<a name="development-workflow"></a>
## 开发流程

When developing a package, it is useful to be able to develop within the context of an application, allowing you to easily view and experiment with your templates, etc. So, to get started, install a fresh copy of the Laravel framework, then use the `workbench` command to create your package structure.

当开发一个包时，能够使用应用程序上文是相当有用的，这样将允许你很容易的解决视图模板的等问题。所以，我们开始，安装一个全新的Laravel框架，使用`workbench`命令创建包结构。

After the `workbench` command has created your package. You may `git init` from the `workbench/[vendor]/[package]` directory and `git push` your package straight from the workbench! This will allow you to conveniently develop the package in an application context without being bogged down by constant `composer update` commands.

在使用`workbench`命令创建包后。你可以在`workbench/[vendor]/[package]`目录使用`git init`，并在workbench中直接`git push`！这将允许你在应用程序上下文中方便开发而不用为反复使用`composer update`命令苦扰。

Since your packages are in the `workbench` directory, you may be wondering how Composer knows to autoload your package's files. When the `workbench` directory exists, Laravel will intelligently scan it for packages, loading their Composer autoload files when the application starts!

当包存放在`workbench`目录时，你可能担心Composer如何知道自动加载包文件。当workbench目录存在，Laravel将智能扫描该目录，在应用程序开始时加载它们的Composer自动加载文件！

If you need to regenerate your package's autoload files, you may use the `php artisan dump-autoload` command. This command will regenerate the autoload files for your root project, as well as any workbenches you have created.

如果你需要重新生成报的自动加载文件，你可以使用`php artisan dump-autoload`命令，这个命令将会重新为您的整个项目生成自动加载文件，也包括你创建的所有工作台(workbenches)

#### 运行Artisan的自动加载命令

	php artisan dump-autoload

<a name="package-routing"></a>
## 包路由

In prior versions of Laravel, a `handles` clause was used to specify which URIs a package could respond to. However, in Laravel 4, a package may respond to any URI. To load a routes file for your package, simply `include` it from within your service provider's `boot` method.

在之前的Laravel版本中，`handlers`用来指定那个URI包会响应。然而，在Laravel4中，一个包可以相应任意URI。要在包中加载路由文件，只需在服务提供器的`boot`方法`include`它。

#### 在服务提供器中包含路由文件

	public function boot()
	{
		$this->package('vendor/package');

		include __DIR__.'/../../routes.php';
	}

> **Note:** If your package is using controllers, you will need to make sure they are properly configured in your `composer.json` file's auto-load section.

> **注意:** 如果你的包中使用了控制器, 你需要确保正确配置了`composer.json`文件的auto-load字段.

<a name="package-configuration"></a>
## 包配置

#### 访问包配置文件

Some packages may require configuration files. These files should be defined in the same way as typical application configuration files. And, when using the default `$this->package` method of registering resources in your service provider, may be accessed using the usual "double-colon" syntax:

有时创建的包可能会需要配置文件。这些配置文件应该和应用程序配置文件相同方法定义。并且，当使用 `$this->package`方法来注册服务提供器时，那么就可以使用“双冒号”语法来访问：

	Config::get('package::file.option');

#### 访问包单一配置文件

However, if your package contains a single configuration file, you may simply name the file `config.php`. When this is done, you may access the options directly, without specifying the file name:

然而，如果你包仅有一个配置文件，你可以简单命名为`config.php`。当你这么做时，你可以直接访问该配置项，而不需要特别指明文件名：

	Config::get('package::option');

#### Registering A Resource Namespace Manually
#### 手动注册资源的命名空间

Sometimes, you may wish to register package resources such as views outside of the typical `$this->package` method. Typically, this would only be done if the resources were not in a conventional location. To register the resources manually, you may use the `addNamespace` method of the `View`, `Lang`, and `Config` classes:

有时候，你可能希望在`$this->package`特有方法之外注册包的资源，比如视图。通常只有在这些资源不在约定的位置的时候才应该这么做。如果需要手动注册资源，你可以使用`View`, `Lang`, 和 `Config` 中的`addNamespace`方法实现:

	View::addNamespace('package', __DIR__.'/path/to/views');

Once the namespace has been registered, you may use the namespace name and the "double colon" syntax to access the resources:

一旦这个资源命名空间被注册，你就可以使用这个空间名，并使用"双冒号"语法去访问这个资源

	return View::make('package::view.name');

The method signature for `addNamespace` is identical on the `View`, `Lang`, and `Config` classes.

`View`, `Lang`, 和 `Config`类中的`addNamespace`方法的参数都是一样的

### 级联配置文件

When other developers install your package, they may wish to override some of the configuration options. However, if they change the values in your package source code, they will be overwritten the next time Composer updates the package. Instead, the `config:publish` artisan command should be used:

	php artisan config:publish vendor/package

When this command is executed, the configuration files for your application will be copied to `app/config/packages/vendor/package` where they can be safely modified by the developer!

> **Note:** The developer may also create environment specific configuration files for your package by placing them in `app/config/packages/vendor/package/environment`.

<a name="package-views"></a>
## Package Views

If you are using a package in your application, you may occasionally wish to customize the package's views. You can easily export the package views to your own `app/views` directory using the `view:publish` Artisan command:

	php artisan view:publish vendor/package

This command will move the package's views into the `app/views/packages` directory. If this directory doesn't already exist, it will be created when you run the command. Once the views have been published, you may tweak them to your liking! The exported views will automatically take precedence over the package's own view files.

<a name="package-migrations"></a>
## Package Migrations

#### Creating Migrations For Workbench Packages

You may easily create and run migrations for any of your packages. To create a migration for a package in the workbench, use the `--bench` option:

	php artisan migrate:make create_users_table --bench="vendor/package"

#### Running Migrations For Workbench Packages

	php artisan migrate --bench="vendor/package"

#### Running Migrations For An Installed Package

To run migrations for a finished package that was installed via Composer into the `vendor` directory, you may use the `--package` directive:

	php artisan migrate --package="vendor/package"

<a name="package-assets"></a>
## Package Assets

#### Moving Package Assets To Public

Some packages may have assets such as JavaScript, CSS, and images. However, we are unable to link to assets in the `vendor` or `workbench` directories, so we need a way to move these assets into the `public` directory of our application. The `asset:publish` command will take care of this for you:

	php artisan asset:publish

	php artisan asset:publish vendor/package

If the package is still in the `workbench`, use the `--bench` directive:

	php artisan asset:publish --bench="vendor/package"

This command will move the assets into the `public/packages` directory according to the vendor and package name. So, a package named `userscape/kudos` would have its assets moved to `public/packages/userscape/kudos`. Using this asset publishing convention allows you to safely code asset paths in your package's views.

<a name="publishing-packages"></a>
## Publishing Packages

When your package is ready to publish, you should submit the package to the [Packagist](http://packagist.org) repository. If the package is specific to Laravel, consider adding a `laravel` tag to your package's `composer.json` file.

Also, it is courteous and helpful to tag your releases so that developers can depend on stable versions when requesting your package in their `composer.json` files. If a stable version is not ready, consider using the `branch-alias` Composer directive.

Once your package has been published, feel free to continue developing it within the application context created by `workbench`. This is a great way to continue to conveniently develop the package even after it has been published.

Some organizations choose to host their own private repository of packages for their own developers. If you are interested in doing this, review the documentation for the [Satis](http://github.com/composer/satis) project provided by the Composer team.
