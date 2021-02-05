# 开发扩展


`Dcat Admin`支持安装扩展工具来帮助丰富你的后台功能。
>需要注意的是，`Laravel Admin`原有的扩展无法直接在`Dcat Admin`中使用，但大部分扩展只需要做一些微小的调整就可以正常使用了，有兴趣的同学可以自行移植。

如果大家在使用的过程中有在`Dcat Admin`的基础上添加一些自己的功能或者组件，不妨做成一个`Dcat Admin`扩展，这样可以给其它`Dcat Admin`使用者提供帮助，并且在其它人的使用反馈中的提升扩展的质量。

这篇文档将会以开发一个 [操作日志扩展](https://github.com/dcat-admin/operation-log) 为例，一步一步的教大家开发一个扩展，并且发布给他人使用，最终的效果参考[操作日志扩展](https://github.com/dcat-admin/operation-log)。


## 开始之前

在开始开发扩展之前，如果是`linux`环境，请先手动在项目根目录创建一个`dcat-admin-extensions`目录，并设置可读**可写**权限，扩展文件将会被安装在`dcat-admin-extensions`目录，请保证拥有以下几个目录的读写权限

1. `项目目录/dcat-admin-extensions` 扩展的安装目录，可以根据配置参数 `admin.extensions.dir` 进行更改
2. `public/vendor` 扩展静态资源发布目录
3. `resources/lang` 语言包目录


## 1.创建扩展

`Dcat Admin`的扩展是一个标准的`composer`扩展包，可以通过`composer`安装，也可以通过系统内部的`本地安装`功能直接安装。我们可以通过命令或界面创建一个新的扩展，下面分别简单介绍一下命令和界面创建的方法


1.通过命令创建扩展 

```bash
php artisan admin:ext-make 扩展包名 --namespace=命名空间 --theme
```

命令参数说明

- `name` 扩展包名称，扩展的名称是一个标准的`composer`包名，请统一使用**小写字母** + **中划线(-)**的风格命名，标准的格式如 `dcat-admin/operation-log`，前面一部分可以是个人名称，后面一部分可以是对扩展包功能的概括词语
- `--namespace=` 扩展包命名空间，默认会根据包名自动生成，例如你的包名是`jiangqh/operation-log`，那么默认的命名空间是`Jiangqh/OperationLog`
- `--theme` 是否为主题扩展

那么在当前这个例子中我们运行一下命令生成扩展包

```php
# `--namespace`
php artisan admin:ext-make dcat-admin/operation-log --namespace="Dcat\Admin\OperationLog"
```

2.通过管理页面创建扩展

打开扩展管理页面`http://localhost/admin/auth/extensions`，然后点击表格下面第一行的**快速创建**，然后输入扩展包名和命名空间即可，在实际开发中也更推荐大家使用界面创建扩展，这样更方便


扩展创建完成之后可以看到扩展文件夹下多了个`dcat-admin/extensions/dcat-admin/operation-log`目录，目录结构如下
```php
├── README.md
├── composer.json # composer配置文件
├── version.php   # 扩展包版本管理文件
├── logo.png      # logo
├── updates       # 扩展包每个版本对应的表迁移文件
├── resources 
│   ├── lang    # 语言包
│   ├── assets  # 静态资源
│   │   ├── css
│   │   │   └── index.css # css示例文件
│   │   └── js
│   │       └── index.js  # js示例文件
│   └── views
│       └── index.blade.php # 视图示例文件
└── src
    ├── OperationLogServiceProvider.php # 扩展包服务提供者
    ├── Setting.php  # 扩展设置表单
    ├── Models  # 模型目录
    └── Http
        ├── routes.php  # 扩展包路由文件
        ├── Middleware  # 扩展包中间件文件夹
        └── Controllers # 扩展包控制器
            └── OperationLogController.php
```

然后你还可以设置扩展的`logo`以及扩展名称，设置之后会在扩展管理页面展示，让你的扩展更具容易被人记住！

<a name="logo"></a>
### 扩展logo

扩展`logo`必须放在扩展的根目录，并且文件名必须是`logo.png`，尺寸建议是`100x100`。

### 扩展名称

扩展名称需要修改`composer.json`里面的`alias`参数，如果不设置则默认展示包名


## 2.启用扩展

扩展创建成功之后就可以在管理页面`http://localhost/admin/auth/extensions` 看到新创建的扩展了，效果如下

<a href="{{public}}/assets/img/2x/ext-1.png" target="_blank">
    ![]({{public}}/assets/img/2x/ext-1.png)
</a>

然后我们分别点击扩展对应的 `更新至1.0.0版本` 以及 `启用` 按钮，就可以使这个扩展生效了。
新创建的扩展会生成一个默认的控制器，在这个例子中，我们可以尝试访问`http://localhost:8000/admin/operation-log`，如果可以正常访问则说明扩展启用成功。


## 3.功能开发

这个扩展的功能主要是用来记录用户的操作记录，然后提供一个查看记录的页面，然后我们可以把默认创建的扩展文件中用不到的给清理掉，清理后的目录结构如下
```php
├── README.md
├── composer.json # composer配置文件
├── version.php   # 扩展包版本管理文件
├── logo.png      # logo
├── updates       # 扩展包每个版本对应的表迁移文件
├── resources 
│   └── lang  # 语言包
└── src
    ├── OperationLogServiceProvider.php # 扩展包服务提供者
    ├── Setting.php  # 扩展设置表单
    ├── Models  # 模型目录
    └── Http
        ├── routes.php  # 扩展包路由文件
        ├── Middleware  # 扩展包中间件文件夹
        └── Controllers # 扩展包控制器
            └── OperationLogController.php
```

接下来就正式进入功能开发部分


### 创建迁移文件 (migration)
首先我们需要创建一个表迁移文件，运行命令 `php artisan make:migration CreateOperationLogTable`，然后写入文件内容如下

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateOperationLogTable extends Migration
{
    // 这里可以指定你的数据库连接
    public function getConnection()
    {
        return config('database.connection') ?: config('database.default');
    }

    public function up()
    {
        if (! Schema::hasTable('admin_operation_log')) {
            Schema::create('admin_operation_log', function (Blueprint $table) {
                $table->bigIncrements('id')->unsigned();
                $table->bigInteger('user_id');
                $table->string('path');
                $table->string('method', 10);
                $table->string('ip');
                $table->text('input');
                $table->index('user_id');
                $table->timestamps();
            });
        }
    }

    public function down()
    {
        Schema::dropIfExists('admin_operation_log');
    }
}
```

然后把文件移动到 `扩展目录/updates` 目录，并重命名为 `create_opration_log_table.php`。最后我们需要修改扩展的版本管理文件 `扩展目录/version.php`，写入迁移文件的文件名，最终内容如下
```php
<?php

return [
    '1.0.0' => [
        '版本变化描述1',
        '版本变化描述2',
        'create_opration_log_table.php', // 迁移文件名称，安装或更新版本时会自动执行
    ],
];
```

关于扩展的版本管理更详细的介绍，请前往[扩展 - 版本管理](extension-f.md#version)章节查看。

### 模型、控制器和路由

创建模型文件 `扩展目录/src/Models/OperationLog`，模型内容点击[OperationLog.php](https://github.com/dcat-admin/operation-log/blob/master/src/Models/OperationLog.php)查看；

然后修改我们的控制器`扩展目录/src/Http/Controllers/OperationLogController.php`，控制器内容点击[LogController.php](https://github.com/dcat-admin/operation-log/blob/master/src/Http/Controllers/LogController.php)查看；

最后需要修改我们的路由文件，尽量让你的路由不与其他路由产生冲突
```php
use Dcat\Admin\OperationLog\Http\Controllers;
use Illuminate\Support\Facades\Route;

Route::get('auth/operation-logs', Controllers\OperationLogController::class.'@index')->name('dcat-admin.operation-log.index');
Route::delete('auth/operation-logs/{id}', Controllers\OperationLogController::class.'@destroy')->name('dcat-admin.operation-log.destroy');
```

### 语言包

在控制器中，我们可以让一些文本描述支持语言包翻译功能，在这个例子中我们以`en`以及`zh_CN`两种语言为例，在`扩展包/resources/lang`目录下分别创建`en/log.php`和`zh_CN/log.php`文件，并写入以下内容

```php
// en
return [
    'title' => 'Operation Log',
    'setting_title' => 'Operation Log',
];

// zh_CN
return [
    'title' => '操作日志',
    'setting_title' => '操作日志',
];
```

最后在控制器中可以通过下面的方法访问语言包内容，关于多语言的更多用法可以参考laravel官方文档

```php
use Dcat\Admin\OperationLog\OperationLogServiceProvider;

OperationLogServiceProvider:trans('log.title');
OperationLogServiceProvider:trans('log.setting_title');
```

### 定义菜单

接下来我们还需要给我们的扩展生成菜单，打开`扩展目录/src/OperationLogServiceProvider.php`，然后修改内容如下

```php
class OperationLogServiceProvider extends ServiceProvider
{
    // 定义菜单
    protected $menu = [
        [
            'title' => 'Operation Log',
            'uri'   => 'auth/operation-logs',
            'icon'  => '', // 图标可以留空
        ],
    ];

    public function settingForm()
    {
        return new Setting($this);
    }
}
```

如果你想注册带有层级关系的菜单，可以通过下面这种方式注册

```php
// 注册菜单
protected $menu = [
    [
        'title' => 'Operation Log',
        'uri'   => '',
        'icon'  => 'feather icon-x', 
    ],
    [
        'parent' => 'Operation Log', // 指定父级菜单
        'title'  => 'List',
        'uri'    => 'auth/operation-logs',
    ],
];
```

### 测试扩展

上面的步骤都完成之后，我们可以先开始测试一下上面的功能，验证下是否有错误，然后再进行后续的开发。


由于刚开始创建扩展的时候，我们已经安装并启用了这个扩展，所以此处我们要先**卸载**当前扩展，再重新更新到`1.0.0`版本，这样数据表和菜单才会被创建。

> {tip} **卸载**功能会删除扩展的数据或数据表，请谨慎操作，以免造成数据丢失！！！

打开扩展管理页面`http://域名/admin/auth/extensions`，找到当前扩展，鼠标移动到扩展行，点击 `卸载` 按钮并确认，然后重新点击`更新至1.0.0版本` 以及 `启用` 按钮，
最后 `F5` 刷新浏览器即可看到新创建的菜单，点击菜单可访问操作日志管理页面`admin/auth/operation-logs`。


### 注册中间件

现在我们的扩展还需要一个中间件来记录用户的操作，创建文件 `扩展目录/src/Http/Middleware/LogOperation.php`，文件内容点击[LogOperation.php](https://github.com/dcat-admin/operation-log/blob/master/src/Http/Middleware/LogOperation.php)查看。

然后我们需要注册这个中间件，让这个中间件生效，打开`扩展目录/src/OperationLogServiceProvider.php`，然后修改内容如下

```php
class OperationLogServiceProvider extends ServiceProvider
{
    protected $middleware = [
        'middle' => [ // 注册中间件
            LogOperation::class,
        ],
    ];

    protected $menu = [
        [
            'title' => 'Operation Log',
            'uri'   => 'auth/operation-logs',
        ],
    ];

    public function settingForm()
    {
        return new Setting($this);
    }
}
```

`$middleware`属性中注册的中间件最后会合并到配置参数`admin.route.middleware`中，中间件注册支持以下三种类型注册

1. `before` 中间件会在最前面，也就是最先执行
2. `middle` 中间件会在`admin.auth`（登陆鉴权中）和`admin.permission`（权限判断）两个中间件**之间**执行
3. `after`  中间件会在最后执行

在这个例子中，操作日志记录用户操作时显然需要记录登陆用户的信息，所以中间件必须在 `admin.auth` 中间件之后执行，这样才能拿到登陆用户数据；
并且无权限的操作也需要记录，所以必须在 `admin.permission` 中间件之前执行，所以只有注册为 `middle` 类型的中间件才能满足上述要求！

注册完中间件之后，我们随意访问一下系统中的其他页面（除了操作日志管理页面），然后再访问操作日志管理页面，就可以看到用户的操作日志了，到这里插件基本开发完毕。

### 配置参数（设置）

在当前这个例子中，我们需要让用户能配置一些自定义参数（比如配置不需要记录操作日志的路由），所以我们还需要一个 `配置表单` 让用户能通过页面直接配置相关参数，
我们需要在 `OperationLogServiceProvider` 类里面的 `settingForm` 返回这个配置表单的对象

```php
class OperationLogServiceProvider extends ServiceProvider
{
    ...

    // 返回配置表单对象，如果不需要保存配置参数，则请删除这个方法  
    public function settingForm()
    {
        return new Setting($this);
    }
}
```

然后我们需要修改配置表单类 `扩展目录/src/Setting.php` 如下

```php
namespace Dcat\Admin\OperationLog;

use Dcat\Admin\Extend\Setting as Form;
use Dcat\Admin\OperationLog\Models\OperationLog;
use Dcat\Admin\Support\Helper;

class Setting extends Form
{
    // 返回表单弹窗标题
    public function title()
    {
        return $this->trans('log.title');
    }

    // 格式化待保存的配置参数值
    protected function formatInput(array $input)
    {
        // 转化为数组，注意如果这里保存的时候是数组，那么读取出来的时候也是数组
        $input['except'] = Helper::array($input['except']);
        $input['allowed_methods'] = Helper::array($input['allowed_methods']);

        return $input;
    }

    public function form()
    {
        // 定义表单字段
        $this->tags('except');
        $this->multipleSelect('allowed_methods')
            ->options(array_combine(OperationLog::$methods, OperationLog::$methods));
        $this->tags('secret_fields');
    }
}
```

以上设置完成之后我们就可以在扩展管理页面保存自定义参数了

<a href="{{public}}/assets/img/2x/ext-2.png" target="_blank">
    ![]({{public}}/assets/img/2x/ext-2.png)
</a>

配置参数读取用法如下，我们可以在中间件 `LogOperation` 中使用这些参数

```php
use Dcat\Admin\OperationLog\OperationLogServiceProvider;

// 读取配置参数
$except = OperationLogServiceProvider::setting('except');
$allowedMethods = OperationLogServiceProvider::setting('allowed_methods');
$secretFields = OperationLogServiceProvider::setting('secret_fields');
```

### 服务注册与初始化

由于当前这个例子中没有使用到服务注册与初始化相关功能，所以这部分内容先略过，有相关需要的同学可以参考[扩展 - 服务注册与初始化](extension-f.md#service)章节。

### 视图 (view)

由于当前这个例子中没有使用到自定义静态资源的功能，所以这部分内容先略过，有相关需要的同学可以参考[扩展 - 视图](extension-f.md#view)章节。



### 静态资源

由于当前这个例子中没有使用到自定义静态资源的功能，所以这部分内容先略过，有相关需要的同学可以参考[扩展 - 静态资源](extension-f.md#assets)章节。


### 修改 composer.json & README.md

代码部分完成之后，需要修改`composer.json`里面的内容，将`description`、`keywords`、`license`、`authors`等内容替换为你的信息，然后不要忘记完善`README.md`，补充使用文档等相关信息。


### 发布扩展


#### 上传应用市场

正式版发布后会上线应用市场功能，开发者可以把扩展发布到应用市场，然后用户就可以通过页面直接安装使用，敬请期待...


<a name="github"></a>
#### 上传到Github
先登录你的Github，创建一个仓库，然后按照页面上的提示把你的代码push上去

```
git init
git remote add origin https://github.com/<your-name>/<your-repository>.git
git add .
git commit -am "Initial commit."
git push origin master
```

<a name="packagist"></a>
#### 发布到Packagist.org
接下来就是发布你的项目到`Packagist.org`，如果没有账号的话，先注册一个，然后打开顶部导航的`Submit`, 填入仓库地址提交

默认情况下，当您推送新代码时，`Packagist.org`不会自动更新，所以，您需要创建一个GitHub服务钩子， 你也可以使用点击页面上的`Update`按钮手动更新它，但我建议自动执行这个过程

提交之后，由于各地的镜像同步时间的延迟，可能在用 `composer` 安装的时候，会暂时找不到你的项目，这个时候可能需要等待同步完成

发布完成之后就可以通过 `composer` 安装你的扩展了

