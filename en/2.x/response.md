# Actions and form responses

The response methods for [action] (action.md), [data form] (model-form.md), and [tool form] (widgets-form.md) are all the same set of methods.

You can get the `Dcat\Admin\Http\JsonResponse` object in the class by using `$this->response()` and respond to the data to the front end with the

```php
return $this->response()->success('success！');

// equal to
use Dcat\Admin\Http\JsonResponse;

return JsonResponse::make()->success('success！');
```

If used in a controller, you need to add the `send` method

```php
public function index()
{
    return JsonResponse::make()->success('成功！')->send();
}
```


### Function
The following describes the main Usages of `JsonResponse`.


#### showcasesuccess information

This method takes an argument of type `string`

```php
$this->response()->success('success！');
```

#### displays error messages

This method takes an argument of type `string`.

```php
$this->response()->error('出错了！');
```

#### Display warning information

This method takes an argument of type `string`.

```php
$this->response()->warning('警告');
```

#### Redirect

This method takes a parameter of type `string` and can be used with methods such as `success`, `error`, `warning`, etc.

```php
$this->response()->redirect('auth/users');
```

#### Redirect (location)

This method takes an argument of type `string`.

```php
$this->response()->location('auth/users');
```

#### Refresh current page

This method can be used with methods such as `success`, `error`, `warning` etc.

```php
$this->response()->success('xxx')->refresh();
```

Download ####

This method takes an argument of type `string`.

```php
$this->response()->download('auth/users?_export_=1');
```

#### show confirmation pop-up

```php
// success
$this->response()->alert(true)->success('...')->detail('Details');

// error
$this->response()->alert(true)->error('...')->detail('Details');

// warning
$this->response()->alert(true)->warning('...')->detail('Details');

// info
$this->response()->alert(true)->info('...')->detail('Details');
```

#### Return to HTML

This method accepts a `string`, `Renderable`, `Htmlable` type parameter and can be used with `success`, `error`, `warning` methods.

> {tip} The `HTML` character will be placed on the action button element by default. If you want to control it, override the [handleHtmlResponse](#handleHtmlResponse) method.

```php
$this->response()->html('<a>a tag</a>');

$this->response()->html(view('...'));
```

#### Execute JS code

This method takes a parameter of type `string` and can be used with methods such as `success`, `error`, `warning`, etc.

```php
$this->response()->script(
	<<<JS
console.log('response', response, target);	
JS	
);
```

### Determine whether to call or not based on conditions

All of the above functional interfaces support the `if` mode, such as the

```php
// If the value of $condition is true, then the refresh method is called
$this->response()->success(...)->ifRefresh($condition);
$this->response()->success(...)->ifLocation($condition, 'auth/users');

// $condition can also be a closure
$this->response()->success(...)->ifRefresh(function () {
    return true;
});
```

