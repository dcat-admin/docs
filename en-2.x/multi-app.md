# Multi-application (multi-backend)

The default installation is in single-application mode, if you want to use multi-application mode in the same `laravel` project. Then you can use a multi-backend model, and the directory structure in the final project would be as follows

```
app
 ├──Admin
 │   ├── Controllers
 │   │   ├── ExampleController.php
 │   │   └── HomeController.php
 │   ├── Metrics
 │   │   └── ...
 │   ├── bootstrap.php
 │   └── routes.php
 │
 ├──Admin2
 │    └── ...
 │   
 │──Admin3
 │    └── ...
 ...
```

### Generate new applications

Run command, this command only accepts one parameter: application name, please be sure to use **PascalCase** for the application name.

```php
php artisan admin:app NewAdmin
```

After running success a new application directory `app/NewAdmin` will be added to your project, as well as a new configuration file `config/new-admin.php`.

```
app
 └──Admin
    ├── Controllers
    │   ├── ExampleController.php
    │   └── HomeController.php
    ├── Metrics
    │   └── ...
    ├── bootstrap.php
    └── routes.php
config
 └──new-admin.php
```

### Enable

Once the new application has been generated, you can start enabling the new application by opening the configuration file `config/admin.php` and adding the following code

```php
return [
    ...
    
    'multi_app' => [
        // Consistent with the new application's profile name
        // set to true to enable, false to disable.
        'new-admin' => true,
    ],

];
```

Then you can open your browser to access the new app `http://localhost:8000/new-admin`。


### Change Route Prefix

Currently only through the route prefix to distinguish between different applications, if you want to change the application's prefix, you can open the configuration file `new-admin.php` to find the `route.prefix` parameter to change it!

### Change the menu

If you want to display a different menu in your new app, you can refer to the following methods

1. First you need to create a new menu table and its associated tables
```sql
CREATE TABLE `new_admin_menu` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `parent_id` int(11) NOT NULL DEFAULT '0',
  `order` int(11) NOT NULL DEFAULT '0',
  `title` varchar(50) COLLATE utf8mb4_unicode_ci NOT NULL,
  `icon` varchar(50) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `uri` varchar(50) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `created_at` timestamp NULL DEFAULT NULL,
  `updated_at` timestamp NULL DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

CREATE TABLE `new_admin_permission_menu` (
  `permission_id` int(11) NOT NULL,
  `menu_id` int(11) NOT NULL,
  `created_at` timestamp NULL DEFAULT NULL,
  `updated_at` timestamp NULL DEFAULT NULL,
  UNIQUE KEY `admin_permission_menu_permission_id_menu_id_index` (`permission_id`,`menu_id`) USING BTREE
) ENGINE=MyISAM DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

CREATE TABLE `new_admin_role_permissions` (
  `role_id` int(11) NOT NULL,
  `permission_id` int(11) NOT NULL,
  `created_at` timestamp NULL DEFAULT NULL,
  `updated_at` timestamp NULL DEFAULT NULL,
  UNIQUE KEY `admin_role_permissions_role_id_permission_id_index` (`role_id`,`permission_id`) USING BTREE
) ENGINE=MyISAM DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

2.Create a new menu model, and note that the default menu model needs to be inherited here!!!
```php
<?php

namespace App\Models;

use Dcat\Admin\Models\Menu;

class NewMenu extends Menu
{
    protected $table = 'new_admin_menu';
}
```

3.Open the new application's configuration file `config/new-admin.php` and modify the following parameters
```php
return [
    ...
	
	'database' => [

	  ...

	  // Writing a new model and menu table
	  'menu_table' => 'new_admin_menu',
	  'menu_model' => App\Models\NewMenu::class,

      ...
	  
	  // New intermediate table
	  'role_menu_table' => 'new_admin_role_menu',
	  'permission_menu_table' => 'new_admin_permission_menu',
	],
];
```

So that new applications can use the separate menu function

### Changing users and permissions

For custom users and permissions, you can refer to the way to change the menu above. In addition, if you are a custom user, you need to change the following parameters in the configuration file `config/new-admin.php`.

```php
   ...

   'auth' => [
        ...
        
        'guard' => 'new-admin', // It has to be a new name.
        
		'guards' => [
			'new-admin' => [
				'driver'   => 'session',
				'provider' => 'new-admin', // It has to be a new name.
			],
		],

		'providers' => [
			'new-admin' => [ // It has to be a new name.
				'driver' => 'eloquent',
				// Here's the new user table model instead
				'model'  => App\Models\NewAdministrator::class,
			],
		],

        ...

    ],
```

### Use different domain names to distinguish applications

By default, applications are distinguished by routing prefixes. If you want to distinguish applications using domain names, you just need to change the following configuration

```php
    'route' => [
        'domain' => 'dev.dcat.com', // configure your domain name

        'prefix' => '', // the route prefix is recommended to be set to empty

        'namespace' => 'App\\Admin\\Controllers',

        'middleware' => ['web', 'admin'],
    ],
```

