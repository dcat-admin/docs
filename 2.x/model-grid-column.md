# 列的基本使用

### 设置列为可排序 (sortable)
```php
$grid->column('id')->sortable();
```

### 设置列的宽度 (width)
设置列的宽度，当字段内容过长时可以使用这个方法限制列宽度
```php
// px
$grid->column('long_text')->width('300px');
// 百分比
$grid->column('long_text')->width('15%');
```

### 固定列 (fixColumns)


通过 `fixColumns` 方法可以给表格设置固定列，第一个参数表示固定从头开始的前三列，第二个参数表示固定从后往前数的两列，（第二个参数可不传，默认为-1）

```php
$grid->fixColumns(2, -2);
```

效果

![](https://cdn.learnku.com/uploads/images/202007/12/38389/8aKnpG11g4.gif!large)
  

### 设置td标签HTML属性

```php
$grid->column('email')->setAttributes(['name' => '...'])
```


### 设置表格头HTML属性
设标题的`html`属性
```php
// 修改颜色
$grid->column('name')->setHeaderAttributes(['style' => 'color:#5b69bc']);
```

### 设置列选择器 (字段显示或隐藏)

此功能默认不启用

```php
// 开启字段选择器功能
$grid->showColumnSelector();

// 设置默认隐藏字段
$grid->hideColumns(['field1', ...]);
``` 

![](https://cdn.learnku.com/uploads/images/202004/26/38389/MTgikMeV1o.png!large)



### 设置列提示信息
`Grid\Column::help`参数：
 - $help `string` 提示内容
 - $style `string` 提示窗背景颜色，支持“primary”、“success”、“danger”、“purple”
 - $placement `string` 提示窗位置，支持“top”、“left”、“right”、“bottom”

![](https://cdn.learnku.com/uploads/images/202004/26/38389/MTgikMeV1o.png!large)


```php
$grid->column('id')->help('提示信息');
```

### 设置列搜索

通过`Grid\Column::filter`方法可以给列设置一个过滤器，可以很方便的根据这一列进行数据表格过滤操作，具体使用方法请参考[列过滤器](model-grid-column-filter.md)。

![](https://cdn.learnku.com/uploads/images/202004/26/38389/8zNK7CHS3V.png!large)




### 扩展列功能

通过`Grid\Column::macro`方法可以扩展列方法。

在 `app/Admin/bootstrap.php` 中添加以下代码

```php
use Dcat\Admin\Grid;

// $value 是当前字段的值
// $p1、$p2 是自定义参数
Grid\Column::macro('myHeader', function ($value, $p1, $p2 = null) {
    // MyHeader 需要实现 Illuminate\Contracts\Support\Renderable 接口
    // 当然这里也可以直接传字符串
    return $this->addHeader(new MyHeader($this, $p1, $p2));
});
```

`MyHeader` 类
```php
use Dcat\Admin\Grid\Column;
use Illuminate\Contracts\Support\Renderable;

class MyHeader implements Renderable
{
    public function __construct(Column $column, $p1, $p2)
    {
        ...
    }
    
    public function render()
    {
        ...
    }
}
```

使用

```php
$grid->column('user')->myHeader($p1, $p2);

$grid->column('first_name')->myHeader($p1);
```


