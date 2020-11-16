# 视图与自定义页面

## 视图


在`Dcat Admin`中我们可以用`admin_view`函数渲染视图，这个功能借鉴了`vue`的设计思想，可以把`HTML`、`CSS`和`JS`代码写在同一个模板文件中，让代码分层更清晰更简洁易读，如

```html
<div class="my-class">...</div>

<style>
	.my-class {
		color: blue;
	}
</style>

<script require="@test1,@test2" init=".my-class">
	$this.css({background: 'red'})
</script>
```

在`php`中渲染这个视图
```php
public function index(Content $content)
{
	return $content->body(admin_view('...'));
}
```

#### 示例解析 

上面示例中的代码，其实相当于下面的代码

```html
<div class="my-class">...</div>
```

```php
public function index(Content $content)
{
	admin_require_assets(['@test1', '@test2']);
	
	admin_style('.my-class {
		color: blue;
	}');
	
	admin_script(
		<<<JS
Dcat.init('.my-class', function (\$this, id) {
	\$this.css({background: 'red'})
});
JS
	);

	return $content->body(view('...'));
}
```

很显然，使用`admin_view`渲染视图会让你的代码更简洁易读，关于`Dcat.init`以及`script`标签中的`init`和`require`属性的用法，请参考文档[静态资源](assets.md)以及[动态监听元素生成 (init)](js.md#init)章节。

## 自定义页面

在`Dcat Admin`中构建自定义页面非常简单，可以参考如下两个例子


### 示例1

> {tip} `Dcat Admin`构建的是一个单页应用，加载的`JS`脚本只会执行一次，所以初始化操作不能直接放在`JS`脚本中，应该使用`Admin::script`方法载入。

```php
<?php

namespace App\Admin\Pages;

use Illuminate\Contracts\Support\Renderable;

class MyPage implements Renderable
{
	public function render()
	{
		return admin_view('admin.pages.my-page');
	}
}
```

视图`admin.pages.my-page`，注意视图代码里面不要包含`<body>`和`<html>`等标签
```html
<div class="my-class">
  <h3>自定义页面演示</h3>
</div>

<!-- 
 	引入页面所需的静态资源，这里会按需加载
	js脚本不能直接包含初始化操作，否则页面刷新后无效 
-->
{!! admin_js(['xxx/js/page.min.js']) !!}
{!! admin_css(['xxx/js/page.min.css']) !!}

<script init=".my-class">
    // js代码也可以放在模板里面
    console.log('所有JS脚本都加载完了!!!');
    
    $this.on('click', function () {
        ...
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

