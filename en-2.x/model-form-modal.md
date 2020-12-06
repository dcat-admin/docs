# Form pop-up


## Data Form Popups

The `Form::dialog` method can be used to quickly build a data form-based form popup, adding only a few lines of code.

> {tip} The form popup is based on the following principle: the `create` and `edit` pages are used to retrieve the built form `HTML` characters, and then the `HTML` characters are rendered using a popup plugin.
If a new `js` script needs to be loaded in the meantime, it will wait until the script is loaded and then execute the form initialization `js` code.

### Simple example

#### opens form pop-up
```php
<?php

use App\Http\Controllers\Controller;
use Dcat\Admin\Form;
use Dcat\Admin\Layout\Content;

class ModalFormController extends Controller
{
    public function index(Content $content)
    {
        return $content
            ->header('Modal Form')
            ->body($this->build());
    }

    protected function build()
    {
        Form::dialog('Add new roles.')
            ->click('.create-form') // Binding click button
            ->url('auth/roles/create') // Form page link, this parameter is replaced by the "data-url" attribute of the button.
            ->width('700px') // Specify the width of the popup window, fill in the percentage, default 720px
            ->height('650px') // Specify the height of the popup window, can fill in the percentage, default 690px
            ->success('Dcat.reload()'); // Refresh page after adding success

        Form::dialog('Editing roles')
            ->click('.edit-form')
            ->success('Dcat.reload()'); // Refresh the page after editing the success

        // When you need to bind different links to the same "class" button, put the link in the button's "data-url" attribute.
        $editPage = admin_base_path('auth/roles/1/edit');

        return "
<div style='padding:30px 0'>
    <span class='btn btn-success create-form'> New Form Popup </span> &nbsp;&nbsp;
    <span class='btn btn-blue edit-form' data-url='{$editPage}'> Edit Form Popup </span>
</div>
";
    }

}
```

#### Form building and saving data
```php
<?php

use App\Admin\Repositories\Role;
use Dcat\Admin\Controllers\HasResourceActions;
use Dcat\Admin\Form;
use Dcat\Admin\Layout\Content;
use Dcat\Admin\Admin;

class RoleController
{
    use HasResourceActions;
    
    /**
         * Edit interface.
         *
         * @param mixed   $id
         * @param Content $content
         *
         * @return Content
         */
        public function edit($id, Content $content)
        {
            return $content
                ->header(trans('admin.roles'))
                ->description(trans('admin.edit'))
                ->body($this->form()->edit($id));
        }
    
        /**
         * Create interface.
         *
         * @param Content $content
         *
         * @return Content
         */
        public function create(Content $content)
        {
            return $content
                ->header(trans('admin.roles'))
                ->description(trans('admin.create'))
                ->body($this->form());
        }
    
    /**
     * Make a form builder.
     *
     * @return Form
     */
    protected function form()
    {
        return Form::make(new Role(), function (Form $form) {
            $form->display('id', 'ID');
            
            $form->text('slug', trans('admin.slug'))->required()->prepareForSave(function ($value) {
                return $value;
            });
            $form->text('name', trans('admin.name'))->required();
    
            $form->tree('permissions')
                ->nodes(function () {
                    $permissionModel = config('admin.database.permissions_model');
                    $permissionModel = new $permissionModel;
    
                    return $permissionModel->allNodes();
                })
                ->customFormat(function ($v) {
                    if (!$v) return [];
    
                    return array_column($v, 'id');
                });
    
            $form->display('created_at', trans('admin.created_at'));
            $form->display('updated_at', trans('admin.updated_at'));
        });
    }
}
```

#### result
<a href="{{public}}/assets/img/screenshots/form-modal.png" target="_blank">
    <img style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" src="{{public}}/assets/img/screenshots/form-modal.png">
</a>

### Functional interface

The form popup must be tied to a clickable page element that is clicked to bring up the popup.

#### set popupTITLE

```php
$modal = Form::dialog('TITLE');
```

#### bind click button
You can bind a clicked button via the `ModalForm::click` method, which will bring up a popup when the button is clicked.

```php
Form::dialog('TITLE')
    ->click('#click-button');
```

#### Set URL

If you are creating a form, you can get the url of the form template by setting it as follows
```php
Form::dialog('新增角色')
    ->click('.create-form')
    ->url('auth/roles/create');
```

If it's an edit type form, multiple URLs are required because the content of the form that pops up with each button clicked is different, so each button has a different link.

At this time, a link set by the `ModalForm::url` method is no longer sufficient, so you need to save the url on the `data-url` property of the clicked button:
```php
Form::dialog('编辑角色')
    ->click('.edit-form')
    ->success('Dcat.reload()'); // Refresh the page after editing the success

    // When you need to bind different links to the same "class" button, put the link in the button's "data-url" attribute.
    $editPage1 = admin_base_path('auth/roles/1/edit');
    $editPage2 = admin_base_path('auth/roles/2/edit');
return "
<div style='padding:30px 0'>
    <span class='btn btn-blue edit-form' data-url='{$editPage1}'> Edit Form Popup1 </span>
    <span class='btn btn-blue edit-form' data-url='{$editPage2}'> Edit Form Popup 2 </span>
</div>
";
```

#### Form save success callback

The `success` method can be used to set the `js` code that will be executed after the form saves the success, and there is a `response` variable in the scope of the `js` code.

```php
Form::dialog('Editing roles')
   ->click('.edit-form')
   ->success(
       <<<JS
// Print the server response data          
console.log(response);

// Prompt success information
Dcat.success(response.message || 'Save success');

// Refresh the page after saving the success
Dcat.reload();
JS        
   );
```

#### Form Save Failure Callback

Through the `error` method, you can set the `js` code that will be executed after the form fails to save, and there is a `response` variable in the scope of this `js` code, through which you can get the `json` data returned by the server.

```php
Form::dialog('Editing roles')
   ->click('.edit-form')
   ->error(
       <<<JS
// Print server response data       
console.log(response);
JS        
   );
```

#### Callback after form save
The `saved` method allows you to set the `js` code to be executed after the form is saved:
- `success` A value of true means the success is saved, otherwise it fails.
- `response` The `json` data returned by the server

```php
Form::dialog('Editing roles')
   ->click('.edit-form')
   ->saved(
       <<<JS
// Print server response data       
console.log(response);

if (success) {
    console.log('Save success');
} else {
    console.log('Save failed');
}
JS        
   );
```

#### Forced refresh

Pull new template data from the server each time you click a button.

```php
Form::dialog('Editing roles')
    ->click('.edit-form')
    ->forceRefresh();
```

#### Set Popup Width and Height

```php
Form::dialog('Editing roles')
    ->click('.edit-form')
    ->dimensions('50%', '400px');
    
// or
Form::dialog('Editing roles')
    ->click('.edit-form')
    ->width('50%')
    ->height('400px');
```


## Tool form pop-up

The popup function of data form is usually more complicated to implement with a resource controller, so the system has a built-in popup function.


<a href="{{public}}/assets/img/screenshots/modal-widget-form.png" target="_blank">
    <img class="img img-full" src="{{public}}/assets/img/screenshots/modal-widget-form.png">
</a>


