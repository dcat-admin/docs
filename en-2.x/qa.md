# FAQ Summary

### 1. How do I set the language to Simplified Chinese?

Open the configuration file `config/app.php` and set the value of the `locale` parameter to `zh_CN`.

### 2. Laravel7 time is displayed in UTC format

This is a pitfall brought on by the `Laravel7` upgrade, see [Date Serialization] for the reason.(https://learnku.com/docs/laravel/7.x/upgrade/7445#date-serialization)。

Solving this problem in this project is simple, just introduce `Dcat\Admin\Traits\HasDateTimeFormatter` as `trait` in `Model`.

```php
<?php

namespace App\Models;

use Dcat\Admin\Traits\HasDateTimeFormatter;
use Illuminate\Database\Eloquent\Model;

class MyModel extends Model
{
     use HasDateTimeFormatter;
}
```

### 3. Form save time error `Array to string conversion`

This problem occurs because the values submitted by the form end up being converted to `array` type, and `MySQL` does not support storing data of type `array` directly

```php
$form->multipleSelect('user_id')->saving(function ($v) {
    // Convert to , separated string
    return implode(',', $v);
});
```

Of course, you can also use `model`'s **mutator** to convert the values of the fields, which can be found in the `laravel` documentation, so I won't repeat that here.

> {tip} For a more elegant way to convert values, see [Dcat Admin Tutorial - How to Elegantly Change the Data Type of Form Values?](https://learnku.com/articles/44386)


### 4. How do I migrate from laravel-admin to dcat-admin?
[Dcat Admin Tutorial - How do I migrate from Laravel admin to dcat admin?](https://learnku.com/articles/44235)

### 5. Rewrite the login page and login logic

Way 1, rewrite the login controller method.

The default login controller uses the class `App\Admin\AuthController`, which can be modified by configuring the parameter `admin.auth.controller`.

```php
<?php

namespace App\Admin\Controllers;

use Dcat\Admin\Controllers\AuthController as BaseAuthController;

class AuthController extends BaseAuthController
{
    // Custom login view template
    protected $view = 'admin.login';
	
	// Rewriting login page logic
	public function getLogin(Content $content)
    {
        ...
    }

    ...
}

```


Mode 2, override routing.

In the routing file `app/Admin/routes.php`, override the routing of the landing page and login logic to implement custom features

```php
Route::group([
    'prefix'        => config('admin.prefix'),
    'namespace'     => Admin::controllerNamespace(),
    'middleware'    => ['web', 'admin'],
], function (Router $router) {

    $router->get('auth/login', 'AuthController@getLogin');
    $router->post('auth/login', 'AuthController@postLogin');
    
});
```

Implement your own login page and login logic in the `getLogin` and `postLogin` methods of your custom router AuthController.


### 6. Exceptions after updating to a new version

If you encounter an update, some components do not work properly, it is possible that `dcat-admin` comes with a static resource has been updated. You need to run the command `php artisan admin:publish --force` to republish the front-end resources. Do not forget to clean your browser cache after publishing.

### 7. File upload failed or inaccessible?

If you find that you are unable to upload a file, then there are usually several reasons for this.

1. `Laravel` file upload is not configured correctly, please refer to the document [Image/File Upload](https://learnku.com/docs/dcat-admin/1.x/picture-file-upload/8106). If you are not familiar with the `laravel` file upload feature, please read the documentation [Laravel - File Storage](https://learnku.com/docs/laravel/7.x/filesystem/7485).
2. The file is too large and needs to be adjusted with the `upload_max_filesize` parameter of `php.ini`.
3. File upload directory without write permissions
4. `php` is not installed or the `fileinfo` extension is not enabled
5. Check if the `upload_tmp_dir` parameter of `php.ini` is set properly.

If the file is uploaded success, but cannot be accessed normally, then it may be that the `.env` configuration file's `APP_URL` parameter is not set correctly.

### 8. About front-end resource loading problems

The `Dcat Admin` supports loading front-end resources on-demand, so that when a component is needed, front-end resources can be introduced, and developers do not need to worry about installing too many components to affect the page loading speed.

Only those resources that need to be introduced in the global page need to be introduced in the `app/Admin/bootstrap.php` or `ServiceProvider::boot` method:

```php
Admin::css('path/to/your/css');
Admin::js('path/to/your/js');
```

### 9. Why does the configuration of roles and permissions still prompt for no access rights?

The reason for this may be due to a misconfiguration of the `URL` path of the privilege, the correct `URL` configuration containing the add/drop check function should be something like `auth/users*`, if it is configured as `auth/users/*`, then it will prompt for unprivileged access.

> {tip} In addition, there are two ways to fill in the custom URL of the tag form: one is to select it and press ``delete key`` to change it; the other is to fill in the form and press ``space bar`` + ``Enter key``.

### 10. Why don't menus with no permissions automatically hide?

This problem is because there are no permissions or roles to bind to menus, just bind permissions or roles to menus that you want to show without permissions.


### 11. Project cannot log in after using HTTPS

The value of the `admin.https` parameter of the configuration file needs to be set to `true`.


### 12.$.get(xxx) no response

`Dcat Admin` is using `jQuery3.x`, the `$.get` method is deprecated in `jQuery3.x`, please use `$.ajax` instead!

### 13. Do uploaded images often delete automatically for no apparent reason?

This problem is most likely due to the fact that `Model` has set an accessor to the fields related to the image, such as

```php
class User extend Model
{
    public function getAvatarAttribute()
    {
        return Storage::disk('admin')->url($this->avatar);
    }
}
```

In this case, there will be automatic deletion of uploaded images, the solution is simple, change the access to the custom method can be

```php
class User extend Model
{
    public function getAvatar()
    {
        return Storage::disk('admin')->url($this->avatar);
    }
}
```

### 14. Frontend and backend session conflict

Since `2.0` version `admin.session` middleware is no longer enabled by default, if your application has both frontend and backend, you need to enable `admin.session` middleware, otherwise it will cause front and backend `session` conflict problem.

Set the value of the configuration parameter `admin.route.enable_session_middleware` to `true` to enable it
```php
    'route' => [
        'domain' => env('ADMIN_ROUTE_DOMAIN'),

        'prefix' => env('ADMIN_ROUTE_PREFIX', 'admin'),

        'namespace' => 'App\\Admin\\Controllers',

        'middleware' => ['web', 'admin'],
        
        // enable admin.session middleware
        'enable_session_middleware' => true,
    ],
```

### 15. Image anti-theft chain
Image requests will have the `referer` field removed by default, but if you have a security chain requirement, you can set this in the configuration file (`config/admin.php`).

```
 "disable_no_referrer_meta" => true
```
