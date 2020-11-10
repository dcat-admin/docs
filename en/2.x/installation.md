# 安装

<a name="env"></a>
## Requirements
+ PHP >= `7.1`
+ Laravel `5.5.0` ~ `8.*`
+ Fileinfo PHP Extension

<a name="start"></a>
## Start installing

> {tip} 如果安装过程中出现`composer`下载过慢或安装失败的情况，请运行命令`composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/`把`composer`镜像更换为阿里云镜像。

First you need to install `laravel`, if you already have it installed you can skip this step.
```bash
composer create-project --prefer-dist laravel/laravel 项目名称 7.*
# 或
composer create-project --prefer-dist laravel/laravel 项目名称
```

After installing `laravel`, you will need to modify the `.env` file to set the database connection to the correct settings.

```dotenv
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=dcat-admin
DB_USERNAME=root
DB_PASSWORD=
```

Installation of `dcat-admin`


```
cd {项目名称}

composer require dcat/laravel-admin:"2.*" -vvv
```

Then run the following command to publish the resource:

```
php artisan admin:publish
```

The command will generate the configuration file `config/admin.php`, in which you can modify the installation address, database connection, and table name, it is recommended that all the default configuration is not modified.

Then run the following command to complete the installation:

> {tip} 执行这一步命令可能会报以下错误`Specified key was too long ... 767 bytes`，如果出现这个报错，请在`app/Providers/AppServiceProvider.php`文件的`boot`方法中加上代码`\Schema::defaultStringLength(191);`，然后删除掉数据库中的所有数据表，再重新运行一遍`php artisan admin:install`命令即可。

```
php artisan admin:install
```

After the above steps are completed, you can configure the webserver, ** Note that you need to point the document root to the `public` directory**! If you are using `nginx`, you also need to add a pseudo-static configuration to the configuration:
```dotenv
location / {
	try_files $uri $uri/ /index.php?$query_string;
}
```

After starting the service, open `http://localhost/admin` in your browser and log in with the username `admin` and password `admin`.


<a name="files"></a>
## Generated documents

After installation, the following file is generated in the project directory:

<a name="config"></a>
### Configuration file

After the installation is complete, all the configuration for `dcat-admin` is in the `config/admin.php` file.

<a name="admin"></a>
### Back-end project document
After the installation is complete, the backend installation directory is `app/Admin`, after which most of the backend development coding work is done in this directory.

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

1. `app/Admin/routes.php` file is used to configure background routing.
2. `app/Admin/bootstrap.php` It is the startup file of `dcat-admin`, please refer to the comments in the file for how to use it.
3. `app/Admin/Controllers` directory is used to store the background controller file, the `HomeController.php` file in this directory is the display controller of the background home page, `ExampleController.php` is an example file.
4. `app/Admin/Metrics/Examples` contains the `statistics card (Metric Card)` sample code.

<a name="assets"></a>
### Static files

The front-end static files needed for the backend are in the `/public/vendor/dcat-admin` directory.

<a name="migrations"></a>
### Data table migration file
The corresponding data table migration file is in the `/database/migrations` directory.

<a name="lang"></a>
### Language packs
The language package files are located in the directory `/resources/lang`.

