# 表单字段的管理


## 扩展自定义组件

> {tip} 扩展自定义Form组件后IDE默认是不会自动补全的，这时候可以通过`php artisan admin:ide-helper`生成IDE提示文件。


### 集成富文本编辑器wangEditor

[wangEditor](http://www.wangeditor.com/)是一个优秀的国产的轻量级富文本编辑器，如果`dcat-admin`自带的基于`ckeditor`的编辑器组件使用上有问题，可以通过下面的步骤可以集成它，并覆盖掉`ckeditor`：

先下载前端库文件[wangEditor](https://github.com/wangfupeng1988/wangEditor/releases)，解压到目录`public/vendor/wangEditor-3.0.9`。

然后新建组件类`app/Admin/Extensions/WangEditor.php`。

```php

<?php

namespace App\Admin\Extensions;

use Dcat\Admin\Form\Field;

class WangEditor extends Field
{
    protected $view = 'admin.wang-editor';

    protected static $css = [
        '/vendor/wangEditor-3.0.9/release/wangEditor.min.css',
    ];

    protected static $js = [
        '/vendor/wangEditor-3.0.9/release/wangEditor.min.js',
    ];

    public function render()
    {
        $name = $this->formatName($this->column);

        $this->script = <<<EOT
// 注意这里的ID一定要通过 replaceNestedFormIndex 函数转换，否则将无法兼容 hasMany 表单
var E = window.wangEditor
var editor = new E(replaceNestedFormIndex('#{$this->id}'));
editor.customConfig.zIndex = 0
editor.customConfig.uploadImgShowBase64 = true
editor.customConfig.onchange = function (html) {
    $('input[name=$name]').val(html);
}
editor.create()

EOT;
        return parent::render();
    }
}

```

新建视图文件`resources/views/admin/wang-editor.blade.php`：
```php
<div class="{{$viewClass['form-group']}} {!! !$errors->has($label) ? '' : 'has-error' !!}">

    <label for="{{$id}}" class="{{$viewClass['label']}} control-label">{{$label}}</label>

    <div class="{{$viewClass['field']}}">

        @include('admin::form.error')

        <div id="{{$id}}" style="width: 100%; height: 100%;">
            <p>{!! old($column, $value) !!}</p>
        </div>

        <input type="hidden" name="{{$name}}" value="{{ old($column, $value) }}" />

		@include('admin::form.help-block')

    </div>
</div>
```

然后注册进`dcat-admin`,在`app/Admin/bootstrap.php`中添加以下代码：

```php

<?php

use App\Admin\Extensions\WangEditor;
use Dcat\Admin\Form;

Form::extend('editor', WangEditor::class);

```

调用:

```

$form->editor('body');

```

### 集成富文本编辑器ckeditor

先下载[ckeditor](http://ckeditor.com/download) 并解压到/public目录，比如放在`/public/packages/`目录下。

然后新建扩展文件`app/Admin/Extensions/Form/CKEditor.php`:
```php
<?php

namespace App\Admin\Extensions\Form;

use Dcat\Admin\Form\Field;

class CKEditor extends Field
{
    public static $js = [
        '/packages/ckeditor/ckeditor.js',
        '/packages/ckeditor/adapters/jquery.js',
    ];

    protected $view = 'admin.ckeditor';

    public function render()
    {
        $this->script = "$('{$this->getElementClassSelector()}').ckeditor();";

        return parent::render();
    }
}
```

新建view `resources/views/admin/ckeditor.blade.php`:
```php
<div class="{{$viewClass['form-group']}} {!! !$errors->has($label) ? '' : 'has-error' !!}">

    <label for="{{$id}}" class="{{$viewClass['label']}} control-label">{{$label}}</label>

    <div class="{{$viewClass['field']}}">

        @include('admin::form.error')

        <textarea class="form-control {{ $class }}" name="{{$name}}" placeholder="{{ $placeholder }}" {!! $attributes !!} >{{ old($column, $value) }}</textarea>

        @include('admin::form.help-block')

    </div>
</div>

```

然后在`app/Admin/bootstrap.php`中引入扩展：
```php
use App\Admin\Extensions\Form\CKEditor;
use Dcat\Admin\Form;

Form::extend('ckeditor', CKEditor::class);
```

然后就能在form中使用了:
```php
$form->ckeditor('content');
```

### 集成PHP editor


通过下面的步骤来扩展一个基于[codemirror](http://codemirror.net/index.html)的PHP代码编辑器，效果参考[PHP mode](http://codemirror.net/mode/php/)。

先将[codemirror](http://codemirror.net/codemirror.zip)库下载并解压到前端资源目录下，比如放在`public/packages/codemirror-5.20.2`目录下。

新建组件类`app/Admin/Extensions/PHPEditor.php`:

```php
<?php

namespace App\Admin\Extensions;

use Dcat\Admin\Form\Field;

class PHPEditor extends Field
{
    protected $view = 'admin.php-editor';

    protected static $css = [
        '/packages/codemirror-5.20.2/lib/codemirror.css',
    ];

    protected static $js = [
        '/packages/codemirror-5.20.2/lib/codemirror.js',
        '/packages/codemirror-5.20.2/addon/edit/matchbrackets.js',
        '/packages/codemirror-5.20.2/mode/htmlmixed/htmlmixed.js',
        '/packages/codemirror-5.20.2/mode/xml/xml.js',
        '/packages/codemirror-5.20.2/mode/javascript/javascript.js',
        '/packages/codemirror-5.20.2/mode/css/css.js',
        '/packages/codemirror-5.20.2/mode/clike/clike.js',
        '/packages/codemirror-5.20.2/mode/php/php.js',
    ];

    public function render()
    {
        $this->script = <<<EOT
// 注意这里的ID一定要通过 replaceNestedFormIndex 函数转换，否则将无法兼容 hasMany 表单
CodeMirror.fromTextArea(document.getElementById(replaceNestedFormIndex("{$this->id}")), {
    lineNumbers: true,
    mode: "text/x-php",
    extraKeys: {
        "Tab": function(cm){
            cm.replaceSelection("    " , "end");
        }
     }
});

EOT;
        return parent::render();

    }
}

```

> {tip} 类中的静态资源也同样可以从外部引入，参考[Editor.php](https://github.com/jqhph/dcat-admin/blob/master/src/Form/Field/Editor.php)

创建视图`resources/views/admin/php-editor.blade.php`:

```php

<div class="{{$viewClass['form-group']}} {!! !$errors->has($label) ? '' : 'has-error' !!}">

    <label for="{{$id}}" class="{{$viewClass['label']}} control-label">{{$label}}</label>

    <div class="{{$viewClass['field']}}">

        @include('admin::form.error')

        <textarea class="form-control" id="{{$id}}" name="{{$name}}" placeholder="{{ trans('admin::lang.input') }} {{$label}}" {!! $attributes !!} >{{ old($column, $value) }}</textarea>
        
        @include('admin::form.help-block')
    </div>
</div>

```

最后找到文件`app/Admin/bootstrap.php`,如果文件不存在，请更新`dcat-admin`，然后新建该文件,添加下面代码：

```
<?php

use App\Admin\Extensions\PHPEditor;
use Dcat\Admin\Form;

Form::extend('php', PHPEditor::class);

```

这样就能在[model-form](model-form.md)中使用PHP编辑器了：

```

$form->php('code');

```

通过这种方式，可以添加任意你想要添加的`form`组件。


## 常用方法

表单组件是`Dcat Admin`中最为复杂的组件，下面列一些在扩展表单组件中可能需要用到的方法


### 处理用户输入的表单值 (prepareInputValue)

通过`prepareInputValue`方法可以处理用户输入的表单值，默认不做任何处理。此方法在`Form`表单的`saving`事件触发之后，表单字段的`saving`方法之前执行

> {tip} 这个功能类似`Laravel model`中的**修改器**。

```php
class PHPEditor extends Field
{
	...
	
	// 把用户输入的表单值转化为 string 格式保存到数据库
	protected function prepareInputValue($value)
	{
		return (string) $value;
	}
}
```

### 处理待显示的字段值 (formatFieldData)

通过`formatFieldData`方法可以处理处理待显示的字段值。此方法在表单字段的`customFormat`方法之前执行

> {tip} 这个功能类似`Laravel model`中的**访问器**。

```php
class PHPEditor extends Field
{
	...
	
	// 把字段值转化为数组格式
	// $data是当前表单的编辑数据，数据格式是 array
	protected function formatFieldData($data)
	{
		return (array) parent::formatFieldData($data);
	}
}
```

### 获取元素CSS选择器 (getElementClassSelector)

表单组件类实例化后会根据字段名称生成一个`css class`，然后传递到模板中，我们通常可以通过这个`class`获取到当前表单的元素对象

模板
```html
<div class="{{$viewClass['form-group']}} {!! !$errors->has($errorKey) ? '' : 'has-error' !!}">

    <div for="{{ $id }}" class="{{ $viewClass['label'] }} control-label">
        <span>{!! $label !!}</span>
    </div>


    <div class="{{$viewClass['field']}}">

        @include('admin::form.error')

        <input type="hidden" name="{{$name}}"/>

		<!-- $class 即是根据字段名称生成的class -->
        <select class="form-control {{$class}}" style="width: 100%;" name="{{$name}}" {!! $attributes !!} >
           <option value=""></option>
		   @foreach($options as $select => $option)
			   <option value="{{$select}}" {{ Dcat\Admin\Support\Helper::equal($select, old($column, $value)) ?'selected':'' }}>{{$option}}</option>
		   @endforeach
        </select>

        @include('admin::form.help-block')

    </div>
</div>
```

使用

```php
class Select extends Field
{
	...
	
	/**
	 * {@inheritdoc}
	 */
	public function render()
	{
		...

		if (empty($this->script)) {
			// 通过 getElementClassSelector 方法可以获取到表单中 select 标签的css选择器
			$this->script = "$(\"{$this->getElementClassSelector()}\").select2($configs);";
		}

		...

		return parent::render();
	}
}
```

### ID属性
表单组件类实例化后会生成一个唯一的随机字符串并保存到`id`属性中，然后传递到模板中，我们通常可以通过这个`id`获取到当前表单组件的关键元素

模板
```html
<div class="{{$viewClass['form-group']}} {!! !$errors->has($label) ? '' : 'has-error' !!}">

    <label for="{{$id}}" class="{{$viewClass['label']}} control-label">{{$label}}</label>

    <div class="{{$viewClass['field']}}">

        @include('admin::form.error')
		
		<!-- $id -->
        <textarea class="form-control" id="{{$id}}" name="{{$name}}" placeholder="{{ trans('admin::lang.input') }} {{$label}}" {!! $attributes !!} >{{ old($column, $value) }}</textarea>
        
        @include('admin::form.help-block')
    </div>
</div>
```

使用`$id`属性

```php
class PHPEditor extends Field
{
    ...

    public function render()
    {
    	// 通过 $this->id 定位元素
        $this->script = <<<JS
// 注意这里的ID一定要通过 replaceNestedFormIndex 函数转换，否则将无法兼容 hasMany 表单
CodeMirror.fromTextArea(document.getElementById(replaceNestedFormIndex("{$this->id}")), {
    lineNumbers: true,
    mode: "text/x-php",
    extraKeys: {
        "Tab": function(cm){
            cm.replaceSelection("    " , "end");
        }
     }
});

JS;
        return parent::render();

    }
}
```


### JS代码

为了让扩展的表单组件能够兼容`HasMany`以及`Table`表单，我们必须把`JS`代码保存在`$script`属性中

```php
class Select extends Field
{
	...
	
	/**
	 * {@inheritdoc}
	 */
	public function render()
	{
		...

		$this->script = "$(\"{$this->getElementClassSelector()}\").select2($configs);";

		...

		return parent::render();
	}
}
```

并且如果你是通过`id`属性去定位元素，则必须使用`replaceNestedFormIndex`函数对原始`id`进行转换，否则将无法兼容`HasMany`以及`Table`表单


```php
class PHPEditor extends Field
{
    ...

    public function render()
    {
    	// 通过 $this->id 定位元素
        $this->script = <<<JS
// 注意这里的ID一定要通过 replaceNestedFormIndex 函数转换，否则将无法兼容 hasMany 表单
CodeMirror.fromTextArea(document.getElementById(replaceNestedFormIndex("{$this->id}")), {
    lineNumbers: true,
    mode: "text/x-php",
    extraKeys: {
        "Tab": function(cm){
            cm.replaceSelection("    " , "end");
        }
     }
});
JS;
        return parent::render();

    }
}
```




