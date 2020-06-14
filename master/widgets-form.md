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
    /**
     * Handle the form request.
     *
     * @param array $input
     *
     * @return Response
     */
    public function handle(array $input)
    {
        // dump($input);

        // return $this->error('Your error message.');

        return $this->success('Processed successfully.', '/');
    }

    /**
     * Build a form here.
     */
    public function form()
    {
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


        // 第一个参数是响应的成功信息，第二个参数是要跳转的路由
        return $this->success('Processed successfully.', '/');
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

### 自定义表单保存的后续行为

> {tip} Since 1.2.0

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
	protected function buildSuccessScript()
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
	protected function buildErrorScript()
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

<a name="modal"></a>
### 在弹窗中显示

工具表单是支持在`modal`弹窗中显示的，可以结合[动作(action)](action.md)一起使用。


#### 行操作弹窗
下面通过一个数据表格修改密码的行操作功能来展示弹窗结合工具表单的用法：


使用命令生成工具表单`php artisan admin:form ResetPassword`，然后修改表单文件如下

```php
<?php

namespace App\Admin\Forms;

use Dcat\Admin\Models\Administrator;
use Dcat\Admin\Widgets\Form;
use Symfony\Component\HttpFoundation\Response;

class ResetPassword extends Form
{
    // 增加一个自定义属性保存用户ID
    protected $id;

    // 构造方法的参数必须设置默认值
    public function __construct($id = null)
    {
        $this->id = $id;

        parent::__construct();
    }

    // 处理请求
    public function handle(array $input)
    {
        $id = $input['id'] ?? null;
        $password = $input['password'] ?? null;

        if (! $id) {
            return $this->error('参数错误');
        }

        $user = Administrator::query()->find($id);

        if (! $user) {
            return $this->error('用户不存在');
        }

        $user->update(['password' => bcrypt($password)]);

        return $this->success('密码修改成功');
    }

    public function form()
    {
        $this->password('password')->required();
        // 密码确认表单
        $this->password('password_confirm')->same('password');

        // 设置隐藏表单，传递用户id
        $this->hidden('id')->value($this->id);
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
use Dcat\Admin\Admin;
use Dcat\Admin\Grid\RowAction;

class ResetPassword extends RowAction
{
    public function render()
    {
        $id = "reset-pwd-{$this->getKey()}";

        // 模态窗
        $this->modal($id);

        return <<<HTML
<span class="grid-expand" data-toggle="modal" data-target="#{$id}">
   <a href="javascript:void(0)">修改密码</a>
</span>
HTML;
    }

    protected function modal($id)
    {
        // 工具表单
        $form = new ResetPasswordForm($this->getKey());

        // 在弹窗标题处显示当前行的用户名
        $username = $this->row->username;

        // 刷新页面时移除模态窗遮罩层
        // 从 v1.5.0 版本开始可以移除这段 JS 代码
        Admin::script('Dcat.onPjaxComplete(function () {
            $(".modal-backdrop").remove();
            $("body").removeClass("modal-open");
        }, true)');

        // 通过 Admin::html 方法设置模态窗HTML
        Admin::html(
            <<<HTML
<div class="modal fade" id="{$id}">
  <div class="modal-dialog modal-lg" role="document">
    <div class="modal-content">
      <div class="modal-header">
        <h4 class="modal-title">修改密码 - {$username}</h4>
         <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span aria-hidden="true">&times;</span></button>
      </div>
      <div class="modal-body">
        {$form->render()}
      </div>
    </div>
  </div>
</div>
HTML
        );
    }
}
```

使用

```php
use App\Admin\Actions\Grid\ResetPassword;

$grid->actions(new ResetPassword());
```

效果

<a href="{{public}}/assets/img/screenshots/modal-widget-form.png" target="_blank">
    <img class="img img-full" src="{{public}}/assets/img/screenshots/modal-widget-form.png">
</a>


<a name="batch-modal"></a>
#### 批量操作弹窗

如果你想在批量操作按钮中使用表单弹窗，可以参考以下例子：


这里我们仍然沿用上面用到的`App\Admin\Forms\ResetPassword`表单，并修改如下

```php
<?php

namespace App\Admin\Forms;

use Dcat\Admin\Models\Administrator;
use Dcat\Admin\Widgets\Form;

class ResetPassword extends Form
{
    // 增加一个自定义属性保存用户ID
       protected $id;
   
       // 构造方法的参数必须设置默认值
       public function __construct($id = null)
       {
           $this->id = $id;
   
           parent::__construct();
       }
   
       // 处理请求
       public function handle(array $input)
       {
           // id转化为数组
           $id = explode(',', $input['id'] ?? null);
           $password = $input['password'] ?? null;
   
           if (! $id) {
               return $this->error('参数错误');
           }
   
           $users = Administrator::query()->find($id);
   
           if ($users->isEmpty()) {
               return $this->error('用户不存在');
           }
   
           // 这里改为循环批量修改
           $users->each(function ($user) use ($password) {
               $user->update(['password' => bcrypt($password)]);
           });
   
           return $this->success('密码修改成功');
       }
   
       public function form()
       {
           $this->password('password')->required();
           // 密码确认表单
           $this->password('password_confirm')->same('password');
   
           // 设置隐藏表单，传递用户id，这里需要加上id属性
            $this->hidden('id')->attribute('id', 'reset-password-id')->value($this->id);
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
use Dcat\Admin\Admin;
use Dcat\Admin\Grid\BatchAction;

class BatchResetPassword extends BatchAction
{
    public function render()
    {
        $id = "batch-reset-pwd";

        $this->modal($id);

        $this->addScript($id);

        return <<<HTML
<span data-toggle="modal" data-target="#{$id}">
   <a href="javascript:void(0)">修改密码</a>
</span>
HTML;
    }

    protected function addScript($id)
    {
        // 弹窗显示后往隐藏的id表单中写入批量选中的行ID
        Admin::script(
            <<<JS
$('#$id').on('shown.bs.modal', function () {
    // 获取选中的ID数组
    var key = {$this->getSelectedKeysScript()}
    
    $('#reset-password-id').val(key);
});
JS
        );
    }

    protected function modal($id)
    {
        // 表单
        $form = new ResetPasswordForm();

        // 刷新页面时移除模态窗遮罩层
        // 从 v1.5.0 版本开始可以移除这段 JS 代码
        Admin::script('Dcat.onPjaxComplete(function () {
            $(".modal-backdrop").remove();
            $("body").removeClass("modal-open");
        }, true)');

        Admin::html(
            <<<HTML
<div class="modal fade" id="{$id}">
  <div class="modal-dialog modal-lg" role="document">
    <div class="modal-content">
      <div class="modal-header">
        <h4 class="modal-title">修改密码</h4>
         <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span aria-hidden="true">&times;</span></button>
      </div>
      <div class="modal-body">
        {$form->render()}
      </div>
    </div>
  </div>
</div>
HTML
        );
    }
}
```

使用

```php
use App\Admin\Actions\Grid\BatchResetPassword;

$grid->batchActions(new BatchResetPassword());
```
