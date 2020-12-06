# Relationships


## One on one
The `users` table and the `posts` table above are related on a one-to-one basis through the `posts.author_id` field, and the `users` and `post` tables are structured as follows:

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
The model is defined as：

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
A repository is defined as：
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

Then you can display the details of the user to which `post` belongs in the following way:

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

> {tip} In order to be able to use the tools in the upper right corner of this panel properly, the `resource` method must be used to set the url access path to the user resource.

If there are other conditional queries required for your associative model, you can refer to the following
```php
use App\Models\User;

$show->author(function ($model) {
    // Model setting query conditions
    $userModel = User::where('state', $model->state);

    return Show::make($model->author_id, $userModel, function (Show $show) {
        // Setting Up Routes
        $show->resource('/users');
    
        $show->id();
        $show->name();
        $show->email();
    });
});
```

### Easy way
If you're simply displaying information from an associated table, you could also write it like this

```php
// If you're using a model, you can specify an association like this
$model = Post::with('author');

// If you're using a repository, you can specify the relationships like this
// $repository = new Post(['author']);

return Show::make($id, $model, function (Show $show) {
    $show->field('author.id', 'Author ID');
    $show->field('author.name', 'Author name');
    
    ...
});
```


If the name of your associated model is named **camelCase** style, then the use needs to be converted to **snake_case** style naming

```php
// Note that this must be named with an underscore style, otherwise the edit data will not be displayed.
$show->field('user_profile.postcode');
$show->field('user_profile.address');
```


## One-to-many
One-to-many will be presented as a [data table](model-grid.md), here is a simple example

The `posts` table and the comment table `comments` are in a one-to-many relationship (a post has multiple comments), which is associated by the `comments.post_id` field.

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
The model is defined as：

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
A repository is defined as：
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

Then the display of comments is made possible by the following code：

```php
use App\Admin\Repositories\Comment;

$show->comments(function ($model) {
    $grid = new Grid(new Comment);
    
    $grid->model()->where('post_id', $model->id);
    
    // Setting Up Routes
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

Note: The `resource()` method must be used to set the url to the `comments` resource in order to use this datagrid functionality properly!

## Many-to-many

Many-to-many will be presented as a [data table](model-grid.md), here is a simple example

The role table `roles` and the permission table `permissions` are in a many-to-many relationship and are related via the middle table `role_permissions`

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
The model is defined as：

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
A repository is defined as：
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

Then the permissions are displayed using the following code：

```php
use App\Admin\Repositories\Permission;

$show->permissions(function ($model) {
    $grid = new Grid(new Permission);

    $grid->model()->join('role_permissions', function ($join) use ($model) {
        $join->on('role_permissions.permission_id', 'id')
            ->where('role_id', '=', $model->id);
    });

    // Setting Up Routes
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



