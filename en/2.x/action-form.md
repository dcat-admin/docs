# Data form actions

Run command

```bash
php artisan admin:action
```

Then enter `4` 

```bash
 Which type of action would you like to make?:
  [0] default
  [1] grid-batch
  [2] grid-row
  [3] grid-tool
  [4] form-tool
  [5] show-tool
  [6] tree-tool
 > 4 # Enter 4
 
```

Next, type in the name of the `Action` class in `PascalCase` style.

```bash

 Please enter a name of action class:
 > Copy

```

The default namespace is `App\Admin\ActionsForm`, just use the default here.

```bash

 Please enter the namespace of action class [App\Admin\Actions\Form]:
 > 

```

The final file is as follows

```php
<?php

namespace App\Admin\Actions\Form;

use Dcat\Admin\Actions\Response;
use Dcat\Admin\Form\AbstractTool;
use Dcat\Admin\Traits\HasPermissions;
use Illuminate\Contracts\Auth\Authenticatable;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Http\Request;

class Copy extends AbstractTool
{
    /**
     * Button Title
     *
     * @return string
     */
	protected $title = 'Title';

    /**
     * Process the request, if you don't need the interface to process it, simply delete this method.
     *
     * @param Request $request
     *
     * @return Response
     */
    public function handle(Request $request)
    {
        // Get Primary Key
        $key = $this->getKey();
        
        return $this->response()
            ->success('Processed successfully.')
            ->redirect('/');
    }

    /**
     * If it's just a tag jump, then just return the jump link here
     * 
     * @return string|void
     */
    protected function href()
    {
        // Get Primary Key
        $key = $this->getKey();
        
        // Get other fields of the current page
        $username = $this->parent->model()->username;
        
        // return admin_url('auth/users');
    }
    
     // If you want to customize the HTML of the action button, you can rewrite this method
    public function html()
    {
        return parent::html();
    }

    /**
     * Confirm the pop-up message. Delete this method if you don't need it. 
     * 
	 * @return string|array|void
	 */
	public function confirm()
	{
		// return ['Confirm?', 'contents'];
	}

    /**
     * Check Permission. If you do not need this method it can be deleted. 
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
     * Returns the parameters of the requesting interface, or removes this method if not required.
     * 
     * @return array
     */
    protected function parameters()
    {
        return [];
    }
}

```

# Usage

```php
$form->tools(function (Form\Tools $tools) {
    $tools->append(new Copy());
});
```
