# 工具栏

## 工具按钮

在`model-grid`的头部默认有`批量删除`和`刷新`两个操作工具，如果有更多的操作需求，`Grid` 提供了自定义工具的功能,下面的示例添加一个性别分类选择的按钮组工具。

先定义工具类`app/Admin/Extensions/Tools/UserGender.php`：

```php
<?php

namespace App\Admin\Extensions\Tools;

use Dcat\Admin\Admin;
use Dcat\Admin\Grid\Tools\AbstractTool;
use Illuminate\Support\Facades\Request;

class UserGender extends AbstractTool
{
    protected function script()
    {
        $url = Request::fullUrlWithQuery(['gender' => '_gender_']);

        return <<<JS
$('input:radio.user-gender').change(function () {
    var url = "$url".replace('_gender_', $(this).val());

    LA.reload(url);
});
JS;
    }

    public function render()
    {
        Admin::script($this->script());

        $options = [
            'all'   => 'All',
            'm'     => 'Male',
            'f'     => 'Female',
        ];

        return view('admin.tools.gender', compact('options'));
    }
}

```

视图`admin.tools.gender`文件为`resources/views/admin/tools/gender.blade.php`:
```html
<div class="btn-group" data-toggle="buttons">
    @foreach($options as $option => $label)
    <label class="btn btn-default btn-sm {{ \Request::get('gender', 'all') == $option ? 'active' : '' }}">
        <input type="radio" class="user-gender" value="{{ $option }}">{{$label}}
    </label>
    @endforeach
</div>
```

在`Grid`引入这个工具：
```php
$grid->tools(new UserGender());
```

在`model-grid`定义中接收到`gender`参数后，做好数据查询就可以了：
```php
if (in_array(Request::get('gender'), ['m', 'f'])) {
    $grid->model()->where('gender', Request::get('gender'));
}
```

可以参考上面的方式来添加自己的工具。


### tools用法

`Grid::tools` 方法允许传入 `string`，`array`, `AbstractTool` 和 `闭包`等类型参数，下面是演示。

```php
// 传入字符串
$grid->tools('<a class="btn btn-sm btn-default">工具按钮测试</a>');

// 传入数组
$grid->tools([
	'<a class="btn btn-sm btn-default">工具按钮测试</a>',
	new UserGender(),
]);

// 传入闭包
$grid->tools(function (Grid\Tools $tools) {
	$tools->append(new UserGender());
});
```


## 批量操作

目前默认实现了批量删除操作的功能，如果要关掉批量删除操作：
```php
$grid->tools(function ($tools) {
    $tools->batch(function ($batch) {
        $batch->disableDelete();
    });
});

// 也可以这样
$grid->disableBatchDelete();

// 或
$grid->batchActions(function (Grid\Tools\BatchActions $batch) {
	$batch->disableDelete();
});
```

如果要添加自定义的批量操作，可以参考下面的例子。

下面是扩展一个对文章批量发布的功能：

先定义操作类`app/Admin/Extensions/Tools/ReleasePost.php`：
```php
<?php

namespace App\Admin\Extensions\Tools;

use Dcat\Admin\Grid\BatchAction;

class ReleasePost extends BatchAction
{
    protected $action;

    public function __construct($title, $action = 1)
    {
        $this->title = $title;
        $this->action = $action;
    }
    
    public function script()
    {
        return <<<JS
        
$('{$this->elementClass()}').on('click', function() {
	// 获取选中的id数组
	var idArray = {$this->getSelectedKeysScript()}
	
    $.ajax({
        method: 'post',
        url: '{$this->resource()}/release',
        data: {
            _token:LA.token,
            ids: idArray.join(),
            action: {$this->action}
        },
        success: function () {
            LA.reload();
            LA.success('操作成功');
        }
    });
});
JS;

    }
}
```
看代码的实现，通过click操作发送一个post请求，把选中的行数据`id`通过数组的形式传给后端接口，后端接口拿到`id`列表和要修改的状态来更新数据，然后前端刷新页面(pjax reload)，并弹出`toastr`提示操作成功。

在`model-grid`中加入这个批量操作功能：
```php
$grid->batchActions([
	new ReleasePost('发布文章', 1),
	new ReleasePost('文章下线', 0)
]);  

// 也可以这么写
$grid->batchActions(function ($batch) {
    $batch->add(new ReleasePost('发布文章', 1));
    $batch->add(new ReleasePost('文章下线', 0));
});

// 或
$grid->tools(function ($tools) {
    $tools->batch(function ($batch) {
    	$batch->add(new ReleasePost('发布文章', 1));
    	$batch->add(new ReleasePost('文章下线', 0));
    });
});
```

这样批量操作下拉按钮下面就会添加两个操作项，最后一步就是添加一个接口来处理批量操作的请求了，接口代码如下：
```php

class PostController extends Controller
{
    ...
    
    public function release(Request $request)
    {
        foreach (Post::find($request->get('ids')) as $post) {
            $post->released = $request->get('action');
            $post->save();
        }
    }
    
    ...
}
```

然后添加路由指向上面的接口：
```php
$router->post('posts/release', 'PostController@release');
```

这样整个流程就完成了。