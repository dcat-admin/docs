# 模型树动作

### 工具栏

运行命令

```bash
php artisan admin:action
```

然后输入 `7` 

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

 > 7 # 输入 7
```

接着输入 `Action` 类名称，这里需要输入 `大驼峰` 风格的英文字母

```bash

 Please enter a name of action class:
 > Copy

```

类名输入完成之后会出现以下信息让开发者输入类的命名空间，默认的命名空间是 `App\Admin\Actions\Tree`，这里使用默认的就行

```bash

 Please enter the namespace of action class [App\Admin\Actions\Tree]:
 > 

```

最后生成文件如下

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
     * 按钮标题
     *
     * @return string
     */
	protected $title = 'Title';

    /**
     * 处理请求，如果不需要接口处理，请直接删除这个方法
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
     * 如果只是a标签跳转，则在这里返回跳转链接即可
     * 
     * @return string|void
     */
    protected function href()
    {
        // return admin_url('auth/users');
    }
    
     // 如果你想自定义动作按钮的HTML，可以重写此方法
    public function html()
    {
        return parent::html();
    }

    /**
     * 确认弹窗信息，如不需要可以删除此方法 
     * 
	 * @return string|array|void
	 */
	public function confirm()
	{
		// return ['Confirm?', 'contents'];
	}

    /**
     * 权限判断，如不需要可以删除此方法 
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
     * 返回请求接口的参数，如不需要可以删除此方法
     * 
     * @return array
     */
    protected function parameters()
    {
        return [];
    }
}

```

使用

```php
$tree->tools(function (Tree\Tools $tools) {
    $tools->add(new Copy());
});
```

<a name="row-action"></a>
### 行操作

运行命令

```bash
php artisan admin:action
```

然后输入 `6` 

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

 > 6 # 输入 6
```

接着输入 `Action` 类名称，这里需要输入 `大驼峰` 风格的英文字母

```bash

 Please enter a name of action class:
 > Copy

```

类名输入完成之后会出现以下信息让开发者输入类的命名空间，默认的命名空间是 `App\Admin\Actions\Tree`，这里使用默认的就行

```bash

 Please enter the namespace of action class [App\Admin\Actions\Tree]:
 > 

```

最后生成文件如下

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
     * 处理请求，如果不需要接口处理，请直接删除这个方法 
     *
     * @param Request $request
     *
     * @return Response
     */
    public function handle(Request $request)
    {
    	// 获取主键
    	$key = $this->getKey();

        return $this->response()
            ->success('Processed successfully.')
            ->redirect('/');
    }

    /**
     * 返回跳转地址，如果不需要可以删除
     * 
     * @return string|void
     */
    protected function href()
    {
        // return admin_url('auth/users/'.$this->getKey());
    }

    /**
     * 确认弹窗信息，如果不需要可以删除 
     * 
	 * @return string|array|void
	 */
	public function confirm()
	{
		// return ['Confirm?', 'contents'];
	}

    /**
     * 权限判断，如果不需要可以删除  
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

使用

```php
$tree->actions(new Copy());
```
