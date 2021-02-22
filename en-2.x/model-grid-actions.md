# Use and extension of rows

### Enable or disable the default action button

By default, `model-grid` has four line operations `edit`, `quick edit`, `delete` and `detail`, which can be turned off in the following way.

```php
use Dcat\Admin\Grid;

 $grid->actions(function (Grid\Displayers\Actions $actions) {
    $actions->disableDelete();
    $actions->disableEdit();
    $actions->disableQuickEdit();
    $actions->disableView();
});

// Buttons can also be enabled or disabled in the following ways
$grid->disableDeleteButton();
$grid->disableEditButton();
$grid->disableQuickEditButton();
$grid->disableViewButton();
```

### Toggle line operation button display mode

The global default line action button display method can be configured through the configuration parameter `admin.grid.grid_action_class` parameter, which currently supports the following two methods of displaying line action buttons:

- `Dcat\Admin\Grid\Displayers\DropdownActions` Drop-down menu
- `Dcat\Admin\Grid\Displayers\Actions` Icon display mode
- `Dcat\Admin\Grid\Displayers\ContextMenuActions` Right mouse button to show drop-down menu

```php
    ...

    'grid' => [

        /*
        |--------------------------------------------------------------------------
        | The global Grid action display class.
        |--------------------------------------------------------------------------
        */
        'grid_action_class' => Dcat\Admin\Grid\Displayers\DropdownActions::class,
    ],
    
    ...
```

Switching the display method in the controller
```php
use Dcat\Admin\Grid;

public function grid()
{
    return Grid(new Model(), function (Grid $grid) {
        $grid->setActionClass(Grid\Displayers\Actions::class);
        
        ...
    });
}
```

### Get the row number (index)

The number is counted from ``0``.

```php
// Use in the display callback
$grid->column('number')->display(function () {
    return $this->_index + 1;
});


// used in a row action
$grid->actions(function ($actions) {
    $index = $this->_index;
    
    ...
});
```

### Get current row data

The data for the current row can be retrieved through the incoming `$actions` parameter.
```php
use Dcat\Admin\Grid;

$grid->actions(function (Grid\Displayers\Actions $actions) {
    // Array of data for the current row
    $rowArray = $actions->row->toArray();
    
    // Data for a field in the current row
    $email = $actions->row->email;
    
    // Get the current row primary key value
    $id = $actions->getKey();
});
```

### Adding custom buttons

If there is a custom action button, it can be added as follows:

```php
use Dcat\Admin\Grid;

$grid->actions(function (Grid\Displayers\Actions $actions) {
    // append an action
    $actions->append('<a href=""><i class="fa fa-eye"></i></a>');

    // append a complex action
    $actions->append(new Copy());

    // prepend an action
    $actions->prepend('<a href=""><i class="fa fa-paper-plane"></i></a>');
}
```


### Adding complex operation buttons

For more complex operations, consider the following approach.


Define the row action class to inherit `Dcat\Admin\RowAction` first

> {tip} For more detailed usage of the action class, please refer to [Action Basic Use](action.md) and [Data Grid Action](action-grid.md).

```php
<?php

namespace App\Admin\Extensions;

use Dcat\Admin\Grid\RowAction;

class CheckRow extends RowAction
{
    /**
     * Return field TITLE
     * 
     * @return string
     */
    public function title()
    {
        return 'Check row';
    }
    
    /**
     * Adding JS
     * 
     * @return string
     */
    protected function script()
    {
        return <<<JS
$('.grid-check-row').on('click', function () {
    
    // Your code.
    console.log($(this).data('id'));
    
});
JS;
    }

    public function html()
    {
        // Get the current row data ID
        $id = $this->getKey();
        
        // Get the username of the current row data
        $username = $this->row->username;
        
        // Here you need to add a class, which corresponds to the script method above.
        $this->setHtmlAttribute(['data-id' => $id, 'email' => $username, 'class' => 'grid-check-row']);

        return parent::html();
    }
}
```
Then add the action:
```php
$grid->actions([new CheckRow()]);

// It can also be added in this way
$grid->actions(function (Grid\Displayers\Actions $actions) {
    $actions->append(new CheckRow());
}
```

### Action buttons need to interact with the API

If your action class needs to interact with the background interface, you can add the `handle` method to your action class, which makes it easy to handle all the logic in the same class!

> {tip} For more detailed usage of the action class, please refer to [Action Basic Use](action.md) and [Data Grid Action](action-grid.md).

```php
<?php

namespace App\Admin\RowActions;

use Dcat\Admin\Grid\RowAction;
use Illuminate\Http\Request;
use Illuminate\Database\Eloquent\Model;

class Copy extends RowAction
{
    protected $model;

    public function __construct(string $model = null)
    {
        $this->model = $model;
    }

    /**
     * TITLE
     *
     * @return string
     */
    public function title()
    {
        return 'Copy';
    }

    /**
     * Set the confirmation popup message, if a null value is returned, the popup window will not show
     *
     * Allows return of string or array types
     *
     * @return array|string|void
     */
    public function confirm()
    {
        return [
            // Confirmation pop-up title
            "Are you sure you want to copy this line of data?？",
            // confirm pop-up window content
            $this->row->username,
        ];
    }

    /**
     * Processing of requests
     *
     * @param Request $request
     *
     * @return \Dcat\Admin\Actions\Response
     */
    public function handle(Request $request)
    {
        // Get the current line ID
        $id = $this->getKey();

        // Gets the parameters passed by the parameters method
        $username = $request->get('username');
        $model = $request->get('model');

        // Copy data
        $model::find($id)->replicate()->save();

        // Returns response results and refreshes the page
        return $this->response()->success("Replication success: [{$username}]")->refresh();
    }

    /**
     * Set the data to be POSTed to the interface.
     *
     * @return array
     */
    public function parameters()
    {
        return [
            // Send the current line's username field data to the interface
            'username' => $this->row->username,
            // Passing the model class name to the interface
            'model' => $this->model,
        ];
    }
}

```

use
```php
use App\Models\User;

$grid->actions([new Copy(User::class)]);

// It can also be added in this way
$grid->actions(function (Grid\Displayers\Actions $actions) {
    $actions->append(new Copy(User::class));
}
```


## Form pop-up

Please refer to the documentation [tools-form - popups](widgets-form.md#modal).
