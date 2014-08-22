# 迁移 & 数据填充

- [简介](#introduction)
- [创建迁移](#creating-migrations)
- [运行迁移](#running-migrations)
- [回滚迁移](#rolling-back-migrations)
- [数据库填充](#database-seeding)

<a name="introduction"></a>
## 简介

迁移是一种数据库的版本控制。它允许一个团队修改数据库结构，并在当前模式下保持最新。迁移通常和 [结构生成器](/docs/schema) 配对使用来管理您应用程序的结构。

<a name="creating-migrations"></a>
## 创建迁移

使用 Artisan 命令行的 `migrate:make` 命令创建一个迁移：

	php artisan migrate:make create_users_table

所有的迁移都被存放在 `app/database/migrations` 目录下，并且包含一个时间戳允许框架检测迁移的顺序。

您可以在创建迁移的时候指定 `--path` 选项，指定的目录应该相对于安装目录的主目录：

	php artisan migrate:make foo --path=app/migrations

`--table` 和 `--create` 选项可以被用来指定表名以及是否创建一个新表：

	php artisan migrate:make add_votes_to_user_table --table=users

	php artisan migrate:make create_users_table --create=users

<a name="running-migrations"></a>
## 运行迁移

#### 运行所有迁移

	php artisan migrate

#### 运行某个路径下的所有迁移

	php artisan migrate --path=app/foo/migrations

#### 运行某个包下的所有迁移

	php artisan migrate --package=vendor/package

> **注意:** 如果在运行迁移的时候收到一个 "class not found" 的错误，请尝试运行 `composer  dump-autoload` 命令。

### Forcing Migrations In Production

Some migration operations are destructive, meaning they may cause you to lose data. In order to protect you from running these commands against your production database, you will prompted for confirmation before these commands are executed. To force thse commands to run without a prompt, use the `--force` flag:

	php artisan migrate --force

<a name="rolling-back-migrations"></a>
## 回滚迁移

#### 回滚最后一次迁移

	php artisan migrate:rollback

#### 回滚所有迁移

	php artisan migrate:reset

#### 回滚所有迁移并重新运行所有迁移

	php artisan migrate:refresh

	php artisan migrate:refresh --seed

<a name="database-seeding"></a>
## 数据库填充

Laravel 还包含一个简单的方式通过填充类使用测试数据填充您的数据库。所有的填充类都存放在 `app/database/seeds` 目录下。填充类可以以形式命名，但最好遵循一些合理的约束，比如 `UserTableSeeder` 等。默认情况下，一个 `DatabaseSeeder` 类以为您定义。在这个类中，您可以使用 `call` 函数运行其他填充类，允许您控制填充顺序。

#### 数据库填充类的例子

	class DatabaseSeeder extends Seeder {

		public function run()
		{
			$this->call('UserTableSeeder');

			$this->command->info('User table seeded!');
		}

	}

	class UserTableSeeder extends Seeder {

		public function run()
		{
			DB::table('users')->delete();

			User::create(array('email' => 'foo@bar.com'));
		}

	}

使用 Artisan 命令行的 `db:seed` 命令填充数据库：

	php artisan db:seed

默认情况下，`db:seed` 命令运行的是 `DatabaseSeeder` 类，这个类还会调用其他seed类。然而，你可以是使用 `--class` 参数来指定一个seed类：

	php artisan db:seed --class=UserTableSeeder

您也可以使用 `migrate:refresh` 命令填充数据库，将会回滚并重新运行所有迁移：

	php artisan migrate:refresh --seed
