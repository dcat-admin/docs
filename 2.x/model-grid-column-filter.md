# 列过滤器

这个功能可以给表格的列设置一个过滤器，可以更方便的根据这一列进行数据表格过滤操作

![]({{public}}/assets/img/screenshots/column-filter.png)


## 字符串比较查询

```php
use Dcat\Admin\Grid;

// WHERE `username` = "$input"
$grid->username->filter(
    Grid\Column\Filter\Equal::make()
);
```

上面的调用可以给 `username` 这一列的头部加上一个 `input` 类型的过滤器，点击过滤器图标展开过滤器，输入查询提交后，会对这一列执行 `等于` 查询。


### 开启字段值查询
这个功能可以给每一列字段的值设置一个过滤器，点击该列字段的值就可以进行数据表格过滤操作，非常方便。

> {tip} 开启此功能之后会把这个字段的原始值作为搜索内容，不会受 `display` 方法影响。

```php
use Dcat\Admin\Grid;

// 设置为 label 或调用 display 方法不会影响查询内容
$grid->ip->label()->filter();

// 相当于
$grid->ip->filter(
    Grid\Column\Filter\Equal::make()->valueFilter()
);
```

鼠标移动到开启了值查询功能的列上面，右边会显示一个“放大镜”图标
<a href="{{public}}/assets/img/screenshots/column-value-filter-1.png" target="_blank">
    <img class="img" width="400px" src="{{public}}/assets/img/screenshots/column-value-filter-1.png">
</a>

点击列之后，表头会出现“重置”按钮，点击可以取消筛选

<a href="{{public}}/assets/img/screenshots/column-value-filter-search.png" target="_blank">
    <img class="img" width="400px" src="{{public}}/assets/img/screenshots/column-value-filter-search.png">
</a>

#### 设置值的字段名称

如果当前列的值并非用户想要搜索的值，可以通过以下方法更改字段名称。

```php
$grid->model()->with('user');

// 实际搜索的会是 name 字段的值
$grid->username->filter('name');

// 二维数组
$grid->user_id->filter('user.id');

// 闭包
$grid->user_id->filter(function () {
    return $this->user['id'];
});
```

#### 隐藏表头的筛选器图标

```php
use Dcat\Admin\Grid;

$grid->user_id->filterByValue();

// 相当于
$grid->user_id->filter(
    Grid\Column\Filter\Equal::make()
        ->valueFilter()
        ->hide()
);
```

效果如下

<a href="{{public}}/assets/img/screenshots/column-value-filter.png" target="_blank">
    <img class="img" width="400px" src="{{public}}/assets/img/screenshots/column-value-filter.png">
</a>



### 其余Input表单类型过滤器

```php
use Dcat\Admin\Grid;

// WHERE `username` LIKE "%{$input}%"
$grid->username->filter(
    Grid\Column\Filter\Like::make()
);

// WHERE `username` LIKE "{$input}%"
$grid->username->filter(
    Grid\Column\Filter\StartWith::make()
);

// WHERE `username` > "$input"
$grid->username->filter(
    Grid\Column\Filter\Gt::make()
);

// WHERE `username` <= "$input"
$grid->username->filter(
    Grid\Column\Filter\Ngt::make()
);

// WHERE `username` < "$input"
$grid->username->filter(
    Grid\Column\Filter\Lt::make()
);

// WHERE `username` >= "$input"
$grid->username->filter(
    Grid\Column\Filter\Nlt::make()
);
```

### 时间日期
如果字段是时间、日期相关的字段，可以使用下面的方法

```php
use Dcat\Admin\Grid;

$grid->date()->filter(
    Grid\Column\Filter\Equal::make()->date()
);

$grid->time()->filter(
    Grid\Column\Filter\Like::make()->time()
);

$grid->datetime()->filter(
    Grid\Column\Filter\Gt::make()->datetime('YYYY-MM-DD HH:mm:ss')
);
```
<a href="{{public}}/assets/img/screenshots/grid-column-filter-between.png" target="_blank">
    <img class="img" width="400px" src="{{public}}/assets/img/screenshots/grid-column-filter-between.png">
</a>


## 多选查询
假设需要在表格数据中通过 `status` 字段过滤一个或者多个状态的数据，使用多选过滤可以非常方便的实现

```php
use Dcat\Admin\Grid;

$grid->column('status', '状态')->filter(
    Grid\Column\Filter\In::make([
        0 => '未知',
        1 => '已下单',
        2 => '已付款',
        3 => '已取消',
    ])
);
```

## 范围查询
假设需要通过 `price` 字段过滤出某个价格范围内的数据

```php
$grid->column('price')->filter(
    Grid\Column\Filter\Between::make()
);
```

或者是时间、日期范围的过滤

```php
use Dcat\Admin\Grid;

$grid->date()->filter(
    Grid\Column\Filter\Between::make()->date()
);

$grid->time()->filter(
    Grid\Column\Filter\Between::make()->time()
);

$grid->datetime()->filter(
    Grid\Column\Filter\Between::make()->datetime()
);
```

## 指定查询字段名

通过`setColumnName`方法可以指定查询字段的名称

```php
$grid->column('column')->filter(
    Grid\Column\Filter\Equal::make()->setColumnName('custom_column')
);
```

### 查询json字段

```php
$grid->column('column')->filter(
    Grid\Column\Filter\Equal::make()->setColumnName('json_column->label')
);
```


### 关联关系字段查询

如果安装了 [dcat/laravel-wherehasin](https://github.com/jqhph/laravel-wherehasin)，则会优先使用`whereHasIn`方法进行查询操作

```php
$grid->column('user.name')->filter(
    Grid\Column\Filter\Equal::make()
);

$grid->column('user_name')->filter(
    Grid\Column\Filter\Equal::make()->setColumnName('user.name')
);
```
