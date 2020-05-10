# 常见问题汇总

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

### 如何从laravel-admin迁移到dcat-admin？
[Dcat Admin 教程 - 如何从 Laravel admin 迁移到 dcat admin？](https://learnku.com/articles/44235)

### 重写登陆页面和登陆逻辑

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
	
	// 重写你的登陆页面逻辑
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


### 更新新版本后出现异常

如果遇到更新之后,部分组件不能正常使用,那有可能是`dcat-admin`自带的静态资源有更新了,需要运行命令`php artisan admin:publish --force`来重新发布前端资源，发布之后不要忘记清理浏览器缓存.

### 文件上传失败或无法访问？

如果你发现无法上传文件，那么通常有几下几点原因：

1. `Laravel`文件上传配置不正确，请参考文档[图片/文件上传](https://learnku.com/docs/dcat-admin/1.x/picture-file-upload/8106)。如果你不了解`laravel`文件上传功能，请阅读文档[Laravel - 文件存储](https://learnku.com/docs/laravel/7.x/filesystem/7485)
2. 文件过大，需要调整`php.ini`的`upload_max_filesize`参数
3. 文件上传目录没有写权限
4. `php`没有安装或没有开启`fileinfo`扩展

如果你的文件上传成功了，却无法正常访问，那么可能是`.env`配置文件中的`APP_URL`参数没有设置正确。

### 关于前端资源加载问题

`Dcat Admin`是支持前端资源按需加载的，在需要用到某个组件的时候再引入前端资源即可，开发者无需担心安装组件过多影响页面加载速度。

只有那种需要在全局页面引入的资源，才需要在`app/Admin/bootstrap.php`或`ServiceProvider::boot`方法中引入：

```php
Admin::css('path/to/your/css');
Admin::js('path/to/your/js');
```

### 谷歌字体加载过慢？

如果出现谷歌字体加载过慢的情况下，可以把谷歌字体下载到你自己的服务器，然后在`app/Admin/bootstrap.php`中加入以下代码，让系统从你自己的服务器中加载字体

```php
Admin::asset()->alias(
	'@nunito', 
	null, 
	asset('你的服务器字体路径/nunito.css?family=Nunito:200,200i,300,300i,400,400i,600,600i,800,800i,900,900i')
);
Admin::asset()->alias(
	'@montserrat', 
	null, 
	asset('你的服务器字体路径/montserrat.css?family=Montserrat:300,400,500,600')
);
```


如果你完全不想使用这两种字体，可以加入以下代码
```php
Admin::asset()->alias('@nunito', null, '');
Admin::asset()->alias('@montserrat', null, '');
```



### 图片防盗链
图片请求默认会去掉 `referer` 字段，如果有防盗链要求，可以在配置文件(`config/admin.php`)中设置：

```
 "disable_no_referrer_meta" => true
```

### 为何不开发成前后端分离项目？

最近有很多同学问我为什么不采用前后端分离技术方案，我在这个帖子里回答的非常详细，有相关疑问的同学请[点击此处查看帖子](https://github.com/jqhph/dcat-admin/issues/27)，这里不再赘述。