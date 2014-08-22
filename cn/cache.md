# 缓存

- [配置](#configuration)
- [缓存用法](#cache-usage)
- [增加 & 减少](#increments-and-decrements)
- [缓存标签](#cache-tags)
- [数据库缓存](#database-cache)

<a name="configuration"></a>
## 配置

Laravel 对不同的缓存机制提供了一套统一的API。缓存配置信息存放于`app/config/cache.php`文件。在该配置文件中，你可以指定整个应用程序所使用的缓存驱动器。Laravel自身支持大多数流行的缓存服务器，例如[Memcached](http://memcached.org)和[Redis](http://redis.io)。

缓存配置文件还包含了其他配置项，文件里都有详细说明，因此，请务必查看这些配置项和其描述信息。默认情况下，Laravel被配置为使用`file`缓存驱动，它将数据序列化，并存放于文件系统中。在大型应用中，强烈建议使用基于内存的缓存系统，例如Memcached或APC。

<a name="cache-usage"></a>
## 缓存用法

#### 将某一数据存入缓存

	Cache::put('key', 'value', $minutes);

#### Using Carbon Objects To Set Expire Time

	$expiresAt = Carbon::now()->addMinutes(10);

	Cache::put('key', 'value', $expiresAt);

#### 当某一数据不在缓存中是才将其保存

	Cache::add('key', 'value', $minutes);

如果该项实际上 **已经添加** 到缓存中，那么 `add` 方法将返回 `true` 。否则，此方法将返回 `false`。

#### 检查缓存中是否有某个key对应的数据

	if (Cache::has('key'))
	{
		//
	}

#### 从缓存中取得数据

	$value = Cache::get('key');

#### 从缓存中取得数据，如果数据不存，则返回指定的默认值

	$value = Cache::get('key', 'default');

	$value = Cache::get('key', function() { return 'default'; });

#### 将数据永久地存于缓存中

	Cache::forever('key', 'value');

有时你可能想从缓存中取得某项数据，但是还希望在数据不存在时存储一项默认值。那就可以通过 `Cache::remember`方法实现：

	$value = Cache::remember('users', $minutes, function()
	{
		return DB::table('users')->get();
	});

还可以将`remember`和`forever`方法结合使用：

	$value = Cache::rememberForever('users', function()
	{
		return DB::table('users')->get();
	});

注意：所有存在于缓存中的数据都是经过序列化的，因此，你可以存储任何类型的数据。

#### Pulling An Item From The Cache

If you need to retrieve an item from the cache and then delete it, you may use the `pull` method:

	$value = Cache::pull('key');

#### 从缓存中删除某项数据

	Cache::forget('key');

<a name="increments-and-decrements"></a>
## 增加 & 减少

除了`文件`和`数据库`驱动器，其他驱动器都支持`增加`和`减少`操作：

#### 让某个值增加

	Cache::increment('key');

	Cache::increment('key', $amount);

#### 让某个值减少

	Cache::decrement('key');

	Cache::decrement('key', $amount);

<a name="cache-tags"></a>
## 缓存标签

> **注意：** 当使用 `file` 或者 `database` 缓存驱动时，是不支持缓存标签的。此外，在使用多个缓存标签时它们将存储为 "forever"。使用一个如 `memcached` 的驱动性能将会是最好的，它会自动清除过时的记录。

#### 访问一个标记的缓存

缓存标签允许你在缓存中标记相关的项目，然后刷新指定名称标签的所有缓存。要访问标记的缓存，请使用 `tags` 方法：

你可以通过传递标签名称的有序列表或数组作为参数，来存储一个标记的缓存。

	Cache::tags('people', 'authors')->put('John', $john, $minutes);

	Cache::tags(array('people', 'artists'))->put('Anne', $anne, $minutes);

你可以在所有缓存存储方法中使用标签，包括 `remember`，`forever`，和 `rememberForever`。你也可以从缓存标签来访问已缓存的项目，以及使用其它缓存方法如 `increment` 和 `decrement`:

#### 从标记的缓存中访问条目

通过与保存时所用相同的标签，作为参数列表来访问标记的缓存。

	$anne = Cache::tags('people', 'artists')->get('Anne');

	$john = Cache::tags(array('people', 'authors'))->get('John');

你可以通过标签名称（或名称列表）来刷新所有相关的缓存项。例如，下面的语句将移除所有标签中包含 `people` 和 `authors` 的缓存项。因此无论是之前例子中的 "Anne" 还是 "John" 都将从缓存中移除：

	Cache::tags('people', 'authors')->flush();

相比之下，下面的语句将移除标签中仅包含 'authors' 的缓存项，因此 "John" 将被移除，但不影响 "Anne" 。

	Cache::tags('authors')->flush();

<a name="database-cache"></a>
## 数据库缓存

当使用`数据库`缓存驱动时，你需要设置一个数据表来缓存数据。以下案例是`Schema`表的定义：

	Schema::create('cache', function($table)
	{
		$table->string('key')->unique();
		$table->text('value');
		$table->integer('expiration');
	});

译者：王赛  [（Bootstrap中文网）](http://www.bootcss.com)
