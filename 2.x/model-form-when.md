# 表单字段动态显示

> {tip} 此功能在[工具表单](widgets-form.md)中一样有效

表单字段动态显示是指，在选择表单项的指定的选项时，联动显示其他的表单项。

<a href="{{public}}/assets/img/screenshots/form-when.gif" target="_blank">
    <img class="img" src="{{public}}/assets/img/screenshots/form-when.gif" />
</a>    


目前支持的表单联动的组件有：

- `select`
- `multipleSelect`
- `radio`
- `checkbox`

## 使用方法

可以将上面的组件分为单选和多选两种类型，其中`select`、`radio`为单选组件，其它为多选组件

### 单选组件

下面的例子中，选择不同的国籍类型，将会切换选择不同的联动表单项：

```php
$form->radio('radio')
    ->when([1, 4], function (Form $form) {
        // 值为1和4时显示文本框
        $form->text('text1');
        $form->text('text2');
        $form->text('text3');
    })
    ->when(2, function (Form $form) {
        $form->editor('editor');
    })
    ->when(3, function (Form $form) {
        $form->image('image');
    })
    ->options([
        1 => '显示文本框',
        2 => '显示编辑器',
        3 => '显示文件上传',
        4 => '还是显示文本框',
    ])
    ->default(1);
```

上例中，方法`when(1, $callback)`等效于`when('=', 1, $callback)`, 如果用操作符`=`，则可以省略这个参数

同时也支持这些操作符，`=`、`>`、`>=`、`<`、`<=`、`!=`使用方法如下：

```php
$form->radio('check')
    ->when('>', 1, function () {

    })->when('>=', 2, function () {

    });
```

`select` 组件的使用方法和`radio`是一样的。

另外需要注意的是，如果使用动态显示功能之后表单不能使用`required`方法，应该使用`required_if`代替，如

```php
$form->radio('type')
    ->when([1, 4], function (Form $form) {
        $form->text('text1')
            ->rules('required_if:type,1,4') // 使用required_if
            ->setLabelClass(['asterisk']); // 显示 * 号
    });
```

### 多选组件

多选组件支持两个操作符：`in`、`notIn`

```php
$form->checkbox('nationality', '国籍')
    ->options([
        1 => '中国',
        2 => '外国',
    ])->when([1, 2], function (Form $form) { 

        $form->text('name', '姓名');
        $form->text('idcard', '身份证');

    })->when('notIn', 2, function (Form $form) { 

        $form->text('name', '姓名');
        $form->text('passport', '护照');

    });
```

`multipleSelect`组件的使用方法和`checkbox`是一样的。


### 布局

表单动态显示功能支持结合`column`以及`tab`布局功能一起使用，用法如下


`tab`布局
```php
$form->tab('Radio', function (Form $form) {
    $form->display('title')->value('单选框动态展示');

    $form->radio('radio')
        ->when([1, 4], function (Form $form) {
            $form->text('text1');
            $form->text('text2');
        })
        ->when(2, function (Form $form) {
            $form->editor('editor');
        })
        ->options($this->options)
        ->default(1);
});
```

`column`布局
```php
$form->column(6, function (Form $form) {
    $form->radio('radio')
        ->when([1, 4], function (Form $form) {
            $form->text('text1');
            $form->text('text2');
        })
        ->when(2, function (Form $form) {
            $form->editor('editor');
        })
        ->options($this->options)
        ->default(1);
});
```