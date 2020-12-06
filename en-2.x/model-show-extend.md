# Detail field display expansion

This feature is used to extend the detail field display, in cases where the built-in display method is not sufficient.

First define the extension class:

```php
<?php

namespace App\Admin\Extensions\Show;

use Dcat\Admin\Show\AbstractField;

class UnSerialize extends AbstractField
{
    // Setting this property to false will not escape the HTML code
    public $escape = false;
    
    public function render($arg = '')
    {
        // Returns any content that can be rendered
        return unserialize($this->value);
    }
}
```
Then register the extension class in `app/Admin/bootstrap.php`

```php
use Dcat\Admin\Show\Field;
use App\Admin\Extensions\Show\UnSerialize;

Field::extend('unserialize', UnSerialize::class);
```
Then use this extension in the controller

```php
$show->column()->unserialize('xxx');
```
Parameters passed into the unserialize() method are passed sequentially into the UnSerialize::render() method.

Several common attributes can be found in the parent class `Dcat\Admin\Show\AbstractField`

```php
/**
 * Field value.
 *
 * @var mixed
 */
protected $value;

/**
 * Current field model.
 *
 * @var Fluent
 */
protected $model;

/**
 * If this field show with a border.
 *
 * @var bool
 */
public $border = true;

/**
 * If this field show escaped contents.
 *
 * @var bool
 */
public $escape = true;
```
Where `$value` and `$model` are the current field value and the current details of the data, in the `render()` method can be used to get the data you want.

`$border` is used to control whether the current display needs a border, `$escape` is used to set the current display to not HTML escape.