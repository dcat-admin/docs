# v1.x Upgrade Guide


### Preface

This section contains only the `1.x` version of the `API` changes, does not contain new features or changes that do not affect the user's use, `2.0` specific version change description please refer to [what changes in 2.0?] (https://learnku.com/articles/50781?#reply164307)

**Estimated upgrade time: 60 minutes**


### 1. Create a new branch and backup configuration files

Create a new branch and backup the configuration file `config/admin.php` named `config/admin.bak.php` to facilitate subsequent comparison of configuration changes.

### 2. Update composer dependencies

Uninstall `1.x` version first
```bash
composer remove dcat/laravel-admin
```

Re-installation
```
composer require dcat/laravel-admin:"2.*"
```

After installing success

1. delete the `public/vendors` directory
2. republish the resource `php artisan admin:publish --force`.
3. write the modified parameters to the new configuration file `config/admin.php` according to the above backed up configuration file, here it should be noted that the default theme color for `1.x` is `indigo` (deprecated), which has been replaced with `default` in the new version
4. Adjust the language pack, the language pack directory is changed from `zh-CN` to `zh_CN` in the new version, you need to move the custom translation files to the new directory, and the translation of `menuTITLE` is also separated into `menus.php`.
5. Run the database migration command `php artisan migrate`, two new tables `admin_settings` and `admin_extensions` have been added in the new version

### 3. Global change namespace

1. Global search namespace `Dcat\Admin\Controllers` and replace it with `Dcat\Admin\Http\Controllers`
2. global search namespace `Dcat\Admin\Auth` and replace it with `Dcat\Admin\Http\Auth


### 4. Form section changes

1. The field hiding function has been adjusted, the ``responsive`` method has been deprecated in the old version, in the new version the field hiding function is enabled as follows

```php
// Turn on the field selector function
$grid->showColumnSelector();

// Set default hidden fields
$grid->hideColumns(['field1', ...]) ;
```
2. The methods `collection` and `fetching` have been removed from the table and can be replaced in the new version by the following events

```php
use Dcat\Admin\Grid;
use Illuminate\Support\Collection;

// Use Grid\Events\Fetched event instead of collection
$grid->listen(Grid\Events\Fetched::class, function ($grid, Collection $rows) {
    $rows->transform(function ($row) {
        // Changing row data
        $row['name'] = $row['first_name'].' '.$row['last_name'];
        
        return $row;
    });
});

// Use Grid\Events\Fetching event instead of fetching
$grid->listen(Grid\Events\Fetching::class, function ($grid) {
    
});
```

3. The use of models is allowed in the table row closure function

```php
$grid->column('avatar')->display(function ({
    // Direct access to model-related methods
    return $this->getAvatar();
});
```


4. Set route prefix method from `resource` to `setResource`.
```php
$grid->setResource('auth/users');
```

5. Tree form `tree` method will be deprecated soon and will be moved to extension center


### 5. Changes to the form section

1. Adjusted form handling response methods, `success`, `error`, `redirect` and `location` methods have been removed from the old version.
   In `2.0` we unified the form response methods with the `action` response methods, please refer to the document [form response](x), Example for detailed Usage

```php
$form->saving(function (Form $form) {
    return $form
        ->response()
        ->success('Save success')
        ->script('console.log("Execution of JS code")')
        ->redirect('auth/users');
});
```

In the case of [tools-form](widgets-form.md), the Usage is as follows
```php
public function handle(array $input)
{
    ...

    return $this
        ->response()
        ->alert()
        ->success('success')
        ->detail('Details');
}
```

2. adjust the form `block` layout function, and deprecate the `setDefaultBlockWidth` method, please refer to the document [form block layout](x), Example for detailed Usage

```php
$form->block(8, function (Form\BlockForm $form) {
    $form->title('Basic Settings');
    $form->showFooter();
    $form->width(9, 2);

    $form->column(6, function (Form\BlockForm $form) {
        $form->display('id');
        $form->text('name');
        $form->email('email');
        $form->image('avatar');
        $form->password('password');
    });

    $form->column(6, function (Form\BlockForm $form) {
        $form->text('username');
        $form->email('mobile');
        $form->textarea('description');
    });
});
$form->block(4, function (Form\BlockForm $form) {
    $form->title('Sub-block 2');

    $form->text('nickname');
    $form->number('age');
    $form->radio('status')->options(['1' => 'Default', 2 => 'Frozen'])->default(1);

    $form->next(function (Form\BlockForm $form) {
        $form->title('Sub-block 3');

        $form->date('birthday');
        $form->date('created_at');
    });
});
```


3. Deprecate the direct form submission, keep only the `ajax` submission method, and rename the `disableAjaxSubmit` method to `ajax`

```php
$form->ajax(false);
```

4. Deprecate step-by-step form, please use [step-by-step form extension](https://github.com/dcat-admin/form-step) instead in new version

6. `map` and `listbox` will be deprecated soon, and the extension center will be moved.



### 6. Data repository part of the changes

1. The naming of the data repository interface has been simplified, the new interface is as follows

```php
interface Repository
{
    /**
     * Get primary key name.
     *
     * @return string
     */
    public function getKeyName();

    /**
     * Gets the creation time field.
     *
     * @return string
     */
    public function getCreatedAtColumn();

    /**
     * Get the update time field.
     *
     * @return string
     */
    public function getUpdatedAtColumn();

    /**
     * Whether to use soft delete.
     *
     * @return bool
     */
    public function isSoftDeletes();

    /**
     * Get Grid table data.
     *
     * @param Grid\Model $model
     *
     * @return \Illuminate\Contracts\Pagination\LengthAwarePaginator|Collection|array
     */
    public function get(Grid\Model $model);

    /**
     * Get edit page data.
     *
     * @param Form $form
     *
     * @return array|\Illuminate\Contracts\Support\Arrayable
     */
    public function edit(Form $form);

    /**
     * Get detail page data.
     *
     * @param Show $show
     *
     * @return array|\Illuminate\Contracts\Support\Arrayable
     */
    public function detail(Show $show);

    /**
     * Add a new record.
     *
     * @param Form $form
     *
     * @return mixed
     */
    public function store(Form $form);

    /**
     * Query the row data before the update.
     *
     * @param Form $form
     *
     * @return array|\Illuminate\Contracts\Support\Arrayable
     */
    public function updating(Form $form);

    /**
     * Update data.
     *
     * @param Form $form
     *
     * @return bool
     */
    public function update(Form $form);

    /**
     * Delete Data.
     *
     * @param Form  $form
     * @param array $deletingData
     *
     * @return mixed
     */
    public function delete(Form $form, array $deletingData);

    /**
     * Query the row data before deletion.
     *
     * @param Form $form
     *
     * @return array|\Illuminate\Contracts\Support\Arrayable
     */
    public function deleting(Form $form);
}
```


2.`EloquentRepository::eloquent()` 重命名为 `EloquentRepository::model()`

### 7. Section changes

In the new version `AdminSection` has been removed, please use `Dcat\Admin\Admin::SECTION` constant instead

```php
use Dcat\Admin\Admin;

admin_inject_default_section(Admin::SECTION['HEAD'], function () {
    return ...;
});
```


### 8. Extensions

For extension-related changes, please refer to the document [extensions](extension-install.md)

### 9. Login logic
1. Login template, if you have customized the login template in the old project, you need to adjust the `JS` code in the login template
```js
Dcat.ready(function () {
    // ajax form submission
    $('#login-form').form({
        validate: true,
    });
});
```

2. Login logic, if the login logic has been rewritten, the response method for the final login success needs to use `sendLoginResponse`

### 10. Other changes

1. Resource registration
```php
use Dcat\Admin\Admin;

// Registered resource path alias
Admin::asset()->alias('test', 'assets/test');

Admin::asset()->alias('Name', [ 
    'js' => [
        // @test will be evaluated as an alias
        '@test/test.js',
    ],
    'css' => [
        '@test/test.css',
    ],
]);


// Loading Resources
Admin::asset()->require('@Name');
// Load only js
Admin::js('@Name');
// Load only css
Admin::css('@Name');
```




