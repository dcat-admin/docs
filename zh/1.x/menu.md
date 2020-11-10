# 菜单

`Dcat Admin`的菜单是保存在数据表`admin_menu`上的，开发者可以在后台菜单管理页面对菜单进行管理。

### 菜单权限
每个菜单都可以与权限或角色进行绑定，如果不设置则为公共菜单，所有账号都能看到。

通过`admin.menu.bind_permission`配置参数可以设置是否允许绑定权限。
> {tip} 默认一个菜单最多能绑定一个权限和一个角色。

### 菜单翻译
在您的语言文件的menu_titles索引中追加菜单标题。 例如“工作单位”标题：

在`resources/lang/{当前语言}/admin.php`中
```php
...
'menu_titles' => [
    'work_units' => 'Unidades de trabajo'
],
```

### 菜单缓存
通过`admin.menu.cache.enable`配置参数可以开启或关闭菜单缓存，建议开启。

### 通过Menu::add接口动态添加菜单
`Dcat Admin`还提供了通过数组的方式在代码中即时添加菜单。

在`app\Admin\bootstrap.php`中添加如下代码：
```php
<?php
use Dcat\Admin\Admin;
use Dcat\Admin\Layout\Menu;

Admin::menu(function (Menu $menu) {
    $menu->add([
        [
            'id'            => '1', // 此id只要保证当前的数组中是唯一的即可
            'title'         => '测试菜单',
            'icon'          => 'fa-file-text-o',
            'uri'           => '',
            'parent_id'     => 0, 
            'permission_id' => 'test', // 与权限绑定
            'roles'         => 'test-roles', // 与角色绑定
        ],  
        [
            'id'            => '2', // 此id只要保证当前的数组中是唯一的即可
            'title'         => '测试菜单2',
            'icon'          => 'fa-file-text-o',
            'uri'           => 'test-menu2',
            'parent_id'     => '1', 
        ],  
    ]);
});

```

### 为何没有权限的菜单不会自动隐藏？

这个问题是因为你没有给菜单绑定权限或者角色，给你想要无权不显示的菜单绑定权限或者角色即可。


