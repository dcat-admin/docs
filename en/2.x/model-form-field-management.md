# Management of form fields


## Extending custom components

> {tip} The IDE does not auto-complete by default after extending a custom Form component, so you can generate an IDE hint file with `php artisan admin:ide-helper`.


### Integrated rich text editor wangEditor

[wangEditor](http://www.wangeditor.com/) is an excellent home-made lightweight rich text editor, if `dcat-admin` comes with an editor component based on `ckeditor`, you can use the following steps to integrate it and override `ckeditor`. .

First download the front-end library file [wangEditor](https://github.com/wangfupeng1988/wangEditor/releases) and extract it to the directory `public/vendor/wangEditor-3.0.9`.

Then create a new component class `app/Admin/Extensions/WangEditor.php`.

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
// Note that the ID here must be converted by the replaceNestedFormIndex function, otherwise it will not be compatible with the hasMany form.
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

New view file `resources/views/admin/wang-editor.blade.php`：
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

Then register into `dcat-admin` and add the following code to `app/Admin/bootstrap.php`.

```php

<?php

use App\Admin\Extensions\WangEditor;
use Dcat\Admin\Form;

Form::extend('editor', WangEditor::class);

```

Calling:

```

$form->editor('body');

```

### Integrated rich text editor ckeditor

Download [ckeditor](http://ckeditor.com/download) and extract it to the /public directory, e.g. put it in `/public/packages/`.

Then create a new extension file `app/Admin/Extensions/Form/CKEditor.php`:
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

New view `resources/views/admin/ckeditor.blade.php`:
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

Then in `app/Admin/bootstrap.php`, import the extension:
```php
use App\Admin\Extensions\Form\CKEditor;
use Dcat\Admin\Form;

Form::extend('ckeditor', CKEditor::class);
```

Then you can use it in a form:
```php
$form->ckeditor('content');
```

### Integrated PHP editor


Extend a PHP code editor based on [codemirror](http://codemirror.net/index.html) by following the steps below, with reference to [PHP mode](http://codemirror.net/mode/php/).

Download and extract the [codemirror](http://codemirror.net/codemirror.zip) library to a front-end resource directory, such as `public/packages/codemirror-5.20.2`.

Create new component class `app/Admin/Extensions/PHPEditor.php`:

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
// Note that the ID here must be converted by the replaceNestedFormIndex function, otherwise it will not be compatible with the hasMany form
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

> {tip} Static resources in the class can also be brought in from outside, see [Editor.php](https://github.com/jqhph/dcat-admin/blob/master/src/Form/Field/Editor.php)

Creating View `resources/views/admin/php-editor.blade.php`:

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

Finally find the file `app/Admin/bootstrap.php`, if the file does not exist, please update `dcat-admin`, then create the file, add the following code:

```
<?php

use App\Admin\Extensions\PHPEditor;
use Dcat\Admin\Form;

Form::extend('php', PHPEditor::class);

```

This will allow you to use the PHP editor in [model-form](model-form.md):

```

$form->php('code');

```

In this way, you can add as many `form` components as you want.


## Common methods

The form component is the most complex component in `Dcat Admin`, the following is a list of methods that may be needed to extend the form component


### Process user-entered form values (prepareInputValue)

The `prepareInputValue` method processes the form values entered by the user and does not do any processing by default. This method is executed after the `Form` form's `saving` event is fired and before the `saving` method of the form field.

> {tip} This is similar to the **mutator** in the `Laravel model`.

```php
class PHPEditor extends Field
{
	...
	
	// Converts user-entered form values into string format for saving to database
	protected function prepareInputValue($value)
	{
		return (string) $value;
	}
}
```

### Process the value of the field to be displayed (formatFieldData)

The `formatFieldData` method is used to process the value of the field to be displayed. This method is executed before the `customFormat` method of the form field.

> {tip} This function is similar to the **accessor** in the `Laravel model`.

```php
class PHPEditor extends Field
{
	...
	
	// Convert field values to array format.
	// $data is the edit data of the current form, the data format is array
	protected function formatFieldData($data)
	{
		return (array) parent::formatFieldData($data);
	}
}
```

### Get the element CSS selector (getElementClassSelector)

When the form component class is instantiated, it generates a `css class` according to the field name, which is then passed to the template.

Template
```html
<div class="{{$viewClass['form-group']}} {!! !$errors->has($errorKey) ? '' : 'has-error' !!}">

    <div for="{{ $id }}" class="{{ $viewClass['label'] }} control-label">
        <span>{!! $label !!}</span>
    </div>


    <div class="{{$viewClass['field']}}">

        @include('admin::form.error')

        <input type="hidden" name="{{$name}}"/>

		<!-- $class That is the class generated according to the field name -->
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

usage

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

### ID attributes
When the form component class is instantiated, it generates a unique random string and stores it in the `id` attribute, which is then passed to the template.

Template
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

Using the `$id` attribute

```php
class PHPEditor extends Field
{
    ...

    public function render()
    {
    	// Locate elements by $this->id
        $this->script = <<<JS
// Note that the ID here must be converted by the replaceNestedFormIndex function, otherwise it will not be compatible with the hasMany form.
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


### JS code

In order for the extended form component to be compatible with `HasMany` and `Table` forms, we must save the `JS` code in the `$script` attribute!

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

And if you are positioning the element by the `id` attribute, you must use the `replaceNestedFormIndex` function to convert the original `id`, otherwise it will not be compatible with `HasMany` and `Table` forms.


```php
class PHPEditor extends Field
{
    ...

    public function render()
    {
    	// Locate elements by $this->id
        $this->script = <<<JS
// Note that the ID here must be converted by the replaceNestedFormIndex function, otherwise it will not be compatible with the hasMany form.
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




