# Form initialization

The callback function set by the `Form::resolving` method is triggered when the `Dcat\Admin\Form` class is instantiated.

The callback function set by the `Form::composing` method is triggered when the `render()` method is called (when the page is rendered).

Developers can change some of the settings or behavior of `Form` in these two events, for example, if they need to disable certain actions, they can add the following code to `app/Admin/bootstrap.php`.

```php
use Dcat\Admin\Form;

Form::resolving(function (Form $form) {

     $form->disableEditingCheck();
    
     $form->disableCreatingCheck();
    
     $form->disableViewCheck();
    
     $form->tools(function (Form\Tools $tools) {
          $tools->disableDelete();
          $tools->disableView();
          $tools->disableList();
     });

});
```
This way you don't have to set it in the code of each controller.

If you want to enable the setting in one of the forms after the global setting, for example, enable the checkbox that shows `Continue Editing`, call `$form->disableEditingCheck(false);` on the corresponding instance!