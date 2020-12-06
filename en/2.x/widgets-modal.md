# Modal windows (Modal)

Basic use


```php
use Dcat\Admin\Widgets\Modal;

$modal = Modal::make()
	->lg()
	->title('TITLE')
	->body(view(...))
	->button('<button class="btn btn-primary">Click to open a pop-up window</button>');
	
return view(..., ['modal' => $modal]);	
```

## Functionality

### TITLE (title)

Set pop-up window TITLE

```php
$modal->title('TITLE');
```

### content (body)

Set popup contents, this method accepts a parameter that allows incoming values of type `string`, `Closure`, `Illuminate\Contracts\Support\Renderable` and `Dcat\Admin\Contracts\LazyRenderable`.

```php
// pass a string
$modal->body('字符串');

// pass a closure, note that the closure must return a string type value or a null value
$modal->body(function () {
	return view(...)->render();
});

// Passing in Renderable
use Dcat\Admin\Widgets\

$modal->body(view(...));
$modal->body(Card::make());

// Passing in LazyRenderable
$modal->body(PostTable::make());
```

### Bottom content (footer)
Set the content at the bottom of the popup window, this method accepts a parameter that allows the passing of `string`, `Closure`, `Illuminate\Contracts\Support\Renderable` and `Dcat\Admin\Contracts\LazyRenderable` type values, Usage as above.

```php
$modal->footer('字符串');

$modal->footer(view(...));
```

### size 
    
    Default `500px`

```php
// 800px
$modal->lg();

// 1140px
$modal->xl();
```

### Buttons (button)

Set button

### Event listening

Supported Events

 - `onShow` Popup Show Event
 - `onShown` Pop-up window shows event
 - `onHide` Popup hide event
 - `onHidden` Pop-ups with hidden event
 
UsageExample

```php
use Dcat\Admin\Admin;

$modal->onShow(
	<<<JS
console.log('shown a pop-up window', target, $(this));	
JS
);

$modal->onHide(
	<<<JS
console.log('Hidden a pop-up window', target, $(this));	
JS
);
``` 
 


<a name="form"></a>
## Form pop-up

Reference [tools-form - popups](widgets-form.md#modal)

