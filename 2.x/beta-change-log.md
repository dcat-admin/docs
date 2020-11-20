# BETA版本更新日志


### v2.0.9-beta

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


### v2.0.8-beta

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


### v2.0.7-beta

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


### v2.0.6-beta

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


### v2.0.5-beta 

发布时间 2020/10/29

BUG修复
1. 修复表格搜索多个关联表字段sql错误问题 [I232T7](https://gitee.com/jqhph/dcat-admin/issues/I232T7)
2. 修复`Form::datetimeRange`表单无法选择日志问题
3. 修复无法添加多个`Form::table`表单字段问题 [#627](https://github.com/jqhph/dcat-admin/issues/627)
4. 修复表格过滤器 MultipleSelectTable 中报错问题
5. 修复表格规格筛选器样式异常问题


### v2.0.4-beta 

发布时间 2020/10/27

BUG修复
1. 修复 admin_javascript_json 函数会自动空滤数组空值问题
2. 修复数据表单使用 tab 布局报错问题 [#620](https://github.com/jqhph/dcat-admin/issues/620)
3. 修复扩展本地安装生成临时目录权限异常问题 [#625](https://github.com/jqhph/dcat-admin/issues/625)
4. 修复表单使用 html 方法设置视图存在 script 标签时报错问题 [#624](https://github.com/jqhph/dcat-admin/issues/624)
5. 修复数据详情使用关联关系（一对多）显示报错问题 [#623](https://github.com/jqhph/dcat-admin/issues/623)
6. 修复 dropdown 下拉菜单计算显示位置异常问题 [#I22S2N](https://gitee.com/jqhph/dcat-admin/issues/I22S2N)


### v2.0.3-beta 

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

### v2.0.2-beta 

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




### v2.0.1-beta 

发布时间 2020/10/20

BUG修复
- 修复数据表格过滤搜索BUG #599 
- 修复代码生成器生成控制器基类命名空间错误问题 #600 

功能改进

- 代码生成器增加页面标题以及面包屑翻译功能
- 异常处理优化
- 增加 admin_setting_array 函数