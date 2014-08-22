# Artisan 开发

- [简介](#introduction)
- [构建命令](#building-a-command)
- [注册命令](#registering-commands)
- [调用其他命令](#calling-other-commands)

<a name="introduction"></a>
## 简介

除了 Artisan 已经提供的命令，你也可以为应用程序构建自己的命令。你可以将自己定制的命令保存在 `app/commands` 目录下，同样，你也可以选择任意的存储目录，只要你的命令能够按照 `composer.json` 中的设置被自动加载即可。

<a name="building-a-command"></a>
## 构建命令

### 生成类

为了创建一个新命令，你可以使用Artisan中的 `command:make` 命令生成一个骨架作为你的起点：

#### 生成一个命令类

	php artisan command:make FooCommand

默认情况下，生成的类文件被存放在 `app/commands` 目录下，同时你也可以指定自定义目录和命名空间：

	php artisan command:make FooCommand --path=app/classes --namespace=Classes

当创建命令时，可以使用 `--command` 选项来指定终端命令的名称：

	php artisan command:make AssignUsers --command=users:assign

### 实现命令

一旦命令被生成，你应当填写这个类的 `name` 和 `description` 属性，这些属性将在展示命令的 `list` 页面显示出来。

当命令运行的时候 `fire` 函数将被调用。你可以在这个函数里实现任何业务逻辑。

### 参数 & 选项

可以通过 `getArguments` 和 `getOptions` 函数为命令定义任何接受的参数和选项。这两个函数都将返回一个数组。

当定义 `arguments`，数组定义值表示如下：

	array($name, $mode, $description, $defaultValue)

参数 `mode` 的值可以是 `InputArgument::REQUIRED`, `InputArgument::OPTIONAL` 中的任意一个。

当定义 `options`，数组定义值表示如下：

	array($name, $shortcut, $mode, $description, $defaultValue)

对于选项，参数 `mode` 的值可以是`InputOption::VALUE_REQUIRED`, `InputOption::VALUE_OPTIONAL`, `InputOption::VALUE_IS_ARRAY`, `InputOption::VALUE_NONE` 中的一个。

`VALUE_IS_ARRAY` 模式表明，该开关可用于多次调用该命令时：

	php artisan foo --option=bar --option=baz

`VALUE_NONE` 选项表明该选项值只能用来作为一个开关：

	php artisan foo --option

### 获取输入

当命令执行的时候，你显然需要获取该命令所接收的参数和选项。要做到这一点，你可以使用 `argument` 和 `option` 函数：

#### 获取一个参数的值

	$value = $this->argument('name');

#### 获取所有参数

	$arguments = $this->argument();

#### 获取一个选项的值

	$value = $this->option('name');

获取所有选项

	$options = $this->option();

### 输出

你可以使用`info`、`comment`、`question` 和 `error`方法将输出发送到控制台。这些函数中的每一个将根据它们的目的使用合适的 ANSI 颜色。

#### 发送信息到终端

	$this->info('Display this on the screen');

#### 发送错误消息到终端

	$this->error('Something went wrong!');

### 询问输入

你可以使用 `ask` 和 `confirm` 函数提示用户输入：

#### 询问用户输入

	$name = $this->ask('What is your name?');

#### 询问用户输入密码

	$password = $this->secret('What is the password?');

#### 询问用户确认

	if ($this->confirm('Do you wish to continue? [yes|no]'))
	{
		//
	}

你也可以为 `confirm` 指定一个默认值，应该是 `true` 或者 `false`：

	$this->confirm($question, true);

<a name="registering-commands"></a>
## 注册命令

#### 注册一个 Artisan 命令

一旦你的命令完成后，你需要使用 Artisan 进行注册，这样才能够被使用。这通常在 `app/start/artisan.php` 文件中完成。在这个文件中，你可以使用 `Artisan::add` 函数注册命令：

**注册一个 Artisan 命令**

	Artisan::add(new CustomCommand);

#### 注册一个在 IoC 容器中的命令

如果你的命令在应用程序的 [IoC 容器](/docs/ioc) 中注册，你可以使用 `Artisan::resolve` 函数使它对 Artisan 可用：

	Artisan::resolve('binding.name');

<a name="calling-other-commands"></a>
## 调用其他命令

有时你可能希望在你的命令中调用其他命令，你可以通过使用 `call` 函数实现：

	$this->call('command.name', array('argument' => 'foo', '--option' => 'bar'));
