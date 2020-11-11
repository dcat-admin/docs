# Display and expansion of columns


The datagrid has a number of built-in methods for manipulating columns, which makes it very flexible to manipulate column data.



## Display different components according to conditions
There are situations where we need to decide whether to use a display function of a column based on certain conditions.
> {tip} Note that `GridColumn::if` is only valid for functions related to the display of columns, other methods such as operations related to table headers do not work with this method!

```php
$grid->column('config')
    ->if(function ($column) {
        // Gets the value of other fields in the current row
        $username = $this->username;
        
    	// $column->getValue() is the value of the current field
        return $column->getValue();
    })
    ->display($view)
    ->copyable()
    ->else()
    ->emptyString();
```
The above is equivalent to
```php
$grid->column('config')
    ->if(function ($column) {
        return $column->getValue();
    })
    ->then(function (Grid\Column $column) {
        $column->display($view)->copyable();
    })
    ->else(function (Grid\Column $column) {
        $column->emptyString();
    });
```

Support for multiple `if`s
```php
$grid->column('config')
    ->if(...)
    ->then(...)
    ->else(...)
    
    ->if(...)
    ->then(...)
    ->else(...);
```




## Column display

`model-grid` has several built-in methods to help you extend the column functionality

### display

The `Dcat\Admin\Grid\Column` object has a built-in `display()` method to handle the current value via an incoming callback function
```php
$grid->column('title')->display(function ($title) {

    return "<span style='color:blue'>$title</span>";
    
});
```
The data can be manipulated in any way in the incoming anonymous function, and in addition the anonymous function binds the data of the current row as a parent object that can be called in the function:
```php
$grid->column('first_name');

$grid->column('last_name');

// Non-existent `full_name` field
$grid->column('full_name')->display(function () {
    return $this->first_name . ' ' . $this->last_name;
});
```

### Set the HTML properties of the column
Set the `html` attribute of the column
```php
$grid->column('name')->setAttributes(['style' => 'font-size:14px']);
```

### Column view
The `view` method can introduce a view file
```php
$grid->column('content')->view('admin.fields.content');
```

Three variables are passed into the view by default:
 - `$model` current row data
 - `$name` field name
 - `$value` is the current value.
 
The template code is as follows:
```blade
<label>name：{{ $name }}</label>
<label>value：{{ $value }}</label>
<label>other fields：{{ $model->title }}</label>
```



### Pictures

```php
$grid->column('picture')->image();

//Setting up servers and width and height
$grid->column('picture')->image('http://xxx.com', 100, 100);

// Show multiple pictures
$grid->column('pictures')->display(function ($pictures) {
    
    return json_decode($pictures, true);
    
})->image('http://xxx.com', 100, 100);
```

### show label label

Supports all the colors built into the `Dcat\Admin\Color` class

```php
use Dcat\Admin\Admin;

$grid->column('name')->label();

// Set color, pass alias directly.
$grid->column('name')->label('danger');

// It can also be used like this
$grid->column('name')->label(Admin::color()->danger());

// You can also pass the color code directly.
$grid->column('name')->label('#222');
```

Set different colors for different values
```php
use Dcat\Admin\Admin;

$grid->column('state')->using([1 => 'unprocessed', 2 => 'processed', ...])->label([
    'default' => 'primary', // Set the default color, or default if not set.
    
	1 => 'primary',
	2 => 'danger',
	3 => 'success',
	4 => Admin::color()->info(),
]);
```

### Displaying badge tags

Supports all the colors built into the `Dcat\Admin\Color` class.

```php
$grid->column('name')->badge();

// Set color, pass alias directly.
$grid->column('name')->badge('danger');

// It can also be used like this
$grid->column('name')->badge(Admin::color()->danger());

// You can also pass the color code directly
$grid->column('name')->badge('#222');
```

Set different colors for different values
```php
use Dcat\Admin\Admin;

$grid->state->using([1 => 'unprocessed', 2 => 'processed', ...])->badge([
    'default' => 'primary', // Set the default color, or default if not set.
    
    1 => 'primary',
	2 => 'danger',
	3 => 'success',
	4 => Admin::color()->info(),
]);
```

<a name="bool"></a>
### Boolean display (bool)


This column is shown as `✓` and `✗`after converting it to a `bool` value

```php
$grid->column('approved')->bool();
```

You can also specify the display according to the value of this column, for example, fields with values `Y` and `N` indicate `true` and `false`.

```php
$grid->column('approved')->bool(['Y' => true, 'N' => false]);
```

result
![]({{public}}/assets/img/screenshots/column-bool.png)



### Dot prefix (dot)

Column text can be preceded by a colored dot using the `dot` method

```php
use Dcat\Admin\Admin;

$grid->column('state')
	->using([1 => 'unprocessed', 2 => 'processed', ...])
	->dot(
		[
			1 => 'primary',
			2 => 'danger',
			3 => 'success',
			4 => Admin::color()->info(),
		], 
	    'primary' // The second parameter is the default value.
	);
```

result

![]({{public}}/assets/img/screenshots/grid-column-dot.png)

<a name="expand"></a>
### Column expansion (expand)
The `expand` method hides the content and displays it on the next line of the table when the button is clicked.
```php
$grid->column('content')
    ->display('details') // Set the button name
    ->expand(function () {
        // Return the displayed details
        // This returns the contents of the content field and wraps it with a card.
        $card = new Card(null, $this->content);
    
        return "<div style='padding:10px 10px 0'>$card</div>";
    });
```
Buttons can also be set in the following ways
```php
$grid->column('content')->expand(function (Grid\Displayers\Expand $expand) {
    // Set the button name
    $expand->button('details');

    // Return the displayed details
    // This returns the contents of the content field and wraps it with a card.
    $card = new Card(null, $this->content);

    return "<div style='padding:10px 10px 0'>$card</div>";
});
```


#### asynchronous loading

> {tip}  For more specific usage of asynchronous loading, please refer to the document [Asynchronous Loading](lazy.md).

Define rendering classes that inherit `Dcat\Admin\Support\LazyRenderable`

```php
use App\Models\Post as PostModel;
use Dcat\Admin\Support\LazyRenderable;
use Dcat\Admin\Widgets\Table;

class Post extends LazyRenderable
{
    public function render()
    {
        // Get ID
        $id = $this->key;
        
        // Get other custom parameters
        $type = $this->post_type;

        $data = PostModel::where('user_id', $id)
            ->where('type', $type)
            ->get(['title', 'body', 'body', 'created_at'])
            ->toArray();

        $titles = [
            'User ID',
            'Title',
            'Body',
            'Created At',
        ];

        return Table::make($titles, $data);
    }
}
```

use
```php
$grid->post->display('View')->expand(Post::make(['post_type' => 1]));

// Allows return of asynchronously loaded class instances within closures starting from v1.6.0
$grid->post->expand(function () {
    // Allows return of asynchronously loaded class instances within than package

    return Post::make(['title' => $this->title]);
});
```

result

<a href="{{public}}/assets/img/screenshots/expand-lazy-render.gif" target="_blank">
	![]({{public}}/assets/img/screenshots/expand-lazy-render.gif)
</a>


#### Asynchronous Loading Tool Form

Define the [tools-form](widgets-form.md) class as follows

> {tip} Starting with `v1.7.0`, most components including [tools-form](widgets-form.md), [charts](widgets-charts.md) are supported for asynchronous rendering, see [load asynchronously](lazy.md) for details

```php
<?php

namespace App\Admin\Forms;

use Dcat\Admin\Contracts\LazyRenderable;
use Dcat\Admin\Traits\LazyWidget;
use Dcat\Admin\Widgets\Form;

class UserProfile extends Form implements LazyRenderable
{
    use LazyWidget;

    public function handle(array $input)
    {
        // Receive externally passed parameters
		$type = $this->payload['type'] ?? null;
        
        return $this->success('Save success');
    }

    public function form()
    {
        // Receive externally passed parameters
        $type = $this->payload['type'] ?? null;
        
        $this->text('name', trans('admin.name'))->required()->help('Nickname');
        $this->image('avatar', trans('admin.avatar'))->autoUpload();

        $this->password('old_password', trans('admin.old_password'));

        $this->password('password', trans('admin.password'))
            ->minLength(5)
            ->maxLength(20)
            ->customFormat(function ($v) {
                if ($v == $this->password) {
                    return;
                }

                return $v;
            })
            ->help('Please enter 5-20 characters');
        $this->password('password_confirmation', trans('admin.password_confirmation'))
            ->same('password')
            ->help('Please confirm your password.');
    }
}
```

use

```php
$grid->user->display('View')->expand(UserProfile::make(['type' => 1]));
```


<a name="modal"></a>
### Pop-up modal box (modal)
The `modal` method hides the content and displays it on the next line of the table when the button is clicked.
```php
$grid->column('content')
    ->display('See') // Set the button name
    ->modal(function ($modal) {
        // Set pop-up window TITLE
        $modal->title('TITLE '.$this->username);
    
        $card = new Card(null, $this->content);
    
        return "<div style='padding:10px 10px 0'>$card</div>";
    });

// You can also set the pop-up TITLE in this way
$grid->column('content')
    ->display('see') // Set the button name
    ->modal('Pop-up window TITLE', ...);
```


#### asynchronous loading

> {tip} For more specific usage of asynchronous loading, please refer to the document [Asynchronous Loading](lazy.md).   

Define rendering classes that inherit `Dcat\Admin\Support\LazyRenderable`

```php
use App\Models\Post as PostModel;
use Dcat\Admin\Support\LazyRenderable;
use Dcat\Admin\Widgets\Table;

class Post extends LazyRenderable
{
    public function render()
    {
        // Get ID
        $id = $this->key;
        
        // Get other custom parameters
        $type = $this->post_type;

        $data = PostModel::where('user_id', $id)
            ->where('type', $type)
            ->get(['title', 'body', 'body', 'created_at'])
            ->toArray();

        $titles = [
            'User ID',
            'Title',
            'Body',
            'Created At',
        ];

        return Table::make($titles, $data);
    }
}
```

use
```php
$grid->post->display('View')->modal('Post', Post::make(['post_type' => 2]));

// Allows return of asynchronously loaded class instances within closures starting from v1.6.0
$grid->post->modal(function ($modal) {
    $modal->title('Custom pop-up window TITLE');

    // Allows return of asynchronously loaded class instances within than package
    return Post::make(['title' => $this->title]);
});
```

result
<a href="{{public}}/assets/img/screenshots/modal-lazy-render.gif" target="_blank">
	![]({{public}}/assets/img/screenshots/modal-lazy-render.gif)
</a>



#### Asynchronous Loading Tool Form

Define the [tools-form](widgets-form.md) class as follows

> {tip} Starting with `v1.7.0`, most components including [tools-form](widgets-form.md), [charts](widgets-charts.md) are supported for asynchronous rendering, see [load asynchronously](lazy.md) for details

```php
<?php

namespace App\Admin\Forms;

use Dcat\Admin\Contracts\LazyRenderable;
use Dcat\Admin\Traits\LazyWidget;
use Dcat\Admin\Widgets\Form;

class UserProfile extends Form implements LazyRenderable
{
    use LazyWidget;

    public function handle(array $input)
    {
        // Receive externally passed parameters
		$type = $this->payload['type'] ?? null;
        
        return $this->success('Save success');
    }

    public function form()
    {
        // Receive externally passed parameters
        $type = $this->payload['type'] ?? null;
        
        $this->text('name', trans('admin.name'))->required()->help('User nickname');
        $this->image('avatar', trans('admin.avatar'))->autoUpload();

        $this->password('old_password', trans('admin.old_password'));

        $this->password('password', trans('admin.password'))
            ->minLength(5)
            ->maxLength(20)
            ->customFormat(function ($v) {
                if ($v == $this->password) {
                    return;
                }

                return $v;
            })
            ->help('Please enter 5-20 characters');
        $this->password('password_confirmation', trans('admin.password_confirmation'))
            ->same('password')
            ->help('Please confirm your password.');
    }
}
```

use

```php
$grid->user->display('View')->modal(UserProfile::make(['type' => 1]));
```


### Progress Bar
`progressBar` progress bar
```php
$grid->rate->progressBar();

//Set the color, default `primary`, optional `danger`, `warning`, `info`, `primary`, `success`.
$grid->rate->progressBar('success');

// Set progress bar size and maximum value
$grid->rate->progressBar('success', 'sm', 100);
```

### showTreeInDialog
The `showTreeInDialog` method can present an array with hierarchies as a tree popup, such as permissions
```php
// Find out all the permissions data
$nodes = (new $permissionModel)->allNodes();

// Passing a two-dimensional array (not hierarchical)
$grid->permissions->showTreeInDialog($nodes);

// You can also pass in closures.
$grid->permissions->showTreeInDialog(function (Grid\Displayers\DialogTree $tree) use (&$nodes, $roleModel) {
    // Set all nodes
    $tree->nodes($nodes);
    
    // Set the node data field name, default "id", "name", "parent_id"
    $tree->setIdColumn('id');
    $tree->setTitleColumn('title');
    $tree->setParentColumn('parent_id');

    // $this->roles can get the value of the current row's field.
    foreach (array_column($this->roles, 'slug') as $slug) {
        if ($roleModel::isAdministrator($slug)) {
            // Select all nodes
            $tree->checkAll();
        }
    }
});
```
![]({{public}}/assets/img/screenshots/grid-column-tree.png)

### Content mapping (using)
```php
$grid->status->using([0 => '未激活', 1 => '正常']);

// The second parameter is the default value
$grid->gender->using([1 => '男', 2 => '女'], '未知');
```

### String split into arrays
The `explode` method splits the string into arrays.
```php
$grid->tag->explode()->label();

// You can specify delimiter, default ",".
$grid->tag->explode('|')->label();
```

### prepend

The `prepend` method is used to insert content in front of a value of type `string` or `array`

```php
// When the value of a field is a string
$grid->email->prepend('mailto:');

// When the value of a field is an array
$grid->arr->prepend('first item');
```

As of `v1.2.5`, the `prepend` method allows the passing of closure parameters.
```php
$grid->email->prepend(function ($value, $original) {
    // $value is the current field value
    // $original is the original value of the current field as queried from the database.
    
    // Get other field values
    $username = $this->username;
    
    return "[{$username}]";
});
```

### append
The `append` method is used to insert content after a value of type `string` or `array`.

```php
// When the value of a field is a string
$grid->email->append('@gmail.com');

// When the value of a field is an array
$grid->arr->append('last item');
```

As of `v1.2.5`, the `append` method allows the passing of closure parameters.
```php
$grid->email->prepend(function ($value, $original) {
    // $value is the current field value
    // $original is the original value of the current field as queried from the database.
    
    // Get other field values
    $username = $this->username;
    
    return "[{$username}]";
});
```

### String or array capture (limit)

```php
// Display up to 50 characters.
$grid->column('content')s->limit(50, '...');

// Also supported if the field value is an array.
$grid->tags->limit(3);
```

### Column QR code (qrcode)
```php
$grid->website->qrcode(function () {
    return $this->url;
}, 200, 200);
```

### Copyable (copyable)
```php
$grid->website->copyable();
```


### orderable (orderable)

Field sortability can be enabled via `Column::orderable`, which requires `use ModelTree` or `use SortableTrait` in your model class, and needs to inherit the `Spatie\EloquentSortable\Sortable` interface.


#### SortableTrait

If your data is not hierarchically structured data, you can just use `use SortableTrait`, see [eloquent-sortable](https://github.com/spatie/eloquent-sortable) for more usage.

model
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Spatie\EloquentSortable\Sortable;
use Spatie\EloquentSortable\SortableTrait;

class MyModel extends Model implements Sortable
{
    use SortableTrait;

    protected $sortable = [
        // Set the sort field name
        'order_column_name' => 'order',
        // If or not to sort automatically on creation, this parameter is recommended to be set to true.
        'sort_when_creating' => true,
    ];
}
```

use
```php
$grid->model()->orderBy('order');

$grid->order->orderable();
```

#### ModelTree

If your data is hierarchical, you can simply use `use Model`, here is an example of permission model to demonstrate usage

> {tip} `ModelTree` actually inherits the functionality of [eloquent-sortable](https://github.com/spatie/eloquent-sortable) as well.

```php
<?php

namespace Dcat\Admin\Models;

use Dcat\Admin\Traits\HasDateTimeFormatter;
use Dcat\Admin\Traits\ModelTree;
use Spatie\EloquentSortable\Sortable;

class Permission extends Model implements Sortable
{
    use HasDateTimeFormatter,
        ModelTree {
            ModelTree::boot as treeBoot;
        }
        
    ...    
}        
```

use
```php
$grid->order->orderable();
```

result
<a href="{{public}}/assets/img/screenshots/grid-display-orderable.png" target="_blank">
    <img  src="{{public}}/assets/img/screenshots/grid-display-orderable.png" style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%">
</a>


### Link (link)

Display the field as a link

```php
// When the link method does not pass parameters, the link's `href` and `text` are the current values
$grid->column('homepage')->link();

// incoming closure
$grid->column('homepage')->link(function ($value) {
    return admin_url('users/'.$value);
});
```

<a name="action"></a>
### row actions (action)

> {tip} Before using this method, please read [custom-actions-row-actions](model-grid-actions.md).

This function displays a column as a button for a **row operation**, such as the following, which shows a marked and unmarked column operation.
After clicking the asterisk icon in this column, the background will switch the state of the field and the page icon will follow. The specific implementation code is as follows:


```php
<?php

namespace App\Admin\Extensions\Grid\RowAction;

use Dcat\Admin\Grid\RowAction;
use Illuminate\Http\Request;

class Star extends RowAction
{
    public function html()
    {
        $icon = ($this->row->{$this->getColumnName()}) ? 'fa-star' : 'fa-star-o';

        return <<<HTML
<i class="{$this->getElementClass()} fa {$icon}"></i>
HTML;
    }

    public function handle(Request $request)
    {
        try {
            $class = $request->class;
            $column = $request->column;
            $id = $this->getKey();

            $model = $class::find($id);
            $model->{$column} = (int) !$model->{$column};
            $model->save();

            return $this->response()->success("success")->refresh();
        } catch (\Exception $e) {
            return $this->response()->error($e->getMessage());
        }
    }

    public function parameters()
    {
        return [
            'class' => $this->modelClass(),
            'column' => $this->getColumnName(),
        ];
    }

    public function getColumnName()
    {
        return $this->column->getName();
    }

    public function modelClass()
    {
        return get_class($this->parent->model()->repository()->eloquent());
    }
}
```

use

```php
protected function grid()
{
    $grid = new Grid(new $this->model());

    $grid->column('star', '-')->action(Star::class);  // here
    $grid->column('id', 'ID')->sortable()->bold();
    
    return $grid;
}
```

result

<a href="{{public}}/assets/img/screenshots/action-star.png" target="_blank">
    <img  src="{{public}}/assets/img/screenshots/action-star.png" style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="140px">
</a>


## Ways to help

### String manipulation
If the current output data is a string, the method of `Illuminate\Support\Str` can be called by chaining methods.

For example, there is a column that displays the string value of the `title` field as follows:

```php
$grid->title();
```

Call the `Str::title()` method on top of the output string in the `title` column
```php
$grid->title()->title();
```
After calling the method, the output is still a string, so you can continue to call the `Illuminate\Support\Str` method:
```php
$grid->title()->title()->ucfirst();

$grid->title()->title()->ucfirst()->substr(1, 10);

```

### Array manipulation
You can directly chain call the `Illuminate\Support\Collection` method if the output from the front is an array.

For example, the `tags` column is an array of data retrieved from a one-to-many relationship:
```php
$grid->tags();

array (
  0 => 
  array (
    'id' => '16',
    'name' => 'php',
    'created_at' => '2016-11-13 14:03:03',
    'updated_at' => '2016-12-25 04:29:35',
    
  ),
  1 => 
  array (
    'id' => '17',
    'name' => 'python',
    'created_at' => '2016-11-13 14:03:09',
    'updated_at' => '2016-12-25 04:30:27',
  ),
)

```

Call the `Collection::puck()` method to retrieve the `name` column from the array
```php
$grid->tags()->pluck('name');

array (
    0 => 'php',
    1 => 'python',
  ),

```
After the `name` column is extracted, the output is still an array, and the method of `Illuminate\Support\Collection` can still be called

```php
$grid->tags()->pluck('name')->map('ucwords');

array (
    0 => 'Php',
    1 => 'Python',
  ),
```
Output an array as a string
```php
$grid->tags()->pluck('name')->map('ucwords')->implode('-');
```

Result:
```
"Php-Python"
```

### Mixed use

In the above two types of method calls, as long as the output from the previous step is a value of a definite type, the method corresponding to the type can be called, so it can be used flexibly in combination.

For example, the `images` field is a JSON-format string type that stores an array of multiple image addresses:
```php

$grid->images();

// "['foo.jpg', 'bar.png']"

// Chained method calls to display multiple graphs
$grid->images()->display(function ($images) {
    return json_decode($images, true);
    
})->map(function ($path) {
    return 'http://localhost/images/'. $path;
    
})->image();

```


## Expanding the display of columns

The column functionality can be extended in two ways, the first being through anonymous functions.
> The IDE will not auto-complete after the expanded column method, so you can generate an IDE hint file with `php artisan admin: ide-helper`.

### anonymous function
Add the following code to `app/Admin/bootstrap.php`:
```php
use Dcat\Admin\Grid\Column;

// The second parameter is a custom parameter
Column::extend('color', function ($value, $color) {
    return "<span style='color: $color'>$value</span>";
});
```
Then use this extension in `model-grid`:
```php
$grid->title()->color('#ccc');
```

### Extensions
If the column display logic is complex, it can be implemented by extending the class.

extension class `app/Admin/Extensions/Popover.php`:
```php
<?php

namespace App\Admin\Extensions;

use Dcat\Admin\Admin;
use Dcat\Admin\Grid\Displayers\AbstractDisplayer;

class Popover extends AbstractDisplayer
{
    public function display($placement = 'left')
    {
        Admin::script("$('[data-toggle=\"popover\"]').popover()");

        return <<<EOT
<button type="button"
    class="btn btn-secondary"
    title="popover"
    data-container="body"
    data-toggle="popover"
    data-placement="$placement"
    data-content="{$this->value}"
    >
  pop-up
</button>
EOT;

    }
}
```
Then register the extension class in `app/Admin/bootstrap.php`:
```php
use Dcat\Admin\Grid\Column;
use App\Admin\Extensions\Popover;

Column::extend('popover', Popover::class);
```
Then it can be used in `model-grid`:
```php
$grid->desciption()->popover('right');
```

### Specify the listing
In addition to the above two ways of extending the column functionality, we can also extend the column functionality by specifying the column name

Add the following code to `app/Admin/bootstrap.php`:
```php
use Dcat\Admin\Grid\Column;

// This extension method is equivalent to the
// $grid->title()->display(function ($value) {
//    return "<span style='color:red'>$value</span>"
// });
Column::define('title', function ($value) {
    return "<span style='color:red'>$value</span>"
});

// This extension method is equivalent to the
// $grid->status()->switch();
Column::define('status', Dcat\Admin\Grid\Displayers\SwitchDisplay::class);
```
The above extensions are then automatically used in the `title` and `status` fields in `model-grid`:
```php
$grid->title();
$grid->status();
```

