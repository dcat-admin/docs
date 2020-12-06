# Repository

Data warehouse (`Repository`) is a concrete implementation of the `Dcat Admin` interface for data add/drop operations.
The intervention of `Repository` can make the construction of the page no longer care about the specific implementation of the data read and write functions.
Developers through the implementation of the `Repository` interface can read and write operations on the data.

> {tip} Of course, for the convenience of the system also retains the ability to use `Model` directly, the underlying layer will automatically convert `Model` to a repository instance, after all, most of the time the direct use of `Model` can also meet our needs.


Components such as data table `Grid`, data form `Form`, data detail `Show`, `Tree`, etc. no longer depend directly on `Model`, but on a repository that provides simpler and clearer interfaces, the following are all the interfaces of `Repository`.

```php
<?php

namespace Dcat\Admin\Contracts;

use Dcat\Admin\Form;
use Dcat\Admin\Grid;
use Dcat\Admin\Show;
use Illuminate\Support\Collection;

interface Repository
{
    /**
     * Get Primary Key Name.
     *
     * @return string
     */
    public function getKeyName();

    /**
     * Get creation time field
     *
     * @return string
     */
    public function getCreatedAtColumn();

    /**
     * Get update time field
     *
     * @return string
     */
    public function getUpdatedAtColumn();

    /**
     * Whether to use soft deletes
     *
     * @return bool
     */
    public function isSoftDeletes();

    /**
     * Get Grid Data
     *
     * @param Grid\Model $model
     *
     * @return \Illuminate\Contracts\Pagination\LengthAwarePaginator|Collection|array
     */
    public function get(Grid\Model $model);

    /**
     * Get Edit Page Data
     *
     * @param Form $form
     *
     * @return array|\Illuminate\Contracts\Support\Arrayable
     */
    public function edit(Form $form);

    /**
     * Get details page data
     *
     * @param Show $show
     *
     * @return array|\Illuminate\Contracts\Support\Arrayable
     */
    public function detail(Show $show);

    /**
     * New record
     *
     * @param Form $form
     *
     * @return mixed
     */
    public function store(Form $form);

    /**
     * Query the row data before updating
     *
     * @param Form $form
     *
     * @return array|\Illuminate\Contracts\Support\Arrayable
     */
    public function updating(Form $form);

    /**
     * Update data
     *
     * @param Form $form
     *
     * @return bool
     */
    public function update(Form $form);

    /**
     * Delete data
     *
     * @param Form  $form
     * @param array $deletingData
     *
     * @return mixed
     */
    public function delete(Form $form, array $deletingData);

    /**
     * Row data before deletion
     *
     * @param Form $form
     *
     * @return array|\Illuminate\Contracts\Support\Arrayable
     */
    public function deleting(Form $form);
}

```

If your data is multi-level structured, you will also need to implement the following interfaces
```php
<?php

namespace Dcat\Admin\Contracts;

interface TreeRepository
{
    /**
     * Get the primary key field name.
     *
     * @return string
     */
    public function getPrimaryKeyColumn();

    /**
     * Get the parent ID field name.
     *
     * @return string
     */
    public function getParentColumn();

    /**
     * Get the TITLE field name
     *
     * @return string
     */
    public function getTitleColumn();

    /**
     * Get the sort field name
     *
     * @return string
     */
    public function getOrderColumn();

    /**
     * Saving hierarchical data sorting
     *
     * @param array $tree
     * @param int   $parentId
     */
    public function saveOrder($tree = [], $parentId = 0);

    /**
     * Setting Data Query Callbacks
     *
     * @param \Closure|null $query
     *
     * @return $this
     */
    public function withQuery($queryCallback);

    /**
     * Get hierarchical data.
     *
     * @return array
     */
    public function toTree();
}
```

## Repository Interface

### getKeyName
This interface requires the primary key field name of the returned data to return a value of type `string`.
```php
    public function getKeyName()
    {
        return 'id';
    }
```

### getCreatedAtColumn
This interface requires the `created_at` field name of the data to be returned, or an empty string or `null` value if there is no value.
```php
    // Returns a null string or `null` value if there is no value
    public function getCreatedAtColumn()
    {
        return 'created_at';
    }
```

### getUpdatedAtColumn
This interface requires the `updated_at` field name of the data to be returned, or an empty string or `null` value if there is no value.
```php
    // Returns a null string or `null` value if there is no value
    public function getCreatedAtColumn()
    {
        return 'updated_at';
    }
```

### isSoftDeletes
This interface requires the return of data to support soft delete or not, return a value of type `bool`.
```php
    public function isSoftDeletes()
    {
        return true;
    }
```

<a name="get"></a>
### get
This interface requires the return of data from the data table `Grid` for data table presentation and requires a value of type `array`, `Illuminate\Support\Collection` or `LengthAwarePaginator`.

#### pagination
Requires the return of a `\Illuminate\Contracts\Pagination\LengthAwarePaginator` type value when data is to be paginated.
```php
    public function get(Grid\Model $model)
    {
        // Get the current number of pages
        $currentPage = $model->getCurrentPage();
        // Get the number of lines per page
        $perPage = $model->getPerPage();

        // Get filter parameters
        $city = $model->filter()->input(Grid\Filter\Scope::QUERY_NAME, 'Guangzhou');

        $start = ($currentPage - 1) * $perPage;

        $client = new \GuzzleHttp\Client();

        $response = $client->get("{$this->api}?{$this->apiKey}&city=$city&start=$start&count=$perPage");
        $data = json_decode((string)$response->getBody(), true);

        return $model->makePaginator(
            $data['total'] ?? 0, // Total number of incoming records
            $data['subjects'] ?? [] // Two-dimensional array of incoming data
        );
    }
```

#### no pagination
If paging is not required, simply return a value of type `array` or `Illuminate\Support\Collection`.
```php
    public function get(Grid\Model $model)
    {
        return [
            ['id' => 1, 'name' => 'n1'],
            ['id' => 2, 'name' => 'n2']
        ];
    }
```

Note that `grid` requires paging to be disabled!
```php
$grid->disablePagination()
```

### edit
This interface requires the return of data from a form edit page for displaying a data form edit page, and needs to return a value of type `array`.
```php
    public function edit(Form $form): array
    {
        // Get data primary key values
        $id = $form->getKey();
    
        return ['id' => 1, 'name' => 'n1'];
    }
```

### detail
This interface requires data from the Data Details page to be returned for displaying data details, and requires a value of type `array` to be returned.
```php
    public function detail(Show $show): array
    {
        // Get data primary key values
        $id = $show->getId();
    
        return ['id' => 1, 'name' => 'n1'];
    }
```

### store
This interface is used to add a new record and can return values of type `int`, `string` or `bool`.
```php
    public function store(Form $form)
    {
        // Access to data to be added
        $attributes = $form->updates();
        
        // Implement your added logic.
    
        // Return the new record id or bool value.
        return 1;
    }
```

### updating
This interface is used to query the raw record when the data form modifies the data, which needs to return a value of type `array` or `Model`.

> {tip} This interface is only used for some special fields, such as image and file upload fields, so when an image or file is changed, the old file can be deleted based on the data retrieved from this interface. So if your form does not use such special fields, this interface can return an empty array.

```php
    public function updating(Form $form)
    {
        // Get data primary key values
        $id = $form->getKey();
    
        return ['id' => 1, 'name' => 'n1'];
    }
```

### update
This interface is used to modify records for data forms, and can return values of type `int`, `string` or `bool`.
```php
    public function update(Form $form)
    {
        // Access to data to be edited
        $attributes = $form->updates();
        
        // Execute your editing logic
    
        // Return success
        return true;
    }
```

### deleting
This interface is used to query the original record when deleting data, which requires the return of a two-dimensional array, or a Collection model.

```php
    public function deleting(Form $form): array
    {
        // Multiple ids when batch deleting
        $id = explode(',', $form->getKey());
        
        // Implement your logic.
    
        // Note that a two-dimensional array needs to be returned here.
        return [
            ['id' => 1, 'name' => 'h1'],
        ];

        // You can also return a collection
        return Modell::find($id);
    }
```


### destroy
The single/batch delete data method, success returns `true`, failure returns `false`.

```php
    public function destroy(Form $form, array $deletingData)
    {
        // Multiple ids when batch deleting
        $id = explode(',', $form->getKey());
        
        // $deletingData is the data returned by the getDataWhenDeleting interface.
        
        // Implement your logic.
    
        return true;
    }
```

## TreeRepository接口

### getPrimaryKeyColumn

This interface is used to return the primary key field name of the data, which requires a value of type `string` to be returned.

```php
    public function getPrimaryKeyColumn()
    {
        return $this->getKeyName();
    }
```

### getParentColumn

This interface is used to return the parent ID field name of the data and requires a value of type `string` to be returned.

```php
    public function getParentColumn()
    {
        return 'parent_id';
    }
```


### getTitleColumn

This interface is used to return the data TITLE field name, which requires a value of type `string` to be returned.

```php
    public function getTitleColumn()
    {
        return 'title';
    }
```

### getOrderColumn

This interface is used to return the name of the data sorting field and requires a value of type `string`.

> {tip} This field is not required, so if your data doesn't support it or doesn't need to be sorted, return an empty value!

```php
    public function getOrderColumn()
    {
        return 'order';
    }
```

### saveOrder

This interface is used to save the sorting of hierarchical data and receives two parameters

+ `$tree` `array` This field is a hierarchical array of fields
+ `$parentId` `int` This field is primarily used to pass parent IDs for recursion.

> {tip} If your data does not support `MySQL`, you can refer to the `Dcat\Admin\Traits\ModelTree::saveOrder` method to do it yourself.

```php
$tree = [
	[
		'id'        => 1,
		'title'     => 'title',
		'parent_id' => 0,
		
		'children' => [
			[
				'id'        => 2,
				'title'     => 'child1',
				'parent_id' => 1,
			],
			[
				'id'        => 3,
				'title'     => 'child2',
				'parent_id' => 1,
			],
		],
	]
];

// Save the sort, please implement your own internal logic.
$repository->saveOrder($tree);  
```

### withQuery

This interface should be used in conjunction with the `toTree` interface to receive a parameter: a callback or parameter primarily used to set the data query operation.

```php
<?php

use Dcat\Admin\Contracts\Repository;
use Dcat\Admin\Contracts\TreeRepository;
use Dcat\Admin\Support\Helper;

class Category implements Repository, TreeRepository
{
    protected $queryCallbacks = [];
    
    public function withQuery($queryCallback)
    {
        $this->queryCallbacks[] = $queryCallback;
        
        return $this;
	}
	
	public function toTree()
	{
	    // The code shown here is just to illustrate what the withQuery method does.
	    $client = ...;
	    
	    foreach ($this->queryCallbacks as $callback) {
	    	$callback($client);
	    }
	    
	    return Helper::buildNestedArray($client->get());
	}
}   
```

### toTree

This interface is mainly used for querying data and returning it at a hierarchical level, and needs to return `array` type values.

```php
	public function toTree()
	{
	    $client = ...;
	    
	    foreach ($this->queryCallbacks as $callback) {
	    	$callback($client);
	    }
	    
	    return Helper::buildNestedArray($client->get());
	}
```

## Models
`Dcat Admin` already has built-in support for `Eloquent model`, so if your data source supports `Model`, then you only need to inherit the `Dcat\Admin\Repositories\EloquentRepository` class to implement the `CRUD` operation on the data, such as.
```php
<?php

namespace App\Admin\Repositories;

use Dcat\Admin\Repositories\EloquentRepository;
use App\Models\Movie as MovieModel;

class Movie extends EloquentRepository
{
    // Just set your model class name here
    protected $eloquentClass = MovieModel::class;
    
    // With this method you can specify the field of the query, default "*"
    public function getGridColumns()
    {
        return [$this->getKeyName(), 'name', 'title', 'created_at'];
    }
    
    // With this method, you can specify the fields of the form page query, default "*"
    public function getFormColumns()
    {
        return [$this->getKeyName(), 'name', 'title', 'created_at'];
    }
    
   // With this method, you can specify the field for the data detail page query, which defaults to "*"
    public function getDetailColumns()
    {
        return ['*'];
    }
    
}
```

## QueryBuilder

If your data supports `QueryBuilder` query, but it is not convenient to build model classes (for example, you need to dynamically look up table data), you can inherit the `Dcat\Admin\Repositories\QueryBuilderRepository` class.

> {tip} Note that `QueryBuilderRepository` does not support `Model`'s associative model, soft delete, model tree and field sorting functions by default.

```php
<?php

namespace App\Admin\Repositories;

use Dcat\Admin\Repositories\QueryBuilderRepository;

class MyRepository extends QueryBuilderRepository
{
    // Set the name of your primary key
    protected $keyName = 'id';
    
    // Set the Create Time field
    protected $createdAtColumn = 'created_at';
    
     // Set the update time field
    protected $updatedAtColumn = 'updated_at';
    
    // Return table name
    public function getTable()
    {
        return 'your_table_name';
    }
    
    // Return the name of your primary key.
    public function getKeyName()
    {
        return $this->keyName;
    }
    
    // With this method you can specify the field of the query, default "*"
    public function getGridColumns()
    {
        return [$this->getKeyName(), 'name', 'title', 'created_at'];
    }
    
    // With this method, you can specify the fields of the form page query, default "*"
    public function getFormColumns()
    {
        return [$this->getKeyName(), 'name', 'title', 'created_at'];
    }
    
   // With this method, you can specify the field for the data detail page query, which defaults to "*"
    public function getDetailColumns()
    {
        return ['*'];
    }
}
```


