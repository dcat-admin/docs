# 表单回调

`Form`目前提供了下面几个方法来接收回调函数：

### creating

在新增页面调用（非提交操作）

```php
$form->creating(function (Form $form) {
    ...
});
```

### editing

在编辑页面调用（非提交操作）

```php
$form->editing(function (Form $form) {
    ...
});
```

### submitted

在表单提交前调用

```php
$form->submitted(function (Form $form) {
    // 获取用户提交参数
    $title = $form->title;
    
    // 上面写法等同于
    $title = $form->input('title');
});
```

### saving
保存前回调

```php
$form->saving(function (Form $form) {
    // 判断是否是新增操作
    if ($form->isCreating()) {
    
    }
});
```

### saved

保存后回调

```php
$form->saved(function (Form $form) {
    // 判断是否是新增操作
    if ($form->isCreating()) {
    
    }
});
```

### deleting

删除前回调

```php
$form->deleting(function (Form $form) {
    // 跳转
    return $form->redirect('auth/user', [
        'message' => '删除失败',
        'status' => false,
    ]);
    
});
```

### deleted

删除后回调

```php
$form->deleted(function (Form $form) {
    // ...
});
```


### 获取模型中的数据
```php
$form->saved(function (Form $form) {

    $id = $form->getKey();

    $username = $form->model()->username;
    
    // 获取最终保存的数组
    $form->getUpdates();

});
```

### 页面跳转

可在任意回调事件内使用

```php
// 跳转并提示成功信息
$form->saved(function (Form $form) {
    return $form->redirect('auth/user', '保存成功');
});

// 跳转并提示错误信息
$form->creating(function (Form $form) {
    return $form->redirect('auth/user', [
        'message' => '系统错误',
        'status' => false,
    ]);
});
```

### 返回字段验证出错信息

```php

// 跳转并提示错误信息
$form->creating(function (Form $form) {
    if (...) { // 你的验证逻辑
        $form->responseValidationMessages('title', 'title格式错误');
        
        // 如有多个错误信息，第二个参数可以传数组
        $form->responseValidationMessages('content', ['content格式错误', 'content不能为空']);
    }
});
```