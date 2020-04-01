# 行的使用和扩展

`model-grid`默认有四个行操作`编辑`、`快捷编辑`、`删除`和`详情`，可以通过下面的方式关闭它们：

```php
 $grid->actions(function ($actions) {
    $actions->disableDelete();
    $actions->disableEdit();
    $actions->disableQuickEidt();
    $actions->disableView();
});

// 或
$grid->disableDeleteButton();
$grid->disableEditButton();
$grid->disableQuickEditButton();
$grid->disableViewButton();
```
可以通过传入的`$actions`参数来获取当前行的数据：
```php
 $grid->actions(function ($actions) {
    // 当前行的数据数组
    $actions->row;
    
    // 当前行的某个字段的数据
    $actions->row->email;
    
    // 获取当前行主键值
    $actions->getKey();
});
```

如果有自定义的操作按钮，可以通过下面的方式添加：

```php
$grid->actions(function ($actions) {
    
    // append一个操作
    $actions->append('<a href=""><i class="fa fa-eye"></i></a>');

    // prepend一个操作
    $actions->prepend('<a href=""><i class="fa fa-paper-plane"></i></a>');
}
```

如果有比较复杂的操作，可以参考下面的方式：


先定义操作类
```php
<?php

namespace App\Admin\Extensions;

use Dcat\Admin\Admin;
use Illuminate\Contracts\Support\Renderable;

class CheckRow implements Renderable
{
    protected $id;

    public function __construct($id)
    {
        $this->id = $id;
    }

    protected function script()
    {
        return <<<JS
$('.grid-check-row').on('click', function () {
    
    // Your code.
    console.log($(this).data('id'));
    
});

JS;
    }

    public function render()
    {
        Admin::script($this->script());

        return "<a class='btn btn-xs btn-success fa fa-check grid-check-row' data-id='{$this->id}'></a>";
    }
    
}
```
然后添加操作：
```php
$grid->actions(function ($actions) {
    
    // 添加操作
    $actions->append(new CheckRow($actions->getKey()));
}
```
