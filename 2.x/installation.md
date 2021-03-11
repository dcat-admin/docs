# 安装

<a name="env"></a>
## 环境
+ PHP >= `7.1`
+ Laravel `5.5.0` ~ `8.*`
+ Fileinfo PHP Extension

<a name="start"></a>
## 开始安装

> {tip} 如果安装过程中出现`composer`下载过慢或安装失败的情况，请运行命令`composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/`把`composer`镜像更换为阿里云镜像。

首先需要安装`laravel`，如已安装可以跳过此步骤
```bash
composer create-project --prefer-dist laravel/laravel 项目名称 7.*
# 或
composer create-project --prefer-dist laravel/laravel 项目名称
```

安装完`laravel`之后需要修改`.env`文件，设置数据库连接设置正确

```dotenv
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=dcat-admin
DB_USERNAME=root
DB_PASSWORD=
```

安装`dcat-admin`


```
cd {项目名称}

composer require dcat/laravel-admin:"2.*" -vvv
```

然后运行下面的命令来发布资源：

```
php artisan admin:publish
```

在该命令会生成配置文件`config/admin.php`，可以在里面修改安装的地址、数据库连接、以及表名，建议都是用默认配置不修改。

然后运行下面的命令完成安装：

> {tip} 执行这一步命令可能会报以下错误`Specified key was too long ... 767 bytes`，如果出现这个报错，请在`app/Providers/AppServiceProvider.php`文件的`boot`方法中加上代码`\Schema::defaultStringLength(191);`，然后删除掉数据库中的所有数据表，再重新运行一遍`php artisan admin:install`命令即可。

```
php artisan admin:install
```

上述步骤操作完成之后就可以配置`web`服务了，**注意需要把`web`目录指向`public`目录**！如果用的是`nginx`，还需要在配置中加上伪静态配置
```dotenv
location / {
	try_files $uri $uri/ /index.php?$query_string;
}
```

启动服务后，在浏览器打开 `http://localhost/admin`，使用用户名 `admin` 和密码 `admin`登陆。


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
│   ├── AuthController.php
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
3. `app/Admin/Controllers`目录用来存放后台控制器文件，该目录下的`HomeController.php`文件是后台首页的显示控制器，`AuthController.php`为后台管理员登录鉴权控制器。
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

