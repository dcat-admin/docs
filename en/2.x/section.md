# Section

## Introduction

`Dcat Admin` provides the `section` feature that allows developers to change the content of various sections of the page at runtime without having to modify the template directly.

> {tip} `Dcat Admin`'s `section` function refers to `Blade` template engine's `@section` function and `Wordpress` `add_filter` function, if the developer understands one of these two can quickly start.

## Use

### admin_section

Output `section` content, this function is similar to the `@yield` command in the `Blade` template and the `apply_filters` function in `WordPress`.

Parameters.
- `$section` {string} Block Name
- `$default` {string|Closure} Default Value
- The `$options` {array} parameter, please note that the value of `key` must start with an English letter, otherwise it cannot be retrieved.

Returned values:
- {string}

```php
echo admin_section('navigation', null, ['count' => 4]);
```

### admin_inject_section

Note the content of `section`, this function is similar to the `add_filter` function in `WordPress`.

Parameters:
- `$section` {string} Block Name
- `$content` {string|Illuminate\Contracts\Support\Renderable|Closure} block content
- If `$append` {bool} is passed as `true`, whether to append it to the last injected content or not, if `false` is passed, then the previous injected content will be replaced.
- `$priority` {int} Default `10`, priority, the higher the value, the higher the order.

When the second argument is passed to the anonymous function, the anonymous function receives an `Illuminate\Support\Fluent` object. The anonymous function receives an `IlluminateSupport\Fluent` object that contains the contents of the `previous` attribute, which can be obtained by accessing the `section` attribute; if the `section` has other parameters, they can also be obtained by accessing the attribute, as follows:

```php
admin_inject_section('navigation', e("<navigation>1</navigation>"));

admin_inject_section('navigation', function ($options) {
    // Get the last content injected into this block
    $previous = $options->previous;
    
    // Get custom parameters
    $count = $options->count;

    return e("<navigation>count:{$count}</navigation>");
}, true, 11);

// output
echo admin_section('navigation', null, ['count' => 4]);
// The final output is
// <navigation>count:4</navigation><navigation>1</navigation>
```

### admin_inject_default_section

Inject default content, which does not take effect if the `admin_inject_section` function is called to inject content (either before or after).

parameters：
- `$section` {string} block name
- `$content` {string|Illuminate\Contracts\Support\Renderable|Closure} Block content, consistent with the second parameter of `admin_inject_section`

```php
admin_inject_default_section('navigation', 'No data available');
```

### admin_has_section

This function returns a value of type `bool` to determine if content has been injected into `section`.
```php
var_dump(admin_has_section('navigation'));
```

### admin_has_default_section

This function returns a value of type `bool` to determine if default content has been injected into `section`.
```php
var_dump(admin_has_default_section('navigation'));
```

## System predefined blocks

The `Dcat Admin` predefines blocks through which the developer can change the content of the page.

All predefined block names are defined in the class `AdminSection`, which is accessed by means of class constants.

### Enter content into the &lt;head> tag

Here the `AdminSection::HEAD` block allows you to enter content into the `<head>` tag.

Add the following code to `app\Admin\bootstrap.php`：
```php
admin_inject_section(\AdminSection::HEAD, function () {
    return '<script src="//oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>';
});
```

### Enter content into the &lt;body> tag

The `AdminSection::BODY_INNER_BEFORE` block allows you to enter content to the beginning of the `<body>` tag interior.

The `AdminSection::BODY_INNER_AFTER` block allows you to enter content to the end position inside the `<body>` tag.


### Enter content into the &lt;div id="app"> tag

The `AdminSection::APP_INNER_BEFORE` block allows you to enter content into the beginning of the `<div id="app">` tag interior.

The `AdminSection::APP_INNER_AFTER` block allows you to enter content to the end position inside the `<div id="app">` tag.

### Changing the contents of the user information panel in the top navigation bar

The `AdminSection::NAVBAR_USER_PANEL` block allows you to change the contents of the user information panel in the top navigation bar.

```php
admin_inject_section(\AdminSection::NAVBAR_USER_PANEL, view('admin::partials.navbar-user-panel'));
```

### Changing the content behind the user information panel in the top navigation bar

The `AdminSection::NAVBAR_AFTER_USER_PANEL` block allows you to change the content behind the user information panel in the top navigation bar.

```php
admin_inject_section(\AdminSection::NAVBAR_AFTER_USER_PANEL, function () {
    return <<<HTML
    <li>
        <a href="#" data-toggle="control-sidebar"><i class="fa fa-gears"></i></a>
    </li>
HTML;    
});
```

### Changing the contents of the menu bar user information panel

The `AdminSection::LEFT_SIDEBAR_USER_PANEL` block allows you to change the contents of the user information panel of the menu bar.
```php
 admin_inject_section(\AdminSection::LEFT_SIDEBAR_USER_PANEL, view('admin::partials.sidebar-user-panel'));
```

### Changing the menu bar

The entire menu bar content can be changed with `AdminSection::LEFT_SIDEBAR_MENU`.
> {tip} The `Dcat Admin` menu is built by injecting default content into the `LEFT_SIDEBAR_MENU` block and developers can easily replace the system's default menu rendering logic.

```php
use Dcat\Admin\Support\Helper;
use Dcat\Admin\Admin;

admin_inject_section(\AdminSection::LEFT_SIDEBAR_MENU, function () {
    $menuModel = config('admin.database.menu_model');
	
	$builder = Admin::menu();

	$html = '';
	foreach (Helper::buildNestedArray((new $menuModel())->allNodes()) as $item) {
		$html .= view('admin::partials.menu', ['item' => $item, 'builder' => $builder])->render();
	}

	return $html;
});
```


