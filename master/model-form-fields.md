# 表单字段的使用

在`model-form`中内置了大量的form组件来帮助你快速的构建form表单

<a name="public"></a>
## 公共方法

<a name="value"></a>
### 设置表单值 (value)
```php
$form->text('title')->value('text...');
```

<a name="default"></a>
### 设置默认值 (default)
```php
$form->text('title')->default('text...');
```

<a name="help"></a>
### 设置提示信息 (help)
```php
$form->text('title')->help('help...');
```

<a name="attr"></a>
### 设置属性 (attribute)
```php
$form->text('title')->attribute(['data-title' => 'title...']);

$form->text('title')->attribute('data-title', 'title...');
```

<a name="required"></a>
### 设置为必填 (required)
```php
$form->text('title')->required();

// 不显示"*"号
$form->text('title')->required(false);
```

<a name="disable"></a>
### 设置为不可编辑 (disable)
```php
$form->text('title')->disable();
```

<a name="placeholder"></a>
### 设置占位符 (placeholder)
```php
$form->text('title')->placeholder('请输入。。。');
```

<a name="setWidth"></a>
### 设置宽度 (width)
```php
$form->text('title')->width(8, 2);
```

<a name="rules"></a>
### 设置验证规则 (rules)
具体使用可参考[表单验证](model-form-validation.md)。


<a name="saving"></a>
### 处理待保存的数据 (saving)
`saving`方法可以更改待保存到数据库的数据。如下面的例子，原本表单提交的是一个以“,”隔开的字符串，我们可以把这个值转化成`json`格式保存到数据库
```php
$form->mutipleFile('files')->saving(function ($paths) {
    // 把“,”隔开的字符串转化为数组
    $paths = Helper::array($paths);
    
    // 最终转化为json保存到数据库
    return json_encode($paths);
});
```

<a name="customFormat"></a>
### 处理待渲染的数据 (customFormat)
`customFormat`方法可以改变从外部注入到表单的字段值。

如上面的例子，我们把原本是以“,”隔开的字符串改为`json`格式保存到数据库，那么从数据库中取出的值也是`json`，这个时候直接注入到表单后并不能被表单识别，所以需要通过`customFormat`方法把这个值重新转回以“,”隔开的字符串。
```php
$form->mutipleFile('files')->saving(function ($paths) {
    $paths = Helper::array($paths);
    
    return json_encode($paths);
})->customFormat(function ($paths) {
    if (!$paths) return '';
    
    // 转化为数组
    $paths = json_decode($paths, true);
    
    // 转为以“,”隔开的字符串
    return join(',', $paths);
});
```

### 在弹窗中隐藏

如果不想在弹窗中显示某个字段，可以使用`Form\Field::hideInDialog`方法

```php
$form->display('id');
$form->text('title');
$form->textare('contents')->hideInDialog();
```


<a name="tab"></a>
## 选项卡表单 (tab)

如果表单元素太多,会导致表单页面太长, 这种情况下可以使用`tab`方法来分隔表单:

```php

$form->tab('Basic info', function ($form) {
    
    $form->text('username');
    $form->email('email');
    
})->tab('Profile', function ($form) {
                       
   $form->image('avatar');
   $form->text('address');
   $form->mobile('phone');
   
})->tab('Jobs', function ($form) {
                         
     $form->hasMany('jobs', function () {
         $form->text('company');
         $form->date('start_date');
         $form->date('end_date');
     });

  })

```

<a name="text"></a>
## 文本 (text)

```php
$form->text($column, [$label]);

// 添加提交验证规则
$form->text($column, [$label])->rules('required|min:10');
```

<a name="hidden"></a>
## 隐藏字段 (hidden)

通过`hidden`方法可以给你的表单设置一个隐藏字段。

```php
$form->hidden('author_id')->value(Admin::user()->getKey());
```

通常可以结合`saving`事件使用
```php
$form->hidden('author_id');

$form->saving(function (Form $form) {
    $form->author_id = Admin::user()->getKey();
});
```


<a name="select"></a>
## 下拉选框单选 (select)
```php
$form->select($column[, $label])->options([1 => 'foo', 2 => 'bar', 'val' => 'Option name']);
```

或者从api中获取选项列表：
```php
$form->select($column[, $label])->options('/api/users');

// 使用ajax并显示所选项目

$form->select($column[, $label])->options(Model::class)->ajax('/api/users');

// 或指定名称和ID

$form->select($column[, $label])->options(Model::class, 'name', 'id')->ajax('/api/users');
```
其中api接口的格式必须为下面格式：
```php
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

如果选项过多，可通过ajax方式动态分页载入选项：

```php
$form->select('user_id')->options(function ($id) {
    $user = User::find($id);

    if ($user) {
        return [$user->id => $user->name];
    }
})->ajax('/admin/api/users');
```

<sub>注：如果你修改了`config/admin.php`配置文件中`route.prefix`的值，此处的接口路由应该修改为`config('admin.route.prefix').'/api/users'`。</sub>

API `/admin/api/users`接口的代码：

```php
public function users(Request $request)
{
    $q = $request->get('q');

    return User::where('name', 'like', "%$q%")->paginate(null, ['id', 'name as text']);
}

```
接口返回的数据结构为
```json
{
    "total": 4,
    "per_page": 15,
    "current_page": 1,
    "last_page": 1,
    "next_page_url": null,
    "prev_page_url": null,
    "from": 1,
    "to": 3,
    "data": [
        {
            "id": 9,
            "text": "xxx"
        },
        {
            "id": 21,
            "text": "xxx"
        },
        {
            "id": 42,
            "text": "xxx"
        },
        {
            "id": 48,
            "text": "xxx"
        }
    ]
}
```

<a name="selectload"></a>
## 下拉选框联动

`select`组件支持父子关系的单向联动：
```php
$form->select('province')->options(...)->load('city', '/api/city');

$form->select('city');

```

其中`load('city', '/api/city');`的意思是，在当前select的选项切换之后，会把当前选项的值通过参数`q`, 调用接口`/api/city`，并把api返回的数据填充为city选择框的选项，其中api`/api/city`返回的数据格式必须符合:

```php
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
控制器action的代码示例如下：

```php
public function city(Request $request)
{
    $provinceId = $request->get('q');

    return ChinaArea::city()->where('parent_id', $provinceId)->get(['id', DB::raw('name as text')]);
}
```

<a name="multipleSelect"></a>
## 下拉选框多选 (multipleSelect)

> {tip} 注入这个字段的数据（从数据库查出来的）可以是一个以“,”隔开的字符串或数组。

```php
$form->multipleSelect($column[, $label])->options([1 => 'foo', 2 => 'bar', 'val' => 'Option name']);

// 使用ajax并显示所选项目：

$form->multipleSelect($column[, $label])->options(Model::class)->ajax('ajax_url');

// 或指定名称和ID

$form->multipleSelect($column[, $label])->options(Model::class, 'name', 'id')->ajax('ajax_url');
```

多选框可以处理两种情况，第一种是`ManyToMany`的关系。

```

class Post extends Models
{
    public function tags()
    {
        return $this->belongsToMany(Tag::class);
    }
}

$form->multipleSelect('tags')->options(Tag::all()->pluck('name', 'id'));

```

第二种是将选项数组存储到单字段中，如果字段是字符串类型，那就需要在模型里面为该字段定义[访问器和修改器](https://laravel.com/docs/5.5/eloquent-mutators)来存储和读取了。

如果选项过多，可通过ajax方式动态分页载入选项：

```php
$form->select('friends')->options(function ($ids) {

    return User::find($ids)->pluck('name', 'id');
    
})->ajax('/admin/api/users');
```

<sub>注：如果你修改了`config/admin.php`配置文件中`route.prefix`的值，此处的接口路由应该修改为`config('admin.route.prefix').'/api/users'`。</sub>

API `/admin/api/users`接口的代码：

```php
public function users(Request $request)
{
    $q = $request->get('q');

    return User::where('name', 'like', "%$q%")->paginate(null, ['id', 'name as text']);
}

```
接口返回的数据结构为
```
{
    "total": 4,
    "per_page": 15,
    "current_page": 1,
    "last_page": 1,
    "next_page_url": null,
    "prev_page_url": null,
    "from": 1,
    "to": 3,
    "data": [
        {
            "id": 9,
            "text": "xxx"
        },
        {
            "id": 21,
            "text": "xxx"
        },
        {
            "id": 42,
            "text": "xxx"
        },
        {
            "id": 48,
            "text": "xxx"
        }
    ]
}
```

<a name="selectResource"></a>
## 弹窗选择器 (selectResource)

选择数据源，选择弹窗里面的表格数据。
> {tip} 注入这个字段的数据（从数据库查出来的）可以是一个以“,”隔开的字符串或数组。

```php
 $form->selectResource('user')
    ->path('auth/users') // 设置表格页面链接
    ->options(function ($v) { // 显示已选中的数据
        if (!$v) return $v;
        $userModel = config('admin.database.users_model');

        return $userModel::findOrFail($v)->pluck('name', 'id');
    });
    
// 设置为多选
 $form->selectResource('user')
    ->path('auth/users') 
    ->multiple() // 设置为多选
    ->options(function ($v) {
        ...
    });  
    
// 限制最大选择数量
 $form->selectResource('user')
    ->path('auth/users') 
    ->multiple(3) // 最多选择3个选项
    ->options(function ($v) {
        ...
    });       
```

然后设置你的路由`app/Admin/routes.php`

> {tip} 这里的添加路由只是示例，如果是新增的控制器需要加路由，如果路由已经存在则不需要再添加。

```php
$router->resource('auth/users', 'UserController');
```

`auth/users`页面实现代码如下：
```php
<?php

use Dcat\Admin\Models\Administrator;
use Dcat\Admin\IFrameGrid;
use Dcat\Admin\Grid;
use Dcat\Admin\Controllers\AdminController;

class UserController extends AdminController
{
    protected function iFrameGrid()
    {
        $grid = new IFrameGrid(new Administrator());
        
        // 指定行选择器选中时显示的值的字段名称
        // 指定行选择器选中时显示的值的字段名称
        // 指定行选择器选中时显示的值的字段名称
        // 如果表格数据中带有 “name”、“title”或“username”字段，则可以不用设置
        $grid->rowSelector()->titleColumn('username');

        $grid->id->sortable();
        $grid->username;
        $grid->name;
    
        $grid->filter(function (Grid\Filter $filter) {
            $filter->equal('id');
            $filter->like('username');
            $filter->like('name');
        });
    
        return $grid;
    }
}
```

<a name="listbox"></a>
## 多选盒 (listbox)

使用方法和`multipleSelect`类似。

> {tip} 注入这个字段的数据（从数据库查出来的）可以是一个以“,”隔开的字符串或数组。

```php
$form->listbox($column[, $label])->options([1 => 'foo', 2 => 'bar', 'val' => 'Option name']);
```

<a name="textarea"></a>
## 长文本 (textarea)
```php
$form->textarea($column[, $label])->rows(10);
```

<a name="radio"></a>
## 单选 (radio)
```php
$form->radio($column[, $label])->options(['m' => 'Female', 'f'=> 'Male'])->default('m');
```

<a name="checkbox"></a>
## 多选 (checkbox)

`checkbox`能处理两种数据存储情况，参考[多选框](#多选框)

`options()`方法用来设置选择项:
```php
$form->checkbox($column[, $label])->options([1 => 'foo', 2 => 'bar', 'val' => 'Option name']);
```

<a name="email"></a>
## 邮箱 (email)
```php
$form->email($column[, $label]);
```

<a name="password"></a>
## 密码 (password)
```php
$form->password($column[, $label]);
```

<a name="url"></a>
## 链接 (url)
```php
$form->url($column[, $label]);
```

<a name="ip"></a>
## IP地址 (ip)
```php
$form->ip($column[, $label]);
```

<a name="mobile"></a>
## 手机 (mobile)
```php
$form->mobile($column[, $label])->options(['mask' => '999 9999 9999']);
```

<a name="time"></a>
## 时间 (time)
```php
$form->time($column[, $label]);

// 设置时间格式，更多格式参考http://momentjs.com/docs/#/displaying/format/
$form->time($column[, $label])->format('HH:mm:ss');
```

<a name="date"></a>
## 日期 (date)
```php
$form->date($column[, $label]);

// 设置日期格式，更多格式参考http://momentjs.com/docs/#/displaying/format/
$form->date($column[, $label])->format('YYYY-MM-DD');
```

<a name="datetime"></a>
## 时间日期 (datetime)
```php
$form->datetime($column[, $label]);

// 设置日期格式，更多格式参考http://momentjs.com/docs/#/displaying/format/
$form->datetime($column[, $label])->format('YYYY-MM-DD HH:mm:ss');
```

<a name="timeRange"></a>
## 时间范围 (timeRange)
`$startTime`、`$endTime`为开始和结束时间字段:
```php
$form->timeRange($startTime, $endTime, 'Time Range');
```

<a name="dateRange"></a>
## 日期范围 (dateRange)
`$startDate`、`$endDate`为开始和结束日期字段:
```php
$form->dateRange($startDate, $endDate, 'Date Range');
```

<a name="datetimeRange"></a>
## 时间日期范围 (datetimeRange)
`$startDateTime`、`$endDateTime`为开始和结束时间日期:
```php
$form->datetimeRange($startDateTime, $endDateTime, 'DateTime Range');
```

<a name="currency"></a>
## 单位符号 (currency)
```php
$form->currency($column[, $label]);

// 设置单位符号
$form->currency($column[, $label])->symbol('￥');
```

<a name="number"></a>
## 数字 (number)
```php
$form->number($column[, $label]);
```

<a name="rate"></a>
## 费率 (rate)
```php
$form->rate($column[, $label]);
```

<a name="file"></a>
## 文件上传 (file)

使用文件上传功能之前需要先完成上传配置。文件上传配置以及内置方法请参考:[图片/文件上传](model-form-upload.md).

文件上传目录在文件`config/admin.php`中的`upload.file`中配置，如果目录不存在，需要创建该目录并开放写权限。
```php
$form->file($column[, $label]);

// 修改文件上传路径和文件名
$form->file($column[, $label])->move($dir, $name);

// 并设置上传文件类型
$form->file($column[, $label])->rules('mimes:doc,docx,xlsx');

// 添加文件删除按钮
$form->file($column[, $label])->removable();
```

<a name="image"></a>
## 图片上传 (image)

使用图片上传功能之前需要先完成上传配置。图片上传配置以及内置方法请参考:[图片/文件上传](model-form-upload.md).

图片上传目录在文件`config/admin.php`中的`upload.image`中配置，如果目录不存在，需要创建该目录并开放写权限。

可以使用压缩、裁切、添加水印等各种方法,需要先安装[intervention/image](http://image.intervention.io/getting_started/installation)。

更多使用方法请参考[[Intervention](http://image.intervention.io/getting_started/introduction)]：
```php
$form->image($column[, $label]);

// 修改图片上传路径和文件名
$form->image($column[, $label])->move($dir, $name);

// 剪裁图片
$form->image($column[, $label])->crop(int $width, int $height, [int $x, int $y]);

// 加水印
$form->image($column[, $label])->insert($watermark, 'center');
```

<a name="multipleImage"></a>
## 多图和多文件上传 (multipleImage/multipleFile)

多图片上传表单继承自单图上传表单，多文件上传表单继承自单文件上传表单。

> {tip} 注入这个字段的数据（从数据库查出来的）可以是一个以“,”隔开的字符串或数组。

```php
// 多图
$form->multipleImage($column[, $label]);

// 限制最大上传数量
$form->multipleImage($column[, $label])->limit(3);

// 多文件
$form->multipleFile($column[, $label]);

// 限制最大上传数量
$form->multipleFile($column[, $label])->limit(5);
```

多图/文件上传的时候提交的数据为以“,”隔开的文件路径字符串。当然你可以通过以下方式把数据在保存进数据库之前改为你想要的格式：
```php
// 转化为json格式保存到数据库
$form->multipleFile($column[, $label])->saving(function ($paths) {
    $paths = Helper::array($paths);
    
    return json_encode($paths);
});

```

<a name="map"></a>
## 地图 (map)

地图组件引用了网络资源，默认关闭,如果要开启这个组件参考[form组件管理](model-form-field-management.md)

地图控件，用来选择经纬度,`$latitude`, `$longitude`为经纬度字段，`Laravel`的`locale`设置为`zh_CN`的时候使用腾讯地图，否则使用Google地图：
```php
$form->map($latitude, $longitude, $label);
```

<a name="slider"></a>
## 滑动条 (slider)
可以用来数字类型字段的选择，比如年龄：
```php
$form->slider($column[, $label])->options(['max' => 100, 'min' => 1, 'step' => 1, 'postfix' => 'years old']);
```
更多options请参考:https://github.com/IonDen/ion.rangeSlider#settings

<a name="editor"></a>
## 富文本编辑器 (editor)

编辑器组件引用了网络资源，默认关闭,如果要开启这个组件参考[form组件管理](model-form-field-management.md).

```php
$form->editor($column[, $label]);
```

<a name="switch"></a>
## 开关 (switch)
开关表单保存到数据库的默认值
```php
$form->switch($column[, $label]);
```

<a name="display"></a>
## 仅显示 (display)
只显示字段，不做任何操作：
```php
$form->display($column[, $label]);


//更复杂的显示
$form->display($column[, $label])->with(function ($value) {
    return "<img src="$value" />";
});
```

<a name="divider"></a>
## 分割线 (divider)
```php
$form->divider();
```

<a name="html"></a>
## 自定义内容 (html)
插入html内容，参数可以是实现了`Htmlable`、`Renderable`或者实现了`__toString()`方法的类
```php
$form->html('你的html内容', $label = '');
```

<a name="tags"></a>
## 标签 (tags)
插入逗号(,)隔开的字符串`tags`
```php
$form->tags('keywords');
```

`tags`同样支持`ManyToMany`的关系，示例如下：

```php
$form->tags('tags', '文章标签')
    ->pluck('name', 'id') // name 为需要显示的 Tag 模型的字段，id 为主键
    ->options(Tag::all());// 下拉框选项
```

注意：处理`ManyToMany`关系时必须调用`pluck`方法，指定显示的字段名和主键。
此外 `options` 方法传入一个`Collection`对象时，`options`会自动调用该对象的`pluck`方法转为`['主键名' => '显示字段名']` 数组，作为下拉框选项。或者可以直接使用`['主键名' => '显示字段名']`这样的数组作为参数。

`tags`还支持`saving`方法用于处理提交的数据，示例如下：

```php
$form->tags('tags', '文章标签')
    ->pluck('name', 'id')
    ->options(Tag::all())
    ->saving(function ($value) {
        return $value;
    });
```

`saving` 方法接收一个「参数为 tags 的提交值，返回值为修改后的 tags 提交值」的闭包，可以用于实现自动创建新 tag 或其它功能。

<a name="icon"></a>
## 图表选择器 (icon)
选择`font-awesome`图标
```php
$form->icon('icon');
```

<a name="tree"></a>
## 树形选择器 (tree)
树形结构表单

```php
$form->tree('permissions')
    ->nodes($permissionModel->allNodes()) // 设置所有节点
    ->customFormat(function ($v) { // 格式化外部注入的值
        if (!$v) return [];

        return array_column($v, 'id');
    });

// 修改节点的字段名称
// 默认 “id” “name” “parent_id”
$form->tree('permissions')
    ->nodes($permissionModel->allNodes())
    ->columnNames('id', 'title', 'parent'); 
    
// 默认是不保存父节点的值的，因为一般来说父节点只是作为标题的形式存在
// 禁止过滤父节点的值
$form->tree('permissions')
    ->nodes($permissionModel->allNodes())
    ->disableFilterParents();
```

<a name="table"></a>
## 表格表单 (table)

> {tip} 表格表单不支持图片和文件上传表单。

如果某一个字段存储的是json格式的二维数组，可以使用table表单组件来实现快速的编辑：

```php
$form->table('extra', function (NestedForm $table) {
    $table->text('key');
    $table->text('value');
    $table->text('desc');
});
```
同时在模型里面给这个字段增加访问器和修改器:
```php
public function getExtraAttribute($extra)
{
    return array_values(json_decode($extra, true) ?: []);
}

public function setExtraAttribute($extra)
{
    $this->attributes['extra'] = json_encode(array_values($extra));
}
```
这个组件类似于hasMany组件，不过是用来处理单个字段的情况，适用于简单的二维数据。


<a name="onemany"></a>
## 一对多 (hasMany)

> {tip} 一对多表单暂不支持图片和文件上传表单，后续版本会增加这个功能。

一对多内嵌表格，用于处理一对多的关系，下面是个简单的例子：

有两张表是一对多关系：

```sql
CREATE TABLE `demo_painters` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `username` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
  `bio` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
  `created_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  `updated_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;

CREATE TABLE `demo_paintings` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `painter_id` int(10) unsigned NOT NULL,
  `title` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
  `body` text COLLATE utf8_unicode_ci NOT NULL,
  `completed_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  `created_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  `updated_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  PRIMARY KEY (`id`),
  KEY painter_id (`painter_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
```

表的模型为：
```php
<?php

namespace App\Models\Demo;

use Illuminate\Database\Eloquent\Model;

class Painter extends Model
{
    public function paintings()
    {
        return $this->hasMany(Painting::class, 'painter_id');
    }
}
```
```php
<?php

namespace App\Models\Demo;

use Illuminate\Database\Eloquent\Model;

class Painting extends Model
{
    protected $fillable = ['title', 'body', 'completed_at'];

    public function painter()
    {
        return $this->belongsTo(Painter::class, 'painter_id');
    }
}
```

构建表单代码如下：
```php
$form->display('id', 'ID');

$form->text('username')->rules('required');
$form->textarea('bio')->rules('required');

$form->hasMany('paintings', function (Form\NestedForm $form) {
    $form->text('title');
    $form->textarea('body');
    $form->datetime('completed_at');
});

$form->display('created_at', 'Created At');
$form->display('updated_at', 'Updated At');

// 也可以设置label

$form->hasMany('paintings', '画作', function (Form\NestedForm $form) {
    
});
```

<a name="embeds"></a>
## 内嵌 (embeds)

> {tip} 内嵌表单不支持图片和文件上传表单。

用于处理`mysql`的`JSON`类型字段数据或者`mongodb`的`object`类型数据，也可以将多个field的数据值以`JSON`字符串的形式存储在`mysql`的字符串类型字段中

比如`orders`表中的`JSON`或字符串类型的`extra`字段，用来存储多个field的数据，先定义model:
```php
class Order extends Model
{
    protected $casts = [
        'extra' => 'json',
    ];
}
```
然后在form中使用：
```php
$form->embeds('extra', function ($form) {

    $form->text('extra1')->rules('required');
    $form->email('extra2')->rules('required');
    $form->mobile('extra3');
    $form->datetime('extra4');

    $form->dateRange('extra5', 'extra6', '范围')->rules('required');

});

// 自定义标题
$form->embeds('extra', '附加信息', function ($form) {
    ...
});
```

回调函数里面构建表单元素的方法调用和外面是一样的。
