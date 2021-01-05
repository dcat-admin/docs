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

    $grid->disableToolbar();
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

### Fetching

监听表格获取数据之前事件，此事件在 `composing` 事件之后触发。

```php
$grid->listen(Grid\Events\Fetching::class, function ($grid) {
	  
});


// 可以在 composing 事件中使用
Grid::composing(function (Grid $grid) {
    $grid->listen(Grid\Events\Fetching::class, function ($grid) {
    	  
    });
});
```

### Fetched

监听表格获取数据之后事件，通过监听此事件可以批量修改数据, 参考下面实例

```php
$grid->listen(Grid\Events\Fetched::class, function ($grid, Collection $rows) {
	// $collection 当前这一个表格数据的模型集合， 你可以根据你的需要来读取或者修改它的数据。

    $rows->transform(function ($row) {
        // 更改行数据
        $row['name'] = $row['first_name'].' '.$row['last_name'];
        
        return $row;
    });
});
```

### ApplyFilter

监听表格过滤器搜索事件，此事件只有在过滤器有搜索条件时才会触发

```php
$grid->listen(Grid\Events\ApplyFilter::class, function ($grid, array $conditions) {
	// $conditions 当前过滤器生成的搜索条件数组

    dd('表格过滤器', $conditions);
});
```


### ApplyQuickSearch

监听表格快捷搜索事件，此事件只有在快捷搜索输入框有值时才会触发

```php
$grid->listen(Grid\Events\ApplyQuickSearch::class, function ($grid, $input) {
	// $input 搜索关键字

    dd('表格快捷搜索', $input);
});
```

### ApplySelector

监听表格规格筛选器事件，此事件只有在规格筛选器选中选项时才会触发

```php
$grid->listen(Grid\Events\ApplySelector::class, function ($grid, array $input) {
	// $input 筛选器选中的选项数组

    dd('表格规格筛选器', $input);
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
    
    // 设置 tr html属性
    $firstRow->setAttributes(['name' => '....']);
    
    if ($firstRow) {
        // 获取第一行的 id
        $id = $firstRow->id;
        // 转化为数组
        $row = $firstRow->toArray();
    }
});
```


