# 表格数据源

表格数据是通过 `Dcat\Admin\Contracts\Repository::get` 接口获取的。

如果你的数据源是 `MySQL`，则可以直接使用 `Dcat\Admin\Repositories\EloquentRepository`，
`EloquentRepository` 已实现了对 `Eloquent Model` 的支持。

如果数据来自其他源，则只需重新实现 `Dcat\Admin\Contracts\Repository::get` 接口即可。

> {tip} `Repository`提供更简单更清晰数据操作接口，开发者可以通过实现不同的数据操作接口轻松支持多数据源，具体使用方法请参照[数据仓库](model-repository.md)。

<a name="model"></a>
## 数据来自模型

当数据源支持`Eloquent Model`时，只需创建一个简单的`Repository`类继承`Dcat\Admin\Repositories\EloquentRepository`即可
```php
<?php

namespace App\Admin\Repositories;

use Dcat\Admin\Repositories\EloquentRepository;
use App\Models\Movie as MovieModel;

class Movie extends EloquentRepository
{
    // 这里定义你的模型类名
    protected $eloquentClass = MovieModel::class;
    
    // 通过这个方法可以指定查询的字段，默认"*"
    public function getGridColumns()
    {
        return [$this->getKeyName(), 'name', 'title', 'created_at'];
    }
}
```


### 直接使用模型
如果你还觉得创建 `Repository` 类麻烦，也可以直接把 `Eloquent Model` 的实例传递到 `Grid` 中，底层会自动把 `Eloquent Model` 转化为 `EloquentRepository` 实例 

```php
use App\Models\Movie as MovieModel;

$grid = new Grid(new MovieModel());

...
```


### 修改来源数据

1、使用 `Grid\Model`
```php
use App\Admin\Repositories\Movie;

$grid = new Grid(new Movie());

// 添加默认查询条件
$grid->model()->where('id', '>', 100);

// 设置初始排序条件
$grid->model()->orderBy('id', 'desc');

...

```
其它查询方法可以参考`eloquent`的查询方法。


2、使用 `Model Query`

```php
use App\Models\Movie as MovieModel;

$grid = new Grid(MovieModel::where('id', '>', 100));

...
```


### 关联数据

有以下三种方式让你的表格支持关联数据


1、使用Repository
```php
use App\Admin\Repositories\Movie;

// 相当于 MovieModel::with('categories')
$grid = new Grid(new Movie(['categories']));

$grid->categories;

...
```


2、使用 `Grid\Model`
```php
use App\Admin\Repositories\Movie;

$grid = new Grid(new Movie());

$grid->model()->with('categories');

$grid->categories;

...
```


3、使用 `Model Query`
```php
use App\Models\Movie as MovieModel;

$grid = new Grid(MovieModel::with('categories'));

$grid->categories;

...
```



<a name="api"></a>
## 数据来自外部API

<a name="example"></a>
### 示例

如果数据是来自外部的API，只需要覆写`Repository`中的`get`方法既可, 下面的例子用`豆瓣电影`的API获取并展示数据：

> {tip} 通过`$model->getFilter()->input()`方法可以获取过滤器定义的字段值，请尽量不要通过`request()`方法去获取过滤器筛选参数值，因为当页面中有多个表格时可能无法获取到值。关于过滤器定义请参考[查询过滤](model-grid-filters.md)。

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
     * 定义主键字段名称 
     * 
     * @return string
     */
    public function getPrimaryKeyColumn()
    {
        return '_id';
    }

    /**
     * 查询表格数据
     *
     * @param Grid\Model $model
     * @return LengthAwarePaginator
     */
    public function get(Grid\Model $model)
    {
        // 获取当前页数
        $currentPage = $model->getCurrentPage();
        // 获取每页显示行数
        $perPage = $model->getPerPage();
        
        // 获取排序字段
        list($orderColumn, $orderType) = $model->getSort();

        // 获取"scope"筛选值
        $city = $model->getFilter()->input(Grid\Filter\Scope::QUERY_NAME, '广州');
        // 如果设置了其他过滤器字段，也可以通过“input”方法获取值，如：
        $title = $model->getFilter()->input('title');
        if ($title !== null) {
            // 执行你的筛选逻辑
        }

        $start = ($currentPage - 1) * $perPage;

        $client = new \GuzzleHttp\Client();

        $response = $client->get("{$this->api}?city=$city&start=$start&count=$perPage");
        $data = json_decode((string)$response->getBody(), true);

        // 实例化分页类
        $paginator = new LengthAwarePaginator(
            $data['subjects'] ?? [],
            $data['total'] ?? 0,
            $perPage, // 传入每页显示行数
            $currentPage // 传入当前页码
        );

        // 设置当前页面路径
        $paginator->setPath(\url()->current());

        return $paginator;
    }

}
```
<a name="grid-model"></a>
### $grid->model() 内置方法

<a name="getCurrentPage"></a>
#### getCurrentPage
获取当前页码
- 返回值： `int|null` 如果不允许分页返回null
```php
$page = $model->getCurrentPage();
```

<a name="getPerPage"></a>
#### getPerPage
获取每页显示行数
- 返回值： `int|null` 如果不允许分页返回null
```php
$limit = $model->getPerPage();
```

<a name="getSort"></a>
#### getSort
获取排序字段
- 返回值： `array` `[$orderColumn, 'desc'|'asc']` || `[null, null]`

```php
// $orderColumn 字段名称，如没有进行排序则为 null
// $orderType 正序或倒序： "desc"、"asc"，如没有进行排序则为 null
list($orderColumn, $orderType) = $model->getSort();
```

<a name="getFilter"></a>
#### getFilter
获取过滤器对象
- 返回值 `Dcat\Admin\Grid\Filter`

```php
// 获取"scope"筛选值
$city = $model->getFilter()->input(Grid\Filter\Scope::QUERY_NAME, '广州');

// 如果设置了其他过滤器字段，也可以通过“input”方法获取值，如：
$title = $model->getFilter()->input('title');
if ($title !== null) {
    // 执行你的筛选逻辑
}
```


<a name="sql"></a>
## 数据来自复杂SQL查询

如果来源数据需要执行比较复杂的SQL语句获取，那么有两个办法, 第一个办法就是上面的方法，覆盖掉`Repository`的`get`方法实现


第二个方式是在数据库中建立视图和model绑定（未测试过，理论上可行）
