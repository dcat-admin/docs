# 快速开始


在日常开发中，我们可以用代码生成器一键生成增删改查页面代码，非常的方便快捷。


下面将会给大家介绍代码生成器的使用方法，以及一个增删改查页面的基本构成。通过学习下面的内容将可以帮助大家快速理解这个系统的基本使用方法。

## 代码生成器

### 创建数据表

安装完`Laravel`之后会内置一个`users`表的`migration`文件(如果不了解`migration`文件作用，请参考文档[数据库迁移](https://learnku.com/docs/laravel/7.x/migrations/7496))，文件路径为`database/migrations/2014_10_12_000000_create_users_table.php`。

然后我们运行以下命令，在`MySQL`中创建这个数据表

```php
php artisan migrate
```

运行完之后可以看到数据库中已经多了一个`users`表，结构如下

```sql
CREATE TABLE `users` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
  `email` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
  `password` varchar(60) COLLATE utf8_unicode_ci NOT NULL,
  `remember_token` varchar(100) COLLATE utf8_unicode_ci DEFAULT NULL,
  `created_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  `updated_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  PRIMARY KEY (`id`),
  UNIQUE KEY `users_email_unique` (`email`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
```

### 一键生成增删改查页面

> {tip} 如果你的开发环境不是`windows`，请注意要给项目目录设置读写权限，否则可能出现无法生成代码的情况。

**1.**首先打开地址`http://你的域名/admin/helpers/scaffold`，进入代码生成器页面；

**2.**由于前面已经创建好了数据表，所以这里我们可以直接通过页面左上角的第二个下拉选框选择`users`表，选择之后会自动填充字段信息，效果如下

<a href="{{public}}/assets/img/screenshots/quick-start-1.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/quick-start-1.png" style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" >
</a>

**3.**修改模型名称为`App\User`

**4.**由于`migration`文件、数据表、以及模型文件(使用内置的`App\User`即可)都已经有了，所以此处我们可以把这三个选项给去掉

**5.**填写字段翻译

最后呈现效果如下

<a href="{{public}}/assets/img/screenshots/quick-start-2.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/quick-start-2.png" style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" >
</a>

最后点击创建按钮即可，创建的文件如下

```
app/Admin
├── Controllers
│   └── UserController.php  # 控制器
└── Repositories            # 数据仓库
│   └── User.php
resouces/lang/{当前语言}
└── user.php                # 语言包
```

### 添加路由配置

打开路由配置文件`app/Admin/routes.php`，往里面添加一行：
```
$router->resource('users', 'UserController');
```

到此，就可以打开浏览器输入地址`http://你的域名/admin/users`访问刚刚创建完的页面了

### 添加左侧菜单

打开`http://你的域名/admin/auth/menu`，添加对应的menu, 然后就能在后台管理页面的左侧边栏看到用户管理页面的链接入口了。

> 其中`uri`填写不包含路由前缀的的路径部分，比如完整路径是`http://你的域名/admin/demo/users`, 那么就填`demo/users`，如果要添加外部链接，只要填写完整的url即可，比如`http://dcat-admin.org/`.

### 菜单翻译

在您的语言文件的`menu_titles`索引中追加菜单标题。
例如“工作单位”标题：

在`resources/lang/{当前语言}/menu.php`中
```php
...
// 用_小写并用_替换空格
'titles' => [
    'work_units' => 'Unidades de trabajo'
],
```

### 完成
这样一个简单的`CURD`功能就构建完成了，剩下的工作就是深度构建数据表格和表单了，打开 `app/Admin/Contollers/UserController.php`,找到`form()`和`grid()`方法，然添加构建代码。
更多详细使用请查看[数据表格](model-grid.md)和[数据表单](model-form.md)。


## 增删改查功能简易说明

为了便于大家理解增删改查功能的基本用法，下面将给大家简单介绍前面使用生成器生成的代码。

### 控制器

`Dcat Admin`的增删改查页面代码是非常简洁和易懂的，对开发者非常的友好，只需极少的代码即可构建出一个功能完善的后台系统，并且非常简单灵活和易于扩展。

打开`app/Admin/Controllers/UserController.php`可以看到如下代码

```php
<?php

namespace App\Admin\Controllers;

use App\Admin\Repositories\User;
use Dcat\Admin\Form;
use Dcat\Admin\Grid;
use Dcat\Admin\Show;
use Dcat\Admin\Http\Controllers\AdminController;

class UserController extends AdminController
{
    /**
     * Make a grid builder.
     *
     * @return Grid
     */
    protected function grid()
    {
        return Grid::make(new User(), function (Grid $grid) {
            // 这里的字段会自动使用翻译文件
            $grid->column('id')->sortable();
            $grid->column('name');
            $grid->column('email');
            $grid->column('email_verified_at');
            $grid->column('password');
            $grid->column('remember_token');
            $grid->column('created_at');
            $grid->column('updated_at')->sortable();
        
            $grid->filter(function (Grid\Filter $filter) {
                $filter->equal('id');
        
            });
        });
    }

    /**
     * Make a show builder.
     *
     * @param mixed $id
     *
     * @return Show
     */
    protected function detail($id)
    {
        return Show::make($id, new User(), function (Show $show) {
            // 这里的字段会自动使用翻译文件
            $show->field('id');
            $show->field('name');
            $show->field('email');
            $show->field('email_verified_at');
            $show->field('password');
            $show->field('remember_token');
            $show->field('created_at');
            $show->field('updated_at');
        });
    }

    /**
     * Make a form builder.
     *
     * @return Form
     */
    protected function form()
    {
        return Form::make(new User(), function (Form $form) {
            // 这里的字段会自动使用翻译文件
            $form->display('id');
            $form->text('name');
            $form->text('email');
            $form->text('email_verified_at');
            $form->text('password');
            $form->text('remember_token');
        
            $form->display('created_at');
            $form->display('updated_at');
        });
    }
}
```


### 数据仓库

`Dcat Admin` 构建页面并不直接依赖于`Model`，而是引入了数据仓库作为中间层，让页面的构建不再与数据的读写产生强耦合关系。

数据仓库是`Dcat Admin`中对数据增删改查操作接口的具体实现，更详细用法请参考[数据仓库](model-repository.md)。

> {tip} 如果你的数据来自`MySQL`，那么你也可以直接使用`Model`实例，底层会自动把`Model`转化为数据仓库实例。这里为了便于大家理解其中的概念，所以创建了数据仓库文件。

我们打开刚刚生成的文件`app/Admin/Repositories/User.php`，可以看到只有如下内容，非常简单

```php
<?php

namespace App\Admin\Repositories;

use Dcat\Admin\Repositories\EloquentRepository;
use App\User as UserModel;

class User extends EloquentRepository
{
    protected $eloquentClass = UserModel::class;
}
```

### 语言包

每个控制器都可以生成自己对应的语言包，并且[数据表格](model-grid.md)、[数据表单](model-form.md)和[数据详情](model-show.md)功能都会自动读取里面的翻译。

下面我们打开`UserController`对应的语言包文件`resouces/lang/{当前语言}/user.php`

```php
<?php 
return [
    // labels是自定义标签翻译
    'labels' => [
        // 这个是页面 title 翻译
        'User' => '用户',
    ],
    // 表字段翻译
    'fields' => [
        'name' => '名称',
        'email' => '邮箱',
        'email_verified_at' => '验证时间',
        'password' => '密码',
        'remember_token' => 'remember_token',
    ],
    'options' => [
    ],
];

```

