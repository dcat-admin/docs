# 表格字段翻译

数据表格中所有使用到字段的地方都会自动读取语言包中的翻译。

> {tip} 语言包的详细使用方法请参考<a>[多语言](trans.md)</a>。

### 语言包名称
语言包名称需要与控制器名相对应，假如控制器名`UserProfileController`，则对应的语言包为`resources/lang/{当前语言}/user-profile.php`（需要转化为小写中划线风格）。

如果想要更改语言包的名称，可以通过下面两种方式进行更改

方式1
```php
use Dcat\Admin\Http\Controllers\AdminController;

class UserController extends AdminController
{
    /**
     * 指定翻译文件名称
     * 
     * @var string 
     */
    protected $translation = 'user1';
    
    ...
}
```

方式2
```php
use Dcat\Admin\Admin;

Admin::translation('user1');
```




### 示例
现在假设语言包`resources/lang/zh_CN/user-profile.php`内容如下：
```php
return [
    'fields' => [
        'name'  => '名称',
        'age'   => '年龄',
        'class' => '班级',
    ],
];
```

控制器`UserProfileController`中设置的`Grid`字段会自动读取以上翻译：
```php
// 不设置labael会自动读取语言包翻译
$grid->id();
$grid->name;
$grid->age;
$grid->class;

$grid->filter(function ($filter) {
    $filter->gt('age');
});

// 上面代码等同于
$grid->name('名称');
$grid->age('年龄');

// 也可这样使用
$grid->id(admin_trans_field('id'));
$grid->name(admin_trans_field('name'));
$grid->age(admin_trans_field('age'));

```

### 公共翻译
当`admin_trans_field`函数找不到当前控制器中对指定字段的翻译时，会去`global.php`中查找。如果某些字段是很多数据表中都有的，可以把这些翻译写在`resources/lang/{当前语言}/global.php`文件中。
```php
return [
    // 常用的字段放在 global.php 中可以所有控制器共用。
    'fields' => [
        'id'         => 'ID',
        'created_at' => '创建时间',
        'updated_at' => '更新时间',
    ],
];
```

