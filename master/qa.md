# 常见问题汇总


### 为何不开发成前后端分离项目？

最近有许多同学问我为什么不采用前后端分离方案？现在`vue`和`react`这么好用，为啥还要用`jQuery`呢？

回答这个问题之前首先要搞清楚两种方案的优劣势：


1. 前后端分离技术的优势


这个问题我帖子里面说了啊。因为这个项目就是为了对后端开发人员友好而设计的，前后端分离比较小众，而且这样做就是后端人员自己折腾的纯技术角度的前后端分离，并不是那种真正意义上的从团队到技术的前后端分离，前端程序员如果不懂PHP基本是接手不了你的前端工作的，你可以参考下nova，就是现在这个模式

### 如何设置语言为简体中文？

打开配置文件`config/app.php`，设置`locale`参数的值为`zh-CN`。

### Laravel7时间显示为UTC格式

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

### 更新新版本后出现异常

如果遇到更新之后,部分组件不能正常使用,那有可能是`dcat-admin`自带的静态资源有更新了,需要运行命令`php artisan admin:publish --force`来重新发布前端资源，发布之后不要忘记清理浏览器缓存.


### 无法上传文件或图片

如果出现无法上传文件或图片的问题，大概率是因为没有配置好文件系统的配置，具体请参考[图片/文件上传](model-form-upload.md)。


### 关于前端资源加载问题

`Dcat Admin`是支持前端资源按需加载的，在需要用到某个组件的时候再引入前端资源即可，开发者无需担心安装组件过多影响页面加载速度。

只有那种需要在全局页面引入的资源，才需要在`app/Admin/bootstrap.php`或`ServiceProvider::boot`方法中引入：

```php
Admin::css('path/to/your/css');
Admin::js('path/to/your/js');
```

