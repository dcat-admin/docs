# BETA版本更新日志


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

**1.图片/文件上传表单`removeable`重命名为`removable`**

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

// 启用删除文件功能 disableRemove 更改为 removeable
$form->image('avatar')->removeable(false);
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