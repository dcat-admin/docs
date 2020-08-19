# 异步加载基本使用

> {tip} Since `v1.7.0` 此功能支持在页面任意位置使用，支持所有内置组件，并且支持**静态资源按需加载**。

通过异步加载功能可以让页面局部组件使用`ajax`异步渲染功能，从而提高页面加载效率，最常见的例子是表单弹窗结合异步加载功能的使用。下面我们通过一个简单的示例来演示异步加载功能的使用


先定义一个异步渲染类

```php
<?php

namespace App\Admin\Renderable;

use Dcat\Admin\Widgets\Table;
use Dcat\Admin\Support\LazyRenderable;

class PostTable extends LazyRenderable
{
    protected $title = ['#', '标题', '内容'];
    
    public function render()
    {
    	// 获取外部传递的参数
    	$id = $this->id;
        
    	$data = [
    	    ['1', '标题1', '内容1'],
    	    ['2', '标题2', '内容2'],
		];
    	
    	// 这里可以返回内置组件，也可以返回视图文件或HTML字符串
        return Table::make($this->title, $data);
	}
}
```

使用

```php
use App\Admin\Renderable\PostTable;
use Dcat\Admin\Widgets\Lazy;
use Dcat\Admin\Layout\Content;

public function index(Content $content)
{
    // 传递参数
    $table = PostTable::make(['id' => ...]);
    
	return $content->body(Lazy::make($table));
}
```

也可以放在内置组件中，
如果是 `Dcat\Admin\Widgets\Card`、`Dcat\Admin\Widgets\Box`、`Dcat\Admin\Widgets\Modal`等组件，则可以略过`Dcat\Admin\Widgets\Lazy`组件，直接传递渲染类实例

```php
use App\Admin\Renderable\PostTable;
use Dcat\Admin\Widgets\Card;
use Dcat\Admin\Layout\Content;

public function index(Content $content)
{
    // 传递参数
    $table = PostTable::make(['id' => ...]);
    
    // Card 组件支持直接传递 LazyRenderable 实例，可以略过 Lazy 对象
	return $content->body(Card::make($table));
}
```

当然也可以防止在视图或者是`HTML`代码中
```php
use App\Admin\Renderable\PostTable;
use Dcat\Admin\Widgets\Lazy;
use Dcat\Admin\Layout\Content;

public function index(Content $content)
{
    // 传递参数
    $table = Lazy::make(PostTable::make(['id' => ...]));
    
	return $content->body(view('admin.xxx', ['table' => $table]));
}
```



## LazyRenderable功能接口

### 参数传递 (payload)

```php
use App\Admin\Renderable\PostTable;

PostTable::make(['key1' => '值', ...]);

// 也可以通过 payload 方法传递
PostTable::make()->payload(['key1' => '值', ...]);
```

获取参数

```php
class PostTable extends LazyRenderable
{
    protected $title = ['#', '标题', '内容'];
    
    public function render()
    {
    	// 获取外部传递的参数
    	$key1 = $this->key1;
        
    	...
	}
}
```


### 载入JS和CSS




