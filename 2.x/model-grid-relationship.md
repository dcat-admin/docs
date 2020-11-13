# 表格关联关系

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
$grid = Grid::make(User::with(['profile']), function (Grid $grid) {    
    $grid->id('ID')->sortable();
    
    $grid->name();
    $grid->email();
    
    // 显示一对一数据
    $grid->column('profile.age');
    $grid->column('profile.gender');
    
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
$grid = Grid::make(Post::with(['comments']), function (Grid $grid) {
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
$grid = new Grid(Comment::with(['post']));

$grid->column('id');
$grid->column('post.title');
$grid->column('content');

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
$grid = Grid::make(User::with('roles'), function (Grid $grid) {
    $grid->id('ID')->sortable();
    $grid->username();
    $grid->name();
    
    $grid->roles()->pluck('name')->label();
    
    $grid->created_at();
    $grid->updated_at();
});
```


