# Static resource loading

`Dcat Admin` supports loading of `js` scripts on demand.


### Changing the domain name of a static resource

Open the configuration file `config/admin.php`, find the parameter `assets_server` and change it; you can also add `.env` to the `.

```dotenv
ADMIN_ASSETS_SERVER=http://xxx.com
```

### Registration path alias

Open `app/Admin/bootstrap.php` and add the following code

```php
Admin::asset()->alias('@my-name1', 'assets/admin1');
Admin::asset()->alias('@my-name2', 'assets/admin2');

// Bulk registration is also possible.
Admin::asset()->alias([
	'@my-name1' => 'assets/admin1',
	'@my-name2' => 'assets/admin2',
]);
```

Use of aliases

```php
Admin::js('@my-name1/index.js');
Admin::css('@my-name1/index.css');
```

### Registration component

When a component has more `js` and `css` files, we can register these static resource files as a single component to make it easier to use. Open `app/Admin/bootstrap.php`, then add the following code

```php
Admin::asset()->alias('@editor-md', [
	'js' => [
		// Support for using path aliases
		'@admin/dcat/plugins/editor-md/lib/raphael.min.js',
		'@admin/dcat/plugins/editor-md/lib/marked.min.js',
		'@admin/dcat/plugins/editor-md/lib/prettify.min.js',
		'@admin/dcat/plugins/editor-md/lib/jquery.flowchart.min.js',
		'@admin/dcat/plugins/editor-md/editormd.min.js',
	],
	'css' => [
		'@admin/dcat/plugins/editor-md/css/editormd.preview.min.css',
		'@admin/dcat/extra/markdown.css',
	],
]);
```

usage

```php
Admin::requireAssets(['@editor-md']);
```

If you only need to load the `js` or `css` of this component and don't want to load all the files, then you can use the following method

```php
// Load js files only
Admin::js('@editor-md');

// Load only css files.
Admin::css('@editor-md');
```


### Loading js scripts

The `Admin::js` method can introduce the `js` script, using the following:
```php
class UserController extend Controller
{
    public function index()
    {
        Admin::js('/assets/js/index.js');
        
        Admin::js([
            '/assets/js/index2.js'
        ]);
    }
}
```

### Loading css scripts

The `Admin::css` method can introduce the `css` script, using the following:
```php
class UserController extend Controller
{
    public function index()
    {
        Admin::css('/assets/css/index.css');
        
        Admin::css([
            '/assets/css/index2.css'
        ]);
    }
}
```

### Adding js code dynamically

The `Admin::script` method can dynamically add `js` code, using the following:
```php
    public function index()
    {
        Admin::script(
            <<<JS
    console.log('Hello world!');
JS            
        );
    }
```

### Adding css code dynamically

The `Admin::style` method can dynamically add `css` code, using the following:
```php
    public function index()
    {
        Admin::style(
            <<<CSS
    body {
        color: #333;
    }
CSS            
        );
    }
```

### Introducing static resources into a template
Manual introduction of static resources into a template requires the `admin_asset` function:

```html
// Include css
<link rel="stylesheet" href="{{ admin_asset("vendor/dcat-admin/dcat-admin/main.min.css") }}">

// Include js
<script src="{{ admin_asset('vendor/dcat-admin/dcat-admin/main.min.js')}}"></script>
```

### Adding js code to a template

The `js` code to be added to the template needs to be executed within the `Dcat.ready` method to ensure that your `js` code executes after all `js` scripts have been loaded:
```html
<script>
Dcat.ready(function () {
   console.log('All the js are loaded'); 
});
</script>
```

