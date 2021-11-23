# Developing extensions

## New Release Preview
``Dcat Admin`` plans to go live with the plug-in marketplace feature in ``2.0`` version, which will refactor the entire extension functionality to enhance the user experience.
The new extension system will allow users to `install`, `update` and `uninstall` plug-ins with just a few clicks of the mouse from the administration page.
The new extension system will allow users to `install`, `update` and `uninstall` plug-ins with just a few mouse clicks from the administration page.

The `Dcat Admin` team will work hard to build an ecosystem that benefits both developers and users, thanks for your support!


<a name="pre"></a>
# Start
`Dcat Admin` supports the installation of extensions to help enrich your backend functionality.
>It should be noted that the original extensions for `Laravel Admin` cannot be used directly in `Dcat Admin`, but most of them can be used with a few minor tweaks and can be migrated.

If you are using `Dcat Admin` to add some of your own features or components, you can make a `Dcat Admin` extension, which can help other `Dcat Admin` users and improve the quality of the extension in other people's feedback.

This document will take the development of a `Dry Fish Camp` extension as an example, step-by-step development of an extension, and released to others to use, the final results of the reference [Demo] ().

<a name="composer"></a>
## Creating Composer Packages

The `Dcat Admin` package will be installed with `composer`, so first create a `composer` package, you can use the built-in `admin:extend` command to generate an extension skeleton.


When running the command, you may be prompted to enter a directory to store your extensions, you can add a configuration `'extension_dir' => admin_path('Extensions')` in `config/admin.php` so that the extensions will be stored in `app/Admin/` Extensions directory, but you can also put it in any other directory.


```php
php artisan admin:ext-make dcat-admin-extensions/gank --namespace="Dcat\Admin\Extension\Gank"
```

The `dcat-admin-extensions/gank` is the package name, and the `namespace` option is the top-level namespace used by the package, after running this command, the directory `dcat-admin-extensions/gank` will be generated in the extension directory set in `config/admin.php` and the following document structure:

    ├── LICENSE
    ├── README.md
    ├── composer.json
    ├── bootstrap.php
    ├── database
    │   ├── migrations
    │   └── seeds
    ├── resources
    │   ├── assets
    │   └── views
    │       └── index.blade.php
    ├── routes
    │   └── web.php
    └── src
        ├── Setting.php
        ├── GankServiceProvider.php
        └── Http
            └── Controllers
                └── GankController.php

`bootstrap.php` is used to register the extension, the code in this file does not need any modification; `resources` is used to place the view files and static resource files; `src` is mainly used to place the logical code; `routes/web.php` is used to store the routing settings for this extension; `database` is used to place the database migrations. Files and data `seeders`.

<a name="dev"></a>
## Functional development

The functionality of this extension is primarily used to display the contents of the [gank.io](http://gank.io) interface, integrated into `Dcat Admin`, which will have a route and a controller with no database files, templates, or static resource files, and we can clean up any files or directories that we don't use, and the cleaned up directory file will be.

    ├── LICENSE
    ├── README.md
    ├── composer.json
    ├── bootstrap.php
    ├── routes
    │   └── web.php
    └── src
        ├── Gank.php
        ├── GankServiceProvider.php
        └── Http
            └── Controllers
                └── GankController.php
                
After generating the extension framework, you may need to debug while developing, so refer to the following local installation, first install the extension into the system, continue development

<a name="router"></a>
### Adding routes
First add a route, a route configuration is already generated automatically in `routes/web.php`.

```php
<?php

use Dcat\Admin\Extension\Gank\Http\Controllers;

Route::get('gank', Controllers\GankController::class.'@index');
```

The access path `http://localhost:8000/admin/gank` will be handled by the `index` method of the `Dcat\Admin\Extension\Gank\Http\Controllers\GankController` controller.

<a name="attrs"></a>
### Setting Extended Properties
`src/Gank.php` is used as an extension class to set the properties of the extension.

```php
<?php

namespace Dcat\Admin\Extension\Gank;

use Dcat\Admin\Extension;

class Gank extends Extension
{
    // Extended aliases for getting configuration information in configuration files
    const NAME = 'gank';

    // service provider class name, just use the auto-generated one, no need to change it.
    public $serviceProvider = GankServiceProvider::class;

    // Static resource directory, which can be deleted if no static resources are needed.  
//    public $assets = __DIR__.'/../resources/assets';

    // composer configuration file path, just use the default.
    public $composerJson = __DIR__.'/../composer.json';

    // custom properties
    public static $categoryColorsMap = [
        'App'      => 'var(--purple)',
        'Front End''     => 'var(--primary)',
        'Recommendation' => 'var(--primary-dark)',
        'Welfare'   => 'var(--blue)',
        'Explosion'     => 'var(--danger)',
        'Android'  => 'var(--purple-dark)',
        'iOS'      => 'var(--info)',
        'Break' => 'var(--warning)',
    ];
}

```
This file is used to set some properties of the extension, `$name` is the alias of the extension, if the extension has a view file to render, you must specify the `$views` property of the extension, similarly if the extension has a static resource file to publish, you must set the `$assets` property, if you need to add a menu button in the left sidebar, set the `$ The menu` attribute can be removed as needed.

Then open `src/GankServiceProvider.php`. This `ServiceProvider` will run when the extension is enabled (no need for the developer to add it) and will be used to register some of the extension's services into the system.

<a name="loadview"></a>
### Load view
If this extension requires a view file to be loaded, add the following code to the `boot` method of `src/GankServiceProvider.php`:

```php
$extension = Gank::make();

if ($extension->views) {
    $this->loadViewsFrom($extension->views, 'gank');
}
```
The first parameter of the `loadViewsFrom` method is the view property set in the extension class `src/Gank.php`, the second parameter is the namespace of the view file directory, after setting `gank`, use `view('gank::index')` to load `resources/views` in the controller. View file under the directory.

<a name="loadassets"></a>
### Introduce static resources
If you have static resource files that need to be introduced into your project, put them in the `resources/assets` directory, such as `resources/assets/foo.js` and `resources/assets/bar.css` files.

Then set the `$assets` attribute in the extension `src/Gank.php`.
```php
public $assets = __DIR__.'/../resources/assets';
```
    
After installation, the file will be copied to the `public/vendor/dcat-admin-extensions/gank` directory.

Then we can add the following code where we need to use these static resources

> {tip} Since `Dcat Admin` supports loading static resources on demand, it is not recommended to introduce static resources directly into `ServiceProvider` or `bootstrap.php`, as this will slow down the loading time of all pages.

```php
use Dcat\Admin\Admin;

Admin::js('vendor/dcat-admin-extensions/gank/phpinfo/foo.js');
Admin::css('vendor/dcat-admin-extensions/gank/phpinfo/bar.css');
```

This completes the introduction of static resources, which can be ignored in the `gank` extension since there are no static resources to introduce.

<a name="addmenu"></a>
### Adding menus
There are two ways to add menus.
- Set the `$menu` attribute in `src/Gank.php`, this way the menu is written into the menu table, which is easier for the user to control
- Use the `Admin::menu` interface to add menus by way of arrays, which is more convenient and supports adding multi-level menus.

The second way to add a menu is used here, adding the following code to the `boot` method of `src/GankServiceProvider.php`.
```php
Admin::menu()->add([
    [
        'id'            => 1,
        'title'         => `Dry Fish Camp`,
        'icon'          => ' fa-newspaper-o',
        'uri'           => 'gank',
        'parent_id'     => 0,
        'permission_id' => 'gank', // Binding privileges
        'roles'         => [['slug' => 'gank']], // Binding Roles
    ],
    [
        'id'            => 2,
        'title'         => 'All dry goods',
        'icon'          => 'fa-smile-o',
        'uri'           => 'gank',
        'parent_id'     => 1,
        'permission_id' => 'gank', // Binding privileges
        'roles'         => [['slug' => 'gank']], // Binding Roles
    ],
]);
```

The final `src/GankServiceProvider.php` code is as follows:
```php
<?php

namespace Dcat\Admin\Extension\Gank;

use Dcat\Admin\Admin;
use Illuminate\Support\ServiceProvider;

class GankServiceProvider extends ServiceProvider
{
    /**
     * {@inheritdoc}
     */
    public function boot()
    {
        $extension = Gank::make();

        if ($extension->views) {
            $this->loadViewsFrom($extension->views, 'gank');
        }

        if ($extension->lang) {
            $this->loadTranslationsFrom($extension->lang, 'gank');
        }

        if ($extension->migrations) {
            $this->loadMigrationsFrom($extension->migrations);
        }

        $this->app->booted(function () use ($extension) {
            $extension->routes(__DIR__.'/../routes/web.php');
        });

        // Add Menu
        $this->registerMenus();
    }

    protected function registerMenus()
    {
        Admin::menu()->add([
			[
				'id'            => 1,
				'title'         => `Dry Fish Camp`,
				'icon'          => ' fa-newspaper-o',
				'uri'           => 'gank',
				'parent_id'     => 0,
				'permission_id' => 'gank', // Binding privileges
				'roles'         => [['slug' => 'gank']], // Binding Roles
			],
			[
				'id'            => 2,
				'title'         => 'All dry goods',
				'icon'          => 'fa-smile-o',
				'uri'           => 'gank',
				'parent_id'     => 1,
				'permission_id' => 'gank', // Binding privileges
				'roles'         => [['slug' => 'gank']], // Binding Roles
			],
        ]);
    }
}
```

<a name="importcode"></a>
## Adding custom installation logic
If you want to perform some custom logic when importing extensions, you can add the following code to `src/Gank.php`.

>Please make sure that the import command is executed repeatedly without error.

```php
...

    public function import(Command $command)
    {
        parent::import($command); // TODO: Change the autogenerated stub
        
        // Write your import logic here
        
        $command->info('Import success');
        
    }

```

<a name="code"></a>
## Code logic development
This extension mainly calls the interface provided by [gank.io](), and then displays the data.

We need to add a new data warehouse file to capture data from [gank.io](), creating the file `src/Repositories/Ganks.php`.
```php
<?php

namespace Dcat\Admin\Extension\Gank\Repositories;

use Dcat\Admin\Grid;
use Dcat\Admin\Repositories\Repository;
use Illuminate\Pagination\LengthAwarePaginator;
use Illuminate\Support\Collection;

class Ganks extends Repository
{
    protected $api = 'http://gank.io/api/data/{category}/{limit}/{page}';

    protected $searchApi = 'http://gank.io/api/search/query/{key}/category/{category}/count/{limit}/page/{page}';

    /**
     * Inquiry Form Data
     *
     * @param Grid\Model $model
     * @return Collection
     */
    public function get(Grid\Model $model)
    {
        $currentPage = $model->getCurrentPage();
        $perPage = $model->getPerPage();

        // Get filter parameters
        $category = $model->filter()->input(Grid\Filter\Scope::QUERY_NAME, 'all');
        $keyword = trim($model->filter()->input('keyword'));

        $api = $keyword ? $this->searchApi : $this->api;

        $client = new \GuzzleHttp\Client();

        $response = $client->get(str_replace(
            ['{category}', '{limit}', '{page}', '{key}'],
            [$category, $perPage, $currentPage, $keyword],
            $api
        ));
        $data = collect(
            json_decode((string)$response->getBody(), true)['results'] ?? []
        );

        $total = $keyword ? 400 : ($category == 'all' ? 1000 : 500);

        $paginator = new LengthAwarePaginator(
            $data,
            $category == '福利' ? 680 : $total,
            $perPage, // Enter the number of lines per page
            $currentPage // Pass in the current page number
        );

        $paginator->setPath(\url()->current());

        return $paginator;
    }

    public function getKeyName()
    {
        return '_id';
    }

}
```

Modify the controller code as follows：

```php
<?php

namespace Dcat\Admin\Extension\Gank\Http\Controllers;

use Dcat\Admin\Extension\Gank\Gank;
use Dcat\Admin\Extension\Gank\Repositories\Ganks;
use Dcat\Admin\Grid;
use Dcat\Admin\Layout\Content;
use Dcat\Admin\Layout\Row;
use Illuminate\Routing\Controller;
use Dcat\Admin\Widgets\Navbar;

class GankController extends Controller
{
    public function index(Content $content)
    {
        $grid = $this->grid();

        $grid->disableFilter();
        $grid->filter()
            ->withoutInputBorder()
            ->expand()
            ->resetPosition()
            ->hiddenResetButtonText();

        return $content
            ->header('All dry goods')
            ->description('Share daily girl pictures and tech tricks')
            ->body($grid->filter())
            ->body(function (Row $row) {
                $items = array_keys(Gank::$categoryColorsMap);
                
                array_unshift($items, 'All');
                
                $navbar = Navbar::make('#', array_combine($items, $items))
                    ->checked(request(Grid\Filter\Scope::QUERY_NAME, 'All'))
                    ->click()
                    ->map(function ($v) {
                        if ($v == 'All') {
                            $url = '?';
                        } else {
                            $url = '?'.Grid\Filter\Scope::QUERY_NAME.'='.$v;
                        }
        
                        return "<a href='$url'>$v</a>";
                    })
                    ->style('max-width:705px');
                
                $row->column(7, $navbar);

            })
            ->body($grid);
    }

    protected function grid()
    {
        $this->define();

        $grid = new Grid(new Ganks);

        $grid->number();
        $grid->desc('Description')->width('300px');
        $grid->images('Pictures')->image(150);
        $grid->type('Categories');
        $grid->who('Authors')->label();
        $grid->publishedAt('Published at');

        $grid->disableActions();
        $grid->disableBatchDelete();
        $grid->disableExport();
        $grid->disableCreateButton();
        $grid->disableFilterButton();
        $grid->disableQuickCreateButton();
        $grid->perPages([]);

        return $grid->filter(function (Grid\Filter $filter) {
            $category = $filter->input(Grid\Filter\Scope::QUERY_NAME, 'All');

            if ($category != 'Benefits') {
                $filter->like('keyword', ucfirst($category))->width('300px')->placeholder('Please enter');
            }

        });
    }

    protected function define()
    {
        Grid\Column::define('desc', function ($v) {
            if ($this->type == 'Benefits') {
                $width = '150';
                $height = '200';
                return "<img data-init='preview' src='{$this->url}' style='max-width:{$width}px;max-height:{$height}px;cursor:pointer' class='img img-thumbnail' />";
            }

            return sprintf('<a href="%s" target="_blank">%s</a>', $this->url, $v);
        });

        Grid\Column::define('publishedAt', function ($v) {
            return date('Y-m-d', strtotime($v));
        });
        Grid\Column::define('datetime', function ($v) {
            return date('Y-m-d H:i:s', strtotime($v));
        });

        Grid\Column::define('type', function ($v) {
            $map = Gank::$categoryColorsMap;

            return "<span class='label' style='background:{$map[$v]}'>$v</span>";
        });
    }
}
```
This way a complete extension is developed.

<a name="updatejson"></a>
## Modify composer.json & README.md
After the code is partially complete, you need to modify the contents of `composer.json`, replace `description`, `keywords`, `license`, `authors` and other content with your information, and then do not forget to improve `README.md`, supplement the use of documentation and other related information.

<a name="install"></a>
## Installation
After completing the extension development, you can install your extensions as follows, depending on the situation

<a name="local"></a>
### Local installation
In the development process, you generally need to debug while developing, so first install locally as follows

Open the `composer.json` file in your project and add the following configuration to it

```
"repositories": [
    {
        "type": "path",
        "url": "app/Admin/Extensions/dcat-admin-extensions/gank"
    }
]
```
Then run `composer require dcat-admin-extensions/gank @dev` to complete the installation, and if you have static files to publish, run the following command:

```
php artisan vendor:publish --provider=Dcat\Admin\Extension\GankServiceProvider
```

<a name="remote"></a>
### Remote installation
If the development is complete, I hope to open source out for everyone to use, follow these steps

<a name="github"></a>
#### Upload to Github
Log in to your Github, create a repository, and then follow the instructions on the page to push your code up.

```
git init
git remote add origin https://github.com/<your-name>/<your-repository>.git
git add .
git commit -am "Initial commit."
git push origin master
```

<a name="release"></a>
#### Release
Local releases can be made in the following ways

```
git tag 0.0.1 && git push --tags
```
It can also be set up manually on the Releases page of Github's repository page.

<a name="packagist"></a>
#### Publish to Packagist.org
Next is to publish your project to `Packagist.org`, if you don't have an account, register one, then open `Submit` in the top navigation, fill in the repository address and submit it.

By default, `Packagist.org` is not automatically updated when you push new code, so you need to create a GitHub service hook. You can also update it manually by clicking the `Update` button on the page, but I recommend doing this automatically!

After committing, due to the delay in the synchronization time of the local mirror, you may not be able to find your project when installing with composer, you may need to wait for the synchronization to complete.

Once released, you can install your extensions through composer.

Add to https://github.com/dcat-admin-extensions
If you would like to add your extensions to `dcat-admin-extensions`, feel free to contact me in any way you can so that more people can see and use your tools.

<a name="import"></a>
## Importing and enabling extensions

<a name="importext"></a>
### Importing extensions
After installation, you can import extensions by running the following commands.

> Generally, you don't need to import views, database migration files, and language packs. If you want to import other files, you need to define them by the extension developer.
> Also this command allows repeat execution.

```php
php artisan admin:import Dcat\Admin\Extension\Gank\Gank
```

<a name="enable"></a>
### Enable expansion
Enabling or disabling the extension is controlled by the configuration file, open `/config/admin-extension.php` and add the following code:
```php
return [
    'gank' => [
        'enable' => true,
    ],
];
```
<a name="view"></a>
### Visual management extensions
Visit `http://localhost:8000/admin/helpers/extensions` to enable or disable extensions, import extensions by page action.
>If `/config/admin-extension.php` does not exist, the system will automatically create it.


This completes the installation, open `http://localhost/admin/gank` to access the extension.

