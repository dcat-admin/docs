# Help function

### admin_exit

`admin_exit` is used to interrupt program execution and respond to data displayed to the browser, instead of `exit` and `die`, the following is a brief description of usage


Usage 1, return `Content` layout object, this usage can be used to return error messages to the front end
```php
use Dcat\Admin\Widgets\Alert;
use Dcat\Admin\Layout\Content;

// Interrupt the program and display a custom page to the front end
admin_exit(
    Content::make()
        ->title('title')
        ->description('description')
        ->body('Page content 1')
        ->body(Alert::make('Server error~', 'Error')->danger())
);
```

The effect is as follows

! [](https://cdn.learnku.com/uploads/images/202101/11/38389/FLg6C7kwRq.png!large)

Usage 2, returning `json` formatted data, often used for `api` request interception for form submissions, or `api` request interception for `Action`

```php
use Dcat\Admin\Admin;

admin_exit(
    Admin::json()
        ->success('succeeded')
        ->refresh()
        ->data([
            ...
        ])
);

// Of course you can also respond directly to the array
admin_exit([
   ...
]);
```

Usage 3, direct corresponding `Response` object or string

```php
admin_exit('Hello world');

admin_exit(response('Hello world', 500));
```

### admin_color

Get the built-in color. For more information on the use of theme colors, please refer to the [Theme - Colors](theme.md#color) section.

```php
// Three ways to get the theme color
$primary = admin_color('primary');
$primary = admin_color()->get('primary');
$primary = admin_color()->primary();

$color = admin_color();
$color->lighten('primary', 10);
```

### admin_js

You can include `js` files anywhere, see the [static resources](assets.md) chapter for more information.

```php
admin_js(['@admin/xxx.js']);
```

### admin_css

You can include `css` files anywhere, see the [static resources](assets.md) chapter for more information.

```php
admin_css(['@admin/xxx.css']);
```

### admin_require_assets

You can include static resource components anywhere, see the [Static Resources](assets.md) chapter for more information.

```php
admin_require_assets(['@datime']);
```


### admin_path

Get the application path where `Dcat Admin` is installed, the default directory is `app/Admin`:

```php
$bootstrap = admin_path('bootstrap.php');
```

### admin_url

Get the full url of the route for the `Dcat Admin` application:

```php
// returns： http://localhost/admin/auth/users
$url = admin_url('auth/users');
```

### admin_route

Get URL by alias

The `app/Admin/routes.php` route is registered as follows
```php
Route::group([
    'prefix'        => config('admin.route.prefix'),
    'namespace'     => config('admin.route.namespace'),
    'middleware'    => config('admin.route.middleware'),
], function (Router $router) {
	// Set aliases
	$router->resource('users', 'UserController', [
		'names' => ['index' => 'my-users'],
	]);

});
```

Get URL by alias

```php
// Get the url
$url = admin_route('my-users');

// Determine the route
$isUsers = request()->routeIs(admin_route_name('users'));
```

### admin_base_path

Get the routing path for the `Dcat Admin` application.
```php
// returns： /admin/auth/users
$path = admin_base_path('auth/users');
```

### admin_toastr

A `toastr` prompt pops up after the page is refreshed with the following parameters:

- `$message` Tip window content
- `$type` Prompt window type, default `success`, support for `success`, `info`, `warning`, `error`
- `$options` toastr configuration parameters

```php
admin_alert('Updating Success', 'success');
```

### admin_success

Displays a success message at the top of the page after a page refresh:
```php
admin_success('TITLE', 'success了');
```

### admin_error

An error message is displayed at the top of the page after a page refresh:
```php
admin_error('TITLE', 'It failed.');
```

### admin_warning

Displays a warning message at the top of the page after a page refresh:
```php
admin_warning('TITLE', 'warning');
```

### admin_info

After a page refresh a message is displayed at the top of the page stating:
```php
admin_info('TITLE', 'content');
```

### admin_asset

Full link to static resources:

> {tip} This function supports aliases

```html
// Include css
<link rel="stylesheet" href="{{ admin_asset("@admin/dcat-admin/main.min.css") }}">

// Include js
<script src="{{ admin_asset('@admin/dcat-admin/main.min.js')}}"></script>
```

### admin_trans_field

To translate the current controller field, remove the `Controller` suffix from the controller name and convert it to a lowercase underscore, which is the name of the language package, e.g.: the controller name is `UserProfileController`, then the corresponding language package name is `user-profile.php`.

> {tip} If the field translation does not exist in the language package corresponding to the current controller, it is looked up in the public translation file `global.php`

```php
$name = admin_trans_field('name');
$createdAt = admin_trans_field('created_at');
```
The contents of the language package are as follows:
```php
return [
    'fields' => [
        'name' => '名称',
        'created_at' => '创建时间',
    ],
];
```


### admin_trans_label

To translate the current controller's custom content, remove the `Controller` suffix from the controller name and convert it to a lowercase, middle-arrow language package name.

> {tip} If the field translation does not exist in the language package corresponding to the current controller, it is looked up in the public translation file `global.php`.

```php
$user = admin_trans_label('User');
```
The contents of the language package are as follows:
```php
return [
    'labels' => [
        'User' => '管理员',
    ],
];
```

### admin_trans_option

To translate the current controller's field options, remove the `Controller` suffix from the controller name and convert it to a lowercase, middle-arrow language package name, e.g.: the controller name is `UserProfileController`, the corresponding language package name is `user-profile.php`.

> {tip} If the field translation does not exist in the language package corresponding to the current controller, it is looked up in the public translation file `global.php`.

```php
$status = admin_trans_option(1, 'status');
```
The contents of the language package are as follows:
```php
return [
    'options' => [
        'status' => [
            1 => '启用',
            0 => '禁用'
        ],
    ],
];
```

