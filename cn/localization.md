# 本地化

- [简介](#introduction)
- [语言文件](#language-files)
- [基本用例](#basic-usage)
- [复数形式](#pluralization)
- [验证消息的本地化](#validation)
- [Overriding Package Language Files](#overriding-package-language-files)

<a name="introduction"></a>
## 简介

Laravel `Lang` 类提供非常方便的方法来从不同语言文件中取得字符串，允许在应用程序中支持多语言。

<a name="language-files"></a>
## 语言文件

语言文字存放在 `app/lang` 目录。应用程序所要支持的语言都需要在此目录建立为子目录。

	/app
		/lang
			/en
				messages.php
			/es
				messages.php

#### 语言文件示例

语言文件只是返回键值（字符串）对数组。例如：

	<?php

	return array(
		'welcome' => 'Welcome to our application'
	);

#### 在运行时改变默认语言

应用程序默认语言配置在 `app/config/app.php`配置文件中 `locale` 配置项.你可以在任何时候使用 `App::setLocale` 方法来改变当前激活语言。

	App::setLocale('es');

#### Setting The Fallback Language

You may also configure a "fallback language", which will be used when the active language does not contain a given language line. Like the default language, the fallback language is also configured in the `app/config/app.php` configuration file:

	'fallback_locale' => 'en',

<a name="basic-usage"></a>
## 基本用例

#### 从语言文件中获取文本

	echo Lang::get('messages.welcome');

传给 `get` 方法字符串第一部分是语言文件名称，第二个部分是要取得文本的名字。

> **注意**: 如果语言不存在该文本，那么 `get` 方法会将键返回。

#### 文本中替换

可以在语言文件中定义占位符：

	'welcome' => 'Welcome, :name',

然后，将要替换的值传递给 `Lang::get` 方法的第二个参数：

	echo Lang::get('messages.welcome', array('name' => 'Dayle'));

#### 判断语言文件是否存在该文本

	if (Lang::has('messages.welcome'))
	{
		//
	}

<a name="pluralization"></a>
## 复数形式

复数形式是一个复杂的问题，因为不同的语言有着不同的复数形式规则。你可以通过简单的在语言文件中使用”管道“符来分开单数和复数文本形式：

	'apples' => 'There is one apple|There are many apples',

然后你就可以使用 `Lang::choice` 方法来取得文本：

	echo Lang::choice('messages.apples', 10);

You may also supply a locale argument to specify the language. For example, if you want to use the Russian (ru) language:

	echo Lang::choice('товар|товара|товаров', $count, array(), 'ru');

由于Laravel翻译机制是用Symfony的翻译组件，你也可以非常简单的创建更加复杂的复数形式规则：

	'apples' => '{0} There are none|[1,19] There are some|[20,Inf] There are many',


<a name="validation"></a>
## 验证

对于验证功能中需要本地化的错误信息和提示信息，请参阅 <a href="/docs/validation#localization">相关文档</a>。

<a name="overriding-package-language-files"></a>
## Overriding Package Language Files

Many packages ship with their own language lines. Instead of hacking the package's core files to tweak these lines, you may override them by placing files in the `app/lang/packages/{locale}` directory. So, for example, if you need to override the English language lines in  `messages.php`  for a package named `skyrim/hearthfire`, you would place a language file at: `app/lang/packages/en/hearthfire.php`. In this file you would define only the language lines you wish to override. Any language lines you don't override will still be loaded from the package's language files.