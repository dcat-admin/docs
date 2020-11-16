# 树状表格

树状表格支持分页和点击加载功能，特别适合用来展示数据量较大的多层级结构数据。

<a href="http://103.39.211.179:8080/admin/tree" target="_blank">
    <img class="img img-full" src="{{public}}/assets/img/screenshots/grid-tree.png">
</a>

### 表结构和模型
要使用树状表格，要遵守约定的表结构：

> {tip} 此表结构和模型可完全兼容 <code>[模型树](model-tree.md)</code> 。

```sql
CREATE TABLE `demo_categories` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `parent_id` int(11) NOT NULL DEFAULT '0',
  `order` int(11) NOT NULL DEFAULT '0', // order 字段不是必须的，不设置也可以
  `title` varchar(50) COLLATE utf8_unicode_ci NOT NULL,
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci
```
上面的表格结构里面有三个必要的字段`parent_id`、`title`,其它字段没有要求。

对应的模型为`app/Models/Category.php`:

> {tip} 为了便于阅读，这里不再展示 `Repository` 代码。

```php
<?php

namespace App\Models\Demo;

use Dcat\Admin\Traits\ModelTree;
use Illuminate\Database\Eloquent\Model;

class Category extends Model
{
    use ModelTree;

    protected $table = 'demo_categories';
}
```
表结构中的三个字段`parent_id`、`order`、`title`的字段名也是可以修改的：
```php
<?php

namespace App\Models\Demo;

use Dcat\Admin\Traits\ModelTree;
use Illuminate\Database\Eloquent\Model;

class Category extends Model
{
    use ModelTree;

    protected $table = 'demo_categories';
    
    protected $titleColumn = 'name';
    
    protected $orderColumn = 'sort';
    
    protected $parentColumn = 'pid';
}
```

如果你的数据表不需要 `order` 字段排序，则在模型中添加如下代码即可
```php
<?php

namespace App\Models\Demo;

use Dcat\Admin\Traits\ModelTree;
use Illuminate\Database\Eloquent\Model;

class Category extends Model
{
    use ModelTree;

    protected $table = 'demo_categories';
    
    // 返回空值即可禁用 order 字段
    public function getOrderColumn()
    {
        return null;
    }
}
```


### 使用

通过调用 `Grid\Column::tree` 方法即可开启树状表格功能，开启之后默认只查询最顶级节点的数据，子节点数据需要点击加载

```php
<?php

namespace App\Admin\Controllers\Demo;

use App\Models\Category;
use Dcat\Admin\Grid;
use Dcat\Admin\Controllers\AdminController;

class CategoryController extends AdminController
{
    protected function grid()
    {
        return Grid::make(new Category(), function (Grid $grid) {
            $grid->id('ID')->bold()->sortable();
            $grid->title->tree(); // 开启树状表格功能 
            $grid->order;
    
            $grid->created_at;
            $grid->updated_at->sortable();
            
            $grid->filter(function (Grid\Filter $filter) {
                $filter->like('slug');
                $filter->like('name');
                $filter->like('http_path');
            });
        });
    }
}
```

上面的代码执行的 `sql` 如下（默认只查询 `parent = 0` 的数据）：
```sql
select count(*) as aggregate from `demo_categories` where `parent_id` = 0

select * from `demo_categories` where `parent_id` = 0 order by `order` asc, `id` asc limit 20 offset 0
```


`Grid\Column::tree` 方法参数

+ `bool` `$showAll`  `false`  是否一次性展示下一层级的所有节点，默认分页展示
+ `bool` `$sortable` `true`   是否开启排序功能

```php
// 禁用分页功能，一次性加载所有下一层级节点
$grid->title->tree(true);


// 不需要 order 字段排序，第二个参数传 false 即可
$grid->title->tree(false, false);
```


### orderable排序

`orderable` 排序功能依赖 <a href="https://github.com/spatie/eloquent-sortable" target="__blank">spatie/eloquent-sortable</a> 组件，需要修改模型：

```php
use Spatie\EloquentSortable\Sortable;

class Category extends Model implements Sortable
{
    use ModelTree;
    
    // 设置排序字段，默认order
    protected $orderColumn = 'sort';
}
```

下面是使用示例

```php
class CategoryController extends AdminController
{
    protected function grid()
    {
        return Grid::make(new Category(), function (Grid $grid) {
            $grid->id('ID')->bold()->sortable();
            $grid->title->tree(); // 开启树状表格功能 
            $grid->order->orderable(); // 开启排序功能
    
            ...;
        });
    }
}
```

### 关于数据搜索

如果在树状表格中使用了搜索功能（`Grid::filter`、`Grid\Column::filter`、`Grid::quickSearch`），为了让用户能搜索到想要的数据，则会<b>取消只查最顶级数据的操作</b>。

> {tip} 使用 [查询过滤](model-grid-filters.md)、[列过滤器](model-grid-column-filter.md)、[快捷搜索](model-grid-quick-search.md) 等搜索功能都会<b>取消只查最顶级数据的操作</b>，只有 [筛选器](model-grid-selector.md) 和 [范围查询scope](model-grid-filters.md#scope) 等功能例外。


例如下面的代码开启了快捷搜索
```php
class CategoryController extends AdminController
{
    protected function grid()
    {
        return Grid::make(new Category(), function (Grid $grid) {
            $grid->id('ID')->bold()->sortable();
            $grid->title->tree(); // 开启树状表格功能 
            $grid->order->orderable(); // 开启排序功能
    
            $grid->quickSearch(['id', 'title']);
            ...;
        });
    }
}
```

且用户在浏览器中使用了快捷搜索，则产生`sql`如下

```sql
select count(*) as aggregate from `demo_categories` where `id` like "%xxx%" or `title` like "%xxx%"

select * from `demo_categories` where `id` like "%xxx%" or `title` like "%xxx%" order by `order` asc, `id` asc limit 20 offset 0
```

### 与模型树功能的差别

[模型树](model-tree.md)同样可用于展示多层级结构数据，并且支持用拖拽的方式实现数据的层级、排序等操作，但是不支持分页和点击加载功能，只能一次性加载完所有数据，
因此[模型树](model-tree.md)并不适合用来展示数据量较大的数据。


而树状表格支持分页和点击加载功能，适合用来展示数据量较大的多层级结构数据，但不支持用拖拽的方式实现数据的层级、排序操作。
