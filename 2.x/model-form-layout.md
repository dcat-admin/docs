# 表单布局

### 多列布局 (column)

<a href="{{public}}/assets/img/screenshots/form-column.png" target="_blank">
    <img class="img" src="{{public}}/assets/img/screenshots/form-column.png" />
</a>   

类似于上图的左右两列布局方式，可以参考下面的代码来实现

```php
// 第一列占据1/2的页面宽度
$form->column(6, function (Form $form) {
    $form->text('name')->required();
    $form->date('born')->required();
    $form->select('education')->options([...])->required();
    $form->text('nation')->required();
    $form->text('native')->required();
    $form->text('job')->required();
    $form->text('code')->required();
    $form->text('phone')->required();
    $form->text('work')->required();
    $form->text('census')->required();
});

// 第二列占据1/2的页面宽度
$form->column(6, function (Form $form) {
    $form->image('avatar');
    $form->decimal('wages');
    $form->decimal('fund');
    $form->decimal('pension');
    $form->decimal('fee');
    $form->decimal('business');
    $form->decimal('other');
    $form->text('area')->default(0);
    $form->textarea('illness');
    $form->textarea('comment');
});

// 调整所有表单的宽度
$form->width(9, 2);
```

以上布局功能使用了`bootstrap`的栅格布局系统，所有列的宽度总和不得超出`12`


### 多行布局 (row)

使用
```php
$form->row(function (Form\Row $form) {
    $form->width(4)->text('username')->required();
	$form->width(3)->text('title');
	...
});

$form->row(function (Form\Row $form) {
	...
});

...
```
效果
<a href="{{public}}/assets/img/screenshots/form-rows.png" target="_blank">
    <img class="img img-full" src="{{public}}/assets/img/screenshots/form-rows.png">
</a>


<a name="tab"></a>
### 选项卡表单 (tab)

如果表单元素太多,会导致表单页面太长, 这种情况下可以使用`tab`方法来分隔表单:

```php
$form->tab('Basic info', function (Form $form) {
    
    $form->text('username');
    $form->email('email');
    
})->tab('Profile', function (Form $form) {
                       
   $form->image('avatar');
   $form->text('address');
   $form->mobile('phone');
   
})->tab('Jobs', function (Form $form) {
                         
     $form->hasMany('jobs', function () {
         $form->text('company');
         $form->date('start_date');
         $form->date('end_date');
     });

  })
```

#### Fieldset布局

```php
$form->fieldset('分组', function (Form $form) {
    $form->text('company');
    $form->date('start_date');
    $form->date('end_date');
});
```

如果想要默认收起

```php
$form->fieldset('分组', function (Form $form) {
    ...
})->collapsed();
```

效果

<a href="{{public}}/assets/img/screenshots/form-fieldset.png" target="_blank">
    <img class="img img-full" src="{{public}}/assets/img/screenshots/form-fieldset.png">
</a>


### 分块布局 (block)

如果你的表单中字段非常多，那么可以通过以下方式让你的表单分块布局

```php
$form->display('id');
$form->text('name');
$form->text('title');
$form->text('author');
$form->textarea('content');

$form->datetimeRange('start', 'end', '开始');

// 设置默认卡片宽度
$form->setDefaultBlockWidth(8);

// 分块显示
$form->block(4, function (Form\BlockForm $form) {
    $form->title('测试');

    $form->text('title')->default('test');
    $form->select('author')->options(['test']);
    $form->radio('state')->options([0 => '待处理', 1 => '已处理', 2 => '已拒绝'])->default(0);
});
```

效果
<a href="{{public}}/assets/img/screenshots/block-form.png" target="_blank">
    <img class="img img-full" src="{{public}}/assets/img/screenshots/block-form.png">
</a>

