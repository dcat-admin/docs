# Basic use of asynchronous loading

> {tip} The asynchronous loading feature supports **load on demand** of static resources, and currently all built-in **all components** support the use of asynchronous rendering in **anywhere** on the page.

The asynchronous loading feature allows for asynchronous rendering of whole or partial components of a page using `ajax` to improve page load efficiency (e.g. popups to load forms asynchronously).


## Basic Usage

The following is a simple example to demonstrate the use of the asynchronous load feature


First, define an asynchronous rendering class that inherits `Dcat\Admin\Support\LazyRenderable`

```php
<?php

namespace App\Admin\Renderable;

use App\Admin\Widgets\Charts\Bar;
use Dcat\Admin\Support\LazyRenderable;

class PostChart extends LazyRenderable
{
    public function render()
    {
    	// Getting externally passed parameters
    	$id = $this->id;
        
    	// Query Data Logic
    	$data = [...];
    	
    	// This can return built-in components, view files or HTML strings.
        return Bar::make($data);
	}
}
```

Then you need to pass the rendering class instance into the `Dcat\Admin\Widgets\Lazy` object in order to achieve the final asynchronous rendering effect.

```php
use App\Admin\Renderable\PostChart;
use Dcat\Admin\Widgets\Lazy;
use Dcat\Admin\Layout\Content;

public function index(Content $content)
{
    // Instantiate asynchronous rendering classes and pass custom parameters
    $chart = PostChart::make(['id' => ...]);
    
	return $content->body(Lazy::make($chart));
}
```

It can also be placed in the built-in components

> {tip} In the case of components such as `Dcat\Admin\Widgets\Card`, `Dcat\Admin\Widgets\Box`, `Dcat\Admin\Widgets\Modal`, `Dcat\Admin\Widgets\Tab`, `Dcat\Admin\Widgets\Tab` may be omitted. `Widgets\Lazy` component, directly passing rendering class instances

```php
use App\Admin\Renderable\PostChart;
use Dcat\Admin\Widgets\Card;
use Dcat\Admin\Layout\Content;

public function index(Content $content)
{
    $chart = PostChart::make(['id' => ...]);
    
    // The Card component supports passing LazyRenderable instances directly, bypassing the Lazy object.
	return $content->body(Card::make($chart));
}

// In the case of Modal, Box, etc., you can just skip Lazy.
use Dcat\Admin\Widgets\Modal;

$chart = PostChart::make(['id' => ...]);
 
$modal = Modal::make()
	->lg()
	->title('TITLE')
	->delay(300) // If the chart is rendered asynchronously, you need to set a delay time, otherwise the chart may render abnormally.
	->body($chart);
```

Of course, it can also be prevented in views or `HTML` code
```php
use App\Admin\Renderable\PostChart;
use Dcat\Admin\Widgets\Lazy;
use Dcat\Admin\Layout\Content;

public function index(Content $content)
{
    $chart = Lazy::make(PostChart::make(['id' => ...]));
    
	return $content->body(view('admin.xxx', ['chart' => $chart]));
}
```

result

<a href="https://cdn.learnku.com/uploads/images/202008/20/38389/Z1X46kZLtM.gif!large" target="_blank">
![](https://cdn.learnku.com/uploads/images/202008/20/38389/Z1X46kZLtM.gif!large)
</a>


### Dcat\Admin\Support\LazyRenderable

#### parameter passing (payload)

```php
use App\Admin\Renderable\PostChart;

PostChart::make(['key1' => '值', ...]);

// Can also be passed via the payload method
PostChart::make()->payload(['key1' => '值', ...]);
```

Get Parameters

```php
class PostChart extends LazyRenderable
{
    protected $title = ['#', 'TITLE', '内容'];
    
    public function render()
    {
    	// Getting externally passed parameters
    	$key1 = $this->key1;
    	$key2 = $this->key2;
        
    	...
	}
}
```


#### Loading JS and CSS

The asynchronous load feature also supports **static resources on demand** and is very easy to use.

```php
<?php

namespace App\Admin\Renderable;

use Dcat\Admin\Support\LazyRenderable;
use Dcat\Admin\Admin;

class CustomView extends LazyRenderable
{
    // Write the paths to the js and css files that need to be loaded here
    public static $js = [
    	'xxx/xxx1.js',
    	'xxx/xxx2.js',     
	];
    
    public static $css = [
        'xxx/xxx1.css',
		'xxx/xxx2.css', 
	];
    
    protected function addScript()
    {
        Admin::script(
            <<<JS
		console.log('The JS scripts are all loaded.~');
JS
        );
    }
    
    public function render()
    {
        // Adding your JS code
        $this->addScript();
        
     	return view('admin.custom', ['...']);   
	}
}
```

Template file code examples, be careful not to include tags such as `body` and `html` etc.

```HTML
<div id="custom" class="custom"><h2>asynchronous load function</h2></div>

<script>
Dcat.ready(function () {
    // JS The code can also be placed in the template file
    console.log('Template file execution js~');
});
</script>
```


### Dcat\Admin\Widgets\Lazy

#### onLoad

This method allows you to listen for asynchronous load completion events

```php
use App\Admin\Renderable\PostChart;
use Dcat\Admin\Widgets\Lazy;

$chart = Lazy::make(PostChart::make())->onLoad(
	<<<JS
console.log('Component rendering complete');	
JS
);
```

#### load

This method controls whether to render the asynchronous component immediately.

```php
use App\Admin\Renderable\PostChart;
use Dcat\Admin\Widgets\Lazy;
use Dcat\Admin\Admin;

$lazy = Lazy::make(PostChart::make())->load(false);

Admin::script(
	<<<JS
setTimeout(function () {
	// Asynchronous rendering event triggered automatically after 3 seconds
	{$lazy->getLoadScript()}
}, 3000);	
JS
);
```

#### JS events

```php
use App\Admin\Renderable\PostChart;
use Dcat\Admin\Widgets\Lazy;
use Dcat\Admin\Admin;

$lazy = Lazy::make(PostChart::make());

Admin::script(
	<<<JS
// Manually triggering asynchronous rendering events	
$('{$lazy->getElementSelector()}').trigger('lazy:load');

// Listening for rendering completion events
$('{$lazy->getElementSelector()}').on('lazy:loaded', function () {
	console.log('The component is rendered.')
});
JS
);
```


<a name="lazy-table"></a>
## Loading data tables asynchronously

If you need to load the data table asynchronously, you need to inherit `Dcat\Admin\Grid\LazyRenderable` when defining the rendering class

```php
<?php

namespace App\Admin\Renderable;

use Dcat\Admin\Grid;
use Dcat\Admin\Grid\LazyRenderable;
use Dcat\Admin\Models\Administrator;

class UserTable extends LazyRenderable
{
    public function grid(): Grid
    {
        return Grid::make(new Administrator(), function (Grid $grid) {
            $grid->column('id');
            $grid->column('username');
            $grid->column('name');
            $grid->column('created_at');
            $grid->column('updated_at');

            $grid->quickSearch(['id', 'username', 'name']);

            $grid->paginate(10);
            $grid->disableActions();

            $grid->filter(function (Grid\Filter $filter) {
                $filter->like('username')->width(4);
                $filter->like('name')->width(4);
            });
        });
    }
}
```

Usage

```php
use App\Admin\Renderable\UserTable;
use Dcat\Admin\Widgets\Modal;
use Dcat\Admin\Layout\Content;

public function index(Content $content)
{
	$modal = Modal::make()
		->lg()
		->title('异步加载 - 表格')
		->body(UserTable::make()) // Modal components support direct passing of rendering class instances.
		->button('Open Form');

	return $content->body($modal);
}
```

result

<a href="https://cdn.learnku.com/uploads/images/202008/23/38389/HiAMIvKext.gif!large" target="_blank">
    ![](https://cdn.learnku.com/uploads/images/202008/23/38389/HiAMIvKext.gif!large)
</a>



Similarly, instances of the Render class can be attached to components such as `Dcat\Admin\Widgets\Card`, `Dcat\Admin\Widgets\Box`, `Dcat\Admin\Widgets\Tab`.

```php
use App\Admin\Renderable\UserTable;
use Dcat\Admin\Widgets\Card;

$table = UserTable::make();

$card = Card::make('TITLE', $table)->withHeaderBorder();
```

When the above code renders the `UserTable` instance, it is actually an instance of the `Dcat\Admin\Widgets\LazyTable` class automatically added at the bottom level.

```php
use App\Admin\Renderable\UserTable;
use Dcat\Admin\Widgets\Card;
use Dcat\Admin\Widgets\LazyTable;

$table = LazyTable::make(UserTable::make()->simple());

$card = Card::make('TITLE', $table)->withHeaderBorder();
```

### Dcat\Admin\Grid\LazyRenderable

`Dcat\AdminGrid\LazyRenderable` is used for asynchronous rendering of data tables and is a subclass of `Dcat\AdminSupport\LazyRenderable`

#### Simplified model

This feature removes the ability to simplify some data forms that are enabled by default and not enabled by default.

```php
use App\Admin\Renderable\UserTable;
use Dcat\Admin\Widgets\LazyTable;
use Dcat\Admin\Layout\Content;

public function index(Content $content)
{
	$table = UserTable::make()->simple();

	return $content->body(LazyTable::make($table));
}
```

Note that if a rendering class instance is injected directly into a component such as `Dcat\Admin\Widgets\Card`, `Dcat\Admin\Widgets\Box`, `Dcat\Admin\WidgetsTab` and `Dcat\Admin\WidgetsModal`, `Dcat\Admin\Widgets` `simple mode` is automatically enabled. 

```php
use App\Admin\Renderable\UserTable;
use Dcat\Admin\Widgets\Card;

// simple mode is automatically enabled here.
$card = Card::make('TITLE', UserTable::make())->withHeaderBorder();
```

If you don't want simple mode enabled, you can pass the LazyTable instance

```php
use App\Admin\Renderable\UserTable;
use Dcat\Admin\Widgets\Card;
use Dcat\Admin\Widgets\LazyTable;

$table = LazyTable::make(UserTable::make());

$card = Card::make('TITLE', $table)->withHeaderBorder();
```

### Dcat\Admin\Widgets\LazyTable

#### onLoad

This method allows you to listen for asynchronous load completion events.

```php
use App\Admin\Renderable\PostChart;
use Dcat\Admin\Widgets\Lazy;

$chart = Lazy::make(PostChart::make())->onLoad(
	<<<JS
console.log('Component rendering complete');	
JS
);
```

#### load

This method controls whether to render the asynchronous component immediately. The default value is `true`.

```php
use App\Admin\Renderable\UserTable;
use Dcat\Admin\Widgets\LazyTable;
use Dcat\Admin\Admin;

$lazy = LazyTable::make(UserTable::make())->load(false);

Admin::script(
	<<<JS
setTimeout(function () {
	// Asynchronous rendering event triggered automatically after 3 seconds
	{$lazy->getLoadScript()}
}, 3000);	
JS
);
```

#### JS events

```php
use App\Admin\Renderable\UserTable;
use Dcat\Admin\Widgets\LazyTable;
use Dcat\Admin\Admin;

$lazy = LazyTable::make(UserTable::make());

Admin::script(
	<<<JS
// Manually triggering asynchronous rendering events	
$('{$lazy->getElementSelector()}').trigger('table:load');

// Listening for rendering completion events
$('{$lazy->getElementSelector()}').on('table:loaded', function () {
	console.log('The component is rendered.')
});
JS
);
```


<a name="lazy-form"></a>
## Asynchronous Loading Tool Form

Define the tool form class, implement the `Dcat\Admin\Contracts\LazyRenderable`, and load the `trait` of `Dcat\Admin\Traits\LazyWidget`.

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
        return $this->response()->success('Save successfully');
    }

    public function form()
    {
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
            ->help('Please enter your confirmation password.');
    }
}
```


use


```php
use App\Admin\Forms\UserProfile;
use Dcat\Admin\Widgets\Modal;
use Dcat\Admin\Layout\Content;

public function index(Content $content)
{
	$modal = Modal::make()
		->lg()
		->title('Asynchronous Loading - Forms')
		->body(UserProfile::make()) // Modal components support direct passing of rendering class instances
		->button('Open Form');

	return $content->body($modal);
}
```

result

![](https://cdn.learnku.com/uploads/images/202008/20/38389/C8InwPTsQG.gif!large)


Asynchronous form instances, of course, can be used in other components as well

```php
use App\Admin\Forms\UserProfile;
use Dcat\Admin\Widgets\Lazy;
use Dcat\Admin\Widgets\Card;

$form = UserProfile::make();

// Passing directly to Card Components
$card = Card::make($form);

// equal to 
$card = Card::make(Lazy::make($form));
```


### Passing custom parameters

Passing parameters to an asynchronous form is very simple, modify the above form class as follows

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
        // Getting externally passed parameters
        $key1 = $this->payload['key1'] ?? null;
        $key2 = $this->payload['key1'] ?? null;
        
        return $this->response()->success('Save successfully');
    }

    public function form()
    {
        // Getting externally passed parameters
		$key1 = $this->payload['key1'] ?? null;
		$key2 = $this->payload['key1'] ?? null;
        
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
            ->help('Please enter your confirmation password.');
    }
    
    public function default()
    {
        // Getting externally passed parameters
		$key1 = $this->payload['key1'] ?? null;
		$key2 = $this->payload['key1'] ?? null;
        
        return [
            'name' => '...',
		];
    }
}
```

Passing parameter code as follows

```php
// Passing custom parameters
$form = UserProfile::make()->payload(['key1' => '...', 'key2' => '...']);

$modal = Modal::make()
	->lg()
	->title('Asynchronous Loading - Forms')
	->body($form)
	->button('Open Form');
```


