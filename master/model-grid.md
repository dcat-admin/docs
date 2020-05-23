# 表格基本使用


## 简单示例
`Dcat\Admin\Grid`类用于生成基于数据模型的表格，先来个例子，数据库中有`movies`表

```sql
CREATE TABLE `movies` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `title` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
  `director` int(10) unsigned NOT NULL,
  `describe` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
  `rate` tinyint unsigned NOT NULL,
  `released` enum(0, 1),
  `release_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  `created_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  `updated_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;

```

对应的数据模型为`App\Models\Movie`，对应的数据仓库为`App\Admin\Repositories\Movie`，数据仓库代码如下：

> {tip} 如果你的数据来自`MySQL`，则`数据仓库`不是必须的，你也可以直接使用`Model`。

```php
<?php

namespace App\Admin\Repositories;

use Dcat\Admin\Repositories\EloquentRepository;
use App\Models\Movie as MovieModel;

class Movie extends EloquentRepository
{
    protected $eloquentClass = MovieModel::class;
    
    /**
     * 设置表格查询的字段，默认查询所有字段
     * 
     * @return array
     */
    public function getGridColumns(){
        return ['id', 'title', 'director', 'rate', ...];
    }
}
```

下面的代码可以生成表`movies`的数据表格：

```php
<?php

namespace App\Admin\Controllers;

use App\Admin\Repositories\Movie;
use Dcat\Admin\Grid;
use Dcat\Admin\Controllers\AdminController;

class MovieController extends AdminController
{
    protected function grid()
    {
        return Grid::make(new Movie(), function (Grid $grid) {
            // 第一列显示id字段，并将这一列设置为可排序列
            $grid->id('ID')->sortable();
            
            // 第二列显示title字段，由于title字段名和Grid对象的title方法冲突，所以用Grid的column()方法代替
            $grid->column('title');
            
            // 第三列显示director字段，通过display($callback)方法设置这一列的显示内容为users表中对应的用户名
            $grid->director->display(function($userId) {
                return User::find($userId)->name;
            });
            
            // 第四列显示为describe字段
            $grid->describe;
            
            // 第五列显示为rate字段
            $grid->rate;
            
            // 第六列显示released字段，通过display($callback)方法来格式化显示输出
            $grid->released('上映?')->display(function ($released) {
                return $released ? '是' : '否';
            });
            
            // 下面为三个时间字段的列显示
            $grid->release_at;
            $grid->created_at;
            $grid->updated_at;
            
            // filter($callback)方法用来设置表格的简单搜索框
            $grid->filter(function ($filter) {
                // 设置created_at字段的范围查询
                $filter->between('created_at', 'Created Time')->datetime();
            });
        });
    }
}
```

## 基本使用方法

### 添加列
> {tip} 通过`$grid->username('用户名');`的方式添加列时IDE默认是没有自动补全的，这时候可以通过`php artisan admin:ide-helper`生成IDE提示文件。

```php
// 直接通过字段名`username`添加列
$grid->username('用户名');

// 效果和上面一样
$grid->column('username', '用户名');

// 或者用属性的方式
// 没有设置label，会自动用翻译文件的翻译，具体请参照“字段翻译”章节
$grid->username;

// 添加多列
$grid->columns('email', 'username' ...);
```

### 修改来源数据
```php
$grid->model()->where('id', '>', 100);

$grid->model()->orderBy('id', 'desc');

// 回收站数据
$grid->model()->onlyTrashed();

...
```

同时也可以使用以下方式

```php
protected function grid()
{
    return Grid::make(Model::with('...')->where(...), function (Grid $grid) {
        ...
    });
}
```

其它查询方法可以参考`eloquent`的查询方法.

### 修改显示输出


```php
$grid->text()->display(function($text) {
    return str_limit($text, 30, '...');
});

// 允许混合使用多个“display”方法
$grid->name()->display(function ($name) {
     return "<b>$name</b>";
 })->display(function ($name) {
    return "<span class='label'>$name</span>";
});

$grid->email->display(function ($email) {
    return "mailto:$email";
});

// 可以直接写字符串
$grid->username->display('...');

// 添加不存在的字段
$grid->column_not_in_table->display(function () {
    return 'blablabla....';
});
```

### 获取当前行数据

`display()`方法接收的匿名函数绑定了当前行的数据对象，可以在里面调用当前行的其它字段数据

```php
$grid->first_name();
$grid->last_name();

// 不存的字段列
$grid->column('full_name')->display(function () {
    return $this->first_name.' '.$this->last_name;
});
```

### 显示表格边框

通过`withBorder`方式可以让表格显示边框

> {tip} Since `v1.2.0`

```php
$grid->withBorder();
```

效果
<a href="http://103.39.211.179:8080/admin/reports" target="_blank">
    <img style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" src="{{public}}/assets/img/screenshots/grid-combination.png">
</a>


> {tip} Since `v1.4.5`
 
禁用边框模式
```php
$grid->withBorder(false);
```

<a name="outline"></a>
### 设置工具栏按钮样式

> {tip} Since `v1.4.5`

从这个`v1.4.5`工具栏按钮默认显示`outline`模式，效果如下


用法
```php
$grid->toolsWithOutline();

// 禁止
$grid->toolsWithOutline(false);
```

效果
<a href="{{public}}/assets/img/screenshots/outline.png" target="_blank">
    <img style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" src="{{public}}/assets/img/screenshots/outline.png">
</a>

禁用`outline`后的效果

<a href="{{public}}/assets/img/screenshots/n-outline.png" target="_blank">
    <img style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" src="{{public}}/assets/img/screenshots/n-outline.png">
</a>

如果你希望某个按钮不使用`outline`模式，可以在按钮的`class`属性中加上`disable-outline`
```php
$grid->tools('<a class="btn btn-primary disable-outline">测试按钮</a>');
```


### 设置表格外层容器
```php
 // 更改表格外层容器
$grid->wrap(function (Renderable $view) {
    $tab = Tab::make();
    
    $tab->add('示例', $view);
    $tab->add('代码', $this->code(), true);

    return $tab;
});
```

### 设置创建按钮

此功能默认开启
```php
// 禁用
$grid->disableCreateButton();
// 显示
$grid->showCreateButton();
```

#### 开启弹窗创建表单

此功能默认不开启

```php
$grid->enableDialogCreate();

// 设置弹窗宽高，默认值为 '700px', '670px'
$grid->setDialogFormDimensions('50%', '50%');
```


### 设置查询过滤器

此功能默认开启

```php
// 禁用
$grid->disableFilter();
// 显示
$grid->showFilter();

// 禁用过滤器按钮
$grid->disableFilterButton();
// 显示过滤器按钮
$grid->showFilterButton();
```


### 设置行选择器
```php
// 禁用
$grid->disableRowSelector();
// 显示
$grid->showRowSelector();
```

#### 设置选择中行的标题字段key
如不设置，默认取 `name`、 `title`、 `username`中的一个。
```php
$grid->full_name;
$grid->age;

...

$grid->rowSelector()->titleColumn('full_name');
```

#### 设置checkbox选择框颜色
默认 `primary`，支持：`default`、 `primary`、 `success`、 `info`、 `danger`、 `purple`、 `inverse`。
```php
$grid->rowSelector()->style('success');
```

#### 点击当前行任意位置选中
此功能默认不开启。
```php
$grid->rowSelector()->click();
```

#### 设置选中行的背景颜色
```php
use Dcat\Admin\Admin;

$grid->rowSelector()->background(Admin::color()->dark20());
```

#### 设置checkbox选择框形状
默认圆形。
```php
// 设置为正方形
$grid->rowSelector()->circle(false);
```

### 设置行操作按钮
```php
// 禁用
$grid->disableActions();
// 显示
$grid->showActions();

// 禁用详情按钮
$grid->disableViewButton();
// 显示详情按钮
$grid->showViewButton();

// 禁用编辑按钮
$grid->disableEditButton();
// 显示编辑按钮
$grid->showEditButton();

// 禁用快捷编辑按钮
$grid->disableQuickEditButton();
// 显示快捷编辑按钮
$grid->showQuickEditButton();

// 设置弹窗宽高，默认值为 '700px', '670px'
$grid->setDialogFormDimensions('50%', '50%);


// 禁用删除按钮
$grid->disableDeleteButton();
// 显示删除按钮
$grid->showDeleteButton();

```

### 设置批量操作按钮
```php
// 禁用
$grid->disableBatchActions();
// 显示
$grid->showBatchActions();

// 禁用批量删除按钮
$grid->disableBatchDelete();
// 显示批量删除按钮
$grid->showBatchDelete();
```

### 设置工具栏
```php
// 禁用
$grid->disableToolbar();
// 显示
$grid->showToolbar();
```

### 设置刷新按钮
```php
// 禁用
$grid->disableRefreshButton();
// 显示
$grid->showRefreshButton();
```

### 设置分页功能
```php
// 禁用
$grid->disablePagination();
// 显示
$grid->showPagination();
```

#### 设置每页显示行数

```php
// 默认为每页20条
$grid->paginate(15);
```

#### 设置分页选择器选项
```php
$grid->perPages([10, 20, 30, 40, 50]);

// 禁用分页选择器
$grid->disablePerPages();
```

## 关联模型

### 一对一
`users`表和`profiles`表通过`profiles.user_id`字段生成一对一关联

```sql
CREATE TABLE `users` (
`id` int(10) unsigned NOT NULL AUTO_INCREMENT,
`name` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
`email` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
`created_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
`updated_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;

CREATE TABLE `profiles` (
`id` int(10) unsigned NOT NULL AUTO_INCREMENT,
`user_id` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
`age` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
`gender` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
`created_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
`updated_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
```

对应的数据模以及数据仓库分别为:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    public function profile()
    {
        return $this->hasOne(Profile::class);
    }
}
```

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Profile extends Model
{
    public function user()
    {
        return $this->belongsTo(User::class);
    }
}
```
数据仓库
```php
<?php

namespace App\Admin\Repositories;

use Dcat\Admin\Repositories\EloquentRepository;
use App\Models\User as UserModel;

class User extends EloquentRepository
{
    protected $eloquentClass = UserModel::class;
}
```

<a name="relation"></a>
#### 三种关联数据表的方法

通过以下三种方式的代码可以关联`profile`表数据:

方式一：直接使用数据仓库关联
```php
use App\Admin\Repositories\User;
use Dcat\Admin\Grid;

// 关联 profile 表数据
$grid = Grid::make(new User(['profile']), function (Grid $grid) {    
    $grid->id('ID')->sortable();
    
    $grid->name();
    $grid->email();
    
    $grid->column('profile.age');
    $grid->column('profile.gender');
    // 或者
    $grid->profile('年龄')->get('age');
    $grid->profile('性别')->get('gender');
    
    $grid->created_at();
    $grid->updated_at();
});
```

方式二：使用`Model::with`方法关联
```php
use App\Models\User;
use Dcat\Admin\Grid;

// 关联 profile 表数据
$grid = Grid::make(User::with(['profile']), function (Grid $grid) {    
    $grid->id('ID')->sortable();
    
    ...
});
```

方式三：使用`Grid\Model`方法关联
```php
use App\Admin\Repositories\User;
use Dcat\Admin\Grid;


$grid = Grid::make(new User(), function (Grid $grid) {
	// 关联 profile 表数据
	$grid->model()->with(['profile']);
    
    $grid->id('ID')->sortable();
    
    ...
});
```


### 一对多

`posts`表和`comments`表通过`comments.post_id`字段生成一对多关联

```sql
CREATE TABLE `posts` (
`id` int(10) unsigned NOT NULL AUTO_INCREMENT,
`title` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
`content` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
`created_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
`updated_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;

CREATE TABLE `comments` (
`id` int(10) unsigned NOT NULL AUTO_INCREMENT,
`post_id` int(10) unsigned NOT NULL,
`content` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
`created_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
`updated_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
```

对应的数据模和数据仓库分别为:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    public function comments()
    {
        return $this->hasMany(Comment::class);
    }
}
```

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Comment extends Model
{
    public function post()
    {
        return $this->belongsTo(Post::class);
    }
}
```

```php
<?php

namespace App\Admin\Repositories;

use App\Models\Post as PostModel;
use Dcat\Admin\Repositories\EloquentRepository;

class Post extends EloquentRepository
{
    protected $eloquentClass = PostModel::class;
}
```

```php
<?php

namespace App\Admin\Repositories;

use App\Models\Comment as CommentModel;
use Dcat\Admin\Repositories\EloquentRepository;

class Comment extends EloquentRepository
{
    protected $eloquentClass = CommentModel::class;
}
```

同样这里支持上述的[三种方式关联数据](#relation)，限于篇幅这里不再重复写所有用法

Post表格

```php
use App\Admin\Repositories\Post;

// 关联 comment 表数据
$grid = Grid::make(new Post(['comments']), function (Grid $grid) {
    $grid->id('id')->sortable();
    $grid->title();
    $grid->content();
    
    $grid->comments('评论数')->display(function ($comments) {
        $count = count($comments);
        
        return "<span class='label label-warning'>{$count}</span>";
    });
    
    $grid->created_at();
    $grid->updated_at();
});
```

Comment表格

```php
use App\Admin\Repositories\Comment;

// 关联 post 表数据
$grid = new Grid(new Comment(['post']));

$grid->id('id');
$grid->post()->get('title');
$grid->content();

$grid->created_at()->sortable();
$grid->updated_at();
```

### 多对多

`users`和`roles`表通过中间表`role_users`产生多对多关系

```sql
CREATE TABLE `users` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `username` varchar(190) COLLATE utf8_unicode_ci NOT NULL,
  `password` varchar(60) COLLATE utf8_unicode_ci NOT NULL,
  `name` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
  `created_at` timestamp NULL DEFAULT NULL,
  `updated_at` timestamp NULL DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `users_username_unique` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci

CREATE TABLE `roles` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(50) COLLATE utf8_unicode_ci NOT NULL,
  `slug` varchar(50) COLLATE utf8_unicode_ci NOT NULL,
  `created_at` timestamp NULL DEFAULT NULL,
  `updated_at` timestamp NULL DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `roles_name_unique` (`name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci

CREATE TABLE `role_users` (
  `role_id` int(11) NOT NULL,
  `user_id` int(11) NOT NULL,
  `created_at` timestamp NULL DEFAULT NULL,
  `updated_at` timestamp NULL DEFAULT NULL,
  KEY `role_users_role_id_user_id_index` (`role_id`,`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci
```

对应的数据模和数据仓库分别为:


User 模型
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    public function roles()
    {
        return $this->belongsToMany(Role::class);
    }
}
```
Role 模型
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Role extends Model
{
    public function users()
    {
        return $this->belongsToMany(User::class);
    }
}
```

数据仓库
```php
<?php

namespace App\Admin\Repositories;

use App\Models\User as UserModel;
use Dcat\Admin\Repositories\EloquentRepository;

class User extends EloquentRepository
{
    protected $eloquentClass = UserModel::class;
}
```

同样这里支持上述的[三种方式关联数据](#relation)，限于篇幅这里不再重复写所有用法

```php
use App\Admin\Repositories\User;

// 关联 role 表数据
$grid = Grid::make(new User('roles'), function (Grid $grid) {
    $grid->id('ID')->sortable();
    $grid->username();
    $grid->name();
    
    $grid->roles()->pluck('name')->label();
    
    $grid->created_at();
    $grid->updated_at();
});
```





