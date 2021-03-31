# 列的显示和扩展


数据表格内置了很多对于列的操作方法，可以通过这些方法很灵活的操作列数据。



## 根据条件显示不同的组件
有些情况我们需要根据某个条件去判断是否使用列的某个显示功能：
> {tip} 需要注意的是，`Grid\Column::if`只对列的显示相关功能有效，其他方法如表头的相关操作都不能使用此方法！

```php
$grid->column('config')
    ->if(function ($column) {
        // 获取当前行其他字段值
        $username = $this->username;
        
    	// $column->getValue() 是当前字段的值
        return $column->getValue();
    })
    ->display($view)
    ->copyable()
    ->else()
    ->display('');
```
上面写法等同于
```php
$grid->column('config')
    ->if(function ($column) {
        return $column->getValue();
    })
    ->then(function (Grid\Column $column) {
        $column->display($view)->copyable();
    })
    ->else(function (Grid\Column $column) {
        $column->display('');
    });
```

支持多个`if`
```php
$grid->column('config')
    ->if(...)
    ->then(...)
    ->else(...)
    
    ->if(...)
    ->then(...)
    ->else(...);
```

终结条件判断`end`
```php
$grid->column('status')
    ->if(...) // 条件1
    ->display(...)
    ->display(...)
    
    ->if(...) // 条件2
    ->display(...)
    ->display(...)
    
    ->end() // 终结前面的条件判断
    ->display(...); // 所有条件都能生效
```


## 列显示

`model-grid`内置了若干方法来帮助你扩展列功能

### display

`Dcat\Admin\Grid\Column`对象内置了`display()`方法来通过传入的回调函数来处理当前列的值，
```php
$grid->column('title')->display(function ($title) {

    return "<span style='color:blue'>$title</span>";
    
});
```
在传入的匿名函数中可以通过任何方式对数据进行处理，另外匿名函数绑定了当前列的数据作为父对象，可以在函数中调用当前行的数据：
```php
$grid->column('first_name');

$grid->column('last_name');

// 不存在的`full_name`字段
$grid->column('full_name')->display(function () {
    return $this->first_name . ' ' . $this->last_name;
});
```

### 设置列的HTML属性
设列的`html`属性
```php
$grid->column('name')->setAttributes(['style' => 'font-size:14px']);
```

### 列视图
`view`方法可以引入一个视图文件。
```php
$grid->column('content')->view('admin.fields.content');
```

默认会传入视图的三个变量：
 - `$model` 当前行数据
 - `$name` 字段名称
 - `$value` 为当前列的值
 
模板代码如下：
```blade
<label>名称：{{ $name }}</label>
<label>值：{{ $value }}</label>
<label>其他字段：{{ $model->title }}</label>
```



### 图片

```php
$grid->column('picture')->image();

//设置服务器和宽高
$grid->column('picture')->image('http://xxx.com', 100, 100);

// 显示多图
$grid->column('pictures')->display(function ($pictures) {
    
    return json_decode($pictures, true);
    
})->image('http://xxx.com', 100, 100);
```

### 显示label标签

支持`Dcat\Admin\Color`类中内置的所有颜色

```php
use Dcat\Admin\Admin;

$grid->column('name')->label();

// 设置颜色，直接传别名
$grid->column('name')->label('danger');

// 也可以这样使用
$grid->column('name')->label(Admin::color()->danger());

// 也可以直接传颜色代码
$grid->column('name')->label('#222');
```

给不同的值设置不同的颜色
```php
use Dcat\Admin\Admin;

$grid->column('state')->using([1 => '未处理', 2 => '已处理', ...])->label([
    'default' => 'primary', // 设置默认颜色，不设置则默认为 default
    
	1 => 'primary',
	2 => 'danger',
	3 => 'success',
	4 => Admin::color()->info(),
]);
```

### 显示badge标签

支持`Dcat\Admin\Color`类中内置的所有颜色

```php
$grid->column('name')->badge();

// 设置颜色，直接传别名
$grid->column('name')->badge('danger');

// 也可以这样使用
$grid->column('name')->badge(Admin::color()->danger());

// 也可以直接传颜色代码
$grid->column('name')->badge('#222');
```

给不同的值设置不同的颜色
```php
use Dcat\Admin\Admin;

$grid->state->using([1 => '未处理', 2 => '已处理', ...])->badge([
    'default' => 'primary', // 设置默认颜色，不设置则默认为 default	
    
    1 => 'primary',
	2 => 'danger',
	3 => 'success',
	4 => Admin::color()->info(),
]);
```

<a name="bool"></a>
### 布尔值显示 (bool)


将这一列转为`bool`值之后显示为`✓`和`✗`。

```php
$grid->column('approved')->bool();
```

你也可以按照这一列的值指定显示，比如字段的值为`Y`和`N`表示`true`和`false`

```php
$grid->column('approved')->bool(['Y' => true, 'N' => false]);
```

效果
![](https://cdn.learnku.com/uploads/images/202007/12/38389/U0OSrJwzyt.png!large)




### 圆点前缀 (dot)

通过`dot`方法可以在列文字前面加上一个带颜色的圆点

```php
use Dcat\Admin\Admin;

$grid->column('state')
	->using([1 => '未处理', 2 => '已处理', ...])
	->dot(
		[
			1 => 'primary',
			2 => 'danger',
			3 => 'success',
			4 => Admin::color()->info(),
		], 
	    'primary' // 第二个参数为默认值
	);
```

效果
![](https://cdn.learnku.com/uploads/images/202004/30/38389/ByUqo6bZc8.png!large)


<a name="expand"></a>
### 列展开 (expand)
`expand`方法可以把内容隐藏，点击按钮的时候显示在表格下一行
```php
$grid->column('content')
    ->display('详情') // 设置按钮名称
    ->expand(function () {
        // 返回显示的详情
        // 这里返回 content 字段内容，并用 Card 包裹起来
        $card = new Card(null, $this->content);
    
        return "<div style='padding:10px 10px 0'>$card</div>";
    });
```
也可以通过以下方式设置按钮
```php
$grid->column('content')->expand(function (Grid\Displayers\Expand $expand) {
    // 设置按钮名称
    $expand->button('详情');

    // 返回显示的详情
    // 这里返回 content 字段内容，并用 Card 包裹起来
    $card = new Card(null, $this->content);

    return "<div style='padding:10px 10px 0'>$card</div>";
});
```


#### 异步加载

> {tip} 更多用法请参考文档[异步加载](lazy.md)

定义渲染类，继承`Dcat\Admin\Support\LazyRenderable`

```php
use App\Models\Post as PostModel;
use Dcat\Admin\Support\LazyRenderable;
use Dcat\Admin\Widgets\Table;

class Post extends LazyRenderable
{
    public function render()
    {
        // 获取ID
        $id = $this->key;
        
        // 获取其他自定义参数
        $type = $this->post_type;

        $data = PostModel::where('user_id', $id)
            ->where('type', $type)
            ->get(['title', 'body', 'body', 'created_at'])
            ->toArray();

        $titles = [
            'User ID',
            'Title',
            'Body',
            'Created At',
        ];

        return Table::make($titles, $data);
    }
}
```

使用
```php
$grid->post->display('View')->expand(Post::make(['post_type' => 1]));

// 可以在闭包内返回异步加载类的实例
$grid->post->expand(function () {
    // 允许在比包内返回异步加载类的实例

    return Post::make(['title' => $this->title]);
});
```

效果

![](https://cdn.learnku.com/uploads/images/202006/14/38389/KMHagem4OZ.gif!large)


#### 异步加载工具表单

定义[工具表单](widgets-form.md)类如下

> {tip} 更多用法请参考[异步加载](lazy.md)

```php
<?php

namespace App\Admin\Forms;

use Dcat\Admin\Contracts\LazyRenderable;
use Dcat\Admin\Traits\LazyWidget;
use Dcat\Admin\Widgets\Form;

class UserProfile extends Form implements LazyRenderable
{
    use LazyWidget;

    public function handle(array $input)
    {
        // 接收外部传递参数
		$type = $this->payload['type'] ?? null;
        
        return $this->response()->success('保存成功');
    }

    public function form()
    {
        // 接收外部传递参数
        $type = $this->payload['type'] ?? null;
        
        $this->text('name', trans('admin.name'))->required()->help('用户昵称');
        $this->image('avatar', trans('admin.avatar'))->autoUpload();

        $this->password('old_password', trans('admin.old_password'));

        $this->password('password', trans('admin.password'))
            ->minLength(5)
            ->maxLength(20)
            ->customFormat(function ($v) {
                if ($v == $this->password) {
                    return;
                }

                return $v;
            })
            ->help('请输入5-20个字符');
        $this->password('password_confirmation', trans('admin.password_confirmation'))
            ->same('password')
            ->help('请输入确认密码');
    }
}
```

使用

```php
$grid->user->display('View')->expand(UserProfile::make(['type' => 1]));
```


<a name="modal"></a>
### 弹出模态框 (modal)
`modal`方法可以把内容隐藏，点击按钮的时候显示在表格下一行
```php
$grid->column('content')
    ->display('查看') // 设置按钮名称
    ->modal(function ($modal) {
        // 设置弹窗标题
        $modal->title('标题 '.$this->username);
        
       // 自定义图标
       $modal->icon('feather icon-x');
    
        $card = new Card(null, $this->content);
    
        return "<div style='padding:10px 10px 0'>$card</div>";
    });

// 也可以通过这种方式设置弹窗标题
$grid->column('content')
    ->display('查看') // 设置按钮名称
    ->modal('弹窗标题', ...);
```


#### 异步加载

> {tip}  关于异步加载的更具体用法，请参考文档[异步加载](lazy.md)       

定义渲染类，继承`Dcat\Admin\Support\LazyRenderable`

```php
use App\Models\Post as PostModel;
use Dcat\Admin\Support\LazyRenderable;
use Dcat\Admin\Widgets\Table;

class Post extends LazyRenderable
{
    public function render()
    {
        // 获取ID
        $id = $this->key;
        
        // 获取其他自定义参数
        $type = $this->post_type;

        $data = PostModel::where('user_id', $id)
            ->where('type', $type)
            ->get(['title', 'body', 'body', 'created_at'])
            ->toArray();

        $titles = [
            'User ID',
            'Title',
            'Body',
            'Created At',
        ];

        return Table::make($titles, $data);
    }
}
```

使用
```php
$grid->post->display('View')->modal('Post', Post::make(['post_type' => 2]));

// 可以在闭包内返回异步加载类的实例
$grid->post->modal(function ($modal) {
    $modal->title('自定义弹窗标题');

    // 允许在比包内返回异步加载类的实例
    return Post::make(['title' => $this->title]);
});
```

效果
![](https://cdn.learnku.com/uploads/images/202006/14/38389/DvvyZUTXpG.gif!large)



#### 异步加载工具表单

定义[工具表单](widgets-form.md)类如下

> {tip} 更多用法请参考[异步加载](lazy.md)

```php
<?php

namespace App\Admin\Forms;

use Dcat\Admin\Contracts\LazyRenderable;
use Dcat\Admin\Traits\LazyWidget;
use Dcat\Admin\Widgets\Form;

class UserProfile extends Form implements LazyRenderable
{
    use LazyWidget;

    public function handle(array $input)
    {
        // 接收外部传递参数
		$type = $this->payload['type'] ?? null;
        
        return $this->response()->success('保存成功');
    }

    public function form()
    {
        // 接收外部传递参数
        $type = $this->payload['type'] ?? null;
        
        $this->text('name', trans('admin.name'))->required()->help('用户昵称');
        $this->image('avatar', trans('admin.avatar'))->autoUpload();

        $this->password('old_password', trans('admin.old_password'));

        $this->password('password', trans('admin.password'))
            ->minLength(5)
            ->maxLength(20)
            ->customFormat(function ($v) {
                if ($v == $this->password) {
                    return;
                }

                return $v;
            })
            ->help('请输入5-20个字符');
        $this->password('password_confirmation', trans('admin.password_confirmation'))
            ->same('password')
            ->help('请输入确认密码');
    }
}
```

使用

```php
$grid->user->display('View')->modal(UserProfile::make(['type' => 1]));
```


### 进度条 (progressBar)
`progressBar`进度条
```php
$grid->rate->progressBar();

//设置颜色，默认`primary`,可选`danger`、`warning`、`info`、`primary`、`success`
$grid->rate->progressBar('success');

// 设置进度条尺寸和最大值
$grid->rate->progressBar('success', 'sm', 100);
```

### showTreeInDialog
`showTreeInDialog`方法可以把一个带有层级关系的数组呈现为树形弹窗，比如权限就可以用此方法展示
```php
// 查出所有的权限数据
$nodes = (new $permissionModel)->allNodes();

// 传入二维数组（未分层级）
$grid->permissions->showTreeInDialog($nodes);

// 也可以传入闭包
$grid->permissions->showTreeInDialog(function (Grid\Displayers\DialogTree $tree) use (&$nodes, $roleModel) {
    // 设置所有节点
    $tree->nodes($nodes);
    
    // 设置节点数据字段名称，默认"id"，"name"，"parent_id"
    $tree->setIdColumn('id');
    $tree->setTitleColumn('title');
    $tree->setParentColumn('parent_id');

    // $this->roles 可以获取当前行的字段值
    foreach (array_column($this->roles, 'slug') as $slug) {
        if ($roleModel::isAdministrator($slug)) {
            // 选中所有节点
            $tree->checkAll();
        }
    }
});
```
![](https://cdn.learnku.com/uploads/images/202004/26/38389/s1htW08Iko.png!large)


### 内容映射 (using)
```php
$grid->status->using([0 => '未激活', 1 => '正常']);

// 第二个参数为默认值
$grid->gender->using([1 => '男', 2 => '女'], '未知');
```

### 字符串分割为数组
`explode`方法可以把字符串分割为数组。
```php
$grid->tag->explode()->label();

// 可以指定分隔符，默认","
$grid->tag->explode('|')->label();
```

### prepend

`prepend` 方法用于给 `string` 或 `array` 类型的值前面插入内容。

```php
// 当字段值是一个字符串
$grid->email->prepend('mailto:');

// 当字段值是一个数组
$grid->arr->prepend('first item');
```

`prepend`方法允许传入闭包参数
```php
$grid->email->prepend(function ($value, $original) {
    // $value 是当前字段值
    // $original 是当前字段从数据库中查询出来的原始值
    
    // 获取其他字段值
    $username = $this->username;
    
    return "[{$username}]";
});
```

### append
`append` 方法用于给 `string` 或 `array` 类型的值后面插入内容。

```php
// 当字段值是一个字符串
$grid->email->append('@gmail.com');

// 当字段值是一个数组
$grid->arr->append('last item');
```

`append`方法允许传入闭包参数
```php
$grid->email->append(function ($value, $original) {
    // $value 是当前字段值
    // $original 是当前字段从数据库中查询出来的原始值
    
    // 获取其他字段值
    $username = $this->username;
    
    return "[{$username}]";
});
```

### 字符串或数组截取 (limit)

```php
// 最多显示50个字符
$grid->column('content')->limit(50, '...');

// 如果字段值是数组也支持
$grid->tags->limit(3);
```

### 列二维码 (qrcode)
```php
$grid->website->qrcode(function () {
    return $this->url;
}, 200, 200);
```

### 可复制 (copyable)
```php
$grid->website->copyable();
```


### 可排序 (orderable)

通过`Column::orderable`可以开启字段可排序功能，此功能需要在你的模型类中`use ModelTree`或`use SortableTrait`，并且需要继承`Spatie\EloquentSortable\Sortable`接口。


#### SortableTrait

如果你的数据不是层级结构数据，可以直接使用`use SortableTrait`，更多用法可参考[eloquent-sortable](https://github.com/spatie/eloquent-sortable)。

模型
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Spatie\EloquentSortable\Sortable;
use Spatie\EloquentSortable\SortableTrait;

class MyModel extends Model implements Sortable
{
    use SortableTrait;

    protected $sortable = [
        // 设置排序字段名称
        'order_column_name' => 'order',
        // 是否在创建时自动排序，此参数建议设置为true
        'sort_when_creating' => true,
    ];
}
```

使用
```php
$grid->model()->orderBy('order');

$grid->order->orderable();
```

#### ModelTree

如果你的数据是层级结构数据，可以直接使用`use Model`，下面以权限模型为例来演示用法

> {tip} `ModelTree`实际上也是继承了[eloquent-sortable](https://github.com/spatie/eloquent-sortable)的功能。

```php
<?php

namespace Dcat\Admin\Models;

use Dcat\Admin\Traits\HasDateTimeFormatter;
use Dcat\Admin\Traits\ModelTree;
use Spatie\EloquentSortable\Sortable;

class Permission extends Model implements Sortable
{
    use HasDateTimeFormatter,
        ModelTree {
            ModelTree::boot as treeBoot;
        }
        
    ...    
}        
```

使用
```php
$grid->order->orderable();
```



### 链接 (link)

将字段显示为一个链接。

```php
// link方法不传参数时，链接的`href`和`text`都是当前列的值
$grid->column('homepage')->link();

// 传入闭包
$grid->column('homepage')->link(function ($value) {
    return admin_url('users/'.$value);
});
```

<a name="action"></a>
### 行操作 (action)

> {tip} 在使用这个方法之前，请先阅读[自定义操作-行操作](model-grid-actions.md)

这个功能可以将某一列显示为一个**行操作**的按钮，比如下面所示是一个标星和取消标星的列操作，
点击这一列的星标图标之后, 后台会切换字段的状态，页面图标也跟着切换，具体实现代码如下：


```php
<?php

namespace App\Admin\Extensions\Grid\RowAction;

use Dcat\Admin\Grid\RowAction;
use Illuminate\Http\Request;

class Star extends RowAction
{
    protected function html()
    {
        $icon = ($this->row->{$this->getColumnName()}) ? 'fa-star' : 'fa-star-o';

        return <<<HTML
<i class="{$this->getElementClass()} fa {$icon}"></i>
HTML;
    }

    public function handle(Request $request)
    {
        try {
            $class = $request->class;
            $column = $request->column;
            $id = $this->getKey();

            $model = $class::find($id);
            $model->{$column} = (int) !$model->{$column};
            $model->save();

            return $this->response()->success("success")->refresh();
        } catch (\Exception $e) {
            return $this->response()->error($e->getMessage());
        }
    }

    public function parameters()
    {
        return [
            'class' => $this->modelClass(),
            'column' => $this->getColumnName(),
        ];
    }

    public function getColumnName()
    {
        return $this->column->getName();
    }

    public function modelClass()
    {
        return get_class($this->parent->model()->repository()->model());
    }
}
```

使用

```php
protected function grid()
{
    $grid = new Grid(new $this->model());

    $grid->column('star', '-')->action(Star::class);  // here
    $grid->column('id', 'ID')->sortable()->bold();
    
    return $grid;
}
```

效果

![](https://cdn.learnku.com/uploads/images/202005/17/38389/g8F7p8gnsE.png!large)


## 帮助方法

### 字符串操作
如果当前里的输出数据为字符串，那么可以通过链式方法调用`Illuminate\Support\Str`的方法。

比如有如下一列，显示`title`字段的字符串值:

```php
$grid->title();
```

在`title`列输出的字符串基础上调用`Str::title()`方法
```php
$grid->title()->title();
```
调用方法之后输出的还是字符串，所以可以继续调用`Illuminate\Support\Str`的方法：
```php
$grid->title()->title()->ucfirst();

$grid->title()->title()->ucfirst()->substr(1, 10);

```

### 数组操作
如果当前列输出的是数组，可以直接链式调用`Illuminate\Support\Collection`方法。

比如`tags`列是从一对多关系取出来的数组数据：
```php
$grid->tags();

array (
  0 => 
  array (
    'id' => '16',
    'name' => 'php',
    'created_at' => '2016-11-13 14:03:03',
    'updated_at' => '2016-12-25 04:29:35',
    
  ),
  1 => 
  array (
    'id' => '17',
    'name' => 'python',
    'created_at' => '2016-11-13 14:03:09',
    'updated_at' => '2016-12-25 04:30:27',
  ),
)

```

调用`Collection::pluck()`方法取出数组的中的`name`列
```php
$grid->tags()->pluck('name');

array (
    0 => 'php',
    1 => 'python',
  ),

```
取出`name`列之后输出的还是数组，还能继续调用用`Illuminate\Support\Collection`的方法

```php
$grid->tags()->pluck('name')->map('ucwords');

array (
    0 => 'Php',
    1 => 'Python',
  ),
```
将数组输出为字符串
```php
$grid->tags()->pluck('name')->map('ucwords')->implode('-');

"Php-Python"
```

### 混合使用

在上面两种类型的方法调用中，只要上一步输出的是确定类型的值，便可以调用类型对应的方法，所以可以很灵活的混合使用。

比如`images`字段是存储多图片地址数组的JSON格式字符串类型：
```php

$grid->images();

// "['foo.jpg', 'bar.png']"

// 链式方法调用来显示多图
$grid->images()->display(function ($images) {
    return json_decode($images, true);
    
})->map(function ($path) {
    return 'http://localhost/images/'. $path;
    
})->image();

```


## 扩展列的显示功能

可以通过两种方式扩展列功能，第一种是通过匿名函数的方式。
>扩展列功能方法后IDE默认是不会自动补全的，这时候可以通过`php artisan admin:ide-helper`生成IDE提示文件。

### 匿名函数
在`app/Admin/bootstrap.php`加入以下代码:
```php
use Dcat\Admin\Grid\Column;

// 第二个参数是自定义参数
Column::extend('color', function ($value, $color) {
    return "<span style='color: $color'>$value</span>";
});
```
然后在`model-grid`中使用这个扩展：
```php
$grid->title()->color('#ccc');
```

### 扩展类
如果列显示逻辑比较复杂，可以通过扩展类来实现。

扩展类`app/Admin/Extensions/Popover.php`:
```php
<?php

namespace App\Admin\Extensions;

use Dcat\Admin\Admin;
use Dcat\Admin\Grid\Displayers\AbstractDisplayer;

class Popover extends AbstractDisplayer
{
    public function display($placement = 'left')
    {
        Admin::script("$('[data-toggle=\"popover\"]').popover()");

        return <<<EOT
<button type="button"
    class="btn btn-secondary"
    title="popover"
    data-container="body"
    data-toggle="popover"
    data-placement="$placement"
    data-content="{$this->value}"
    >
  弹出提示
</button>
EOT;

    }
}
```
然后在`app/Admin/bootstrap.php`注册扩展类：
```php
use Dcat\Admin\Grid\Column;
use App\Admin\Extensions\Popover;

Column::extend('popover', Popover::class);
```
然后就能在`model-grid`中使用了：
```php
$grid->desciption()->popover('right');
```

### 指定列名
出了上述两种方式扩展列功能，我们还可以通过指定列名称的方式扩展列功能

在`app/Admin/bootstrap.php`加入以下代码:
```php
use Dcat\Admin\Grid\Column;

// 这个扩展方法等同于
// $grid->title()->display(function ($value) {
//    return "<span style='color:red'>$value</span>"
// });
Column::define('title', function ($value) {
    return "<span style='color:red'>$value</span>"
});

// 这个扩展方法等同于
// $grid->status()->switch();
Column::define('status', Dcat\Admin\Grid\Displayers\SwitchDisplay::class);
```
然后在`model-grid`中`title`和`status`字段会自动使用以上扩展：
```php
$grid->title();
$grid->status();
```

