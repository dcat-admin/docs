# Model tree action

### Toolbar

Run command

```bash
php artisan admin:action
```

Then enter `7`. 

```bash
 Which type of action would you like to make?:
  [0] default
  [1] grid-batch
  [2] grid-row
  [3] grid-tool
  [4] form-tool
  [5] show-tool
  [6] tree-row
  [7] tree-tool

 > 7 # Enter 7
 
```

Next, type in the name of the `Action` class in `PascalCase` style.

```bash

 Please enter a name of action class:
 > Copy

```

The default namespace is `App\Admin\Actions\Tree`, just use the default one here.

```bash

 Please enter the namespace of action class [App\Admin\Actions\Tree]:
 > 

```

The final file is as follows

```php
<?php

namespace App\Admin\Actions\Tree;

use Dcat\Admin\Actions\Response;
use Dcat\Admin\Tree\AbstractTool;
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
        // return admin_url('auth/users');
    }
    
     // If you want to customize the HTML of the action button, you can rewrite this method
    public function html()
    {
        return parent::html();
    }

    /**
     * Confirm the pop-up message, and delete this method if you don't need it. 
     * 
	 * @return string|array|void
	 */
	public function confirm()
	{
		// return ['Confirm?', 'contents'];
	}

    /**
     * Check Permissions. If you do not need this method can be deleted. 
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

Usage

```php
$tree->tools(function (Tree\Tools $tools) {
    $tools->add(new Copy());
});
```

<a name="row-action"></a>

### Line operations

Run Command

```bash
php artisan admin:action
```

Then enter `6`
```bash
 Which type of action would you like to make?:
  [0] default
  [1] grid-batch
  [2] grid-row
  [3] grid-tool
  [4] form-tool
  [5] show-tool
  [6] tree-row
  [7] tree-tool

 > 6 # Enter 6
```
Next, enter the name of the `Action` class, here you need to enter the letters of the `Big Hump` style.

```bash

 Please enter a name of action class:
 > Copy

```

After the class name is entered, the following message will appear for the developer to enter the namespace of the class, the default namespace is `App\Admin\Actions\Tree`.

```bash

 Please enter the namespace of action class [App\Admin\Actions\Tree]:
 > 

```

The final file is generated as follows

```php
<?php

namespace App\Admin\Actions\Tree;

use Dcat\Admin\Tree\RowAction;
use Dcat\Admin\Actions\Response;
use Dcat\Admin\Traits\HasPermissions;
use Illuminate\Contracts\Auth\Authenticatable;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Http\Request;

class Copy extends RowAction
{
    /**
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
    	// Get the primary key 
    	$key = $this->getKey();

        return $this->response()
            ->success('Processed successfully.')
            ->redirect('/');
    }

    /**
     * Returns the jump address, which can be deleted if not needed.
     * 
     * @return string|void
     */
    protected function href()
    {
        // return admin_url('auth/users/'.$this->getKey());
    }

    /**
     * Acknowledge pop-up messages and delete them if they are not needed. 
     * 
	 * @return string|array|void
	 */
	public function confirm()
	{
		// return ['Confirm?', 'contents'];
	}

    /**
     * Permissions are checked here and can be deleted if not needed.
     * 
     * @param Model|Authenticatable|HasPermissions|null $user
     *
     * @return bool
     */
    protected function authorize($user): bool
    {
        return true;
    }
}
```

Usage

```php
$tree->actions(new Copy());
```