# 快速开始

## 数据表结构
用`Laravel`自带的`users`表举例,表结构为：
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
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci
```
对应的数据模型为文件 `App\User.php`

`dcat-admin`可以通过使用以下几步来快速生成`users`表的`CURD`操作页面：


## 创建数据仓库
创建文件`app/Admin/Repositories/User.php`并添加如下内容

> {tip} 如果你的数据来自`MySQL`，则`数据仓库`不是必须的，你也可以直接使用`Model`。

```php
<?php

namespace App\Admin\Repositories;

use Dcat\Admin\Repositories\EloquentRepository;
use App\User as UserModel;

class User extends EloquentRepository
{
    /**
     * @var string
     */
    protected $eloquentClass = UserModel::class;
    
    /**
     * 设置表格查询的字段，默认查询所有字段
     * 
     * @return array
     */
    public function getGridColumns(){
        return ['id', 'name', 'email', 'created_at', 'updated_at'];
    }
}
```

## 创建控制器

创建控制器文件`app/Admin/Controllers/UserController.php`，并写入如下内容：
```php
<?php

namespace App\Admin\Controllers;

use Dcat\Admin\Form;
use Dcat\Admin\Grid;
use Dcat\Admin\Show;
use App\Admin\Repositories\User;
use Dcat\Admin\Controllers\AdminController;

class UserController extends AdminController
{
    /**
	 * Title for current resource.
	 *
	 * @var string
	 */
	protected $title = '用户';
    
    /**
     * Make a grid builder.
     *
     * @return Grid
     */
    protected function grid()
    {
        return Grid::make(new User(), function (Grid $grid) {
            $grid->id->bold()->sortable();
            $grid->email;
            $grid->created_at;
            $grid->updated_at->sortable();
            
            $grid->filter(function (Grid\Filter $filter) {
                $filter->equal('id');
            });
        });
    }

    /**
     * Make a show builder.
     *
     * @param mixed $id
     * @return Show
     */
    protected function detail($id)
    {
        return Show::make($id, new User(), function (Show $show) {
            $show->id;
            $show->email;
            $show->created_at;
            $show->updated_at;
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
            $form->display('id');
            $form->text('email');
            
            $form->display('created_at');
            $form->display('updated_at');
        });
    }
}
```


## 添加路由配置

在`dcat-admin`的路由配置文件`app/Admin/routes.php`里添加一行：
```
$router->resource('users', 'UserController');
```

## 添加左侧菜单

打开`http://localhost:8000/admin/auth/menu`，添加对应的menu, 然后就能在后台管理页面的左侧边栏看到用户管理页面的链接入口了。

> 其中`uri`填写不包含路由前缀的的路径部分，比如完整路径是`http://localhost:8000/admin/demo/users`, 那么就填`demo/users`，如果要添加外部链接，只要填写完整的url即可，比如`http://dcat-admin.org/`.

### 菜单翻译

在您的语言文件的`menu_titles`索引中追加菜单标题。
例如“工作单位”标题：

在`resources/lang/{当前语言}/admin.php`中
```php
...
// 用_小写并用_替换空格
'menu_titles' => [
    'work_units' => 'Unidades de trabajo'
],
```

## 完成
一个简单的`CURD`功能页面就构建完成了，打开浏览器访问`http://localhost:8000/admin/users`即可。

剩下的工作就是构建数据表格和表单了，打开 `app/Admin/Contollers/UserController.php`,找到`form()`和`grid()`方法，然添加构建代码。
更多详细使用请查看[数据表格](model-grid.md)和[数据表单](model-form.md)。

