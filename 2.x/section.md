# 区块（section）

## 简介

`Dcat Admin`提供`section`功能允许开发者在项目运行时更改页面各个部分的内容，而不需要直接去修改模板。

> {tip} `Dcat Admin`的`section`功能参考了`Blade`模板引擎的`@section`功能和`Wordpress`的`add_filter`功能，如果开发者理解这两者其中之一就能快速上手。

## 使用

### admin_section

输出`section`内容，此函数类似于`Blade`模板中的`@yield`指令以及`WordPress`中的`apply_filters`函数。

参数：
- `$section` {string} 区块名称
- `$default` {string|Closure} 默认值
- `$options` {array} 参数，需要注意的是`key`值必须使用英文字母开头，否则无法被获取到

返回值：
- {string}

```php
echo admin_section('navigation', null, ['count' => 4]);
```

### admin_inject_section

注入`section`内容，此函数类似`WordPress`中的`add_filter`函数。

参数：
- `$section` {string} 区块名称
- `$content` {string|Illuminate\Contracts\Support\Renderable|Closure} 区块内容
- `$append` {bool} 默认`true`，是否追加到上一个注入的内容后面，如果传`false`则会替换掉前面注入的内容
- `$priority` {int} 默认`10`，优先级，值越大排序越靠前

当第二个参数传入的是匿名函数时，匿名函数接收一个`Illuminate\Support\Fluent`对象。匿名函数中接收一个`Illuminate\Support\Fluent`对象，此对象包含有前面注入到此区块的内容，通过`previous`属性可以获得；如果此`section`还有其他参数，也可以通过访问属性的方式获得，如下：

```php
admin_inject_section('navigation', e("<navigation>1</navigation>"));

admin_inject_section('navigation', function ($options) {
    // 获取上一个注入此区块的内容
    $previous = $options->previous;
    
    // 获取自定义参数
    $count = $options->count;

    return e("<navigation>count:{$count}</navigation>");
}, true, 11);

// 输出
echo admin_section('navigation', null, ['count' => 4]);
// 最终输出结果为
// <navigation>count:4</navigation><navigation>1</navigation>
```

### admin_inject_default_section

注入默认内容，如果调用了`admin_inject_section`函数注入内容（无论是在前面还是后面都一样），则此函数不生效。

参数：
- `$section` {string} 区块名称
- `$content` {string|Illuminate\Contracts\Support\Renderable|Closure} 区块内容，与`admin_inject_section`的第二个参数一致

```php
admin_inject_default_section('navigation', '暂无数据');
```

### admin_has_section

判断是否注入过内容到`section`，此函数返回一个`bool`类型值。
```php
var_dump(admin_has_section('navigation'));
```

### admin_has_default_section

判断是否注入过默认内容到`section`，此函数返回一个`bool`类型值。
```php
var_dump(admin_has_default_section('navigation'));
```

## 系统预定义区块

`Dcat Admin`预定义了一些区块，开发者可以通过这些区块改变页面内容。

所有的预定义区块名称都定义在`Dcat\Admin\Admin::SECTION`这个类常量中，通过类常量的方式访问。

### 往&lt;head>标签内输入内容

此通过`Admin::SECTION['HEAD']`区块可以往`<head>`标签内输入内容。

在`app\Admin\bootstrap.php`中加入以下代码：
```php
use Dcat\Admin\Admin;

admin_inject_section(Admin::SECTION['HEAD'], function () {
    return '<script src="//oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>';
});
```

### 往&lt;body>标签内输入内容

通过`Admin::SECTION['BODY_INNER_BEFORE']`区块可以往`<body>`标签内部的开头位置输入内容。

通过`Admin::SECTION['BODY_INNER_AFTER']`区块可以往`<body>`标签内部的结束位置输入内容。


### 往&lt;div id="app">标签内输入内容

通过`Admin::SECTION['APP_INNER_BEFORE']`区块可以往`<div id="app">`标签内部的开头位置输入内容。

通过`Admin::SECTION['APP_INNER_AFTER']`区块可以往`<div id="app">`标签内部的结束位置输入内容。

### 更改顶部导航栏用户信息面板内容

通过`Admin::SECTION['NAVBAR_USER_PANEL']`区块可以更改顶部导航栏的用户信息面板内容。

```php
admin_inject_section(Admin::SECTION['NAVBAR_USER_PANEL'], view('admin::partials.navbar-user-panel'));
```

### 更改顶部导航栏前面内容

```php
admin_inject_section(Admin::SECTION['NAVBAR_BEFORE'], view('...'));
```
### 更改顶部导航栏后面内容

```php
admin_inject_section(Admin::SECTION['NAVBAR_AFTER'], view('...'));
```

### 更改顶部导航栏用户信息面板后面内容

通过`Admin::SECTION['NAVBAR_AFTER_USER_PANEL']`区块可以更改顶部导航栏的用户信息面板后面的内容。

```php
admin_inject_section(Admin::SECTION['NAVBAR_AFTER_USER_PANEL'], function () {
    return <<<HTML
    <li>
        <a href="#" data-toggle="control-sidebar"><i class="fa fa-gears"></i></a>
    </li>
HTML;    
});
```

### 更改菜单栏用户信息面板内容

通过`Admin::SECTION['LEFT_SIDEBAR_USER_PANEL']`区块可以更改菜单栏的用户信息面板的内容。
```php
 admin_inject_section(Admin::SECTION['LEFT_SIDEBAR_USER_PANEL'], view('admin::partials.sidebar-user-panel'));
```

### 更改菜单栏

通过`Admin::SECTION['LEFT_SIDEBAR_MENU']`可以更改整个菜单栏内容。
> {tip} `Dcat Admin`的菜单是通过注入默认内容到`LEFT_SIDEBAR_MENU`区块构建的，开发者可以替换掉系统默认的菜单渲染逻辑。

```php
use Dcat\Admin\Support\Helper;
use Dcat\Admin\Admin;

admin_inject_section(Admin::SECTION['LEFT_SIDEBAR_MENU'], function () {
    $menuModel = config('admin.database.menu_model');
	
	$builder = Admin::menu();

	$html = '';
	foreach (Helper::buildNestedArray((new $menuModel())->allNodes()) as $item) {
		$html .= view('admin::partials.menu', ['item' => $item, 'builder' => $builder])->render();
	}

	return $html;
});
```


