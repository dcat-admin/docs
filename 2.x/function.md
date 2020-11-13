# 帮助函数

### admin_color

获取内置颜色，关于主题颜色更多用法请参考[主题 - 颜色](theme.md#color)章节

```php
// 获取主题色的三种方式
$primary = admin_color('primary');
$primary = admin_color()->get('primary');
$primary = admin_color()->primary();

$color = admin_color();
$color->lighten('primary', 10);
```

### admin_js

可以在任意位置引入`js`文件，更多用法参考[静态资源](assets.md)章节

```php
admin_js(['@admin/xxx.js']);
```

### admin_css

可以在任意位置引入`css`文件，更多用法参考[静态资源](assets.md)章节

```php
admin_css(['@admin/xxx.css']);
```

### admin_require_assets

可以在任意位置引入静态资源组件，更多用法参考[静态资源](assets.md)章节

```php
admin_require_assets(['@datime']);
```


### admin_path

获取`Dcat Admin`安装的应用路径，默认目录是`app/Admin`：

```php
$bootstrap = admin_path('bootstrap.php');
```

### admin_url

获取`Dcat Admin`应用的路由完整url：

```php
// 返回： http://localhost/admin/auth/users
$url = admin_url('auth/users');
```

### admin_base_path

获取`Dcat Admin`应用的路由路径：
```php
// 返回： /admin/auth/users
$path = admin_base_path('auth/users');
```

### admin_toastr

在页面刷新后弹出一个`toastr`提示窗，参数：

- `$message` 提示窗内容
- `$type` 提示窗类型，默认`success`，支持`success`、`info`、`warning`、`error`
- `$options` toastr配置参数

```php
admin_alert('更新成功', 'success');
```

### admin_success

在页面刷新后在页面顶部显示一个成功消息：
```php
admin_success('标题', '成功了');
```

### admin_error

在页面刷新后在页面顶部显示一个错误消息：
```php
admin_error('标题', '失败了');
```

### admin_warning

在页面刷新后在页面顶部显示一个警告消息：
```php
admin_warning('标题', '警告');
```

### admin_info

在页面刷新后在页面顶部显示一个提示消息：
```php
admin_info('标题', '内容');
```

### admin_asset

获取静态资源的完整链接：

> {tip} 此函数支持别名.

```html
// 引入css
<link rel="stylesheet" href="{{ admin_asset("@admin/dcat-admin/main.min.css") }}">

// 引入js
<script src="{{ admin_asset('@admin/dcat-admin/main.min.js')}}"></script>
```

### admin_trans_field

翻译当前控制器的字段，控制器名称去除`Controller`后缀之后再转化为小写中划线就是语言包的名称，如：控制器名称为`UserProfileController`，则对应的语言包名称为`user-profile.php`。

> {tip} 如果当前控制器对应的语言包中不存在该字段翻译，则会去公共翻译文件`global.php`中查找。

```php
$name = admin_trans_field('name');
$createdAt = admin_trans_field('created_at');
```
语言包内容如下：
```php
return [
    'fields' => [
        'name' => '名称',
        'created_at' => '创建时间',
    ],
];
```


### admin_trans_label

翻译当前控制器的自定义内容，控制器名称去除`Controller`后缀之后再转化为小写中划线就是语言包的名称，如：控制器名称为`UserProfileController`，则对应的语言包名称为`user-profile.php`。

> {tip} 如果当前控制器对应的语言包中不存在该字段翻译，则会去公共翻译文件`global.php`中查找。

```php
$user = admin_trans_label('User');
```
语言包内容如下：
```php
return [
    'labels' => [
        'User' => '管理员',
    ],
];
```

### admin_trans_option

翻译当前控制器的字段选项值，控制器名称去除`Controller`后缀之后再转化为小写中划线就是语言包的名称，如：控制器名称为`UserProfileController`，则对应的语言包名称为`user-profile.php`。

> {tip} 如果当前控制器对应的语言包中不存在该字段翻译，则会去公共翻译文件`global.php`中查找。

```php
$status = admin_trans_option(1, 'status');
```
语言包内容如下：
```php
return [
    'options' => [
        'status' => [
            1 => '启用',
            0 => '禁用'
        ],
    ],
];
```

