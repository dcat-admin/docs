# Management of form fields


## Extending custom components

It is recommended to use the `Dcat.init` method to listen and initialize the `JS` code of the form component, otherwise it may not be compatible with dynamically generated fields such as `Form::hasMany` and `Form::array`. Form Type.

### Integrated rich text editor wangEditor

[wangEditor](http://www.wangeditor.com/) is an excellent home-made lightweight rich text editor, if `dcat-admin` comes with an editor component based on `ckeditor`, you can use the following steps to integrate it and override `ckeditor`. .

First download the front-end library file [wangEditor](https://github.com/wangfupeng1988/wangEditor/releases) and extract it to the directory `public/vendor/wangEditor-3.0.9`.

Then create a new component class `app/Admin/Extensions/WangEditor.php`.

```php
<?php

namespace App\Admin\Extensions\Form;

use Dcat\Admin\Form\Field;

class WangEditor extends Field
{
    protected $view = 'admin.wang-editor';
}
```

New view file `resources/views/admin/wang-editor.blade.php`：
```php
<div class="{{$viewClass['form-group']}}">

    <label class="{{$viewClass['label']}} control-label">{{$label}}</label>

    <div class="{{$viewClass['field']}}">

        @include('admin::form.error')

        <div {!! $attributes !!} style="width: 100%; height: 100%;">
            <p>{!! $value !!}</p>
        </div>

        <input type="hidden" name="{{$name}}" value="{{ old($column, $value) }}" />

		@include('admin::form.help-block')

    </div>
</div>

<script require="@wang-editor" init="{!! $selector !!}">
    var E = window.wangEditor
    var editor = new E('#' + id);
    editor.customConfig.zIndex = 0
    editor.customConfig.uploadImgShowBase64 = true
    editor.customConfig.onchange = function (html) {
        $this.parents('.form-field').find('input[name={{ $name }}]').val(html);
    }
    editor.create()
</script>

```

Then register into `dcat-admin` and add the following code to `app/Admin/bootstrap.php`.

```php
<?php

use App\Admin\Extensions\Form\WangEditor;
use Dcat\Admin\Form;

// Register front-end component aliases
Admin::asset()->alias('@wang-editor', [
    'js' => ['/vendor/wangEditor-3.0.9/release/wangEditor.min.js'],
    'css' => ['/vendor/wangEditor-3.0.9/release/wangEditor.min.css'],
]);

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
    protected $view = 'admin.ckeditor';
}
```

New view `resources/views/admin/ckeditor.blade.php`:
```html
<div class="{{$viewClass['form-group']}}">

    <label class="{{$viewClass['label']}} control-label">{{$label}}</label>

    <div class="{{$viewClass['field']}}">

        @include('admin::form.error')

        <textarea name="{{ $name}}" placeholder="{{ $placeholder }}" {!! $attributes !!} >{!! $value !!}</textarea>
        
        @include('admin::form.help-block')

    </div>
</div>

<script require="@ckeditor" init="{!! $selector !!}">
    $this.ckeditor();
</script>
```

Then in `app/Admin/bootstrap.php`, import the extension:
```php
use App\Admin\Extensions\Form\CKEditor;
use Dcat\Admin\Form;

// Register front-end component aliases
Admin::asset()->alias('@ckeditor', [
    'js' => [
        '/packages/ckeditor/ckeditor.js',
        '/packages/ckeditor/adapters/jquery.js',
    ],
]);

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

namespace App\Admin\Extensions\Form;

use Dcat\Admin\Form\Field;

class PHPEditor extends Field
{
    protected $view = 'admin.php-editor';

}

```

> {tip} Static resources in the class can also be brought in from outside, see [Editor.php](https://github.com/jqhph/dcat-admin/blob/master/src/Form/Field/Editor.php)

Creating View `resources/views/admin/php-editor.blade.php`:

```html
<div class="{{$viewClass['form-group']}}">

    <label class="{{$viewClass['label']}} control-label">{{$label}}</label>

    <div class="{{$viewClass['field']}}">

        @include('admin::form.error')

        <textarea class="{{ $class }}"   {!! $attributes !!} >{!! $value !!}</textarea>

        <input type="hidden" name="{{$name}}" value="{{ old($column, $value) }}" />

        @include('admin::form.help-block')
    </div>
</div>

<script require="@phpeditor" init="{!! $selector !!}">
    var Editor = CodeMirror.fromTextArea(document.getElementById(id), {
        lineNumbers: true,
        mode: "text/x-php",
        extraKeys: {
            "Tab": function(cm){
                cm.replaceSelection("    " , "end");
            }
        }
    });
    Editor.on("change", function (Editor, changes) {
        let val = Editor.getValue();
        //console.log(val);
        $this.parents('.form-field').find('input[name={{ $name }}]').val(val);
    });
</script>
```

Finally find the file `app/Admin/bootstrap.php`, if the file does not exist, please update `dcat-admin`, then create the file, add the following code:

```
<?php

use App\Admin\Extensions\Form\PHPEditor;
use Dcat\Admin\Form;

Admin::asset()->alias('@phpeditor', [
    'js' => [
        '/packages/codemirror-5.20.2/lib/codemirror.js',
        '/packages/codemirror-5.20.2/addon/edit/matchbrackets.js',
        '/packages/codemirror-5.20.2/mode/htmlmixed/htmlmixed.js',
        '/packages/codemirror-5.20.2/mode/xml/xml.js',
        '/packages/codemirror-5.20.2/mode/javascript/javascript.js',
        '/packages/codemirror-5.20.2/mode/css/css.js',
        '/packages/codemirror-5.20.2/mode/clike/clike.js',
        '/packages/codemirror-5.20.2/mode/php/php.js',
    ],
    'css' => '/packages/codemirror-5.20.2/lib/codemirror.css',
]);

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
<div class="{{$viewClass['form-group']}}">

    <div class="{{ $viewClass['label'] }} control-label">
        <span>{!! $label !!}</span>
    </div>

    <div class="{{$viewClass['field']}}">

        @include('admin::form.error')

        <input type="hidden" name="{{$name}}"/>

        <select style="width: 100%;" name="{{$name}}" {!! $attributes !!} >
           <option value=""></option>
		   @foreach($options as $select => $option)
			   <option value="{{$select}}" {{ Dcat\Admin\Support\Helper::equal($select, $value) ?'selected':'' }}>{{$option}}</option>
		   @endforeach
        </select>

        @include('admin::form.help-block')

    </div>
</div>
<script require="..." init="{!! $selector !!}">
    // where $selector is the css selector for the current field.
    $this.select2($configs);

```

### JS code

In order for the extended form component to be compatible with `HasMany`, `array` and `Table` forms, we must use the [dynamic listening element generation (init)](js.md#init) feature

```html
<div class="{{$viewClass['form-group']}}">
    ...
</div>

<script require="..." init="{!! $selector !!}">
    $this.select2($configs);
</script>
```

