# Tree table

The tree table supports pagination and click-to-load functionality and is particularly suitable for displaying large amounts of data in a multi-level structure.

<a href="http://103.39.211.179:8080/admin/tree" target="_blank">
    <img class="img img-full" src="{{public}}/assets/img/screenshots/grid-tree.png">
</a>

### Table structures and models
To use a tree table, follow the agreed table structure:

> {tip} This table structure and model are fully compatible with <code>[model-tree](model-tree.md)</code> .

```sql
CREATE TABLE `demo_categories` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `parent_id` int(11) NOT NULL DEFAULT '0',
  `order` int(11) NOT NULL DEFAULT '0', // order 字段不是必须的，不设置也可以
  `title` varchar(50) COLLATE utf8_unicode_ci NOT NULL,
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci
```

The above table structure has three required fields inside `parent_id`, `title`, the other fields are not required.

The corresponding model is `app/Models/Category.php`:

> {tip} For ease of reading, the `Repository` code is no longer shown here.

```php
<?php

namespace App\Models\Demo;

use Dcat\Admin\Traits\ModelTree;
use Illuminate\Database\Eloquent\Model;

class Category extends Model
{
    use ModelTree;

    protected $table = 'demo_categories';
}
```

The names of the three fields `parent_id`, `order` and `title` in the table structure can also be modified:
```php
<?php

namespace App\Models\Demo;

use Dcat\Admin\Traits\ModelTree;
use Illuminate\Database\Eloquent\Model;

class Category extends Model
{
    use ModelTree;

    protected $table = 'demo_categories';
    
    protected $titleColumn = 'name';
    
    protected $orderColumn = 'sort';
    
    protected $parentColumn = 'pid';
}
```

If your data table does not require the `order` field to be sorted, add the following code to the model
```php
<?php

namespace App\Models\Demo;

use Dcat\Admin\Traits\ModelTree;
use Illuminate\Database\Eloquent\Model;

class Category extends Model
{
    use ModelTree;

    protected $table = 'demo_categories';
    
    // Disable the order field by returning a null value.
    public function getOrderColumn()
    {
        return null;
    }
}
```


### Use

The tree table function can be enabled by calling `Grid\Column::tree` method, after that, only the data of the top node will be queried by default.

```php
<?php

namespace App\Admin\Controllers\Demo;

use App\Models\Category;
use Dcat\Admin\Grid;
use Dcat\Admin\Controllers\AdminController;

class CategoryController extends AdminController
{
    protected function grid()
    {
        return Grid::make(new Category(), function (Grid $grid) {
            $grid->id('ID')->bold()->sortable();
            $grid->title->tree(); // Enable the tree table feature 
            $grid->order;
    
            $grid->created_at;
            $grid->updated_at->sortable();
            
            $grid->filter(function (Grid\Filter $filter) {
                $filter->like('slug');
                $filter->like('name');
                $filter->like('http_path');
            });
        });
    }
}
```

The above code executes `sql` as follows (by default, only data with `parent = 0` is queried):
```sql
select count(*) as aggregate from `demo_categories` where `parent_id` = 0

select * from `demo_categories` where `parent_id` = 0 order by `order` asc, `id` asc limit 20 offset 0
```


`Grid\Column::tree` Method parameters

+ `bool` `$showAll`  `false`  Whether to display all the nodes of the next level at once, the default pagination display
+ `bool` `$sortable` `true`   Whether to enable the sorting function

```php
// Disable paging and load all next level nodes at once.
$grid->title->tree(true);


// No need to sort the order field, pass false for the second argument
$grid->title->tree(false, false);
```


### orderable排序

The `orderable` sorting function relies on the <a href="https://github.com/spatie/eloquent-sortable" target="__blank">spatie/eloquent-sortable</a> component, which requires modifying the model:

```php
use Spatie\EloquentSortable\Sortable;

class Category extends Model implements Sortable
{
    use ModelTree;
    
    // Set the sort field, default order
    protected $orderColumn = 'sort';
}
```

Here's how to use Example

```php
class CategoryController extends AdminController
{
    protected function grid()
    {
        return Grid::make(new Category(), function (Grid $grid) {
            $grid->id('ID')->bold()->sortable();
            $grid->title->tree(); // Enable the tree table feature 
            $grid->order->orderable(); // Enable sorting function
    
            ...;
        });
    }
}
```

### About Data Search

If the search function is used in the tree table (`Grid::filter`, `Grid\Column::filter`, `Grid::quickSearch`), it <b>cancels</b> the operation of searching only the top-level data in order to allow the user to search for the desired data.

> {tip} Using search functions such as [query filters](model-grid-filters.md), [column filters](model-grid-column-filter.md), [quick-search](model-grid-quick-search.md) all <b>cancel</b> the operation of searching only the top-level data. with the exception of features such as [filter](model-grid-selector.md) and [scope query scope](model-grid-filters.md#scope).


For example, the following code enables quick search
```php
class CategoryController extends AdminController
{
    protected function grid()
    {
        return Grid::make(new Category(), function (Grid $grid) {
            $grid->id('ID')->bold()->sortable();
            $grid->title->tree(); // Enable the tree table feature 
            $grid->order->orderable(); // Enable sorting function
    
            $grid->quickSearch(['id', 'title']);
            ...;
        });
    }
}
```

and the user uses a quick search in the browser, then `sql` is generated as follows

```sql
select count(*) as aggregate from `demo_categories` where `id` like "%xxx%" or `title` like "%xxx%"

select * from `demo_categories` where `id` like "%xxx%" or `title` like "%xxx%" order by `order` asc, `id` asc limit 20 offset 0
```

### Difference from model tree function

The [model-tree](model-tree.md) can also be used to display multi-level structured data, and support drag-and-drop data hierarchy, sorting, and other operations, but does not support paging and click to load the function, can only load all the data at once.
Therefore, [model-tree](model-tree.md) is not suitable for displaying data with large amounts of data.


The tree table supports pagination and click to load, suitable for displaying large amounts of data multi-level structured data, but does not support the use of drag and drop to achieve the data hierarchy, sorting operations.
