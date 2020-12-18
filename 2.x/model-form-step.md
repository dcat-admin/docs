# 分步表单

## 安装

前往[https://github.com/dcat-admin/form-step](https://github.com/dcat-admin/form-step)下载分步表单扩展，然后安装并启用。

> {tip} 扩展安装请参考文档[扩展基本使用](extension-f.md)章节。

## 简单示例

```php
protected function form()
{
    return Form::make(new Model(), function (Form $form) {
        $form->title('分步表单');
        $form->action('step');
        $form->disableListButton();
    
        $form->multipleSteps()
            ->remember() // 记住表单步骤，默认不开启
            ->width('950px')
            ->add('基本信息', function ($step) {
                $info = '<i class="fa fa-exclamation-circle"></i> 表单字段支持前端验证和后端验证混用，前端验证支持H5表单验证以及自定义验证。';
    
                $step->html(Alert::make($info)->info());
    
                $step->text('name', '姓名')->required()->maxLength(20);
                // h5 表单验证
                $step->text('age', '年龄')
                    ->required()
                    ->type('number')
                    ->attribute('max', 150)
                    ->help('前端验证');
    
                $step->radio('sex', '性别')->options(['未知', '男', '女'])->default(0);
    
                // 后端验证
                $step->text('birthplace', '籍贯')
                    ->rules('required')
                    ->help('演示后端字段验证');
    
                $step->url('homepage', '个人主页');
    
                $step->textarea('description', '简介');
    
            })
            ->add('兴趣爱好', function ($step) {
                $step->tags('hobbies', '爱好')
                    ->options(['唱', '跳', 'RAP', '踢足球'])
                    ->required();
    
                $step->text('books', '书籍');
                $step->text('music', '音乐');
    
                // 事件
                $step->shown(function () {
                    return <<<JS
    Dcat.info('兴趣爱好');
    console.log('兴趣爱好', args);
    JS;
                });
    
            })
            ->add('地址', function ($step) {
                $step->text('address', '街道地址');
                $step->text('post_code', '邮政编码');
                $step->tel('tel', ' 联系电话');
            })
            ->done(function () use ($form) {
                $resource = $form->getResource(0);
    
                $data = [
                    'title'       => '操作成功',
                    'description' => '恭喜您成为第10086位用户',
                    'createUrl'   => $resource,
                    'backUrl'     => $resource,
                ];
    
                return view('admin::form.done-step', $data);
            });
    });
}
```

效果

<a href="{{public}}/assets/img/screenshots/step-form-1.png" target="_blank">
    <img style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" src="{{public}}/assets/img/screenshots/step-form-1.png">
</a>

## 运行逻辑

分步表单的使用很简单，运行逻辑也与普通表单没有太大区别，下面简单说说分步表单的运行逻辑。

> {tip} 分步表单没有 `update` 的概念。

#### 参数验证
当用户点击 `下一步` 时，会向后端发起请求验证参数是否正确，如果参数不符合要求则显示错误信息，验证通过才会进入下一个步骤。

#### 表单提交
当进行到最后一步时，会对所有步骤的表单参数一起提交到后端并进行验证，如果参数不符合要求则显示错误信息，验证通过则保存表单数据，保存的方法与普通表单完全一致。

> {tip} 最后保存表单的方法是 `Form::store`。

#### 完成页面

表单保存完成之后会显示完成页面，此步骤无法忽略。


## 编辑表单

分步表单默认是没有编辑功能的，用户输入了长步骤的表单之后不需要再分步编辑，因此如果需要对分步表单进行编辑，可以参考以下方式

```php
protected function form()
{
    return Form::make(new MyRepository(), function (Form $form) {
        // 判断是否是编辑页面
        if ($form->isEditing()) {
            $form->text('age', '年龄')
                 ->required()
                 ->type('number')
                 ->attribute('max', 150)
                 ->help('前端验证')
        
            ...
            
            return;
        }
    
    
        $form->multipleSteps()
            ->remember()
            ->width('950px')
            ->add('基本信息', function (Form\StepForm $step) {
                $info = '<i class="fa fa-exclamation-circle"></i> 表单字段支持前端验证和后端验证混用，前端验证支持H5表单验证以及自定义验证。';
    
                $step->html(Alert::make($info)->info());
    
                $step->text('name', '姓名')->required()->maxLength(20);
                // h5 表单验证
                $step->text('age', '年龄')
                    ->required()
                    ->type('number')
                    ->attribute('max', 150)
                    ->help('前端验证');
    
                $step->radio('sex', '性别')->options(['未知', '男', '女'])->default(0);
    
                ...
    
            })
            ->add('兴趣爱好', function (Form\StepForm $step) {
                ...
            })
            ->done(function () use ($form) {
                ...
            });
    });
}
```

## 功能接口

### 设置容器最大宽度

默认 `1000px`。

> {tip} 此方法只针对大屏幕，手机端页面会自动调节大小。

```php
$form->multipleSteps()->width('900px');
```

### 记住表单数据

开启此功能之后，当用户点击 `下一步` 按钮并且参数验证通过后，会把表单数据保存在 `session` 中，直到整个分步表单保存完毕之后才会销毁。

> {tip} 此功能默认不开启。

```php
// 开启
$form->multipleSteps()->remember();

// 关闭
$form->multipleSteps()->remember(false);
```

### 设置容器间距

默认值 `30px 18px 30px`。

```php
$form->multipleSteps()->padding('30px 18px 30px');
```

### 监听页面离开事件

监听所有步骤表单页面离开事件，可添加多个。

```php
$form->multipleSteps()->leaving(<<<JS
     // 获取当前页面的步骤索引
     var index = args.index; 
                 
     Dcat.info("你将要离开第 " + (index + 1) + " 个页面");
     
     // args变量是一个js对象，包含当前事件对象、当前步骤选项、表单对象和表单值等字段。
     console.log("leaving", args);
     
     // 获取当前事件对象
     var evt = args.event;
     // 获取步骤表单标题tap对象
     var tab = args.tab;
     // 获取动向是前往上一步还是下一步页面："forward"、"backward"
     var direction = args.direction;
     // 获取当前步骤的表单JQ对象
     var $form = args.form;
     // 获取当前步骤页的表单值
     var formArray = args.formArray;
     
     // 获取指定步骤的表单JQ对象
     var $firstForm = args.getForm(0);
     // 获取指定步骤的表单值
     var firstFormArray = args.getFormArray(0);
     
     // 停止离开当前页面
     return false;
JS    
)->leaving(...);
```

### 监听页面显示事件

监听所有步骤表单页面显示事件，可添加多个。

```php
$form->multipleSteps()->shown(<<<JS
     // 获取当前页面的步骤索引
     var index = args.index; 
                 
     Dcat.info("当前显示的是第 " + (index + 1) + " 个页面");
     
     // args变量的值与“leaving”事件的值相同。
     console.log("shown", args);
JS    
)->shown(...);
```

### 增加步骤表单

步骤表单支持所有表单字段。

```php
use Dcat\Admin\Form;

$form->multipleSteps()->add('标题', function (Form\StepForm $step) {

    $step->text('username')->rules('required');
    
    ...
});
```

#### 监听步骤页面离开事件

监听当前步骤页面离开事件，支持监听多次。

```php
use Dcat\Admin\Form;

$form->multipleSteps()->add('基本信息', function (Form\StepForm $step) {
    ...
    
    $step->leaving(<<<JS
    
    Dcat.info("你将要离开 基本信息 页面");
    
    // args变量是一个js对象，包含当前事件对象、当前步骤选项、表单对象和表单值等字段。
    console.log("离开 基本信息", args);
    
    // 获取当前事件对象
    var evt = args.event;
    // 获取当前页面的步骤索引
    var index = args.index;
    // 获取步骤表单标题tap对象
    var tab = args.tab;
    // 获取动向是前往上一步还是下一步页面："forward"、"backward"
    var direction = args.direction;
    // 获取当前步骤的表单JQ对象
    var $form = args.form;
    // 获取当前步骤页的表单值
    var formArray = args.formArray;
    
    // 获取指定步骤的表单JQ对象
    var $firstForm = args.getForm(0);
    // 获取指定步骤的表单值
    var firstFormArray = args.getFormArray(0);
    
    // 停止离开当前页面
    return false;
JS    
    );
    
    // 监听多次
    $step->leaving(...);
});
```

#### 监听步骤页面显示事件

监听当前步骤页面显示事件，支持监听多次。

```php
use Dcat\Admin\Form;

$form->multipleSteps()->add('基本信息', function (Form\StepForm $step) {
    ...
    
    $step->shown(<<<JS
    
    Dcat.info("当前步骤是 基本信息");
    
    // args变量的值与“leaving”事件的值相同。
    console.log("显示 基本信息", args);
JS    
    );
    
    // 监听多次
    $step->shown(...);
});
```

### 设置完成页面

当所有步骤都完成之后会显示完成页面，系统提供一个默认的完成页面，开发者也可以通过以下方法自定义完成页面要显示的内容。

```php
use Dcat\Admin\Form;

$form->multipleSteps()->done(function (Form\DoneStep $done) {
    
    // 获取新增ID
    // 由 Repository::store 返回的值
    $newId = $done->getNewId();
    
    // 返回你要显示的内容，可以使一个视图也可以是一个字符串。
    return view(...);
});
```


