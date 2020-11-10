# BETA版本更新日志

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

**功能接口破坏性变动**

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