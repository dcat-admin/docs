# 列的显示和扩展


`model-grid`内置了很多对于列的操作方法，可以通过这些方法很灵活的操作列数据。

## 列显示

`model-grid`内置了若干方法来帮助你扩展列功能

### display

`Dcat\Admin\Grid\Column`对象内置了`display()`方法来通过传入的回调函数来处理当前列的值，
```php
$grid->column('title')->display(function ($title) {

    return "<span style='color:blue'>$title</span>";
    
});
```
在传入的匿名函数中可以通过任何方式对数据进行处理，另外匿名函数绑定了当前列的数据作为父对象，可以在函数中调用当前行的数据：
```php
$grid->first_name();

$grid->last_name();

// 不存在的`full_name`字段
$grid->column('full_name')->display(function () {
    return $this->first_name . ' ' . $this->last_name;
});
```

### 设置列的HTML属性
设列的`html`属性
```php
$grid->name->setAttributes(['style' => 'font-size:14px']);
```

### 列视图
`view`方法可以引入一个视图文件。
```php
$grid->content->view('admin.fields.content');
```

默认会传入视图的三个变量：
 - `$model` 当前行数据
 - `$name` 字段名称
 - `$value` 为当前列的值
 
模板代码如下：
```blade
<label>名称：{{ $name }}</label>
<label>值：{{ $value }}</label>
<label>其他字段：{{ $model->title }}</label>
```

### 列编辑

通过`editable.js`的帮助，可以让你在表格中直接编辑数据，使用方法如下
```php
$grid->title()->editable();

$grid->title()->editable('textarea');

$grid->title()->editable('select', [1 => 'option1', 2 => 'option2', 3 => 'option3']);

// select 支持传递闭包作为参数，该闭包接收参数为当前行对应的模型
$grid->title()->editable('select', function($row) {
    if ($row->title === 'test') {
        return ['test1', 'test2'];
    }
    return ['test3', 'test4'];
});

$grid->birth()->editable('date');

$grid->published_at()->editable('datetime');

$grid->year->editable('year');

$grid->month->editable('month');

$grid->day->editable('day');

```
![]({{public}}/assets/img/screenshots/grid-column-editable.png)


### 开关

> {tip} 注意：在`grid`中对某字段设置`switch`默认的保存结果是`0`或`1`，如需保存其他值请在`Model`、`Repository`或`Form`中修改。

快速将列变成开关组件，使用方法如下：
```php
$grid->status()->switch();
```

### 开关组

> {tip} 注意：在`grid`中对某字段设置`switchGroup`默认的保存结果是`0`或`1`，如需保存其他值请在`Model`、`Repository`或`Form`中修改。

快速将列变成开关组件组，使用方法如下：
```php
$grid->switch_group->switchGroup([
    'hot'        => '热门',
    'new'        => '最新',
    'recommend'  => '推荐',
    'image.show' => '显示图片', // 更新对应关联模型
]);
// 或
// 不写label会自动从翻译文件翻译，具体使用请参照“字段翻译”章节
$grid->switch_group->switchGroup(['is_new', 'is_hot', 'published']);
```
![]({{public}}/assets/img/screenshots/grid-column-switch-group.png)


### 下拉选框

```php
$grid->options()->select([
    1 => 'Sed ut perspiciatis unde omni',
    2 => 'voluptatem accusantium doloremque',
    3 => 'dicta sunt explicabo',
    4 => 'laudantium, totam rem aperiam',
]);
```

`select` 也支持参数为闭包，使用方法和`editable`的`select`类似。

![]({{public}}/assets/img/screenshots/grid-column-select.png)

### 单选框
```php
$grid->options()->radio([
    1 => 'Sed ut perspiciatis unde omni',
    2 => 'voluptatem accusantium doloremque',
    3 => 'dicta sunt explicabo',
    4 => 'laudantium, totam rem aperiam',
]);
```

`radio` 也支持参数为闭包，使用方法和`editable`的`select`类似。

### 多选框
```php
$grid->options()->checkbox([
    1 => 'Sed ut perspiciatis unde omni',
    2 => 'voluptatem accusantium doloremque',
    3 => 'dicta sunt explicabo',
    4 => 'laudantium, totam rem aperiam',
]);
```

`checkbox` 也支持参数为闭包，使用方法和`editable`的`select`类似。
![]({{public}}/assets/img/screenshots/grid-column-checkbox.png)


### 图片

```php
$grid->picture()->image();

//设置服务器和宽高
$grid->picture()->image('http://xxx.com', 100, 100);

// 显示多图
$grid->pictures()->display(function ($pictures) {
    
    return json_decode($pictures, true);
    
})->image('http://xxx.com', 100, 100);
```

### 显示label标签
```php
$grid->name()->label();

// 设置颜色，默认`success`,可选`danger`、`warning`、`info`、`primary`、`default`、`success`、`custom`、`purple`、`blue`、`inverse`、`tear`
$grid->name()->label('danger');

// 接收数组
$grid->keywords()->label();
```

### 显示badge标签
```php
$grid->name()->badge();

// 设置颜色，默认`red`,可选`red`、`yellow`、`aqua`、`green`、`gray`、`custom`、`purple`、`blue`、`inverse`、`tear`
$grid->name()->badge('danger');

// 接收数组
$grid->keywords()->badge();
```

### 列展开
`expand`方法可以把内容隐藏，点击按钮的时候显示在表格下一行
```php
$grid->content
    ->display('详情') // 设置按钮名称
    ->expand(function () {
        // 返回显示的详情
        // 这里返回 content 字段内容，并用 Card 包裹起来
        $card = new Card(null, $this->content);
    
        return "<div style='padding:10px 10px 0'>$card</div>";
    });
```
也可以通过以下方式设置按钮
```php
$grid->content->expand(function (Grid\Displayers\Expand $expand) {
    // 设置按钮名称
    $expand->button('详情');

    // 返回显示的详情
    // 这里返回 content 字段内容，并用 Card 包裹起来
    $card = new Card(null, $this->content);

    return "<div style='padding:10px 10px 0'>$card</div>";
});
```


### 弹出模态框
`modal`方法可以把内容隐藏，点击按钮的时候显示在表格下一行
```php
$grid->content
    ->display('查看') // 设置按钮名称
    ->modal(function () {
        $card = new Card(null, $this->content);
    
        return "<div style='padding:10px 10px 0'>$card</div>";
    });
```

### 进度条
`progressBar`进度条
```php
$grid->rate->progressBar();

//设置颜色，默认`primary`,可选`danger`、`warning`、`info`、`primary`、`success`
$grid->rate->progressBar('success');

// 设置进度条尺寸和最大值
$grid->rate->progressBar('success', 'sm', 100);
```

### showTreeInDialog
`showTreeInDialog`方法可以把一个带有层级关系的数组呈现为树形弹窗，比如权限就可以用此方法展示
```php
// 查出所有的权限数据
$nodes = (new $permissionModel)->allNodes();

// 传入二维数组（未分层级）
$grid->permissions->showTreeInDialog($nodes);

// 也可以传入闭包
$grid->permissions->showTreeInDialog(function (Grid\Displayers\Tree $tree) use (&$nodes, $roleModel) {
    // 设置所有节点
    $tree->nodes($nodes);
    
    // 设置节点数据字段名称，默认"id"，"name"，"parent_id"
    $tree->columnNames('id', 'name', 'parent_id');

    // $this->roles 可以获取当前行的字段值
    foreach (array_column($this->roles, 'slug') as $slug) {
        if ($roleModel::isAdministrator($slug)) {
            // 选中所有节点
            $tree->checkedAll();
        }
    }
});
```
<a href="{{public}}/assets/img/screenshots/grid-column-tree.png" target="_blank">
    <img style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" src="{{public}}/assets/img/screenshots/grid-column-tree.png">
</a>

### 内容映射using
```php
$grid->status->using([0 => '未激活', 1 => '正常']);

// 第二个参数为默认值
$grid->gender->using([1 => '男', 2 => '女'], '未知');
```

### 字符串分割为数组
`explode`方法可以把字符串分割为数组。
```php
$grid->tag->explode()->label();

// 可以指定分隔符，默认","
$grid->tag->explode('|')->label();
```

### prepend

`prepend` 方法用于给 `string` 或 `array` 类型的值前面插入内容。

```php
// 当字段值是一个字符串
$grid->email->prepend('mailto:');

// 当字段值是一个数组
$grid->arr->prepend('first item');
```

### append
`prepend` 方法用于给 `string` 或 `array` 类型的值后面插入内容。

```php
// 当字段值是一个字符串
$grid->email->append('@gmail.com');

// 当字段值是一个数组
$grid->arr->append('last item');
```

### 列字符串或数组截取

```php
// 最多显示50个字符
$grid->contents->limit(50, '...');

// 如果字段值是数组也支持
$grid->tags->limit(3);
```

### 列二维码
```php
$grid->website->qrcode(function () {
    return $this->url;
}, 200, 200);
```

### 可复制
```php
$grid->website->copyable();
```

## 帮助方法

### 字符串操作
如果当前里的输出数据为字符串，那么可以通过链式方法调用`Illuminate\Support\Str`的方法。

比如有如下一列，显示`title`字段的字符串值:

```php
$grid->title();
```

在`title`列输出的字符串基础上调用`Str::title()`方法
```php
$grid->title()->title();
```
调用方法之后输出的还是字符串，所以可以继续调用`Illuminate\Support\Str`的方法：
```php
$grid->title()->title()->ucfirst();

$grid->title()->title()->ucfirst()->substr(1, 10);

```

### 数组操作
如果当前列输出的是数组，可以直接链式调用`Illuminate\Support\Collection`方法。

比如`tags`列是从一对多关系取出来的数组数据：
```php
$grid->tags();

array (
  0 => 
  array (
    'id' => '16',
    'name' => 'php',
    'created_at' => '2016-11-13 14:03:03',
    'updated_at' => '2016-12-25 04:29:35',
    
  ),
  1 => 
  array (
    'id' => '17',
    'name' => 'python',
    'created_at' => '2016-11-13 14:03:09',
    'updated_at' => '2016-12-25 04:30:27',
  ),
)

```

调用`Collection::pluck()`方法取出数组的中的`name`列
```php
$grid->tags()->pluck('name');

array (
    0 => 'php',
    1 => 'python',
  ),

```
取出`name`列之后输出的还是数组，还能继续调用用`Illuminate\Support\Collection`的方法

```php
$grid->tags()->pluck('name')->map('ucwords');

array (
    0 => 'Php',
    1 => 'Python',
  ),
```
将数组输出为字符串
```php
$grid->tags()->pluck('name')->map('ucwords')->implode('-');

"Php-Python"
```

### 混合使用

在上面两种类型的方法调用中，只要上一步输出的是确定类型的值，便可以调用类型对应的方法，所以可以很灵活的混合使用。

比如`images`字段是存储多图片地址数组的JSON格式字符串类型：
```php

$grid->images();

// "['foo.jpg', 'bar.png']"

// 链式方法调用来显示多图
$grid->images()->display(function ($images) {
    return json_decode($images, true);
    
})->map(function ($path) {
    return 'http://localhost/images/'. $path;
    
})->image();

```


## 扩展列的显示功能

可以通过两种方式扩展列功能，第一种是通过匿名函数的方式。
>扩展列功能方法后IDE默认是不会自动补全的，这时候可以通过`php artisan admin::ide-helper`生成IDE提示文件，具体请参考[IDE自动补全](helpers-ide.md)。

### 匿名函数
在`app/Admin/bootstrap.php`加入以下代码:
```php
use Dcat\Admin\Grid\Column;

// 第二个参数为 `Column` 对象， 第三个参数是自定义参数
Column::extend('color', function ($value, $column, $color) {
    return "<span style='color: $color'>$value</span>"
});
```
然后在`model-grid`中使用这个扩展：
```php
$grid->title()->color('#ccc');
```

### 扩展类
如果列显示逻辑比较复杂，可以通过扩展类来实现。

扩展类`app/Admin/Extensions/Popover.php`:
```php
<?php

namespace App\Admin\Extensions;

use Dcat\Admin\Admin;
use Dcat\Admin\Grid\Displayers\AbstractDisplayer;

class Popover extends AbstractDisplayer
{
    public function display($placement = 'left')
    {
        Admin::script("$('[data-toggle=\"popover\"]').popover()");

        return <<<EOT
<button type="button"
    class="btn btn-secondary"
    title="popover"
    data-container="body"
    data-toggle="popover"
    data-placement="$placement"
    data-content="{$this->value}"
    >
  弹出提示
</button>
EOT;

    }
}
```
然后在`app/Admin/bootstrap.php`注册扩展类：
```php
use Dcat\Admin\Grid\Column;
use App\Admin\Extensions\Popover;

Column::extend('popover', Popover::class);
```
然后就能在`model-grid`中使用了：
```php
$grid->desciption()->popover('right');
```

### 指定列名
出了上述两种方式扩展列功能，我们还可以通过指定列名称的方式扩展列功能

在`app/Admin/bootstrap.php`加入以下代码:
```php
use Dcat\Admin\Grid\Column;

// 这个扩展方法等同于
// $grid->title()->display(function ($value) {
//    return "<span style='color:red'>$value</span>"
// });
Column::define('title', function ($value) {
    return "<span style='color:red'>$value</span>"
});

// 这个扩展方法等同于
// $grid->status()->switch();
Column::define('status', Dcat\Admin\Grid\Displayers\SwitchDisplay::class);
```
然后在`model-grid`中`title`和`status`字段会自动使用以上扩展：
```php
$grid->title();
$grid->status();
```

## 根据条件判断是否使用列的显示功能
有些情况我们需要根据某个条件去判断是否使用列的某个显示功能：
> {tip} 需要注意的是，`Grid\Column::if`只对列的显示相关功能有效，其他方法如表头的相关操作都不能使用此方法！

```php
$grid->config()
    ->if(function () {
        return $this->config ? true : false;
    })
    ->display($view)
    ->expand($this->getExpandHandler('config'))
    ->else()
    ->showEmpty();
```
上面写法等同于
```php
$grid->config()
    ->if(function () {
        return $this->config ? true : false;
    })
    ->then(function (Grid\Column $column) {
        $column->display($view)->expand($this->getExpandHandler('config'));
    })
    ->else(function (Grid\Column $column) {
        $column->showEmpty();
    });
```

支持多个`if`
```php
$grid->title
    ->if(...)
    ->then(...)
    ->else(...)
    
    ->if(...)
    ->then(...)
    ->else(...);
```


