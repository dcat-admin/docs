# 下拉菜单

通过`Dcat\Admin\Widgets\Dropdown`这个类可以快速帮大家构建下拉菜单功能。


### 基本用法

```php
<?php

use Dcat\Admin\Widgets\Dropdown;
use Dcat\Admin\Layout\Content;

class MyController
{
    public function index(Content $content)
    {
        $options = [
            '名称1',
            '名称2',
            '名称3',
            '名称4',
            '名称5',
        ];
        
        $dropdown = Dropdown::make($options)
            ->button('分类导航') // 设置按钮
            ->buttonClass('btn btn-white  waves-effect') // 设置按钮样式
            ->map(function ($label, $key) {
                // 格式化菜单选项
                $url = admin_url('categories/'.$key);

                return "<a href='$url'>{$label}</a>";
            });
         
        return $content->body($dropdown);
    }
}
```
效果

<a href="{{public}}/assets/img/screenshots/dropdown-1.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/dropdown-1.png"  style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="70%" >
</a>

### 点击菜单更换按钮文本

`click`方法可以让选中的菜单文本显示在按钮中，做到类似下拉选框的效果。

```php
$options = [
    ...
];

$dropdown = Dropdown::make($options)
    ->button('选择') // 设置按钮
    ->click();
```

### 设置标题

```php
$options1 = [
    '名称1',
    '名称2',
];

$options2 = [
    '测试1',
    '测试2',
];

$dropdown = Dropdown::make()
    ->button('使用标题')
    ->options($options1, '标题1')
    ->options($options2, '标题2');
```
效果

<a href="{{public}}/assets/img/screenshots/dropdown-2.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/dropdown-2.png"  style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="70%" >
</a>

### 增加分割线

```php
$options = [
    '名称1',
    '名称2',
    Dropdown::DIVIDER,
    '名称3',
    '名称4',
];

$dropdown = Dropdown::make()
    ->button('使用分割线')
    ->options($options)
```
效果

<a href="{{public}}/assets/img/screenshots/dropdown-3.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/dropdown-3.png"  style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="70%" >
</a>

### 自定义按钮

```php
    public function index(Content $content)
    {
        $options = [
            '名称1',
            '名称2',
            '名称3',
            '名称4',
            '名称5',
        ];
        
        $dropdown = Dropdown::make($options)
            ->map(function ($label, $key) {
                // 格式化菜单选项
                $url = admin_url('categories/'.$key);

                return "<a href='$url'>{$label}</a>";
            });
         
        return $content->body(
            <<<HTML
<div class='dropdown'>
     <button class='btn btn-primary dropdown-toggle' data-toggle='dropdown'>
        <i class='feather icon-email'></i> 自定义按钮 
     </button>
     {$dropdown->render()}
</div>            
HTML            
        );
    }
```

效果

<a href="{{public}}/assets/img/screenshots/dropdown-4.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/dropdown-4.png"  style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="70%" >
</a>
