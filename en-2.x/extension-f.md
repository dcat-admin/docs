# Extension of basic use

## Basic use

### Set the read and write permissions for extension-related directories

Before using the extension function, you need to ensure that the user has the read and write permissions of the extension related directories, otherwise it may cause the extension installation to fail, please ensure that you have the read and write permissions of the following directories

1. `ProjectDirectory/dcat-admin-extensions` extension installation directory, which can be changed according to the configuration parameter `admin.extensions.dir`
2. `public/vendor` extends the static resource distribution directory
3. `resources/lang` language package directory


### Extension installation

The extensions in `Dcat Admin` support the following three installation methods, and after installing success, you can see the related extension information in `admin/auth/extensions` extension management page.

#### 1. Install via the App Market

Stay tuned...

#### 2. Local installation

Download the extended `zip` archive, note that it must be `zip` format, and then open the extensions management page `admin/auth/extensions`, and then click the form toolbar `local installation` button to upload and submit.

#### 3.composer installation

Install directly with composer according to the instructions provided in the extended developer documentation.

```bash
composer require {extension name}
```

#### enable extensions

After installation, you need to click `Update to xxx version` and update the `Enable` button on the extensions management page before you can use it normally.


## Developing extensions

For a detailed tutorial, please refer to the [Development Extensions](extension-dev.md) chapter.


<a name="service"></a>
### Service registration and initialization

> {tip} If you're not familiar with the concept of a service provider, head over to [Laravel Documentation - Service Providers](https://learnku.com/docs/laravel/8.x/providers/9362) to learn it first.

The extended `ServiceProvider` class is actually a [service provider](https://learnku.com/docs/laravel/8.x/providers/9362), the only difference is that the extended `ServiceProvider` can't override `boot`! method, you need to replace the `boot` method with `init`.


<a name="version"></a>
### Version management

Each extension has a `version.php` file, through this file you can achieve the version management function, every time we release a new version we just need to add a new version number and the relevant description to this file

```php
<?php

return [
    // key It's the version number, be careful not to prefix it with v!
    '1.0.0' => [
        'Version description information, can have multiple',
        'Description 2...',
        'create_operation_log.php', // Version migration file, can have multiple
    ],
    
    '1.0.1' => [
        'Version description information, can have multiple',
        'update_operation_log.php', // Version migration files
    ],
    
    ...
];
```


#### Upgrades

After installing the new version of the code, you can click the update button on the extension management page `admin/auth/extensions` to upgrade.

Clicking update will run the migration file if there is a migration file, recreate the menu if there is a menu, and republish the resource file if there is a static resource.


#### Rollback version

You can roll back to the specified version by commanding `php artisan admin:ext-rollback {extension name} {version number}`, but be aware that rolling back extensions will delete data and may result in data loss, so please be careful!!!!

#### Uninstallation

If your extension is already installed, you can uninstall the extension through the extension page `admin/auth/extensions`, but be aware that uninstalling the extension will delete data and may result in data loss, please be careful!


If you want to remove the extensions completely, just delete the extensions folder in the `dcat-admin-extensions` directory.

<a name="assets"></a>
### Static resources

Assuming your extension package name is `dcat-admin/form-step`, you can register aliases for your static resources via the `$js` and `$css` attributes on `FormStepServiceProvider` if your extension contains static resources as follows

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

Then you can load the static resource by doing the following

```php
use Dcat\Admin\Admin;

// Use your package name directly to bring in the static resources of the extensions!
Admin::requireAssets('@dcat-admin.form-step');
```


Of course, you can also load static resources without registering aliases via the `$js` and `$css` attributes, but you can also load them directly using the following method, with the same effect

```php
// The above is equivalent to
Admin::js(['@dcat-admin.form-step/js/index.js']);
Admin::css(['@dcat-admin.form-step/css/index.css']);
```
