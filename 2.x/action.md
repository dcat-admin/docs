# 动作基本使用

开发者通过 `Action` 动作类可以非常方便的开发出一个含有特定功能的操作，可以非常方便的让用户与服务器产生交互。

例如，页面上需要一个按钮，用户点击之后可以向服务器发起请求，通过弹窗展示当前登录用户的信息，那么这个功能按钮就可以用 `Action` 来开发。

## 示例

下面我们就开始开发一个用于查看登录用户信息的按钮：


### 使用命令创建Action类
首先需要先创建 `Action` 类，运行命令

```bash
php artisan admin:action
```

运行成功之后会看到命令窗口出现如下信息，让开发者选择一个 `Action` 类的类型，这里我们输入 `0` 就行

> {tip} `default`类型的动作类，可以用在页面的任意位置。

```bash
 Which type of action would you like to make?:
  [0] default
  [1] grid-batch
  [2] grid-row
  [3] grid-tool
  [4] form-tool
  [5] show-tool
  [6] tree-tool
 > 0 # 输入 0
 
```

接着输入 `Action` 类名称，这里需要输入 `大驼峰` 风格的英文字母

```bash

 Please enter a name of action class:
 > ShowCurrentAdminUser

```

类名输入完成之后会出现以下信息让开发者输入类的命名空间，默认的命名空间是 `App\Admin\Actions`，这里我们直接按回车跳过就行了

```bash

 Please enter the namespace of action class [App\Admin\Actions]:
 >

```

这样一个 `Action` 类就创建完成了，刚刚创建的类路径是 `app/Admin/Actions/ShowCurrentAdminUser.php`


### 使用

修改 `Action` 类如下

> {tip} 如果你的动作类中需要通过构造方法传递参数，则一定要给构造方法的所有参数都设置一个默认值！

```php
<?php

namespace App\Admin\Actions;

use Dcat\Admin\Admin;
use Dcat\Admin\Widgets\Table;
use Dcat\Admin\Actions\Action;
use Dcat\Admin\Actions\Response;
use Dcat\Admin\Traits\HasPermissions;
use Illuminate\Contracts\Auth\Authenticatable;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Http\Request;

class ShowCurrentAdminUser extends Action
{
    /**
     * 按钮标题
     *
     * @var string
     */
    protected $title = '个人信息';

    /**
     * @var string
     */
    protected $modalId = 'show-current-user';

    /**
     * 处理当前动作的请求接口，如果不需要请直接删除
     *
     * @param Request $request
     *
     * @return Response
     */
    public function handle(Request $request)
    {
        // 获取当前登录用户模型
        $user = Admin::user();
        // 这里我们用表格展示模型数据
        $table = Table::make($user->toArray());

        return $this->response()
            ->success('查询成功')
            ->html($table);
    }

    /**
     * 处理响应的HTML字符串，附加到弹窗节点中
     *
     * @return string
     */
    protected function handleHtmlResponse()
    {
        return <<<'JS'
function (target, html, data) {
    var $modal = $(target.data('target')); 
    
    $modal.find('.modal-body').html(html)
    $modal.modal('show')
}        
JS;
    }

    /**
     * 设置HTML标签的属性
     *
     * @return void
     */
    protected function setupHtmlAttributes()
    {
        // 添加class
        $this->addHtmlClass('btn btn-primary');

        // 保存弹窗的ID
        $this->setHtmlAttribute('data-target', '#'.$this->modalId);

        parent::setupHtmlAttributes();
    }

    /**
     * 设置按钮的HTML，这里我们需要附加上弹窗的HTML
     *
     * @return string|void
     */
    public function html()
    {
        // 按钮的html
        $html = parent::html();

        return <<<HTML
{$html}        
<div class="modal fade" id="{$this->modalId}" tabindex="-1" role="dialog">
  <div class="modal-dialog modal-lg" role="document">
    <div class="modal-content">
      <div class="modal-header">
        <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span aria-hidden="true">&times;</span></button>
        <h4 class="modal-title">{$this->title()}</h4>
      </div>
      <div class="modal-body"></div>
    </div>
  </div>
</div>
HTML;
    }

    /**
     * 确认弹窗信息，如不需要可以删除此方法 
     *
     * @return string|void
     */
    public function confirm()
    {
        // return ['Confirm?', 'contents'];
    }

    /**
     * 动作权限判断，返回false则表示无权限，如果不需要可以删除此方法
     *
     * @param Model|Authenticatable|HasPermissions|null $user
     *
     * @return bool
     */
    protected function authorize($user): bool
    {
        return true;
    }

    /**
     * 通过这个方法可以设置动作发起请求时需要附带的参数，如果不需要可以删除此方法
     *
     * @return array
     */
    protected function parameters()
    {
        return [];
    }
}
```


修改完之后就可以开始使用了

```php
<?php

use App\Admin\Actions\ShowCurrentAdminUser;

class IndexController
{
	public function index(Content $content)
	{
	    return $content->body(ShowCurrentAdminUser::make());
	}
}
```

效果如下

<a href="{{public}}/assets/img/screenshots/action-default.png" target="_blank">
    <img class="img img-full" src="{{public}}/assets/img/screenshots/action-default.png" />
</a>

## 属性

`Dcat\Admin\Actions\Action` 类可用属性说明

| 属性名      | 类型     | 默认值     | 描述     |
| ---------- | -------- |---------- | -------- |
| title | `string`  |  | 标题 |
| selectorPrefix | `public` `string` | `.admin-action-` | 目标元素的`Css`选择器 |
| method | `string` | `POST`  | 与服务器交互的请求方法 |
| event | `string` | `click` | 目标元素绑定的事件，默认为点击事件 |
| disabled | `bool` | `false` | 是否渲染动作元素，设置`true`则不渲染 |
| usingHandler | `bool` | `true` | 当此属性设置为`false`，则无论`Action`中是否包含`handle`方法都不会向服务器发起请求 |


## 方法

`Dcat\Admin\Actions\Action` 类方法说明

### 创建实例 (make)

此方法是一个静态方法，用于实例化动作类

```php
$action = MyAction::make($param1, $param2...);
```

### 处理请求 (handle)

当`Action`类中包含此方法之时，目标元素会被绑定通过`event`属性设置的事件（默认为`click`）。如果事件被触发，则会向服务器发起请求，而`handle`方法则可以处理并响应此请求。
 
> {tip} 如果没有此方法，则目标元素不会被绑定事件。

### 响应 (response)

参考 [动作以及表单响应](response.md) 章节

### 设置请求数据 (parameters)

通过这个方法可以设置动作发起请求时需要附带的参数

```php
<?php

use Dcat\Admin\Actions\Action;
use Illuminate\Http\Request;

class MyAction extends Action
{
    public function handle(Request $request) 
    {
        // 接收参数
        $key1 = $request->get('key1');
        $key2 = $request->get('key2');
        
        return $this->response()->success('成功！');
    }
    
    public function parameters()
    {
        return [
        	'key1' => 'value1',
        	'key2' => 'value2',    
		];
    }
}

```

### 确认弹窗 (confirm)

设置确认信息，此方法要求返回一个`string`类型参数。

当此方法返回值不为空时会加入确认窗功能，当事件被触发时自动弹出确认框，点击确认后才会进行下一步操作。


```php
public function confirm()
{
	return '你确定要删除此行内容吗？';
}
```

显示弹窗标题和内容

```php
public function confirm()
{
	return ['你确定要删除此行内容吗？', '弹窗内容'];
}
```

### 发起请求之前执行的JS代码 (actionScript)

设置动作执行的前置`js`代码，当按钮绑定的事件被触发后，发起请求之前会执行通过此方法设置的`js`代码，此方法要求返回一个`js`的匿名函数。

`js`匿名函数接收以下三个参数：

- `data` `object` 需要传递给接口的数据对象
- `target` `object` 动作按钮的`jQuery`对象
- `action` `object` 动作的管理对象

```php
protected function actionScript()
{
	return <<<JS
function (data, target, action) {
    console.log('发起请求之前', data, target, action);
    
    // return false; 在这里return false可以终止执行后面的操作	
    
    // 更改传递到接口的主键值
    action.options.key = 123;
}
JS	
}
```

<a name="handleHtmlResponse"></a>
### 处理服务器响应的HTML代码 (handleHtmlResponse)

处理服务器响应的`HTML`代码，此方法要求返回一个`js`匿名函数。

```php
protected function handleHtmlResponse()
{
        return <<<'JS'
function (target, html, data) {
    // target 参数是动作按钮的JQ对象
    // html 参数是接口返回HTML字符串
    // data 参数是接口返回的完整数据的json对象

    target.html(html);
}
JS;
}
```

### 权限 (authorize)

此方法用于判断登录用户的操作权限，默认返回`true`

```php
protected function authorize($user): bool
{
	return $user->can('do-action');
}
```

### 无权限响应 (failedAuthorization)

此方法用于设置鉴权失败的响应内容，如果需要则可覆写此方法

```php
public function failedAuthorization()
{
	return $this->response()->error(__('admin.deny'));
}
```


### 隐藏或显示 (disable)

设置显示或隐藏此动作

```php
// 隐藏
MyAction::make()->disable();

// 显示
MyAction::make()->disable(false);
```


### 判断是否显示 (allowed)

判断动作是否允许显示
```php
if (MyAction::make()->allowed()) {
	...
}
```

### 设置主键 (setKey)

设置数据主键

```php
$id = ...;

MyAction::make()->setKey($id);
```

### 获取主键值 (getKey)

获取数据主键，此方法在`handle`方法内也同样可用

```php
<?php

use Dcat\Admin\Actions\Action;
use Illuminate\Http\Request;

class MyAction extends Action
{
    public function handle(Request $request) 
    {
        $id = $this->getKey();
        
        ...
        
        return $this->response()->success('成功！');
    }
    
    public function title()
    {
        return "标题 {$this->key()}";
    }
}
```


### 获取目标元素样式 (getElementClass)

获取动作目标元素（按钮）的`class`

```php
$class = MyAction::make()->getElementClass();
```

### 获取目标元素的Css选择器 (selector)

获取动作目标元素（按钮）的CSS选择器

```php
$selector = MyAction::make()->selector();

Admin::script(
	<<<JS
	$('$selector').click(...);
JS	
);
```

### 追加样式 (addHtmlClass)

追加获取动作目标元素（按钮）的`class`

```php
MyAction::make()->addHtmlClass('btn btn-primary');

MyAction::make()->addHtmlClass(['btn', 'btn-primary']);
```

### 设置目标元素的HTML (html)

此方法用于设置动作目标元素的`HTML`代码，如有需要可以覆写

```php
protected function html()
{
	return <<<HTML
<a {$this->formatHtmlAttributes()}>{$this->title()}</a>
HTML;
}
```

### 添加JS代码 (script)

此方法用于在`render`方法执行完毕之前添加`JS`代码

```php
protected function script()
{
	return <<<JS
console.log('...')
JS;
}
```

### 设置目标元素的HTML属性 (setHtmlAttribute)

设置目标元素的`HTML`标签属性

```php
MyAction::make()->setHtmlAttribute('name', $value);

// 批量设置
MyAction::make()->setHtmlAttribute(['name' => $value]);

```

