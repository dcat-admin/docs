# 表格规格筛选器


这个功能用来构建类似淘宝或京东商品的规格选择。

<a href="{{public}}/assets/img/screenshots/grid-selector.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/grid-selector.png" style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" >
</a>


### 基本使用


> {tip} `select`和`selectOne`方法的第二个参数为选择器`label`, 可以省略，如果省略的话将会自动使用翻译文件的翻译.


如下代码所示，假设`brand`字段的4个取值分别对应4个品牌，下面的方式会构建出`brand`的规格选择器

```php
use Dcat\Admin\Grid;

$grid->selector(function (Grid\Tools\Selector $selector) {
    $selector->select('brand', '品牌', [
        1 => '华为',
        2 => '小米',
        3 => 'OPPO',
        4 => 'vivo',
    ]);
});
```

`select`方法默认是多选的，在页面上点击每一个选项的右边的加号，这个字段的查询会增加一个查询选项，如果字段筛选只允许选择一项，使用`selectOne`方法

```php
$selector->selectOne('brand', '品牌', [
    1 => '华为',
    2 => '小米',
    3 => 'OPPO',
    4 => 'vivo',
]);
```

### 关联关系字段查询

如果安装了 [dcat/laravel-wherehasin](https://github.com/jqhph/laravel-wherehasin)，则会优先使用`whereHasIn`方法进行查询操作

```php
use Dcat\Admin\Grid;

$grid->selector(function (Grid\Tools\Selector $selector) {
    $selector->select('brand.id', '品牌', [
        1 => '华为',
        2 => '小米',
        3 => 'OPPO',
        4 => 'vivo',
    ]);
});
```


### 自定义查询
上面的方式会使用在选择器上选定的值，作为查询条件进行查询，但是有些情况需要更灵活的控制查询方式，那么可以使用下面的方式自定义查询：

```php
$selector->select('price', '价格', ['0-999', '1000-1999', '2000-2999'], function ($query, $value) {
    $between = [
        [0, 999],
        [1000, 1999],
        [2000, 2999],
    ];
    
    $value = current($value);

    $query->whereBetween('price', $between[$value]);
});
```

如上所示，传入一个匿名函数作为第四个参数，价格字段price选择之后，将会使用匿名函数中的逻辑进行数据查询，这样你可以定义任意的查询方式。
