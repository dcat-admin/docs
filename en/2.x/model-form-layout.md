# Form layout

### Multi-column layout (column)

<a href="{{public}}/assets/img/screenshots/form-column.png" target="_blank">
    <img class="img" src="{{public}}/assets/img/screenshots/form-column.png" />
</a>   

Similar to the above diagram of the left and right two-column layout, you can refer to the following code to achieve the following

```php
// The first column takes up 1/2 of the page width.
$form->column(6, function (Form $form) {
    $form->text('name')->required();
    $form->date('born')->required();
    $form->select('education')->options([...])->required();
    $form->text('nation')->required();
    $form->text('native')->required();
    $form->text('job')->required();
    $form->text('code')->required();
    $form->text('phone')->required();
    $form->text('work')->required();
    $form->text('census')->required();
});

// The second column takes up 1/2 of the page width.
$form->column(6, function (Form $form) {
    $form->image('avatar');
    $form->decimal('wages');
    $form->decimal('fund');
    $form->decimal('pension');
    $form->decimal('fee');
    $form->decimal('business');
    $form->decimal('other');
    $form->text('area')->default(0);
    $form->textarea('illness');
    $form->textarea('comment');
});

// Adjust the width of all forms.
$form->width(9, 2);
```

The above layout functions use the `bootstrap` grid layout system, where the sum of the widths of all columns must not exceed `12`


### Multi-row layout (row)

use
```php
$form->row(function (Form\Row $form) {
    $form->width(4)->text('username')->required();
	$form->width(3)->text('title');
	...
});

$form->row(function (Form\Row $form) {
	...
});

...
```
result
<a href="{{public}}/assets/img/screenshots/form-rows.png" target="_blank">
    <img class="img img-full" src="{{public}}/assets/img/screenshots/form-rows.png">
</a>


<a name="tab"></a>
### Tab form (tab)

If the form has too many elements, the form page will be too long, in this case you can use the `tab` method to separate the form:

```php
$form->tab('Basic info', function (Form $form) {
    
    $form->text('username');
    $form->email('email');
    
})->tab('Profile', function (Form $form) {
                       
   $form->image('avatar');
   $form->text('address');
   $form->mobile('phone');
   
})->tab('Jobs', function (Form $form) {
                         
     $form->hasMany('jobs', function () {
         $form->text('company');
         $form->date('start_date');
         $form->date('end_date');
     });

  })
```

#### Fieldset Layout

```php
$form->fieldset('分组', function (Form $form) {
    $form->text('company');
    $form->date('start_date');
    $form->date('end_date');
});
```

If you want to hide by default

```php
$form->fieldset('分组', function (Form $form) {
    ...
})->collapsed();
```

result

<a href="{{public}}/assets/img/screenshots/form-fieldset.png" target="_blank">
    <img class="img img-full" src="{{public}}/assets/img/screenshots/form-fieldset.png">
</a>


### Block layout (block)

If you have a form with a lot of fields, you can make your form block layout by

```php
$form->display('id');
$form->text('name');
$form->text('title');
$form->text('author');
$form->textarea('content');

$form->datetimeRange('start', 'end', '开始');

// Set default card width
$form->setDefaultBlockWidth(8);

// Display in blocks
$form->block(4, function (Form\BlockForm $form) {
    $form->title('testing');

    $form->text('title')->default('test');
    $form->select('author')->options(['test']);
    $form->radio('state')->options([0 => 'pending', 1 => 'processed', 2 => 'declined'])->default(0);
});
```

result
<a href="{{public}}/assets/img/screenshots/block-form.png" target="_blank">
    <img class="img img-full" src="{{public}}/assets/img/screenshots/block-form.png">
</a>

