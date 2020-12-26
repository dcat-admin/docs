# 表单验证

### rule

`model-form`使用laravel的验证规则来验证表单提交的数据：

```php
$form->text('title')->rules('required|min:3');

// 复杂的验证规则可以在回调里面实现
$form->text('title')->rules(function (Form $form) {
    
    // 如果不是编辑状态，则添加字段唯一验证
    if (!$id = $form->model()->id) {
        return 'unique:users,email_address';
    }
    
});
```

也可以给验证规则自定义错误提示消息：

```php
$form->text('code')->rules('required|regex:/^\d+$/|min:10', [
    'regex' => 'code必须全部为数字',
    'min'   => 'code不能少于10个字符',
]);
```

如果要允许字段为空，首先要在数据库的表里面对该字段设置为`NULL`，然后

```php
$form->text('title')->rules('nullable');
```

更多规则请参考[Validation](https://laravel.com/docs/5.5/validation)。

### creationRules

此方法用法和`Form\Field::rule`用法完全一致，不同的是此方法只有在新增数据时才有效。

> {tip} 如果调用了`creationRules`方法，则`rule`方法设置的验证规则将会被忽略。

### updateRules

此方法用法和`Form\Field::rule`用法完全一致，不同的是此方法只有在更新数据时才有效。

> {tip} 如果调用了`updateRules`方法，则`rule`方法设置的验证规则将会被忽略。


## responseValidationMessages

通过`Form::responseValidationMessages`方法可以返回自定义验证错误信息，并中断后续逻辑，用法如下：

```php
// 编辑提交时是“PUT”方法
if (request()->getMethod() == 'PUT') {
    if (...) { // 你的验证逻辑
        $form->responseValidationMessages('title', 'title格式错误');
        
        // 如有多个错误信息，第二个参数可以传数组
        $form->responseValidationMessages('content', ['content格式错误', 'content不能为空']);
    }
}

$form->text('title');
$form->text('content');
```

也可以在`submitted`事件中使用这个方法
```php
$form->submitted(function ($form) {
    if (...) { // 你的验证逻辑
        $form->responseValidationMessages('title', 'title格式错误');
        
        // 如有多个错误信息，第二个参数可以传数组
        $form->responseValidationMessages('content', ['content格式错误', 'content不能为空']);
    }
});
```

## 前端验证

系统继承了<a href="https://github.com/1000hz/bootstrap-validator" target="_blank">bootstrap-validator</a>进行前端表单验证，支持H5表单类型的验证。

> {tip} 不支持H5的浏览器也可以使用前端验证，系统已经做好了兼容。大部分表单都支持前端和后端验证，两者可以同时工作不冲突，少部分表单前端验证无效。

### H5验证

#### required

必填
```php
$form->text('title')->required();
```

#### number

只允许输入数字
```php
$form->text('age')->type('number');
```

限制范围
```php
// 只允许输入 10-60 范围内的数字
$form->text('age')
    ->type('number')
    ->attribute('min', 10)
    ->attribute('max', 60);
```

#### email

邮箱
```php
$form->email('email');
```

#### url

链接
```php
$form->text('website')->type('url');
```

### 其它

#### minLength

限制字符最小长度

```php
$form->text('title')->minLength(20);

// 设置错误信息
$form->text('title')->minLength(20, '最少输入20个字符');
```

#### maxLength

限制字符最大长度
```php
$form->text('title')->maxLength(50);

// 设置错误信息
$form->text('title')->minLength(50, '不能超过50个字符');
```

#### same

限制当前字段值必须与给定字段的值相等，常用于密码确认

```php
$form->password('password');

$form->password('password_confirm')->same('password');

// 设置错误信息
$form->password('password_confirm')->same('password', '两次密码输入不一致');
```

### 自定义

开发者可以通过以下方法自定义前端验证规则。



在 `app/Admin/bootstrap.php` 中添加以下代码。
```php
use Dcat\Admin\Form\Field;

Field\Text::macro('len', function (int $length, ?string $error = null) {
    // 前端验证逻辑扩展
    Admin::script(
                <<<'JS'
Dcat.validator.extend('len', function ($el) {
    return $el.val().length != $el.attr('data-len');
});
JS
        );

        // 同时添加后端验证逻辑，这个可以看需要
        $this->rules('size:'.$length);

        return $this->attribute([
            'data-len'       => $length,
            'data-len-error' => str_replace(
                [':attribute', ':len'],
                [$this->label, $length],
                $error ?: "只能输入:len个字符"
            ),
        ]);
});
```

使用

```php
$form->text('name')->len(10);
```



