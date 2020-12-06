# Head and feet

The data table supports filling the head and foot blocks as you wish.

```php
$grid->header(function ($collection) {
    return 'header';
});

$grid->footer(function ($collection) {
    return 'footer'; 
});
```

The parameter `$collection` of the closure function is `Illuminate\Support\Collection` class instance, which is the current page table data, the following is an example of the use of two different scenarios

## Head

```php
$grid->header(function ($collection) use ($grid) {
	$query = Model::query();
    	
	// Get the table filter where the conditional array is traversed.
	$grid->model()->getQueries()->unique()->each(function ($value) use (&$query) {
		if (in_array($value['method'], ['paginate', 'get', 'orderBy', 'orderByDesc'], true)) {
			return;
		}

		$query = call_user_func_array([$query, $value['method']], $value['arguments'] ?? []);
	});
	
	// Identify statistical data
	$data = $query->get();

    // Custom component
    return new Card($data);
});
```

Component implementation for custom header display
```php
<?php

use Illuminate\Contracts\Support\Renderable;
use Dcat\Admin\Admin;

class Card implements Renderable
{
	public static $js = [
		'xxx/js/card.min.js',
	];
	public static $css = [
		'xxx/css/card.min.css',
	];
	
	protected $data;
	
	public function __construct($data) 
	{
	    $this->data = $data;
	}

	public function script()
	{
		return <<<JS
		console.log('All JS scripts are loaded.');
		$('xxx').card();
JS;		
	}

	public function render()
	{
		// You can introduce your js or css file here.
		Admin::js(static::$js);
		Admin::css(static::$css);
		
		// JS code to be executed on the page
		// The JS code set by Admin::script will be executed automatically when all JS scripts are loaded.
		Admin::script($this->script());
		
		return view('...', ['data' => $this->data])->render();
	}
}
```


## Footer

A more common scenario is to display statistics in the footer of a data form, such as adding revenue statistics to the footer of an order form, which can be implemented with the following code.

```php
$grid->footer(function ($collection) use ($grid) {
	$query = Model::query();
	
	// Get the table to filter the where condition array for iterations
	$grid->model()->getQueries()->unique()->each(function ($value) use (&$query) {
		if (in_array($value['method'], ['paginate', 'get', 'orderBy', 'orderByDesc'], true)) {
		    return;
		}

		$query = call_user_func_array([$query, $value['method']], $value['arguments'] ?? []);
	});
	
	// Identifying statistics
	$data = $query->get();

    return "<div style='padding: 10px;'>gross income ： $data</div>";
});
```

If there is a more complex footer that needs to be displayed, you can also use a view object or wrap it into a class.