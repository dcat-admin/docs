# 关联关系


## 一对一
`users`表和上面的`posts`表为一对一关联关系，通过`posts.author_id`字段关联，`users`和`post`表结构如下：

```sql
CREATE TABLE `posts` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `author_id` int(10) unsigned NOT NULL ,
  `title` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
  `content` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
  `rate` int(255) COLLATE utf8_unicode_ci NOT NULL,
  `release_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  `created_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  `updated_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;

CREATE TABLE `users` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
  `email` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
  `created_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  `updated_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
```
模型定义为：

```php
class User extends Model
{
}

class Post extends Model
{
    public function author()
    {
        return $this->belongsTo(User::class, 'author_id');
    }
}
```
数据仓库定义为：
```php
<?php

namespace App\Admin\Repositories;

use Dcat\Admin\Repositories\EloquentRepository;
use User as UserModel;

class User extends EloquentRepository
{
    protected $eloquentClass = UserModel::class;
}
```

那么可以用下面的方式显示`post`所属的用户的详细：

```php
use App\Models\User;

$show->author(function ($model) {
    return Show::make($model->author_id, new User(), function (Show $show) {
        $show->resource('/users');
    
        $show->id();
        $show->name();
        $show->email();
    });
});
```

> {tip} 为了能够正常使用这个面板右上角的工具，必须用`resource`方法设置用户资源的url访问路径。

如果你的关联模型还需要有其他的条件查询，则可以参考以下方式
```php
use App\Models\User;

$show->author(function ($model) {
    // 模型设置查询条件
    $userModel = User::where('state', $model->state);

    return Show::make($model->author_id, $userModel, function (Show $show) {
        // 设置路由
        $show->resource('/users');
    
        $show->id();
        $show->name();
        $show->email();
    });
});
```

### 简单方式
如果你只是简单的展示关联表信息，也可以这么写

```php
// 如果你用的是模型，可以这样指定关联关系
$model = Post::with('author');

// 如果你用的是数据仓库，可以这样指定关联关系
// $repository = new Post(['author']);

return Show::make($id, $model, function (Show $show) {
    $show->field('author.id', '作者ID');
    $show->field('author.name', '作者名称');
    
    ...
});
```


如果你的关联模型名称的命名是**驼峰**风格，那么使用的时候需要转化为**下划线**风格命名

```php
// 注意这里必须使用下划线风格命名，否则将无法显示编辑数据
$show->field('user_profile.postcode');
$show->field('user_profile.address');
```


## 一对多
一对多会以[数据表格](model-grid.md)的方式呈现，下面是简单的例子

`posts`表和评论表`comments`为一对多关系(一条post有多条comments)，通过`comments.post_id`字段关联

```sql
CREATE TABLE `comments` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `post_id` int(10) unsigned NOT NULL,
  `content` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
  `created_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  `updated_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci
```
模型定义为：

```php
class Post extends Model
{
    public function comments()
    {
        return $this->hasMany(Comment::class);
    }
}

class Comment extends Model
{
}
```
数据仓库定义为：
```php
<?php

namespace App\Admin\Repositories;

use Dcat\Admin\Repositories\EloquentRepository;
use Comment as CommentModel;

class Comment extends EloquentRepository
{
    protected $eloquentClass = CommentModel::class;
}
```

那么评论的显示通过下面的代码实现：

```php
use App\Admin\Repositories\Comment;

$show->comments(function ($model) {
    $grid = new Grid(new Comment);
    
    $grid->model()->where('post_id', $model->id);
    
    // 设置路由
    $grid->resource('comments');

    $grid->id();
    $grid->content()->limit(10);
    $grid->created_at();
    $grid->updated_at();

    $grid->filter(function ($filter) {
        $filter->like('content')->width('300px');
    });
    
    return $grid;
});
```

注意：为了能够正常使用这个数据表格的功能，必须用`resource()`方法设置`comments`资源的url访问路径

## 多对多

多对多会以[数据表格](model-grid.md)的方式呈现，下面是简单的例子

角色表`roles`和权限表`permissions`为多对多关系，通过中间表`role_permissions`关联

```sql
CREATE TABLE `roles` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(50) COLLATE utf8mb4_unicode_ci NOT NULL,
  `slug` varchar(50) COLLATE utf8mb4_unicode_ci NOT NULL,
  `created_at` timestamp NULL DEFAULT NULL,
  `updated_at` timestamp NULL DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `admin_roles_name_unique` (`name`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

CREATE TABLE `permissions` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(50) COLLATE utf8mb4_unicode_ci NOT NULL,
  `slug` varchar(50) COLLATE utf8mb4_unicode_ci NOT NULL,
  `http_method` varchar(255) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `http_path` text COLLATE utf8mb4_unicode_ci,
  `order` int(11) NOT NULL DEFAULT '0',
  `parent_id` int(11) NOT NULL DEFAULT '0',
  `created_at` timestamp NULL DEFAULT NULL,
  `updated_at` timestamp NULL DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `admin_permissions_name_unique` (`name`)
) ENGINE=InnoDB AUTO_INCREMENT=9 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

CREATE TABLE `role_permissions` (
  `role_id` int(11) NOT NULL,
  `permission_id` int(11) NOT NULL,
  `created_at` timestamp NULL DEFAULT NULL,
  `updated_at` timestamp NULL DEFAULT NULL,
  KEY `admin_role_permissions_role_id_permission_id_index` (`role_id`,`permission_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```
模型定义为：

```php
class Role extends Model
{
  public function permissions() : BelongsToMany
  {
      return $this->belongsToMany(Permission::class, 'role_permissions', 'role_id', 'permission_id');
  }

}

class Permission extends Model
{
}
```
数据仓库定义为：
```php
<?php

namespace App\Admin\Repositories;

use Dcat\Admin\Repositories\EloquentRepository;
use Permission as PermissionModel;

class Permission extends EloquentRepository
{
    protected $eloquentClass = PermissionModel::class;
}
```

那么权限的显示通过下面的代码实现：

```php
use App\Admin\Repositories\Permission;

$show->permissions(function ($model) {
    $grid = new Grid(new Permission);

    $grid->model()->join('role_permissions', function ($join) use ($model) {
        $join->on('role_permissions.permission_id', 'id')
            ->where('role_id', '=', $model->id);
    });

    // 设置路由
    $grid->resource('auth/permissions');

    $grid->id;
    $grid->name;
    $grid->slug;
    $grid->http_path;

    $grid->filter(function (Grid\Filter $filter) {
        $filter->equal('id')->width('300px');
    });

    return $grid;
});
```



