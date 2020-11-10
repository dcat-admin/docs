# 数据详情动作

运行命令

```bash
php artisan admin:action
```

然后输入 `5` 

```bash
 Which type of action would you like to make?:
  [0] default
  [1] grid-batch
  [2] grid-row
  [3] grid-tool
  [4] form-tool
  [5] show-tool
  [6] tree-tool
 > 5 # 输入 5
 
```

接着输入 `Action` 类名称，这里需要输入 `大驼峰` 风格的英文字母

```bash

 Please enter a name of action class:
 > Copy

```

类名输入完成之后会出现以下信息让开发者输入类的命名空间，默认的命名空间是 `App\Admin\Actions\Show`，这里使用默认的就行

```bash

 Please enter the namespace of action class [App\Admin\Actions\Show]:
 > 

```

最后生成文件如下

```php
<?php

namespace App\Admin\Actions\Show;

use Dcat\Admin\Actions\Response;
use Dcat\Admin\Show\AbstractTool;
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
        // 获取主键
        $key = $this->getKey();
        
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
        // 获取主键
        $key = $this->getKey();
        
        // 获取当前页其他字段
        $username = $this->parent->model()->username;
        
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
$show->tools(function (Show\Tools $tools) {
    $tools->append(new Copy());
});
```
