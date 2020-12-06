# Table field translation

All the places in the data table where the fields are used will automatically read the translations from the language pack.

> {tip} See <a>[Multilingual](trans.md)</a> for details on how to use the language package.

### Language package name
If the controller is `UserProfileController`, the corresponding language package is `resources/lang/{current language}/user-profile.php` (needs to be converted to lower case strikethrough style).


### Example
Now suppose that the language package `resources/lang/zh_CN/user-profile.php` contains the following:
```php
return [
    'fields' => [
        'name'  => '名称',
        'age'   => '年龄',
        'class' => '班级',
    ],
];
```

The `Grid' field set in the controller `UserProfileController` will automatically read the above translation:
```php
// Don't set labael to read translations automatically.
$grid->id();
$grid->name;
$grid->age;
$grid->class;

$grid->filter(function ($filter) {
    $filter->gt('age');
});

// The above code is equivalent to
$grid->name('名称');
$grid->age('年龄');

// Can also be used like this
$grid->id(admin_trans_field('id'));
$grid->name(admin_trans_field('name'));
$grid->age(admin_trans_field('age'));

```

### Public interpretation
When the `admin_trans_field` function can't find the translation of a given field in the current controller, it looks for it in `global.php`. If some fields are present in many data tables, you can write those translations in the `resources/lang/{current language}/global.php` file.
```php
return [
    // Commonly used fields are placed in global.php and can be used by all controllers.
    'fields' => [
        'id'         => 'ID',
        'created_at' => '创建时间',
        'updated_at' => '更新时间',
    ],
];
```

