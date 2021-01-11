# 常见问题汇总

### 1.如何设置语言为简体中文？

打开配置文件`config/app.php`，设置`locale`参数的值为`zh_CN`。

### 2.Laravel7时间显示为UTC格式

这个是`Laravel7`升级后带来的坑，原因请参考[日期序列化](https://learnku.com/docs/laravel/7.x/upgrade/7445#date-serialization)。

在本项目中解决这个问题很简单，只需在`Model`中引入`Dcat\Admin\Traits\HasDateTimeFormatter`这个`trait`即可。

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

### 3.表单保存时报错`Array to string conversion`

出现这个问题是因为表单提交的值最后转换成了`array`类型，而`MySQL`是不支持直接存储`array`类型数据的，在`dcat-admin`中可以用以下方式对数据格式进行转换

```php
$form->multipleSelect('user_id')->saving(function ($v) {
    // 转为 , 隔开的字符串
    return implode(',', $v);
});
```

当然，也可以通过`model`的**修改器**去转化字段的值，这方面内容可以参考`laravel`文档，这里就不再赘述。

> {tip} 更优雅的转化值方法，可参考 [Dcat Admin 教程 - 如何优雅地更改表单值的数据类型？](https://learnku.com/articles/44386)


### 4.如何从laravel-admin迁移到dcat-admin？
[Dcat Admin 教程 - 如何从 Laravel admin 迁移到 dcat admin？](https://learnku.com/articles/44235)

### 5.重写登陆页面和登陆逻辑

方式一，重写登陆控制器方法：

默认的登陆控制器用的是`App\Admin\AuthController`这个类，可以通过配置参数`admin.auth.controller`进行修改

```php
<?php

namespace App\Admin\Controllers;

use Dcat\Admin\Controllers\AuthController as BaseAuthController;

class AuthController extends BaseAuthController
{
    // 自定义登陆view模板
    protected $view = 'admin.login';
	
	// 重写登陆页面逻辑
	public function getLogin(Content $content)
    {
        ...
    }

    ...
}

```


方式二，覆写路由：

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


### 6.更新新版本后出现异常

如果遇到更新之后,部分组件不能正常使用,那有可能是`dcat-admin`自带的静态资源有更新了,需要运行命令`php artisan admin:publish --force`来重新发布前端资源，发布之后不要忘记清理浏览器缓存.

### 7.文件上传失败或无法访问？

如果你发现无法上传文件，那么通常有几下几点原因：

1. `Laravel`文件上传配置不正确，请参考文档[图片/文件上传](https://learnku.com/docs/dcat-admin/1.x/picture-file-upload/8106)。如果你不了解`laravel`文件上传功能，请阅读文档[Laravel - 文件存储](https://learnku.com/docs/laravel/7.x/filesystem/7485)
2. 文件过大，需要调整`php.ini`的`upload_max_filesize`参数
3. 文件上传目录没有写权限
4. `php`没有安装或没有开启`fileinfo`扩展
5. 检查`php.ini`的`upload_tim_dir`参数设置是否正常

如果文件上传成功了，却无法正常访问，那么可能是`.env`配置文件中的`APP_URL`参数没有设置正确。

### 8.关于前端资源加载问题

`Dcat Admin`是支持前端资源按需加载的，在需要用到某个组件的时候再引入前端资源即可，开发者无需担心安装组件过多影响页面加载速度。

只有那种需要在全局页面引入的资源，才需要在`app/Admin/bootstrap.php`或`ServiceProvider::boot`方法中引入：

```php
Admin::css('path/to/your/css');
Admin::js('path/to/your/js');
```

### 9.为何配置了角色和权限，依然提示无权访问？

这个原因可能是由于权限的`URL`路径配置错误导致的，正确的包含增删改查功能的`URL`配置应该是`auth/users*`这样的，如果配置成了`auth/users/*`，那么就会提示无权访问。

> {tip} 另外标签表单填写自定义URL有两种方法：一种是选中后按`删除键`进行更改；另一种是填写后按`空格键` + `回车键`。

### 10.为何没有权限的菜单不会自动隐藏？

这个问题是因为没有给菜单绑定权限或者角色，给想要无权不显示的菜单绑定权限或者角色即可。


### 11.项目使用HTTPS之后无法登陆

需要把配置文件的`admin.https`参数的值设置为`true`


### 12.$.get(xxx) 没有反应

`Dcat Admin`使用的是`jQuery3.x`，`$.get`方法在`jQuery3.x`中已经被废弃，请使用`$.ajax`代替

### 13.上传的图片经常莫名其妙自动删除？

出现这个问题的原因很可能是因为`Model`中设置了图片相关字段的访问器，如

```php
class User extend Model
{
    public function getAvatarAttribute()
    {
        return Storage::disk('admin')->url($this->avatar);
    }
}
```

这种情况下会出现自动删除已上传图片的情况，解决方法也很简单，把访问器改成自定义方法即可

```php
class User extend Model
{
    public function getAvatar()
    {
        return Storage::disk('admin')->url($this->avatar);
    }
}
```

###14.前后台session发生冲突

从`2.0`的版本之后 `admin.session` 中间件不再默认启用，如果您的应用同时有前台和后台，则需要开启 `admin.session` 中间件，否则会造成前后台 `session` 冲突问题。

把配置参数 `admin.route.enable_session_middleware` 的值设置为 `true` 即可开启
```php
    'route' => [
        'domain' => env('ADMIN_ROUTE_DOMAIN'),

        'prefix' => env('ADMIN_ROUTE_PREFIX', 'admin'),

        'namespace' => 'App\\Admin\\Controllers',

        'middleware' => ['web', 'admin'],
        
        // 开启 admin.session 中间件
        'enable_session_middleware' => true,
    ],
```

### 15.图片防盗链
图片请求默认会去掉 `referer` 字段，如果有防盗链要求，可以在配置文件(`config/admin.php`)中设置：

```
 "disable_no_referrer_meta" => true
```
