# Data soft delete


Refer to the `Laravel` documentation first to implement the model's [soft delete](https://learnku.com/docs/laravel/6.x/eloquent/5176#soft-deleting):

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Post extends Model
{
    use SoftDeletes;
}
```

This way, the data displayed in the `grid` list is all undeleted data.

```php
return Grid::make(new Post(), function (Grid $grid) {
	$grid->id('ID')->sortable();
	$grid->title('Title');
	$grid->created_at('Created at');
	$grid->updated_at('Updated at');
});
```

## Recycle Bin Entrance

Next, we need to add an entry point that will allow us to see the soft-deleted data, which can be done using `model-grid`'s [scope filter](model-grid-filters.md#scope).

```php
$grid->filter(function () {

    // Range filter, calling the model's `onlyTrashed` method to query out the soft-deleted data.
    $filter->scope('trashed', 'recycle bin')->onlyTrashed();

});
```

A `Recycle Bin` button will appear in the drop-down menu of the filter button in the table header, and clicking it will call the model's `onlyTrashed` method to look up the deleted data from the table, i.e., the data in the Recycle Bin.

<img style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="40%" src="{{public}}/assets/img/screenshots/trash-button.png">



## Row recovery operations

By following this method, we can add a recovery action to each row of data in the Recycle Bin for easy data recovery

Define the operation class `app/Admin/Actions/Post/Restore.php` first.

```php
<?php

namespace App\Admin\Actions\Post;

use Dcat\Admin\Grid\RowAction;
use Illuminate\Http\Request;

class Restore extends RowAction
{
    protected $title = 'Restore';
        
	protected $model;
	
	// Note that the construct method parameters must have default values
	public function __construct(string $model = null) 
	{
		$this->model = $model;
	}

    public function handle(Request $request)
    {
        $key = $this->getKey();
        $model = $request->get('model');
        
        $model::withTrashed()->findOrFail($key)->restore();

        return $this->response()->success('Restored')->refresh();
    }

    public function confirm()
    {
        return ['Are you sure?'];
    }
    
    public function parameters()
    {
        return [
            'model' => $this->model,    
		];
	}
}
```

Add to Row Action:

```php
use App\Models\Post;
use App\Admin\Actions\Post\Restore;

$grid->actions(function (Grid\Displayers\Actions $actions) {
    if (request('_scope_') == 'trashed') {
        $actions->append(new Restore(Post::class));
    }
});
```

## Batch recovery operations

Define the operation class `app/Admin/Actions/Post/BatchRestore.php` first:

```php
<?php

namespace App\Admin\Actions\Post;

use Dcat\Admin\Grid\BatchAction;
use Illuminate\Http\Request;

class BatchRestore extends BatchAction
{
    protected $title = 'Restore';
    
    protected $model;
    
    // Note that the construct method parameters must have default values
    public function __construct(string $model = null) 
    {
        $this->model = $model;
    }

    public function handle(Request $request)
    {
        $model = $request->get('model');
        
        foreach ((array) $this->getKey() as $key) {
			$model::withTrashed()->findOrFail($key)->restore();
		}
        
        return $this->response()->success('Restored')->refresh();
    }

    public function confirm()
    {
        return ['Are you sure？'];
    }
    
	public function parameters()
	{
		return [
			'model' => $this->model,    
		];
	}
}
```

Add to batch operation:

```php
use App\Models\Post;
use App\Admin\Actions\Post\BatchRestore;

$grid->batchActions(function (Grid\Tools\BatchActions $batch) {
	if (request('_scope_') == 'trashed') {
		$batch->add(new BatchRestore(Post::class));
	}
});
```
