# Editing within data table rows

All column fields in a data table that use in-row editing must have an identical form field defined in the `form` form or they will not be editable.


### Text (editable)

enable
```php
$grid->column('title')->editable();

// Refresh the page after editing successfully
$grid->column('nickname')->editable(true);
```

result
![](https://cdn.learnku.com/uploads/images/202109/14/38389/mX4Za4nj1y.png!large)


### Switch (switch)


Quickly turn columns into switch components, using the following methods:
```php
$grid->column('status')->switch();

// Set Color
use Dcat\Admin\Admin;

$grid->column('status')->switch(Admin::color()->info());
$grid->column('status')->switch('#333');

```
This feature requires you to set a `status` field in the `form` form method as well

```php
$form->hidden('status')
	->customFormat(function ($v) {
		return $v == '打开' ? 1 : 0;
	})
	->saving(function ($v) {
		return $v ? '打开' : '关闭';
	});

// or
$form->switch('status')
	->customFormat(function ($v) {
		return $v == '打开' ? 1 : 0;
	})
	->saving(function ($v) {
		return $v ? '打开' : '关闭';
	});
```


// Refresh the page after editing successfully
```php
$grid->column('status')->switch('', true);
```


### switchGroup (switchGroup)

> {tip} Note: The default result of setting `switchGroup` in `grid` is `0` or `1`, if you want to change it, you can use `$form->hidden(xxx)->saving(...) ` method.

Quickly turn columns into groups of switch components, using the following methods:
```php
$grid->switch_group->switchGroup([
    'hot'        => 'popular',
    'new'        => 'latest',
    'recommend'  => 'recommended',
    'image.show' => 'Show pictures', // Updating the related models
]);
// or
// Without writing a label it is automatically translated from a translated file, please refer to the section "Field translation" for details.
$grid->switch_group->switchGroup(['is_new', 'is_hot', 'published']);
```

This requires you to set the corresponding field in the `form` form method as well

```php
$form->switch('hot')
	->customFormat(function ($v) {
		return $v == '打开' ? 1 : 0;
	})
	->saving(function ($v) {
		return $v ? '打开' : '关闭';
	});

$form->switch('new')
	->customFormat(function ($v) {
		return $v == '打开' ? 1 : 0;
	})
	->saving(function ($v) {
		return $v ? '打开' : '关闭';
	});
```



Refresh the page after editing success
```php
$grid->column('switch_group')->switchGroup([...], true);
```


![]({{public}}/assets/img/screenshots/grid-column-switch-group.png)


### Dropdown box (select)

```php
$grid->column('options')->select([
    1 => 'Sed ut perspiciatis unde omni',
    2 => 'voluptatem accusantium doloremque',
    3 => 'dicta sunt explicabo',
    4 => 'laudantium, totam rem aperiam',
]);
```

`select` also supports closures and is used in a similar way to `select` of `editable`.



Refresh the page after editing success
```php
$grid->column('options')->select([...], true);
```


![]({{public}}/assets/img/screenshots/grid-column-select.png)

### radio box (radio)
```php
$grid->column('options')->radio([
    1 => 'Sed ut perspiciatis unde omni',
    2 => 'voluptatem accusantium doloremque',
    3 => 'dicta sunt explicabo',
    4 => 'laudantium, totam rem aperiam',
]);
```

`radio` also supports closures, similar to `select` of `editable`.


Refresh the page after editing success
```php
$grid->column('options')->radio([...], true);
```

![](https://cdn.learnku.com/uploads/images/202109/14/38389/6Bo4phkB3f.png!large)

### checkboxes (checkbox)
```php
$grid->column('options')->checkbox([
    1 => 'Sed ut perspiciatis unde omni',
    2 => 'voluptatem accusantium doloremque',
    3 => 'dicta sunt explicabo',
    4 => 'laudantium, totam rem aperiam',
]);
```

`checkbox` also supports closures with parameters similar to `select` of `editable`.



Refresh the page after editing success
```php
$grid->column('options')->checkbox([...], true);
```

![]({{public}}/assets/img/screenshots/grid-column-checkbox.png)


### textarea

```php
$grid->column('...')->textarea();
```

![](https://cdn.learnku.com/uploads/images/202109/14/38389/wViO5EoPBg.png!large)
