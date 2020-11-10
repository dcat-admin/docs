# Custom pages

Building a custom page in `Dcat Admin` is very simple, you can refer to the following two examples


### Example 1

> {tip} `Dcat Admin` is a single page application and the loaded `JS` script will only be executed once, so the initialization operation cannot be placed directly in the `JS` script, it should be loaded using the `Admin::script` method.

```php
<?php

use Illuminate\Contracts\Support\Renderable;
use Dcat\Admin\Admin;

class MyPage implements Renderable
{
    // Define the static resources needed for the page, which will be loaded here on demand.
	public static $js = [
	     // The js script cannot contain the initialization operation directly, otherwise the page refresh is invalid.
		'xxx/js/page.min.js',
	];
	public static $css = [
		'xxx/css/page.min.css',
	];

	public function script()
	{
		return <<<JS
		console.log('All JS scripts are loaded');
		// The initialization operation is written here.
JS;		
	}

	public function render()
	{
		// This is where you can bring in your js or css files.
		Admin::js(static::$js);
		Admin::css(static::$css);
		
		// JS code to be executed on the page
		// The JS code set by Admin::script will be executed automatically when all JS scripts are loaded.
		Admin::script($this->script());
		
		return view('admin.pages.my-page')->render();
	}
}
```

View `admin.pages.my-page`, make sure the view code does not contain tags such as `<body>` and `<html>`.
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

Usage

```php
public function index(Content $content)
{
    return $content->body(new MyPage());
}
```


### Example 2

The dashboard page `/admin` in the background can also be seen as a custom page, the code implementation is as follows
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

