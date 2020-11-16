# Form Widget

The Form Widget (`Dcat\Admin\Widgets\Form`) is used to build the form and process the submitted data, making it easy to process the data independently without the need for additional model routes.


### Basic Use
Use the command `admin:form` to generate the form class file.

```bash
php artisan admin:form Setting
```
The form file `app/Admin/Forms/Setting.php` will be generated.

```php
<?php

namespace App\Admin\Forms;

use Dcat\Admin\Widgets\Form;
use Symfony\Component\HttpFoundation\Response;

class Setting extends Form
{
    // Handling form submission requests
    public function handle(array $input)
    {
        // dump($input);

        return $this->response()->success('Processed successfully.')->refresh();
    }

    // Building Forms
    public function form()
    {
        // pop-up confirmation pop-up 
        $this->confirm('Are you sure you want to submit the form?', 'content');
        
        $this->text('name')->required();
        $this->email('email')->rules('email');
    }

    /**
     * Return form data, delete this method if you don't want it.
     *
     * @return array
     */
    public function default()
    {
        return [
            'name'  => 'John Doe',
            'email' => 'John.Doe@gmail.com',
        ];
    }
}
```
In the form class above, build the form item in the `form` method, using the same method as [data form](model-form.md), the `default` method is used to set the default data for this form item.

After filling in the data in the page and submitting the form, the request goes into the `handle` method, where you can add your data processing logic, and after processing, you can respond to the data to the frontend via the `success` or `error` method.
```php
    public function handle(array $input)
    {
        // $input is the form data you receive.
        // You can write your processing logic here.


        return $this->response()->success('Processed successfully.')->refresh();
    }
```

Then place the above form on your page as follows:

```php
<?php

use App\Admin\Forms\Setting;
use App\Http\Controllers\Controller;
use Dcat\Admin\Widgets\Card;
use Dcat\Admin\Layout\Content;

class UserController extends Controller
{
    public function setting(Content $content)
    {
        return $content
            ->title('Website settings')
            ->body(new Card(new Setting()));
    }
}
```

### Pop-up confirmation pop-up

The second parameter is optional.

```php
$this->confirm('title', 'content');
```


### Response methods

Refer to [Actions and Form Responses](response.md) chapter


### Customizing the subsequent behavior of form saves

```php
<?php

namespace App\Admin\Forms;

use Dcat\Admin\Widgets\Form;
use Symfony\Component\HttpFoundation\Response;

class Setting extends Form
{
    ...
    
    /**
 	 * Set the JS to be executed after the form is saved.
 	 *
 	 * Please use the buildSuccessScript method before v1.6.5.
 	 * 
	 * @return string|void
	 */
	protected function addSavedScript()
	{
	    return <<<JS
		// data Return data for the interface
		if (! data.status) {
			Dcat.error(data.message);

			return false;
		}

		Dcat.success(data.message);

		if (data.redirect) {
			Dcat.reload(data.redirect)
		}

		// Abort follow-up logic (default logic)
		return false;
JS;
	}

	/**
	 * Set the JS to be executed after a failed form save
 	 * 
 	 * Please use the buildErrorScript method before v1.6.5. 
	 * 
	 * @return string|void
	 */
	protected function addErrorScript()
	{
		return <<<JS
		var errorData = JSON.parse(response.responseText);
		
		if (errorData) {
			Dcat.error(errorData.message);
		} else {
			console.log('submission error', response.responseText);
		}
		
		// Abort follow-up logic (default logic)
		return false;
JS;
	}
}
```

### Cancel/enable Ajax submission of forms

By default, all form submissions are done using Ajax, which is controlled by injecting [Form.js](https://github.com/jqhph/dcat-admin/blob/2.0/resources/assets/dcat/js/extensions/Form.js). This feature can be disabled if native form behavior is required.

```php
$form->ajax(false);
```

<a name="layout"></a>
### Layout

`column` multi-column layout
```php
<?php

use Dcat\Admin\Widgets\Form;

class Setting extends Form
{
    public function form()
    {
        $this->column(6, function () {
            $this->text('text1');
            
            ...
        });
        
        $this->column(6, function () {
            $this->text('text2');
            
            ...
        });
    }    
}
```

`tab` tab layout
```php
<?php

use Dcat\Admin\Widgets\Form;

class Setting extends Form
{
    public function form()
    {
        $this->tab('Tab 1', function () {
            $this->text('text1');
            
            ...
        });
        
        $this->tab('Tab 2', function () {
            $this->text('text2');
            
            ...
        });
    }    
}
```

`row` multi-line layout
```php
public function form()
{
    $this->row(function ($row) {
        $row->width(3)->text('text1');
        
        ...
    });
    
    $this->row(function ($row) {
        $row->width(3)->text('text2');
        
        ...
    });
} 
```


<a name="modal"></a>
### Display in a pop-up window

#### basic Usage

Use the command generator tool form `php artisan admin:form ResetPassword`, then modify the form file as follows

```php
<?php

namespace App\Admin\Forms;

use Dcat\Admin\Widgets\Form;

class ResetPassword extends Form
{
    // Processing of requests
    public function handle(array $input)
    {
        $password = $input['password'] ?? null;

        // logical operation

        return $this->response()->success('Password change success');
    }

    public function form()
    {
        $this->password('password')->required();
        // Password confirmation form
        $this->password('password_confirm')->same('password');
    }

    // Return form data, delete this method if you don't want it.
    public function default()
    {
        return [
            'password'         => '',
            'password_confirm' => '',
        ];
    }
}
```

use

```php
use App\Admin\Forms\ResetPassword;
use Dcat\Admin\Widgets\Modal;

$modal = Modal::make()
	->lg()
	->title('change password')
	->body(ResetPassword::make())
	->button('change password');
```

#### asynchronous loading
     
Just make the `Form` form class implement the `Dcat\Admin\Contracts\LazyRenderable` interface to support the asynchronous rendering feature, and modify the tool form class created above as follows

```php
<?php

namespace App\Admin\Forms;

use Dcat\Admin\Widgets\Form;
use Dcat\Admin\Traits\LazyWidget;
use Dcat\Admin\Contracts\LazyRenderable;

class ResetPassword extends Form implements LazyRenderable
{
    use LazyWidget; 
    
    // Processing of requests
	public function handle(array $input)
	{
	    // Get external passing parameters
	    $id = $this->payload['id'] ?? null;
	    
		$password = $input['password'] ?? null;

		// logical operation

		return $this->response()->success('密码修改success');
	}

	public function form()
	{
	    // Get external passing parameters
		$id = $this->payload['id'] ?? null;
	    
		$this->password('password')->required();
		// Password confirmation form
		$this->password('password_confirm')->same('password');
	}

	// Return form data, delete this method if you don't want it.
	public function default()
	{
	    // Get external passing parameters
		$id = $this->payload['id'] ?? null;
	    
		return [
			'password'         => '',
			'password_confirm' => '',
		];
	}
}
```

The code is basically the same as above, and we can use the `payload` method to pass custom parameters into the form.

```php
use App\Admin\Forms\ResetPassword;
use Dcat\Admin\Widgets\Modal;

$modal = Modal::make()
	->lg()
	->title('change password')
	->body(ResetPassword::make()->payload(['id' => '...'])) // Passing custom parameters
	->button('change password');
```



#### table row manipulation pop-up

The following shows the pop-up combining tool form's Usages via a data form change password row action function.


Use the command generator tool form `php artisan admin:form ResetPassword`, and then modify the form file as follows

```php
<?php

namespace App\Admin\Forms;

use Dcat\Admin\Models\Administrator;
use Dcat\Admin\Traits\LazyWidget;
use Dcat\Admin\Widgets\Form;
use Dcat\Admin\Contracts\LazyRenderable;

class ResetPassword extends Form implements LazyRenderable
{
    use LazyWidget; // Using the asynchronous load function
    
    // Processing of requests
    public function handle(array $input)
    {
        // Get external passing parameters
        $id = $this->payload['id'] ?? null;
        
        // Form parameters
        $password = $input['password'] ?? null;

        if (! $id) {
            return $this->response()->error('Parameter errors');
        }

        $user = Administrator::query()->find($id);

        if (! $user) {
            return $this->response()->error('user does not exist');
        }

        $user->update(['password' => bcrypt($password)]);

        return $this->response()->success('Password change success');
    }

    public function form()
    {
        // Get external passing parameters
		//$id = $this->payload['id'] ?? null;
        
        $this->password('password')->required();
        // Password confirmation form
        $this->password('password_confirm')->same('password');
    }

    // Return form data, delete this method if you don't want it.
    public function default()
    {
        return [
            'password'         => '',
            'password_confirm' => '',
        ];
    }
}
```

Then run the `php artisan admin:action` command, select option `2`, generate the data form row action class, and modify the following.

```php
<?php

namespace App\Admin\Actions\Grid;

use App\Admin\Forms\ResetPassword as ResetPasswordForm;
use Dcat\Admin\Widgets\Modal;
use Dcat\Admin\Grid\RowAction;

class ResetPassword extends RowAction
{
    protected $title = '修改密码';
    
    public function render()
    {
        // Instantiating a form class and passing custom parameters
        $form = ResetPasswordForm::make()->payload(['id' => $this->getKey()]);
        
        return Modal::make()
        	->lg()
        	->title($this->title)
        	->body($form)
        	->button($this->title);
    }
}
```

use

```php
use App\Admin\Actions\Grid\ResetPassword;

$grid->actions([new ResetPassword()]);
```

result

![]({{public}}/assets/img/screenshots/modal-widget-form.png)


<a name="batch-modal"></a>
#### Table Batch Operations pop-up

If you want to use form pop-ups in the Bulk Action button, consider the following example.


Here we still use the `App\Admin\Forms\ResetPassword` form used above, and modify it as follows

```php
<?php

namespace App\Admin\Forms;

use Dcat\Admin\Models\Administrator;
use Dcat\Admin\Widgets\Form;
use Dcat\Admin\Traits\LazyWidget;
use Dcat\Admin\Contracts\LazyRenderable;

class ResetPassword extends Form implements LazyRenderable
{
    use LazyWidget;
    
    // Processing of requests
    public function handle(array $input)
    {
	    // ids to array
	    $id = explode(',', $input['id'] ?? null);
	    $password = $input['password'] ?? null;

	    if (! $id) {
		    return $this->response()->error('Parameter errors');
	    }

	    $users = Administrator::query()->find($id);

	    if ($users->isEmpty()) {
		    return $this->response()->error('user does not exist');
	    }

	    // Here it is changed to loop batch modification
	    $users->each(function ($user) use ($password) {
	 	    $user->update(['password' => bcrypt($password)]);
	    });

	    return $this->response()->success('密码修改success');
    }

    public function form()
    {
        $this->password('password')->required();
        // Password confirmation form
        $this->password('password_confirm')->same('password');
   
        // Set hidden form to pass user id
        $this->hidden('id')->attribute('id', 'reset-password-id');
    }
   
    // Return form data, delete this method if you don't want it.
    public function default()
    {
        return [
            'password'         => '',
            'password_confirm' => '',
        ];
    }
}
```

Then run the `php artisan admin:action` command and select option `1` to generate the data form batch action class and modify the following.

```php
<?php

namespace App\Admin\Actions\Grid;

use App\Admin\Forms\ResetPassword as ResetPasswordForm;
use Dcat\Admin\Widgets\Modal;
use Dcat\Admin\Grid\BatchAction;

class BatchResetPassword extends BatchAction
{
    protected $title = '修改密码';
    
    public function render()
    {
        // Instantiation Form Class
		$form = ResetPasswordForm::make();
		
		return Modal::make()
			->lg()
			->title($this->title)
			->body($form)
			// Because the form is loaded asynchronously, you must use the onLoad method
			// If the form is loaded non-asynchronously, you need to change to the onShow method.
			->onLoad($this->getModalScript())
			->button($this->title);
    }

    protected function getModalScript()
    {
        // Write batch selected row IDs to the hidden id form after the popup is displayed.
        return <<<JS
// Get the selected array of IDs
var key = {$this->getSelectedKeysScript()}

$('#reset-password-id').val(key);
JS;
	}
}
```

use

```php
use App\Admin\Actions\Grid\BatchResetPassword;

$grid->batchActions([new BatchResetPassword()]);
```



