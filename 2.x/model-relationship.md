# 表单关联关系


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

对应的数据模分别为:

```php
<?php

namespace App\Admin\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    public function profile()
    {
        return $this->hasOne(Profile::class);
    }
}

class Profile extends Model
{
    public function user()
    {
        return $this->belongsTo(User::class);
    }
}
```
对应的数据仓库为：
```php
<?php

namespace App\Admin\Repositories;

use Dcat\Admin\Repositories\EloquentRepository;
use User as UserModel;

class User extends \Dcat\Admin\Repositories\EloquentRepository
{
    protected $eloquentClass = UserModel::class;
}
```


通过下面的代码可以关联在一个form里面:
> {tip} 实例化数据仓库时需要传入关联模型定义的关联名称，相当于主动使用`Eloquent\Model::with`方法。

```php
use App\Admin\Repositories\User;

// 注意这里实例化数据仓库`User`时必须传入"profile"，否则将无法关联"profiles"表数据
$form = Form::make(User::with('profile'), function (Form $form) {
    $form->display('id');
    
    $form->text('name');
    $form->text('email');
    
    $form->text('profile.age');
    $form->text('profile.gender');
    
    $form->datetime('created_at');
    $form->datetime('updated_at');
});
```

如果你不想使用数据仓库，也可以直接使用模型
```php
use App\Admin\Models\User;

// 注意这里是直接使用模型，没有使用数据仓库
$form = Form::make(User::with('profile'), function (Form $form) {
    $form->display('id');
    
    ...
});
```


### 一对多

一对多的使用请参考文档[表单字段的使用-一对多](model-form-fields.md#onemany)

### 多对多


下面以项目内置的`角色管理`模块的**角色绑定权限**功能为例来演示多对多关联模型的用法

模型`Role`
```php
<?php

namespace Dcat\Admin\Models;

use Dcat\Admin\Traits\HasDateTimeFormatter;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class Role extends Model
{
    use HasDateTimeFormatter;

    /**
     * 定义你的关联模型.
     *
     * @return BelongsToMany
     */
    public function permissions(): BelongsToMany
    {
        $pivotTable = 'admin_role_permissions'; // 中间表

        $relatedModel = Permission::class; // 关联模型类名

        return $this->belongsToMany($relatedModel, $pivotTable, 'role_id', 'permission_id');
    }
}
```

```php
use Dcat\Admin\Models\Permission;

// 实例化数据仓库时传入 permissions，则会自动关联关联模型的数据
// 这里传入 permissions 关联权限模型的数据
$repository = Role::with(['permissions']);

return Form::make($repository, function (Form $form) {
    $form->display('id', 'ID');

    $form->text('slug', trans('admin.slug'))->required();
    $form->text('name', trans('admin.name'))->required();
    
    // 这里的数据会自动保存到关联模型中
    $form->tree('permissions')
        ->nodes(function () {
            return (new Permission())->allNodes();
        })
        ->customFormat(function ($v) {
            if (!$v) return [];

            // 这一步非常重要，需要把数据库中查出来的二维数组转化成一维数组
            return array_column($v, 'id');
        });

    ...
});
```

如果你不想使用数据仓库，也可以直接使用模型
```php
use Dcat\Admin\Models\Role;

// 注意这里是直接使用模型，没有使用数据仓库
$form = Form::make(Role::with('permissions'), function (Form $form) {
    $form->display('id');
    
    ...
});
```

最终效果如下

![](https://cdn.learnku.com/uploads/images/202004/26/38389/aeYpYDrUQP.png!large)


### 关联模型名称为驼峰风格

如果你的关联模型名称的命名是**驼峰**风格，那么使用的时候需要转化为**下划线**风格命名


例如
```php
class User extend Model
{
    public function userProfile()
    {
        return ...;
    }
}
```

使用
```php
return Form::make(User::with(['userProfile']), function (Form $form) {

    ...
    
    // 注意这里必须使用下划线风格命名，否则将无法显示编辑数据
    $form->text('user_profile.postcode');
    $form->text('user_profile.address');
    
});
```