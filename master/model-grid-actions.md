# 行的使用和扩展

### 启用或禁用默认操作按钮

`model-grid`默认有四个行操作`编辑`、`快捷编辑`、`删除`和`详情`，可以通过下面的方式关闭它们：

```php
use Dcat\Admin\Grid;

 $grid->actions(function (Grid\Displayers\Actions $actions) {
    $actions->disableDelete();
    $actions->disableEdit();
    $actions->disableQuickEidt();
    $actions->disableView();
});

// 也可以通过以下方式启用或禁用按钮
$grid->disableDeleteButton();
$grid->disableEditButton();
$grid->disableQuickEditButton();
$grid->disableViewButton();
```


### 获取当前行数据

可以通过传入的`$actions`参数来获取当前行的数据：
```php
use Dcat\Admin\Grid;

$grid->actions(function (Grid\Displayers\Actions $actions) {
    // 当前行的数据数组
    $rowArray = $actions->row->toArray();
    
    // 当前行的某个字段的数据
    $email = $actions->row->email;
    
    // 获取当前行主键值
    $id = $actions->getKey();
});
```

### 添加自定义按钮

如果有自定义的操作按钮，可以通过下面的方式添加：

```php
use Dcat\Admin\Grid;

$grid->actions(function (Grid\Displayers\Actions $actions) {
    // append一个操作
    $actions->append('<a href=""><i class="fa fa-eye"></i></a>');

    // prepend一个操作
    $actions->prepend('<a href=""><i class="fa fa-paper-plane"></i></a>');
}
```


### 添加复杂操作按钮

如果有比较复杂的操作，可以参考下面的方式：


先定义行操作类继承`Dcat\Admin\Grid\RowAction`
```php
<?php

namespace App\Admin\Extensions;

use Dcat\Admin\Grid\RowAction;

class CheckRow extends RowAction
{
    /**
     * 返回字段标题
     * 
     * @return string
     */
    public function title()
    {
        return 'Check row';
    }
    
    /**
     * 添加JS
     * 
     * @return string
     */
    protected function script()
    {
        return <<<JS
$('.grid-check-row').on('click', function () {
    
    // Your code.
    console.log($(this).data('id'));
    
});
JS;
    }

    public function html()
    {
        // 获取当前行数据ID
        $id = $this->getKey();
        
        // 获取当前行数据的用户名
        $username = $this->row->username;
        
        $this->setHtmlAttribute(['data-id' => $id, 'email' => $username]);

        return parent::html();
    }
}
```
然后添加操作：
```php
$grid->actions(new CheckRow());
```

### 操作按钮需要与API交互

如果你的操作类需要与后台接口交互，则可以在你的操作类中加上`handle`方法，这样就可以很方便的在同一个类里面处理完所有逻辑

```php
<?php

namespace App\Admin\RowActions;

use Dcat\Admin\Grid\RowAction;
use Illuminate\Http\Request;
use Illuminate\Database\Eloquent\Model;

class Copy extends RowAction
{
    protected $model;

    public function __construct(string $model = null)
    {
        $this->model = $model;
    }

    /**
     * 标题
     *
     * @return string
     */
    public function title()
    {
        return 'Copy';
    }

    /**
     * 设置确认弹窗信息，如果返回空值，则不会弹出弹窗
     *
     * 允许返回字符串或数组类型
     *
     * @return array|string|void
     */
    public function confirm()
    {
        return [
            // 确认弹窗 title
            "您确定要复制这行数据吗？",
            // 确认弹窗 content
            $this->row->username,
        ];
    }

    /**
     * 处理请求
     *
     * @param Request $request
     *
     * @return \Dcat\Admin\Actions\Response
     */
    public function handle(Request $request)
    {
        // 获取当前行ID
        $id = $this->getKey();

        // 获取 parameters 方法传递的参数
        $username = $request->get('username');

        /* @var Model $model */
        $model = $request->get('model');

        // 复制数据
        $model::find($id)->replicate()->save();

        // 返回响应结果并刷新页面
        return $this->response()->success("复制成功: [{$username}]")->refresh();
    }

    /**
     * 设置要POST到接口的数据
     *
     * @return array
     */
    public function parameters()
    {
        return [
            // 发送当前行 username 字段数据到接口
            'username' => $this->row->username,
            // 把模型类名传递到接口
            'model' => $this->model,
        ];
    }
}

```

使用
```php
use App\Models\User;

$grid->actions([new Copy(User::class)]);
```

