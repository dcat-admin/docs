# 页面内容和布局

`Dcat Admin`的布局可参考后台首页的布局文件`app/Admin/Controllers/HomeController.php`的`index()`方法。


`Dcat\Admin\Layout\Content`类用来实现内容区的布局。`Content::body($content)`方法用来添加页面内容。


一个简单的后台页面代码如下：

```php
public function index()
{
    return Admin::content(function (Content $content) {

        // 选填
        $content->header('填写页面头标题');
        
        // 选填
        $content->description('填写页面描述小标题');
        
        // 添加面包屑导航
        $content->breadcrumb(
            ['text' => '首页', 'url' => '/admin'],
            ['text' => '用户管理', 'url' => '/admin/users'],
            ['text' => '编辑用户']
        );

        // 填充页面body部分，这里可以填入任何可被渲染的对象
        $content->body('hello world');
    });
}

```

其中 `$content->body()` 方法是 `$content->row()` 的别名方法，可以接受任何可字符串化的对象作为参数，可以是字符串、数字、包含了`__toString`方法的对象，实现了`Renderable`、`Htmlable`接口的对象，包括laravel的视图。


### 布局

`dcat-admin`的布局使用bootstrap的栅格系统，每行的长度是12，下面是几个简单的示例：

添加一行内容:

```php
$content->row('hello')

---------------------------------
|hello                          |
|                               |
|                               |
|                               |
|                               |
|                               |
---------------------------------

```

行内添加多列：

```php
$content->row(function(Row $row) {
    $row->column(4, 'foo');
    $row->column(4, 'bar');
    $row->column(4, 'baz');
});
----------------------------------
|foo       |bar       |baz       |
|          |          |          |
|          |          |          |
|          |          |          |
|          |          |          |
|          |          |          |
----------------------------------


$content->row(function(Row $row) {
    $row->column(4, 'foo');
    $row->column(8, 'bar');
});
----------------------------------
|foo       |bar                  |
|          |                     |
|          |                     |
|          |                     |
|          |                     |
|          |                     |
----------------------------------

```

列中添加行：

```php
$content->row(function (Row $row) {

    $row->column(4, 'xxx');

    $row->column(8, function (Column $column) {
        $column->row('111');
        $column->row('222');
        $column->row('333');
    });
});
----------------------------------
|xxx       |111                  |
|          |---------------------|
|          |222                  |
|          |---------------------|
|          |333                  |
|          |                     |
----------------------------------


```


列中添加行, 行内再添加列：

```php
$content->row(function (Row $row) {

    $row->column(4, 'xxx');

    $row->column(8, function (Column $column) {
        $column->row('111');
        $column->row('222');
        $column->row(function(Row $row) {
            $row->column(6, '444');
            $row->column(6, '555');
        });
    });
});
----------------------------------
|xxx       |111                  |
|          |---------------------|
|          |222                  |
|          |---------------------|
|          |444      |555        |
|          |         |           |
----------------------------------
```

### 事件

系统会在`Dcat\Admin\Layout\Content`类被实例化时和`render()`方法被调用时触发以下两个事件，开发者可以在这两个事件中改变或添加一些行为。

#### resolving
通过`Content::resolving`方法设置的回调函数会在`Dcat\Admin\Layout\Content`类被实例化时触发；

```php
use Dcat\Admin\Layout\Content;

Content::resolving(function (Content $content) {
    
    $content->setView('app.admin.content');
    
});
```

#### composing
通过`Content::composing`方法设置的回调函数会在`Dcat\Admin\Layout\Content::render`方法被调用时触发；

```php
use Dcat\Admin\Layout\Content;

Content::composing(function (Content $content) {
    
    $content->setView('app.admin.content');
    
});
```

#### composed
通过`Content::composed`方法设置的回调函数会在所有通过`Content::row`或`Content::body`方法设置的内容都构建完毕后触发。


```php
use Dcat\Admin\Layout\Content;

class IndexController
{
	public function index(Content $content)
	{
		Content::composed(function (Content $content) {
            // Grid已执行render方法
            
        });
        
        return $content->body(function ($row) {
        	$grid = new Grid(...);
        	
        	...
        	
        	$row->column(12, $grid);
        });
	}
}
```
