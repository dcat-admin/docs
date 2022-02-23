# Model tree

This feature implements a tree component that can be dragged and dropped to achieve data hierarchy, sorting, and other operations, and here is the basic Usage.

![](https://cdn.learnku.com/uploads/images/202004/26/38389/RfWVwRHMs7.png!large)


## Table structures and models
To use `model-tree`, follow the agreed table structure:

> {tip} The `parent_id` field must default to `0`!!!!

```sql
CREATE TABLE `demo_categories` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `parent_id` int(11) NOT NULL DEFAULT '0',
  `order` int(11) NOT NULL DEFAULT '0',
  `title` varchar(50) COLLATE utf8_unicode_ci NOT NULL,
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci
```
The above table structure has three required fields inside `parent_id`, `order`, `title`, the other fields are not required.

The corresponding models are`app/Models/Category.php`:
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
    
    // Parent ID field name, default value is parent_id
    protected $parentColumn = 'pid';
    
    // Sort by field name, default value is order
    protected $orderColumn = 'sort';
    
    // TITLE field name, default value is title
    protected $titleColumn = 'name';
    
    // Since v2.1.6-beta，Defining the depthColumn property will save the current row's hierarchy in the data table
    protected $depthColumn = 'depth';

}
```

## Methods of use
Then it's time to use `model-tree` on the page:
```php
<?php

namespace App\Admin\Controllers\Demo;

use App\Models\Category;
use Dcat\Admin\Layout\Row;
use Dcat\Admin\Layout\Content;
use Dcat\Admin\Tree;
use Dcat\Admin\Controllers\AdminController;

class CategoryController extends AdminController
{
    public function index(Content $content)
    {
        return $content->header('树状模型')
            ->body(function (Row $row) {
                $tree = new Tree(new Category);
                
                $row->column(12, $tree);
            });
    }
}
```
The display of row data can be modified in the following ways：
```php
$tree = new Tree(new Category);

$tree->branch(function ($branch) {
    $src = config('admin.upload.host') . '/' . $branch['logo'] ;
    $logo = "<img src='$src' style='max-width:30px;max-height:30px' class='img'/>";

    return "{$branch['id']} - {$branch['title']} $logo";
});
```
The string type data returned in the callback function is what is displayed for each row in the tree component, and the `$branch` parameter is an array of data for the current row.

### Modify Model Lookup Criteria

If you want to modify the model's query, use the following
```php
$tree->query(function ($model) {
    return $model->where('type', 1);
});
```

### Limit the maximum number of levels

Default `5`

```php
$tree->maxDepth(3);
```

## Custom line operations

```php
use Dcat\Admin\Tree;

$tree->actions(function (Tree\Actions $actions) {
    if ($actions->row->id > 5) {
        $actions->disableDelete(); // Disable the delete button.
    }

    // Add a new action
    $actions->append(...);
});

// Batch add action
$tree->actions([
    new Action1(),
    "<div>...</div>",
    ...
]);
```

Customize complex actions, refer to [model-tree action](action-tree.md#row-action).

## Customize toolbar buttons

Please refer to the document [model-tree action](action-tree.md).




