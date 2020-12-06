# Basic use of motion

The `Action` action class makes it very easy for developers to develop an action that contains a specific function that allows the user to interact with the server very easily.

For example, if there is a button on a page, and user clicks on it to request the server to display the information of the logged in user in a popup window, then this button can be developed by `Action`.

## Example

Let's start developing a button for viewing logged-in user information.


### Creating an Action class with a command
First, you need to create the `Action` class and run the command

```bash
php artisan admin:action
```

After successful execution, you will see the following message in the command window, asking the developer to select an `Action` class type, here we type `0`.

> {tip} An action class of type ``default`` can be used anywhere on the page.

```bash
 Which type of action would you like to make?:
  [0] default
  [1] grid-batch
  [2] grid-row
  [3] grid-tool
  [4] form-tool
  [5] show-tool
  [6] tree-tool
 > 0 # Enter 0
 
```

Next, type in the name of the `Action` class in `PascalCase` style.

```bash

 Please enter a name of action class:
 > ShowCurrentAdminUser

```

The default namespace is `App\Admin\Actions`, here we just press Enter to skip it.

```bash

 Please enter the namespace of action class [App\Admin\Actions]:
 >

```

So an `Action` class is created, and the path of the class just created is `app/Admin/Actions/ShowCurrentAdminUser.php`.


### Use

Modify the `Action` class as follows

> {tip} If you need to pass arguments via a constructor in your action class, be sure to set a default value for all arguments to the constructor!

```php
<?php

namespace App\Admin\Actions;

use Dcat\Admin\Admin;
use Dcat\Admin\Widgets\Table;
use Dcat\Admin\Actions\Action;
use Dcat\Admin\Actions\Response;
use Dcat\Admin\Traits\HasPermissions;
use Illuminate\Contracts\Auth\Authenticatable;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Http\Request;

class ShowCurrentAdminUser extends Action
{
    /**
     * Button Title
     *
     * @var string
     */
    protected $title = 'Personal Information';

    /**
     * @var string
     */
    protected $modalId = 'show-current-user';

    /**
     * If you don't need it, please delete it.
     *
     * @param Request $request
     *
     * @return Response
     */
    public function handle(Request $request)
    {
        // Get the currently logged in user model
        $user = Admin::user();
        // Here we present the model data in a table
        $table = Table::make($user->toArray());

        return $this->response()
            ->success('Search Success')
            ->html($table);
    }

    /**
     * Processes the HTML string of the response and appends it to the popup node.
     *
     * @return string
     */
    protected function handleHtmlResponse()
    {
        return <<<'JS'
function (target, html, data) {
    var $modal = $(target.data('target')); 
    
    $modal.find('.modal-body').html(html)
    $modal.modal('show')
}        
JS;
    }

    /**
     * Setting the properties of HTML tags
     *
     * @return void
     */
    protected function setupHtmlAttributes()
    {
        // Add class
        $this->addHtmlClass('btn btn-primary');

        // ID of the saved popup
        $this->setHtmlAttribute('data-target', '#'.$this->modalId);

        parent::setupHtmlAttributes();
    }

    /**
     * To set the HTML of the button, here we need to attach the HTML of the popup window.
     *
     * @return string|void
     */
    public function html()
    {
        // Button's html
        $html = parent::html();

        return <<<HTML
{$html}        
<div class="modal fade" id="{$this->modalId}" tabindex="-1" role="dialog">
  <div class="modal-dialog modal-lg" role="document">
    <div class="modal-content">
      <div class="modal-header">
        <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span aria-hidden="true">&times;</span></button>
        <h4 class="modal-title">{$this->title()}</h4>
      </div>
      <div class="modal-body"></div>
    </div>
  </div>
</div>
HTML;
    }

    /**
     * Confirm the pop-up message, and delete this method if you don't need it. 
     *
     * @return string|void
     */
    public function confirm()
    {
        // return ['Confirm?', 'contents'];
    }

    /**
     * Action permission, return false means no permission, if you don't need it, you can delete this method.
     *
     * @param Model|Authenticatable|HasPermissions|null $user
     *
     * @return bool
     */
    protected function authorize($user): bool
    {
        return true;
    }

    /**
     * This method allows you to set the parameters to be attached to an action request.
     *
     * @return array
     */
    protected function parameters()
    {
        return [];
    }
}
```


After modifying it, you can start using it.

```php
<?php

use App\Admin\Actions\ShowCurrentAdminUser;

class IndexController
{
	public function index(Content $content)
	{
	    return $content->body(ShowCurrentAdminUser::make());
	}
}
```

The results are as follows

<a href="{{public}}/assets/img/screenshots/action-default.png" target="_blank">
    <img class="img img-full" src="{{public}}/assets/img/screenshots/action-default.png" />
</a>

## Properties

`Dcat\Admin\Actions\Action` Description of Available Class Properties

| attribute name      | type     | default    | description     |
| ---------- | -------- |---------- | -------- |
| title | `string`  |  | TITLE |
| selectorPrefix | `public` `string` | `.admin-action-` | `Css` selector for the target element |
| method | `string` | `POST`  | Request methods for interaction with the server |
| event | `string` | `click` | Events bound to the target element, default to click events. |
| disabled | `bool` | `false` | If or not to render the action element, set `true` to not render the action element |
| usingHandler | `bool` | `true` | When this attribute is set to `false`, no request is made to the server regardless of whether `Action` contains the `handle` method or not! |


## Methods

Description of the `Dcat\Admin\Actions\Action` class methodology

### Create an instance (make)

This method is a static method that instantiates the action class

```php
$action = MyAction::make($param1, $param2...);
```

### Process request (handle)

When this method is included in the `Action` class, the target element is bound to an event set by the `event` attribute (default is `click`). If the event is triggered, a request is made to the server, and the `handle` method handles and responds to the request.
 
> {tip} The `handle` method handles and responds to the request. If there is no such method, the target element is not bound to the event.

### Respond (response)

Refer to [Actions and Form Responses](response.md) chapter

### Set request data (parameters)

This method allows you to set the parameters that should be attached to the request when the action is initiated.

```php
<?php

use Dcat\Admin\Actions\Action;
use Illuminate\Http\Request;

class MyAction extends Action
{
    public function handle(Request $request) 
    {
        // Receiving parameters
        $key1 = $request->get('key1');
        $key2 = $request->get('key2');
        
        return $this->response()->success('success！');
    }
    
    public function parameters()
    {
        return [
        	'key1' => 'value1',
        	'key2' => 'value2',    
		];
    }
}

```

### Confirm popups

This method requires a `string` parameter to be returned.

A confirmation window is added when the return value of this method is not null, and a confirmation box pops up automatically when the event is triggered.


```php
public function confirm()
{
	return 'Are you sure you want to remove this line of content?';
}
```

Show popup title and content

```php
public function confirm()
{
	return ['Are you sure you want to delete this line of content?' , 'pop-up content'];
}
```

### JS code that is executed before the request is initiated (actionScript)

The `js` code preceding the execution of the set action executes the `js` code set by this method before initiating the request when the button-bound event is triggered, and this method requires a `js` anonymous function to be returned.

The `js` anonymous function takes the following three arguments:

- `data` `object` The data object to be passed to the interface.
- `target` `object` Action button's `jQuery` object
- `action` `object` Action management object.

```php
protected function actionScript()
{
	return <<<JS
function (data, target, action) {
    console.log('Before initiating a request', data, target, action);
    
    // return false; Here return false to stop the rest of the operation.	
    
    // Changing the primary key value passed to the interface
    action.options.key = 123;
}
JS	
}
```

<a name="handleHtmlResponse"></a>
### HTML code to handle the server response (handleHtmlResponse)

Processes the `HTML` code of the server response, which requires a `js` anonymous function to be returned.

```php
protected function handleHtmlResponse()
{
        return <<<'JS'
function (target, html, data) {
    // target JQ object of the action button
    // html HTML string returned by the interface
    // data json object of the complete data returned by the interface.

    target.html(html);
}
JS;
}
```

### Permissions (authorize)

This method is used to determine the privileges of the logged in user, the default is `true`

```php
protected function authorize($user): bool
{
	return $user->can('do-action');
}
```

### No Authority Response (failedAuthorization)

This method is used to set the response to an authentication failure and can be overridden if desired

```php
public function failedAuthorization()
{
	return $this->response()->error(__('admin.deny'));
}
```


### Hide or show (disable)

Set to show or hide this action

```php
// hide
MyAction::make()->disable();

// show
MyAction::make()->disable(false);
```


### Determine whether or not to show (allowed)

Determine whether the action is allowed to be displayed
```php
if (MyAction::make()->allowed()) {
	...
}
```

### setKey

Setting the Data Primary Key

```php
$id = ...;

MyAction::make()->setKey($id);
```

### Get the primary key value (getKey)

Gets the data primary key, also available in the `handle` method

```php
<?php

use Dcat\Admin\Actions\Action;
use Illuminate\Http\Request;

class MyAction extends Action
{
    public function handle(Request $request) 
    {
        $id = $this->getKey();
        
        ...
        
        return $this->response()->success('msng！');
    }
    
    public function title()
    {
        return "TITLE {$this->key()}";
    }
}
```


### Get target element style (getElementClass)

Get the `class` of the action target element (button)

```php
$class = MyAction::make()->getElementClass();
```

### Get the Css selector for the target element

Get the CSS selector for the action target element (button)

```php
$selector = MyAction::make()->selector();

Admin::script(
	<<<JS
	$('$selector').click(...);
JS	
);
```

### Add style (addHtmlClass)

Append the `class` of the action target element (button)

```php
MyAction::make()->addHtmlClass('btn btn-primary');

MyAction::make()->addHtmlClass(['btn', 'btn-primary']);
```

### Set the HTML (html) of the target element

This method is used to set the `HTML` code of the action target element, overriding it if necessary

```php
protected function html()
{
	return <<<HTML
<a {$this->formatHtmlAttributes()}>{$this->title()}</a>
HTML;
}
```

### Adding JS code (script)

This method is used to add the `JS` code before the `render` method finishes executing

```php
protected function script()
{
	return <<<JS
console.log('...')
JS;
}
```

### Set the HTML attribute of the target element (setHtmlAttribute)

Set the `HTML` tag property of the target element

```php
MyAction::make()->setHtmlAttribute('name', $value);

// batch setting
MyAction::make()->setHtmlAttribute(['name' => $value]);

```

