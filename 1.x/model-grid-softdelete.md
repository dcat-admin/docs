# 数据软删除


先参考`Laravel`文档实现模型的[软删除](https://learnku.com/docs/laravel/6.x/eloquent/5176#soft-deleting):

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Post extends Model
{
    use SoftDeletes;
}
```

这样在`grid`列表中显示的数据都是未被删除的数据

```php
return Grid::make(new Post(), function (Grid $grid) {
	$grid->id('ID')->sortable();
	$grid->title('Title');
	$grid->created_at('Created at');
	$grid->updated_at('Updated at');
});
```

## 回收站入口

接下来需要增加一个入口，能让我们看到被软删除的数据，这里可以使用`model-grid`的[范围过滤器](model-grid-filters.md#scope)来实现

```php
$grid->filter(function () {

    // 范围过滤器，调用模型的`onlyTrashed`方法，查询出被软删除的数据。
    $filter->scope('trashed', '回收站')->onlyTrashed();

});
```

在表头的筛选按钮的下拉菜单中就会出现一个`回收站`按钮，点击它，就会调用模型的`onlyTrashed`方法，从表中查询出被删除的数据，也就是回收站中的数据。

<img style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="40%" src="{{public}}/assets/img/screenshots/trash-button.png">



## 行恢复操作

按照下面的方法，我们可以在回收站中的每一行数据加上一个恢复操作，方便恢复数据

先定义操作类`app/Admin/Actions/Post/Restore.php`：

```php
<?php

namespace App\Admin\Actions\Post;

use Dcat\Admin\Grid\RowAction;
use Illuminate\Http\Request;

class Restore extends RowAction
{
    protected $title = '恢复';
        
	protected $model;
	
	// 注意构造方法的参数必须要有默认值
	public function __construct(string $model = null) 
	{
		$this->model = $model;
	}

    public function handle(Request $request)
    {
        $key = $this->getKey();
        $model = $request->get('model');
        
        $model::withTrashed()->findOrFail($key)->restore();

        return $this->response()->success('已恢复')->refresh();
    }

    public function confirm()
    {
        return ['确定恢复吗？'];
    }
    
    public function parameters()
    {
        return [
            'model' => $this->model,    
		];
	}
}
```

添加到行操作:

```php
use App\Models\Post;
use App\Admin\Actions\Post\Restore;

$grid->actions(function (Grid\Displayers\Actions $actions) {
    if (request('_scope_') == 'trashed') {
        $actions->append(new Restore(Post::class));
    }
});
```

## 批量恢复操作

先定义操作类`app/Admin/Actions/Post/BatchRestore.php`：

```php
<?php

namespace App\Admin\Actions\Post;

use Dcat\Admin\Grid\BatchAction;
use Illuminate\Http\Request;

class BatchRestore extends BatchAction
{
    protected $title = '恢复';
    
    protected $model;
    
    // 注意构造方法的参数必须要有默认值
    public function __construct(string $model = null) 
    {
        $this->model = $model;
    }

    public function handle(Request $request)
    {
        $model = $request->get('model');
        
        foreach ((array) $this->getKey() as $key) {
			$model::withTrashed()->findOrFail($key)->restore();
		}
        
        return $this->response()->success('已恢复')->refresh();
    }

    public function confirm()
    {
        return ['确定恢复吗？'];
    }
    
	public function parameters()
	{
		return [
			'model' => $this->model,    
		];
	}
}
```

添加到批量操作:

```php
use App\Models\Post;
use App\Admin\Actions\Post\BatchRestore;

$grid->batchActions(function (Grid\Tools\BatchActions $batch) {
	if (request('_scope_') == 'trashed') {
		$batch->add(new BatchRestore(Post::class));
	}
});
```
