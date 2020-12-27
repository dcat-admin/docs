# Required reading before development

## Please turn on debug mode for development environment.
`Dcat Admin` provides some development tools (such as code generator) that need to be in `debug` mode to use.
It is recommended that developers turn on `debug` mode in their development environment and set `APP_DEBUG` to `true` in the `.env` configuration file.


## Introduce JS scripts on demand

`Dcat Admin` uses `jquery-pjax` to build refresh-free pages (single-page applications) and supports <b>loading</b> `JS` scripts on-demand, allowing `JS` scripts to be introduced in any page method (except template files) and each page to load only the `js` scripts needed for the current page.


Example：

To write a custom page, this page component needs to introduce some front-end static resource files

> {tip} The `Dcat Admin` build is a single page application, and the `JS` script that is loaded will only be executed once, so the initialization operation cannot be placed directly in the `JS` script, it should be loaded using the `Admin::script` method.

```php
<?php

use Illuminate\Contracts\Support\Renderable;
use Dcat\Admin\Admin;

class Card implements Renderable
{
	public static $js = [
	    // The js script cannot contain the initialization operation directly, otherwise the page refresh is invalid
		'xxx/js/card.min.js',
	];
	public static $css = [
		'xxx/css/card.min.css',
	];

	public function script()
	{
		return <<<JS
		console.log('All JS scripts are loaded.');
		// Initialization operations
		$('xxx').card();
JS;		
	}

	public function render()
	{
		// This is where you can bring in your js or css files
		Admin::js(static::$js);
		Admin::css(static::$css);
		
		// JS code that needs to be executed on the page, such as initialization code.
		// The JS code set by Admin::script will be executed automatically when all JS scripts are loaded.
		Admin::script($this->script());
		
		return view('...')->render();
	}
}
```

Using this component in the controller
```php
use Dcat\Admin\Layout\Content;
use Card;

class HomeController
{
	public function index(Content $content)
	{
		// Use the card component above.
		// Static files required by the Card component will only be loaded on the current request.
		// Other requests will not load 
		return $content->body(new Card());
	}
}
```


## Adding JS code to the page

Due to the addition of the ability to load `JS` scripts on demand, the `JS` code added in this project must use the `Dcat.ready` method to listen to the `JS` script load completion event, in which the `JS` code will be executed after all `JS` scripts have been loaded.


The `JS` code added using the `Dcat\Admin\Admin::script` method is automatically placed inside the `Dcat.ready` method for execution.

```php
<?php

use Dcat\Admin\Admin;
use Dcat\Admin\Layout\Content;

class UserController
{
   public function index(Content $content)
   {
       Admin::script(
           <<<JS
(function () {
    // If there is a need to define local variables, it is better to put them inside an anonymous function to prevent variable contamination.
    var name = 'test';

    console.log('All JS scripts are loaded.~~', name)
})()    
JS
       );
       
       return $content->header(...)->body(...);
   }
}
```


If you are adding `JS` code to a template file, you need to execute the code in `Dcat.ready`

```html
<script>
// Replace $() with Dcat.ready()
// This method will be executed after all js scripts are loaded
Dcat.ready(function () {
    // Writing your js code
    console.log('All js scripts are loaded!~~');
});
</script>
```

## Page content and layout

> {tip} The page content layout function is the cornerstone of `Dcat Admin`, mastering this function of the Usages, you can easily use `Dcat Admin` to build pages or extend functionality, so please read carefully.

The layout of `Dcat Admin` can be referred to the `index()` method of the `app/Admin/Controllers/HomeController.php` layout file on the backend home page.


The `Dcat\Admin\Layout\Content` class is used to implement the layout of the content area. The `Content::body($content)` method is used to add page content.


A simple backstage page code would be as follows:

```php
use Dcat\Admin\Layout\Content;

public function index(Content $content)
{
	// optional filling
	$content->header('Fill page header TITLE');
	
	// optional filling
	$content->description('Fill page descriptions with small TITLE');
	
	// Adding breadcrumb navigation
	$content->breadcrumb(
		['text' => 'first page', 'url' => '/admin'],
		['text' => 'user management', 'url' => '/admin/users'],
		['text' => 'Edit User']
	);

	// Fill the body part of the page with any objects that can be rendered.
	return $content->body('hello world');
}

```

The `$content->body()` method is an alias of `$content->row()`, which can accept any stringable object as arguments, including strings, numbers, objects with the `__toString` method, and implements the `Renderable` and `Htmlable` interfaces. objects, including laravel views.


### Layout

The layout of `dcat-admin` uses a bootstrap grid system with each line being 12 in length, and the following are a few simple Examples:

Add a line of content:

```php
$content->row('hello')

---------------------------------
|hello                          |
|                               |
|                               |
|                               |
|                               |
|                               |
---------------------------------

```

Add multiple columns within rows：

```php
$content->row(function(Row $row) {
    $row->column(4, 'foo');
    $row->column(4, 'bar');
    $row->column(4, 'baz');
});
----------------------------------
|foo       |bar       |baz       |
|          |          |          |
|          |          |          |
|          |          |          |
|          |          |          |
|          |          |          |
----------------------------------


$content->row(function(Row $row) {
    $row->column(4, 'foo');
    $row->column(8, 'bar');
});
----------------------------------
|foo       |bar                  |
|          |                     |
|          |                     |
|          |                     |
|          |                     |
|          |                     |
----------------------------------

```

Adding rows to columns：

```php
$content->row(function (Row $row) {

    $row->column(4, 'xxx');

    $row->column(8, function (Column $column) {
        $column->row('111');
        $column->row('222');
        $column->row('333');
    });
});
----------------------------------
|xxx       |111                  |
|          |---------------------|
|          |222                  |
|          |---------------------|
|          |333                  |
|          |                     |
----------------------------------


```


Add rows to columns, and columns within rows.：

```php
$content->row(function (Row $row) {

    $row->column(4, 'xxx');

    $row->column(8, function (Column $column) {
        $column->row('111');
        $column->row('222');
        $column->row(function(Row $row) {
            $row->column(6, '444');
            $row->column(6, '555');
        });
    });
});
----------------------------------
|xxx       |111                  |
|          |---------------------|
|          |222                  |
|          |---------------------|
|          |444      |555        |
|          |         |           |
----------------------------------
```

### Build a page without a menu bar (full)

The page built in the above way defaults to a page with a left menu bar and a top navigation bar.
But sometimes we'll need to build a full page without a menu bar and top navigation bar, such as a landing page, or a page that needs to be loaded in IFRAME, and so on.


The page is rendered without menu bar and top navigation bar, and all the features and components of `Dcat Admin` can be used, which can significantly improve the efficiency.

The following is an implementation of the login page that demonstrates this function of the Usage

controller
```php
use Dcat\Admin\Layout\Content;

class AuthController extends Controller
{
    public function getLogin(Content $content)
    {
        if ($this->guard()->check()) {
            return redirect($this->redirectPath());
        }
		// Building landing pages using the full method
        return $content->full()->body(view($this->view));
    }
	
	...
}	
```

The following is the login function of the template content, because the controller uses the `Content::full` method to build the page, so there is no need to write `head` in the template, and do not care what static resources are introduced, just write the HTML of the current page can be, and can also use the `Dcat Admin` in all the functions, such as the following form submission function used.

```html
<style>
    html body {background: #fff;}
</style>

<link rel="stylesheet" href="{{ admin_asset('@admin/css/pages/authentication.css') }}">

<section class="row flexbox-container">
	<!-- Here is the HTML code for your landing page -->
	...
</section>

<script>
Dcat.ready(function () {
    // ajax form submission
    $('#login-form').form({
        validate: true,
    });
});

</script>
```

This landing page uses the `ajax` form submission function, and comes with a button `loading` result, which is better than the original landing page.


### Event

The following two events are triggered when the `Dcat\Admin\Layout\Content` class is instantiated and when the `render()` method is called, and the developer can change or add some behavior in both events.

#### instantiate (resolve)
The callback function set through the `Content::resolving` method is triggered when the `Dcat\Admin\Layout\Content` class is instantiated.

```php
use Dcat\Admin\Layout\Content;

Content::resolving(function (Content $content) {
    
    $content->view('app.admin.content');
    
});
```

#### building pages (composing)
The callback function set by the `Content::composing` method is triggered when the `Dcat\Admin\Layout\Content::render` method is called.

```php
use Dcat\Admin\Layout\Content;

Content::composing(function (Content $content) {
    
    $content->view('app.admin.content');
    
});
```

#### build is complete (composed)
The callback function set by the `Content::composed` method is triggered when all the content set by the `Content::row` or `Content::body` method has been constructed.


```php
use Dcat\Admin\Layout\Content;

class IndexController
{
	public function index(Content $content)
	{
		Content::composed(function (Content $content) {
            // Grid has executed the render method.
            
        });
        
        return $content->body(function ($row) {
        	$grid = new Grid(...);
        	
        	...
        	
        	$row->column(12, $grid);
        });
	}
}
```

<a name="admincontroller"></a>
## AdminController

Through the above page layout of the relevant content of the learning, we understand the `Dcat Admin` page composition of the construction of the method.
So how does an add, delete, and check feature actually work? We can see an add/drop controller code generated by the code generator as follows

```php
use App\Admin\Repositories\User;
use Dcat\Admin\Form;
use Dcat\Admin\Grid;
use Dcat\Admin\Show;
use Dcat\Admin\Http\Controllers\AdminController;

class UserController extends AdminController
{
    // Data table
    protected function grid()
    {
        return Grid::make(new User(), function (Grid $grid) {
            ...
        });
    }

    // Data details
    protected function detail($id)
    {
        return Show::make($id, new User(), function (Show $show) {
            ...
        });
    }

    // Form 
    protected function form()
    {
        return Form::make(new User(), function (Form $form) {
            ...
        });
    }
}
```
The above code mainly contains `grid`, `detail` and `form`, from these codes, we do not have a way to change the layout of a page, so how is this page actually built? And how do we change the layout of the page? Let's take a look at the `AdminController`

```php
<?php

namespace Dcat\Admin\Controllers;

use Dcat\Admin\Layout\Content;
use Illuminate\Routing\Controller;

class AdminController extends Controller
{
    // Page TITLE
    protected $title;

    // Page Description Information
    protected $description = [
        //        'index'  => 'Index',
        //        'show'   => 'Show',
        //        'edit'   => 'Edit',
        //        'create' => 'Create',
    ];

    // Back to page TITLE
    protected function title()
    {
        return $this->title ?: admin_trans_label();
    }

    // Back to description information
    protected function description()
    {
        return $this->description;
    }

    // list page
    public function index(Content $content)
    {
        return $content
            ->title($this->title())
            ->description($this->description()['index'] ?? trans('admin.list'))
            ->body($this->grid());
    }

    // detail page
    public function show($id, Content $content)
    {
        return $content
            ->title($this->title())
            ->description($this->description()['show'] ?? trans('admin.show'))
            ->body($this->detail($id));
    }

    // edit page
    public function edit($id, Content $content)
    {
        return $content
            ->title($this->title())
            ->description($this->description()['edit'] ?? trans('admin.edit'))
            ->body($this->form()->edit($id));
    }

    // create page
    public function create(Content $content)
    {
        return $content
            ->title($this->title())
            ->description($this->description()['create'] ?? trans('admin.create'))
            ->body($this->form());
    }

    // Modify interface
    public function update($id)
    {
        return $this->form()->update($id);
    }

    // New interface
    public function store()
    {
        return $this->form()->store();
    }

    // Delete/Batch Delete Interface
    public function destroy($id)
    {
        return $this->form()->destroy($id);
    }
}
```

Is it now possible to understand the components of the entire page? In fact, a lot of code in the system are known by name, easy to understand, many times we only need to read the code to know the Usage.
For example, if we want to change the page TITLE, by reading this code, we can know that it can be achieved by rewriting the `title` method or changing its entry in the translation file. Very simple, isn't it?

Let's demonstrate the practical application of changing the page layout by implementing a list page of data tables + statistics cards

```php
use App\Admin\Metrics\Examples\NewDevices;
use App\Admin\Metrics\Examples\NewUsers;
use App\Admin\Metrics\Examples\TotalUsers;
use Dcat\Admin\Layout\Content;
use Dcat\Admin\Layout\Row;

public function index(Content $content)
{
    return $content
        ->title($this->title())
        ->description($this->description()['index'] ?? trans('admin.list'))
        ->body(function (Row $row) {
            $row->column(4, new TotalUsers());
            $row->column(4, new NewUsers());
            $row->column(4, new NewDevices());
        })
        ->body($this->grid());
}
```

The result is as follows

![](https://cdn.learnku.com/uploads/images/202005/06/38389/p1lAW4NpQi.png!large)


<a name="bootstrap-styles"></a>
## Bootstrap4公共样式

`Dcat Admin` uses the `bootstrap4` grid system for page layout, which is both simple and powerful, and you need to know it before you start development!
In addition, `bootsrap4` provides a number of very useful [public styles](https://getbootstrap.net/docs/utilities/borders/), which are very helpful for writing page components and can significantly improve development efficiency.
It is recommended that you consult the documentation before writing components, and the following is a list of recommended styles to learn.

- [Grid layout](https://getbootstrap.net/docs/layout/grid/)
- [Display Properties](https://getbootstrap.net/docs/utilities/display/) With our display utility for quickly and efficiently toggling component display values and more, including support for some of the more common values, this list of styles is very helpful for responsive layouts.
- [flex layout](https://getbootstrap.net/docs/utilities/flex/) The introduction of the new Flex Flexible Layout allows for quick management of the layout, alignment and size of grid columns, navigation, components, etc. through a set of responsive and flexible utilities. More complex presentation styles are also possible through the progressive definition of CSS.
- [Color](https://getbootstrap.net/docs/utilities/colors/) There are a number of ways to define this, including support for linking, hovering, selecting, and other state-related style sets.
- [Float property](https://getbootstrap.net/docs/utilities/float/) Use our responsive float float universal style to toggle the float on any device breakpoint (browser size).                             
- [sizing](https://getbootstrap.net/docs/utilities/sizing/) Easily define the width or height of any element (relative to its parent) using the system width and height styles
- [spacing](https://getbootstrap.net/docs/utilities/spacing/) A variety of quick indent, isolate, and fill spacing tools are built in, responding to margin and fill utility classes to modify the appearance of elements.
- [text handling](https://getbootstrap.net/docs/utilities/text/) Used to control text alignment, grouping, word weight, and other Examples as well as to work with documents.
- [vertical alignment](https://getbootstrap.net/docs/utilities/vertical-align/) Easily change the vertical alignment of inline, inline block, inline table and table cell elements.


### Built-in style
In addition to the previously mentioned [``bootstrap4` public styles](notice.md#bootstrap-styles), the system also has the following common styles built in:


#### Colors
Please refer to [color chart styles](theme.md# color chart and styles).

#### shadows
<style>
	.shadow-item {
		width:120px;height:100px;line-height:100px;margin:5px 10px;display:inline-block;text-align:center;
	}
</style>

<div class="shadow-item" style="box-shadow:0 2px 4px 0 rgba(0,0,0,.08);">
	<code>.shadow</code>
</div>
<div class="shadow-item" style="box-shadow:0 3px 1px -2px rgba(0,0,0,.05), 0 2px 2px 0 rgba(0,0,0,.05), 0 1px 5px 1px rgba(0,0,0,.05);">
	<code>.shadow-100</code>
</div>
<div class="shadow-item" style="box-shadow:0 3px 1px -2px rgba(0,0,0,.1), 0 2px 2px 0 rgba(0,0,0,.1), 0 1px 5px 1px rgba(0,0,0,.1)">
	<code>.shadow-200</code>
</div>
