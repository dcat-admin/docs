# Shortcut table search

Quick Search is another way to search tabular data in addition to `filter`, to quickly filter the data you want, open the following way:

```php
$grid->quickSearch();

// Set form prompt value
$grid->quickSearch()->placeholder('search...');
```
This will bring up a search box in the table header:
<a href="{{public}}/assets/img/screenshots/grid-quick-search.png" target="_blank">
    <img style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" src="{{public}}/assets/img/screenshots/grid-quick-search.png">
</a>


By passing different parameters to the `quickSearch` method, to set up different search methods, there are several ways to use the following

## Like search

The first way to do a simple like query is by setting the field name
```php
$grid->quickSearch('title');

// After submission, the model performs the following queries
$model->where('title', 'like', "%{$input}%");
```

Or do like queries on multiple fields:
```php
$grid->quickSearch('title', 'desc', 'content');
// or
$grid->quickSearch(['title', 'desc', 'content']);

// After submission, the model performs the following queries
$model->where('title', 'like', "%{$input}%")
    ->orWhere('desc', 'like', "%{$input}%")
    ->orWhere('content', 'like', "%{$input}%");
```

### relationships


If [dcat/laravel-wherehasin](https://github.com/jqhph/laravel-wherehasin) is installed, the `whereHasIn` method will be used in preference to queries

```php
$grid->quickSearch('user.name', 'user.username', 'content');
```


## Custom search
The second way gives you more flexible control over your search criteria

```php
$grid->quickSearch(function ($model, $query) {
    $model->where('title', $query)->orWhere('desc', 'like', "%{$query}%");
});
```
where the parameter `$query` of the closure is the contents of the search box you fill in, and then submit it for the search in the closure.

## Quick Syntax Search
The third way references the search syntax of `Github` to perform a quick search, invoked as follows：

```php
// non-parametric
$grid->quickSearch();
```
The following syntax will be used to fill in the search box, and the corresponding query will be made after submission :

### Compare queries
`title:foo` 、`title:!foo`

```php
$model->where('title', 'foo');

$model->where('title', '!=', 'foo');
```

`rate:>10`、`rate:<10`、`rate:>=10`、`rate:<=10`
```php
$model->where('rate', '>', 10);

$model->where('rate', '<', 10);

$model->where('rate', '>=', 10);

$model->where('rate', '<=', 10);
```

### In, NotIn queries
`status:(1,2,3,4)`、`status:!(1,2,3,4)`

```php
$model->whereIn('status', [1,2,3,4]);

$model->whereNotIn('status', [1,2,3,4]);
```

### Between queries
`score:[1,10]`

```php
$model->whereBetween('score', [1, 10]);
```

### Time and date function queries
`created_at:date,2019-06-08`

```php
$model->whereDate('created_at', '2019-06-08');
```

`created_at:time,09:57:45`
```php
$model->whereTime('created_at', '09:57:45');
```

`created_at:day,08`
```php
$model->whereDay('created_at', '08');
```

`created_at:month,06`
```php
$model->whereMonth('created_at', '06');
```

`created_at:year,2019`
```php
$model->whereYear('created_at', '2019');
```

### Like Search
`content:%Laudantium%`、`content:Laud%`

```php
$model->where('content', 'like', '%Laudantium%');

$model->where('content', 'like', 'Laud%');
```

### regular query
`username:/song/`

> {tip} Please use MYSQL regular syntax here

```php
$model->where('username', 'REGEXP', 'song');
```

### Multi-criteria combination search
Separate multiple search statements with spaces to achieve multiple fields of the AND query, such as `username:%song% status:(1,2,3)`, after submission will run the following search
```php
$model->where('username', 'like', '%song%')->whereIn('status', [1, 2, 3]);
```

If a condition is an `OR` query, just add a | symbol in front of the statement cell: `username:%song% |status:(1,2,3)`
```php
$model->where('username', 'like', '%song%')->orWhereIn('status', [1, 2, 3]);
```

> {tip} If the filled query text contains spaces, it needs to be inside double quotes: `updated_at: "2019-06-08 09:57:45"`

### Label as a query field name
If it is not convenient to get the field name, you can simply use the label name as the query field.

```php
 // For example, if the table header of `user_status` is set to `user_status`.
$grid->column('user_status', 'user_status');
```

Then you can fill in `user status:(1,2,3)` to execute the following query

```php
$model->whereIn('user_status', [1, 2, 3]);
```


## Disable automatic submission

If you don't want auto-submission, you can disable it in the following ways

> {tip} When the auto-submit feature is disabled, you need to search by pressing the `Enter` key.

```php
$grid->quickSearch()->auto(false);
```
