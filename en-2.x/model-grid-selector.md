# Filter for table specifications


This feature is used to build a selection of specifications similar to Taobao or Jingdong products.

<a href="{{public}}/assets/img/screenshots/grid-selector.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/grid-selector.png" style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" >
</a>


### Basic use


> {tip} The second argument of `select` and `selectOne` method is selector `label`, which can be omitted, if omitted the translation of the translation file will be used automatically.


As shown in the following code, assuming that the four values of the `brand` field correspond to four brands, the following way will build the specification selector of `brand`.

```php
use Dcat\Admin\Grid;

$grid->selector(function (Grid\Tools\Selector $selector) {
    $selector->select('brand', 'brands', [
        1 => 'Huawei',
        2 => 'millet',
        3 => 'OPPO',
        4 => 'vivo',
    ]);
});
```

`select` method is multiple choice by default, click on the page to the right of each option of the plus sign, the query will add a query option for this field, if the field filter allows only one selection, use the `selectOne` method

```php
$selector->selectOne('brand', 'brand', [
        1 => 'Huawei',
        2 => 'millet',
        3 => 'OPPO',
        4 => 'vivo',
]);
```

### Relational field lookup

If [dcat/laravel-wherehasin](https://github.com/jqhph/laravel-wherehasin) is installed, the `whereHasIn` method is used preferentially for queries.

```php
use Dcat\Admin\Grid;

$grid->selector(function (Grid\Tools\Selector $selector) {
    $selector->select('brand.id', 'brands', [
        1 => 'Huawei',
        2 => 'millet',
        3 => 'OPPO',
        4 => 'vivo',
    ]);
});
```


### Custom queries
The above method will use the value selected on the selector as the query condition, but in some cases where more flexible control over the query is needed, the query can be customized using the following method:

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

As shown above, an anonymous function is passed in as the fourth argument. Once price is selected, the logic in the anonymous function will be used to query the data, so that you can define any kind of query.
