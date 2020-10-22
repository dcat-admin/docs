# 头部和脚部

数据表格支持往头部和脚部两个区块填入你想要的内容

```php
$grid->header(function ($collection) {
    return 'header';
});

$grid->footer(function ($collection) {
    return 'footer'; 
});
```

其中闭包函数的参数`$collection`为`Illuminate\Support\Collection`类实例，是当前页表格数据，下面是两个不同场景的使用举例

## 头部

```php
$grid->header(function ($collection) use ($grid) {
	$query = Model::query();
    	
	// 拿到表格筛选 where 条件数组进行遍历
	$grid->model()->getQueries()->unique()->each(function ($value) use (&$query) {
		if (in_array($value['method'], ['paginate', 'get', 'orderBy', 'orderByDesc'], true)) {
			return;
		}

		$query = call_user_func_array([$query, $value['method']], $value['arguments'] ?? []);
	});
	
	// 查出统计数据
	$data = $query->get();

    // 自定义组件
    return new Card($data);
});
```

自定义头部展示的组件实现
```php
<?php

use Illuminate\Contracts\Support\Renderable;
use Dcat\Admin\Admin;

class Card implements Renderable
{
	public static $js = [
		'xxx/js/card.min.js',
	];
	public static $css = [
		'xxx/css/card.min.css',
	];
	
	protected $data;
	
	public function __construct($data) 
	{
	    $this->data = $data;
	}

	public function script()
	{
		return <<<JS
		console.log('所有JS脚本都加载完了');
		$('xxx').card();
JS;		
	}

	public function render()
	{
		// 在这里可以引入你的js或css文件
		Admin::js(static::$js);
		Admin::css(static::$css);
		
		// 需要在页面执行的JS代码
		// 通过 Admin::script 设置的JS代码会自动在所有JS脚本都加载完毕后执行
		Admin::script($this->script());
		
		return view('...', ['data' => $this->data])->render();
	}
}
```


## 脚部

一个比较常见的场景是在数据表格的脚部显示统计信息，比如在订单表格的脚部加入收入统计，可以参考下面的代码实现：

```php
$grid->footer(function ($collection) use ($grid) {
	$query = Model::query();
	
	// 拿到表格筛选 where 条件数组进行遍历
	$grid->model()->getQueries()->unique()->each(function ($value) use (&$query) {
		if (in_array($value['method'], ['paginate', 'get', 'orderBy', 'orderByDesc'], true)) {
		    return;
		}

		$query = call_user_func_array([$query, $value['method']], $value['arguments'] ?? []);
	});
	
	// 查出统计数据
	$data = $query->get();

    return "<div style='padding: 10px;'>总收入 ： $data</div>";
});
```

如果有比较复杂的脚部需要显示，也可以使用视图对象或者封装成一个类来实现。