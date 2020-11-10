# Help function

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

