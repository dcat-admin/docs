# Basic use of tables


## Simple Example
The `Dcat\Admin\Grid` class is used to generate a table based on the data model, starting with an example where there is a `movies` table in the database

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

The corresponding data model is `App\Admin\Repositories\Movie` and the corresponding data warehouse is `App\Admin\Repositories\Movie` with the following repository code.

> {tip} If your data comes from `MySQL`, then `repository` is not necessary, you can also just use `Model`.

```php
<?php

namespace App\Admin\Repositories;

use Dcat\Admin\Repositories\EloquentRepository;
use App\Models\Movie as MovieModel;

class Movie extends EloquentRepository
{
    protected $eloquentClass = MovieModel::class;
    
    /**
     * Set the fields to be queried in the table, and query all fields by default.
     * 
     * @return array
     */
    public function getGridColumns(){
        return ['id', 'title', 'director', 'rate', ...];
    }
}
```

The following code generates the data table for table `movies`：

```php
<?php

namespace App\Admin\Controllers;

use App\Admin\Repositories\Movie;
use Dcat\Admin\Grid;
use Dcat\Admin\Controllers\AdminController;

class MovieController extends AdminController
{
    protected function grid()
    {
        return Grid::make(new Movie(), function (Grid $grid) {
            // The first column displays the id field and sets this column as a rankable sequence.
            $grid->column('id', 'ID')->sortable();
            
            // The second column shows the title field, since the name of the title field conflicts with the title method of the Grid object, so use the column() method of the Grid instead.
            $grid->column('title');
            
            // The third column will display the director field and the contents of this column can be set to the user name in the users table by displaying($callback)
            $grid->column('director')->display(function($userId) {
                return User::find($userId)->name;
            });
            
            // The fourth column is shown as the describe field
            $grid->column('describe');
            
            // The fifth column is shown as the rate field
            $grid->column('rate');
            
            // The sixth column displays the released field, which is formatted to display the output via the display($callback) method.
            $grid->column('released', 'shown?')->display(function ($released) {
                return $released ? 'yes' : 'no';
            });
            
            // The following columns are displayed for the three time fields
            $grid->column('release_at');
            $grid->column('created_at');
            $grid->column('updated_at');
            
            // The filter($callback) method is used to set the table's simple search box.
            $grid->filter(function ($filter) {
                // Set the range query for the created_at field
                $filter->between('created_at', 'Created Time')->datetime();
            });
        });
    }
}
```

## Table display mode

### table_collapse

Starting in this release, the default table layout will be in `table_collapse` mode, with the following result

<a href="https://cdn.learnku.com/uploads/images/202007/24/38389/4bCfBdtvq5.png!large" target="_blank">
    <img class="img" src="https://cdn.learnku.com/uploads/images/202007/24/38389/4bCfBdtvq5.png!large" />
</a>
<a href="https://cdn.learnku.com/uploads/images/202007/24/38389/35KJXfVXib.png!large" target="_blank">
    <img class="img" src="https://cdn.learnku.com/uploads/images/202007/24/38389/35KJXfVXib.png!large" />
</a>    

If you want to switch back to the old table layout, you can add to `app/Admin/bootstrap.php` the following

```php
Grid::resolving(function (Grid $grid) {
    $grid->tableCollapse(false);
});
```

### Border mode

You can make the table display a border by `withBorder` method.

```php
$grid->withBorder();
```

result
![](https://cdn.learnku.com/uploads/images/202004/26/38389/lKTZe0jwGg.png!large)



Disable border mode
```php
$grid->withBorder(false);
```


## Basic usage

### Adding columns (column)

```php
// Add single column
$grid->column('username', 'Name');

// Add multiple columns
$grid->columns('email', 'username' ...);
```

### Modify source data
```php
$grid->model()->where('id', '>', 100);

$grid->model()->orderBy('profile.age');

// Recycle bin data
$grid->model()->onlyTrashed();

...
```

It is also possible to use the following

```php
protected function grid()
{
    return Grid::make(Model::with('...')->where(...), function (Grid $grid) {
        ...
    });
}
```

Other query methods can be found in the `eloquent` query method



### Modify display output (display)


```php
$grid->column('text')->display(function($text) {
    return str_limit($text, 30, '...');
});

// Allows mixing of multiple "display" methods
$grid->column('name')->display(function ($name) {
     return "<b>$name</b>";
 })->display(function ($name) {
    return "<span class='label'>$name</span>";
});

$grid->column('email')->display(function ($email) {
    return "mailto:$email";
});

// You can write strings directly
$grid->column('username')->display('...');

// Adding non-existent fields
$grid->column('column_not_in_table')->display(function () {
    return 'blablabla....'.$this->id;
});
```

### Display number (number)

The `number` method allows you to add a row number column starting with `1` to the table.

```php
$grid->number();
```

### Set name (setName)

When there are multiple `Grid` tables on the page, you need to set different names for the table, otherwise some of the functions may conflict!

```php
$grid->setName('name1');
```

### Get current row data (row)

The `display()` method receives an anonymous function that binds the data object of the current row, in which other field data of the current row can be called.

```php
$grid->column('first_name');
$grid->column('last_name');

// Non-existent field columns
$grid->column('full_name')->display(function () {
    return $this->first_name.' '.$this->last_name;
});
```


<a name="outline"></a>
### Set toolbar button styles

The toolbar buttons show `outline` mode by default, with the following result


Usage
```php
$grid->toolsWithOutline();

// Outline Surpressed
$grid->toolsWithOutline(false);
```

result
![](https://cdn.learnku.com/uploads/images/202005/23/38389/hKWC1crYHw.png!large)

Result after disabling `outline`:

![](https://cdn.learnku.com/uploads/images/202005/23/38389/aaMdymSxoY.png!large)


If you don't want a button to use `outline` mode, you can add `disable-outline` to the button's `class` attribute
```php
$grid->tools('<a class="btn btn-primary disable-outline">测试按钮</a>');
```

### Set create button

This feature is enabled by default
```php
// Disable Create Button
$grid->disableCreateButton();
// show
$grid->showCreateButton();
```

#### opens a pop-up window to create a form

This feature is not enabled by default

```php
$grid->enableDialogCreate();

// Set popup width and height, default value is '700px', '670px'.
$grid->setDialogFormDimensions('50%', '50%');
```

### Modify the routing of the Create and Update buttons (setResource)

Set the route prefix for the Modify Create and Update buttons.

```php
$grid->setResource('auth/users');
```


### Set the query filter

This feature is enabled by default.

```php
// Disable
$grid->disableFilter();
// Display
$grid->showFilter();

// Disable filter button
$grid->disableFilterButton();
// Display filter button
$grid->showFilterButton();
```


### line selector (rowSelector)
```php
// Disable
$grid->disableRowSelector();
// show
$grid->showRowSelector();
```

#### Set the title field of the selected middle row
If not set, the default is ``name``, ``title``, ``username``.
```php
$grid->column('full_name');
$grid->column('age');

...

$grid->rowSelector()->titleColumn('full_name');
```

#### Set the ID field of the selected middle row.
Set the fields to be saved when selected, default is the Data Table Primary Key (id) field.
```php
$grid->column('new_id');

...

$grid->rowSelector()->idColumn('new_id');
```

#### Set the checkbox selection box color
Default `primary`, support: `default`, `primary`, `success`, `info`, `danger`, `purple`, `inverse`.
```php
$grid->rowSelector()->style('success');
```

#### Click anywhere in the current row to select
This feature is not enabled by default
```php
$grid->rowSelector()->click();
```

#### Set the background color of selected rows
```php
use Dcat\Admin\Admin;

$grid->rowSelector()->background(Admin::color()->dark20());
```

#### set default check (check)

> Since `v2.0.21`

The ``check`` method allows you to set the default selected row, which accepts an ``array`` type or an ``anonymous function`` argument

```php
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

#### Disable changing the check state (disable)

> Since `v2.0.21`

The `disable` method allows you to set the row whose selected state is disabled, and accepts an `array` type or an `anonymous function` argument

```php
// disable row 1/3/5 from changing selection status
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

### Set row action buttons
```php
// Disable
$grid->disableActions();
// Show
$grid->showActions();

// Disable details button
$grid->disableViewButton();
// Show details button
$grid->showViewButton();

// Disable Edit Button
$grid->disableEditButton();
// Show edit button
$grid->showEditButton();

// Disable Quick Edit Button
$grid->disableQuickEditButton();
// Show quick edit buttons
$grid->showQuickEditButton();

// Set the width and height of the popup window, default value is '700px', '670px'
$grid->setDialogFormDimensions('50%', '50%');


// Disable the delete button
$grid->disableDeleteButton();
// Show delete button
$grid->showDeleteButton();

```

### Set the batchActions button (batchActions)
```php
// Disable
$grid->disableBatchActions();
// show
$grid->showBatchActions();

// 禁用批量删除按钮
$grid->disableBatchDelete();
// 显示批量删除按钮
$grid->showBatchDelete();
```

### Setting the toolbar (toolbar)
```php
// Disable
$grid->disableToolbar();
// show
$grid->showToolbar();
```

### Setting the refresh button (refresh)
```php
// Disable
$grid->disableRefreshButton();
// show
$grid->showRefreshButton();
```

### Setting up pagination (paginate)
```php
// Disable
$grid->disablePagination();
// show
$grid->showPagination();
```

#### Simple Pagination (simplePaginate)

Enabling the `simplePaginate` function will use the [simplePaginate](https://laravel.com/docs/8.x/pagination#simple-pagination) function of `Laravel` for pagination, which can greatly improve the page response speed when the data volume is large. However, it is important to note that the **total rows** of the data table will not be queried after using this feature.

```php
// Enable
$grid->simplePaginate();

// Disable
$grid->simplePaginate(false);
```


#### Set the number of rows per page

```php
// Default is 20 items per page
$grid->paginate(15);
```

#### Set pagination selector options
```php
$grid->perPages([10, 20, 30, 40, 50]);

// Disable pagination selector
$grid->disablePerPages();
```

### addTableClass


The `css` style can be added to table `table` through `addTableClass`.

```php
$grid->addTableClass(['class1', 'class2']);
```

### Set table text centering (text-center)

```php
$grid->addTableClass(['table-text-center']);
```

### Show horizontal scrollbar (scrollbarX)

Show the horizontal scrollbar of the table, which is not displayed by default

```php
// Enable
$grid->scrollbarX();

// Disable
$grid->scrollbarX(false);
```

### Setting the form outer container
```php
 // Change the table outer container
$grid->wrap(function (Renderable $view) {
    $tab = Tab::make();
    
    $tab->add('example', $view);
    $tab->add('code', $this->code(), true);

    return $tab;
});
```


## Relationship model

Reference [table-relationship](model-grid-relationship.md)
