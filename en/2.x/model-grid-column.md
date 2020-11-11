# Basic use of columns

### Set columns to be sortable
```php
$grid->column('id')->sortable();
```

### Set the width of the column (width)
Set the column width, which can be used to limit the column width when the field is too long
```php
// px
$grid->column('long_text')->width('300px');
// percentage
$grid->column('long_text')->width('15%');
```

### fixColumns


Through `fixColumns` method, you can set fixed columns for the table, the first parameter is to fix the first three columns from the beginning, the second parameter is to fix two columns from the back to the front, (the second parameter can not be passed, the default is -1).

```php
$grid->fixColumns(2, -2);
```

result

<a href="{{public}}/assets/img/screenshots/fixcolumn.gif" target="_blank">
    <img class="img" src="{{public}}/assets/img/screenshots/fixcolumn.gif" />
</a>    


### Set the table header HTML attributes
Set the `html` attribute of TITLE
```php
// Change Color
$grid->column('name')->setHeaderAttributes(['style' => 'color:#5b69bc']);
```

### Set the columns to show or hide
The `responsive` method is used to enable the [RWD-Table-Patterns](https://github.com/nadangergeo/RWD-Table-Patterns) plugin, which controls whether to show or hide the current field in the upper-right corner of the form.
```php
$grid->column('email')->responsive();

// 0 not visible
// 1 Remain visible, but can be hidden by filtering in the drop-down list.
// 2 Visible below 480px resolution.
// 3 640px visible below
// 4 800px visible below
// 5 960px visible below
// 6 1120px visible below
$grid->column('name')->responsive(2);
```

#### default hide as responsive
```php
// Hidden fields
$grid->column('email')->hide();
// equal to
$grid->column('name')->responsive(0);
```
<a href="{{public}}/assets/img/screenshots/grid-column-responsive.png" target="_blank">
    <img class="img" src="{{public}}/assets/img/screenshots/grid-column-responsive.png" />
</a>    


### Setting up column prompts
`Grid\Column::help` parameters.
 - $help `string` prompt content
 - $style `string` prompt window background color, support "primary", "success", "danger", "purple" and so on.
 - $placement `string` prompts the window position, supports "top", "left", "right", and "bottom"

<a href="{{public}}/assets/img/screenshots/grid-column-help.png" target="_blank">
    <img class="img" src="{{public}}/assets/img/screenshots/grid-column-help.png" />
</a>

```php
$grid->column('id')->help('tip');
```

### Set column search

The `Grid\Column::filter` method allows you to set a filter for a column, which makes it easy to filter the data table by that column. Refer to [column filter](model-grid-column-filter.md) for the detailed usage.

<a href="{{public}}/assets/img/screenshots/grid-column-filter.png" target="_blank">
    <img class="img" src="{{public}}/assets/img/screenshots/grid-column-filter.png" />
</a>


### Extended column function

Column methods can be extended by the `Grid\Column::macro` method.

Add the following code to `app/Admin/bootstrap.php`:

```php
use Dcat\Admin\Grid;

// $value is the value of the current field
// $p1、$p2 are custom parameters
Grid\Column::macro('myHeader', function ($value, $p1, $p2 = null) {
    // MyHeader needs to implement the Illuminate\Contracts\Support\Renderable interface
    // Of course, you can also pass a string here
    return $this->addHeader(new MyHeader($this, $p1, $p2));
});
```

The `MyHeader` class
```php
use Dcat\Admin\Grid\Column;
use Illuminate\Contracts\Support\Renderable;

class MyHeader implements Renderable
{
    public function __construct(Column $column, $p1, $p2)
    {
        ...
    }
    
    public function render()
    {
        ...
    }
}
```

use

```php
$grid->column('user')->myHeader($p1, $p2);

$grid->column('first_name')->myHeader($p1);
```


