# BETA version update log

## v2.1.4-beta

Release date 2021/9/16

To upgrade, execute the following commands step by step, and finally clear the **browser cache**
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin: "2.1.4-beta"
php artisan admin:update # will not overwrite the translation files menu.php and global.php
```

### BUG FIXES

1. fix the problem that the pop-up window of admin list is empty when you click to view permission
2. fix the problem that `non-equal` query for the fields in the data table is not effective
3. repair the problem that the original field value cannot be obtained in `displayer` after using `if` method for data table `column`.

## v2.1.3-beta

Release date 2021/9/14

To upgrade, run the following commands step by step, and finally clear the **browser cache**
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin: "2.1.3-beta"
php artisan admin:update # will not overwrite the translation files menu.php and global.php
```

### Functional improvements

**1. Refactored in-line editing of data tables**

The current version refactored the `UI` style of the in-line edit form for `editable`, `checkbox` and `radio` to show the form in a popup window. And a new in-line editing form `textarea` has been added, with the following effect.

! [](https://cdn.learnku.com/uploads/images/202109/14/38389/mX4Za4nj1y.png!large)
! [](https://cdn.learnku.com/uploads/images/202109/14/38389/9A2GdY3nSx.png!large)
! [](https://cdn.learnku.com/uploads/images/202109/14/38389/6Bo4phkB3f.png!large)
! [](https://cdn.learnku.com/uploads/images/202109/14/38389/wViO5EoPBg.png!large)


**2. Add `favicon` parameter to configuration file**

From the current version, you can configure `favicon` link in `config/admin.php` with the parameter `favicon`.

**3. Support displaying the `message` of exceptions thrown when submitting data forms**

As of the current version, if an exception is thrown when submitting a form, it will look like this
```php
$form->submitted(function ($form) {
    throw new \Exception('Access forbidden');
});
```

Then you will see the following message in the page

! [](https://cdn.learnku.com/uploads/images/202109/14/38389/S0KtwNRYGK.png!large)


**5. Optimize the use of `array` and `table` forms in tool forms**

In the old version of `Widgtet/Form`, if you use `array` and `table` forms, and if you use file upload form in `array` and `table` forms, you need to customize the file upload address in order to upload properly, so this version has optimized this function, and after the updated version, you don't need to customize the upload address anymore.
```php
$this->array('...' , function ($form) {
    // No need to customize the upload address
    $form->image('img');
});
```

**4. Add support for `addElementClass` method on data form `html`**

In the old version, the `class` set by `addElementClass` did not work for the `html` method, so this has been improved in the new version
```php
$form->html(...) ->addElementClass(['class1', ...]) ;
```

**5. Data form image upload form does not check if image exists when accessing thumbnails**

[@zhaiyuxin103](https://github.com/jqhph/dcat-admin/pull/1455)


### BUG FIXES

1. fix the problem that data forms cannot be merged correctly when using closure validation rules [#1429 @Edwin](https://github.com/jqhph/dcat-admin/pull/1429)
2. fix the problem that the file name generated after the `sequenceName` method is enabled for file upload has a duplicate suffix
3. repair the problem that files cannot be uploaded after setting `required` validation rules for one-to-one fields in file upload form
4. repair the problem of blank filtering sidebar under some operations [#1445 @Abbotton](https://github.com/jqhph/dcat-admin/pull/1445)
5. fix the problem of invalid second parameter specified in `model` method of `selectTable`, `multipleSelectTable` and other fields [#1460 @hhniao](https://github.com/jqhph/dcat-admin/pull/1460)
6. fix the problem of not being able to translate the text of `dimensions` verification failure prompt in the image upload form
7. fix the problem that the picture upload form cannot be submitted after using picture upload form in `array`, `table` and `hasMany` forms and setting `dimensions` validation rules
8. repair the problem that only the first picture can be previewed in multi-picture upload
9. repair the problem that the dynamic display of the form does not take effect if the field value has decimal point



## v2.1.2-beta

Release date 2021/7/12

To upgrade, step-by-step execute the following commands and clear the browser cache
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin: "2.1.2-beta"
php artisan admin:update # will not overwrite the translation files menu.php and global.php
```

### Feature improvements

**1. Add Grid\Model::apply() method**

This method can be used to apply quick search and filtering query criteria to a data table.

In the old version, it was very troublesome to use the quick search and filtering query criteria
```php
$grid->header(function ($collection) use ($grid) {
    $query = Model::query();

    // Get the table filter where condition array to iterate through
    $grid->model()->getQueries()->unique()->each(function ($value) use (&$query) {
        if (in_array($value['method'], ['paginate', 'get', 'orderBy', 'orderByDesc'], true)) {
            return;
        }

        $query = call_user_func_array([$query, $value['method']], $value['arguments'] ? []);
    });

    // Find out the statistics
    $data = $query->get();

    // Customize the component
    return new Card($data);
});
```

Starting with the current version you can use the `apply` method to simplify the above code
```php
$grid->header(function ($collection) use ($grid) {
    $query = Model::query();

    // Get the table filter where condition array to iterate through
    $grid->model()->apply($query);

    // Find out the statistics
    $data = $query->get();

    // Customize the component
    return new Card($data);
});
```



### BUG FIXES

1. fix the problem that data form can't disable bulk delete button
2. fix the problem that Gaode map form does not zoom when it has coordinates [#1377 @gzxy-0102](https://github.com/jqhph/dcat-admin/pull/1377)
3. repair the problem that the download button of file upload form is not displayed [#1405](https://github.com/jqhph/dcat-admin/issues/1405)
4. fix the problem of data filtering parameters not taking effect after using `simplePaginate` in data table [#1405](https://github.com/jqhph/dcat-admin/issues/1405)

## v2.1.1-beta

Release date 2021/7/12

To upgrade, execute the following commands step by step and clear your browser cache
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.1.1-beta"
php artisan admin:update # The translation files menu.php and global.php will not be overwritten
```

### New features

**1. Add model tree expand method to control whether to expand all child node data**

Expand all child node data by default

```php
// Expand the child node data
$tree->expand();

// Collapse all child node data
$tree->expand(false);
```

**2. Add file upload form download function**

```php
$form->file('...')->downloadable();
```


**3. Add the Gaudet map form**

Set in the configuration file `config/admin.php` [#1331 @gaizhixin](https://github.com/jqhph/dcat-admin/pull/1331)
```php
    'map' => [
        'provider' => 'amap',
        'keys' => [
            // Configure the key of Gaode Map
            'amap' => 'key',
        ],
    ],
```


**4. New addElementClass method for setting custom classes to form fields**

```php
// If you don't want to add a prefix, set the second parameter to false
$form->text(...)->addElementClass(['class1', 'class2'], false);
```

**5. Add table batch operation to set the drop-down menu split line function**

Support the following two ways

```php
// Way 1
$grid->batchActions(function ($batch) {
    $batch->add(...);
    
    // Show split line
    $batch->divider();
    
    ...
});

// Way 2
use Dcat\Admin\Grid\Tools\ActionDivider;

$grid->batchActions([
    new Action1(),
    ...
    new ActionDivider(),
    ...
]);
```

### Feature improvements


**1.table form support custom view**

```php
$this->table(...)->setView('...');
```

**2. Optimize the operation experience and UI after menu shrinkage**

When the menu shrinks, the cursor moves up and automatically expands if the menu is clicked, it will automatically shrink back after jumping; and fixes the problem of `mini-logo` being shown incorrectly.


**3. Data table row action column no longer shows empty dropdown menu when there is no action button**

Data table row action column no longer shows empty dropdown menu when there is no action button [#1327 @jiangyuntao](https://github.com/jqhph/dcat-admin/pull/1331)

**4. Optimize display of image upload form images**

[#1366 @ShermanTsang](https://github.com/jqhph/dcat-admin/pull/1366)

### BUG FIXES

1. repair the problem that `Grid::__toString()` will report an error if there is no data when the tree table (`tree`) expands its sub-nodes
2. repair the problem of invalid filtering conditions reduction after enabling asynchronous rendering function of data table
3. repair the problem of abnormal display of the number of filtering items in asynchronous rendering of the table
4. repair the problem that setting `class` of form fields will overwrite the default `class`.
5. repair the problem of displaying abnormal messages when visiting the page without authority after closing `debug` mode
6. repair the problem that `Grid::disableBatchDelete` fails after the configuration file custom batch delete button
7. repair the problem that tertiary menu cannot be hidden after menu indentation
8. repair the problem that the built-in permission system reports error when clicking `Add permission` when it is set to no route prefix
9. repair the problem of not disabling the authority middleware when the built-in authority system is disabled.
10. repair the problem that editing data cannot be displayed when the second parameter of `select` and `model` of `selectTable` is not `id`.
11. fix the problem that some forms setting size style does not take effect [#1361 @Abbotton](https://github.com/jqhph/dcat-admin/pull/1361)
12. fix the problem that the table sorting function is not compatible with `Grid\Model::latest` and `oldest` methods


## v2.1.0-beta

Release date 2021/5/23

To upgrade, step-by-step execute the following commands and clear the browser cache

```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin: "2.1.0-beta"
php artisan admin:update # will not overwrite the translation files menu.php and global.php
```

### New features

**1. Add table asynchronous rendering feature**

When the table on a page displays a particularly large amount of data (many columns and many rows) and loads more components, it may lag, so you can use the table asynchronous rendering feature to effectively reduce the page lag: the

```php
// Enable asynchronous rendering of tables
$grid->async();
```

Note that there is no need to enable this feature if the page is not visibly stuttering, and that it is not available if there are multiple data tables on the page! Refer to [datagrid - asynchronous rendering](model-grid-async.md) for detailed usage


### Feature improvements


**1. Support for `Laravel Octane 1.x` version**

This version adapts the changes related to `Laravel Octane 1.x` version, refer to [Laravel Octane](laravel-octane.md) for the specific usage.


**2. Call `expand(false)` to disable automatic pop-up filter sidebar**

```php
// disable automatic popup filter sidebar
$grid->filter(function ($filter) {
    $filter->expand(false);
    
    ...
});
```

**3. Upgrade `tinymce` to `v5.8.0` version**

[@yiming0](https://github.com/jqhph/dcat-admin/pull/1263)


**4. Automatically clear the menu cache after binding the menu to the permissions and roles pages**


### BUG FIXES

1. fix `withConstraints` method invalid for detail page `url` problem [#1232](https://github.com/jqhph/dcat-admin/issues/1232)
2. fix the problem that the form value is converted into an associated array when the multi-image/file upload form deletes pictures
3. fix the problem that Baidu map component cannot be used after `https` is enabled [#1162](https://github.com/jqhph/dcat-admin/issues/1162)

## v2.0.24-beta

Release date 2021/4/30

To upgrade, step-by-step execute the following command and clear the browser cache
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin: "2.0.24-beta"
php artisan admin:update # will not overwrite the translation files menu.php and global.php
```

### New features

**1. Add the ability to bind menus directly when creating or editing roles and permissions**

This feature is enabled by default and can be disabled by configuring the parameters `admin.menu.role_bind_menu` and `admin.menu.permission_bind_menu`, the effect is as follows

! [](https://cdn.learnku.com/uploads/images/202104/30/38389/OUgvZVSA5l.jpg!large)

**2. New `Form\Tree::treeStatus()` method to allow separate parent node selection**

Usage
```php
$form->tree('xxx')
    ->treeState(false) # allow separate parent selection
    ->setTitleColumn('title')
    ->nodes(...) ;
```

Effects
! [](https://cdn.learnku.com/uploads/images/202104/30/38389/oChwzky2BT.gif!large)

### BUG FIX

1. fix the problem of not being able to use domain name to distinguish applications in case of multiple applications
2. fix the problem that `Admin::pjax()` method is not declared as `static`.
3. repair the problem that only a single option takes effect after `grid filter checkbox` can only select multiple options [@outer199](https://github.com/jqhph/dcat-admin/pull/1174)
4. fix the problem that the default icon setting of menu is invalid
5. repair the problem of error when using `embeds` form in `php7.4` or above [#1204](https://github.com/jqhph/dcat-admin/issues/1204)
6. fix the problem of using `$form->list(...)' under step-by-step form ->limit(...) Refresh the page with error after the parameter check does not pass [#1206](https://github.com/jqhph/dcat-admin/issues/1206)
7. fix the problem of not being able to jump after editing and creating after disabling `pjax` [#1208](https://github.com/jqhph/dcat-admin/issues/1208)

## v2.0.23-beta

Released on 2021/4/18

To upgrade, execute the following commands step by step and clear the browser cache
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin: "2.0.23-beta"
php artisan admin:update
```

### New features

**1. Add support for `Laravel Octane`**

[Laravel Octane](https://github.com/laravel/octane) is a `Swoole/RoadRunner` driven project that can improve the performance of the `Laravel` framework, after installation it can significantly improve the performance of `Laravel` projects, ` Dcat Admin` is also compatible with the `Laravel Octane` environment, just add the following configuration to the configuration file `config/octane.php`.

```php

    'listeners' => [
        ... ,

        RequestReceived::class => [
            ... .Octane::prepareApplicationForNextOperation(),
            ... .Octane::prepareApplicationForNextRequest(),
            
            // Enable support for Dcat Admin
            Dcat\Admin\Octane\Listeners\FlushAdminState::class,
        class],
        
        ...
    ],    
```

> [Laravel Octane](https://github.com/laravel/octane) is still in `beta` phase, for installation and more information about [Laravel Octane](https://github.com/laravel/octane) Please go to the documentation at https://github.com/laravel/octane for more information.



**2. Add `simplePaginate` feature** to tables

Enabling `simplePaginate` function will use `Laravel`'s [simplePaginate](https://laravel.com/docs/8.x/pagination#simple-pagination) function for pagination, which can greatly improve the page response speed when the data volume is large. However, it is important to note that the **total rows** of the data table will not be queried after using this feature.

```php
// Enable
$grid->simplePaginate();

// Disable
$grid->simplePaginate(false);
```

### Functional improvements

**1. Refactor translation function**

In past versions, the field names could not be translated automatically when using the [asynchronous forms](lazy.md) and [asynchronous forms](lazy.md) functions, but from the current version onwards you can specify the path to the translation file for automatic translation using the ``translation`'' attribute, as follows

```php
<?php

namespace App\Admin\Forms;

use Dcat\Admin\Traits\LazyWidget;
use Dcat\Admin\Widgets\Form;
use Dcat\Admin\Contracts\LazyRenderable;

class MyForm extends Form implements LazyRenderable
{
    use LazyWidget;
    
    /**
     * Specify the translation file name
     * 
     * @var string 
     */
    protected $translation = 'my-form';
    
    public function form()
    {
        $this->text('field1');
        $this->text('field2');
        
        ...
    }
    
    ...
}
```

The language package `resources/lang/{locale}/my-form.php` has the following contents
```php
<?php

return [
    'fields' => [
        'field1' => 'field1',
        'field2' => 'field2',
        
        ...
    ],
];
```

And you can also specify the path to the current translation file in the controller

```php
use Dcat\Admin\Http\Controllers\AdminController;

class UserController extends AdminController
{
    /**
     * Specify the name of the translation file
     * 
     * @var string 
     */
    protected $translation = 'user1';
    
    ...
}
```

Of course it can also be specified via the `Admin::translation()` method

```php
use Dcat\Admin\Admin;

Admin::translation('user');
```


**2. Add form-related configuration parameters**

Add several form-related configuration file parameters `admin.grid.*`:

```php
    'grid' => [

        // table row action class
        'grid_action_class' => Dcat\Admin\Grid\Displayers\DropdownActions::class,

        // Grid batch action class
        'batch_action_class' => Dcat\Admin\Grid\Tools\BatchActions::class,

        // Table paginator class
        'paginator_class' => Dcat\Admin\Grid\Tools\Paginator::class,

        // The default configuration of several action classes for the grid row
        'actions' => [
            'view' => Dcat\Admin\Grid\Actions\Show::class,
            'edit' => Dcat\Admin\Grid\Actions\Edit::class,
            'quick_edit' => Dcat\Admin\Grid\Actions\QuickEdit::class,
            'delete' => Dcat\Admin\Grid\Actions\Delete::class,
            'batch_delete' => Dcat\Admin\Grid\Tools\BatchDelete::class,
        ],

        ...
    ],
```


**3. Remove display of table footer information after disabling table paging**

Starting from the current version, the table footer information will no longer be displayed if paging of the table is disabled.

**4. Optimize `Filter::panel()` layout spacing**


**5. Optimize the file publishing function, the `menu.php` and `global.php` files will no longer be overwritten when publishing language pack files**

Starting from the current version, the `menu.php` and `global.php` files are no longer overwritten when using the `admin:update` and `admin:publish --force` command files.

**6. Update `Tinymce` version to `5.6.2` **

[@wk1025](https://github.com/jqhph/dcat-admin/pull/1154)

### BUG FIXES

1. fix the problem of error when field type is `Object` in data table [@xiaohuilam](https://github.com/jqhph/dcat-admin/pull/1154)
2. fix the problem that file upload fails and still prompts successful upload
3. repair the problem of not using the incoming `$request` object when matching the middleware of permission judgment [@asmodai1985](https://github.com/jqhph/dcat-admin/pull/1157)
4. fix the problem of invalid setting `message` for the second parameter of `rules` method of the form
5. fix the problem that `Helper::array()` converts `0` to an empty array
6. fix the problem that the row selector cannot select sub-level rows of tree table
7. Fix the problem that `KeyValue::setValueLabe
8. fix the problem that `hasmany` and `table` form resets the form when deleting options
9. fix the problem of displaying wrong text of edit button for table row operation [@GemaDynamic](https://github.com/jqhph/dcat-admin/pull/1174)
10. fix the problem that sub-level rows of tree table can't use form pop-up window properly [#813](https://github.com/jqhph/dcat-admin/issues/1174)

## v2.0.22-beta

Release date 2021/4/1

To upgrade, step-by-step execute the following command and clear the browser cache
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin: "2.0.22-beta"
php artisan admin:update
```

### BUG FIXES

1. repair some functions missing `CSRF_TOKEN` error problem
2. fix the error reported by menu auto-adaptation height function


## v2.0.21-beta

Release date 2021/3/30

To upgrade, step-by-step execute the following commands and clear the browser cache
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin: "2.0.21-beta"
php artisan admin:update
```

### New features

**1. Add table column selector `ColumnSelector` to support persistent storage**

In the configuration file `config/admin.php` you can configure the way to store the state of the column selector, the supported storage methods are as follows

- `Dcat\Admin\Grid\ColumnSelector\SessionStore` ColumnSelector state data is stored in `session`, only valid in the login state
- `Dcat\Admin\Grid\ColumnSelector\CacheStore` Column selector state data is stored in [Laravel Cache](https://laravel.com/docs/8.x/cache#driver-prerequisites) cache system for up to `300` days and can be configured with `admin.grid.column_selector.store_params.driver` which defaults to `file`

```php
    'grid' => [

        ...

        'column_selector' => [
            'store' => Dcat\Admin\Grid\ColumnSelector\SessionStore::class,
            'store_params' => [
                'driver' => 'file',
            ],
        ],
    ],
```

**2. Add an end method (`end`) to the form conditional judgment (`if`) **

``` php
$grid->column('status')
    ->if(...) // Condition 1
    ->display(...)
    ->display(...)
    
    ->if(...) // condition 2
    ->display(...)
    ->display(...)
    
    ->end() // end the previous condition
    ->display(...) ; // All conditions are valid
```

**3. Add the default check (`check`) and disable (`disable`) functions of the table row selector**


The `check` method can be used to set the default selected rows, this method accepts an `array` type or an `anonymous function` parameter

``` php
// Set the default check for rows 1/3/5
$grid->rowSelector()->check([0, 2, 4]);

// Pass the closure
$grid->rowSelector()->check(function () {
    // set the default to check rows 1/3/5
    return in_array($this->_index, [0, 2, 4]);
});

// Use other fields of the current row in the closure
$grid->rowSelector()->check(function () {
    // set the default to check rows with id > 10
    return $this->id > 10;
});
```

The ``disable`` method allows you to set the rows that are not allowed to change their check status, this method accepts an ``array`` type or an ``anonymous function`` parameter

``` php
// Disable row 1/3/5 from being selected
$grid->rowSelector()->disable([0, 2, 4]);

// Pass the closure
$grid->rowSelector()->disable(function () {
    // disable row 1/3/5 from changing selection status
    return in_array($this->_index, [0, 2, 4]);
});

// Use other fields of the current row in the closure
$grid->rowSelector()->disable(function () {
    // disable the selected state for rows with id > 10
    return $this->id > 10;
});

// disable can be used in conjunction with the check method
$grid->rowSelector()->check([2, 4])->disable([0, 2, 4]);
```

**4. Add `KeyValue` form custom title translation function**

``` php
$form->keyValue(...) ->setKeyLabel('keyName')->setValueLabel('keyValue');
```


**5. Add `Grid::scrollbarX` method to show horizontal scrollbar of table**

Show the horizontal scrollbar of the grid, not by default

``` php
// Enable
$grid->scrollbarX();

// Disable
$grid->scrollbarX(false);
```


**6. Add `admin:update` command**
Starting from the current version, you can run `admin:update` directly after upgrading, which is equivalent to running

```
php artisan admin:publish --assets --migrations --lang --force
php artisan migrate
``` 

### Feature improvements

**1. Form display is compatible with hump style association names when editing data**

In the old version, if you need to edit a model association field and the model association name is hump style, you need to change the name to underscore style to display it properly, which is very unfriendly to developers. Starting from the current version, you can use the hump style association names directly without any special handling
```php
return Form::make(User::with(['myProfile']), function (Form $form) {
    // Directly use camel style naming, no need to do anything else
    $form->text('myProfile.full_name');
    
    ...
});
```


**2. Adjust the tool form data setting logic to get `default` method data in the `form` method**

```php
use Dcat\Admin\Widgets\Form;

class Setting extends Form
{
    public function form()
    {
        // 获取 default 方法设置的数据
        $id = $this->data()->id;
        $name = $this->data()->name;
    
        $this->text('name');
        
        ...
    }
    
    public function default()
    {
        return [
            'id' => 1,
            'name' => 'abc',
            ...
        ];
    }
}
```

**3.`hasMany` and `array` forms support overall validation using `rules` validation rules**

``` php
$form->hasMany(...) ->rules('size:2');
```

**4. Adjust the default placeholder for `selectTable` to `select ... `**

**5. Optimize form row layout spacing**

[#1092](https://github.com/jqhph/dcat-admin/issues/1092)


### BUG

1. fix the problem that `ModelTree` can't delete the child nodes with more than one level of interval when deleting nodes
2. repair the problem that the export function may be abnormal in some environments
3. repair the problem that the initial value of form linkage (`load`) is lost after loading [@xqbumu](https://github.com/jqhph/dcat-admin/pull/1103)
4. repair the problem of abnormal display of prompt information after file upload failure
5. repair the problem of incorrect primary key assignment if the data details `Show` instantiation function passes the model [@jisuye](https://github.com/jqhph/dcat-admin/pull/1112)
6. fix the problem that the table `setConstraints` method is invalid for quick editing [#1119](https://github.com/jqhph/dcat-admin/issues/1119)
7. fix the problem of abnormal preview image function in `iframe` page
8. fix the problem that the form cannot be submitted after the fields with `required` validation rules are deleted in `hasMany` and `array` forms
9. fix the abnormal loading problem of table `dialogTree` when the top ID is string `0` [#1122](https://github.com/jqhph/dcat-admin/issues/1122)


## v2.0.20-beta

Release date 2021/3/8

To upgrade, step-by-step execute the following commands and clear the browser cache
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin: "2.0.20-beta"
php artisan admin:publish --assets --migrations --force
php artisan migrate
```

### Feature improvements

**1. `selectTable` form retains the data state of the previous page when the form is refreshed**

`selectTable` form will retain the selected or unselected data state of the previous page after refreshing or paging

**2. `selectTable` form set display field name optimization**

In the old version, if you want to set the selected or displayed fields of `selectTable` form, you need to set them in the `Renderable` object, which is very troublesome and inconvenient, from the current version, developers can set the selected or displayed fields directly by the following method

```php
$form->selectTabole('user_id')
	->from(UserTable::make())
	->pluck('full_name', 'id'); // the first parameter is the field to be displayed, the second parameter is the field that will be saved to the form when selected

// You can also use the following method directly
$form->selectTabole('user_id')
	->from(UserTable::make())
	->model(UserModel::class, 'id', 'full_name');
```


**3.`selectTable`, `multiSelectTable`, `radio`, `checkbox` forms add `load` method**

Starting from the current version, `selectTable`, `multipleSelectTable`, `radio`, `checkbox` can also link `select` and `multipleSelect` forms using the `load` method

```php
$form->radio('type')->options([...]) ->load('category', 'categories/options');

$form->select('categories');
```

The interface `categories/options` returns the following format

```json
[
    {
        "id": 9,
        "text": "xxx"
    },
    {
        "id": 21,
        "text": "xxx"
    },
    ...
]
```


**4. Add menu horizontal layout automatically adapt to the page and the menu height change function**

Enable the menu horizontal layout function, when the page height or menu height changes, the page will be self-adapting, self-adjusting content spacing

****


**5. Add table `Grid::dropColumn()` method to delete set columns**

```php
$nameColumn = $grid->column('name');

// Delete the column with the name `name`
$grid->dropColumn('name');
// Equivalent to
$grid->dropColumn($nameColumn);
```

**6. Add the `admin_javascript` function**

This function can be used to add `JS` code to the configuration `array` of `php`, with the following usage

```php
$form->text('number')->inputmask([
	'oncomplete' => admin_javascript('function () {
		// Here is the js code
	    alert('inputmask complete');
	}'),
]);
```

**6.`Form` form bottom can be checked by default `View`, `Continue editing`, `Continue creating` and other option functions**

Use the following [#1073](https://github.com/jqhph/dcat-admin/pull/1073)

```php
$form->footer(function (Footer $footer) {
    // set ``view`` to be selected by default
    $footer->defaultViewChecked();

    // set `Continue editing` to be checked by default
    $footer->defaultEditingChecked();
    
    // set `Continue creating` to be checked by default
    $footer->defaultCreatingChecked();
});

// set `Viewing` to be checked by default
$form->defaultViewChecked();

// set `Continue editing` to be checked by default
$form->defaultEditingChecked();

// set `Continue creating` to be checked by default
$form->defaultCreatingChecked();
```


### BUG

1. fix the problem that the form must be sorted by `with` first when using the correlation field
2. repair the problem that multiple asynchronous components cannot be rendered at the same time on the same page
3. repair the problem of abnormal jumping after deleting child node data under tree table [#1071](https://github.com/jqhph/dcat-admin/issues/1071)
4. repair the abnormal export problem when there is an empty array in the export field of the table
5. repair the problem that the picture element of the page disappears when using `sortable` function to sort the form multi-picture upload
6. repair the problem of invalid setting `false` for `disable` method of form fields
7. repair the problem that `multipleSelect` form cannot pass all selected options into the interface when using `load` linkage loading [#1076](https://github.com/jqhph/dcat-admin/issues/1076)
8. fix the problem of abnormal selection function when there are multiple options starting with 0 in table specification selector


## v2.0.19-beta

Release date 2021/2/21

To upgrade, step-by-step execute the following command and clear the browser cache
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.19-beta"
php artisan admin:publish --assets --migrations --force
php artisan migrate
```

### BUG FIXES

1. fix ``no access`` problem when requesting ``built-in api`` in non-admin role
2. fix the problem of using time range form in popup window to report error




## v2.0.18-beta

Released on 2021/2/20

To upgrade, run the following commands step by step and clear the browser cache
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.18-beta"
php artisan admin:publish --assets --migrations --force
php artisan migrate
```

### Feature improvements


**1. Add horizontal layout at the top of the menu (Horizontal)**

Set the value of the configuration parameter `admin.layout.horizontal_menu` to `true` to enable this feature, the effect is as follows

![](https://cdn.learnku.com/uploads/images/202102/20/38389/SpmXMujJ3D.png!large)

**Permission middleware and skip login judgment can fill in the route alias and do not need to increase the prefix **

Configuration file and permission setting route alias without filling in the route prefix

```php
    'permission' => [
        ...
    
        // Skip the permissions
        'except' => [
            // You can fill in the route alias directly, and you don't need to write the route prefix
            'custom.users',
        ],

    ],
```

**3. Add `_index` field to save row number for data table row data**

The `_index` field is used to save the row number, starting from `0`, and is used as follows

``` php
// Use in the display callback
$grid->column('serial number')->display(function () {
    return $this->_index + 1;
});


// used in a row action
$grid->actions(function ($actions) {
    $index = $this->_index;
    
    ...
});
```


**4. Rename markdown component static resource aliases to avoid conflicts with custom blade tags**

**5. Add configuration parameter `admin.menu.default_icon` to set the default menu icon**

`admin.menu.default_icon` is used to set the default menu icon, the default value is `feather icon-circle`

**6. Add new block locations `NAVBAR_BEFORE` and `NAVBAR_AFTER`**

```php
use Dcat\Admin\Admin;

// Output content to the front of the top navigation bar
admin_inject_section(Admin::SECTION['NAVBAR_BEFORE'], view('...'));

// Output content after the top navigation bar
admin_inject_section(Admin::SECTION['NAVBAR_AFTER'], view('...'));
```

**6. Optimize form field selector code**


### BUG FIXES

1. fix the problem of abnormal display of `new` tag on the extension management page [#1044](https://github.com/jqhph/dcat-admin/issues/1044)
2. fix the problem of error reported after successful file uploading and direct deletion [#1058](https://github.com/jqhph/dcat-admin/issues/1058)
3. repair the abnormal input value problem of `Form::number` form after using `min` and `max` methods



## v2.0.17-beta

Release date 2021/2/5

To upgrade, step-by-step execute the following commands and clear the browser cache
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.17-beta"
php artisan admin:publish --assets --migrations --force
php artisan migrate
```

### Function improvement

**1. Optimize the table sorting function**

Support `orderBy` to sort directly using related table fields, note that only `one to one` and `one to many` related relationships are supported here

```php
$grid->model()->orderBy('profile.age');
```

**2. Add the function of customizing `parent_id` value in the model tree and tree table**

Model tree and tree table can be customized in `model` with `parent_id` value, the default value is `0`.
```php
class Category extends Model
{
	use ModelTree;

    // Set the default parent_id to A
	protected $defaultParentId = 'A';
}
```

**3. Data details `file` supports displaying multiple files**

[#985](https://github.com/jqhph/dcat-admin/pull/985)

```php
$show->field('...')->files();
```

Result

![](https://cdn.learnku.com/uploads/images/202102/02/38389/B0a2qZEBUL.png!large)


**4.`Form::input` supports array batch settings**

```php
$form->submitted(function ($form) {
    $form->input(['k1' => 'v1', 'k2' => 'v2' ...]);
});
```

**5. Extension management supports `logo` and `alternate display`**

Refer to the documentation [extensions](extension-dev.md#logo) for detailed usage


**6. Add admin_route method to get URL by alias**

The `app/Admin/routes.php` route is registered as follows
```php
Route::group([
    'prefix'        => config('admin.route.prefix'),
    'namespace'     => config('admin.route.namespace'),
    'middleware'    => config('admin.route.middleware'),
], function (Router $router) {
	// Set alias
	$router->resource('users', 'UserController', [
		'names' => ['index' => 'my-users'],
	]);

});
```

Get URL by alias

```php
// Get the url
$url = admin_route('my-users');

// Determine the route
$isUsers = request()->routeIs(admin_route_name('users'));
```

**JsonResponse::location allows not to pass parameters**

The current page will be automatically refreshed after ``1`` seconds if no parameters are passed

```php
return Admin::json()->success('operation successful')->location();
```

**9.Page LayoutLayout\Column support equal width layout**

Equal-width layout is used when the column width is set to `0` [#1018](https://github.com/jqhph/dcat-admin/pull/1018)

```php
use Dcat\Admin\Layout\Row;
use Dcat\Admin\Layout\Content;

return Content::make()
	->body(function (Row $row) {
	    $row->column(0, view('...'));
	});
```

**10. The page layout Layout\Row supports no-gutters property**

`.row` with `margin-left: -15px;margin-right: -15px;` attribute, you can eliminate this attribute by defining the `.no-gutters` attribute on `.row` so that the page is not `30px` extra wide, i.e. `<div class="row no- gutters"... `
```php
$content->row(function (Row $row) {
	// Enable no-gutters
	$row->noGutters();

	$row->column(9, function (Column $column) {
		$column->row($this->card(['col-md-12', 20], '#4DB6AC'));
		
		$column->row(function (Row $row) {
			// Enable no-gutters
			$row->noGutters();

			$row->column(4, $this->card(['col-md-4', 30], '#80CBC4'));
			$row->column(4, $this->card(['col-md-4', 30], '#4DB6AC'));
			$row->column(4, function (Column $column) {
				$column->row(function (Row $row) {
					// Enable no-gutters
					$row->noGutters();

					$row->column(6, $this->card(['col-md-6', 30], '#26A69A'));
					$row->column(6, $this->card(['col-md-6', 30], '#26A69A'));
				});
			});
		});
	});
});
```

The effect is as follows

![](https://cdn.learnku.com/uploads/images/202102/05/38389/4YlO8aOPCW.jpg!large)

**11. Retain the get parameter of the URL after deleting data from the form**

In the previous version, the `get` parameter of the `URL` was lost after deleting data, resulting in jumping back to the first page of the form. This version has optimized this feature, and the `get` parameter of the `URL` is still retained after deletion [#961](https://github.com/jqhph/dcat-admin/issues/ 961)


**12. Refactor file upload front-end code**

This feature is a technical optimization, this version refactored the file upload front-end code, split the code to make it easier to read and maintain

### BUG FIXES

1. fix `MultipleSelect` form style exception problem [#967](https://github.com/jqhph/dcat-admin/issues/967)
2. fix the abnormal problem of using `select2` form after loading `markdown` component [#990](https://github.com/jqhph/dcat-admin/issues/990)
3. fix the problem of losing traditional Chinese file name saved under `Linux` server when uploading files [#993](https://github.com/jqhph/dcat-admin/issues/993)
4. fix the problem that `Widgets\Dropdown::click` cannot display default options
5. fix the problem of `NaN` when clicking the plus/minus button when the text content of `number` component in `form` form is empty [#995](https://github.com/jqhph/dcat-admin/issues/995)
6. repair the problem that the picture preview fails to indicate that the translation file cannot be used
7. repair the problem of abnormal judgment of `Range` form of one-to-one association relationship using validation rules
8. repair the problem of route alias conflict under multiple applications



## v2.0.16-beta

Release date 2021/1/11

To upgrade, step-by-step execute the following command
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.16-beta"
php artisan admin:publish --assets --migrations --force
php artisan migrate
```


### Breaking changes

**1. Adjusted `disableHorizontal` to `horizontal` for form fields **

Change the form field layout to `horizontal`, this is enabled by default and is used as follows

```php
// Disable the horizontal layout
$form->text('...')->horizontal(false);
```

### Feature improvements

**1. Enhance the sortable function of form fields**

Make form fields support sortable for relational table fields and `json` fields

> Note that associative relationships only support `hasOne` and `belongsTo` types of field sorting, and do not support multiple levels of nesting!

```php
// Sorting fields in the associated table
$grid->column('profile.age')->sortable();

// Specify the name of the field to be sorted
$grid->column('my_age')->sortable('profile.age');

// json field sorting
$grid->column('options.price')->sortable('options->price');
// Sort the json fields of the association table
$grid->column('profile.options.price')->sortable('profile.options->price');
```

Support `MySql` ```order by cast(`{field}` as {type})``` usage

```php
$grid->column('profile.age')->sortable(null, 'SIGNED');

$grid->column('profile.options.price')->sortable('profile.options->price', 'SIGNED');
```


**2. Add admin_exist function to replace exit**

`admin_exist` is used to interrupt the execution of the program and respond to the browser with data for display, instead of `exit` and `die`, the following is a brief description of usage


Usage 1, return `Content` layout object, this usage can be used to return error messages to the front end
```php
use Dcat\Admin\Widgets\Alert;
use Dcat\Admin\Layout\Content;

// Interrupt the program and display a custom page to the front end
admin_exit(
    Content::make()
        ->title('title')
        ->description('description')
        ->body('Page content 1')
        ->body(Alert::make('Server error~', 'Error')->danger())
);
```

The effect is as follows

![](https://cdn.learnku.com/uploads/images/202101/11/38389/FLg6C7kwRq.png!large)

Usage 2, returning `json` formatted data, often used for `api` request interception for form submissions, or `api` request interception for `Action`

```php
use Dcat\Admin\Admin;

admin_exit(
    Admin::json()
        ->success('succeeded')
        ->refresh()
        ->data([
            ...
        ])
);

// Of course you can also respond directly to the array
admin_exit([
   ...
]);
```

Usage 3, direct corresponding `Response` object or string

```php
admin_exit('Hello world');

admin_exit(response('Hello world', 500));
```


**3. Add Show\Field::bool() and Show\Field::bold() methods**

Show `✓` if the field value is true, otherwise show `✗` [#940](https://github.com/jqhph/dcat-admin/pull/940)

```php
$show->field('...')->bool();
```

Bolded field values

```php
$show->field('...') ->bold();

// Specify the color
$show->field('...') ->bold(admin_color()->primary());
```

**4. Add Form\Footer::view() method**

The ``Form\Footer::view()` method allows you to customize the bottom view of a data form [#957](https://github.com/jqhph/dcat-admin/pull/957)

```php
$form->footer(function ($footer) {
    $footer->view('...' , [...]) ;
});
```


**5. Add form default to show specified Tab function**

```php
// Show the Tab with the title Title 2 by default
$form->getTab()->active('Title2');
// You can also specify an offset
$form->getTab()->activeByIndex(1);

$form->tab('Heading 1', function ($form) {
    ...
});

$form->tab('Title2', function ($form) {
    ...
});
```

**6. Add form Form\Row::horizontal() method**

Set the layout to ``horizontal``

```php
$form->row(function (Form\Row $form) {
	$form->horizontal();

	...
});
```

**7. Form Modal adds custom icon functionality**


```php
$grid->column('...') ->modal(function ($modal) {
    // Custom icons
    $modal->icon('feather icon-x');
    
    return ... ;
});
```


**8. Add route domain restriction configuration**

You can restrict the domain name of a route by configuring the parameter `admin.route.domain`, opening the configuration file `config/admin.php`

```php
    'route' => [
        'domain' => env('ADMIN_ROUTE_DOMAIN'),

        'prefix' => env('ADMIN_ROUTE_PREFIX', 'admin'),

        'namespace' => 'App\\Admin\\Controllers',

        'middleware' => ['web', 'admin'],
    ],
```


**9. Add enable or disable configuration for admin.session middleware**

After `2.0` version `admin.session` middleware is no longer enabled by default, if your application has both frontend and backend, you need to enable `admin.session` middleware, otherwise it will cause front and backend `session` conflict problem.

Set the value of the configuration parameter `admin.route.enable_session_middleware` to `true` to enable it
```php
    'route' => [
        'domain' => env('ADMIN_ROUTE_DOMAIN'),

        'prefix' => env('ADMIN_ROUTE_PREFIX', 'admin'),

        'namespace' => 'App\Admin\\Controllers',

        'middleware' => ['web', 'admin'],
        
        // enable admin.session middleware
        'enable_session_middleware' => true,
    ],
```

### BUG FIXES

1. fix the problem that `Model` is converted to `array` format in the first parameter of `Grid::header` and `Grid::footer` callback of data table
2. repair the problem that the color of file upload button cannot be changed when switching themes [#938](https://github.com/jqhph/dcat-admin/issues/938)
3. repair the invalid setting of the third parameter of `Widgets\Table` construction method
4. fix the problem of using `config(['admin.layout.color' => '...']) in `app/Admin/bootstrap.php` ' Override theme color may be invalid
5. fix the problem of invalid data table filter reset association form fields [#949](https://github.com/jqhph/dcat-admin/issues/949)
6. repair the problem of abnormal display of `group` function of table filter [#929](https://github.com/jqhph/dcat-admin/issues/929)
7. fix the problem that only the first set `title` is displayed in all popups when there are multiple `selectTable` forms on the page [#926](https://github.com/jqhph/dcat-admin/issues/926)

## v2.0.15-beta

Release date 2021/1/3

To upgrade method, step by step execute the following command
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.15-beta"
php artisan admin:publish --assets --migrations --force
php artisan migrate
```

### Feature improvements

**1. Upgrade select2 to v4.1.x-beta version**


Upgrade `select` component to `v4.1.x-beta`, make `tags` form experience better, and support multi-language translation.

**Widgets/Modal added vertical centering and scrollable popups**.

Use as follows [#901](https://github.com/jqhph/dcat-admin/pull/901)

```php
$modal = Modal::make()
    ->xl()
    ->centered() // Set the pop-up window to be vertically centered
    ->scrollable() // Set the content of the pop-up window to be scrollable
    ->title(...)
    ->body(...);
```

**3. `Admin::requiredAssets` supports passing dynamic parameters**

```php
use Dcat\Admin\Admin;

// Register front-end component aliases
// {lang} is a dynamic parameter
Admin::asset()->alias('@test', [
    'js' => ['/vendor/test/js/{lang}.min.js'],
]);

// {lang} will be replaced with zh_CN
Admin::requireAssets('@test', ['lang' => 'zh_CN']);
// It can also be used like this
Admin::requireAssets('@test?lang=zh_CN');
```


### Bug修复

1. Fix the problem that the form `block` layout cannot save data [#883](https://github.com/jqhph/dcat-admin/issues/883)
2. Fix `currency` failure under `hasMany` form [#886](https://github.com/jqhph/dcat-admin/issues/886)
3. Repair the problem of automatically jumping to the detail page after saving the data form [#893](https://github.com/jqhph/dcat-admin/issues/893)
4. Fix the problem that `editor` form can not clear the data [#895](https://github.com/jqhph/dcat-admin/issues/895)
5. Fix the `required` validation exception of `tags` under `hasMany` form [#905](https://github.com/jqhph/dcat-admin/issues/905)
6. Repair the problem of all files being emptied when deleting a single file in multi-file upload form [#914](https://github.com/jqhph/dcat-admin/issues/914)
7. fix the problem that form fields cannot use model accessor



## v2.0.14-beta

Release Date 2020/12/24

To upgrade the method, step by step execute the following command
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.14-beta"
php artisan admin:publish --assets --migrations --force
php artisan migrate
```

### Function improvement

**1. Optimize the error message prompt for file upload failure**

In the old version, the error message for file upload failure was not very clear, which made it difficult to define the reason for the error, so in this version, the error message is optimized, and the specific reason will be displayed once the file upload fails.

### Bug fixes

1. fix the problem that the form field conflicts with the model `casts` attribute and the abnormal display problem using string splicing in the `display` closure [#876](https://github.com/jqhph/dcat-admin/issues/876)
2. fix the problem that the dynamic display function of the form cannot be used [#879](https://github.com/jqhph/dcat-admin/issues/879)
3. fix the problem of not being able to display editing data when using `Block` layout [#877](https://github.com/jqhph/dcat-admin/issues/877)



## v2.0.13-beta

Release Date 2020/12/23

To upgrade the method, step by step execute the following command
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.13-beta"
php artisan admin:publish --assets --migrations --force
php artisan migrate
```

### Bug fixes

1. fix the problem that the table may report an error when displaying related fields when the related data does not exist [#867](https://github.com/jqhph/dcat-admin/issues/867)
2. fix the problem that `display` method is invalid when the table uses data repository to return arrays or non-model `collection` [#869](https://github.com/jqhph/dcat-admin/issues/869)


## v2.0.12-beta

Release Date 2020/12/22

To upgrade the method, step by step execute the following command
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.12-beta"
php artisan admin:publish --assets --migrations --force
php artisan migrate
```

### Disruptive changes

**1. image/file upload form `removeable` renamed to `removable`**

```php
$form->file('...')->removable();
```

### Feature improvement

**1.Support PHP8.0**

**2. Image/file upload form supports listening to WebUploader events**

The `on` method can listen to [WebUploader file upload related events](http://fex.baidu.com/webuploader/doc/index.html#WebUploader_Uploader_events)

```php
$form->file('...')
	->on('startUpload', <<<JS
		function () {
			console.log('File upload started...', this);
			
			// Attach custom parameters to the file upload interface before uploading files
			this.uploader.options.formData['custom_field'] = '...';
		}
JS
    )
	->on('uploadFinished', <<<JS
    	function () {
    		console.log('File upload is complete');
    	}
JS
    );
    
//       
```

**3. Listen for changes when a file is successfully uploaded or deleted**

You can listen for changes when a file **is uploaded successfully** or when a file **is deleted** by the following methods

```php
$file = $form->file('...');

Admin::script(
	<<<JS
$('{$file->getElementClassSelector()} .file-input').on('change', function () {
	console.log('Document changed', this.value);
});
JS
);
```

**4. Allow intercepting and responding to error messages in the uploading event**

Starting from this version, we support intercepting file uploads in the `uploading` event of the form and responding to error messages on the front-end.

```php
$form->uploading(function (Form $form) {
	return $form->response()->error('File upload failed, please try again!');
});
```



**5. Listen to selectTable selected value change**

```php
$selectTable = $form->selectTable('...')->from(...);

Admin::script(
	<<<JS
$('{$selectTable->getElementClassSelector()} input[type="hidden"]').on('change', function () {
	console.log('Change of selected values', this.value);
});
JS
);
```

**6. adjust the tree form no data return, cancel the return of 404 status code **

**7. Adjust the value type of row attribute of table displayer class to model**

**8. Dark mode details optimization**

```php
$grid->column(...)->modal(function () {
    // $this points to the model object
    dd($this);
});

$grid->actions(function () {
    // $this points to the model object
    dd($this);
});
```


**9. Optimize the problem of overflowing chart display in cards**

[#822](https://github.com/jqhph/dcat-admin/pull/822)


**10.widget component add when method**

```php
$modal = Dcat\Admin\Widgets\Modal::make()
	->when($condition, function ($modal) {
		// When the value of $condition is true, the logic inside the closure will be executed
	    $modal->xl();
	})
	->body(...)
	->render();
```



### Bug fixes

1. fix `Grid\Filter::group` can't keep selected state [#739](https://github.com/jqhph/dcat-admin/pull/793)
2. fix `Form::hasMany` problem of validating `required` even after form entry is deleted [#795](https://github.com/jqhph/dcat-admin/pull/795)
3. fix the problem that map form cannot be used
4. fix the problem that `guessClassFileName` will report error when `composer` class mapping file is generated and the class file is deleted
5. fix the problem of error when using `Fetched` event for data export [#815](https://github.com/jqhph/dcat-admin/issues/815)
6. fix the problem that `filter` cannot be reset after setting `Grid name`.
7. fix the problem that `select2` can't use Chinese language pack automatically [#839](https://github.com/jqhph/dcat-admin/issues/839)
8. fix the problem that `continue to create` and `continue to edit` jump route error [#814](https://github.com/jqhph/dcat-admin/issues/814)
9. fix `range` form setting `rules` invalid when one-to-one association is enabled
10. fix the problem that time filtering dropdown will be blocked when `fixColumns` is enabled [#833](https://github.com/jqhph/dcat-admin/issues/833)
11. fix the problem that menu `fa` icon can't be aligned automatically [#758](https://github.com/jqhph/dcat-admin/pull/758)
12. fix the problem of submitting errors with `hasMany` in `row` layout [#801](https://github.com/jqhph/dcat-admin/issues/801)
13. fix form `hasMany` can't use `select` linkage [#769](https://github.com/jqhph/dcat-admin/issues/769)



## v2.0.11-beta

Release date 2020/12/06

To upgrade, step-by-step execute the following command
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin: "2.0.11-beta"
php artisan admin:publish --assets --force
php artisan admin:publish --migrations --force # table structure changes
php artisan migrate
```


### Bug fixes

1. fix `Dcat.init` listener failure due to repeated page refreshes using `pjax`
2. fix `admin:export-seed --users` which generates redundant code
3. repair the problem of abnormal jumping after saving the form editing page
4. repair the problem that `hasMany` will add data repeatedly if you choose to continue editing on the form page
5. fix the problem that the original `select2 config` is lost after linking `select` forms [#779](https://github.com/jqhph/dcat-admin/issues/779)
6. fix `map` form loading exception problem [#764](https://github.com/jqhph/dcat-admin/issues/764)
7. fix the problem that the page cannot be refreshed automatically after the data is deleted by the batch delete function
8. fix the problem that `hasMany` form editing page cannot display `row` and `column` layouts properly
9. fix the problem that `Dcat.init` listener will be unbound by asynchronous pop-up window
10. fix a bug that the table toolbar dropdown menu is blocked by the fixed list pane [#728](https://github.com/jqhph/dcat-admin/issues/728)
11. fix a problem where cached content is still read when `showColumnSelector` is disabled

### Functional improvements

**1. Add title parameter to Form::divider**.

Add `title` parameter to display the title in the middle of the divider, usage

``` php
$form->divider('title');
```


**2. Grid::footer and Grid::header adjusted to support multiple callbacks**

``` php
$grid->header(...) ;

$grid->header(...) ;

$grid->header(...) ;
```

**3. Optimize form specification filters and select form styles**


## v2.0.10-beta


Posted on 2020/11/29

To upgrade the method, execute the following step-by-step commands
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.10-beta"
php artisan admin:publish --assets --force
php artisan admin:publish --migrations --force # Table structure changed.
php artisan migrate
```
### Functional improvements

**1. Add a prompt window in the upper right corner of the form to display field validation error messages**.

This feature is enabled by default and can be disabled by the `validationErrorToastr` method.

```php
$form->validationErrorToastr(false);
```

**2.Add Tree::maxDepth method to limit the maximum level of the model tree **.

```php
$tree->maxDepth(5);
```

**3. Optimize the export function, support title settings associated with the relational fields and automatically read the title of the grid column **.

In the current version, exported columns are the same as `column` columns by default, so you no longer need to set the exported column name and translation manually, and the associated relational fields are supported.

```php
$grid->column('id');
$grid->column('name');
...

// Default is the same as the column above
$grid->export();
```

**4. Add a resetButton and submitButton method to the tool form**.

```php
// Disable the Reset and Submit buttons
$form->resetButton(false);
$form->submitButton(false);
```

**5. Add parameters to the `disable` and `readOnly` methods of the form fields to control whether they are enabled or not**.

```php
// pass false to disable
$form->text(...)->disable(false);
```

**6. Adding `withDeleteData` to file uploads allows users to set request parameters and add primary key fields to the upload and delete interfaces**.

The `withDeleteData` method allows you to pass custom parameters to the file delete interface.

```php
$form->file(...)->withDeleteData(['key' => 'value]);
```

**7. Add `embeds` form to disable display of titles**.

The second parameter, passed as `false`, does not display the title.

```php
$form->embeds('field', false, function ($form) {
    ...
});
```

**8. Rewrite some of the unit test cases to support 2.x usage**.

### Bug fixes

1. fix the problem that existing permissions cannot be selected on admin detail page.
2. fix the problem of `admin:export-seed` command exporting `seeder` class name exception.
3. fix the form deletion jump exception
4. fix the problem of form jumping exception when continuing to edit.
5. fix the problem that parent table fields cannot be saved when parent table and `hasMany` have the same field name.
6. fix the problem that the style of selected submenu is abnormal in dark mode [#712](https://github.com/jqhph/dcat-admin/issues/712)
7. fix the problem that form dynamic display function is invalid under form `block` layout [#723](https://github.com/jqhph/dcat-admin/issues/723)
8. optimize the display of `selectOptions` hierarchy and solve the problem of prefix rendering increasing with the hierarchy depth index [#618](https://github.com/jqhph/dcat-admin/issues/618)
9. fix the problem that `admin_view` does not return data.
10. fix the problem that links set by `select` form, `ajax` and `load` cannot take parameters [#745](https://github.com/jqhph/dcat-admin/issues/745)
11. fix the problem that the `handle` method of the table row operation `action` can only get `id` of the last row of data.
12. fix the problem that `list` form edit page cannot delete existing data [#759](https://github.com/jqhph/dcat-admin/issues/759)
13. Fix the error in the `embeds` scope form's `name` attribute.
    
## v2.0.9-beta

Posted on 2020/11/18

To upgrade the method, execute the following step-by-step commands
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.9-beta"
php artisan admin:publish --assets --force
php artisan admin:publish --migrations --force # 表结构有变动
php artisan migrate
```


**Bug Fix**.

1. fix the function failure of form `filter::select` form remote load `/load/ajax` etc.
2. fix the front-end `moment-timezone` component path loading error [#701](https://github.com/jqhph/dcat-admin/issues/701)
3. fix the problem of not being able to set permissions due to `Form::tree` not being able to save data.
4. fix the problem of filling default value exception when the `hasMany` form has the same field name as the parent table.
5. repair the problem of adding new page report when form `tab` layout is nested with `row` layout [#648](https://github.com/jqhph/dcat-admin/issues/648)
6. fix the problem of not being able to get all the values under `range` after submission when the form has `range` type field.
7. fix the problem that select2 component is invalid when `Form::select` uses form linkage.


## v2.0.8-beta

Posted on 2020/11/16

To upgrade the method, execute the following command step by step
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.8-beta"
php artisan admin:publish --assets --force
php artisan admin:publish --migrations --force # Table structure changed
php artisan migrate
```

As a supplement to `2.0.7-beta`, the following issues are fixed in this release

1. fix the problem of not being able to view permissions on the admin page
2. fix the problem of form block layout failure
3. fix the problem of abnormal initialization of file upload form.
4. fix the problem that some form fields don't support rendering multiple fields on a single page at the same time.


## v2.0.7-beta

Posted on 2020/11/15

To upgrade the method, execute the following command step by step
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.7-beta"
php artisan admin:publish --assets --force
php artisan admin:publish --migrations --force # Changes in table structure
php artisan migrate
```

**Functional improvements**

1. Introduce the [jquery.initialize](https://github.com/pie6k/jquery.initialize) component to listen to dynamically generated page elements and set a callback, the following is a simple example to demonstrate usage.

In older versions, if an element is dynamically generated by `JS`, and we need to bind a click event to that element, then we usually do this

```html
<div class="selector">test</div>

<script>
Dcat.ready(function () {
    // You need to be off first and then on, otherwise page refresh will cause double bind problem.
    $(document).off('click', '.selector').on('click', '.selector', function () {
        ...
    })
});
</script>
```

The above approach is troublesome, you need to `off` and then `on`; secondly, you can not do some special treatment for dynamically generated elements, for example, if you want to change the background color after `.selector` is generated, there is no way to do this.

In this version we can use the `Dcat.init` method to listen to the dynamically generated elements.

```html
<div class="selector">test</div>

<script>
Dcat.ready(function () {
    // $this is the jquery dom object of the current element.
    // id is the id attribute of the current element, if the current element has no id, a random id will be generated automatically.
    Dcat.init('.selector', function ($this, id) {
        // Change the background color of the element.
        $this.css({background: "#fff"});
        
        // There's no need for off and then on again, because the anonymous function will only be executed once!
        $this.on('click', function () {
            ...
        });
    });
});
</script>
```

Thanks to the introduction of the [jquery.initialize](https://github.com/pie6k/jquery.initialize) component, in the current version we have optimized the front-end code of the form component to support the dynamically generated form type `HasMany` more easily. Significantly reduces the complexity of the code.


2.Form::hasMany and Form::array forms support column and row layout

If there are more fields, you can use `column` and `row` layout to save space.

```php
$form->array('field', function (NestedForm $form) {
    $form->column(6, function (NestedForm $form) {
        $form->text('...');
        
        ...
    });
    
    $form->column(6, function (NestedForm $form) {
        ...
    });
});
```

3.Routes configured with the admin.auth.except parameter do not require authentication privileges [#673](https://github.com/jqhph/dcat-admin/issues/673)


4. Add when method to Form, Grid and Show field classes

Usage examples, similar to `Laravel QueryBuilder`s `when` method

in the table
```php
// Closing code will be executed when the first parameter is true.
$grid->column('title')->when(true, function (Grid\Column $column, $value) {
    $column->label();
});
```

form (document)
```php
// Closing code will be executed when the first parameter is true.
$form->text('email')->when(true, function (Form\Field\Text $text, $value) {
    $text->type('email');
});
```

5. Administrator model add canSeeMenu method to control whether the menu is visible or not.

```php
<?php

namespace App\Models;

use Dcat\Admin\Models\Administrator as Model;

class Administrator extends Model
{
    /**
     * :: Control whether the menu is visible or not, return true by default
     * 
     * @param array|\Illuminate\Database\Eloquent\Model $menu menu node
     * @return bool
     */
    public function canSeeMenu($menu)
    {
        return true;
    }
}
```

6.Add admin_script、admin_style、admin_js、admin_css and admin_require_assets functions

```php
// Equivalent to Admin::script
admin_script('console.log(xxx)');

// Equivalent to Admin::style
admin_style('.my-class {color: red}');

// Equivalent to Admin::js() 
admin_js(['@admin/xxx.js']);

// Equivalent to Admin::css() 
admin_css(['@admin/xxx.css']);

// Equivalent to Admin::requireAssets() 
admin_require_assets(['@select2']);
```

7. simplify the action (Action) of the `JS` code logic, to remove the memory of the `selector` function

**BUG FIX**

1. Fix anomaly in the orderable form [#674](https://github.com/jqhph/dcat-admin/issues/674)
2. fix the JsonResponse methodIf error.
3. fix table, form, and data detail specifying `label` [#684](https://github.com/jqhph/dcat-admin/issues/684)
4. fix the problem that the table `Grid::rows` callback doesn't work properly.
5. fix the problem of some types of statistical cards failing to load asynchronously due to exceptions in adding `JS` code to widgets.
6. fix the getKey method exception for table row operations [#691](https://github.com/jqhph/dcat-admin/issues/691)
7. fix the problem of not being able to use the linkage function when there are multiple select forms in the page.
8. Fix the problem of the table not being able to refresh automatically after deleting data.


## v2.0.6-beta

Posted on 2020/11/7

To upgrade the method, execute the following command step by step
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.6-beta"
php artisan admin:publish --assets --force
php artisan admin:publish --migrations --force # 表结构有变动
php artisan migrate
```

**Breaking Changes**

1.`Form::tags` form is saved as `array` type by default
```php
// You need to convert the format you save to the database yourself
$form->tags('tag')->saveAsJson();
```

2.The session middleware is disabled by default

3.`Form\Tree::disableFilterParents` renamed to `Form\Tree::exceptParentNode`
```php
$form->tree('cate')->exceptParentNode(false);
```

4. Adjust the method name of the file upload form part
```php
// Enable chunked uploads, disableChunked changed to chunked
$form->image('avatar')->chunked(true);

// Enable auto-save field values, disableAutoSave changed to autoSave
$form->image('avatar')->autoSave(false);

// Enable the file deletion function, disableRemove changed to removable
$form->image('avatar')->removable(false);
```


**Functional improvements**

1.Code generator add field dragging sorting function, this method is contributed by partner [@codingyu](https://github.com/codingyu).

2. menu table to add `show` and `extension` field, `show` field is used to control whether to display the menu; `extension` field is used to mark whether to expand the menu

3.`Form::table`、`Form::array`、`Form::embeds` supports relational fields
```php
$form->table('profile.options', function ($form) {
    ...
});
```

4. Add vertical display of `Form::checkbox` and `Form::radio` form options.
```php
$form->checkbox('xxx')->inline(false)->options([...]);
```

5. Configuration file skip login and permission authentication allows configuration of routing aliases.
```php
'auth' => [
    'except' => [
        ...
        'user.login',
    ],
],
```

6.`Form\Row` adds `getKey` and `model` methods

7. Optimize the form filter select form selection effect, the default is not selected

8. Form tab layout optimization


**BUG FIX**
1. fix `Form::checkbox` problem when check/uncheck all options.
2. fix the problem that the default menu TITLE can not be translated in Taiwan Traditional Chinese. 
3. fix the problem that the `number` field in `NestedForm` is filtered when the input value is 0 [#634](https://github.com/jqhph/dcat-admin/issues/634)
4. fix the problem of getting primary key error when the model tree `RowAction` asynchronously processes the interface.
5. Fix the problem of table filters failing to reset the search value of the associated table fields [#650] (https://github.com/jqhph/dcat-admin/issues/650)
6. fix form filter multipleSelect form exception problem


## v2.0.5-beta 

Posted on 2020/10/29

BUG FIXES
1. fix the problem of table search multiple related table fields sql error [I232T7](https://gitee.com/jqhph/dcat-admin/issues/I232T7)
2. fix the problem of `Form::datetimeRange` form not being able to select logs.
3. Fix the problem of not being able to add multiple `Form::table` form fields [#627](https://github.com/jqhph/dcat-admin/issues/627)
4. fix the error in the form filter MultipleSelectTable.
5. Fix the problem of abnormal style of table specification filter.


## v2.0.4-beta 

Posted on 2020/10/27

BUG FIXES
1. fix the problem that the admin_javascript_json function will automatically empty-filter an array of null values.
2. fix the error of using tab layout for data form [#620](https://github.com/jqhph/dcat-admin/issues/620)
3. fix the problem of abnormal permissions for temporary directories generated by extended local installation [#625](https://github.com/jqhph/dcat-admin/issues/625)
4. fix the error when using html method to set view with script tag [#624](https://github.com/jqhph/dcat-admin/issues/624).
5. Fix the error of displaying data details using correlations (one-to-many) [#623](https://github.com/jqhph/dcat-admin/issues/623)
6. fix the dropdown menu calculation display position exception [#I22S2N](https://gitee.com/jqhph/dcat-admin/issues/I22S2N)


## v2.0.3-beta 

Posted on 2020/10/27

BUG FIXES
1. fix the problem of abnormal display of return information in form row editing
2. Fix invalid setting of `admin.auth.member` [#613](https://github.com/jqhph/dcat-admin/issues/613)
3. fix the abnormal Chinese translation problem of `editor` form [#611](https://github.com/jqhph/dcat-admin/issues/611)
4. fix the problem that `Filter::scope` can't filter pagination parameters when selecting filter item.
5. fix issues related to form event interception
6. Fix the problem of abnormal use of tree form [#619](https://github.com/jqhph/dcat-admin/issues/619)

Functional improvements
1. add the configuration of skip privileges and login authentication
2. Extending the service provider to include middleware and route-checking registrations
3. batch operation change event monitoring optimization
4. Increase `RowSelector` robustness to avoid errors due to unprocessed `json` array type fields [#609](https://github.com/jqhph/dcat-admin/pull/609)

## v2.0.2-beta 

Posted on 2020/10/21

BUG FIXES
1. Fix naming space exception in controller file generated by code generator #600 
2. fix the problem of configuration file logo path error
3. Fix the problem of invalid search for associated fields in tables
4. fix the problem of duplicate selectors generated by model tree operation
5. fix the problem of accessing permissionless page report
6. Fix the problem that table filter multipleSelect cannot select the value of the associated table field #603 
7. Fix invalid form tab layout #605 

Functional improvements
1. Auth\Permission to move to Http directory
2. Replace the json field in the data table with text
3. add login password error translation
4. add admin_javascript_json function, make most of the component configuration support passing JS code
5. Admin::color Add dark mode color.




## v2.0.1-beta 

Posted on 2020/10/20

BUG FIXES
- Fix data table filter search bug #599 
- Fix code generator error in generating controller base class namespace #600 

Functional improvements

- Code Generator adds page TITLE and breadcrumb translation functionality.
- Exception handling optimization
- Add admin_setting_array function.