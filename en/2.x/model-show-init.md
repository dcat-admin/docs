# Data detail initialization

The callback function set through the `Show::resolving` method is triggered when the `Dcat\Admin\Show` class is instantiated.

The callback function set by the `Show::composing` method is triggered when the `render()` method is called.

Developers can change some of the settings or behavior of `Show` in these two events, for example, if they need to disable certain actions, they can add the following code to `app/Admin/bootstrap.php`.

```php
use Dcat\Admin\Show;

Show::resolving(function (Show $show) {

     $show->showQuickEdit();
   
});
```
