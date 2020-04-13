# 常见问题汇总


### 如何设置语言为简体中文？

打开配置文件`config/app.php`，设置`locale`参数的值为`zh-CN`。

### Laravel7出现时间显示为UTC格式

这个是`Laravel7`升级后带来的坑，原因请参考[日期序列化](https://learnku.com/docs/laravel/7.x/upgrade/7445#date-serialization)。

在本项目中解决这个问题很简单，只需在你的`Model`中引入`Dcat\Admin\Traits\HasDateTimeFormatter`这个`trait`即可。

```php
<?php

namespace App\Models;

use Dcat\Admin\Traits\HasDateTimeFormatter;
use Illuminate\Database\Eloquent\Model;

class MyModel extends Model
{
    use HasDateTimeFormatter;
}
```


### 关于前端资源问题

`Dcat Admin`是支持前端资源按需加载的，开发者无需担心安装组件过多影响页面加载速度。
开发者在需要用到某个组件的时候再引入前端资源即可，如果需要引入全局可用的前端资源，可以在`app/Admin/bootstrap.php`中引入：

```php
Admin::css('path/to/your/css');
Admin::js('path/to/your/js');
```


### 重写登陆页面和登陆逻辑

在路由文件`app/Admin/routes.php`中，覆盖掉登陆页面和登陆逻辑的路由，即可实现自定义的功能

```php
Route::group([
    'prefix'        => config('admin.prefix'),
    'namespace'     => Admin::controllerNamespace(),
    'middleware'    => ['web', 'admin'],
], function (Router $router) {

    $router->get('auth/login', 'AuthController@getLogin');
    $router->post('auth/login', 'AuthController@postLogin');
    
});

```

在自定义的路由器AuthController中的`getLogin`、`postLogin`方法里分别实现自己的登陆页面和登陆逻辑。

参考控制器文件[AuthController.php](https://github.com/z-song/dcat-admin/blob/master/src/Controllers/AuthController.php)，视图文件[login.blade.php](https://github.com/z-song/dcat-admin/blob/master/views/login.blade.php)

### 更新静态资源

如果遇到更新之后,部分组件不能正常使用,那有可能是`dcat-admin`自带的静态资源有更新了,需要运行命令`php artisan vendor:publish --tag=dcat-admin-assets --force`来重新发布前端资源，发布之后不要忘记清理浏览器缓存.