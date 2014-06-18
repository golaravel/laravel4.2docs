# 版本说明

- [Laravel 4.2](#laravel-4.2)
- [Laravel 4.1](#laravel-4.1)

<a name="laravel-4.2"></a>
## Laravel 4.2

在一个4.2版本的安装目录下，通过运行命令“php artisan changes”来获得此版本完整的变更列表, 也可以查看Github上的变更文件(https://github.com/laravel/framework/blob/4.2/src/Illuminate/Foundation/changes.json)。 These notes only cover the major enhancements and changes for the release.

> **注意:** 在4.2版本的发布周期中, 很多小的bug修复和功能改进被合并进了Laravel 4.1的各个发布版本中。 所以，也一定要检查Laravel 4.1的变更列表！

### PHP 5.4 为最低版本要求

Laravel 4.2 要求 PHP 5.4 或更高版本. 这种PHP版本的升级需求使得我们能够使用PHP的新特性，例如，为[Laravel Cashier](/docs/billing)等工具提供的更具表现力的接口。 与PHP 5.3相比，PHP 5.4 在速度和执行效率上也有显著的提高。

### Laravel Forge

Laravel Forge, 是一个新的并基于web的应用程序，它提供一种简单的方式来创建和管理你自己云端的PHP服务，这些服务包括Linode、 DigitalOcean、 Rackspace　和 Amazon EC2。 支持自动化配置Nginx， 访问SSH key, Cron job 自动化, 通过 NewRelic 或 Papertrail进行的服务监控, "Push To Deploy", 配置Laravel队列工人, 还有, Forge提供最简单且最经济的方式来运行你的所有Laravel应用程序。

默认情况下，Laravel 4.2安装目录下的文件`app/config/database.php`是用来配置Forge的, 使得新的应用能更方便的部署到这个平台上来。

在Forge的官方网站(https://forge.laravel.com)上，可以找到更多的关于Laravel Forge的信息。

### Laravel Homestead

Laravel Homestead是一个官方的Vagrant环境，它被用来开发强健的Laravel和PHP应用程序。在箱子被打包派发之前，箱子供应需求的绝大部分会被处理，允许箱子极其快速的启动。 Homestead包括Nginx 1.6、PHP 5.5.12、MySQL、Postgres、Redis、Memcached、Beanstalk、Node、Gulp、Grunt和Bower。Homestaed包含一个简单的配置文件“Homestead.yaml”，它能在单个箱子里管理多个Laravel应用程序.

现在，默认的Laravel 4.2安装目录包含一个配置文件“app/config/local/database.php”，它是用来配置使用箱子以外的Homestead数据库的, 这使得Laravel初始化安装目录和配置更加方便.

官方文档已经包括Homestead文档(/docs/homestead)。

### Laravel 出纳

Laravel 出纳是一个简单的、富于表现力的库，它用来管理条码的订阅计费。随着Laravel 4.2的引入，尽管安装组件本身仍然是可选的，但是我们在Laravel主文档里包含了Cashier文档。 这个版本的Cashier修复了很多bug, 支持多币种, 并兼容最新的条码API。

### 守护进程队列工人

Artisan的“queue:work”命令现在支持“--daemon”选项，它用来启动一个“守护进程模式”的工人， 这个模式使得工人在不需要重启框架的情况下继续工作。 这会显著的减少CPU的使用率，其代价只是使得应用程序的部署过程稍显复杂。

在队列文档（/docs/queue#daemon-queue-workers）里能找到更多的队列工人相关的信息。

### 邮件接口驱动程序

Laravel 4.2为“邮件”功能引入了新的Mailgun和Mandrill接口驱动程序。对很多应用程序来说，比起SMTP方式，此接口提供了一种更快和更可靠发送电子邮件的方法。新的驱动程序采用Guzzle 4 HTTP库。

### 软删除特性

一种为“软删除”和其它“全局范围”而生的更干净的架构通过PHP 5.4的特性被引入进来。这种新架构考虑到了更简单的构建全局特性等功能， 和一个更干净的框架本身的关注点分离。

在文档（/docs/eloquent#soft-deleting）里能找到更多的关于“软删除特性”的信息。

### 方便的认证 和 提醒特性

The default Laravel 4.2 installation now uses simple traits for including the needed properties for the authentication and password reminder user interfaces. This provides a much cleaner default `User` model file out of the box.

### "Simple Paginate"

A new `simplePaginate` method was added to the query and Eloquent builder which allows for more efficient queries when using simple "Next" and "Previous" links in your pagination view.

### Migration Confirmation

In production, destructive migration operations will now ask for confirmation. Commands may be forced to run without any prompts using the `--force` command.

<a name="laravel-4.1"></a>
## Laravel 4.1

### 全部变更列表

The full change list for this release by running the `php artisan changes` command from a 4.1 installation, or by [viewing the change file on Github](https://github.com/laravel/framework/blob/4.1/src/Illuminate/Foundation/changes.json). These notes only cover the major enhancements and changes for the release.

### New SSH Component

An entirely new `SSH` component has been introduced with this release. This feature allows you to easily SSH into remote servers and run commands. To learn more, consult the [SSH component documentation](/docs/ssh).

The new `php artisan tail` command utilizes the new SSH component. For more information, consult the `tail` [command documentation](http://laravel.com/docs/ssh#tailing-remote-logs).

### Boris In Tinker

The `php artisan tinker` command now utilizes the [Boris REPL](https://github.com/d11wtq/boris) if your system supports it. The `readline` and `pcntl` PHP extensions must be installed to use this feature. If you do not have these extensions, the shell from 4.0 will be used.

### Eloquent Improvements

A new `hasManyThrough` relationship has been added to Eloquent. To learn how to use it, consult the [Eloquent documentation](/docs/eloquent#has-many-through).

A new `whereHas` method has also been introduced to allow [retrieving models based on relationship constraints](/docs/eloquent#querying-relations).

### Database Read / Write Connections

Automatic handling of separate read / write connections is now available throughout the database layer, including the query builder and Eloquent. For more information, consult [the documentation](/docs/database#read-write-connections).

### Queue Priority

Queue priorities are now supported by passing a comma-delimited list to the `queue:listen` command.

### Failed Queue Job Handling

The queue facilities now include automatic handling of failed jobs when using the new `--tries` switch on `queue:listen`. More information on handling failed jobs can be found in the [queue documentation](/docs/queues#failed-jobs).

### Cache Tags

Cache "sections" have been superseded by "tags". Cache tags allow you to assign multiple "tags" to a cache item, and flush all items assigned to a single tag. More information on using cache tags may be found in the [cache documentation](/docs/cache#cache-tags).

### Flexible Password Reminders

The password reminder engine has been changed to provide greater developer flexibility when validating passwords, flashing status messages to the session, etc. For more information on using the enhanced password reminder engine, [consult the documentation](/docs/security#password-reminders-and-reset).

### Improved Routing Engine

Laravel 4.1 features a totally re-written routing layer. The API is the same; however, registering routes is a full 100% faster compared to 4.0. The entire engine has been greatly simplified, and the dependency on Symfony Routing has been minimized to the compiling of route expressions.

### Improved Session Engine

With this release, we're also introducing an entirely new session engine. Similar to the routing improvements, the new session layer is leaner and faster. We are no longer using Symfony's (and therefore PHP's) session handling facilities, and are using a custom solution that is simpler and easier to maintain.

### Doctrine DBAL

If you are using the `renameColumn` function in your migrations, you will need to add the `doctrine/dbal` dependency to your `composer.json` file. This package is no longer included in Laravel by default.