# Form validation

### rule

`model-form` uses lavel's validation rules to validate data submitted by the form

```php
$form->text('title')->rules('required|min:3');

// Complex validation rules can be implemented inside callbacks
$form->text('title')->rules(function (Form $form) {
    
    // Add field unique validation if not edit status
    if (!$id = $form->model()->id) {
        return 'unique:users,email_address';
    }
    
});
```

Custom error messages can also be given for validation rules:

```php
$form->text('code')->rules('required|regex:/^\d+$/|min:10', [
    'regex' => 'The code must be all numeric',
    'min'   => 'Code cannot be less than 10 characters',
]);
```

If you want to allow the field to be empty, first set the field to `NULL` in the database table, then

```php
$form->text('title')->rules('nullable');
```

See [Validation](https://laravel.com/docs/5.5/validation) for more rules.

### creationRules

This method is exactly the same as `Form\Field::rule`, except that it only works when data is added.

> {tip} If the `creationRules` method is called, the validation rules set by the `rule` method will be ignored.

### updateRules

This method is exactly the same as `Form\Field::rule`, except that it only works when you update the data.

> {tip} If the `updateRules` method is called, the validation rules set by the `rule` method will be ignored.


## responseValidationMessages

A custom validation error message can be returned via the `Form::responseValidationMessages` method, as follows:
```php
// "PUT" method for editing submissions
if (request()->getMethod() == 'PUT') {
    if (...) { // Your validation logic
        $form->responseValidationMessages('title', 'Wrong title format');
        
        // If there are multiple error messages, the second parameter can be passed as an array.
        $form->responseValidationMessages('content', ['content format error', 'content cannot be empty']);
    }
}

$form->text('title');
$form->text('content');

$form->submitted(function ($form) {
    if (...) { 
        $form->responseValidationMessages('title', '...');
    }
});
```

## Front-end verification

The system inherits <a href="https://github.com/1000hz/bootstrap-validator" target="_blank">bootstrap-validator</a> for front-end form validation and supports validation of H5 form types.

> {tip} Browsers that don't support H5 can also use front-end validation, and the system has been made compatible. Most forms support both front-end and back-end validation, both can work at the same time without conflict, a few forms front-end validation is invalid.

### H5 verification

#### required

mandatory fields
```php
$form->text('title')->required();
```

#### number

Only numbers are allowed
```php
$form->text('age')->type('number');
```

limits
```php
// Only numbers in the range of 10-60 are allowed.
$form->text('age')
    ->type('number')
    ->attribute('min', 10)
    ->attribute('max', 60);
```

#### email

email address
```php
$form->email('email');
```

#### url

links
```php
$form->text('website')->type('url');
```

### Other

#### minLength

Limit minimum character length

```php
$form->text('title')->minLength(20);

// Setting error messages
$form->text('title')->minLength(20, 'Minimum of 20 characters');
```

#### maxLength

Limit the maximum length of characters
```php
$form->text('title')->maxLength(50);

// Setting error messages
$form->text('title')->maxLength(50, 'No more than 50 characters.');
```

#### same

Limits the value of the current field to be equal to the value of the given field.

```php
$form->password('password');

$form->password('password_confirm')->same('password');

// Setting error messages
$form->password('password_confirm')->same('password', 'Two inconsistent password entries');
```

### Custom

Developers can customize front-end validation rules in the following ways



Add the following code to `app/Admin/bootstrap.php`.
```php
use Dcat\Admin\Form\Field;

Field\Text::macro('len', function (int $length, ?string $error = null) {
    // Extensions to the front-end verification logic
    Admin::script(
                <<<'JS'
Dcat.validator.extend('len', function ($el) {
    return $el.val().length != $el.attr('data-len');
});
JS
        );

        // Also add back-end verification logic, depending on the need
        $this->rules('size:'.$length);

        return $this->attribute([
            'data-len'       => $length,
            'data-len-error' => str_replace(
                [':attribute', ':len'],
                [$this->label, $length],
                $error ?: "Only :len characters can be entered."
            ),
        ]);
});
```

use

```php
$form->text('name')->len(10);
```



