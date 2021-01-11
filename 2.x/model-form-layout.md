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

以上布局功能使用了`bootstrap`的栅格布局系统，所有列的宽度总和不得超出`12`，并且也支持在`hasMany`和`array`表单中使用

```php
$form->hasMany('jobs', function ($form) {
     $form->column(6, function (Form $form) {
         $form->text('name')->required();
         $form->date('born')->required();
     });
     
     $form->column(6, function (Form $form) {
         $form->image('avatar');
         $form->decimal('wages');
     });
});
```




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

并且也支持在`hasMany`和`array`表单中使用

```php
$form->hasMany('jobs', function ($form) {
     $form->row(function (Form\Row $form) {
     	...
     });
     
     $form->row(function (Form\Row $form) {
     	...
     });
});
```



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
                         
     $form->hasMany('jobs', function ($form) {
         $form->text('company');
         $form->date('start_date');
         $form->date('end_date');
     });

})
```

同时，`tab` 布局中也允许嵌套使用`column`和`row`布局

```php
$form->tab('Basic info', function (Form $form) {
    $form->column(6, function (Form\BlockForm $form) {
		$form->display('id');
		$form->text('name');
	});

	$form->column(6, function (Form\BlockForm $form) {
		$form->text('username');
	});
})
```

设置默认显示的 `Tab`

```php
// 默认显示标题为 标题2 的 Tab
$form->getTab()->active('标题2');
// 也可以指定偏移量
$form->getTab()->activeByIndex(1);

$form->tab('标题1', function ($form) {
    ...
});

$form->tab('标题2', function ($form) {
    ...
});
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

![](https://cdn.learnku.com/uploads/images/202005/12/38389/B0tXWUxHDp.png!large)


### 分块布局 (block)

如果你的表单中字段非常多，那么可以通过以下方式让你的表单分块布局，并且允许嵌套使用`column`和`row`布局

```php
$form->block(8, function (Form\BlockForm $form) {
	// 设置标题
    $form->title('基本设置');
    
    // 显示底部提交按钮
    $form->showFooter();
    
    // 设置字段宽度
    $form->width(9, 2);

    $form->column(6, function (Form\BlockForm $form) {
        $form->display('id');
        $form->text('name');
        $form->email('email');
        $form->image('avatar');
        $form->password('password');
    });

    $form->column(6, function (Form\BlockForm $form) {
        $form->text('username');
        $form->email('mobile');
        $form->textarea('description');
    });
});
$form->block(4, function (Form\BlockForm $form) {
    $form->title('分块2');

    $form->text('nickname');
    $form->number('age');
    $form->radio('status')->options(['1' => '默认', 2 => '冻结'])->default(1);

    $form->next(function (Form\BlockForm $form) {
        $form->title('分块3');

        $form->date('birthday');
        $form->date('created_at');
    });
});
```

效果
<a href="https://cdn.learnku.com/uploads/images/202010/19/38389/AMCtHBcmSQ.jpg!large" target="_blank">
    <img class="img img-full" src="https://cdn.learnku.com/uploads/images/202010/19/38389/AMCtHBcmSQ.jpg!large">
</a>

