# 数据仓库


数据仓库(`Repository`)是`Dcat Admin`中对数据增删改查操作接口的具体实现，通过`Repository`的介入可以让页面的构建不再关心数据读写功能的具体实现，开发者通过实现`Repository`接口即可对数据进行读写操作。

> {tip} 当然为了方便系统也保留了直接使用 `Model` 的功能，底层会自动把`Model`转化为数据仓库实例，毕竟大多数时候直接使用 `Model` 也能满足我们的需求。



数据表格`Grid`、数据表单`Form`、数据详情`Show`、`Tree` 等组件不再直接依赖于`Model`，而是依赖于提供更简单清晰的接口的数据仓库，下面是`Repository`的所有接口：

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
     * 获取主键名称.
     *
     * @return string
     */
    public function getKeyName();

    /**
     * 获取创建时间字段.
     *
     * @return string
     */
    public function getCreatedAtColumn();

    /**
     * 获取更新时间字段.
     *
     * @return string
     */
    public function getUpdatedAtColumn();

    /**
     * 是否使用软删除.
     *
     * @return bool
     */
    public function isSoftDeletes();

    /**
     * 获取Grid表格数据.
     *
     * @param Grid\Model $model
     *
     * @return \Illuminate\Contracts\Pagination\LengthAwarePaginator|Collection|array
     */
    public function get(Grid\Model $model);

    /**
     * 获取编辑页面数据.
     *
     * @param Form $form
     *
     * @return array|\Illuminate\Contracts\Support\Arrayable
     */
    public function edit(Form $form);

    /**
     * 获取详情页面数据.
     *
     * @param Show $show
     *
     * @return array|\Illuminate\Contracts\Support\Arrayable
     */
    public function detail(Show $show);

    /**
     * 新增记录.
     *
     * @param Form $form
     *
     * @return mixed
     */
    public function store(Form $form);

    /**
     * 查询更新前的行数据.
     *
     * @param Form $form
     *
     * @return array|\Illuminate\Contracts\Support\Arrayable
     */
    public function updating(Form $form);

    /**
     * 更新数据.
     *
     * @param Form $form
     *
     * @return bool
     */
    public function update(Form $form);

    /**
     * 删除数据.
     *
     * @param Form  $form
     * @param array $deletingData
     *
     * @return mixed
     */
    public function delete(Form $form, array $deletingData);

    /**
     * 查询删除前的行数据.
     *
     * @param Form $form
     *
     * @return array|\Illuminate\Contracts\Support\Arrayable
     */
    public function deleting(Form $form);
}

```

如果你的数据是多层级结构，则还需要实现以下接口
```php
<?php

namespace Dcat\Admin\Contracts;

interface TreeRepository
{
    /**
     * 获取主键字段名称.
     *
     * @return string
     */
    public function getPrimaryKeyColumn();

    /**
     * 获取父级ID字段名称.
     *
     * @return string
     */
    public function getParentColumn();

    /**
     * 获取标题字段名称.
     *
     * @return string
     */
    public function getTitleColumn();

    /**
     * 获取排序字段名称.
     *
     * @return string
     */
    public function getOrderColumn();

    /**
     * 保存层级数据排序.
     *
     * @param array $tree
     * @param int   $parentId
     */
    public function saveOrder($tree = [], $parentId = 0);

    /**
     * 设置数据查询回调.
     *
     * @param \Closure|null $query
     *
     * @return $this
     */
    public function withQuery($queryCallback);

    /**
     * 获取层级数据.
     *
     * @return array
     */
    public function toTree();
}
```

## Repository接口

### getKeyName
此接口要求返回数据的主键字段名称，需要返回`string`类型值。
```php
    public function getKeyName()
    {
        return 'id';
    }
```

### getCreatedAtColumn
此接口要求返回数据的`created_at`字段名称，如果没有值可以返回空字符串或`null`值。
```php
    // 如果没有值可以返回空字符串或`null`值
    public function getCreatedAtColumn()
    {
        return 'created_at';
    }
```

### getUpdatedAtColumn
此接口要求返回数据的`updated_at`字段名称，如果没有值可以返回空字符串或`null`值。
```php
    // 如果没有值可以返回空字符串或`null`值
    public function getCreatedAtColumn()
    {
        return 'updated_at';
    }
```

### isSoftDeletes
此接口要求返回数据是否支持软删除，请返回一个`bool`类型的值。
```php
    public function isSoftDeletes()
    {
        return true;
    }
```

<a name="get"></a>
### get
此接口要求返回数据表格`Grid`的数据，用于数据表格展示，要求返回一个`array`、`Illuminate\Support\Collection`或`LengthAwarePaginator`类型值。

#### 分页
当数据需要分页时要求返回一个`\Illuminate\Contracts\Pagination\LengthAwarePaginator`类型值：
```php
    public function get(Grid\Model $model)
    {
        // 获取当前页数
        $currentPage = $model->getCurrentPage();
        // 获取每页显示行数
        $perPage = $model->getPerPage();

        // 获取筛选参数
        $city = $model->filter()->input(Grid\Filter\Scope::QUERY_NAME, '广州');

        $start = ($currentPage - 1) * $perPage;

        $client = new \GuzzleHttp\Client();

        $response = $client->get("{$this->api}?{$this->apiKey}&city=$city&start=$start&count=$perPage");
        $data = json_decode((string)$response->getBody(), true);

        return $model->makePaginator(
            $data['total'] ?? 0, // 传入总记录数
            $data['subjects'] ?? [] // 传入数据二维数组
        );
    }
```

#### 不分页
如果不需要分页，则直接返回一个`array`或`Illuminate\Support\Collection`类型值即可。
```php
    public function get(Grid\Model $model)
    {
        return [
            ['id' => 1, 'name' => 'n1'],
            ['id' => 2, 'name' => 'n2']
        ];
    }
```

注意，`grid`需要禁用分页
```php
$grid->disablePagination()
```

### edit
此接口要求返回表单编辑页面的数据，用于显示数据表单编辑页面，需要返回`array`类型值。
```php
    public function edit(Form $form): array
    {
        // 获取数据主键值
        $id = $form->getKey();
    
        return ['id' => 1, 'name' => 'n1'];
    }
```

### detail
此接口要求返回数据详情页面的数据，用于显示数据详情，需要返回`array`类型值。
```php
    public function detail(Show $show): array
    {
        // 获取数据主键值
        $id = $show->getId();
    
        return ['id' => 1, 'name' => 'n1'];
    }
```

### store
此接口用于新增一条记录，可以返回`int`、`string`或`bool`类型值。
```php
    public function store(Form $form)
    {
        // 获取待新增的数据
        $attributes = $form->updates();
        
        // 执行你的新增逻辑
    
        // 返回新增记录id或bool值
        return 1;
    }
```

### updating
此接口用于数据表单修改数据时查询原始记录，需要返回`array`或`Model`类型值。

> {tip} 此接口只有某些特殊字段会用到，如图片、文件上传字段，当更改了图片或文件时可以根据这个接口查出的数据删除旧文件。所以如果你的表单中没有用到此类特殊字段，此接口可以返回一个空数组。

```phpjie
    public function updating(Form $form)
    {
        // 获取数据主键值
        $id = $form->getKey();
    
        return ['id' => 1, 'name' => 'n1'];
    }
```

### update
此接口用于数据表单修改记录，可以返回`int`、`string`或`bool`类型值。
```php
    public function update(Form $form)
    {
        // 获取待编辑的数据
        $attributes = $form->updates();
        
        // 执行你的编辑逻辑
    
        // 返回成功
        return true;
    }
```

### deleting
此接口用于删除数据时查询原始记录，需要返回二维数组，或Collection model。

```php
    public function deleting(Form $form): array
    {
        // 当批量删除时id为多个
        $id = explode(',', $form->getKey());
        
        // 执行你的逻辑
    
        // 注意这里需要返回二维数组
        return [
            ['id' => 1, 'name' => 'h1'],
        ];
        
        // 也可以返回collection
        return Modell::find($id);
    }
```


### destroy
单行/批量删除数据方法，成功返回`true`，失败返回`false`。

```php
    public function destroy(Form $form, array $deletingData)
    {
        // 当批量删除时id为多个
        $id = explode(',', $form->getKey());
        
        // $deletingData 是 getDataWhenDeleting 接口返回的数据
        
        // 执行你的逻辑
    
        return true;
    }
```

## TreeRepository接口

### getPrimaryKeyColumn

此接口用于返回数据的主键字段名称，需要返回`string`类型值。

```php
    public function getPrimaryKeyColumn()
    {
        return $this->getKeyName();
    }
```

### getParentColumn

此接口用于返回数据的父ID字段名称，需要返回`string`类型值。

```php
    public function getParentColumn()
    {
        return 'parent_id';
    }
```


### getTitleColumn

此接口用于返回数据标题字段名称，需要返回`string`类型值。

```php
    public function getTitleColumn()
    {
        return 'title';
    }
```

### getOrderColumn

此接口用于返回数据排序字段名称，需要返回`string`类型值。

> {tip} 此字段不是必须的，如果你的数据不支持或不需要排序，请返回空值！

```php
    public function getOrderColumn()
    {
        return 'order';
    }
```

### saveOrder

此接口用于保存层级数据的排序，并且接收两个参数

+ `$tree` `array` 此字段是一个已分好层级的数组
+ `$parentId` `int` 此字段主要用于递归时传递父ID使用

> {tip} 如果你的数据不支持 `MySQL`，可参考 `Dcat\Admin\Traits\ModelTree::saveOrder` 方法自行实现。

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

// 保存排序，内层逻辑请自行实现
$repository->saveOrder($tree);  
```

### withQuery

此接口需结合 `toTree` 接口使用，接收一个参数：主要用于设置数据查询操作的相关回调或参数。

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
	    // 这里演示的代码只是为了说明 withQuery 方法的作用
	    $client = ...;
	    
	    foreach ($this->queryCallbacks as $callback) {
	    	$callback($client);
	    }
	    
	    return Helper::buildNestedArray($client->get());
	}
}   
```

### toTree

此接口主要用于查询数据并分好层级返回，需要返回 `array` 类型值。

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

## 模型
`Dcat Admin`已经内置了对`Eloquent model`的支持，如果你的数据源是支持`Model`的，那么只需继承`Dcat\Admin\Repositories\EloquentRepository`类即可实现对数据的`CURD`操作，如：
```php
<?php

namespace App\Admin\Repositories;

use Dcat\Admin\Repositories\EloquentRepository;
use App\Models\Movie as MovieModel;

class Movie extends EloquentRepository
{
    // 这里设置你的模型类名即可
    protected $eloquentClass = MovieModel::class;
    
    // 通过这个方法可以指定查询的字段，默认"*"
    public function getGridColumns()
    {
        return [$this->getKeyName(), 'name', 'title', 'created_at'];
    }
    
    // 通过这个方法可以指定表单页查询的字段，默认"*"
    public function getFormColumns()
    {
        return [$this->getKeyName(), 'name', 'title', 'created_at'];
    }
    
   // 通过这个方法可以指定数据详情页查询的字段，默认"*"
    public function getDetailColumns()
    {
        return ['*'];
    }
    
}
```

## QueryBuilder

如果你的数据支持`QueryBuilder`查询，但不方便建模型类（比如需要动态查表数据），则可以继承`Dcat\Admin\Repositories\QueryBuilderRepository`类。

> {tip} 注意，`QueryBuilderRepository`默认是不支持`Model`的关联模型、软删除、模型树以及字段排序等功能，如果需要这些功能，请自定实现上述相关接口即可。

```php
<?php

namespace App\Admin\Repositories;

use Dcat\Admin\Repositories\QueryBuilderRepository;

class MyRepository extends QueryBuilderRepository
{
    // 设置你的主键名称
    protected $keyName = 'id';
    
    // 设置创建时间字段
    protected $createdAtColumn = 'created_at';
    
     // 设置更新时间字段
    protected $updatedAtColumn = 'updated_at';
    
    // 返回表名
    public function getTable()
    {
        return 'your_table_name';
    }
    
    // 返回你的主键名称
    public function getKeyName()
    {
        return $this->keyName;
    }
    
    // 通过这个方法可以指定查询的字段，默认"*"
    public function getGridColumns()
    {
        return [$this->getKeyName(), 'name', 'title', 'created_at'];
    }
    
    // 通过这个方法可以指定表单页查询的字段，默认"*"
    public function getFormColumns()
    {
        return [$this->getKeyName(), 'name', 'title', 'created_at'];
    }
    
   // 通过这个方法可以指定数据详情页查询的字段，默认"*"
    public function getDetailColumns()
    {
        return ['*'];
    }
}
```


