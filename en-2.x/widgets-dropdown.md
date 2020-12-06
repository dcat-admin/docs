# Drop-down menu

The `Dcat\Admin\Widgets\Dropdown` class is a quick way to help you build dropdown menu functions.


### Basic Usage

```php
<?php

use Dcat\Admin\Widgets\Dropdown;
use Dcat\Admin\Layout\Content;

class MyController
{
    public function index(Content $content)
    {
        $options = [
            'Name 1',
            'Name 2',
            'Name 3',
            'Name 4',
            'Name 5',
        ];
        
        $dropdown = Dropdown::make($options)
            ->button('Category Navigation') // Set button
            ->buttonClass('btn btn-white  waves-effect') // Set the button style
            ->map(function ($label, $key) {
                // Format menu options
                $url = admin_url('categories/'.$key);

                return "<a href='$url'>{$label}</a>";
            });
         
        return $content->body($dropdown);
    }
}
```
result

<a href="{{public}}/assets/img/screenshots/dropdown-1.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/dropdown-1.png"  style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="70%" >
</a>

### Click on the menu to change button text

The `click` method allows the text of the selected menu to be displayed in the button, similar to the result of a drop-down selection.

```php
$options = [
    ...
];

$dropdown = Dropdown::make($options)
    ->button('option') // Set button
    ->click();
```

### Set TITLE

```php
$options1 = [
    'Name 1',
    'Name 2',
];

$options2 = [
    'Test 1',
    'Test 2',
];

$dropdown = Dropdown::make()
    ->button('Use of TITLE')
    ->options($options1, 'TITLE1')
    ->options($options2, 'TITLE2');
```
result

<a href="{{public}}/assets/img/screenshots/dropdown-2.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/dropdown-2.png"  style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="70%" >
</a>

### 增加分割线

```php
$options = [
    'Name 1',
    'Name 2',
    Dropdown::DIVIDER,
    'Name 3',
    'Name 4',
];

$dropdown = Dropdown::make()
    ->button('Using dividing lines')
    ->options($options)
```
result

<a href="{{public}}/assets/img/screenshots/dropdown-3.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/dropdown-3.png"  style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="70%" >
</a>

### Custom buttons

```php
    public function index(Content $content)
    {
        $options = [
            'Name 1',
            'Name 2',
            'Name 3',
            'Name 4',
            'Name 5',
        ];
        
        $dropdown = Dropdown::make($options)
            ->map(function ($label, $key) {
                // Formatting menu options
                $url = admin_url('categories/'.$key);

                return "<a href='$url'>{$label}</a>";
            });
         
        return $content->body(
            <<<HTML
<div class='dropdown'>
     <button class='btn btn-primary dropdown-toggle' data-toggle='dropdown'>
        <i class='feather icon-email'></i> Custom buttons 
     </button>
     {$dropdown->render()}
</div>            
HTML            
        );
    }
```

result

<a href="{{public}}/assets/img/screenshots/dropdown-4.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/dropdown-4.png"  style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="70%" >
</a>
