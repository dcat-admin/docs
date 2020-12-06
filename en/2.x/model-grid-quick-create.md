# Quick Create


After opening this feature in the form, will add a `form` form in the head of the form to create data, for some simple form page, you can easily and quickly create data, without having to jump to the creation of the page to operate

<a href="{{public}}/assets/img/screenshots/quick-create.png" target="_blank">
    <img  src="{{public}}/assets/img/screenshots/quick-create.png" style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%">
</a>


### Basic use

> {tip} Note that each item in the Quick Create form is set to the same type of form item on the `form` form page.

```php
$grid->quickCreate(function (Grid\Tools\QuickCreate $create) {
    $create->text('name', '名称');
    $create->email('email', '邮箱');
});
```

### Set the submission address


```php
$grid->quickCreate(function (Grid\Tools\QuickCreate $create) {
    $create->action('auth/users');
    $create->method('GET');
});
```


The following types of form items are supported by the form


### Text(text)
text input box

```php
$create->text('column_name', 'placeholder...');
```

### Hidden form
text input box


```php
$create->hidden('column_name');
```

### E-mail (email)
E-mail input box
```php
$create->email('column_name', 'placeholder...');
```

### IP input box
ip address input box

```php
$create->ip('column_name', 'placeholder...');
```

### URL input box
url input box
```php
$create->url('column_name', 'placeholder...');
```


### Password
Password input box
```php
$create->password('column_name', 'placeholder...');
```

### Phone number (mobile)
Mobile phone number input box
```php
$create->mobile('column_name', 'placeholder...');
```

### integer.
Integer input box
```php
$create->integer('column_name', 'placeholder...');
```

### Drop-down checkbox (select)
radio box
```php
$create->select('column_name', 'placeholder...')->options([
    1 => 'foo',
    2 => 'bar',
]);
```


### Dropdown box multipleSelect
multiple-choice box
```php
$create->multipleSelect('column_name', 'placeholder...')->options([
    1 => 'foo',
    2 => 'bar',
]);
```

### Tags (tags)
```php
$create->tags('column_name', 'placeholder...')->options([
    1 => 'foo',
    2 => 'bar',
]);
```

### Popup window selector (selectResource)


Through the `selectResource` form you can build a pop-up selector, you can select from the pop-up form data, and support for data filtering and other operations.


```php
 $form->selectResource('user')
    ->path('auth/users') // Setting up links to form pages
    ->options(function ($v) { // Show selected data
        if (!$v) return $v;
        $userModel = config('admin.database.users_model');

        return $userModel::find($v)->pluck('name', 'id');
    });
    
// Set to Multiple Choice
 $form->selectResource('user')
    ->path('auth/users') 
    ->multiple() // Set to Multiple Choice
    ->options(function ($v) {
        ...
    });  
    
// Limit the maximum number of selections
 $form->selectResource('user')
    ->path('auth/users') 
    ->multiple(3) // Select up to 3 options
    ->options(function ($v) {
        ...
    });       
```

Then set up your route `app/Admin/routes.php`

> {tip} Adding a route here is just Example, if you are adding a new controller you need to add a route, if the route already exists then you don't need to add it.

```php
$router->resource('auth/users', 'UserController');
```

The `auth/users` page implements the following code:
```php
<?php

use Dcat\Admin\Models\Administrator;
use Dcat\Admin\IFrameGrid;
use Dcat\Admin\Grid;
use Dcat\Admin\Controllers\AdminController;

class UserController extends AdminController
{
    protected function iFrameGrid()
    {
        $grid = new IFrameGrid(new Administrator());
        
        // Specify the field name of the value to be displayed when the line selector is selected.
        // Specify the field name of the value to be displayed when the line selector is selected.
        // Specify the field name of the value to be displayed when the line selector is selected.
        // If the form data has a "name", "title", or "username" field, you do not have to set this.
        $grid->rowSelector()->titleColumn('username');

        $grid->id->sortable();
        $grid->username;
        $grid->name;
    
        $grid->filter(function (Grid\Filter $filter) {
            $filter->equal('id');
            $filter->like('username');
            $filter->like('name');
        });
    
        return $grid;
    }
}
```

result

<a href="{{public}}/assets/img/screenshots/select-resource.png" target="_blank">
    <img  src="{{public}}/assets/img/screenshots/select-resource.png" style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%">
</a>


### Date and time selection
Time and date input box
```php
$create->datetime('column_name', 'placeholder...');
```

### Time selection (time)
Time input box
```php
$create->time('column_name', 'placeholder...');
```

### Date selection
```php
$create->date('column_name', 'placeholder...');
```
