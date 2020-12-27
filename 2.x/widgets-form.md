# 工具表单

工具表单(`Dcat\Admin\Widgets\Form`)用来构建表单和处理提交数据，可以很方便的独立处理数据，而不需要额外注册路由。


### 基本使用
用命令`admin:form`来生成表单类文件：

```bash
php artisan admin:form Setting
```
将会生成表单文件`app/Admin/Forms/Setting.php`

```php
<?php

namespace App\Admin\Forms;

use Dcat\Admin\Widgets\Form;
use Symfony\Component\HttpFoundation\Response;

class Setting extends Form
{
    // 处理表单提交请求
    public function handle(array $input)
    {
        // dump($input);

        return $this->response()->success('Processed successfully.')->refresh();
    }

    // 构建表单
    public function form()
    {
        // 弹出确认弹窗 
        $this->confirm('您确定要提交表单吗', 'content');
        
        $this->text('name')->required();
        $this->email('email')->rules('email');
    }

    /**
     * 返回表单数据，如不需要可以删除此方法
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
在上面的表单类里面，在`form`方法中构建表单项，使用方法和[数据表单](model-form.md)一致，`default`方法用来给这个表单项设置默认数据。

在页面中填入数据提交表单之后，请求会进入到`handle`方法中，在这里可以加入你的数据处理逻辑，处理完成之后可以通过`success`或`error`方法响应数据到前端：
```php
    public function handle(array $input)
    {
        // $input是你接收到的表单数据
        // 在这里可以写你的处理逻辑


        return $this->response()->success('Processed successfully.')->refresh();
    }
```

然后按照下面的方法将上面的表单放到你的页面中：

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
            ->title('网站设置')
            ->body(new Card(new Setting()));
    }
}
```

### 弹出确认弹窗

第二个参数可忽略

```php
$this->confirm('title', 'content');
```


### 响应方法

参考 [动作以及表单响应](response.md) 章节


### 自定义表单保存的后续行为

```php
<?php

namespace App\Admin\Forms;

use Dcat\Admin\Widgets\Form;
use Symfony\Component\HttpFoundation\Response;

class Setting extends Form
{
    ...
    
    /**
 	 * 设置表单保存成功后执行的JS
 	 *
	 * @return string|void
	 */
	protected function savedScript()
	{
	    return <<<JS
		// data 为接口返回数据
		if (! data.status) {
			Dcat.error(data.message);

			return false;
		}

		Dcat.success(data.message);

		if (data.redirect) {
			Dcat.reload(data.redirect)
		}

		// 中止后续逻辑（默认逻辑）
		return false;
JS;
	}

	/**
	 * 设置表单保存失败后执行的JS
 	 * 
	 * @return string|void
	 */
	protected function errorScript()
	{
		return <<<JS
		var errorData = JSON.parse(response.responseText);
		
		if (errorData) {
			Dcat.error(errorData.message);
		} else {
			console.log('提交出错', response.responseText);
		}
		
		// 终止后续逻辑执行（默认逻辑）
		return false;
JS;
	}
}
```

### 取消/启用表单的Ajax提交

默认表单提交都使用Ajax进行提交,通过注入的 [Form.js](https://github.com/jqhph/dcat-admin/blob/2.0/resources/assets/dcat/js/extensions/Form.js) 进行流程控制,如果需要原生表单行为,可以禁用此特

```php
$form->ajax(false);
```

<a name="layout"></a>
### 布局

`column`多列布局
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

`tab`选项卡布局
```php
<?php

use Dcat\Admin\Widgets\Form;

class Setting extends Form
{
    public function form()
    {
        $this->tab('选项卡1', function () {
            $this->text('text1');
            
            ...
        });
        
        $this->tab('选项卡2', function () {
            $this->text('text2');
            
            ...
        });
    }    
}
```

`row`多行布局
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
### 在弹窗中显示

#### 基本用法

使用命令生成工具表单`php artisan admin:form ResetPassword`，然后修改表单文件如下

```php
<?php

namespace App\Admin\Forms;

use Dcat\Admin\Widgets\Form;

class ResetPassword extends Form
{
    // 处理请求
    public function handle(array $input)
    {
        $password = $input['password'] ?? null;

        // 逻辑操作

        return $this->response()->success('密码修改成功');
    }

    public function form()
    {
        $this->password('password')->required();
        // 密码确认表单
        $this->password('password_confirm')->same('password');
    }

    // 返回表单数据，如不需要可以删除此方法
    public function default()
    {
        return [
            'password'         => '',
            'password_confirm' => '',
        ];
    }
}
```

使用

```php
use App\Admin\Forms\ResetPassword;
use Dcat\Admin\Widgets\Modal;

$modal = Modal::make()
	->lg()
	->title('修改密码')
	->body(ResetPassword::make())
	->button('修改密码');
```

#### 异步加载

只需要让`Form`表单类实现`Dcat\Admin\Contracts\LazyRenderable`接口即可支持异步渲染功能，修改上面创建的工具表单类如下

```php
<?php

namespace App\Admin\Forms;

use Dcat\Admin\Widgets\Form;
use Dcat\Admin\Traits\LazyWidget;
use Dcat\Admin\Contracts\LazyRenderable;

class ResetPassword extends Form implements LazyRenderable
{
    use LazyWidget; 
    
    // 处理请求
	public function handle(array $input)
	{
	    // 获取外部传递参数
	    $id = $this->payload['id'] ?? null;
	    
		$password = $input['password'] ?? null;

		// 逻辑操作

		return $this->response()->success('密码修改成功');
	}

	public function form()
	{
	    // 获取外部传递参数
		$id = $this->payload['id'] ?? null;
	    
		$this->password('password')->required();
		// 密码确认表单
		$this->password('password_confirm')->same('password');
	}

	// 返回表单数据，如不需要可以删除此方法
	public function default()
	{
	    // 获取外部传递参数
		$id = $this->payload['id'] ?? null;
	    
		return [
			'password'         => '',
			'password_confirm' => '',
		];
	}
}
```

使用代码与上面基本一致，并且我们可以用`payload`方法往表单里面传递自定义参数

```php
use App\Admin\Forms\ResetPassword;
use Dcat\Admin\Widgets\Modal;

$modal = Modal::make()
	->lg()
	->title('修改密码')
	->body(ResetPassword::make()->payload(['id' => '...'])) // 传递自定义参数
	->button('修改密码');
```



#### 表格行操作弹窗

下面通过一个数据表格修改密码的行操作功能来展示弹窗结合工具表单的用法：


使用命令生成工具表单`php artisan admin:form ResetPassword`，然后修改表单文件如下

```php
<?php

namespace App\Admin\Forms;

use Dcat\Admin\Models\Administrator;
use Dcat\Admin\Traits\LazyWidget;
use Dcat\Admin\Widgets\Form;
use Dcat\Admin\Contracts\LazyRenderable;

class ResetPassword extends Form implements LazyRenderable
{
    use LazyWidget; // 使用异步加载功能
    
    // 处理请求
    public function handle(array $input)
    {
        // 获取外部传递参数
        $id = $this->payload['id'] ?? null;
        
        // 表单参数
        $password = $input['password'] ?? null;

        if (! $id) {
            return $this->response()->error('参数错误');
        }

        $user = Administrator::query()->find($id);

        if (! $user) {
            return $this->response()->error('用户不存在');
        }

        $user->update(['password' => bcrypt($password)]);

        return $this->response()->success('密码修改成功');
    }

    public function form()
    {
        // 获取外部传递参数
		//$id = $this->payload['id'] ?? null;
        
        $this->password('password')->required();
        // 密码确认表单
        $this->password('password_confirm')->same('password');
    }

    // 返回表单数据，如不需要可以删除此方法
    public function default()
    {
        return [
            'password'         => '',
            'password_confirm' => '',
        ];
    }
}
```

然后运行`php artisan admin:action`命令，选择选项`2`，生成数据表格行操作类，并修改如下：

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
        // 实例化表单类并传递自定义参数
        $form = ResetPasswordForm::make()->payload(['id' => $this->getKey()]);
        
        return Modal::make()
        	->lg()
        	->title($this->title)
        	->body($form)
        	->button($this->title);
    }
}
```

使用

```php
use App\Admin\Actions\Grid\ResetPassword;

$grid->actions([new ResetPassword()]);
```

效果

![]({{public}}/assets/img/screenshots/modal-widget-form.png)


<a name="batch-modal"></a>
#### 表格批量操作弹窗

如果你想在批量操作按钮中使用表单弹窗，可以参考以下例子：


这里我们仍然沿用上面用到的`App\Admin\Forms\ResetPassword`表单，并修改如下

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
    
    // 处理请求
    public function handle(array $input)
    {
	    // id转化为数组
	    $id = explode(',', $input['id'] ?? null);
	    $password = $input['password'] ?? null;

	    if (! $id) {
		    return $this->response()->error('参数错误');
	    }

	    $users = Administrator::query()->find($id);

	    if ($users->isEmpty()) {
		    return $this->response()->error('用户不存在');
	    }

	    // 这里改为循环批量修改
	    $users->each(function ($user) use ($password) {
	 	    $user->update(['password' => bcrypt($password)]);
	    });

	    return $this->response()->success('密码修改成功');
    }

    public function form()
    {
        $this->password('password')->required();
        // 密码确认表单
        $this->password('password_confirm')->same('password');
   
        // 设置隐藏表单，传递用户id
        $this->hidden('id')->attribute('id', 'reset-password-id');
    }
   
    // 返回表单数据，如不需要可以删除此方法
    public function default()
    {
        return [
            'password'         => '',
            'password_confirm' => '',
        ];
    }
}
```

然后运行`php artisan admin:action`命令，选择选项`1`，生成数据表格批量操作类，并修改如下：

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
        // 实例化表单类
		$form = ResetPasswordForm::make();
		
		return Modal::make()
			->lg()
			->title($this->title)
			->body($form)
			// 因为此处使用了表单异步加载功能，所以一定要用 onLoad 方法
			// 如果是非异步方式加载表单，则需要改成 onShow 方法
			->onLoad($this->getModalScript())
			->button($this->title);
    }

    protected function getModalScript()
    {
        // 弹窗显示后往隐藏的id表单中写入批量选中的行ID
        return <<<JS
// 获取选中的ID数组
var key = {$this->getSelectedKeysScript()}

$('#reset-password-id').val(key);
JS;
	}
}
```

使用

```php
use App\Admin\Actions\Grid\BatchResetPassword;

$grid->batchActions([new BatchResetPassword()]);
```



