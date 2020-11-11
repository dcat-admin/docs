# Data table events

### Initialization


The `Grid::resolving` method allows you to listen for table initialization events.


The developer can change some of the settings or behavior of `Grid` in both events, for example, to disable certain actions, by adding the following code to `app/Admin/bootstrap.php`.

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


// Only need to listen once.
Grid::resolving(function (Grid $grid) {
    ...
}, true);
```
This way you don't have to set it in the code of each controller.

If, after the global setting, you want to turn on the setting in one of the tables, for example to turn on the display action column, call `$grid->disableActions(false);` on the corresponding instance.


### Building

The `Grid::composing` method allows you to listen for events when the table is called.

```php
Grid::composing(function (Grid $grid) {
    ...
});

// You only need to listen in once.
Grid::composing(function (Grid $grid) {
    ...
}, true);
```

### fetching callbacks

The `Grid::fetching` method allows you to listen for the event before the table fetches data, which is triggered after the `composing` event.

```php

$grid = new Grid(...);

$grid->fetching(function () {
    ...
});


// can be used in the composing event.
Grid::composing(function (Grid $grid) {
    $grid->fetching(function (Grid $grid) {
        ...
    });
});
```

### rows callbacks

The `Grid::rows` method allows you to listen for events after the table gets data.

```php
use Dcat\Admin\Grid\Row;
use Illuminate\Support\Collection;

$grid->rows(function (Collection $rows) {
    /**
     * Get the first line of data
     *
     * @var Row $firstRow
     */
    $firstRow = $rows->first();
    
    if ($firstRow) {
        // Get the id of the first line
        $id = $firstRow->id;
        // to arrays
        $row = $firstRow->toArray();
    }
});
```

### collection callbacks

This method differs from the `display` callback in that it can batch modify data, see the following example for a few use cases.

```php
use Illuminate\Support\Collection;

$grid->model()->collection(function (Collection $collection) {

    // 1. You can add fields to each column, similar to the mentioned display callback role
    $collection->transform(function ($item) {
        $item['full_name'] = $item['first_name'] . ' ' . $item['last_name'];

        return $item;
    });

    // 2. Add a numbered column to the table
    $collection->transform(function ($item, $index) {
        $item['number'] = $index;

        return $item;
    });

    // 3. Get data from external interfaces to populate the model collection
    $ids = $collection->pluck('id');
    $data = getDataFromApi($ids);
    $collection->transform(function ($item, $index) use ($data) {
        $item['column_name'] = $data[$index];

        return $item;
    });

    // Finally, be sure to return the collection object
    return $collection;
});
```

`$collection` represents the model collection of the current table data, which you can read or modify according to your needs.


