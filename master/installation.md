# 安装

<a name="env"></a>
## 环境
+ PHP >= `7.1`
+ Laravel `5.5.0` ~ `7.*`
+ Fileinfo PHP Extension

<a name="start"></a>
## 开始安装

首先需要安装`laravel`，如已安装可以跳过此步骤
```bash
composer create-project --prefer-dist laravel/laravel 项目名称 7.*
# 或
composer create-project --prefer-dist laravel/laravel 项目名称
```

安装完`laravel`之后需要设置数据库连接设置正确

```
composer require dcat/laravel-admin
```

然后运行下面的命令来发布资源：

```
php artisan admin:publish
```

在该命令会生成配置文件`config/admin.php`，可以在里面修改安装的地址、数据库连接、以及表名，建议都是用默认配置不修改。

然后运行下面的命令完成安装：

> {tip} 执行这一步命令可能会报以下错误“Specified key was too long ... 767 bytes”，如果出现这个报错，则在`app/Providers/AppServiceProvider.php`文件的`boot`方法中加上代码`\Schema::defaultStringLength(191);`，再重新运行一遍`php artisan admin:install`命令即可。

```
php artisan admin:install
```


启动服务后，在浏览器打开 `http://localhost/admin/` ,使用用户名 `admin` 和密码 `admin`登陆.

<a name="files"></a>
## 生成的文件

安装完成之后,会在项目目录中生成以下的文件:

<a name="config"></a>
### 配置文件

安装完成之后，`dcat-admin`所有的配置都在`config/admin.php`文件中。

<a name="admin"></a>
### 后台项目文件
安装完成之后，后台的安装目录为`app/Admin`，之后大部分的后台开发编码工作都是在这个目录下进行。

```
app/Admin
├── Controllers
│   ├── ExampleController.php
│   └── HomeController.php
├── Metrics
│   └── Examples
│       ├── NewDevices.php
│       ├── NewUsers.php
│       ├── ProductOrders.php
│       ├── Sessions.php
│       ├── Tickets.php
│       └── TotalUsers.php
├── bootstrap.php
└── routes.php
```

1. `app/Admin/routes.php`文件用来配置后台路由。
2. `app/Admin/bootstrap.php` 是`dcat-admin`的启动文件, 使用方法请参考文件里面的注释.
3. `app/Admin/Controllers`目录用来存放后台控制器文件，该目录下的`HomeController.php`文件是后台首页的显示控制器，`ExampleController.php`为实例文件。
4. `app/Admin/Metrics/Examples`里面存放的是`数据统计卡片(Metric Card)`的示例代码.

<a name="assets"></a>
### 静态文件

后台所需的前端静态文件在`/public/vendor/dcat-admin`目录下。

<a name="migrations"></a>
### 数据表迁移文件
对应的数据表迁移文件在`/database/migrations`目录下。

<a name="lang"></a>
### 语言包
语言包文件在`/resources/lang`目录下。

## 版本升级

升级命令
```bash
composer update dcat/laravel-admin
```

升级成功之后一般需要重新发布语言包、配置文件、前端静态资源和数据库迁徙等文件
```bash
php artisan admin:publish --force
```

只更新语言包
```bash
php artisan admin:publish --force --lang
```

只更新配置文件
```bash
php artisan admin:publish --force --config
```


只更新前端静态资源
```bash
php artisan admin:publish --force --assets
```


只更新数据库迁徙文件
```bash
php artisan admin:publish --force --migrations
```