# 工具栏

## 工具按钮

在`model-grid`的头部默认有`批量删除`和`刷新`两个操作工具，如果有更多的操作需求，系统提供了自定义工具的功能,下面的示例添加一个性别分类选择的按钮组工具。


<a name="outline"></a>
### 设置工具栏按钮样式

工具栏按钮默认显示`outline`模式，效果如下


用法
```php
$grid->toolsWithOutline();

// 禁止
$grid->toolsWithOutline(false);
```

效果
<a href="{{public}}/assets/img/screenshots/outline.png" target="_blank">
    <img style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" src="{{public}}/assets/img/screenshots/outline.png">
</a>

禁用`outline`后的效果

<a href="{{public}}/assets/img/screenshots/n-outline.png" target="_blank">
    <img style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" src="{{public}}/assets/img/screenshots/n-outline.png">
</a>

如果你希望某个按钮不使用`outline`模式，可以在按钮的`class`属性中加上`disable-outline`
```php
$grid->tools('<a class="btn btn-primary disable-outline">测试按钮</a>');
```

### 自定义工具栏按钮

先定义工具类`app/Admin/Extensions/Tools/UserGender.php`继承工具类的基类`Dcat\Admin\Grid\Tools\AbstractTool`：

```php
<?php

namespace App\Admin\Grid\Tools;

use Dcat\Admin\Admin;
use Dcat\Admin\Grid\Tools\AbstractTool;

class UserGender extends AbstractTool
{
    protected function script()
    {
        $url = request()->fullUrlWithQuery(['gender' => '_gender_']);

        return <<<JS
$('input:radio.user-gender').change(function () {
    var url = "$url".replace('_gender_', $(this).val());

    Dcat.reload(url);
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
    <label class="btn btn-default {{ request()->get('gender', 'all') == $option ? 'active' : '' }}">
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
if (in_array($gender = request()->get('gender'), ['m', 'f'])) {
    $grid->model()->where('gender', $gender);
}
```

可以参考上面的方式来添加自己的工具。

### 进阶用法

如果你的工具按钮需要与后端API进行交互，则可以参考以下方式定义：

> {tip} `AbstractTool`类是属于`Dcat\Admin\Actions\Action`的子类，本质也是动作类的一种，更详细用法请参考[动作类基本使用](action.md)。


```php
<?php

namespace App\Admin\Grid\Tools;

use Dcat\Admin\Grid\Tools\AbstractTool;
use Illuminate\Http\Request;

class SendMessage extends AbstractTool
{
    /**
     * 按钮样式定义，默认 btn btn-white waves-effect
     * 
     * @var string 
     */
    protected $style = 'btn btn-white waves-effect';
    
    
    /**
     * 按钮文本
     * 
     * @return string|void
     */
    public function title()
    {
        return '发送提醒';
    }
    
    /**
     *  确认弹窗，如果不需要则返回空即可
     * 
     * @return array|string|void
     */
    public function confirm()
    {
        // 只显示标题
//        return '您确定要发送新的提醒消息吗？';
        
        // 显示标题和内容
        return ['您确定要发送新的提醒消息吗？', '确认信息内容，如没有可以留空'];
    }
    
    /**
     * 处理请求
     * 如果你的类中包含了此方法，则点击按钮后会自动向后端发起ajax请求，并且会通过此方法处理请求逻辑
     * 
     * @param Request $request
     */
    public function handle(Request $request)
    {
        // 你的代码逻辑
        
        return $this->response()->success('发送成功')->refresh();
    }
    
    /**
     * 设置请求参数
     * 
     * @return array|void
     */
    public function parameters()
    {
        return [
            
        ];
    }
}
```

使用

```php
use App\Admin\Grid\Tools\SendMessage;

$grid->tools(new SendMessage());
```


### 添加工具类

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

<a name="batch"></a>
## 批量操作

### 禁用批量删除

系统默认开启了批量删除操作的功能，如果要禁用批量删除操作：

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

### 自定义批量操作

下面通过扩展一个对文章批量发布的功能来演示自定义批量操作的功能：

先定义操作类`app/Admin/Extensions/Tools/ReleasePost.php`，继承`Dcat\Admin\Grid\BatchAction`：

> {tip} `BatchAction`类是属于`Dcat\Admin\Actions\Action`的子类，本质也是动作类的一种，更详细用法请参考[动作类基本使用](action.md)。

```php
<?php

namespace App\Admin\Extensions\Tools;

use Dcat\Admin\Grid\BatchAction;
use Illuminate\Http\Request;

class ReleasePost extends BatchAction
{
    protected $action;

    // 注意action的构造方法参数一定要给默认值
    public function __construct($title = null, $action = 1)
    {
        $this->title = $title;
        $this->action = $action;
    }
    
    // 确认弹窗信息
    public function confirm()
    {
        return '您确定要发布已选中的文章吗？';
    }
    
    // 处理请求
    public function handle(Request $request)
    {
        // 获取选中的文章ID数组
        $keys = $this->getKey();
        
        // 获取请求参数
        $action = $request->get('action');
        
        foreach (Post::find($keys) as $post) {
            $post->released = $action;
            $post->save();
        }
        
        $message = $action ? '文章发布成功' : '文章下线成功';
        
        return $this->response()->success($message)->refresh();
    }
    
    // 设置请求参数
    public function parameters()
    {
        return [
            'action' => $this->action,
        ];
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

#### 获取复选框选中的ID数组

通过`getSelectedKeysScript`方法可以获取到复选框选中的ID数组，用法如下

> {tip} `getSelectedKeysScript`方法返回的是`js`代码，只能在`js`代码中使用。

```php
<?php

namespace App\Admin\Extensions\Tools;

use Dcat\Admin\Grid\BatchAction;
use Illuminate\Http\Request;

class ReleasePost extends BatchAction
{
    protected $action;

    public function __construct($title, $action = 1)
    {
        $this->title = $title;
        $this->action = $action;
    }
    
    // 确认弹窗信息
    public function confirm()
    {
        return '您确定要发布已选中的文章吗？';
    }
    
    // 处理请求
    public function handle(Request $request)
    {
        ...
    }
    
    /**
     * 设置动作发起请求前的回调函数，返回false可以中断请求. 
     * 
     * @return string
     */
    public function actionScript(){
        $warning = __('No data selected!');

        return <<<JS
function (data, target, action) { 
    var key = {$this->getSelectedKeysScript()}

    if (key.length === 0) {
        Dcat.warning('{$warning}');
        return false;
    }

    // 设置主键为复选框选中的行ID数组
    action.options.key = key;
}
JS;
    }
}
```

### 表单弹窗

请参考文档[工具表单 - 弹窗](widgets-form.md#modal)
