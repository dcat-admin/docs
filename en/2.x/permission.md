# Permission control

`Dcat Admin` has a built-in `RBAC` permission control module, expand `Auth` on the left sidebar, and below there are three management panels for users, roles, and permissions.

### Routing control

In `Dcat Admin`, permissions and routes are bound together, set the routes that the current permissions can access in the edit permissions page, select the method to access the routes in the `HTTP method` select box, and fill in the path that can be accessed in the `HTTP path`.

For example, if you want to add a permission, which can access the path `/admin/users` by `GET`, then `HTTP method` select `GET`, and `HTTP path` fill in `/users`.


If you want to access all the paths prefixed with `/admin/users`, then `HTTP path` fill in `/users*`; if you want to access the edit page, then `HTTP path` fill in `/users/*/edit`; if the method of each path in multiple paths is different, then `HTTP path` fill in `GET:users/*'. `.


If the above method is not sufficient, `HTTP path` also supports **routing aliases**, such as `admin.users.show`.


### Disable permissions function

Setting the value of the `admin.permission.enable` configuration parameter to `false` completely disables the built-in permission system.

### Super Administrator

The default role `administrator` in `Dcat Admin` is the super admin role, do not change the identity or it will become a normal role.

### Skip privilege verification

Interfaces that require skipping permission validation can be added to the configuration file `admin.permission.except` parameter.

```php
	'permission' => [
		// Whether enable permission.
		'enable' => true,

		// All method to path like: auth/users/*/edit
		// or specific method to path like: get:auth/users.
		'except' => [
			'/',
			'auth/login',
			'auth/logout',
			'auth/setting',
		],

	],
```

### Page control

If you want to control a user's permissions on a page, consider the following example

#### Scenario 1

For example, now there is a scenario to do permission management for the article publishing module to create articles

First of all, create a permission, enter `http://localhost/admin/auth/permissions`, permission identification (slug) fill `create-post`, permission name fill `create article`, so that the permission is created.

The second step can be this permission directly attached to the individual or role, in the user edit page can be created directly above the permission attached to the current editor of the user, or in the edit role page attached to a role.

The third step is to add the control code inside the create article controller:
```php
use Dcat\Admin\Auth\Permission;

class PostController extends Controller
{
    public function create()
    {
        // 检查权限，有create-post权限的用户或者角色可以访问创建文章页面
        Permission::check('create-post');
    }
}
```
This completes the permissions control of a page.

#### Scenario 2

If you want to control the user's display of an element in a table, you need to define two permissions, such as the permissions `delete-image` and `view-title-column`, which are used to control the permission to delete an image and the permission to display a column, respectively.：
```php
$grid->actions(function ($actions) {

    // Roles without `delete-image` permission do not show the delete button
    if (!Admin::user()->can('delete-image')) {
        $actions->disableDelete();
    }
});

// Only users with the `view-title-column` permission can display the `title` column
if (Admin::user()->can('view-title-column')) {
    $grid->column('title');
}
```

### Related methods

Get the current user object
```php
Admin::user();
```

Get current user id
```php
Admin::user()->id;
```

Get user roles
```php
Admin::user()->roles;
```

Get user permissions
```php
Admin::user()->permissions;
```

Whether the user has a role
```php
Admin::user()->isRole('developer');
```

Is there a certain permission
```php
Admin::user()->can('create-post');
```

Is there no certain permission
```php
Admin::user()->cannot('delete-post');
```

Are you a super administrator?
```php
Admin::user()->isAdministrator();
```

Is it one of the roles?
```php
Admin::user()->inRoles(['editor', 'developer']);
```

### Privilege Middleware

Route configuration can be combined with permission middleware to control permissions on routes

```php

// Allow the administrator and editor roles to access routes in the group.
Route::group([
    'middleware' => 'admin.permission:allow,administrator,editor',
], function ($router) {

    $router->resource('users', UserController::class);
    ...
    
});

// Deny the roles of developer and operator access to the routes in the group.
Route::group([
    'middleware' => 'admin.permission:deny,developer,operator',
], function ($router) {

    $router->resource('users', UserController::class);
    ...
    
});

// Users with permission to edit-post, create-post, and delete-post can access the routes in the group
Route::group([
    'middleware' => 'admin.permission:check,edit-post,create-post,delete-post',
], function ($router) {

    $router->resource('posts', PostController::class);
    ...
    
});
```

Permissions middleware is used in the same way as other middleware.

### Why does it still say no access even though I have configured roles and permissions?

The reason for this may be due to a misconfiguration of the `URL` path of the privilege, the correct `URL` configuration that contains the add/drop check function should be something like `auth/users*`, if you configure it as `auth/users/*`, then it will indicate that there is no right to access.

> {tip} In addition, there are two ways to fill in the custom URL of the tag form: one is to select it and press ``delete key`` to change it; the other is to fill in the form and press ``space bar`` + ``Enter key``.

