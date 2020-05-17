# 多应用 (多后台)

> {tip} Since `v1.4.0`

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



