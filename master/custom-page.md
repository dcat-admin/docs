# 自定义页面

在`Dcat Admin`中构建自定义页面非常简单，可以参考如下两个例子


### 示例1

```php
<?php

use Illuminate\Contracts\Support\Renderable;
use Dcat\Admin\Admin;

class MyPage implements Renderable
{
    // 定义页面所需的静态资源，这里会按需加载
	public static $js = [
		'xxx/js/page.min.js',
	];
	public static $css = [
		'xxx/css/page.min.css',
	];

	public function script()
	{
		return <<<JS
		console.log('所有JS脚本都加载完了');
JS;		
	}

	public function render()
	{
		// 在这里可以引入你的js或css文件
		Admin::js(static::$js);
		Admin::css(static::$css);
		
		// 需要在页面执行的JS代码
		// 通过 Admin::script 设置的JS代码会自动在所有JS脚本都加载完毕后执行
		Admin::script($this->script());
		
		return view('admin.pages.my-page')->render();
	}
}
```

视图`admin.pages.my-page`，注意视图代码里面不要包含`<body>`和`<html>`等标签
```html
<div>
  <h3>自定义页面演示</h3>
</div>
<script>
Dcat.ready(function () {
    // js代码也可以放在模板里面
    console.log('所有JS脚本都加载完了!!!');
});
</script>
```

使用

```php
public function index(Content $content)
{
    return $content->body(new MyPage());
}
```


### 示例2

后台的仪表盘页面`/admin`，也可以看做是一个自定义页面，代码实现如下
```php
public function index(Content $content)
{
    return $content
        ->header('Dashboard')
        ->description('Description...')
        ->body(function (Row $row) {
            $row->column(6, function (Column $column) {
                $column->row(Dashboard::title());
                $column->row(new Examples\Tickets());
            });

            $row->column(6, function (Column $column) {
                $column->row(function (Row $row) {
                    $row->column(6, new Examples\NewUsers());
                    $row->column(6, new Examples\NewDevices());
                });

                $column->row(new Examples\Sessions());
                $column->row(new Examples\ProductOrders());
            });
        });
}
```

