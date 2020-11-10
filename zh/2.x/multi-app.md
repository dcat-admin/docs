# 多应用 (多后台)

默认安装后使用的是单应用模式，如果你想在同一个`laravel`项目中使用多应用模式，那么可以采用多后台模式，最终项目中的目录结构大概如下

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

### 生成新应用

运行命令，此命令只接受一个参数：应用名称，注意这里的应用名称请一定要使用**大驼峰风格**命名

```php
php artisan admin:app NewAdmin
```

运行成功后你的项目中会新增一个新的应用目录`app/NewAdmin`，以及新的配置文件`config/new-admin.php`

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

### 启用

新应用生成完之后，就可以开始启用这个新应用了，打开配置文件`config/admin.php`，加入以下代码

```php
return [
    ...
    
    'multi_app' => [
        // 与新应用的配置文件名称一致
        // 设置为true启用，false则是停用
        'new-admin' => true,
    ],

];
```

然后就可以打开浏览器访问这个新应用了`http://localhost:8000/new-admin`。


### 更改路由前缀

目前只能通过路由前缀区分不同应用，如果你想要更改应用的前缀，可以打开配置文件`new-admin.php`找到`route.prefix`参数进行更改即可

### 更改菜单

如果你想要在新应用中展示不同的菜单，可以参考以下方法

1.首先需要创建新的菜单表以及其关联表
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

2.创建新的菜单模型，注意这里需要继承默认的菜单模型！！！
```php
<?php

namespace App\Models;

use Dcat\Admin\Models\Menu;

class NewMenu extends Menu
{
    protected $table = 'new_admin_menu';
}
```

3.打开新应用的配置文件`config/new-admin.php`，然后修改以下参数
```php
return [
    ...
	
	'database' => [

	  ...

	  // 写入新的模型和菜单表
	  'menu_table' => 'new_admin_menu',
	  'menu_model' => App\Models\NewMenu::class,

      ...
	  
	  // 新的中间表
	  'role_menu_table' => 'new_admin_role_menu',
	  'permission_menu_table' => 'new_admin_permission_menu',
	],
];
```

这样新的应用就可以使用独立的菜单功能了

### 更改用户和权限

自定义用户和权限可以参考以上更改菜单的方式。另外如果是自定义用户的话，还需要更改配置文件`config/new-admin.php`中的以下参数

```php
   ...

   'auth' => [
        ...
        
        'guard' => 'new-admin', // 必须是一个新的名字
        
		'guards' => [
			'new-admin' => [
				'driver'   => 'session',
				'provider' => 'new-admin', // 必须是一个新的名字
			],
		],

		'providers' => [
			'new-admin' => [ // 必须是一个新的名字
				'driver' => 'eloquent',
				// 这里换成新用户表的模型
				'model'  => App\Models\NewAdministrator::class,
			],
		],

        ...

    ],
```



