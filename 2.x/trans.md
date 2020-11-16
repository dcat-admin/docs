# 多语言

在`Dcat Admin`中使用多语言翻译非常方便，数据表格、数据表单、数据详情和模型树的字段都支持自动读取语言包翻译，具体可参考[表格字段翻译](model-grid-trans.md)、[表单字段翻译](model-form-trans.md)、[数据详情字段翻译](model-show-trans.md)。


## 语言包文件

语言包文件类型大概如下

```bash
resources/lang
    ├── ...
    └── en
        ├── admin.php   # 系统内容语言包，包括菜单标题翻译等都在里面
        ├── global.php  # 控制器公共语言包
        ├── {xxx}.php   # 控制器语言包，一个控制器对应一个语言包
        └── ...         
```


控制器语言包名称需要与控制器名相对应，例如控制器名`UserProfileController`，则对应的语言包为`resources/lang/{当前语言}/user-profile.php`（需要转化为小写中划线风格）。



## 控制器语言包内容格式

控制器语言包（包括`global.php`）中的内容分为三个类别：

- `fields` 数据字段的翻译，这个类别下放置数据字段的翻译
- `labels` 自定义内容翻译，这个类别下是对数据字段外的内容翻译，可以是任何自定义内容
- `options` 枚举选项翻译

下面是例子：


假设控制器语言包`user-profile.php`内容如下：
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

则语言包可以这么使用

```php
class UserProfileController extend AdminController
{
    public function title()
    {
        // labels翻译示例，最终翻译成 “用户中心”
        return admin_trans_label('UserProfile');    
    }

    // fields和options翻译示例
    public function grid()
    {
        $grid = new Grid(new UserProfile());
        
        // 显示调用语言包翻译，这里会把 “name” 字段翻译成 “名称”
        $grid->name(admin_trans_field('name'));
        
        // 隐式使用语言包翻译，“author” 字段会自动翻译成 “作者”
        $grid->author;
        
        // 调用options翻译
        $grid->status()->using(admin_trans('user-profile.options.status'));
        
        return $grid;
    }
}
```



## 使用

### admin_trans_field
这个函数用于翻译`fields`类别下内容，会自动找对应控制器下的翻译文件，如果翻译不存在会去找`global.php`中的翻译。
```php
admin_trans_field('name');
```

### admin_trans_label
这个函数用于翻译`labels`类别下内容，会自动找对应控制器下的翻译文件，如果翻译不存在会去找`global.php`中的翻译。
```php
admin_trans_label('Posts');

admin_trans_label('pagination.range', ['first' => 1, 'last' => 1, 'total' => 0]);
```

### admin_trans_option
这个函数用于翻译`options`类别下内容，会自动找对应控制器下的翻译文件，如果翻译不存在会去找`global.php`中的翻译。
```php
admin_trans_option(1, 'status');
```

### admin_trans
此方法与`Laravel`框架自带的`trans`方法用法没有区别，唯一的区别是：当翻译的内容找不到时，会去`global.php`中再找一次。
```php
// 先去 user.php 中找 first_name 的翻译，如果找不到会去 global.php 中找
admin_trans('user.first_name');
```

## 公共翻译文件
所有常用的翻译都可以放在`resources/lang/{当前语言}/global.php`中，当控制器翻译文件不存在时会读取公共翻译文件翻译。

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


## 默认面包屑翻译

例如你的访问路径是`/admin/my-users`，控制器是`MyUserController`，那么则可以在控制器对应的翻译文件中加上

```php
return [
    'labels' => [
        'my-users' => '用户',
    ],
    ...
];
```


