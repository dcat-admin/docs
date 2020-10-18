# 表单弹窗


## 数据表单弹窗

通过`Form::dialog`方法可以快速构建一个基于数据表单的表单弹窗，仅需增加数行代码。

> {tip} 表单弹窗的实现原理是：通过`create`和`edit`页面获取构建好的表单`HTML`字符，然后使用弹窗插件把这部分`HTML`字符渲染出来。
如果期间需要加载新的`js`脚本，则会等待脚本加载完毕再执行表单初始化`js`代码。

### 简单示例

#### 开启表单弹窗
```php
<?php

use App\Http\Controllers\Controller;
use Dcat\Admin\Form;
use Dcat\Admin\Layout\Content;

class ModalFormController extends Controller
{
    public function index(Content $content)
    {
        return $content
            ->header('Modal Form')
            ->body($this->build());
    }

    protected function build()
    {
        Form::dialog('新增角色')
            ->click('.create-form') // 绑定点击按钮
            ->url('auth/roles/create') // 表单页面链接，此参数会被按钮中的 “data-url” 属性替换。。
            ->width('700px') // 指定弹窗宽度，可填写百分比，默认 720px
            ->height('650px') // 指定弹窗高度，可填写百分比，默认 690px
            ->success('Dcat.reload()'); // 新增成功后刷新页面

        Form::dialog('编辑角色')
            ->click('.edit-form')
            ->success('Dcat.reload()'); // 编辑成功后刷新页面

        // 当需要在同个“class”的按钮中绑定不同的链接时，把链接放到按钮的“data-url”属性中即可
        $editPage = admin_base_path('auth/roles/1/edit');

        return "
<div style='padding:30px 0'>
    <span class='btn btn-success create-form'> 新增表单弹窗 </span> &nbsp;&nbsp;
    <span class='btn btn-blue edit-form' data-url='{$editPage}'> 编辑表单弹窗 </span>
</div>
";
    }

}
```

#### 表单构建以及保存数据
```php
<?php

use App\Admin\Repositories\Role;
use Dcat\Admin\Controllers\HasResourceActions;
use Dcat\Admin\Form;
use Dcat\Admin\Layout\Content;
use Dcat\Admin\Admin;

class RoleController
{
    use HasResourceActions;
    
    /**
         * Edit interface.
         *
         * @param mixed   $id
         * @param Content $content
         *
         * @return Content
         */
        public function edit($id, Content $content)
        {
            return $content
                ->header(trans('admin.roles'))
                ->description(trans('admin.edit'))
                ->body($this->form()->edit($id));
        }
    
        /**
         * Create interface.
         *
         * @param Content $content
         *
         * @return Content
         */
        public function create(Content $content)
        {
            return $content
                ->header(trans('admin.roles'))
                ->description(trans('admin.create'))
                ->body($this->form());
        }
    
    /**
     * Make a form builder.
     *
     * @return Form
     */
    protected function form()
    {
        return Form::make(new Role(), function (Form $form) {
            $form->display('id', 'ID');
            
            $form->text('slug', trans('admin.slug'))->required()->prepareForSave(function ($value) {
                return $value;
            });
            $form->text('name', trans('admin.name'))->required();
    
            $form->tree('permissions')
                ->nodes(function () {
                    $permissionModel = config('admin.database.permissions_model');
                    $permissionModel = new $permissionModel;
    
                    return $permissionModel->allNodes();
                })
                ->customFormat(function ($v) {
                    if (!$v) return [];
    
                    return array_column($v, 'id');
                });
    
            $form->display('created_at', trans('admin.created_at'));
            $form->display('updated_at', trans('admin.updated_at'));
        });
    }
}
```

#### 效果
<a href="{{public}}/assets/img/screenshots/form-modal.png" target="_blank">
    <img style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" src="{{public}}/assets/img/screenshots/form-modal.png">
</a>

### 功能接口

表单弹窗必须绑定一个可点击的页面元素，通过点击这个元素弹出弹窗。

#### 设置弹窗标题

```php
$modal = Form::dialog('标题');
```

#### 绑定点击按钮
通过`ModalForm::click`方法可以绑定点击按钮，绑定后当点击该按钮时会弹出弹窗

```php
Form::dialog('标题')
    ->click('#click-button');
```

#### 设置URL

如果是创建类型的表单，则可以通过以下方法设置获取表单模板的url
```php
Form::dialog('新增角色')
    ->click('.create-form')
    ->url('auth/roles/create');
```

如果是编辑类型的表单，则需要多个url，因为点击每个按钮弹出弹窗的表单内容是不同的，所以每个按钮的链接也不同。

这个时候通过`ModalForm::url`方法设置的一个链接已经无法满足需求了，因而需要在点击按钮的`data-url`属性上保存url：
```php
Form::dialog('编辑角色')
    ->click('.edit-form')
    ->success('Dcat.reload()'); // 编辑成功后刷新页面

    // 当需要在同个“class”的按钮中绑定不同的链接时，把链接放到按钮的“data-url”属性中即可
    $editPage1 = admin_base_path('auth/roles/1/edit');
    $editPage2 = admin_base_path('auth/roles/2/edit');
return "
<div style='padding:30px 0'>
    <span class='btn btn-blue edit-form' data-url='{$editPage1}'> 编辑表单弹窗1 </span>
    <span class='btn btn-blue edit-form' data-url='{$editPage2}'> 编辑表单弹窗2 </span>
</div>
";
```

#### 表单保存成功回调

通过`success`方法可以设置表单保存成功之后执行的`js`代码，在这段`js`代码作用域内有一个`response`变量，通过这个变量可以获取服务端返回的`json`数据。

```php
Form::dialog('编辑角色')
   ->click('.edit-form')
   ->success(
       <<<JS
// 打印服务端响应数据             
console.log(response);

// 提示成功信息
Dcat.success(response.message || '保存成功');

// 保存成功之后刷新页面
Dcat.reload();
JS        
   );
```

#### 表单保存失败回调

通过`error`方法可以设置表单保存失败之后执行的`js`代码，在这段`js`代码作用域内有一个`response`变量，通过这个变量可以获取服务端返回的`json`数据。

```php
Form::dialog('编辑角色')
   ->click('.edit-form')
   ->error(
       <<<JS
// 打印服务端响应数据       
console.log(response);
JS        
   );
```

#### 表单保存之后回调
通过`saved`方法可以设置表单保存之后执行的`js`代码，在这段`js`代码作用域内有两个`js`变量：
- `success` 值为真表示保存成功，否则失败
- `response` 服务端返回的`json`数据

```php
Form::dialog('编辑角色')
   ->click('.edit-form')
   ->saved(
       <<<JS
// 打印服务端响应数据       
console.log(response);

if (success) {
    console.log('保存成功');
} else {
    console.log('保存失败');
}
JS        
   );
```

#### 强制刷新

每次点击按钮都重新从服务端拉取新的模板数据

```php
Form::dialog('编辑角色')
    ->click('.edit-form')
    ->forceRefresh();
```

#### 设置弹窗宽高

```php
Form::dialog('编辑角色')
    ->click('.edit-form')
    ->dimensions('50%', '400px');
    
// 或
Form::dialog('编辑角色')
    ->click('.edit-form')
    ->width('50%')
    ->height('400px');
```


## 工具表单弹窗

数据表单的弹窗功能通常需要结合一个资源控制器去实现，相对会比较复杂一点，所以系统也内置了另外一种更简便的表单弹窗功能，使用方法请参考[工具表单-弹窗](widgets-form.md#modal)。


<a href="{{public}}/assets/img/screenshots/modal-widget-form.png" target="_blank">
    <img class="img img-full" src="{{public}}/assets/img/screenshots/modal-widget-form.png">
</a>


