# Single/check box

## radio boxes

The `Dcat\Admin\Widgets\Radio` class makes it easy to quickly build a radio form.

### Basic use

```php
<?php
use Dcat\Admin\Widgets\Radio;

// Form name Attribute
$name = 'state';
// options
$options = [
   1 => 'unprocessed',
   2 => 'processed',
   3 => 'rejected',
];

$radio = Radio::make($name, $options)->check(1); // Check the first option
```

result

<a href="{{public}}/assets/img/screenshots/radio-1.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/radio-1.png"  style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" >
</a>

### Display on the same line (inline)

```php
<?php
use Dcat\Admin\Widgets\Radio;

$name = 'state';
$options = [
   1 => 'unprocessed',
   2 => 'processed',
   3 => 'rejected',
];

$radio = Radio::make($name, $options)->check(1)->inline();
```
result

<a href="{{public}}/assets/img/screenshots/radio-2.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/radio-2.png"  style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" >
</a>

### Setting the unchecked options (disable)


```php
<?php
use Dcat\Admin\Widgets\Radio;

$name = 'state';
$options = [
   1 => 'unprocessed',
   2 => 'processed',
   3 => 'rejected',
];

$radio = Radio::make($name, $options)->inline()->disable([2, 3]);
```
result

<a href="{{public}}/assets/img/screenshots/radio-3.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/radio-3.png"  style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" >
</a>

### set the style (style)

The `style` method allows you to set the style of the radio box, supporting `primary`, `info`, `success`, `danger`.

### Set the size
    
    The radio box supports size 3 in the following way


`small` Set to small size
```php
$radio->small();
```

`large` Set to large size
```php
$radio->large();
```


## Checkboxes

The `Dcat\Admin\Widgets\Checkbox` class makes it easy to quickly build checkbox forms, and the checkbox class is a subclass of `Radio`, so the Usage is basically the same as the `Radio` class.

### Basic Usage

```php
<?php
use Dcat\Admin\Widgets\Checkbox;

// Form name attribute, with brackets because it is multiple choice
$name = 'hobbies[]';
// options
$options = [
   1 => 'sing',
   2 => 'jump',
   3 => 'RAP',
   4 => 'play basketball',
];

$checkbox = Checkbox::make($name, $options)
    ->inline()
    ->check([1, 2]); // Allow passing arrays here, multiple options are selected by default
```

result

<a href="{{public}}/assets/img/screenshots/checkbox-1.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/checkbox-1.png"  style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" >
</a>

### 全选

The `checkAll` method allows you to check all options.

```php
<?php
use Dcat\Admin\Widgets\Checkbox;

// Form name attribute, with brackets because it is multiple choice
$name = 'hobbies[]';
// options
$options = [
   1 => 'sing',
   2 => 'jump',
   3 => 'RAP',
   4 => 'play basketball',
];

$checkbox = Checkbox::make($name, $options)
    ->inline()
    ->checkAll(); // Check all
```

The `checkAll` method also allows you to exclude specified options while selecting all.

```php
$checkbox = Checkbox::make($name, $options)
    ->inline()
    ->checkAll([1, 3]); // Select all, but exclude options with keys 1 and 3
```

### More Usages

More Usages with `Radio` all the way, so I won't repeat them here.
