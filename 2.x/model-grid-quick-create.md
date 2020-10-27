# 快捷创建


在表格中开启这个功能之后，会在表格头部增加一个`form`表单来创建数据，对于一些简单的表格页面，可以方便快速创建数据，不用跳转到创建页面操作

<a href="{{public}}/assets/img/screenshots/quick-create.png" target="_blank">
    <img  src="{{public}}/assets/img/screenshots/quick-create.png" style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%">
</a>


### 基本使用

> {tip} 需要注意的是，快捷创建表单中的每一项，在`form`表单页面要设置相同类型的表单项。

```php
$grid->quickCreate(function (Grid\Tools\QuickCreate $create) {
    $create->text('name', '名称');
    $create->email('email', '邮箱');
});
```

### 设置提交地址


```php
$grid->quickCreate(function (Grid\Tools\QuickCreate $create) {
    $create->action('auth/users');
    $create->method('GET');
});
```


表单支持的表单项有下面的几种类型


### 文本(text)
文本输入框

```php
$create->text('column_name', 'placeholder...');
```

### 隐藏表单(hidden)
文本输入框


```php
$create->hidden('column_name');
```

### 邮箱(email)
邮箱输入框
```php
$create->email('column_name', 'placeholder...');
```

### IP输入框
ip地址输入框

```php
$create->ip('column_name', 'placeholder...');
```

### URL输入框
url输入框
```php
$create->url('column_name', 'placeholder...');
```


### 密码(password)
密码输入框
```php
$create->password('column_name', 'placeholder...');
```

### 手机号(mobile)
手机号输入框
```php
$create->mobile('column_name', 'placeholder...');
```

### 整数(integer)
整形数字输入框
```php
$create->integer('column_name', 'placeholder...');
```

### 下拉选框(select)
单选框
```php
$create->select('column_name', 'placeholder...')->options([
    1 => 'foo',
    2 => 'bar',
]);
```


### 下拉选框多选(multipleSelect)
多选框
```php
$create->multipleSelect('column_name', 'placeholder...')->options([
    1 => 'foo',
    2 => 'bar',
]);
```

### 标签(tags)
```php
$create->tags('column_name', 'placeholder...')->options([
    1 => 'foo',
    2 => 'bar',
]);
```

### 弹窗选择器(selectResource)


通过`selectResource`表单可以构建一个弹窗选择器，可以从弹窗里面选择表格数据，并且支持数据筛选等操作。


```php
 $form->selectResource('user')
    ->path('auth/users') // 设置表格页面链接
    ->options(function ($v) { // 显示已选中的数据
        if (!$v) return $v;
        $userModel = config('admin.database.users_model');

        return $userModel::find($v)->pluck('name', 'id');
    });
    
// 设置为多选
 $form->selectResource('user')
    ->path('auth/users') 
    ->multiple() // 设置为多选
    ->options(function ($v) {
        ...
    });  
    
// 限制最大选择数量
 $form->selectResource('user')
    ->path('auth/users') 
    ->multiple(3) // 最多选择3个选项
    ->options(function ($v) {
        ...
    });       
```

然后设置你的路由`app/Admin/routes.php`

> {tip} 这里的添加路由只是示例，如果是新增的控制器需要加路由，如果路由已经存在则不需要再添加。

```php
$router->resource('auth/users', 'UserController');
```

`auth/users`页面实现代码如下：
```php
<?php

use Dcat\Admin\Models\Administrator;
use Dcat\Admin\IFrameGrid;
use Dcat\Admin\Grid;
use Dcat\Admin\Controllers\AdminController;

class UserController extends AdminController
{
    protected function iFrameGrid()
    {
        $grid = new IFrameGrid(new Administrator());
        
        // 指定行选择器选中时显示的值的字段名称
        // 指定行选择器选中时显示的值的字段名称
        // 指定行选择器选中时显示的值的字段名称
        // 如果表格数据中带有 “name”、“title”或“username”字段，则可以不用设置
        $grid->rowSelector()->titleColumn('username');

        $grid->id->sortable();
        $grid->username;
        $grid->name;
    
        $grid->filter(function (Grid\Filter $filter) {
            $filter->equal('id');
            $filter->like('username');
            $filter->like('name');
        });
    
        return $grid;
    }
}
```

效果

<a href="{{public}}/assets/img/screenshots/select-resource.png" target="_blank">
    <img  src="{{public}}/assets/img/screenshots/select-resource.png" style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%">
</a>


### 日期时间选择
时间日期输入框
```php
$create->datetime('column_name', 'placeholder...');
```

### 时间选择(time)
时间输入框
```php
$create->time('column_name', 'placeholder...');
```

### 日期选择
```php
$create->date('column_name', 'placeholder...');
```
