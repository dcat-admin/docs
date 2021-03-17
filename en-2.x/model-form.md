# Basic use of forms

## Example
The `Dcat\Admin\Form` class is used to quickly generate form pages. Let's start with an example. There is a `movies` table in the database:

```sql
CREATE TABLE `movies` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `title` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
  `director` int(10) unsigned NOT NULL,
  `describe` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
  `rate` tinyint unsigned NOT NULL,
  `released` enum(0, 1),
  `release_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  `created_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  `updated_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;

```

The corresponding data model is `App\Admin\Repositories\Movie' and the data repository is `App\Admin\Repositories\Movie':

```php
use App\Admin\Repositories\Movie;
use Dcat\Admin\Form;
use Dcat\Admin\Admin;

$form = Form::make(new Movie(), function (Form $form) {
    // 显示记录id
    $form->display('id', 'ID');
    
    // 添加text类型的input框
    $form->text('title', 'Film TITLE');
    
    $directors = [
        1 => 'John',
        2 => 'Smith',
        3 => 'Kate',
    ];
    
    $form->select('director', 'Director')->options($directors);
    
    // Add textarea input box for description.
    $form->textarea('describe', 'INTRODUCTION');
    
    // numeric input box
    $form->number('rate', 'give a mark');
    
    // Adding Switch Operation
    $form->switch('released', 'published？');
    
    // Add a date and time selection box
    $form->datetime('release_at', 'Release Date');
    
    // Two time displays
    $form->display('created_at', 'Creation Date');
    $form->display('updated_at', 'Modification Date');
});
```

## Repositories

Data warehouse (`Repository`) is a class that can provide a specific interface to read and write operations on the data, through the introduction of data warehouses, you can make the construction of the page no longer care about the specific implementation of data read and write functions, developers only need to implement a specific interface to easily switch data sources. For detailed usage of the data warehouse, please refer to the document [data warehouse](model-repository.md).


## Form definition

It is recommended to build the form in the following way
```php
use App\Admin\Repositories\Movie;
use Dcat\Admin\Form;
use Dcat\Admin\Admin;

$form = Form::make(new Movie, function (Form $form) {
    // Display record id
    $form->display('id', 'ID');

    $form->select('director', 'Director')->options($directors);
    
    ...
});
```

### Get current model data

The data of the current model is available within the closed package (edit)
```php
Form::make(new Movie, function (Form $form) {
    // Display record id
    $form->display('id', 'ID');

    // Gets model data and displays the "rate" field if "status == 1
    if ($form->model()->status == 1) {
        $form->number('rate');
    }
    
    $form->select('director', 'Director')->options($directors);
    
    ...
});
```


## Custom tools

By default, there are two buttons on the top right corner of the form, Back and Jump List, which can be modified in the following way:

```php
$form->tools(function (Form\Tools $tools) {
    // Removing the Jump List button
    $tools->disableList();
    // Removing the Jump to Details button
    $tools->disableView();
    // Removing the delete button
    $tools->disableDelete();

    // To add a button, arguments can be a string, an anonymous function, or an instance of an object that implements the Renderable or Htmlable interface.
    $tools->append('<a class="btn btn-sm btn-danger"><i class="fa fa-trash"></i>&nbsp;&nbsp;delete</a>');
});

// Remove the entire toolbar content
$form->disableHeader();

// You can also remove the default buttons from the toolbar via the following methods
$form->disableListButton();
$form->disableViewButton();
$form->disableDeleteButton();

```

### Custom complex tool buttons

Please refer to the document [form action](action-form.md).


## Bottom of the form
Use the following method to remove elements from the bottom of a form

```php
$form->footer(function ($footer) {

    // Remove `Reset' button
    $footer->disableReset();

    // Remove the `Submit' button
    $footer->disableSubmit();

    // Remove the `view` checkbox
    $footer->disableViewCheck();

    // Remove `Continue to edit` checkbox
    $footer->disableEditingCheck();

    // Remove `Continue to create `checkbox
    $footer->disableCreatingCheck();
    
    // Set `View` to be checked by default
	$footer->defaultViewChecked();

	// set `Continue editing` to be checked by default
	$footer->defaultEditingChecked();
	
	// set `Continue creating` to be checked by default
	$footer->defaultCreatingChecked();
});

// Remove the entire bottom portion
$form->disableFooter();

// You can also remove the bottom elements via the following methods
$form->disableSubmitButton();
$form->disableResetButton();
$form->disableViewCheck();
$form->disableEditingCheck();
$form->disableCreatingCheck();

// Set `Viewing` to be checked by default
$form->defaultViewChecked();

// set `Continue editing` to be checked by default
$form->defaultEditingChecked();

// set `Continue creating` to be checked by default
$form->defaultCreatingChecked();
```

Customizing Views

```php
$form->footer(function ($footer) {
    $footer->view('...' , [...]) ;
});
```

## Common methods

### Form layout

Please refer to [form layout](model-form-layout.md)

### Return field validation error messages (responseValidationMessages)

The `responseValidationMessages` method makes it easy to return field validation error messages without using the `Laravel validation` feature.

General use
```php
protected function form()
{
	return Form::make(new Model(), function (Form $form) {
		if (...) { // Validation logic
			$form->responseValidationMessages('title', 'Wrong title format');
			
			// If there are multiple error messages, the second parameter can be passed as an array.
			$form->responseValidationMessages('content', ['content format error', 'content cannot be empty']);
		}
	});
}
```
Use in events
> {tip} This method is only available for the `submitted` event

```php
$form->submitted(function (Form $form) {
	// Receive form parameters
	$title = $form->title;

    if (...) { // Validation logic
        $form->responseValidationMessages('title', 'Wrong title format');
        
        // If there are multiple error messages, the second parameter can be passed as an array.
        $form->responseValidationMessages('content', ['content format error', 'content cannot be empty']);
    }
});
```

### Remove the submit button

```php
// Remove the submit button
$form->disableSubmitButton();

// Remove the reset button
$form->disableResetButton();
```

### Ignore fields that don't need to be saved (ignore)

```php
$form->ignore(['column1', 'column2', 'column3']);

// Cancel ignored fields
$form->removeIgnoredFields(['column1',]);
```

### Set the width

Here the width value is a number between `1-12`, the first parameter is the width of ```field``` and the second parameter is the width of ```label``` which can be omitted!

```php
$form->width(10, 2);
```

### Set the action for form submission


```php
$form->action('auth/users');
```

### Determine if you are adding (isCreating)

Both adding pages and saving new data can be determined using this method

```php
if ($form->isCreating()) {
    ...
}
```

### Determine if isEditing

This method can be used for both editing pages and saving editing data

```php
if ($form->isEditing()) {
    ...
}
```

### Determine if it is a deletion (isDeleting)

```php
if ($form->isDeleting()) {
    ...
}
```

### Get ID (getKey)

Adding a new page is not valid and must be used in a closed package.

```php
return Form::make(new User, function (Form $form) {
    $id = $form->getKey();
    
    ...
});
```

### Get edit data (model)
Adding a new form is invalid and must be used inside a closed package.

```php
return Form::make(new User, function (Form $form) {
    $username = $form->model()->xxx;
    
    ...
});
```

### Get data from form submission (input)

```php
$form->saving(function (Form $form) {
    $username = $form->username;
    
    // equal to
    $username = $form->input('username');
});
```

### Modify or delete form submission data

```php
$form->saving(function (Form $form) {
    // modify
    $form->input('username', 'Marry');
    // or
    $form->username = 'Marry';
    
    // remove
    $form->deleteInput('username');
});
```


### Get the final saved data (updates)

This method only works on `saved` callbacks.

```php
$form->saved(function (Form $form) {

    $data = $form->updates();
    
});
```

<a name="redirect"></a>
### Page Jump (redirect)

Jump to the specified page, this method is only available within the [form callback](model-form-callback.md) event

```php
// Jump and prompt for success information
$form->saved(function (Form $form) {
    return $form->response()->success('Save success')->redirect('auth/user');
});

// Jump and error messages
$form->saving(function (Form $form) {
    return $form->response()->error('system error')->redirect('auth/user');
});
```

<a name="confirm"></a>
### Show confirmation popup window

A popup confirmation popup appears when you click the submit button.
```php
$form->confirm('Are you sure you want to submit the form?？', 'content');
```


### Set the outer container (wrap)
```php
 // Changing the tab's outer container
$form->wrap(function (Renderable $view) {
    $tab = Tab::make();
    
    $tab->add('Example', $view);
    $tab->add('Code', $this->code(), true);

    return $tab;
});
```


<a name="saving"></a>
### Modify form input values to be saved (saving)

The `saving` method allows you to change the format of the data to be saved.

```php
use Dcat\Admin\Support\Helper;

$form->mutipleFile('files')->saving(function ($paths) {
    $paths = Helper::array($paths);
    
    // Get other fields of the current row of the database
    $username = $this->username;
    
    // finally convert to json and save to database
    return json_encode($paths);
});
```

<a name="customFormat"></a>
### Modify form data display (customFormat)
The `customFormat` method can be used to change the value of a field injected into the form from an external source.

In the following example, the `mutipleFile` field requires the field value to be rendered in array format, which can be converted to `array` format by using the `customFormat` method.
```php
use Dcat\Admin\Support\Helper;

$form->mutipleFile('files')->saving(function ($paths) {
    $paths = Helper::array($paths);
    
    return json_encode($paths);
})->customFormat(function ($paths) {
    // Get other fields of the current row of the database
    $username = $this->username;

    // convert to array
    return Helper::array($paths);
});
```
