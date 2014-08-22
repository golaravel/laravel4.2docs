# 结构生成器

- [简介](#introduction)
- [创建和删除表](#creating-and-dropping-tables)
- [添加字段](#adding-columns)
- [重命名字段](#renaming-columns)
- [删除字段](#dropping-columns)
- [检查存在性](#checking-existence)
- [添加索引](#adding-indexes)
- [外键](#foreign-keys)
- [删除索引](#dropping-indexes)
- [Dropping Timestamps & Soft Deletes](#dropping-timestamps)
- [存储引擎](#storage-engines)

<a name="introduction"></a>
## 简介

Laravel 的 `Schema` 类提供了一种与数据库无关的方式维护表。它和 Laravel 所支持的所有数据库都能很好的工作，并且提供了统一的接口。

<a name="creating-and-dropping-tables"></a>
## 创建和删除表

使用 `Schema::create` 创建一个数据库的表：

	Schema::create('users', function($table)
	{
		$table->increments('id');
	});

传递给 `create` 函数的第一个参数是表的名字，第二个参数是一个闭包，将接受一个 `Blueprint` 对象用于定义新的表。

使用 `rename` 函数重命名一个已存在的表：

	Schema::rename($from, $to);

使用 `Schema::connection` 函数指定结构操作所使用的数据库连接：

	Schema::connection('foo')->create('users', function($table)
	{
		$table->increments('id');
	});

使用 `Schema::drop` 函数删除一个表：

	Schema::drop('users');

	Schema::dropIfExists('users');

<a name="adding-columns"></a>
## 添加字段

使用 `Schema::table` 函数更新一个已存在的表：

	Schema::table('users', function($table)
	{
		$table->string('email');
	});


表生成器包含一系列的字段类型用于构建表：

命令  | 描述
------------- | -------------
`$table->bigIncrements('id');`  |  Incrementing ID using a "big integer" equivalent.
`$table->bigInteger('votes');`  |  BIGINT equivalent to the table
`$table->binary('data');`  |  BLOB equivalent to the table
`$table->boolean('confirmed');`  |  BOOLEAN equivalent to the table
`$table->char('name', 4);`  |  CHAR equivalent with a length
`$table->date('created_at');`  |  DATE equivalent to the table
`$table->dateTime('created_at');`  |  DATETIME equivalent to the table
`$table->decimal('amount', 5, 2);`  |  DECIMAL equivalent with a precision and scale
`$table->double('column', 15, 8);`  |  DOUBLE equivalent with precision
`$table->enum('choices', array('foo', 'bar'));` | ENUM equivalent to the table
`$table->float('amount');`  |  FLOAT equivalent to the table
`$table->increments('id');`  |  Incrementing ID to the table (primary key).
`$table->integer('votes');`  |  INTEGER equivalent to the table
`$table->longText('description');`  |  LONGTEXT equivalent to the table
`$table->mediumInteger('numbers');`  |  MEDIUMINT equivalent to the table
`$table->mediumText('description');`  |  MEDIUMTEXT equivalent to the table
`$table->morphs('taggable');`  |  Adds INTEGER `taggable_id` and STRING `taggable_type`
`$table->nullableTimestamps();`  |  Same as `timestamps()`, except allows NULLs
`$table->smallInteger('votes');`  |  SMALLINT equivalent to the table
`$table->tinyInteger('numbers');`  |  TINYINT equivalent to the table
`$table->softDeletes();`  |  Adds **deleted\_at** column for soft deletes
`$table->string('email');`  |  VARCHAR equivalent column
`$table->string('name', 100);`  |  VARCHAR equivalent with a length
`$table->text('description');`  |  TEXT equivalent to the table
`$table->time('sunrise');`  |  TIME equivalent to the table
`$table->timestamp('added_on');`  |  TIMESTAMP equivalent to the table
`$table->timestamps();`  |  Adds **created\_at** and **updated\_at** columns
`->nullable()`  |  Designate that the column allows NULL values
`->default($value)`  |  Declare a default value for a column
`->unsigned()`  |  Set INTEGER to UNSIGNED


#### 在 MySQL 中使用 After

如果你使用 MySQL 数据库，您可以使用 `after` 函数指明字段的顺序：

	$table->string('name')->after('email');

<a name="renaming-columns"></a>
## 重命名字段

使用 `renameColumn` 函数重命名一个字段：

	Schema::table('users', function($table)
	{
		$table->renameColumn('from', 'to');
	});

> **注意:** 不支持重命名 `enum` 字段类型.

<a name="dropping-columns"></a>
## 删除字段

#### 从表中删除一个字段

	Schema::table('users', function($table)
	{
		$table->dropColumn('votes');
	});

#### 从表中删除多个字段

	Schema::table('users', function($table)
	{
		$table->dropColumn('votes', 'avatar', 'location');
	});

<a name="checking-existence"></a>
## 检查存在性

#### 检查表是否存在

您可以使用 `hasTable` 和 `hasColumn` 检查一个表或一个字段是否存在：

	if (Schema::hasTable('users'))
	{
		//
	}

#### 检查字段是否存在

	if (Schema::hasColumn('users', 'email'))
	{
		//
	}

<a name="adding-indexes"></a>
## 添加索引

结构生成器支持多种类型的索引，有两种方法可以添加它们。首先，您可以在字段定义后链式的定义它们，或者独立的添加它们：

	$table->string('email')->unique();

或者，您可以选择在不同的行添加索引。下面是全部支持的索引类型：

命令  | 描述
------------- | -------------
`$table->primary('id');`  |  添加一个主键
`$table->primary(array('first', 'last'));`  |  添加组合键
`$table->unique('email');`  |  添加唯一键
`$table->index('state');`  |  添加一个索引

<a name="foreign-keys"></a>
## 外键

Laravel 也支持向表中添加外键约束：

	$table->foreign('user_id')->references('id')->on('users');

在这个例子中，我们指明 `user_id` 字段参照 `users` 表中的 `id` 字段。

您也可以指明 "on delete" 以及 "on update" 行为选项：

	$table->foreign('user_id')
          ->references('id')->on('users')
          ->onDelete('cascade');

可以使用 `dropForeign` 函数删除一个外键。像其他索引一样，一个相似的命名惯例被使用于外键的命名：

	$table->dropForeign('posts_user_id_foreign');

> **注意:** 当创建一个参照递增整数类型的外键的时候，记得把外键字段的类型定义为无符号。

<a name="dropping-indexes"></a>
## 删除索引

为了删除索引，必须指明索引的名字。Laravel 默认为索引分配了一个合理的名字。通过连接表明、索引的字段名以及索引类型的形式。这里是一些例子：

命令  | 描述
------------- | -------------
`$table->dropPrimary('users_id_primary');`  |  从 "users" 表中删除一个主键
`$table->dropUnique('users_email_unique');`  |  从 "users" 表中删除一个唯一键
`$table->dropIndex('geo_state_index');`  |  从 "geo" 表中删除一个索引

<a name="dropping-timestamps"></a>
## Dropping Timestamps & SoftDeletes

To drop the `timestamps`, `nullableTimestamps` or `softDeletes` column types, you may use the following methods:

Command  | Description
------------- | -------------
`$table->dropTimestamps();`  |  Dropping the **created\_at** and **updated\_at** columns from the table
`$table->dropSoftDeletes();`  |  Dropping **deleted\_at** column from the table

<a name="storage-engines"></a>
## 存储引擎

通过在结构生成器设置 `engine` 属性为表设置存储引擎：

    Schema::create('users', function($table)
    {
        $table->engine = 'InnoDB';

        $table->string('email');
    });
