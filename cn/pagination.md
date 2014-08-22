# 分页

- [配置](#configuration)
- [基本用法](#usage)
- [给分页链接添加自定义信息](#appending-to-pagination-links)
- [Converting To JSON](#converting-to-json)
- [Custom Presenters](#custom-presenters)

<a name="configuration"></a>
## 配置

在其它的框架中，分页有时很痛苦. 但是Laravel让分页简单到不可思议. 默认Laravel包含了两个分页视图, 在`app/config/view.php` 文件中的pagination选项中指定分页链接具体使用哪一个视图.

`pagination::slider`视图 基于当前所在页数给出一个浮动的页数范围,`pagination::simple` 视图只是简单的给出 '上一页' '下一页' 两个链接. **两个视图都能完美的和bootstrap框架结合**

<a name="usage"></a>
## 基本用法

Laravel有多种方式实现分页. 最简单的是在普通查询或Eloquent模型查询中使用 `paginate` 方法.

#### 从数据库查询中分页

	$users = DB::table('users')->paginate(15);

#### 从一个 Eloquent 模型中分页

也可以从 [Eloquent](/docs/eloquent) 模型中分页:

	$allUsers = User::paginate(15);

	$someUsers = User::where('votes', '>', 100)->paginate(15);

你只需要吧每页查询的记录数传给`paginate`方法即可. 在获得包含分页的查询结果集之后, 就可以在视图中展示了, 在视图中输出分页链接使用 `links` 方法:

	<div class="container">
		<?php foreach ($users as $user): ?>
			<?php echo $user->name; ?>
		<?php endforeach; ?>
	</div>

	<?php echo $users->links(); ?>

以上就是创建一个分页链接的所需的所有信息了! 我们还没有提到Laravel做了什么. Laravel会自己把其它的事情做完.

If you would like to specify a custom view to use for pagination, you may pass a view to the `links` method:

	<?php echo $users->links('view.name'); ?>

你也可以通过下面的方法获取关于分页更加详尽的信息:

- `getCurrentPage`
- `getLastPage`
- `getPerPage`
- `getTotal`
- `getFrom`
- `getTo`
- `count`

#### "Simple Pagination"

If you are only showing "Next" and "Previous" links in your pagination view, you have the option of using the `simplePaginate` method to perform a more efficient query. This is useful for larger datasets when you do not require the display of exact page numbers on your view:

	$someUsers = User::where('votes', '>', 100)->simplePaginate(15);

#### 自定义分页

有时你可能希望自定义分页, 只需使用 `Paginator::make` 方法,并把 记录集合(当前页需要展示的数据集), 总记录数(int), 每页记录数(int) 作为参数:


	$paginator = Paginator::make($items, $totalItems, $perPage);
    
#### Customizing The Paginator URI

You may also customize the URI used by the paginator via the `setBaseUrl` method:

	$users = User::paginate();

	$users->setBaseUrl('custom/url');

The example above will create URLs like the following: http://example.com/custom/url?page=2

<a name="appending-to-pagination-links"></a>
## 给分页链接添加自定义信息

可以通过分页器的 `appends` 方法为分页链接添加上自定查询字符串:

	<?php echo $users->appends(array('sort' => 'votes'))->links(); ?>

最后产生的url如下:

	http://example.com/something?page=2&sort=votes

If you wish to append a "hash fragment" to the paginator's URLs, you may use the `fragment` method:

	<?php echo $users->fragment('foo')->links(); ?>

This method call will generate URLs that look something like this:

	http://example.com/something?page=2#foo

<a name="converting-to-json"></a>
## Converting To JSON

The `Paginator` class implements the `Illuminate\Support\Contracts\JsonableInterface` contract and exposes the `toJson` method. You can may also convert a `Paginator` instance to JSON by returning it from a route. The JSON'd form of the instance will include some "meta" information such as `total`, `current_page`, `last_page`, `from`, and `to`. The instance's data will be available via the `data` key in the JSON array.

<a name="custom-presenters"></a>
## Custom Presenters

The default pagination presenter is Bootstrap compatible out of the box; however, you may customize this with a presenter of your choice.

### Extending The Abstract Presenter

Extend the `Illuminate\Pagination\Presenter` class and implement its abstract methods. An example presenter for Zurb Foundation might look like this:

    class ZurbPresenter extends Illuminate\Pagination\Presenter {

        public function getActivePageWrapper($text)
        {
            return '<li class="current"><a href="">'.$text.'</a></li>';
        }

        public function getDisabledTextWrapper($text)
        {
            return '<li class="unavailable">'.$text.'</li>';
        }

        public function getPageLinkWrapper($url, $page)
        {
            return '<li><a href="'.$url.'">'.$page.'</a></li>';
        }

    }

### Using The Custom Presenter

First, create a view in your `app/views` directory that will server as your custom presenter. Then, replace `pagination` option in the `app/config/view.php` configuration file with the new view's name. Finally, the following code would be placed in your custom presenter view:

    <ul class="pagination">
        <?php echo with(new ZurbPresenter($paginator))->render(); ?>
    </ul>
