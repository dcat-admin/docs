# 模态窗 (Modal)

> {tip} Since `v1.7.0`

基本使用

```php
use Dcat\Admin\Widgets\Modal;

$modal = Modal::make()
	->lg()
	->title('标题')
	->body(view(...))
	->button('<button class="btn btn-primary">点击打开弹窗</button>');
	
return view(..., ['modal' => $modal]);	
```

## 功能

### 标题 (title)

设置弹窗标题

```php
$modal->title('标题');
```

### 内容 (body)

设置弹窗内容，此方法接受一个参数，允许传入`string`、`Cloure`、`Illuminate\Contracts\Support\Renderable`以及`Dcat\Admin\Contracts\LazyRenderable`类型值

```php
// 传入字符串
$modal->body('字符串');

// 传入闭包，注意闭包必须返回字符串类型值或空值
$modal->body(function () {
	return view(...)->render();
});

// 传入 Renderable
use Dcat\Admin\Widgets\

$modal->body(view(...));
$modal->body(Card::make());

// 传入 LazyRenderable
$modal->body(PostTable::make());
```

### 底部内容 (footer)
设置弹窗底部内容，此方法接受一个参数，允许传入`string`、`Cloure`、`Illuminate\Contracts\Support\Renderable`以及`Dcat\Admin\Contracts\LazyRenderable`类型值，用法同上

```php
$modal->footer('字符串');

$modal->footer(view(...));
```

### 尺寸 

默认 `500px`

```php
// 800px
$modal->lg();

// 1140px
$modal->xl();
```

### 按钮 (button)

设置按钮

### 事件监听

支持事件

 - `onShow` 弹窗显示事件
 - `onShown` 弹窗已显示事件
 - `onHide` 弹窗隐藏事件
 - `onHidden` 弹窗已隐藏事件
 
用法示例

```php
use Dcat\Admin\Admin;

$modal->onShow(
	<<<JS
console.log('显示弹窗', target, $(this));	
JS
);

$modal->onHide(
	<<<JS
console.log('隐藏弹窗', target, $(this));	
JS
);
``` 
 


<a name="form"></a>
## 表单弹窗

通过`Modal`实现表单弹窗功能非常地简单和方便，下面是一个简单的例子


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

        return $this->success('密码修改成功');
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

### 异步加载

上面的例子中表单是同步渲染到页面中的，如果表单比较多或者是涉及的逻辑比较复杂，则对页面的加载速度会有影响，这是可以同过异步加载功能避免这个问题，用法如下：


修改上面创建的工具表单类如下

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

		return $this->success('密码修改成功');
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


