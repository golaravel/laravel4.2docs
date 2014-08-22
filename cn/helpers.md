# 助手函数

- [数组助手](#arrays)
- [应用路径](#paths)
- [字符串助手](#strings)
- [URLs路径](#urls)
- [杂项](#miscellaneous)

<a name="arrays"></a>
## 数组助手

### array_add

`array_add` 函数将一个指定键的元素添加进数组，如果数组中已有该键，则不添加。

  $array = array('foo' => 'bar');

	$array = array_add($array, 'key', 'value');

### array_divide

`array_divide` 返回两个数组，第一个包含数组里的所有键，第二个包含数组里的所有值。

	$array = array('foo' => 'bar');

	list($keys, $values) = array_divide($array);

### array_dot

`array_dot` 函数将多维数组转为一维数组，该数组不需要规则的结构。所有的键用'.'分割。

	$array = array('foo' => array('bar' => 'baz'));

	$array = array_dot($array);

	// array('foo.bar' => 'baz');

### array_except

`array_except` 函数移除指定键的元素（数组的第一维），第二个参数包含所有要移除键的数组，并返回新数组。

	$array = array_except($array, array('keys', 'to', 'remove'));

### array_fetch

`array_fetch` 获取多维数组的最终值，参数为 第一维的键.第二维的键.第三维的键.... 的形式，指定的维数 数组的形式需一致，否则Laravel将抛出'Undefined index'。

	$array = array(
		array('developer' => array('name' => 'Taylor')),
		array('developer' => array('name' => 'Dayle')),
	);

	$array = array_fetch($array, 'developer.name');

### array_first

`array_first` 方法返回第一个 满足匿名函数（该匿名函数作为参数传入） 返回true的元素的值。

	$array = array(100, 200, 300);

	$value = array_first($array, function($key, $value)
	{
		return $value >= 150;
	});

array_first的第三个参数为该操作指定默认返回值，若匿名函数永远不可能返回true将返回默认值。

	$value = array_first($array, $callback, $default);
    
### array_last

The `array_last` method returns the last element of an array passing a given truth test.

	$array = array(350, 400, 500, 300, 200, 100);

	$value = array_last($array, function($key, $value)
	{
		return $value > 350;
	});

	// 500

A default value may also be passed as the third parameter:

	$value = array_last($array, $callback, $default);

### array_flatten

`array_flatten` 获取多维数组的最终值，该数组不需要规则的结构。并丢掉键。

	$array = array('name' => 'Joe', 'languages' => array('PHP', 'Ruby'));

	$array = array_flatten($array);

	// array('Joe', 'PHP', 'Ruby');

### array_forget

`array_forget` 方法移除数组内指定的元素，通过 键.键.键 的形式来寻找指定要移除的元素

	$array = array('names' => array('joe' => array('programmer')));

	array_forget($array, 'names.joe');

### array_get

`array_get` 方法获取数组内指定的元素，通过 键.键.键 的形式来寻找指定的元素

	$array = array('names' => array('joe' => array('programmer')));

	$value = array_get($array, 'names.joe');

> **Note:** Want something like `array_get` but for objects instead? Use `object_get`.

### array_only

`array_only` 方法返回数组内指定键(仅限第一维的键)的的元素，参数为将获取的数组元素键的数组。

	$array = array('name' => 'Joe', 'age' => 27, 'votes' => 1);

	$array = array_only($array, array('name', 'votes'));

### array_pluck

`array_pluck` 返回数组内指定键的值，并丢掉键，只能指定一个键。

	$array = array(array('name' => 'Taylor'), array('name' => 'Dayle'));

	$array = array_pluck($array, 'name');

	// array('Taylor', 'Dayle');

### array_pull

`array_pull` 从原数组中删除指定键的元素，并返回被删除的元素的值。

	$array = array('name' => 'Taylor', 'age' => 27);

	$name = array_pull($array, 'name');

### array_set

`array_set` 为一个指定键的元素设置值，通过 键.键.键 的形式来寻找将被设置或重新赋值的元素。

	$array = array('names' => array('programmer' => 'Joe'));

	array_set($array, 'names.editor', 'Taylor');

### array_sort

`array_sort` 函数对数组里的第一维元素进行自定义排序，并保持第一维数组键。

	$array = array(
		array('name' => 'Jill'),
		array('name' => 'Barry'),
	);

	$array = array_values(array_sort($array, function($value)
	{
		return $value['name'];
	}));

### array_where

Filter the array using the given Closure.

	$array = array(100, '200', 300, '400', 500);

	$array = array_where($array, function($key, $value)
	{
		return is_string($value);
	});

	// Array ( [1] => 200 [3] => 400 )

### head

返回数组的第一个元素，内部实际调用reset方法。经常在链式调用时使用。

	$first = head($this->returnsArray('foo'));

### last

返回数组的最后一个元素，内部实际调用end方法。经常在链式调用时使用。

	$last = last($this->returnsArray('foo'));

<a name="paths"></a>
## 应用路径

### app_path

获取 `app` (/app[laravel4]) 目录的绝对路径。

	$path = app_path();
    
### base_path

获取laravel应用所在的绝对路径。

### public_path

获取 `public` 目录的绝对路径。

### storage_path

获取 `app/storage` 目录的绝对路径.

<a name="strings"></a>
## 字符串助手

### camel_case

将字符串转为 `驼峰命名法` 的格式.

	$camel = camel_case('foo_bar');

	// fooBar

### class_basename

获取不带命名空间的类名。

	$class = class_basename('Foo\Bar\Baz');

	// Baz

### e

`htmlentites` 的简写，以utf8格式转换字符串。

	$entities = e('<html>foo</html>');

### ends_with

判断字符串是否以另一个指定的字符串结束。

	$value = ends_with('This is my name', 'name');

### snake_case

将字符串转换为下划线命名方式。

	$snake = snake_case('fooBar');

	// foo_bar

### str_limit

Limit the number of characters in a string.

	str_limit($value, $limit = 100, $end = '...')

Example:

	$value = str_limit('The PHP framework for web artisans.', 7);

	// The PHP...

### starts_with

判断字符串是否以另一个指定的字符串开始。

	$value = starts_with('This is my name', 'This');

### str_contains

判断字符串是否包含另一个指定的字符串。

	$value = str_contains('This is my name', 'my');

### str_finish

以指定字符结束字符串，且保证字符串结尾有且只有一个指定的字符。

	$string = str_finish('this/string', '/');

	// this/string/

### str_is

判断字符串是否符合指定的格式。 下面的星号是作为通配符使用。

	$value = str_is('foo*', 'foobar');

### str_plural

将字符串转为复数形式（只适合英语）。

	$plural = str_plural('car');

### str_random

生成一个指定长度的字符串。

	$string = str_random(40);

### str_singular

将字符串转为单数形式（只适合英语）。

	$singular = str_singular('cars');

### studly_case

将字符串转为 `StudlyCase` 格式。

	$value = studly_case('foo_bar');

	// FooBar

### trans

翻译指定语言行（请配合本地化语言文件使用）。`Lang::get` 的别名。

	$value = trans('validation.required'):

### trans_choice

根据数量条件翻译指定语言行（主要针对外语的名词复数形式）。`Lang::choice` 的别名。

	$value = trans_choice('foo.bar', $count);

<a name="urls"></a>
## URLs

### action

从指定的控制器的方法生成url。

	$url = action('HomeController@getIndex', $params);

### 路由

为命名路由生成URL。

	$url = route('routeName', $params);

### asset

生成一个指向样式文件的url。

	$url = asset('img/photo.jpg');

### link_to

生成一个a标签的html代码，并能为该a标签指定详细信息。

	echo link_to('foo/bar', $title, $attributes = array(), $secure = null);

### link_to_asset

生成一个指向样式文件的a标签。

	echo link_to_asset('foo/bar.zip', $title, $attributes = array(), $secure = null);

### link_to_route

生成一个指向特定路由的a标签。

	echo link_to_route('route.name', $title, $parameters = array(), $attributes = array());

### link_to_action

生成一个指向特定控制器方法的a标签。

	echo link_to_action('HomeController@getIndex', $title, $parameters = array(), $attributes = array());

### secure_asset

生成一个指向样式文件的a标签并使用使用 HTTPS 安全链接。

	echo secure_asset('foo/bar.zip', $title, $attributes = array());

### secure_url

生成一个完全自定义的a标签并使用 HTTPS 安全链接。

	echo secure_url('foo/bar', $parameters = array());

### url

生成一个完全自定义的a标签。

	echo url('foo/bar', $parameters = array(), $secure = null);

<a name="miscellaneous"></a>
## 杂项

### csrf_token

获取当前 CSRF 标记。

	$token = csrf_token();

### dd

格式化输出指定的变量，并结束脚本执行。

	dd($value);

### value

如果参数为 `Closure`，value函数将返回 `Closure` 的返回值。 否则，返回参数本。

	$value = value(function() { return 'bar'; });

### with

参数为对象，直接返回该对象，在PHP 5.3.x中进行链式调用很有用。

	$value = with(new Foo)->doWork();
