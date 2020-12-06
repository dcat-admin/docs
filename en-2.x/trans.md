# Multilingual

It is very convenient to use multi-language translation in `Dcat Admin`. The fields of data table, data form, data detail and model tree all support automatic read language package translation, which can be referred to [form field translation] (model-grid-trans.md), [form field translation] (model-form-trans.md), [data detail field-translation] (model-show-trans.md).


## Language package files

The file types of language packages are roughly as follows

```bash
resources/lang
    ├── ...
    └── en
        ├── admin.php   # System content language package, including menu TITLE translation, etc. are included
        ├── global.php  # Controller Common Language Package
        ├── {xxx}.php   # Controller language pack, one language pack for one controller.
        └── ...         
```


For example, if the controller name is `UserProfileController`, the corresponding language pack is `resources/lang/{current-language}/user-profile.php` (which needs to be converted to lower-case strikethrough style).



## Controller language package content format

The contents of the controller language package (including `global.php`) are divided into three categories.

- `fields` Translation of data fields, which is placed under this category
- `labels` Custom Content Translation, under this category is the translation of content outside of the data field, which can be any custom content
- `options` Translation of enumeration options

The following are examples:


Suppose that the controller language package `user-profile.php` reads as follows:
```php
<?php 

return [
    'labels' => [
        'UserProfile' => '用户中心',
        'list'        => '列表',
        
        'pagination' => [
            'range' => '从 :first 到 :last ，总共 :total 条',
        ],
    ],
    'fields' => [
        'name'      => '名称',
        'published' => '发布',
        'author'    => '作者',
        'status'    => '状态',
    ],
    'options' => [
        'status' => [
            0 => '未激活',
            1 => '已激活',
        ],
    ],
];
```

then language packs can be used like this

```php
class UserProfileController extend AdminController
{
    public function title()
    {
        // translates label UserProfile, which eventually translates to  “用户中心”
        return admin_trans_label('UserProfile');    
    }

    // fields and options translation Example
    public function grid()
    {
        $grid = new Grid(new UserProfile());
        
        // Displays the call language pack translation, where the "name" field is translated to "name"
        $grid->name(admin_trans_field('name'));
        
        // Implicitly using language pack translation, the "author" field is automatically translated to “作者”
        $grid->author;
        
        // Call options translation
        $grid->status()->using(admin_trans('user-profile.options.status'));
        
        return $grid;
    }
}
```



## use

### admin_trans_field
This function is used to translate the content under the category `fields`, and will automatically find the translation file under the corresponding controller, if the translation does not exist, it will find the translation in `global.php`.
```php
admin_trans_field('name');
```

### admin_trans_label
This function is used to translate the content under the category `labels`, and will automatically find the translation file under the corresponding controller, if the translation does not exist, it will find the translation in `global.php`.
```php
admin_trans_label('Posts');

admin_trans_label('pagination.range', ['first' => 1, 'last' => 1, 'total' => 0]);
```

### admin_trans_option
This function is used to translate the contents under the category `options`, it will automatically find the translation file under the corresponding controller, if the translation does not exist, it will find the translation in `global.php`.
```php
admin_trans_option(1, 'status');
```

### admin_trans
This method is the same as the `trans` method Usage that comes with the `Laravel` framework; the only difference is that when the translation is not found, it goes to `global.php` and looks for it again.
```php
// Go to user.php and look for the translation of first_name, if you can't find it, go to global.php and look for it.
admin_trans('user.first_name');
```

## Publicly translated documents
All common translations can be placed in `resources/lang/{current language}/global.php` and the public translation file will be read for translation when the controller translation file does not exist.

```php
<?php

return [
    'fields' => [
        'id'                    => 'ID',
        'name'                  => '名称',
        'username'              => '用户名',
        'email'                 => '邮箱',
        'password'              => '密码',
    ],
    'labels' => [
        'list'   => '列表',
        'edit'   => '编辑',
        'detail' => '详细',
        'create' => '创建',
        'root'   => '顶级',
        'scaffold' => '代码生成器',
    ],
    'options' => [
    ],
];
```


## Default breadcrumb translation

For example, if your access path is `/admin/my-users` and the controller is `MyUserController`, then you can add the following to the translation file corresponding to the controller

```php
return [
    'labels' => [
        'my-users' => '用户',
    ],
    ...
];
```


