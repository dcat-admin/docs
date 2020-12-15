# Grid relationships

### One-on-one
The `users` and `profiles` tables are related one-to-one via the `profiles.user_id` field.

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

The corresponding data model and repository are:

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
Repository
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
#### Three ways to link data tables

The `profile` table data can be associated with the code in the following three ways:

Way 1: Directly using repository relationships
```php
use App\Admin\Repositories\User;
use Dcat\Admin\Grid;

// Associated profile table data
$grid = Grid::make(User::with(['profile']), function (Grid $grid) {        
    $grid->id('ID')->sortable();
    
    $grid->name();
    $grid->email();
    
    // Display one-to-one data
    $grid->column('profile.age');
    $grid->column('profile.gender');
    
    $grid->created_at();
    $grid->updated_at();
});
```

Way 2: Use the `Model::with` method 
```php
use App\Models\User;
use Dcat\Admin\Grid;

// Associated profile table data
$grid = Grid::make(User::with(['profile']), function (Grid $grid) {    
    $grid->id('ID')->sortable();
    
    ...
});
```

Method 3: Use the `Grid\Model` method to relate.
```php
use App\Admin\Repositories\User;
use Dcat\Admin\Grid;


$grid = Grid::make(new User(), function (Grid $grid) {
	// Associated profile table data
	$grid->model()->with(['profile']);
    
    $grid->id('ID')->sortable();
    
    ...
});
```


### One-to-many

`posts` and `comments` tables generate one-to-many associations via the `comments.post_id` field.

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

The corresponding data model and repository:

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

The same [three ways to relate data] (#relation) as above is supported here, but we won't repeat all the Usages here.

Post form

```php
use App\Admin\Repositories\Post;

// Related comment table data
$grid = Grid::make(Post::with(['comments']), function (Grid $grid) {
    $grid->id('id')->sortable();
    $grid->title();
    $grid->content();
    
    $grid->comments('Number of comments')->display(function ($comments) {
        $count = count($comments);
        
        return "<span class='label label-warning'>{$count}</span>";
    });
    
    $grid->created_at();
    $grid->updated_at();
});
```

Comment Form

```php
use App\Admin\Repositories\Comment;

// Related post table data
$grid = new Grid(Comment::with(['post']));

$grid->column('id');
$grid->column('post.title');
$grid->column('content');

$grid->created_at()->sortable();
$grid->updated_at();
```

### Many-to-many

The `users` and `roles` tables generate many-to-many relationships through the middle table `role_users`.

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

The corresponding data model and data warehouse are:


User model
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
Role model
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

Repository
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

The same [three ways to relate data] (#relation) as above is supported here, but we won't repeat all the Usages here.

```php
use App\Admin\Repositories\User;

// Associated role table data
$grid = Grid::make(User::with('roles'), function (Grid $grid) {
    $grid->id('ID')->sortable();
    $grid->username();
    $grid->name();
    
    $grid->roles()->pluck('name')->label();
    
    $grid->created_at();
    $grid->updated_at();
});
```


