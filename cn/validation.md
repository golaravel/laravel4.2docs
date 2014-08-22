# 验证

- [基本使用](#basic-usage)
- [使用错误消息](#working-with-error-messages)
- [错误消息 & 视图](#error-messages-and-views)
- [可用的验证规则](#available-validation-rules)
- [有条件的添加规则](#conditionally-adding-rules)
- [自定义错误消息](#custom-error-messages)
- [自定义验证规则](#custom-validation-rules)

<a name="basic-usage"></a>
## 基本使用

Laravel 自带一个简单、方便的 `Validation` 类用于验证数据以及获取错误消息。

#### 基本验证例子

	$validator = Validator::make(
		array('name' => 'Dayle'),

		array('name' => 'required|min:5')
	);

传递给 `make` 函数的第一个参数是待验证的数据，第二个参数是对该数据需要应用的验证规则。

#### 通过数组指定验证规则

多个验证规则可以通过 "|" 字符进行分隔，或者作为数组的一个单独的元素。

	$validator = Validator::make(
		array('name' => 'Dayle'),
		array('name' => array('required', 'min:5'))
	);

#### 验证多个字段

    $validator = Validator::make(
        array(
            'name' => 'Dayle',
            'password' => 'lamepassword',
            'email' => 'email@example.com'
        ),
        array(
            'name' => 'required',
            'password' => 'required|min:8',
            'email' => 'required|email|unique:users'
        )
    );

一旦一个 `Validator` 实例被创建，可以使用 `fails` （或者 `passes`）函数执行这个验证。

	if ($validator->fails())
	{
		// The given data did not pass validation
	}

如果验证失败，你可以从验证器中获取错误消息。

	$messages = $validator->messages();

你也可以使用 `failed` 函数得到不带错误消息的没有通过验证的规则的数组。

	$failed = $validator->failed();

#### 文件验证

`Validator` 类提供了一些验证规则用于验证文件，比如 `size`、`mimes`等。在验证文件的时候，你可以和其他数据一起传递给验证器。

<a name="working-with-error-messages"></a>
## 使用错误消息

在一个 `Validator` 实例上调用 `messages` 函数之后，将会得到一个 `MessageBag` 实例，该实例拥有很多方便使用错误消息的函数。

#### 获取一个字段的第一个错误消息

	echo $messages->first('email');

#### 获取一个字段的全部错误消息**

	foreach ($messages->get('email') as $message)
	{
		//
	}

#### 获取全部字段的全部错误消息

	foreach ($messages->all() as $message)
	{
		//
	}

#### 检查一个字段是否存在消息

	if ($messages->has('email'))
	{
		//
	}

#### 以某种格式获取一条错误消息

	echo $messages->first('email', '<p>:message</p>');

> **注意:** 默认情况下，消息将使用与 Bootstrap 兼容的语法进行格式化。

#### 以某种格式获取所有错误消息

	foreach ($messages->all('<li>:message</li>') as $message)
	{
		//
	}

<a name="error-messages-and-views"></a>
## 错误消息 & 视图

一旦你执行了验证，你需要一种简单的方法向视图反馈错误消息。这在 Lavavel 中能够方便的处理。以下面的路由作为例：

	Route::get('register', function()
	{
		return View::make('user.register');
	});

	Route::post('register', function()
	{
		$rules = array(...);

		$validator = Validator::make(Input::all(), $rules);

		if ($validator->fails())
		{
			return Redirect::to('register')->withErrors($validator);
		}
	});

注意当验证失败，我们使用 `withErrors` 函数把 `Validator` 实例传递给 Redirect。这个函数将刷新 Session 中保存的错误消息，使得它们在下次请求中可用。

然而，请注意在我们的 GET 路由中并没有明确的绑定错误消息到视图。这是因为 Laravel 总会检查 Session 中的错误，并且如果它们是可用的将自动绑定到视图。**所以，需要特别注意的是，对于每个请求，一个 `$errors` 变量在所有视图中总是可用的**，允许你方便的认为 `$errors` 变量总是被定义并可以安全使用的。`$errors` 变量将是一个 `MessageBag` 类的实例。

所以，在跳转之后，你可以在视图中使用自动绑定的 `$errors` 变量：

	<?php echo $errors->first('email'); ?>

### Named Error Bags

If you have multiple forms on a single page, you may wish to name the `MessageBag` of errors. This will allow you to retrieve the error messages for a specific form. Simply pass a name as the second argument to `withErrors`:

	return Redirect::to('register')->withErrors($validator, 'login');

You may then access the named `MessageBag` instance from the `$errors` variable:

	<?php echo $errors->login->first('email'); ?>

<a name="available-validation-rules"></a>
## 可用的验证规则

下面是一个所有可用的验证规则的列表以及它们的功能：

- [Accepted](#rule-accepted)
- [Active URL](#rule-active-url)
- [After (Date)](#rule-after)
- [Alpha](#rule-alpha)
- [Alpha Dash](#rule-alpha-dash)
- [Alpha Numeric](#rule-alpha-num)
- [Array](#rule-array)
- [Before (Date)](#rule-before)
- [Between](#rule-between)
- [Confirmed](#rule-confirmed)
- [Date](#rule-date)
- [Date Format](#rule-date-format)
- [Different](#rule-different)
- [Digits](#rule-digits)
- [Digits Between](#rule-digits-between)
- [E-Mail](#rule-email)
- [Exists (Database)](#rule-exists)
- [Image (File)](#rule-image)
- [In](#rule-in)
- [Integer](#rule-integer)
- [IP Address](#rule-ip)
- [Max](#rule-max)
- [MIME Types](#rule-mimes)
- [Min](#rule-min)
- [Not In](#rule-not-in)
- [Numeric](#rule-numeric)
- [Regular Expression](#rule-regex)
- [Required](#rule-required)
- [Required If](#rule-required-if)
- [Required With](#rule-required-with)
- [Required With All](#rule-required-with-all)
- [Required Without](#rule-required-without)
- [Required Without All](#rule-required-without-all)
- [Same](#rule-same)
- [Size](#rule-size)
- [Unique (Database)](#rule-unique)
- [URL](#rule-url)

<a name="rule-accepted"></a>
#### accepted

验证此规则的值必须是 _yes_、 _on_ 或者是 _1_。这在验证是否同意"服务条款"的时候非常有用。

<a name="rule-active-url"></a>
#### active_url

验证此规则的值必须是一个合法的 URL，根据 PHP 函数 `checkdnsrr`。

<a name="rule-after"></a>
#### after:_date_

验证此规则的值必须在给定日期之后，该日期将被传递到 PHP 的 `strtotime` 函数。

<a name="rule-alpha"></a>
#### alpha

验证此规则的值必须全部由字母字符构成。

<a name="rule-alpha-dash"></a>
#### alpha_dash

验证此规则的值必须全部由字母、数字、中划线或下划线字符构成。

<a name="rule-alpha-num"></a>
#### alpha_num

验证此规则的值必须全部由字母和数字构成。

<a name="rule-array"></a>
#### array

The field under validation must be of type array.

<a name="rule-before"></a>
#### before:_date_

验证此规则的值必须在给定日期之前，该日期将被传递到 PHP 的 `strtotime` 函数。

<a name="rule-between"></a>
#### between:_min_,_max_

验证此规则的值必须在给定的 _min_ 和 _max_ 之间。字符串、数字以及文件都将使用大小规则进行比较。

<a name="rule-confirmed"></a>
#### confirmed

验证此规则的值必须和 `foo_confirmation` 的值相同。比如，需要验证此规则的字段是 `password`，那么在输入中必须有一个与之相同的 `password_confirmation` 字段。

<a name="rule-date"></a>
#### date

验证此规则的值必须是一个合法的日期，根据 PHP 函数 `strtotime`。

<a name="rule-date-format"></a>
#### date_format:_format_

验证此规则的值必须符合给定的 _format_ 的格式，根据 PHP 函数 `date_parse_from_format`。

<a name="rule-different"></a>
#### different:_field_

验证此规则的值必须与指定的 _field_ 字段的值不同。

<a name="rule-digits"></a>
#### digits:_value_

验证此规则的值必须是一个 _数字_ 并且必须满足 _value_ 设定的精确长度。

<a name="rule-digits-between"></a>
#### digits_between:_min_,_max_

验证此规则的值，它的长度必须介于 _min_ 和 _max_ 之间。

<a name="rule-email"></a>
#### email

验证此规则的值必须是一个合法的电子邮件地址。

<a name="rule-exists"></a>
#### exists:_table_,_column_

验证此规则的值必须在指定的数据库的表中存在。

#### Exists 规则的基础使用

	'state' => 'exists:states'

#### 指定列名

	'state' => 'exists:states,abbreviation'

你也可以指定更多的条件，将以 "where" 的形式添加到查询。

	'email' => 'exists:staff,email,account_id,1'

传递 `NULL` 到 "where" 子句中，将会直接在数据库中查找 `NULL` 值：

	'email' => 'exists:staff,email,deleted_at,NULL'

<a name="rule-image"></a>
#### image

验证此规则的值必须是一个图片 (jpeg, png, bmp 或者 gif)。

<a name="rule-in"></a>
#### in:_foo_,_bar_,...

验证此规则的值必须在给定的列表中存在。

<a name="rule-integer"></a>
#### integer

验证此规则的值必须是一个整数。

<a name="rule-ip"></a>
#### ip

验证此规则的值必须是一个合法的 IP 地址。

<a name="rule-max"></a>
#### max:_value_

验证此规则的值必须小于等于最大值 _value_。字符串、数字以及文件都将使用大小规则进行比较。

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

验证此规则的文件的 MIME 类型必须在给定的列表中。

#### MIME 规则的基础使用

	'photo' => 'mimes:jpeg,bmp,png'

<a name="rule-min"></a>
#### min:_value_

验证此规则的值必须大于最小值 _value_。字符串、数字以及文件都将使用大小规则进行比较。

<a name="rule-not-in"></a>
#### not_in:_foo_,_bar_,...

验证此规则的值必须在给定的列表中不存在。

<a name="rule-numeric"></a>
#### numeric

验证此规则的值必须是一个数字。

<a name="rule-regex"></a>
#### regex:_pattern_

验证此规则的值必须符合给定的正则表达式。

**注意：** 当使用 `regex` 模式的时候，可能需要在一个数组中指定规则，而不是使用 "|" 分隔符，特别是正则表达式中包含一个 "|" 字符的时候。

<a name="rule-required"></a>
#### required

验证此规则的值必须在输入数据中存在。

<a name="rule-required-if"></a>
#### required\_if:_field_,_value_

如果指定的 _field_ 字段等于指定的 _value_ ，那么验证此规则的值必须存在。

<a name="rule-required-with"></a>
#### required_with:_foo_,_bar_,...

_仅当_ 任意其它指定的字段存在的时候，验证此规则的值必须存在。

<a name="rule-required-with-all"></a>
#### required_with_all:_foo_,_bar_,...

_仅当_ 所有其它指定的字段存在的时候，验证此规则的值必须存在。

<a name="rule-required-without"></a>
#### required_without:_foo_,_bar_,...

_仅当_ 其它指定的字段有一个不存在的时候，验证此规则的值必须存在。

<a name="rule-required-without-all"></a>
#### required_without_all:_foo_,_bar_,...

_仅当_ 其它指定的字段都不存在的时候，验证此规则的值必须存在。

<a name="rule-same"></a>
#### same:_field_

验证此规则的值必须与给定的 _field_ 字段的值相同。

<a name="rule-size"></a>
#### size:_value_

验证此规则的值的大小必须与给定的 _value_ 相同。对于字符串，_value_ 代表字符的个数；对于数字，_value_ 代表它的整数值，对于文件，_value_ 代表文件以KB为单位的大小。

<a name="rule-unique"></a>
#### unique:_table_,_column_,_except_,_idColumn_

验证此规则的值必须在给定的数据库的表中唯一。如果 `column` 没有被指定，将使用该字段的名字。

#### Unique 规则的基础使用

	'email' => 'unique:users'

#### 指定列名

	'email' => 'unique:users,email_address'

#### 强制忽略一个给定的 ID

	'email' => 'unique:users,email_address,10'

#### 添加额外的where语句

你还可以指定更多条件，这些条件将被添加到查询的"where"语句中：

	'email' => 'unique:users,email_address,NULL,id,account_id,1'

在上面的规则中，只有`account_id` 为 `1` 的行才会被包含到unique检查中。

<a name="rule-url"></a>
#### url

验证此规则的值必须是一个合法的 URL。

> **Note:** This function uses PHP's `filter_var` method.

<a name="conditionally-adding-rules"></a>
## 有条件的添加规则

In some situations, you may wish to run validation checks against a field **only** if that field is present in the input array. To quickly accomplish this, add the `sometimes` rule to your rule list:

	$v = Validator::make($data, array(
		'email' => 'sometimes|required|email',
	));

In the example above, the `email` field will only be validated if it is present in the `$data` array.

#### Complex Conditional Validation

有时你可能希望给定的字段仅当另一个字段的值大于100的时候必须存在。或者你可能需要两个字段均含有一个给定的值，仅当另一个字段存在的时候。添加这些验证规则并没有那么麻烦。首先，创建一个使用你永远不会改变的 _static rules_ 的 `Validator` 实例：

	$v = Validator::make($data, array(
		'email' => 'required|email',
		'games' => 'required|numeric',
	));

假设我们的WEB应用程序是服务于游戏收藏爱好者们。如果一个游戏收藏爱好者注册了我们的应用程序，并且他们拥有100多款游戏，我们想让他们说明为什么他们会拥有如此多的游戏。例如，或许他们要开一个游戏转售店，或者也许他们只是喜欢收集。为了有条件的添加这个需求，我们可以使用 `Validator` 实例的 `sometimes` 函数。

	$v->sometimes('reason', 'required|max:500', function($input)
	{
		return $input->games >= 100;
	});

`sometimes` 函数的第一个参数是我们有条件的验证的字段名。第二个参数是我们要添加的规则。如果 `Closure` 作为第三个参数且返回了 `true`，规则将被添加。这种方法可以很容易构建复杂的条件验证。你甚至可以一次性为多个字段添加条件验证：

	$v->sometimes(array('reason', 'cost'), 'required', function($input)
	{
		return $input->games >= 100;
	});

> **注意：** 传递到你的 `Closure` 中的 `$input` 参数将被作为 `Illuminate\Support\Fluent` 的一个实例，并且可能被用作一个对象来访问你的输入和文件。

<a name="custom-error-messages"></a>
## 自定义错误消息

如果有需要，你可以使用自定义的错误消息代替默认的消息。这里有好几种自定义错误消息的方法。

#### 传递自定义消息到验证器

	$messages = array(
		'required' => 'The :attribute field is required.',
	);

	$validator = Validator::make($input, $rules, $messages);

> 注意:* `:attribute` 占位符将被实际要验证的字段名替换，你也可以在错误消息中使用其他占位符。

#### 其他验证占位符

	$messages = array(
		'same'    => 'The :attribute and :other must match.',
		'size'    => 'The :attribute must be exactly :size.',
		'between' => 'The :attribute must be between :min - :max.',
		'in'      => 'The :attribute must be one of the following types: :values',
	);

#### 对一个指定的字段指定自定义的错误消息

有些时候，你可能希望只对一个指定的字段指定自定义的错误消息：

	$messages = array(
		'email.required' => 'We need to know your e-mail address!',
	);

<a name="localization"></a>
#### 在语言文件中指定错误消息

在一些情况下，你可能希望在一个语言文件中指定你的错误消息而不是直接传递给 `Validator`。为了实现这个目的，请在 `app/lang/xx/validation.php` 文件中添加你的自定义消息到 `custom` 数组。

	'custom' => array(
		'email' => array(
			'required' => 'We need to know your e-mail address!',
		),
	),

<a name="custom-validation-rules"></a>
## 自定义验证规则

#### 注册一个自定义的验证规则

Laravel 提供了一系列有用的验证规则；但是，你可能希望添加自己的验证规则。其中一种方法是使用 `Validator::extend` 函数注册自定义的验证规则：

	Validator::extend('foo', function($attribute, $value, $parameters)
	{
		return $value == 'foo';
	});

自定义的验证器接受三个参数：待验证属性的名字、待验证属性的值以及传递给这个规则的参数。

你也可以传递一个类的函数到 `extend` 函数，而不是使用闭包：

	Validator::extend('foo', 'FooValidator@validate');

注意你需要为你的自定义规则定义错误消息。你既可以使用一个行内的自定义消息数组，也可以在验证语言文件中进行添加。

#### 扩展验证器类

你也可以扩展 `Validator` 类本身，而不是使用闭包回调扩展验证器。为了实现这个目的，添加一个继承自 `Illuminate\Validation\Validator` 的验证器类。你可以在类中添加以 `validate` 开头的验证函数：

	<?php

	class CustomValidator extends Illuminate\Validation\Validator {

		public function validateFoo($attribute, $value, $parameters)
		{
			return $value == 'foo';
		}

	}

#### 你需要注册自定义的验证器扩展

下面，你需要注册自定义的验证器扩展：

	Validator::resolver(function($translator, $data, $rules, $messages)
	{
		return new CustomValidator($translator, $data, $rules, $messages);
	});

当创建一个自定义的验证规则，你有时需要为错误消息定义一个自定义的占位符。为了实现它，你可以像上面那样创建一个自定义的验证器，并且在验证器中添加一个 `replaceXXX` 函数：

	protected function replaceFoo($message, $attribute, $rule, $parameters)
	{
		return str_replace(':foo', $parameters[0], $message);
	}

If you would like to add a custom message "replacer" without extending the `Validator` class, you may use the `Validator::replacer` method:

	Validator::replacer('rule', function($message, $attribute, $rule, $parameters)
	{
		//
	});