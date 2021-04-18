# BETA版本更新日志

## v2.0.23-beta

发布时间 2021/4/18

升级方法，逐步执行以下命令，并清除浏览器缓存
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.23-beta"
php artisan admin:update
```

### 新增功能

**1.增加对`Laravel Octane`的支持**

[Laravel Octane](https://github.com/laravel/octane) 是一个基于 `Swoole/RoadRunner` 驱动的可以提升 `Laravel` 框架性能的项目，安装后可以大幅提升`Laravel`项目的性能，`Dcat Admin`也兼容了`Laravel Octane`环境，只需在配置文件`config/octane.php`中加入如下配置即可：

```php

    'listeners' => [
        ...,

        RequestReceived::class => [
            ...Octane::prepareApplicationForNextOperation(),
            ...Octane::prepareApplicationForNextRequest(),
            
            // 开启对 Dcat Admin 的支持
            Dcat\Admin\Octane\Listeners\FlushAdminState::class,
        ],
        
        ...
    ],    
```

> [Laravel Octane](https://github.com/laravel/octane)目前仍处于`beta`版本阶段，关于[Laravel Octane](https://github.com/laravel/octane)的安装与更多介绍请前往文档 https://github.com/laravel/octane 查看。



**2.增加表格简化分页（`simplePaginate`）功能**

启用 `simplePaginate` 功能后会使用`Laravel`的[simplePaginate](https://laravel.com/docs/8.x/pagination#simple-pagination)功能进行分页，当数据量较大时可以大幅提升页面的响应速度，但需要注意的是，使用此功能后将不会查询数据表的**总行数**。

```php
// 启用
$grid->simplePaginate();

// 禁用
$grid->simplePaginate(false);
```

### 功能改进

**1.重构翻译功能**

在过去版本中，当使用[异步表单](lazy.md)以及[异步表格](lazy.md)功能时无法自动对字段名称进行翻译，从当前版本开始可以通过`translation`属性指定翻译文件的路径进行自动翻译，用法如下

```php
<?php

namespace App\Admin\Forms;

use Dcat\Admin\Traits\LazyWidget;
use Dcat\Admin\Widgets\Form;
use Dcat\Admin\Contracts\LazyRenderable;

class MyForm extends Form implements LazyRenderable
{
    use LazyWidget;
    
    /**
     * 指定翻译文件名称
     * 
     * @var string 
     */
    protected $translation = 'my-form';
    
    public function form()
    {
        $this->text('field1');
        $this->text('field2');
        
        ...
    }
    
    ...
}
```

语言包`resources/lang/{locale}/my-form.php`内容如下
```php
<?php

return [
    'fields' => [
        'field1' => '字段1',
        'field2' => '字段2',
        
        ...
    ],
];
```

并且在控制器中也可以指定当前翻译文件的路径

```php
use Dcat\Admin\Http\Controllers\AdminController;

class UserController extends AdminController
{
    /**
     * 指定翻译文件名称
     * 
     * @var string 
     */
    protected $translation = 'user1';
    
    ...
}
```

当然也可以通过`Admin::translation()`方法指定

```php
use Dcat\Admin\Admin;

Admin::translation('user');
```


**2.增加表格相关配置参数**

增加数个表格相关的配置文件参数`admin.grid.*`:

```php
    'grid' => [

        // 表格行操作类
        'grid_action_class' => Dcat\Admin\Grid\Displayers\DropdownActions::class,

        // 表格批量操作类
        'batch_action_class' => Dcat\Admin\Grid\Tools\BatchActions::class,

        // 表格分页类
        'paginator_class' => Dcat\Admin\Grid\Tools\Paginator::class,

        // 表格行默认的几个操作类配置
        'actions' => [
            'view' => Dcat\Admin\Grid\Actions\Show::class,
            'edit' => Dcat\Admin\Grid\Actions\Edit::class,
            'quick_edit' => Dcat\Admin\Grid\Actions\QuickEdit::class,
            'delete' => Dcat\Admin\Grid\Actions\Delete::class,
            'batch_delete' => Dcat\Admin\Grid\Tools\BatchDelete::class,
        ],

        ...
    ],
```


**3.移除禁用表格分页后显示表格页脚信息**

从当前版本开始，如果禁用了表格的分页功能，将不会再显示表格的页脚信息。

**4.优化 `Filter::panel()` 布局间距**


**5.优化文件发布功能，发布语言包文件时将不再覆盖`menu.php`以及`global.php`文件**

从当前版本开始，当使用`admin:update`以及`admin:publish --force`命令文件时，将不再覆盖`menu.php`以及`global.php`文件。

**6.更新`Tinymce`版本至`5.6.2`**

[@wk1025](https://github.com/jqhph/dcat-admin/pull/1154)

### BUG修复

1. 修复数据表格中字段类型为`Object`时报错问题 [@xiaohuilam](https://github.com/jqhph/dcat-admin/pull/1154)
2. 修复文件上传失败仍然提示上传成功问题
3. 修复权限判断中间件匹配时没有使用传入的`$request`对象问题 [@asmodai1985](https://github.com/jqhph/dcat-admin/pull/1157)
4. 修复表单`rules`方法第二个参数设置`message`无效问题
5. 修复`Helper::array()`会把`0`转为空数组问题
6. 修复行选择器无法选中树形表格子级行问题
7. 修复`KeyValue::setValueLabel()`方法无效问题
8. 修复`hasmany`以及`table`表单删除选项时会重置表单问题
9. 修复表格行操作编辑按钮文本显示错误问题 [@GemaDynamic](https://github.com/jqhph/dcat-admin/pull/1174)
10. 修复树形表格子级行无法正常使用表单弹窗问题 [#813](https://github.com/jqhph/dcat-admin/issues/1174)

## v2.0.22-beta

发布时间 2021/4/1

升级方法，逐步执行以下命令，并清除浏览器缓存
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.22-beta"
php artisan admin:update
```

### BUG修复

1. 修复部分功能缺失`CSRF_TOKEN`报错问题
2. 修复菜单自动适应高度功能报错问题


## v2.0.21-beta

发布时间 2021/3/30

升级方法，逐步执行以下命令，并清除浏览器缓存
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.21-beta"
php artisan admin:update
```

### 新增功能

**1.增加表格列选择器`ColumnSelector`支持持久化存储功能**

在配置文件`config/admin.php`可以配置存储列选择器状态的方式，支持的存储方式如下

- `Dcat\Admin\Grid\ColumnSelector\SessionStore` 列选择器状态数据保存在`session`中，仅在登陆状态中有效
- `Dcat\Admin\Grid\ColumnSelector\CacheStore`  列选择器状态数据保存在[Laravel Cache](https://laravel.com/docs/8.x/cache#driver-prerequisites)缓存系统中，最长可保存`300`天，并可以通过`admin.grid.column_selector.store_params.driver`可以配置缓存驱动，默认为`file`

```php
    'grid' => [

        ...

        'column_selector' => [
            'store' => Dcat\Admin\Grid\ColumnSelector\SessionStore::class,
            'store_params' => [
                'driver' => 'file',
            ],
        ],
    ],
```

**2.增加表格条件判断(`if`)的终结方法(`end`)**

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

**3.增加表格行选择器默认选中(`check`)以及禁止更改选中状态(`disable`)功能**


通过`check`方法可以设置默认选中的行，此方法接受一个`array`类型或`匿名函数`参数

```php
// 设置默认选中第 1/3/5 行
$grid->rowSelector()->check([0, 2, 4]);

// 传递闭包
$grid->rowSelector()->check(function () {
    // 设置默认选中第 1/3/5 行
    return in_array($this->_index, [0, 2, 4]);
});

// 在闭包中使用当前行其他字段
$grid->rowSelector()->check(function () {
    // 设置默认选中 id > 10 的行
    return $this->id > 10;
});
```

通过`disable`方法可以设置禁止更改选中状态的行，此方法接受一个`array`类型或`匿名函数`参数

```php
// 禁止第 1/3/5 行更改选中状态
$grid->rowSelector()->disable([0, 2, 4]);

// 传递闭包
$grid->rowSelector()->disable(function () {
    // 禁止第 1/3/5 行更改选中状态
    return in_array($this->_index, [0, 2, 4]);
});

// 在闭包中使用当前行其他字段
$grid->rowSelector()->disable(function () {
    // 禁止 id > 10 的行更改选中状态
    return $this->id > 10;
});

// disable 可以和 check 方法一起使用
$grid->rowSelector()->check([2, 4])->disable([0, 2, 4]);
```

**4.增加`KeyValue`表单自定义标题翻译功能**

```php
$form->keyValue(...)->setKeyLabel('键名')->setValueLabel('键值');
```


**5.增加`Grid::scrollbarX`显示表格横向滚动条方法**

显示表格横向滚动条，默认不显示

```php
// 启用
$grid->scrollbarX();

// 禁用
$grid->scrollbarX(false);
```


**6.增加`admin:update`命令**
从当前版本开始，升级后可以直接运行 `admin:update`，相当于运行

```
php artisan admin:publish --assets --migrations --lang --force
php artisan migrate
``` 

### 功能改进

**1.表单显示编辑数据时兼容驼峰风格的关联关系名称**

在旧版本中，如果需要编辑模型关联关系字段，且模型的关联关系名称为驼峰风格时，需要把名称更改为下划线风格才能正常显示，这对开发者非常不友好。从当前版本开始，可以直接使用驼峰风格的关联关系名称，不需要做任何特殊处理
```php
return Form::make(User::with(['myProfile']), function (Form $form) {
    // 直接使用驼峰风格命名，不需做其他处理
    $form->text('myProfile.full_name');
    
    ...
});
```


**2.调整工具表单数据设置逻辑，可在`form`方法中获取`default`方法数据**

```php
use Dcat\Admin\Widgets\Form;

class Setting extends Form
{
    public function form()
    {
        // 获取 default 方法设置的数据
        $id = $this->data()->id;
        $name = $this->data()->name;
    
        $this->text('name');
        
        ...
    }
    
    public function default()
    {
        return [
            'id' => 1,
            'name' => 'abc',
            ...
        ];
    }
}
```

**3.`hasMany`以及`array`表单支持整体使用`rules`验证规则校验**

```php
$form->hasMany(...)->rules('size:2');
```

**4.调整`selectTable`默认占位符为`选择 ...`**

**5.优化表单row布局间距**

[#1092](https://github.com/jqhph/dcat-admin/issues/1092)


### BUG

1. 修复 `ModelTree` 删除节点时无法删除间隔一个层级以上的子节点问题
2. 修复导出功能在部分环境可能会出现异常问题
3. 修复表单联动(`load`)加载后初始值丢失的问题 [@xqbumu](https://github.com/jqhph/dcat-admin/pull/1103)
4. 修复文件上传失败后显示提示信息异常问题
5. 修复数据详情`Show`实例化函数如果传递模型，主键赋值错误的问题  [@jisuye](https://github.com/jqhph/dcat-admin/pull/1112)
6. 修复表格`setConstraints`方法对快速编辑无效问题 [#1119](https://github.com/jqhph/dcat-admin/issues/1119)
7. 修复在`iframe`页面中预览图片功能异常问题
8. 修复`hasMany`以及`array`表单下删除使用`required`验证规则的字段后导致无法提交表单问题
9. 修复表格`dialogTree`当顶级ID为字符串`0`时加载异常问题 [#1122](https://github.com/jqhph/dcat-admin/issues/1122)


## v2.0.20-beta

发布时间 2021/3/8

升级方法，逐步执行以下命令，并清除浏览器缓存
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.20-beta"
php artisan admin:publish --assets --migrations --force
php artisan migrate
```

### 功能改进

**1.`selectTable`表单表格刷新时保留前页的数据状态**

`selectTable`表单表格刷新或翻页后会保留前页选中或取消的数据状态

**2.`selectTable`表单设置显示字段名称功能优化**

在旧版本中，如果想要设置`selectTable`表单选中的字段或显示的字段，需要在`Renderable`对象中设置，非常的麻烦和不便，从当前版本开始，开发者可以直接通过下面的方法设置选中的字段或显示的字段

```php
$form->selectTabole('user_id')
	->from(UserTable::make())
	->pluck('full_name', 'id'); // 第一个参数为显示的字段，第二个参数为选中后将保存到表单的字段

// 也可以直接使用下面的方法
$form->selectTabole('user_id')
	->from(UserTable::make())
	->model(UserModel::class, 'id', 'full_name');
```


**3.`selectTable`、`multipleSelectTable`、`radio`、`checkbox`表单增加`load`方法**

从当前版本开始，`selectTable`、`multipleSelectTable`、`radio`、`checkbox`也可以使用`load`方法联动`select`和`multipleSelect`表单

```php
$form->radio('type')->options([...])->load('category', 'categories/options');

$form->select('categories');
```

接口`categories/options`返回格式如下

```json
[
    {
        "id": 9,
        "text": "xxx"
    },
    {
        "id": 21,
        "text": "xxx"
    },
    ...
]
```


**4.增加菜单水平布局自动适应页面以及菜单高度变化功能**

启用菜单水平布局功能后，当页面高度或菜单高度发生变动时，页面会自适应、自行调整内容间距

****


**5.增加表格`Grid::dropColumn()`方法用于删除设置的列**

```php
$nameColumn = $grid->column('name');

// 删除名称为 `name` 的列
$grid->dropColumn('name');
// 等同于
$grid->dropColumn($nameColumn);
```

**6.增加`admin_javascript`函数**

此函数可用于往`php`的配置`array`中添加`JS`代码，用法如下

```php
$form->text('number')->inputmask([
	'oncomplete' => admin_javascript('function () {
		// 这里是js代码
	    alert('inputmask complete');
	}'),
]);
```

**6.`Form`表单底部可默认勾选`查看`、`继续编辑`、`继续创建`等选项功能**

用法如下 [#1073](https://github.com/jqhph/dcat-admin/pull/1073)

```php
$form->footer(function (Footer $footer) {
    // 设置`查看`默认选中
    $footer->defaultViewChecked();

    // 设置`继续编辑`默认选中
    $footer->defaultEditingChecked();
    
    // 设置`继续创建`默认选中
    $footer->defaultCreatingChecked();
});

// 设置`查看`默认选中
$form->defaultViewChecked();

// 设置`继续编辑`默认选中
$form->defaultEditingChecked();

// 设置`继续创建`默认选中
$form->defaultCreatingChecked();
```


### BUG

1. 修复表格使用关联关系字段排序时必须先`with`问题
2. 修复同个页面无法同时渲染多个异步组件问题
3. 修复树形表格下删除子节点数据后跳转异常问题 [#1071](https://github.com/jqhph/dcat-admin/issues/1071)
4. 修复表格导出字段中存在空数组时导出异常问题
5. 修复表单多图上传使用`sortable`功能进行排序会导致页面的图片元素消失不见问题
6. 修复表单字段`disable`方法设置`false`无效问题
7. 修复`multipleSelect`表单使用`load`联动加载时无法把所有选中选项传入接口问题 [#1076](https://github.com/jqhph/dcat-admin/issues/1076)
8. 修复表格规格选择器存在多个0开头选项时选中功能异常问题


## v2.0.19-beta

发布时间 2021/2/21

升级方法，逐步执行以下命令，并清除浏览器缓存
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.19-beta"
php artisan admin:publish --assets --migrations --force
php artisan migrate
```

### BUG修复

1. 修复非超管角色下请求 `内置api` 时提示 `无权访问` 问题
2. 修复在弹窗中使用时间范围表单报错问题




## v2.0.18-beta

发布时间 2021/2/20

升级方法，逐步执行以下命令，并清除浏览器缓存
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.18-beta"
php artisan admin:publish --assets --migrations --force
php artisan migrate
```

### 功能改进


**1.增加菜单顶部横向布局 (Horizontal)**

设置配置参数 `admin.layout.horizontal_menu` 的值为 `true` 开启此功能，效果如下

![](https://cdn.learnku.com/uploads/images/202102/20/38389/SpmXMujJ3D.png!large)

**2.权限中间件以及跳过登陆判断时可以填写路由别名并且无需增加前缀**

配置文件以及权限设置路由别名时无需填写路由前缀

```php
    'permission' => [
        ...
    
        // 跳过权限判断
        'except' => [
            // 可以直接填写路由别名，并且无需写路由前缀
            'custom.users',
        ],

    ],
```

**3.数据表格行数据增加 `_index` 字段用于保存行序号**

数据表格行数据增加 `_index` 字段用于保存行序号，从 `0` 开始，用法如下

```php
// 在 display 回调中使用
$grid->column('序号')->display(function () {
    return $this->_index + 1;
});


// 在行操作 action 中使用
$grid->actions(function ($actions) {
    $index = $this->_index;
    
    ...
});
```


**4.重命名 markdown 组件静态资源别名，避免与自定义 blade 标签产生冲突**

**5.增加配置参数 `admin.menu.default_icon` 用于设置默认菜单图标**

`admin.menu.default_icon` 用于设置菜单默认图标，默认值为 `feather icon-circle`

**6.增加新的区块位置 `NAVBAR_BEFORE` 以及 `NAVBAR_AFTER`**

```php
use Dcat\Admin\Admin;

// 往顶部导航栏前输出内容
admin_inject_section(Admin::SECTION['NAVBAR_BEFORE'], view('...'));

// 往顶部导航栏后输出内容
admin_inject_section(Admin::SECTION['NAVBAR_AFTER'], view('...'));
```

**6.优化表格字段选择器代码**


### BUG修复

1. 修复扩展管理页面`new`标签显示异常问题 [#1044](https://github.com/jqhph/dcat-admin/issues/1044)
2. 修复文件上传成功后直接删除报错问题 [#1058](https://github.com/jqhph/dcat-admin/issues/1058)
3. 修复 `Form::number` 表单在使用 `min` 和 `max` 方法后输入值异常问题



## v2.0.17-beta

发布时间 2021/2/5

升级方法，逐步执行以下命令，并清除浏览器缓存
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.17-beta"
php artisan admin:publish --assets --migrations --force
php artisan migrate
```

### 功能改进

**1.优化表格排序功能**

支持`orderBy`直接使用关联关系表字段进行排序，注意这里仅支持`一对一`以及`一对多`关联关系

```php
$grid->model()->orderBy('profile.age');
```

**2.增加模型树以及树形表格自定义`parent_id`值功能**

模型树以及树形表格可以在`model`中自定义`parent_id`值，默认值为`0`
```php
class Category extends Model
{
	use ModelTree;

	// 设置默认 parent_id 为 A
	protected $defaultParentId = 'A';
}
```

**3.数据详情 `file` 支持展示多文件**

[#985](https://github.com/jqhph/dcat-admin/pull/985)

```php
$show->field('...')->files();
```

效果

![](https://cdn.learnku.com/uploads/images/202102/02/38389/B0a2qZEBUL.png!large)


**4.`Form::input`支持数组批量设置**

```php
$form->submitted(function ($form) {
    $form->input(['k1' => 'v1', 'k2' => 'v2' ...]);
});
```

**5.扩展管理支持`logo`以及`别名展示`**

详细用法参考文档 [扩展](extension-dev.md#logo)


**6.增加admin_route方法根据别名获取URL**

`app/Admin/routes.php`路由注册如下
```php
Route::group([
    'prefix'        => config('admin.route.prefix'),
    'namespace'     => config('admin.route.namespace'),
    'middleware'    => config('admin.route.middleware'),
], function (Router $router) {
	// 设置别名
	$router->resource('users', 'UserController', [
		'names' => ['index' => 'my-users'],
	]);

});
```

根据别名获取URL

```php
// 获取url
$url = admin_route('my-users');

// 判断路由
$isUsers = request()->routeIs(admin_route_name('users'));
```

**8.JsonResponse::location 允许不传参**

如果不传参会在`1`秒之后自动刷新当前页面

```php
return Admin::json()->success('操作成功')->location();
```

**9.页面布局Layout\Column支持等宽布局**

当列宽度设置为`0`时会使用等宽布局 [#1018](https://github.com/jqhph/dcat-admin/pull/1018)

```php
use Dcat\Admin\Layout\Row;
use Dcat\Admin\Layout\Content;

return Content::make()
	->body(function (Row $row) {
	    $row->column(0, view('...'));
	});
```

**10.页面布局Layout\Row支持no-gutters属性**

`.row`上带有`margin-left: -15px;margin-right: -15px;`属性，你可以在`.row`上上定义`.no-gutters`属性，从而消除这个属性，使页面不会额外宽出`30px`，即`<div class="row no-gutters"...`
```php
$content->row(function (Row $row) {
	// 启用 no-gutters
	$row->noGutters();

	$row->column(9, function (Column $column) {
		$column->row($this->card(['col-md-12', 20], '#4DB6AC'));
		
		$column->row(function (Row $row) {
			// 启用 no-gutters
			$row->noGutters();

			$row->column(4, $this->card(['col-md-4', 30], '#80CBC4'));
			$row->column(4, $this->card(['col-md-4', 30], '#4DB6AC'));
			$row->column(4, function (Column $column) {
				$column->row(function (Row $row) {
					// 启用 no-gutters
					$row->noGutters();

					$row->column(6, $this->card(['col-md-6', 30], '#26A69A'));
					$row->column(6, $this->card(['col-md-6', 30], '#26A69A'));
				});
			});
		});
	});
});
```

效果如下

![](https://cdn.learnku.com/uploads/images/202102/05/38389/4YlO8aOPCW.jpg!large)

**11.表格删除数据后保留URL的get参数**

在之前的版本中，删除数据后会丢失`URL`的`get`参数，导致跳转回表格第一页，这个版本对这个功能进行了优化，删除后依然会保留`URL`的`get`参数 [#961](https://github.com/jqhph/dcat-admin/issues/961)


**12.重构文件上传前端代码**

此功能为技术性优化，这个版本对文件上传前端代码进行了重构、拆分代码，使其更易阅读和维护

### BUG修复

1. 修复`MultipleSelect`表单样式异常问题 [#967](https://github.com/jqhph/dcat-admin/issues/967)
2. 修复加载`markdown`组件后使用`select2`表单异常问题 [#990](https://github.com/jqhph/dcat-admin/issues/990)
3. 修复文件上传在`Linux`服务器下保存繁体中文文件名丢失问题 [#993](https://github.com/jqhph/dcat-admin/issues/993)
4. 修复 `Widgets\Dropdown::click` 无法显示默认选项问题
5. 修复`form`表单中的`number`组件文本内容为空时点击加减按钮出现`NaN`问题  [#995](https://github.com/jqhph/dcat-admin/issues/995)
6. 修复图片预览失败提示无法使用翻译文件问题
7. 修复一对一关联关系`Range`表单使用验证规则判断异常问题
8. 修复多应用情况下路由别名冲突的问题



## v2.0.16-beta

发布时间 2021/1/11

升级方法，逐步执行以下命令
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.16-beta"
php artisan admin:publish --assets --migrations --force
php artisan migrate
```


### 破坏性变动

**1.表单字段的 `disableHorizontal` 调整为 `horizontal` **

更改表单字段布局方式为`horizontal`，此功能默认启用，用法如下

```php
// 禁用 horizontal 布局
$form->text('...')->horizontal(false);
```

### 功能改进

**1.增强表格字段排序(sortable)功能**

使表格字段支持关联关系表字段以及`json`字段的排序

> 注意，关联关系仅支持`hasOne`以及`belongsTo`两种类型的字段排序，并且不支持多层级嵌套！

```php
// 关联关系表字段排序
$grid->column('profile.age')->sortable();

// 指定需要排序的字段名称
$grid->column('my_age')->sortable('profile.age');

// json字段排序
$grid->column('options.price')->sortable('options->price');
// 关联关系表的 json 字段排序
$grid->column('profile.options.price')->sortable('profile.options->price');
```

支持`MySql`的```order by cast(`{field}` as {type})```用法

```php
$grid->column('profile.age')->sortable(null, 'SIGNED');

$grid->column('profile.options.price')->sortable('profile.options->price', 'SIGNED');
```


**2.增加 admin_exist 函数，用于代替 exit**

`admin_exist` 用于中断程序执行，并响应数据到浏览器进行显示，用于代替 `exit` 和 `die`，下面简单介绍下用法


用法1，返回 `Content` 布局对象，此用法可用于返回错误信息显示到前端
```php
use Dcat\Admin\Widgets\Alert;
use Dcat\Admin\Layout\Content;

// 中断程序，并显示自定义页面到前端
admin_exit(
    Content::make()
        ->title('标题')
        ->description('描述')
        ->body('页面内容1')
        ->body(Alert::make('服务器出错了~', 'Error')->danger())
);
```

效果如下

![](https://cdn.learnku.com/uploads/images/202101/11/38389/FLg6C7kwRq.png!large)

用法2，返回 `json` 格式数据，此用法经常用于表单提交数据的`api`请求拦截，或`Action`的`api`请求拦截

```php
use Dcat\Admin\Admin;

admin_exit(
    Admin::json()
        ->success('成功了')
        ->refresh()
        ->data([
            ...
        ])
);

// 当然也可以直接响应数组
admin_exit([
   ...
]);
```

用法3，直接相应`Response`对象或字符串

```php
admin_exit('Hello world');

admin_exit(response('Hello world', 500));
```


**3.增加 Show\Field::bool() 和 Show\Field::bold() 方法**

字段值为真显示 `✓`, 否则显示 `✗` [#940](https://github.com/jqhph/dcat-admin/pull/940)

```php
$show->field('...')->bool();
```

字段值加粗显示

```php
$show->field('...')->bold();

// 指定颜色
$show->field('...')->bold(admin_color()->primary());
```


**4.增加 Form\Footer::view()  方法**
 
通过 `Form\Footer::view()` 方法可以自定义数据表单的底部视图 [#957](https://github.com/jqhph/dcat-admin/pull/957)

```php
$form->footer(function ($footer) {
    $footer->view('...', [...]);
});
```


**5.增加表单默认显示指定 Tab 功能**

```php
// 默认显示标题为 标题2 的 Tab
$form->getTab()->active('标题2');
// 也可以指定偏移量
$form->getTab()->activeByIndex(1);

$form->tab('标题1', function ($form) {
    ...
});

$form->tab('标题2', function ($form) {
    ...
});
```

**6.增加表单 Form\Row::horizontal() 方法**

设置布局为 `horizontal`

```php
$form->row(function (Form\Row $form) {
	$form->horizontal();

	...
});
```

**7.表格 Modal 增加自定义图标功能**


```php
$grid->column('...')->modal(function ($modal) {
    // 自定义图标
    $modal->icon('feather icon-x');
    
    return ...;
});
```

**8.增加路由域名限制配置**

通过配置参数`admin.route.domain`可以限制路由的域名, 打开配置文件 `config/admin.php`

```php
    'route' => [
        'domain' => env('ADMIN_ROUTE_DOMAIN'),

        'prefix' => env('ADMIN_ROUTE_PREFIX', 'admin'),

        'namespace' => 'App\\Admin\\Controllers',

        'middleware' => ['web', 'admin'],
    ],
```


**9.增加 admin.session 中间件的启用或禁用配置**

从`2.0`的版本之后 `admin.session` 中间件不再默认启用，如果您的应用同时有前台和后台，则需要开启 `admin.session` 中间件，否则会造成前后台 `session` 冲突问题。

把配置参数 `admin.route.enable_session_middleware` 的值设置为 `true` 即可开启
```php
    'route' => [
        'domain' => env('ADMIN_ROUTE_DOMAIN'),

        'prefix' => env('ADMIN_ROUTE_PREFIX', 'admin'),

        'namespace' => 'App\\Admin\\Controllers',

        'middleware' => ['web', 'admin'],
        
        // 开启 admin.session 中间件
        'enable_session_middleware' => true,
    ],
```

### BUG修复

1. 修复数据表格`Grid::header`以及`Grid::footer`回调的第一个参数中`Model`被转化为`array`格式问题
2. 修复切换主题时文件上传按钮颜色无法跟着改变问题 [#938](https://github.com/jqhph/dcat-admin/issues/938)
3. 修复 `Widgets\Table` 构造方法第三个参数设置无效问题
4. 修复 `app/Admin/bootstrap.php` 中使用 `config(['admin.layout.color' => '...'])` 覆盖主题色可能无效问题
5. 修复数据表格过滤器重置关联关系表单字段无效问题 [#949](https://github.com/jqhph/dcat-admin/issues/949)
6. 修复表格过滤器 `group` 功能显示异常问题 [#929](https://github.com/jqhph/dcat-admin/issues/929)
7. 修复当页面存在多个 `selectTable` 表单时所有弹窗都只显示第一个设置的 `title` 问题 [#926](https://github.com/jqhph/dcat-admin/issues/926)


## v2.0.15-beta

发布时间 2021/1/3

升级方法，逐步执行以下命令
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.15-beta"
php artisan admin:publish --assets --migrations --force
php artisan migrate
```

### 功能改进

**1.升级select2到v4.1.x-beta版本**


`select`组件升级至`v4.1.x-beta`，使`tags`表单体验更好，并支持了多国语言翻译。

**2.Widgets/Modal增加弹窗垂直居中以及可滚动功能**

用法如下 [#901](https://github.com/jqhph/dcat-admin/pull/901)

```php
$modal = Modal::make()
    ->xl()
    ->centered() // 设置弹窗垂直居中
    ->scrollable() // 设置弹窗内容可滚动
    ->title(...)
    ->body(...);
```

**3.`Admin::requiredAssets`支持传递动态参数**

```php
use Dcat\Admin\Admin;

// 注册前端组件别名
// {lang} 为动态参数
Admin::asset()->alias('@test', [
    'js' => ['/vendor/test/js/{lang}.min.js'],
]);

// {lang} 会被替换为 zh_CN
Admin::requireAssets('@test', ['lang' => 'zh_CN']);
// 也可以这样使用
Admin::requireAssets('@test?lang=zh_CN');
```


### Bug修复

1. 修复表单`block`布局无法保存数据问题 [#883](https://github.com/jqhph/dcat-admin/issues/883)
2. 修复`hasMany`表单下使用`currency`失效问题 [#886](https://github.com/jqhph/dcat-admin/issues/886)
3. 修复数据表单保存后自动跳转到详情页问题 [#893](https://github.com/jqhph/dcat-admin/issues/893)
4. 修复`editor`表单无法清空数据问题 [#895](https://github.com/jqhph/dcat-admin/issues/895)
5. 修复`hasMany`表单下使用`tags`的`required`验证异常问题 [#905](https://github.com/jqhph/dcat-admin/issues/905)
6. 修复多文件上传表单删除单个文件时会导致全部文件被清空问题 [#914](https://github.com/jqhph/dcat-admin/issues/914)
7. 修复表格字段无法使用模型访问器问题

## v2.0.13-beta

发布时间 2020/12/23

升级方法，逐步执行以下命令
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.13-beta"
php artisan admin:publish --assets --migrations --force
php artisan migrate
```



## v2.0.14-beta

发布时间 2020/12/24

升级方法，逐步执行以下命令
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.14-beta"
php artisan admin:publish --assets --migrations --force
php artisan migrate
```

### 功能改进

**1.优化文件上传失败错误信息提示**

在旧版本中，文件上传失败的错误提示信息不太明确，导致难以定义错误原因，所以在这个版本中对错误提示进行了优化，一旦文件上传失败会显示具体原因。

### Bug修复

1. 修复表格字段与模型`casts`属性产生冲突，以及`display`闭包中使用字符串拼接显示异常问题 [#876](https://github.com/jqhph/dcat-admin/issues/876)
2. 修复表单动态显示功能无法使用问题 [#879](https://github.com/jqhph/dcat-admin/issues/879)
3. 修复表单使用`Block`布局时无法显示编辑数据问题 [#877](https://github.com/jqhph/dcat-admin/issues/877)

## v2.0.13-beta

发布时间 2020/12/23

升级方法，逐步执行以下命令
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.13-beta"
php artisan admin:publish --assets --migrations --force
php artisan migrate
```

### Bug修复

1. 修复表格展示关联关系字段当关联数据不存在时有可能报错问题 [#867](https://github.com/jqhph/dcat-admin/issues/867)
2. 修复当表格使用数据仓库返回数组或非模型`collection`时`display`方法无效问题 [#869](https://github.com/jqhph/dcat-admin/issues/869)


## v2.0.12-beta

发布时间 2020/12/22

升级方法，逐步执行以下命令
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.12-beta"
php artisan admin:publish --assets --migrations --force
php artisan migrate
```

### 破坏性变动

**1.图片/文件上传表单`removable`重命名为`removable`**

```php
$form->file('...')->removable();
```

### 功能改进

**1.支持PHP8.0**

**2.图片/文件上传表单支持监听WebUploader事件**

通过 `on` 方法可以监听 [WebUploader文件上传相关事件](http://fex.baidu.com/webuploader/doc/index.html#WebUploader_Uploader_events)

```php
$form->file('...')
	->on('startUpload', <<<JS
		function () {
			console.log('文件开始上传...', this);
			
			// 上传文件前附加自定义参数到文件上传接口
			this.uploader.options.formData['custom_field'] = '...';
		}
JS
    )
	->on('uploadFinished', <<<JS
    	function () {
    		console.log('文件上传完毕');
    	}
JS
    );
    
//       
```

**3.监听文件上传成功或文件被删除时产生的变动**

通过以下方法可以监听文件**上传成功**或文件**被删除**时产生的变动

```php
$file = $form->file('...');

Admin::script(
	<<<JS
$('{$file->getElementClassSelector()} .file-input').on('change', function () {
	console.log('文件发生变动', this.value);
});
JS
);
```

**4.允许在uploading事件中拦截并响应错误信息**

从这个版本开始支持在表单的 `uploading` 事件中拦截文件上传并支持响应错误信息到前端

```php
$form->uploading(function (Form $form) {
	return $form->response()->error('文件上传失败，请重试！');
});
```



**5.监听selectTable选中值变动**

```php
$selectTable = $form->selectTable('...')->from(...);

Admin::script(
	<<<JS
$('{$selectTable->getElementClassSelector()} input[type="hidden"]').on('change', function () {
	console.log('选中值发生变化', this.value);
});
JS
);
```

**6.调整树状表格无数据返回，取消返回404状态码**

**7.调整表格displayer类的row属性值类型为model**

**8.暗黑模式细节优化**

```php
$grid->column(...)->modal(function () {
    // $this 指向 model 对象
    dd($this);
});

$grid->actions(function () {
    // $this 指向 model 对象
    dd($this);
});
```


**9.优化卡片中图表显示溢出的问题**

[#822](https://github.com/jqhph/dcat-admin/pull/822)


**10.widget组件增加when方法**

```php
$modal = Dcat\Admin\Widgets\Modal::make()
	->when($condition, function ($modal) {
		// 当 $condition 的值为 真 时，会执行闭包里面的逻辑
	    $modal->xl();
	})
	->body(...)
	->render();
```



### Bug修复

1. 修复 `Grid\Filter::group` 无法保持选择状态问题 [#739](https://github.com/jqhph/dcat-admin/pull/793)
2. 修复 `Form::hasMany` 表单条目删除后仍然验证 `required` 问题 [#795](https://github.com/jqhph/dcat-admin/pull/795)
3. 修复 地图 表单无法使用问题
4. 修复当生成 `composer` 类映射文件且类文件被删除的情况下使用 `guessClassFileName` 会报错问题
5. 修复数据导出使用 `Fetched` 事件报错问题 [#815](https://github.com/jqhph/dcat-admin/issues/815)
6. 修复设置 `Grid name` 之后无法重置 `filter` 问题
7. 修复 `select2` 无法自动使用中文语言包问题 [#839](https://github.com/jqhph/dcat-admin/issues/839)
8. 修复表单勾选 `继续创建` 以及 `继续编辑` 跳转路由错误问题 [#814](https://github.com/jqhph/dcat-admin/issues/814)
9. 修复一对一关联关系 `range` 表单设置 `rules` 无效问题
10. 修复当启用 `fixColumns` 时，时间筛选下拉会被遮挡问题 [#833](https://github.com/jqhph/dcat-admin/issues/833)
11. 修复菜单使用`fa`图标无法自动对齐问题 [#758](https://github.com/jqhph/dcat-admin/pull/758)
12. 修复表单 `row` 布局下使用 `hasMany` 提交报错问题 [#801](https://github.com/jqhph/dcat-admin/issues/801)
13. 修复表单 `hasMany` 无法使用 `select` 联动问题 [#769](https://github.com/jqhph/dcat-admin/issues/769)



## v2.0.11-beta

发布时间 2020/12/06

升级方法，逐步执行以下命令
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.11-beta"
php artisan admin:publish --assets --force
php artisan admin:publish --migrations --force # 表结构有变动
php artisan migrate
```

### Bug修复

1. 修复使用 `pjax` 重复刷新页面可能导致 `Dcat.init` 监听失效问题
2. 修复 `admin:export-seed --users` 会生成多余代码问题
3. 修复表单编辑页面保存后跳转异常问题
4. 修复表单页面选择继续编辑会导致 `hasMany` 重复添加数据问题
5. 修复 `select` 表单联动后原 `select2 config` 丢失问题 [#779](https://github.com/jqhph/dcat-admin/issues/779)
6. 修复 `map` 表单加载异常问题 [#764](https://github.com/jqhph/dcat-admin/issues/764)
7. 修复批量删除功能删除数据后无法自动刷新页面问题
8. 修复 `hasMany` 表单编辑页面无法正常展示 `row` 以及 `column` 布局问题
9. 修复 `Dcat.init` 监听会被异步弹窗解绑问题
10. 修复表格工具栏下拉菜单会被固定列表格遮挡问题  [#728](https://github.com/jqhph/dcat-admin/issues/728)
11. 修复禁用 `showColumnSelector` 时仍然会读取缓存内容问题


### 功能改进

**1.Form::divider 增加 title 参数**

增加 `title` 参数用于在分割线中间显示标题功能，用法

```php
$form->divider('标题');
```


**2.Grid::footer以及Grid::header调整为支持多次回调**

```php
$grid->header(...);

$grid->header(...);

$grid->header(...);
```

**3.优化表格规格筛选器以及select表单样式**


## v2.0.10-beta

发布时间 2020/11/29

升级方法，逐步执行以下命令
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.10-beta"
php artisan admin:publish --assets --force
php artisan admin:publish --migrations --force # 表结构有变动
php artisan migrate
```

### 功能改进

**1.增加表单右上角提示窗展示字段验证错误信息**

此功能默认开启，可以通过`validationErrorToastr`方法禁用

```php
$form->validationErrorToastr(false);
```

**2.增加 Tree::maxDepth 方法用于限制模型树最大层级**

```php
$tree->maxDepth(5);
```

**3.优化导出功能，支持标题设置关联关系字段以及自动读取grid表格列的标题**

在当前版本中导出列默认与`column`列一致，不再需要手动设置导出的列名称以及翻译，并且支持关联关系字段

```php
$grid->column('id');
$grid->column('name');
...

// 默认与上面的列相同
$grid->export();
```

**4.工具表单增加resetButton与submitButton方法**

```php
// 禁用重置和提交按钮
$form->resetButton(false);
$form->submitButton(false);
```

**5.表单字段的`disable`以及`readOnly`方法增加参数控制是否启用**

```php
// 传 false 则不启用
$form->text(...)->disable(false);
```

**6.文件上传增加`withDeleteData`允许用户设置请求参数，并在上传接口以及删除接口中增加主键字段**

通过 `withDeleteData` 方法可以传递自定义参数到文件删除接口

```php
$form->file(...)->withDeleteData(['key' => 'value]);
```

**7.增加`embeds`表单禁止显示标题功能`**

第二个参数传 `false` 则不显示标题

```php
$form->embeds('field', false, function ($form) {
    ...
});
```

**8.重新编写部分单元测试用例，以支持2.x用法**

### Bug修复

1. 修复管理员详情页无法选中已有权限问题 
2. 修复 `admin:export-seed` 命令导出 `seeder` 类名异常问题
3. 修复表单删除跳转异常问题
4. 修复表单继续编辑跳转异常问题
5. 修复父表与 `hasMany` 存在同样字段名称时无法保存父表字段问题
6. 修复暗黑模式下选中子菜单样式异常问题 [#712](https://github.com/jqhph/dcat-admin/issues/712)
7. 修复表单 `block` 布局下表单动态显示功能无效问题 [#723](https://github.com/jqhph/dcat-admin/issues/723)
8. 优化 `selectOptions` 层级结构显示，解决前缀呈现随层级深度指数增加问题 [#618](https://github.com/jqhph/dcat-admin/issues/618)
9. 修复 `admin_view` 没有返回数据问题
10. 修复 `select` 表单 `ajax` 以及 `load` 设置的链接不能带参数问题 [#745](https://github.com/jqhph/dcat-admin/issues/745)
11. 修复表格行操作 `action` 的 `handle` 方法只能获取最后一行数据的 `id` 问题
12. 修复 `list` 表单编辑页无法删除已有数据问题 [#759](https://github.com/jqhph/dcat-admin/issues/759)
13. 修复 `embeds` 范围表单 `name` 属性错误问题

## v2.0.9-beta

发布时间 2020/11/18

升级方法，逐步执行以下命令
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.9-beta"
php artisan admin:publish --assets --force
php artisan admin:publish --migrations --force # 表结构有变动
php artisan migrate
```

**Bug修复**

1. 修复表格`filter::select` 表单远程加载 `/load/ajax` 等功能失效问题
2. 修复前端 `moment-timezone` 组件路径加载错误问题 [#701](https://github.com/jqhph/dcat-admin/issues/701)
3. 修复 `Form::tree`无法保存数据导致的无法设置权限问题
4. 修复当 `hasMany` 表单与父表有同样字段名称时填充值默认值异常问题
5. 修复表单`tab`布局嵌套`row`布局时新增页面报错问题 [#648](https://github.com/jqhph/dcat-admin/issues/648)
6. 修复当表单存在 `range` 类型字段时提交后无法获取 `range` 下面所有表单值问题
7. 修复 `Form::select` 使用表单联动功能时select2组件无效问题


## v2.0.8-beta

发布时间 2020/11/16

升级方法，逐步执行以下命令
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.8-beta"
php artisan admin:publish --assets --force
php artisan admin:publish --migrations --force # 表结构有变动
php artisan migrate
```

本次版本作为`2.0.7-beta`的补充版本，主要修复以下几个问题

1. 修复管理员页面无法查看权限问题
2. 修复表单block布局失效问题
3. 修复文件上传表单渲初始化功能异常问题
4. 修复部分表单字段不支持单个页面同时渲染多个的问题


## v2.0.7-beta

发布时间 2020/11/15

升级方法，逐步执行以下命令
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.7-beta"
php artisan admin:publish --assets --force
php artisan admin:publish --migrations --force # 表结构有变动
php artisan migrate
```

**功能改进**

1.引入[jquery.initialize](https://github.com/pie6k/jquery.initialize)组件，用于监听动态生成的页面元素并设置一个回调，下面来举一个简单的例子来演示用法：

在旧版本中，假如一个元素是`JS`动态生成的，如果我们需要对这个元素绑定一个点击事件的话，那么我们通常需要这么做

```html
<div class="selector">test</div>

<script>
Dcat.ready(function () {
    // 需要先 off 再 on 否则页面刷新后会造成重复绑定问题
    $(document).off('click', '.selector').on('click', '.selector', function () {
        ...
    })
});
</script>
```

上面这种做法一来比较麻烦，需要先`off`再`on`；二来无法对动态生成的元素做一些特殊处理，例如你想在`.selector`生成后改变背景颜色，这个操作就没办法做到。

在这个版本中我们可以使用`Dcat.init`方法来监听元素动态生成，可以很方便的解决上面两个问题

```html
<div class="selector">test</div>

<script>
Dcat.ready(function () {
    // $this 是当前元素的jquery dom对象
    // id 是当前元素的id属性，如果当前元素没有id则会自动生成一个随机id
    Dcat.init('.selector', function ($this, id) {
        // 修改元素的背景色
        $this.css({background: "#fff"});
        
        // 这里不需要 off 再重新 on，因为这个匿名函数只会执行一次
        $this.on('click', function () {
            ...
        });
    });
});
</script>
```

得益于这个[jquery.initialize](https://github.com/pie6k/jquery.initialize)组件的引入，在当前这个版本中我们对表单组件的前端代码也进行了优化，使其更容易支持`HasMany`这种动态生成的表单类型，大大降低了代码的复杂性。


2.Form::hasMany以及Form::array表单支持column和row布局

如果字段比较多，可以用`column`和`row`布局以节省空间

```php
$form->array('field', function (NestedForm $form) {
    $form->column(6, function (NestedForm $form) {
        $form->text('...');
        
        ...
    });
    
    $form->column(6, function (NestedForm $form) {
        ...
    });
});
```

3.配置过admin.auth.except参数的路由不需要验证权限 [#673](https://github.com/jqhph/dcat-admin/issues/673)


4.Form、Grid以及Show字段类增加when方法

用法示例，类似`Laravel QueryBuilder`的`when`方法

在表格中
```php
// 当第一个参数的值为 真 时才会执行闭包的代码
$grid->column('title')->when(true, function (Grid\Column $column, $value) {
    $column->label();
});
```

表单
```php
// 当第一个参数的值为 真 时才会执行闭包的代码
$form->text('email')->when(true, function (Form\Field\Text $text, $value) {
    $text->type('email');
});
```

5.管理员模型增加canSeeMenu方法控制是否可见菜单

```php
<?php

namespace App\Models;

use Dcat\Admin\Models\Administrator as Model;

class Administrator extends Model
{
    /**
     * 控制菜单是否可见，默认返回true
     * 
     * @param array|\Illuminate\Database\Eloquent\Model $menu 菜单节点
     * @return bool
     */
    public function canSeeMenu($menu)
    {
        return true;
    }
}
```

6.增加 admin_script、admin_style、admin_js、admin_css以及admin_require_assets函数

```php
// 相当于 Admin::script
admin_script('console.log(xxx)');

// 相当于 Admin::style
admin_style('.my-class {color: red}');

// 相当于 Admin::js() 
admin_js(['@admin/xxx.js']);

// 相当于 Admin::css() 
admin_css(['@admin/xxx.css']);

// 相当于 Admin::requireAssets() 
admin_require_assets(['@select2']);
```

7.简化动作(Action)的`JS`代码逻辑，去除记住`selector`功能

**BUG修复**

1. 修复表格 orderable 功能异常问题 [#674](https://github.com/jqhph/dcat-admin/issues/674)
2. 修复 JsonResponse methodIf 用法报错问题
3. 修复表格、表单以及数据详情指定 `label`  [#684](https://github.com/jqhph/dcat-admin/issues/684)
4. 修复表格 `Grid::rows` 回调无法正常使用问题
5. 修复widget添加`JS`代码异常导致部分类型的统计卡片异步加载功失效问题
6. 修复表格行操作 getKey 方法异常问题 [#691](https://github.com/jqhph/dcat-admin/issues/691)
7. 修复当页面存在多个 select 表单时无法使用联动功能问题
8. 修复表格删除数据后无法自动刷新页面问题


## v2.0.6-beta

发布时间 2020/11/7

升级方法，逐步执行以下命令
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.6-beta"
php artisan admin:publish --assets --force
php artisan admin:publish --migrations --force # 表结构有变动
php artisan migrate
```

**破坏性变动**

1.`Form::tags`表单默认保存为`array`类型
```php
// 需要自己转换保存到数据库的格式
$form->tags('tag')->saveAsJson();
```

2.默认禁用 session 中间件

3.`Form\Tree::disableFilterParents` 重命名为 `Form\Tree::exceptParentNode`
```php
$form->tree('cate')->exceptParentNode(false);
```

4.文件上传表单部分方法名称调整
```php
// 启用分块上传功能 disableChunked 更改为 chunked
$form->image('avatar')->chunked(true);

// 启用自动保存字段值功能 disableAutoSave 更改为 autoSave
$form->image('avatar')->autoSave(false);

// 启用删除文件功能 disableRemove 更改为 removable
$form->image('avatar')->removable(false);
```


**功能改进**

1.代码生成器增加字段拖动排序功能，此方法由小伙伴[@codingyu](https://github.com/codingyu)贡献

2.菜单表增加`show`和`extension`字段，`show`字段用于控制是否显示菜单；`extension`字段用于标记是否为扩展菜单

3.`Form::table`、`Form::array`、`Form::embeds`表单支持关联关系字段
```php
$form->table('profile.options', function ($form) {
    ...
});
```

4.增加 `Form::checkbox` 以及 `Form::radio` 表单选项竖排显示功能
```php
$form->checkbox('xxx')->inline(false)->options([...]);
```

5.配置文件跳过登录以及权限验证功能允许配置路由别名
```php
'auth' => [
    'except' => [
        ...
        'user.login',
    ],
],
```

6.`Form\Row` 增加 `getKey` 以及 `model` 方法

7.优化表格过滤器select表单选中效果，默认不选中

8.表单tab布局优化


**BUG修复**
1. 修复 `Form::checkbox` 选中/取消选中全部选项时动态显示表单功能无效问题
2. 修复台湾繁体无法翻译默认菜单标题问题 
3. 修复 `NestedForm` 中的 `number` 字段输入值为 0 时会被过滤问题 [#634](https://github.com/jqhph/dcat-admin/issues/634)
4. 修复模型树`RowAction`异步处理接口时获取主键报错问题
5. 修复表格过滤器无法重置关联表字段搜索值问题 [#650](https://github.com/jqhph/dcat-admin/issues/650)
6. 修复表格过滤器multipleSelect表单异常问题


## v2.0.5-beta 

发布时间 2020/10/29

BUG修复
1. 修复表格搜索多个关联表字段sql错误问题 [I232T7](https://gitee.com/jqhph/dcat-admin/issues/I232T7)
2. 修复`Form::datetimeRange`表单无法选择日志问题
3. 修复无法添加多个`Form::table`表单字段问题 [#627](https://github.com/jqhph/dcat-admin/issues/627)
4. 修复表格过滤器 MultipleSelectTable 中报错问题
5. 修复表格规格筛选器样式异常问题


## v2.0.4-beta 

发布时间 2020/10/27

BUG修复
1. 修复 admin_javascript_json 函数会自动空滤数组空值问题
2. 修复数据表单使用 tab 布局报错问题 [#620](https://github.com/jqhph/dcat-admin/issues/620)
3. 修复扩展本地安装生成临时目录权限异常问题 [#625](https://github.com/jqhph/dcat-admin/issues/625)
4. 修复表单使用 html 方法设置视图存在 script 标签时报错问题 [#624](https://github.com/jqhph/dcat-admin/issues/624)
5. 修复数据详情使用关联关系（一对多）显示报错问题 [#623](https://github.com/jqhph/dcat-admin/issues/623)
6. 修复 dropdown 下拉菜单计算显示位置异常问题 [#I22S2N](https://gitee.com/jqhph/dcat-admin/issues/I22S2N)


## v2.0.3-beta 

发布时间 2020/10/27

BUG修复
1. 修复表单行内编辑显示返回信息异常问题
2. 修复`admin.auth.remember`设置无效问题 [#613](https://github.com/jqhph/dcat-admin/issues/613)
3. 修复`editor`表单中文翻译异常问题 [#611](https://github.com/jqhph/dcat-admin/issues/611)
4. 修复`Filter::scope`选择筛选项不会过滤分页参数问题
5. 修复表单事件拦截相关问题
6. 修复树形表单使用异常问题 [#619](https://github.com/jqhph/dcat-admin/issues/619)

功能改进
1. 增加跳过权限和登录验证的配置方式
2. 扩展service provider增加middleware和路由排查注册功能
3. 批量操作change事件监听优化
4. 增加`RowSelector`健壮性, 以避免遇到`json`数组类型字段无法处理而报错 [#609](https://github.com/jqhph/dcat-admin/pull/609)

## v2.0.2-beta 

发布时间 2020/10/21

BUG修复
1. 修复代码生成器生成控制器文件命名空间异常问题 #600 
2. 修复配置文件logo路径错误问题
3. 修复表格关联字段搜索无效问题
4. 修复模型树行操作生成选择器重复问题
5. 修复访问无权限页面报错问题
6. 修复表格过滤器multipleSelect无法选中关联表字段值问题 #603 
7. 修复表单tab布局无效问题 #605 

功能改进
1. Auth\Permission移至Http目录
2. 数据表json字段改成text
3. 增加登录账号密码错误翻译
4. 增加 admin_javascript_json 函数，使大部分组件配置支持传递JS代码
5. Admin::color 增加暗黑模式颜色




## v2.0.1-beta 

发布时间 2020/10/20

BUG修复
- 修复数据表格过滤搜索BUG #599 
- 修复代码生成器生成控制器基类命名空间错误问题 #600 

功能改进

- 代码生成器增加页面标题以及面包屑翻译功能
- 异常处理优化
- 增加 admin_setting_array 函数
