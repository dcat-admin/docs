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

### 为何不开发成前后端分离项目？

最近有许多同学问我为什么不采用前后端分离方案？现在`vue`和`react`这么好用，为啥还要用`jQuery`呢？回答这个问题之前首先要搞清楚两种方案的优劣势

#### 前后端分离技术的优势与劣势
**优势**
1. 从用户体验角度来说，前后端分离项目用户体验会更好一点
2. 从技术角度而言，前后端分离技术现在也已是主流，采用这个方案也更适合我们提升自己的前端技术熟练度

**劣势**
1. 前后端分离技术的学习成本会更高，要求使用者必须有`vue`和`react`经验，否则就要花许多精力去学习才能上手
2. 前后端分离技术的引入会是项目功能扩展变得复杂和艰难，可能页面增加一个小功能，也需要单独打包前端资源，这无疑是给自己找麻烦
3. 这种前后端分离技术并不是真正意义的从团队到技术的前后端分离，也就是说如果你的前端不懂PHP代码，基本是无法接手维护你的前端代码的，其实所有工作还是需要后端一个人完成

#### jQuery技术方案的优势与劣势

**优势**

`jQuery`项目的优点就是简单易用，效率高，学习成本极低，上手快；
扩展功能也非常地简单和灵活，基本可以让开发者摆脱冗余的HTML代码。

**劣势**

`jQuery`项目从技术角度而言已经有点落伍，属于比较老旧的技术体系。

#### 总结
分析完上面的两种方案的优劣势，你们大概也明白我为啥要采用`jQuery`方案了，这个项目的定位就是一个对**后端开发友好**的项目，目的是要满足大多数人的需求，追求的就是**简单**、**易用**和**高效率**，而不是提升开发者的前端技术水平。

`jQuery`虽然不是前卫的技术，但是实用性并不差，前后端分离能实现的功能，`jQuery`也一样能实现。而且在这个项目里面，你还能更快、更省事、更容易上手。
如果你是想要提升前端技术，或是展示自己的前端技术水平，那么推荐去尝试下`Laravel nova`。

