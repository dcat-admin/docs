# 数据表格事件

### 初始化


通过 `Grid::resolving` 方法可以监听表格初始化事件。


开发者可以在这两个事件中改变 `Grid` 的一些设置或行为，比如需要禁用掉某些操作，可以在 `app/Admin/bootstrap.php` 加入下面的代码：

```php
use Dcat\Admin\Grid;

Grid::resolving(function (Grid $grid) {
    $grid->disableActions();

    $grid->disablePagination();

    $grid->disableCreateButton();

    $grid->disableFilter();

    $grid->disableRowSelector();

    $grid->disableTools();

    $grid->disableExport();
});


// 只需要监听一次
Grid::resolving(function (Grid $grid) {
    ...
}, true);
```
这样就不用在每一个控制器的代码中来设置了。

如果全局设置后，要在其中某一个表格中开启设置，比如开启显示操作列，在对应的实例上调用 `$grid->disableActions(false);` 就可以了


### 构建

通过 `Grid::composing` 方法可以监听表格被调用事件。

```php
Grid::composing(function (Grid $grid) {
    ...
});

// 只需要监听一次
Grid::composing(function (Grid $grid) {
    ...
}, true);
```

### fetching回调

通过 `Grid::fetching` 方法可以监听表格获取数据之前事件，此事件在 `composing` 事件之后触发。

```php

$grid = new Grid(...);

$grid->fetching(function () {
    ...
});


// 可以在 composing 事件中使用
Grid::composing(function (Grid $grid) {
    $grid->fetching(function (Grid $grid) {
        ...
    });
});
```

### rows回调

通过 `Grid::rows` 方法可以监听表格获取数据之后事件。

```php
use Dcat\Admin\Grid\Row;
use Illuminate\Support\Collection;

$grid->rows(function (Collection $rows) {
    /**
     * 获取第一行数据
     *
     * @var Row $firstRow
     */
    $firstRow = $rows->first();
    
    if ($firstRow) {
        // 获取第一行的 id
        $id = $firstRow->id;
        // 转化为数组
        $row = $firstRow->toArray();
    }
});
```

### collection回调

这个方法和`display`回调不同的是，它可以批量修改数据, 参考下面实例中的几个使用场景：

```php
use Illuminate\Support\Collection;

$grid->model()->collection(function (Collection $collection) {

    // 1. 可以给每一列加字段，类似上面display回调的作用
    $collection->transform(function ($item) {
        $item['full_name'] = $item['first_name'] . ' ' . $item['last_name'];

        return $item;
    });

    // 2. 给表格加一个序号列
    $collection->transform(function ($item, $index) {
        $item['number'] = $index;

        return $item;
    });

    // 3. 从外部接口获取数据填充到模型集合中
    $ids = $collection->pluck('id');
    $data = getDataFromApi($ids);
    $collection->transform(function ($item, $index) use ($data) {
        $item['column_name'] = $data[$index];

        return $item;
    });

    // 最后一定要返回集合对象
    return $collection;
});
```

`$collection`表示当前这一个表格数据的模型集合， 你可以根据你的需要来读取或者修改它的数据。


