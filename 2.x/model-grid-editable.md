# 数据表格行内编辑

数据表格所有使用行内编辑的列字段，都必须在`form`表单中定义一个相同的表单字段，否则将无法进行编辑。


### 文本 (editable)

启用
```php
$grid->column('title')->editable();

// 编辑成功后刷新页面
$grid->column('nickname')->editable(true);
```

效果
<a href="{{public}}/assets/img/screenshots/editable.gif" target="_blank">
    <img  src="{{public}}/assets/img/screenshots/editable.gif" style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%">
</a>


### 开关 (switch)


快速将列变成开关组件，使用方法如下：
```php
$grid->column('status')->switch();

// 设置颜色
use Dcat\Admin\Admin;

$grid->column('status')->switch(Admin::color()->info());
$grid->column('status')->switch('#333');

```
这个功能需要你在`form`表单方法中同样设置一个`status`字段

```php
$form->hidden('status')
	->customFormat(function ($v) {
		return $v == '打开' ? 1 : 0;
	})
	->saving(function ($v) {
		return $v ? '打开' : '关闭';
	});

// 或者
$form->switch('status')
	->customFormat(function ($v) {
		return $v == '打开' ? 1 : 0;
	})
	->saving(function ($v) {
		return $v ? '打开' : '关闭';
	});
```


编辑成功后刷新页面
```php
$grid->column('status')->switch('', true);
```


### 开关组 (switchGroup)

> {tip} 注意：在`grid`中对某字段设置`switchGroup`默认的保存结果是`0`或`1`，如需修改可以通过`$form->hidden(xxx)->saving(...)`方法修改。

快速将列变成开关组件组，使用方法如下：
```php
$grid->switch_group->switchGroup([
    'hot'        => '热门',
    'new'        => '最新',
    'recommend'  => '推荐',
    'image.show' => '显示图片', // 更新对应关联模型
]);
// 或
// 不写label会自动从翻译文件翻译，具体使用请参照 “字段翻译” 章节
$grid->switch_group->switchGroup(['is_new', 'is_hot', 'published']);
```

这个功能需要你在`form`表单方法中同样设置对应的字段

```php
$form->switch('hot')
	->customFormat(function ($v) {
		return $v == '打开' ? 1 : 0;
	})
	->saving(function ($v) {
		return $v ? '打开' : '关闭';
	});

$form->switch('new')
	->customFormat(function ($v) {
		return $v == '打开' ? 1 : 0;
	})
	->saving(function ($v) {
		return $v ? '打开' : '关闭';
	});
```



编辑成功后刷新页面
```php
$grid->column('switch_group')->switchGroup([...], true);
```


![]({{public}}/assets/img/screenshots/grid-column-switch-group.png)


### 下拉选框 (select)

```php
$grid->column('options')->select([
    1 => 'Sed ut perspiciatis unde omni',
    2 => 'voluptatem accusantium doloremque',
    3 => 'dicta sunt explicabo',
    4 => 'laudantium, totam rem aperiam',
]);
```

`select` 也支持参数为闭包，使用方法和`editable`的`select`类似。



编辑成功后刷新页面
```php
$grid->column('options')->select([...], true);
```


![]({{public}}/assets/img/screenshots/grid-column-select.png)

### 单选框 (radio)
```php
$grid->column('options')->radio([
    1 => 'Sed ut perspiciatis unde omni',
    2 => 'voluptatem accusantium doloremque',
    3 => 'dicta sunt explicabo',
    4 => 'laudantium, totam rem aperiam',
]);
```

`radio` 也支持参数为闭包，使用方法和`editable`的`select`类似。


编辑成功后刷新页面
```php
$grid->column('options')->radio([...], true);
```

### 多选框 (checkbox)
```php
$grid->column('options')->checkbox([
    1 => 'Sed ut perspiciatis unde omni',
    2 => 'voluptatem accusantium doloremque',
    3 => 'dicta sunt explicabo',
    4 => 'laudantium, totam rem aperiam',
]);
```

`checkbox` 也支持参数为闭包，使用方法和`editable`的`select`类似。



编辑成功后刷新页面
```php
$grid->column('options')->checkbox([...], true);
```

![]({{public}}/assets/img/screenshots/grid-column-checkbox.png)

