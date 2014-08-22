# 队列

- [配置](#configuration)
- [基础用法](#basic-usage)
- [队列闭包](#queueing-closures)
- [运行队列监听器](#running-the-queue-listener)
- [Daemon Queue Worker](#daemon-queue-worker)
- [推送队列](#push-queues)
- [Failed Jobs](#failed-jobs)

<a name="configuration"></a>
## 配置

Laravel的队列组件为许多队列服务提供了统一的API接口。队列服务让你可以异步处理一个耗时任务，比如延迟发送一封邮件，从而大大加快了应用的Web请求处理速度。

队列的设置信息储存在 `app/config/queue.php` 文件中。在这个文件中你可以找到所有目前支持的队列驱动的连接设置，包括[Beanstalkd](http://kr.github.com/beanstalkd)、[IronMQ](http://iron.io)、[Amazon SQS](http://aws.amazon.com/sqs)、[Redis](http://redis.io)和同步处理（本地环境使用）驱动。

下面是相应队列驱动所需的依赖性包：

- Beanstalkd: `pda/pheanstalk`
- Amazon SQS: `aws/aws-sdk-php`
- IronMQ: `iron-io/iron_mq`

<a name="basic-usage"></a>
## 基础用法

#### 推送一个任务到队列中

使用 `Queue::push` 方法推送一个新任务到队列中：

	Queue::push('SendEmail', array('message' => $message));

`push` 方法的第一个参数是用来处理任务的类的名称。第二个参数是一个数组，包含了需要传递给处理器的数据。一个任务处理器应该像这样定义：

	class SendEmail {

		public function fire($job, $data)
		{
			//
		}

	}

注意，类唯一需要的方法是 `fire` 方法，它接受一个 `Job` 实例就像 `data` 数组一样发送到队列。

#### 指定一个自定义处理器方法

如果你希望任务调用 `fire` 以外的方法，你可以在推送任务时指定相应方法：

	Queue::push('SendEmail@send', array('message' => $message));

*Specifying The Queue / Tube For A Job**

You may also specify the queue / tube a job should be sent to:

	Queue::push('SendEmail@send', array('message' => $message), 'emails');

#### Passing The Same Payload To Multiple Jobs

If you need to pass the same data to several queue jobs, you may use the `Queue::bulk` method:

	Queue::bulk(array('SendEmail', 'NotifyUser'), $payload);

#### Delaying The Execution Of A Job

Sometimes you may wish to delay the execution of a queued job. For instance, you may wish to queue a job that sends a customer an e-mail 15 minutes after sign-up. You can accomplish this using the `Queue::later` method:

	$date = Carbon::now()->addMinutes(15);

	Queue::later($date, 'SendEmail@send', array('message' => $message));

In this example, we're using the [Carbon](https://github.com/briannesbitt/Carbon) date library to specify the delay we wish to assign to the job. Alternatively, you may pass the number of seconds you wish to delay as an integer.

#### 删除一个处理完的任务

一旦你处理完了一个任务，必须从队列中将它删除，可以通过 `Job` 实例中的 `delete` 方法完成这项工作：

	public function fire($job, $data)
	{
		// Process the job...

		$job->delete();
	}

#### 将一个任务放回队列

如果你想要将一个任务放回队列，你可以使用 `release` 方法：

	public function fire($job, $data)
	{
		// Process the job...

		$job->release();
	}

你也可以指定几秒后将任务放回队列：

	$job->release(5);

#### 获取尝试处理任务的次数

如果任务在处理时发生异常，它会被自动放回队列。你可以使用 `attempts` 方法获取尝试处理任务的次数：

	if ($job->attempts() > 3)
	{
		//
	}

#### 获取任务ID

你也可以获取任务的ID：

	$job->getJobId();

<a name="queueing-closures"></a>
## 队列闭包

你可以将一个闭包函数推送到队列中。这非常便于快速、简单地处理队列任务：

#### 推送一个闭包函数到队列中

	Queue::push(function($job) use ($id)
	{
		Account::delete($id);

		$job->delete();
	});

当使用 Iron.io [推送队列](#push-queues) 时，你需要特别谨慎地处理队列闭包。接受队列信息的结尾应该带有Iron.io的验证令牌。例如，推送队列的结尾应该类似： `https://yourapp.com/queue/receive?token=SecretToken`。你也许需要在封装队列请求前检查一下应用的秘密令牌的值。

<a name="running-the-queue-listener"></a>
## 运行队列监听器

Laravel包含了一个用于运行已推送到队列的任务的Artisan服务。可以使用 `queue:listen` 命令来运行这个功能：

#### 开启队列监听器

	php artisan queue:listen

你也可以指定队列监听器需要使用的连接：

	php artisan queue:listen connection

注意一旦任务启动，它会一直运行除非你手动停止它。可以使用进程监视工具（例如 [Supervisor](http://supervisord.org/)）来确保队列监听器处于运行状态。

You may pass a comma-delimited list of queue connections to the `listen` command to set queue priorities:

	php artisan queue:listen --queue=high,low

In this example, jobs on the `high-connection` will always be processed before moving onto jobs from the `low-connection`.

#### 设置任务的超时参数

You may also set the length of time (in seconds) each job should be allowed to run:

	php artisan queue:listen --timeout=60

#### Specifying Queue Sleep Duration

另外，你还可以指定新任务轮询之前所需要等待的秒数：

	php artisan queue:listen --sleep=5

Note that the queue only "sleeps" if no jobs are on the queue. If more jobs are available, the queue will continue to work them without sleeping.

如果只想处理队列的第一个任务，你可以使用 `queue:work` 命令：

#### 处理队列的第一个任务

To process only the first job on the queue, you may use the `queue:work` command:

	php artisan queue:work

<a name="daemon-queue-workers"></a>
## Daemon Queue Workers

The `queue:work` also includes a `--daemon` option for forcing the queue worker to continue processing jobs without ever re-booting the framework. This results in a significant reduction of CPU usage when compared to the `queue:listen` command, but at the added complexity of needing to drain the queues of currently executing jobs during your deployments.

To start a queue worker in daemon mode, use the `--daemon` flag:

	php artisan queue:work connection --daemon

	php artisan queue:work connection --daemon --sleep=3

	php artisan queue:work connection --daemon --sleep=3 --tries=3

As you can see, the `queue:work` command supports most of the same options available to `queue:listen`. You may use the `php artisan help queue:work` command to view all of the available options.

### Deploying With Daemon Queue Workers

The simplest way to deploy an application using daemon queue workers is to put the application in maintenance mode at the beginning of your deploymnet. This can be done using the `php artisan down` command. Once the application is in maintenance mode, Laravel will now accept any new jobs off of the queue, but will continue to process existing jobs. Once enough time has passed for all of your existing jobs to execute (usually no longer than 30-60 seconds), you may stop the worker and continue your deployment process.

If you are using Supervisor or Laravel Forge, which utilizes Supervisor, you may typically stop a worker with a command like the following:

	supervisorctl stop worker-1

Once the queues have been drained and your fresh code has been deployed to your server, you should restart the daemon queue work. If you are using Supervisor, this can typically be done with a command like this:

	supervisorctl start worker-1

<a name="push-queues"></a>
## 推送队列

推送队列可以让你在没有守护进程和后台监听器的情况下使用 Laravel 4 强大的队列工具。当前，推送队列仅支持[Iron.io](http://iron.io)驱动。在开始前，创建一个 Iron.io 账户，然后将Iron的认证信息填入到 `app/config/queue.php` 配置文件中。

#### 注册一个推送队列订阅

接下来，你可以使用 `queue:subscribe` Artisan命令注册一个用来接收新的推送队列任务的URL结尾。

	php artisan queue:subscribe queue_name http://foo.com/queue/receive

现在，当你登录到Iron后台，你可以看见新的推送队列和订阅的URL。你可以为一个指定的队列订阅多个URL。接下来，为 `queue/receive` 结尾的URL创建一个路由，并且返回来自 `Queue::marshal` 方法的响应：

	Route::post('queue/receive', function()
	{
		return Queue::marshal();
	});

`marshal` 方法会自动执行相应的任务处理器类。想要处理推送队列上的任务，可以像处理一般的队列一样使用 `Queue::push` 方法。

<a name="failed-jobs"></a>
## Failed Jobs

Since things don't always go as planned, sometimes your queued jobs will fail. Don't worry, it happens to the best of us! Laravel includes a convenient way to specify the maximum number of times a job should be attempted. After a job has exceeded this amount of attempts, it will be inserted into a `failed_jobs` table. The failed jobs table name can be configured via the `app/config/queue.php` configuration file.

To create a migration for the `failed_jobs` table, you may use the `queue:failed-table` command:

	php artisan queue:failed-table

You can specify the maximum number of times a job should be attempted using the `--tries` switch on the `queue:listen` command:

	php artisan queue:listen connection-name --tries=3

If you would like to register an event that will be called when a queue job fails, you may use the `Queue::failing` method. This event is a great opportunity to notify your team via e-mail or [HipChat](https://www.hipchat.com).

	Queue::failing(function($connection, $job, $data)
	{
		//
	});

To view all of your failed jobs, you may use the `queue:failed` Artisan command:

	php artisan queue:failed

The `queue:failed` command will list the job ID, connection, queue, and failure time. The job ID may be used to retry the failed job. For instance, to retry a failed job that has an ID of 5, the following command should be issued:

	php artisan queue:retry 5

If you would like to delete a failed job, you may use the `queue:forget` command:

	php artisan queue:forget 5

To delete all of your failed jobs, you may use the `queue:flush` command:

	php artisan queue:flush