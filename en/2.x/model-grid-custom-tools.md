# Toolbar

## Tool buttons

In the `model-grid` head of the default `batch delete` and `refresh` two operation tools, if there are more operation needs, the system provides the ability to customize the tool, the following Example add a gender category selection of the button group tool.


<a name="outline"></a>
### Set toolbar button styles

The toolbar buttons show `outline` mode by default, with the following result


usage
```php
$grid->toolsWithOutline();

// 禁止
$grid->toolsWithOutline(false);
```

result
<a href="{{public}}/assets/img/screenshots/outline.png" target="_blank">
    <img style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" src="{{public}}/assets/img/screenshots/outline.png">
</a>

Result after disabling `outline`:

<a href="{{public}}/assets/img/screenshots/n-outline.png" target="_blank">
    <img style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" src="{{public}}/assets/img/screenshots/n-outline.png">
</a>

If you don't want a button to use `outline` mode, you can add `disable-outline` to the button's `class` attribute.
```php
$grid->tools('<a class="btn btn-primary disable-outline">测试按钮</a>');
```

### Customize toolbar buttons

Define the tool class `app/Admin/Extensions/Tools/UserGender.php` to inherit the base class of the tool class `Dcat\Admin\GridToolsAbstractTool`:

```php
<?php

namespace App\Admin\Grid\Tools;

use Dcat\Admin\Admin;
use Dcat\Admin\Grid\Tools\AbstractTool;

class UserGender extends AbstractTool
{
    protected function script()
    {
        $url = request()->fullUrlWithQuery(['gender' => '_gender_']);

        return <<<JS
$('input:radio.user-gender').change(function () {
    var url = "$url".replace('_gender_', $(this).val());

    Dcat.reload(url);
});
JS;
    }

    public function render()
    {
        Admin::script($this->script());

        $options = [
            'all'   => 'All',
            'm'     => 'Male',
            'f'     => 'Female',
        ];

        return view('admin.tools.gender', compact('options'));
    }
}
```

The view `admin.tools.gender` file is `resources/views/admin/tools/gender.blade.php`:
```html
<div class="btn-group" data-toggle="buttons">
    @foreach($options as $option => $label)
    <label class="btn btn-default {{ request()->get('gender', 'all') == $option ? 'active' : '' }}">
        <input type="radio" class="user-gender" value="{{ $option }}">{{$label}}
    </label>
    @endforeach
</div>
```

Introduce this tool in the `Grid`.
```php
$grid->tools(new UserGender());
```

After receiving the `gender` parameter in the `model-grid` definition, you can do a data query:
```php
if (in_array($gender = request()->get('gender'), ['m', 'f'])) {
    $grid->model()->where('gender', $gender);
}
```

You can refer to the above to add your own tools.

### Advanced Usages

If your tool buttons need to interact with the backend API, they can be defined in the following manner:

> {tip} The `AbstractTool` class is a subclass of the `Dcat\Admin\Actions\Action` class, which is essentially a kind of action class, and for more detailed usage, please refer to [Action Class Basic Use](action.md).


```php
<?php

namespace App\Admin\Grid\Tools;

use Dcat\Admin\Grid\Tools\AbstractTool;
use Illuminate\Http\Request;

class SendMessage extends AbstractTool
{
    /**
     * Button style, default btn btn-white-waves-effect
     * 
     * @var string 
     */
    protected $style = 'btn btn-white waves-effect';
    
    
    /**
     * Button Text
     * 
     * @return string|void
     */
    public function title()
    {
        return 'send a reminder';
    }
    
    /**
     *  popup window confirmation. If not, return empty.
     * 
     * @return array|string|void
     */
    public function confirm()
    {
        // Show only TITLE
//        return 'Are you sure you want to send a new reminder message?？';
        
        // Show TITLE and content
        return ['Are you sure you want to send a new reminder message?', 'Confirm the content of the message, if not, leave it blank'];
    }
    
    /**
     * Processing of requests
     * If this method is included in your class, clicking the button will automatically initiate an ajax request to the backend, and the request logic will be processed by this method
     * 
     * @param Request $request
     */
    public function handle(Request $request)
    {
        // The logic of your code
        
        return $this->response()->success('send success')->refresh();
    }
    
    /**
     * Set request parameters
     * 
     * @return array|void
     */
    public function parameters()
    {
        return [
            
        ];
    }
}
```

use

```php
use App\Admin\Grid\Tools\SendMessage;

$grid->tools(new SendMessage());
```


### Adding Tools

The `Grid::tools` method allows passing parameters of type `string`, `array`, `AbstractTool` and `close`, as shown in the following demo.

```php
// pass a string
$grid->tools('<a class="btn btn-sm btn-default">Tool Button Test</a>');

// pass an array
$grid->tools([
	'<a class="btn btn-sm btn-default">Tool Button Test</a>',
	new UserGender(),
]);

// pass a closure
$grid->tools(function (Grid\Tools $tools) {
	$tools->append(new UserGender());
});
```

<a name="batch"></a>
## Batch operations

### Disable bulk delete

The system enables the bulk delete operation by default. if you want to disable the bulk delete operation:

```php
$grid->tools(function ($tools) {
    $tools->batch(function ($batch) {
        $batch->disableDelete();
    });
});

// Or this.
$grid->disableBatchDelete();

// or
$grid->batchActions(function (Grid\Tools\BatchActions $batch) {
	$batch->disableDelete();
});
```

### Custom batch operation

The following is a demonstration of the custom batch operation by extending a feature for bulk posting to articles:

Define the operation class `app/Admin/Extensions/Tools/ReleasePost.php` first, inherit `Dcat\Admin\Grid\BatchAction`:

> {tip} The `BatchAction` class is a subclass of the `Dcat\Admin\Actions\Action` class, which is essentially a kind of action class, and for more detailed Usage, please refer to [Action Class Basic Use](action.md).

```php
<?php

namespace App\Admin\Extensions\Tools;

use Dcat\Admin\Grid\BatchAction;
use Illuminate\Http\Request;

class ReleasePost extends BatchAction
{
    protected $action;

    // Note that the action constructor parameters must be given default values
    public function __construct($title = null, $action = 1)
    {
        $this->title = $title;
        $this->action = $action;
    }
    
    // Confirming pop-up messages
    public function confirm()
    {
        return 'Are you sure you want to publish the selected article?？';
    }
    
    // Processing of requests
    public function handle(Request $request)
    {
        // Get an array of selected article IDs
        $keys = $this->getKey();
        
        // Get request parameters
        $action = $request->get('action');
        
        foreach (Post::find($keys) as $post) {
            $post->released = $action;
            $post->save();
        }
        
        $message = $action ? 'Article publishing success' : 'Article unpublishing success';
        
        return $this->response()->success($message)->refresh();
    }
    
    // Set request parameters
    public function parameters()
    {
        return [
            'action' => $this->action,
        ];
    }
}
```
The implementation of the code does:
Sending a post request with a click action.
Pass the selected row data `id` to the backend interface as an array. The back-end interface gets the list of `id` and the state to be modified to update the data. Then the front end refreshes the page (pjax reload) and the `toastr` pop-up prompts for a success operation.

Add this batch operation feature to `model-grid`:
```php
$grid->batchActions([
	new ReleasePost('Post an article', 1),
	new ReleasePost('Article offline', 0)
]);  

// It could also read like this
$grid->batchActions(function ($batch) {
    $batch->add(new ReleasePost('Post an article', 1));
    $batch->add(new ReleasePost('Article offline', 0));
});

// or
$grid->tools(function ($tools) {
    $tools->batch(function ($batch) {
    	$batch->add(new ReleasePost('Post an article', 1));
    	$batch->add(new ReleasePost('Article offline', 0));
    });
});
```

#### gets the array of IDs selected by the checkboxes

Through the `getSelectedKeysScript` method, you can get an array of IDs selected by the checkbox, the Usages are as follows

> {tip} The `getSelectedKeysScript` method returns `js` code, which can only be used in `js` code

```php
<?php

namespace App\Admin\Extensions\Tools;

use Dcat\Admin\Grid\BatchAction;
use Illuminate\Http\Request;

class ReleasePost extends BatchAction
{
    protected $action;

    public function __construct($title, $action = 1)
    {
        $this->title = $title;
        $this->action = $action;
    }
    
    // confirmation pop-up messages
    public function confirm()
    {
        return 'Are you sure you want to publish the selected article？';
    }
    
    // Processing of requests
    public function handle(Request $request)
    {
        ...
    }
    
    /**
     * Set the callback function before the action initiates the request, return false to interrupt the request. 
     * 
     * @return string
     */
    public function actionScript(){
        $warning = __('No data selected!');

        return <<<JS
function (data, target, action) { 
    var key = {$this->getSelectedKeysScript()}

    if (key.length === 0) {
        Dcat.warning('{$warning}');
        return false;
    }

    // Set the primary key to the array of row IDs selected in the check box.
    action.options.key = key;
}
JS;
    }
}
```

### Form pop-up

Please refer to the documentation [tools-form - popups](widgets-form.md#modal).
