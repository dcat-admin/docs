# 模态窗 (Modal)

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
 
### 垂直居中 (centered)

设置弹窗垂直居中

```php
$modal = Modal::make()
    ->xl()
    ->centered() // 设置弹窗垂直居中
    ->title(...)
    ->body(...);
```

### 内容可滚动 (scrollable)

设置弹窗内容可滚动

```php
$modal = Modal::make()
    ->xl()
    ->scrollable() // 设置弹窗内容可滚动
    ->title(...)
    ->body(...);
```

<a name="form"></a>
## 表单弹窗

参考文档 [工具表单 - 弹窗](widgets-form.md#modal)

