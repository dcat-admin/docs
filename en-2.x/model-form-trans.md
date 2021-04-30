# Form field translation

All the places in the data form form where the fields are used will automatically read the translations from the language pack.

> {tip} See <a>[Multilingual](trans.md)</a> for details on how to use the language package.

### Language package name
If the controller is `UserProfileController`, the corresponding language package is `resources/lang/{current language}/user-profile.php` (needs to be converted to lower case strikethrough style).


If you want to change the name of the language pack, you can do so in the following two ways

Method 1
```php
use Dcat\Admin\Http\Controllers\AdminController;

class UserController extends AdminController
{
    /**
     * Specify the translation file name
     * 
     * @var string 
     */
    protected $translation = 'user1';
    
    ...
}
```

Method 2
```php
use Dcat\Admin\Admin;

Admin::translation('user1');
```


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

The `Form` field set in the controller `UserProfileController` will automatically read the above translation:
```php
// If you don't set labels, language package translations are used automatically
$form->display('id');
$form->text('name');
$form->text('age');
$form->text('class');
```

### Public interpretation
When the `admin_trans_field` function can't find the translation of a given field in the current controller, it looks for it in `global.php`. If some fields are present in many data tables, you can write those translations in the `resources/lang/{current language}/global.php` file.
```php
return [
    // Commonly used fields are placed in global.php and can be used by all controllers
    'fields' => [
        'id'         => 'ID',
        'created_at' => '创建时间',
        'updated_at' => '更新时间',
    ],
];
```

