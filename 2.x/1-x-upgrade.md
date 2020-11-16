# v1.x版本升级指南


### 前言

本章节内容只包含 `1.x` 版本中 `API` 改动的部分，不包含新增特性或对用户使用无影响的改动说明， `2.0` 的具体版本变化说明请参考 [2.0有哪些变化？](https://learnku.com/articles/50781?#reply164307)

**预计升级时间：60 分钟**


### 1.创建新分支，备份配置文件

创建一个新的分支，然后备份配置文件 `config/admin.php` 命名为 `config/admin.bak.php`，方便后续对比配置变动。

### 2.更新composer依赖

先卸载 `1.x` 版本
```bash
composer remove dcat/laravel-admin
```

再安装
```
composer require dcat/laravel-admin:"2.*"
```

安装成功后

1. 删除 `public/vendors` 目录
2. 重新发布资源 `php artisan admin:publish --force`
3. 根据上面备份后的配置文件，把修改过的参数写到新的配置文件 `config/admin.php` 中，这里需要注意的是`1.x`的默认主题色是`indigo`（已被废弃），在新版本中已经替换成`default`了
4. 调整语言包，新版本中语言包目录由 `zh-CN` 变成了 `zh_CN`，需要把自定义的翻译文件移动到新目录，并且 `菜单标题` 的翻译也独立出来到 `menus.php` 中了
5. 运行数据库迁移命令命令 `php artisan migrate` ，新版本中新增了两个表`admin_settings` 以及 `admin_extensions`

### 3.全局更改命名空间

1. 全局搜索命名空间 `Dcat\Admin\Controllers` 并替换为 `Dcat\Admin\Http\Controllers`
2. 全局搜索命名空间 `Dcat\Admin\Auth` 并替换为 `Dcat\Admin\Http\Auth`


### 4.表格部分变动

1.字段隐藏功能调整，旧版本 `responsive` 方法已废弃，在新版本中开启字段隐藏功能方法如下

```php
// 开启字段选择器功能
$grid->showColumnSelector();

// 设置默认隐藏字段
$grid->hideColumns(['field1', ...]);
```

2.表格 `collection`、`fetching` 等方法已被移除，在新版本中可以通过下面的事件代替

```php
use Dcat\Admin\Grid;
use Illuminate\Support\Collection;

// 使用 Grid\Events\Fetched 事件代替 collection
$grid->listen(Grid\Events\Fetched::class, function ($grid, Collection $rows) {
    $rows->transform(function ($row) {
        // 更改行数据
        $row['name'] = $row['first_name'].' '.$row['last_name'];
        
        return $row;
    });
});

// 使用 Grid\Events\Fetching 事件代替 fetching
$grid->listen(Grid\Events\Fetching::class, function ($grid) {
    
});
```

3.表格行相关闭包中允许使用模型

```php
$grid->column('avatar')->display(function ({
    // 可直接访问模型相关方法
    return $this->getAvatar();
});
```


4.设置路由前缀方法由 `resource` 调整为 `setResource` 
```php
$grid->setResource('auth/users');
```

5.树形表格 `tree` 方法即将被废弃，将会移动到扩展中心


### 5.表单部分变动

1.调整表单处理响应方法，旧版本中的`success`、`error`、`redirect` 以及 `location` 方法已被移除，
在 `2.0` 中我们让表单的响应方法和 `action` 的响应方法统一了起来，详细用法请参考文档 [动作和表单响应](response.md)，示例

```php
$form->saving(function (Form $form) {
    return $form
        ->response()
        ->success('保存成功')
        ->script('console.log("执行JS代码")')
        ->redirect('auth/users');
});
```

如果是在[工具表单](widgets-form.md)中，用法如下
```php
public function handle(array $input)
{
    ...

    return $this
        ->response()
        ->alert()
        ->success('成功')
        ->detail('详细内容');
}
```

2.调整表单 `block` 布局功能，并废弃 `setDefaultBlockWidth` 方法，详细用法请参考文档 [表单block布局](model-form-layout.md)，示例

```php
$form->block(8, function (Form\BlockForm $form) {
    $form->title('基本设置');
    $form->showFooter();
    $form->width(9, 2);

    $form->column(6, function (Form\BlockForm $form) {
        $form->display('id');
        $form->text('name');
        $form->email('email');
        $form->image('avatar');
        $form->password('password');
    });

    $form->column(6, function (Form\BlockForm $form) {
        $form->text('username');
        $form->email('mobile');
        $form->textarea('description');
    });
});
$form->block(4, function (Form\BlockForm $form) {
    $form->title('分块2');

    $form->text('nickname');
    $form->number('age');
    $form->radio('status')->options(['1' => '默认', 2 => '冻结'])->default(1);

    $form->next(function (Form\BlockForm $form) {
        $form->title('分块3');

        $form->date('birthday');
        $form->date('created_at');
    });
});
```


3.废弃表单直接提交，只保留 `ajax` 提交的方式，并重命名 `disableAjaxSubmit` 方法为 `ajax`

```php
$form->ajax(false);
```

4.废弃分步表单，新版本请使用[分步表单扩展](https://github.com/dcat-admin/form-step)代替

6.`map`以及`listbox`、`slider`也即将废弃，并移动扩展中心

7.表单字段扩展功能变动，具体请参考文档[表单字段扩展](model-form-field-management.md)章节



### 6.数据仓库部分变动

1.数据仓库的接口命名做了简化处理，新的 interface 如下

```php
interface Repository
{
    /**
     * 获取主键名称.
     *
     * @return string
     */
    public function getKeyName();

    /**
     * 获取创建时间字段.
     *
     * @return string
     */
    public function getCreatedAtColumn();

    /**
     * 获取更新时间字段.
     *
     * @return string
     */
    public function getUpdatedAtColumn();

    /**
     * 是否使用软删除.
     *
     * @return bool
     */
    public function isSoftDeletes();

    /**
     * 获取Grid表格数据.
     *
     * @param Grid\Model $model
     *
     * @return \Illuminate\Contracts\Pagination\LengthAwarePaginator|Collection|array
     */
    public function get(Grid\Model $model);

    /**
     * 获取编辑页面数据.
     *
     * @param Form $form
     *
     * @return array|\Illuminate\Contracts\Support\Arrayable
     */
    public function edit(Form $form);

    /**
     * 获取详情页面数据.
     *
     * @param Show $show
     *
     * @return array|\Illuminate\Contracts\Support\Arrayable
     */
    public function detail(Show $show);

    /**
     * 新增记录.
     *
     * @param Form $form
     *
     * @return mixed
     */
    public function store(Form $form);

    /**
     * 查询更新前的行数据.
     *
     * @param Form $form
     *
     * @return array|\Illuminate\Contracts\Support\Arrayable
     */
    public function updating(Form $form);

    /**
     * 更新数据.
     *
     * @param Form $form
     *
     * @return bool
     */
    public function update(Form $form);

    /**
     * 删除数据.
     *
     * @param Form  $form
     * @param array $deletingData
     *
     * @return mixed
     */
    public function delete(Form $form, array $deletingData);

    /**
     * 查询删除前的行数据.
     *
     * @param Form $form
     *
     * @return array|\Illuminate\Contracts\Support\Arrayable
     */
    public function deleting(Form $form);
}
```


2.`EloquentRepository::eloquent()` 重命名为 `EloquentRepository::model()`

### 7.Section变动

在新版本中 `AdminSection` 已被移除，请使用 `Dcat\Admin\Admin::SECTION` 常量代替

```php
use Dcat\Admin\Admin;

admin_inject_default_section(Admin::SECTION['HEAD'], function () {
    return ...;
});
```


### 8.扩展

扩展相关变动请参考文档[扩展](extension-f.md)

### 9.登录逻辑
1.登录模板，如果你在旧项目中自定义过登录模板，则需要调整登录模板中的`JS`代码
```js
Dcat.ready(function () {
    // ajax表单提交
    $('#login-form').form({
        validate: true,
    });
});
```

2.登录逻辑，如果重写过登录逻辑，则最后登录成功的响应方法需要使用 `sendLoginResponse`

### 10.其他变动

1.资源注册
```php
use Dcat\Admin\Admin;

// 注册资源路径别名
Admin::asset()->alias('test', 'assets/test');

Admin::asset()->alias('名称', [ 
    'js' => [
        // @test 会判定为别名
        '@test/test.js',
    ],
    'css' => [
        '@test/test.css',
    ],
]);


// 加载资源
Admin::asset()->require('@名称');
// 仅加载 js
Admin::js('@名称');
// 仅加载 css
Admin::css('@名称');
```




