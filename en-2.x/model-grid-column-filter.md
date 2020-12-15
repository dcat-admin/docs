# Column filter

This feature allows you to set a filter for a column in the table, making it easier to filter the data table by that column.

![]({{public}}/assets/img/screenshots/column-filter.png)


## String Comparison Queries

```php
use Dcat\Admin\Grid;

// WHERE `username` = "$input"
$grid->username->filter(
    Grid\Column\Filter\Equal::make()
);
```

The above call can add a `input` type filter to the header of `username` column, click the filter icon to expand the filter, and after inputting the query, the `equal` query will be executed on this column.


### Turn on field value query
This feature allows you to set a filter for the value of each column field, and click on the value of the column field to perform data table filtering operations, which is very convenient.

> {tip} When this feature is enabled, the raw value of this field is used as the search content and is not affected by the `display` method.

```php
use Dcat\Admin\Grid;

// Setting to label or calling the display method will not affect the query contents.
$grid->ip->label()->filter();

// Equivalent to
$grid->ip->filter(
    Grid\Column\Filter\Equal::make()->valueFilter()
);
```

Move the mouse over the column where the value search is enabled, and a "magnifying glass" icon will appear on the right
<a href="{{public}}/assets/img/screenshots/column-value-filter-1.png" target="_blank">
    <img class="img" width="400px" src="{{public}}/assets/img/screenshots/column-value-filter-1.png">
</a>

After clicking on the column, the "Reset" button will appear in the table header

<a href="{{public}}/assets/img/screenshots/column-value-filter-search.png" target="_blank">
    <img class="img" width="400px" src="{{public}}/assets/img/screenshots/column-value-filter-search.png">
</a>

#### Field name for set value

If the value at the top is not the value the user wants to search for, you can change the field name by doing the following.

```php
$grid->model()->with('user');

// The actual search will be for the value of the name field
$grid->username->filter('name');

// two-dimensional array
$grid->user_id->filter('user.id');

// closure
$grid->user_id->filter(function () {
    return $this->user['id'];
});
```

#### Hide the filter icon in the table header

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

The result is as follows

<a href="{{public}}/assets/img/screenshots/column-value-filter.png" target="_blank">
    <img class="img" width="400px" src="{{public}}/assets/img/screenshots/column-value-filter.png">
</a>



### The rest of the Input form type filters

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

### Date and time
If the field is a time and date related field, you can use the following methods

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


## Multiple choice queries
Suppose you need to filter one or more states of data in the form data by using the `status` field.

```php
use Dcat\Admin\Grid;

$grid->column('status', 'state')->filter(
    Grid\Column\Filter\In::make([
        0 => 'Unknown',
        1 => 'Ordered',
        2 => 'Paid',
        3 => 'cancelled',
    ])
);
```

## Range search
Suppose you need to filter out data from a price range by the `price` field.

```php
$grid->column('price')->filter(
    Grid\Column\Filter\Between::make()
);
```

Or a time and date range filter

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

## Specify the query field name

The name of the query field can be specified by the `setColumnName` method.

```php
$grid->column('column')->filter(
    Grid\Column\Filter\Equal::make()->setColumnName('custom_column')
);
```

### Query the json field

```php
$grid->column('column')->filter(
    Grid\Column\Filter\Equal::make()->setColumnName('json_column->label')
);
```


### Relational field query

If [dcat/laravel-wherehasin](https://github.com/jqhph/laravel-wherehasin) is installed, the `whereHasIn` method is used preferentially for queries.

```php
$grid->column('user.name')->filter(
    Grid\Column\Filter\Equal::make()
);

$grid->column('user_name')->filter(
    Grid\Column\Filter\Equal::make()->setColumnName('user.name')
);
```
