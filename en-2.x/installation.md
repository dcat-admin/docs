# 安装

<a name="env"></a>
## Requirements
+ PHP >= `7.1`
+ Laravel `5.5.0` ~ `8.*`
+ Fileinfo PHP Extension

<a name="start"></a>
## Start installing

> {tip} If `composer` downloads too slowly or fails to install during installation
        Please run the command `composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/` to replace the `composer` packagist server with the Aliyun mirror.

First you need to install `laravel`, if you already have it installed you can skip this step.
```bash
composer create-project --prefer-dist laravel/laravel ProjectName 7.*
# 或
composer create-project --prefer-dist laravel/laravel ProjectName
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
cd {ProjectName}

composer require dcat/laravel-admin:"2.*" -vvv
```

Then run the following command to publish the resource:

```
php artisan admin:publish
```

The command will generate the configuration file `config/admin.php`, in which you can modify the installation address, database connection, and table name, it is recommended that all the default configuration is not modified.

Then run the following command to complete the installation:

> {tip} Executing this command may result in the following error `Specified key was too long ... 767 bytes`
        If this error occurs, add the code `\Schema::defaultStringLength(191);` to the `boot` method of the `app/Providers/AppServiceProvider.php` file.
        Then delete all the data tables in the database and run the `php artisan admin:install` command again.

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

1. `app/Admin/routes.php` file is used to configure background routing.
2. `app/Admin/bootstrap.php` It is the startup file of `dcat-admin`, please refer to the comments in the file for how to use it.
3. `app/Admin/Controllers` directory is used to store the backend controller files, the `HomeController.php` file in this directory is the backend home page display controller, `AuthController.php` is the backend administrator login authentication controller.
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

