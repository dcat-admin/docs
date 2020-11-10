# Static resource loading

`Dcat Admin` modifies the `pjax` source code and adds the ability to load `js` scripts on-demand, so that the developer only needs to introduce the `js` components needed in the controller `action` instead of introducing all the `js` components during project initialization.

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

