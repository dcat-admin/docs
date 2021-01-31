# 扩展基本使用

## 基本使用

### 设置扩展相关目录读写权限

在使用扩展功能之前，需要先保证所在用户拥有扩展相关目录的读写权限，否则可能造成扩展安装失败，请保证拥有以下几个目录的读写权限

1. `项目目录/dcat-admin-extensions` 扩展的安装目录，可以根据配置参数 `admin.extensions.dir` 进行更改
2. `public/vendor` 扩展静态资源发布目录
3. `resources/lang` 语言包目录


### 扩展安装

`Dcat Admin`中扩展支持以下三种安装方式，安装成功后均能在扩展管理页面`admin/auth/extensions`看到相关扩展信息

#### 1.通过应用市场安装

敬请期待...

#### 2.本地安装

下载扩展的`zip`压缩包，注意必须是`zip`格式，然后打开扩展管理页面`admin/auth/extensions`，然后点击表格工具栏的`本地安装`按钮上传提交即可。

#### 3.composer安装

根据扩展开发者文档提供的说明，直接使用composer安装即可

```bash
composer require {扩展名称}
```

#### 启用扩展

安装之后，需要在扩展管理页面点击 `更新至xxx版本` 以及更新 `启用` 按钮之后方可正常使用


## 开发扩展

详细的开发教程，请参考文档 [开发扩展](extension-dev.md) 章节。


<a name="service"></a>
### 服务注册与初始化

> {tip} 如果你对服务提供者的概念并不熟悉，请先前往[Laravel文档 - 服务提供者](https://learnku.com/docs/laravel/8.x/providers/9362)学习。

扩展的 `ServiceProvider` 类实际上是一个[服务提供者](https://learnku.com/docs/laravel/8.x/providers/9362)，唯一的区别是扩展的 `ServiceProvider` 不能重写 `boot` 方法，需要通过 `init` 代替 `boot`方法。


<a name="version"></a>
### 版本管理

每个扩展都有一个`version.php`文件，通过这个文件可以实现版本管理功能，每次发布新版本我们只需要往这个文件添加新的版本号以及相关描述即可

```php
<?php

return [
    // key 是版本号，注意这里不要带 v 前缀！
    '1.0.0' => [
        '版本描述信息，可以有多条',
        '描述2...',
        'create_operation_log.php', // 版本迁移文件，可以有多条
    ],
    
    '1.0.1' => [
        '版本描述信息，可以有多条',
        'update_operation_log.php', // 版本迁移文件
    ],
    
    ...
];
```


#### 升级版本

安装了新的版本代码之后，可以在扩展管理页面`admin/auth/extensions`中点击更新按钮进行升级。

点击升级后如果有迁移文件则会运行迁移文件，如果有菜单则会重新创建菜单，如果有静态资源则会自动重新发布资源文件。


#### 回滚版本

通过命令 `php artisan admin:ext-rollback {扩展名称} {版本号}` 可以回滚到指定版本，但需要注意的是，回滚扩展会删除数据，可能会导致数据丢失，请谨慎操作！！！

#### 卸载

如果你的扩展已安装，通过扩展页面 `admin/auth/extensions` 可以扩展进行卸载，但需要注意的是，卸载扩展会删除数据，可能会导致数据丢失，请谨慎操作！！！


如果你想完全移除扩展的代码，则直接删除 `dcat-admin-extensions` 目录下对应的扩展文件夹即可。

<a name="view"></a>
### 视图 (view)

视图的默认目录为`扩展目录/resources/view`

```bash
├── resources 
│   ├── ...
│   └── views # 视图目录
│       └── index.blade.php # 视图示例文件
```

只要把视图文件放在上述目录，系统就会自动给视图目录注册别名，别名与扩展名称相同。假设你的扩展包名称为 `dcat-admin/form-step`，则可以通过以下方式加载视图

```php
return view('dcat-admin.form-step::index');
```

<a name="assets"></a>
### 静态资源

假设你的扩展包名称为 `dcat-admin/form-step`，如果你的扩展中包含静态资源如下，那么你可以通过 `FormStepServiceProvider` 上的 `$js` 和 `$css` 属性为你的静态资源注册别名

```php
└── resources 
    └─── assets
      ├── css
      │   └── index.css
      └── js
          └── index.js
```

```php
class FormStepServiceProvider extends ServiceProvider
{
    protected $js = [
        'js/index.js',
    ];
    protected $css = [
        'css/index.css',
    ];
}
```

然后就可以通过下面的方法加载静态资源

```php
use Dcat\Admin\Admin;

// 直接用你的 包名 即可引入扩展包的静态资源
Admin::requireAssets('@dcat-admin.form-step');
```


当然你也可以不通过 `$js` 和 `$css` 属性注册别名，那么也可以用下面的方法直接加载静态资源，效果是一样的

```php
// 上面的写法相当于
Admin::js(['@dcat-admin.form-step/js/index.js']);
Admin::css(['@dcat-admin.form-step/css/index.css']);
```
