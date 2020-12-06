# Tabular data sources


A data warehouse (`Repository`) is a class that provides a specific interface for reading and writing operations on data.
With the introduction of the data warehouse, it is possible to build pages that do not care about the specific implementation of the read and write functions of the data, developers only need to implement a specific interface to easily switch data sources. For detailed Usages of the data warehouse, please refer to the document [data warehouse](model-repository.md).


> {tip} The data for the table is obtained through the `Dcat\Admin\Contracts\Repository::get` interface.


<a name="model"></a>
## Data from the model

> {tip} If your data comes from `Model`, then you can also use the `Model` instance directly, and the underlying layer will automatically convert `Model` into a repository instance.


When the data source supports `Eloquent Model`, just create a simple `Repository` class that inherits `Dcat\Admin\Repositories\EloquentRepository`.
```php
<?php

namespace App\Admin\Repositories;

use Dcat\Admin\Repositories\EloquentRepository;
use App\Models\Movie as MovieModel;

class Movie extends EloquentRepository
{
    // Define your model class name here
    protected $eloquentClass = MovieModel::class;
    
    // With this method you can specify the field of the query, default "*"
    public function getGridColumns()
    {
        return [$this->getKeyName(), 'name', 'title', 'created_at'];
    }
}
```


### Use the model directly
If you still find it troublesome to create the `Repository` class, you can also pass the instance of `Eloquent Model` directly into the `Grid`, and the underlying layer will automatically convert `Eloquent Model` into an instance of `EloquentRepository`. 

```php
use App\Models\Movie as MovieModel;

$grid = new Grid(new MovieModel());

...
```


### Modify source data

1、use `Grid\Model`
```php
use App\Admin\Repositories\Movie;

$grid = new Grid(new Movie());

// Add default query conditions
$grid->model()->where('id', '>', 100);

// Set initial sort criteria
$grid->model()->orderBy('id', 'desc');

...

```
Other queries can refer to the `eloquent` query method.


2、use `Model Query`

```php
use App\Models\Movie as MovieModel;

$grid = new Grid(MovieModel::where('id', '>', 100));

...
```


### Associated data

There are three ways to make your forms support Linked Data


1、Using the Repository
```php
use App\Admin\Repositories\Movie;

// Equivalent to MovieModel::with('categories')
$grid = new Grid(new Movie(['categories']));

$grid->categories;

...
```


2、use `Grid\Model`
```php
use App\Admin\Repositories\Movie;

$grid = new Grid(new Movie());

$grid->model()->with('categories');

$grid->categories;

...
```


3、use `Model Query`
```php
use App\Models\Movie as MovieModel;

$grid = new Grid(MovieModel::with('categories'));

$grid->categories;

...
```



<a name="api"></a>
## Data from external API

<a name="example"></a>
### Example

If the data is from an external API simply override the `get` method of the `Repository`. For specific usage, refer to the following Example, which uses the `Douban movie` API to obtain and display data.

> {tip} It should be noted that the `get` method returns different types of parameter values for paginated and unpaginated cases, see [data warehouse - get](model-repository.md#get).


```php
<?php
namespace App\Admin\Repositories;

use Dcat\Admin\Grid;
use Dcat\Admin\Repositories\Repository;
use Illuminate\Pagination\LengthAwarePaginator;

class ComingSoon extends Repository
{
    protected $api = 'https://api.douban.com/v2/movie/coming_soon';
    
    /**
     * Defining Primary Key Field Names 
     * 
     * @return string
     */
    public function getPrimaryKeyColumn()
    {
        return '_id';
    }

    /**
     * Inquiry Form Data
     *
     * @param Grid\Model $model
     * @return LengthAwarePaginator
     */
    public function get(Grid\Model $model)
    {
        // Current number of pages
		$currentPage = $model->getCurrentPage();
		// Number of lines per page
		$perPage = $model->getPerPage();

		// Get the sort field
		[$orderColumn, $orderType] = $model->getSort();

		// Get "scope" filter value
		$city = $model->filter()->input(Grid\Filter\Scope::QUERY_NAME, 'Guangzhou');
		
		// If other filter fields are set, values can also be obtained by the "input" method, e.g..
		$title = $model->filter()->input('title');
		if ($title !== null) {
			// Execute your filtering logic
			
		}

		$start = ($currentPage - 1) * $perPage;

		$client = new \GuzzleHttp\Client();

		$response = $client->get("{$this->api}?{$this->apiKey}&city=$city&start=$start&count=$perPage");
		$data = json_decode((string)$response->getBody(), true);

		return $model->makePaginator(
            $data['total'] ?? 0, // Total number of incoming records
            $data['subjects'] ?? [] // Two-dimensional array of incoming data
		);
    }

}
```
<a name="grid-model"></a>
### Grid\Model Common Methods

<a name="getCurrentPage"></a>
#### gets current page (getCurrentPage)
Get the current page number
- Return value: `int|null` If paging is not allowed, return null.
```php
$page = $model->getCurrentPage();
```

<a name="getPerPage"></a>
#### get number of lines per page (getPerPage)
Get the number of lines per page
- Return value: `int|null` If paging is not allowed, return null.
```php
$limit = $model->getPerPage();
```

<a name="getSort"></a>
#### getSort field (getSort)
Get sort field
- Return value: `array` `[$orderColumn, 'desc'|'asc']` || `[null, null]`

```php
// $orderColumn field name is null if not sorted.
// $orderType ascending or descending: "desc", "asc", null if not sorted.
list($orderColumn, $orderType) = $model->getSort();
```

<a name="getFilter"></a>
#### get filter object (filter)
Get the filter object, through the filter object can get the value of the search form, Usages are as follows
- Return Value `Dcat\Admin\Grid\Filter`

```php
// Get "scope" filter value
$city = $model->filter()->input(Grid\Filter\Scope::QUERY_NAME, 'Guangzhou');

// If other filter fields are set, values can also be obtained by the "input" method, e.g..
$title = $model->filter()->input('title');
if ($title !== null) {
    // Execute your filtering logic
}
```

#### Get Quick Search Form Values

The Quick Search form values can be retrieved in the following ways

```php
$quickSearch = $model->grid()->quickSearch()->value();
```


<a name="sql"></a>
## Data from complex SQL queries

If the source data needs to be retrieved with a complex SQL statement, there are two ways to do it, the first being the above method.
Override the `get` method implementation of `Repository`.

> {tip} It should be noted that the `get` method returns different types of parameter values for paginated and unpaginated cases, see [data warehouse - get](model-repository.md#get).

