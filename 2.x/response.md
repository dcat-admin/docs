# 动作以及表单响应

[动作](action.md)、[数据表单](model-form.md)以及[工具表单](widgets-form.md)的响应方法都是同一套方法。

在类中可以通过 `$this->response()` 获取到 `Dcat\Admin\Http\JsonResponse`对象并响应数据到前端

```php
return $this->response()->success('成功！');

// 等同于
use Dcat\Admin\Admin;
use Dcat\Admin\Http\JsonResponse;

return JsonResponse::make()->success('成功！');

return Admin::make()->success('成功！');
```

如果是在控制器中使用，需要加上`send`方法

```php
public function index()
{
    return JsonResponse::make()->success('成功！')->send();
}
```


### 功能
下面介绍一下 `JsonResponse` 的主要用法


#### 展示成功信息

此方法接收一个`string`类型参数

```php
$this->response()->success('成功！');
```

#### 展示错误信息

此方法接收一个`string`类型参数

```php
$this->response()->error('出错了！');
```

#### 展示警告信息

此方法接收一个`string`类型参数

```php
$this->response()->warning('警告');
```

#### 跳转

此方法接收一个`string`类型参数，可以与`success`、`error`、`warning`等方法同时使用

```php
$this->response()->redirect('auth/users');
```

#### 跳转 (location)

`1`秒后自动跳转（非局部刷新），此方法接收一个`string`类型参数

```php
$this->response()->success('操作成功')->location('auth/users');
```

如果不传参则刷新当前页面

```php
$this->response()->success('操作成功')->location();
```

#### 刷新当前页面

此方法可以与`success`、`error`、`warning`等方法同时使用

```php
$this->response()->success('xxx')->refresh();
```

#### 下载

此方法接收一个`string`类型参数

```php
$this->response()->download('auth/users?_export_=1');
```

#### 展示确认弹窗

```php
// 成功
$this->response()->alert(true)->success('...')->detail('详细内容');

// 错误
$this->response()->alert(true)->error('...')->detail('详细内容');

// 警告
$this->response()->alert(true)->warning('...')->detail('详细内容');

// 提示
$this->response()->alert(true)->info('...')->detail('详细内容');
```

#### 返回HTML

此方法可接收一个`string`、`Renderable`、`Htmlable`类型参数，可以与`success`、`error`、`warning`等方法同时使用

> {tip} 响应的`HTML`字符默认会被置入动作按钮元素上，如果需要自己控制，则覆写[handleHtmlResponse](#handleHtmlResponse)方法即可。

```php
$this->response()->html('<a>a标签</a>');

$this->response()->html(view('...'));
```

#### 执行JS代码

此方法接收一个`string`类型参数，可以与`success`、`error`、`warning`等方法同时使用

```php
$this->response()->script(
	<<<JS
console.log('response', response, target);	
JS	
);
```

### 根据条件判断是否调用

上面所有功能接口都支持`if`模式，如

```php
// 如果 $condition 的值为 真，则调用 refresh 方法
$this->response()->success(...)->ifRefresh($condition);
$this->response()->success(...)->ifLocation($condition, 'auth/users');

// $condition 也可以是闭包
$this->response()->success(...)->ifRefresh(function () {
    return true;
});
```

