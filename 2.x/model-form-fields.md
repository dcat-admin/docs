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



此方法仅在**新增页面**有效，如果**编辑页面**也需要设置默认值，可以参考以下方法

```php
$form->text('xxx')->default('默认值', true);
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
### 修改待保存的表单输入值 (saving)

通过`saving`方法可以更改待保存数据的格式。

```php
use Dcat\Admin\Support\Helper;

$form->mutipleFile('files')->saving(function ($paths) {
    $paths = Helper::array($paths);
    
    // 获取数据库当前行的其他字段
    $username = $this->username;
    
    // 最终转化为json保存到数据库
    return json_encode($paths);
});
```

<a name="customFormat"></a>
### 修改表单数据显示 (customFormat)
通过`customFormat`方法可以改变从外部注入到表单的字段值。

如下例子中，`mutipleFile`字段要求待渲染的字段值为数组格式，我们可以通过`customFormat`方法把从数据库查出的字段值转化为`array`格式
```php
use Dcat\Admin\Support\Helper;

$form->mutipleFile('files')->saving(function ($paths) {
    $paths = Helper::array($paths);
    
    return json_encode($paths);
})->customFormat(function ($paths) {
	// 获取数据库当前行的其他字段
    $username = $this->username;

    // 转为数组
    return Helper::array($paths);
});
```

### 在弹窗中隐藏

如果不想在弹窗中显示某个字段，可以使用`Form\Field::hideInDialog`方法

```php
$form->display('id');
$form->text('title');
$form->textarea('contents')->hideInDialog();
```

### 保存为字符串类型 `saveAsString`


`saveAsString`方法可以把表单值转化为`string`类型保存，当保存的数据库字段值不允许为`null`时非常有用；

```php
$form->text('nickname')->saveAsString();
```

### 保存为json类型 `saveAsJson`


`saveAsJson`可以把表单值保存为`json`格式

```php
$form->text('nickname')->saveAsJson();
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
})->ajax('api/users');
```

使用model方法在編輯時載入資料：
```php
$form->select('user_id')->model(User::class, 'id', 'name');
```
上面的代碼跟下面這個是一樣的
```php
$form->select('user_id')->options(function ($id) {
    $user = User::find($id);

    if ($user) {
        return [$user->id => $user->name];
    }
});
```


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
## 下拉选框联动 (load)

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

`selectTable`、`multipleSelectTable`、`radio`、`checkbox`也可以使用`load`方法联动`select`和`multipleSelect`表单，用法和上面的示例一致。

### 联动多个字段 (loads)

使用`loads`方法可以联动多个字段，用法如下

```php
$form->select('status')
    ->options(...)
    ->loads(['field1', 'field2'], ['/api/field1', '/api/field2']);

$form->select('field1');
$form->select('field2');
```

`api`返回的数据格式与`load`方法一致，`selectTable`、`multipleSelectTable`、`radio`、`checkbox`也可以使用`loads`方法联动。


<a name="multipleSelect"></a>
## 下拉选框多选 (multipleSelect)

> {tip} 注入这个字段的数据（从数据库查出来的）可以是一个以`,`隔开的字符串，也可以是`json`字符串或`array`数组。

```php
$form->multipleSelect($column[, $label])
	->options([1 => 'foo', 2 => 'bar', 'val' => 'Option name'])
	->saving(function ($value) {
		// 转化成json字符串保存到数据库
		return json_encode($value);
	});

// 使用ajax并显示所选项目：

$form->multipleSelect($column[, $label])->options(Model::class)->ajax('ajax_url');

// 或指定名称和ID

$form->multipleSelect($column[, $label])->options(Model::class, 'name', 'id')->ajax('ajax_url');
```

多选框可以处理两种情况，第一种是`ManyToMany`的关系。

```php
class Post extends Models
{
    public function tags()
    {
        return $this->belongsToMany(Tag::class);
    }
}

return Form::make(Post::with('tags'), function (Form $form) {
    ...

    $form->multipleSelect('tags')
        ->options(Tag::all()->pluck('name', 'id'))
        ->customFormat(function ($v) {
            if (! $v) {
                return [];
            }
            
            // 从数据库中查出的二维数组中转化成ID
            return array_column($v, 'id');
        });
});

```

第二种是将选项数组存储到单字段中，如果字段是字符串类型，那就需要在模型里面为该字段定义[访问器和修改器](https://laravel.com/docs/5.5/eloquent-mutators)来存储和读取了。

如果选项过多，可通过ajax方式动态分页载入选项：

```php
$form->select('friends')->options(function ($ids) {

    return User::find($ids)->pluck('name', 'id');
    
})->ajax('api/users');
```

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

<a name="select-table"></a>
## 表格选择器 (selectTable)

```php
use App\Admin\Renderable\UserTable;
use Dcat\Admin\Models\Administrator;

$form->selectTable($field)
	->title('弹窗标题')
	->dialogWidth('50%') // 弹窗宽度，默认 800px
	->from(UserTable::make(['id' => $form->getKey()])) // 设置渲染类实例，并传递自定义参数
	->model(Administrator::class, 'id', 'name'); // 设置编辑数据显示
	
// 上面的代码等同于
$form->selectTable($field)
	->from(UserTable::make(['id' => $form->getKey()])) // 设置渲染类实例，并传递自定义参数
	->options(function ($v) { // 设置编辑数据显示
		if (! $v) {
			return [];
		}
		
		return Administrator::find($v)->pluck('name', 'id');
	});
```

定义渲染类如下，需要继承`Dcat\Admin\Grid\LazyRenderable`

> {tip} 这里使用了数据表格异步加载功能，详细用法请参考[异步加载](lazy.md)

```php
<?php

namespace App\Admin\Renderable;

use Dcat\Admin\Grid;
use Dcat\Admin\Grid\LazyRenderable;
use Dcat\Admin\Models\Administrator;

class UserTable extends LazyRenderable
{
    public function grid(): Grid
    {
        // 获取外部传递的参数
        $id = $this->id;
        
        return Grid::make(Administrator::where('xxx_id', $id), function (Grid $grid) {
            $grid->column('id');
            $grid->column('username');
            $grid->column('name');
            $grid->column('created_at');
            $grid->column('updated_at');
            
            $grid->quickSearch(['id', 'username', 'name']);

            $grid->paginate(10);
            $grid->disableActions();

            $grid->filter(function (Grid\Filter $filter) {
                $filter->like('username')->width(4);
                $filter->like('name')->width(4);
            });
        });
    }
}
```

效果

<a href="https://cdn.learnku.com/uploads/images/202008/23/38389/P5hZXiqAj9.gif!large" target="_blank">
![](https://cdn.learnku.com/uploads/images/202008/23/38389/P5hZXiqAj9.gif!large)
</a>

### 设置选中后将保存到表单的字段和显示的字段

```php
$form->selectTable($field)
	->from(UserTable::make(['id' => $form->getKey()]))
	->options(function ($v) { // 设置编辑数据显示
		if (! $v) {
			return [];
		}
		
		return Administrator::find($v)->pluck('name', 'id');
	})
	->pluck('name', 'id'); // 第一个参数为显示的字段，第二个参数为选中后将保存到表单的字段
	
// 上面的代码也可以简化为
$form->selectTable($field)
	->from(UserTable::make(['id' => $form->getKey()]))
	->model(Administrator::class, 'id', 'name'); // 设置编辑数据显示
```



### 多选 (multipleSelectTable)

多选的用法与上述`selectTable`方法一致

```php
$form->multipleSelectTable($field)
	->max(10) // 最多选择 10 个选项，不传则不限制
	->from(UserTable::make(['id' => $form->getKey()])) // 设置渲染类实例，并传递自定义参数
	->model(Administrator::class, 'id', 'name')  // 设置编辑数据显示
	->saving(function ($v) {
		// $v 是表单提交的字段值，默认是数组类型，这里需要手动转换一下
		// 保存为以 "," 隔开的字符串，如果是多对多关联关系，则不需要转换。
		return implode(',', $v);
	});
```


## 多选盒 (listbox)

使用方法和`multipleSelect`类似。

> {tip} 注入这个字段的数据（从数据库查出来的）可以是一个以`,`隔开的字符串，也可以是`json`字符串或`array`数组。

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

`options()`方法用来设置选择项:
```php
$form->checkbox($column[, $label])
	->options([1 => 'foo', 2 => 'bar', 'val' => 'Option name'])
	->saving(function ($value) {
		// 转化成json字符串保存到数据库
		return json_encode($value);
	});
```


启用选中全部功能

```php
$form->checkbox('column')->canCheckAll();
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

<a name="color"></a>
## 颜色选择器 (color)


```php
$form->color($column[, $label]);
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

## 范围 (range)


```php
$form->range('start_column', 'end_column', '范围');
```


<a name="file"></a>
## 文件上传 (file)

使用文件上传功能之前需要先完成上传配置。文件上传配置以及内置方法请参考:[图片/文件上传](model-form-upload.md).

文件上传目录在文件`config/admin.php`中的`upload.file`中配置，如果目录不存在，需要创建该目录并开放写权限。


> {tip} 文件或图片上传表单字段请不要在模型中设置**访问器**和**修改器**拼接域名，如有相关需求可参考[文件/图片域名拼接](model-form-upload.md#withhost)。


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


> {tip} 文件或图片上传表单字段请不要在模型中设置**访问器**和**修改器**拼接域名，如有相关需求可参考[文件/图片域名拼接](model-form-upload.md#withhost)。


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

> {tip} 注入这个字段的数据（从数据库查出来的）可以是一个以`,`隔开的字符串，也可以是`json`字符串或`array`数组。

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

多图/文件上传的时候提交的数据是一个`array`数组，你可以通过以下方式把数据在保存进数据库之前改为你想要的格式：
```php
// 转化为json格式保存到数据库
$form->multipleFile($column[, $label])->saving(function ($paths) {
    // 可以转化为由 , 隔开的字符串格式
    // return implode(',', $paths);
    
    // 也可以转化为json
    return json_encode($paths);
});
```

当然，如果你想保存为其他任意格式都是可以的，只是如果要保存其他格式，需要通过用`customFormat`方法把数据转化为一维数组展示，如：
```php
$form->multipleFile($column[, $label])
    ->customFormat(function ($paths) {
        return collect($paths)->map(function ($value) {
        	return explode('|', $value);
        })->flatten()->toArray();
    })
    ->saving(function ($paths) {
        return implode('|', $paths);
    });
```


启用可排序功能

```php
$form->multipleImage('images')->sortable();
```


<a name="editor"></a>
## 富文本编辑器 (editor)

系统集成了`TinyMCE`编辑器作为内置编辑器。


#### 基本使用
```php
$form->editor($column[, $label]);
```

#### 设置语言包

默认支持简体中文和英文两种语言，如需其他语言可以通过以下方式设置语言包地址。

```php
$form->editor('content')->languageUrl(url('TinyMCE/langs/de.js'));
```

#### 编辑器配置

具体配置等的使用可以参考[官方文档](https://www.tiny.cloud/docs)或[中文文档](http://tinymce.ax-z.cn/)

```php
<?php

use Dcat\Admin\Support\JavaScript;

$form->editor('content')->options([
    'toolbar' => [],
    'setup' => JavaScript::make(
        <<<JS
function (editor) {
    console.log('编辑器初始化成功', editor)
}
JS
    ),
]);
```

#### 设置高度

默认值为`400`

```php
$form->editor('content')->height('600');
```


#### 只读

```php
$form->editor('content')->readOnly();

// 或
$form->editor('content')->disable();
```

#### 图片上传

设置图片上传`disk`配置，默认上传到`admin.upload.disk`指定的配置

```php
// 上传到oss
$form->editor('content')->disk('oss');
```

设置图片上传目录，默认为`tinymce/images`
```php
$form->editor('content')->imageDirectory('editor/images');
```

自定义上传接口，接口返回格式需要是`{"location": "图片url"}`
```php
$form->editor('content')->imageUrl('editor/upload-image');
```


#### 全局设置

如果你需要对编辑器进行全局设置，可以在`app\Admin\bootstrap.php`加上以下代码

```php
<?php

use Dcat\Admin\Form\Field\Editor;

Editor::resolving(function (Editor $editor) {
    // 设置默认配置
    $editor->options([...]);
    
    // 设置编辑器图片默认上传到七牛云
    $editor->disk('qiniu');
});
```



<a name="markdown"></a>
## Markdown编辑器 (markdown)

系统集成了`editor-md`编辑器作为内置Markdown编辑器。


#### 基本使用
```php
$form->markdown($column[, $label]);
```

#### 设置语言包

默认支持简体中文和英文两种语言，如需其他语言可以通过以下方式设置语言包地址。

```php
$form->markdown('content')->languageUrl(admin_asset('@admin/dcat/plugins/editor-md/languages/zh-tw.js'));
```

#### 编辑器配置

具体配置等的使用可以参考[官方文档](https://pandao.github.io/editor.md/)

```php
<?php

use Dcat\Admin\Support\JavaScript;

$form->markdown('content')->options([
    'onpreviewing' => JavaScript::make(
        <<<JS
function() {
	// console.log("onpreviewing =>", this, this.id, this.settings);
	// on previewing you can custom css .editormd-preview-active
}
JS
    ),
]);
```

#### 设置高度

默认值为`500`

```php
$form->markdown('content')->height('600');
```


#### 只读

```php
$form->markdown('content')->readOnly();

// 或
$form->markdown('content')->disable();
```


#### 图片上传

设置图片上传`disk`配置，默认上传到`admin.upload.disk`指定的配置

```php
// 上传到oss
$form->markdown('content')->disk('oss');
```

设置图片上传目录，默认为`markdown/images`
```php
$form->markdown('content')->imageDirectory('markdown/images');
```

自定义上传接口，接口返回格式需要是`{"success": 1, "url": "图片url"}`
```php
$form->markdown('content')->imageUrl('markdown/upload-image');
```

#### 全局设置

如果你需要对编辑器进行全局设置，可以在`app\Admin\bootstrap.php`加上以下代码

```php
<?php

use Dcat\Admin\Form\Field\Markdown;

Markdown::resolving(function (Markdown $markdown) {
    // 设置默认配置
    $markdown->options([...]);
    
    // 设置编辑器图片默认上传到七牛云
    $markdown->disk('qiniu');
});
```

<a name="switch"></a>
## 开关 (switch)

使用
```php
$form->switch($column[, $label]);
```

开关表单保存到数据库的默认值为`1`和`0`，如果需要更改保存到数据库的值，可以这样使用

```php
$form->switch($column[, $label])
	->customFormat(function ($v) {
		return $v == '打开' ? 1 : 0;
	})
	->saving(function ($v) {
		return $v ? '打开' : '关闭';
	});
```


<a name="map"></a>
## 地图 (map)

地图控件，用来选择经纬度,`$latitude`, `$longitude`为经纬度字段。

使用这个功能需要在 `config/admin.php` 文件中修改 `map_provider` 的值（目前支持的地图为："tencent", "google", "yandex"，不同地图需要自己申请相应的 KEY 并在 .env 文件中配置，然后需要在`app/Admin/bootstrap.php`中加入以下代码
```php
Form\Field\Map::requireAssets();
```

使用

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



如果选项过多，可通过ajax方式动态分页载入选项：

```php
$form->tags('friends')->options(function ($ids) {
    return User::find((array) $ids)->pluck('name', 'id');
    
})->ajax('api/users');
```

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


<a name="icon"></a>
## 图标选择器 (icon)
选择`font-awesome`图标
```php
$form->icon('icon');
```

<a name="tree"></a>
## 树形选择器 (tree)
树形结构表单

```php
$form->tree('permissions')
    ->nodes(Model::get()->toArray()) // 设置所有节点
    ->customFormat(function ($v) { // 格式化外部注入的值
        if (!$v) return [];

        return array_column($v, 'id');
    });

// 修改节点的字段名称
// 默认 “id” “name” “parent_id”
$form->tree('permissions')
    ->nodes($permissionModel->allNodes())
    ->setIdColumn('id')
    ->setTitleColumn('title')
    ->setParentColumn('parent'); 
    
// 默认是不保存父节点的值的，因为一般来说父节点只是作为标题的形式存在
// 禁止过滤父节点的值
$form->tree('permissions')
    ->nodes($permissionModel->allNodes())
    ->exceptParentNode(false);
    
// 默认收缩子节点
$form->tree('permissions')
    ->nodes($permissionModel->allNodes())
    ->expand(false);
```

<a name="table"></a>
## 表格表单 (table)

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
use App\Models\Demo\Painter;

// 这里需要显式地指定关联关系
$builder = Painter::with('paintings');

// 如果你使用的是数据仓库，则可以这样指定关联关系
// $repository = new Painter(['paintings']);

return Form::make($builder, function (Form $form) {
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
    // $form->hasMany('paintings', '画作', function (Form\NestedForm $form) {
    //    ...
    // });
});
```

效果

![]({{public}}/assets/img/screenshots/has-many.png)


<a name="has-many-table"></a>
### 表格模式 (table)

如果你想要显示表格模式，可以这样使用

```php
use Dcat\Admin\Support\Helper;

$form->hasMany('paintings', function (Form\NestedForm $form) {
    ...
})->useTable();
```
效果

<a href="{{public}}/assets/img/screenshots/has-many-table.png" target="_blank">
    <img  src="{{public}}/assets/img/screenshots/has-many-table.png" style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%">
</a>


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
