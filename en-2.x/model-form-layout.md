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

The above layout features use the `bootstrap` grid layout system, the sum of the width of all columns must not exceed `12`, and also supports the use of `hasMany` and `array` forms.

```php
$form->hasMany('jobs', function ($form) {
     $form->column(6, function (Form $form) {
         $form->text('name')->required();
         $form->date('born')->required();
     });
     
     $form->column(6, function (Form $form) {
         $form->image('avatar');
         $form->decimal('wages');
     });
});
```



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

It is also supported in `hasMany` and `array` forms.

```php
$form->hasMany('jobs', function ($form) {
     $form->row(function (Form\Row $form) {
     	...
     });
     
     $form->row(function (Form\Row $form) {
     	...
     });
});
```



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
                         
     $form->hasMany('jobs', function ($form) {
         $form->text('company');
         $form->date('start_date');
         $form->date('end_date');
     });

  });
```

The `tab` layout also allows nested use of `column` and `row` layouts.

```php
$form->tab('Basic info', function (Form $form) {
    $form->column(6, function (Form\BlockForm $form) {
		$form->display('id');
		$form->text('name');
	});

	$form->column(6, function (Form\BlockForm $form) {
		$form->text('username');
	});
})
```


Set the default display of ``Tab``

```php
// Show the Tab with the title Title 2 by default
$form->getTab()->active('Title2');
// You can also specify an offset
$form->getTab()->activeByIndex(1);

$form->tab('Heading 1', function ($form) {
    ...
});

$form->tab('Title2', function ($form) {
    ...
});
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

![](https://cdn.learnku.com/uploads/images/202005/12/38389/B0tXWUxHDp.png!large)



### Block layout (block)

If your form has a lot of fields, you can block your form and allow nested `column` and `row` layouts by doing the following

```php
$form->block(8, function (Form\BlockForm $form) {
	// Set the title
    $form->title('Basic settings');

    // Show bottom submit button    
    $form->showFooter();
    
    // Set the field width
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
    $form->title('Block 2');

    $form->text('nickname');
    $form->number('age');
    $form->radio('status')->options(['1' => 'default', 2 => 'freeze'])->default(1);

    $form->next(function (Form\BlockForm $form) {
        $form->title('Block 3');
        $form->date('birthday');
        $form->date('created_at');
    });

});
```

result
<a href="https://cdn.learnku.com/uploads/images/202010/19/38389/AMCtHBcmSQ.jpg!large" target="_blank">
    <img class="img img-full" src="https://cdn.learnku.com/uploads/images/202010/19/38389/AMCtHBcmSQ.jpg!large">
</a>

