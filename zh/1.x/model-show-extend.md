# 详情字段显示扩展

这个功能用来扩展详情字段显示, 在内置的显示方法不满足需求的情况下，可以使用这个功能来实现

首先定义扩展类：

```php
<?php

namespace App\Admin\Extensions\Show;

use Dcat\Admin\Show\AbstractField;

class UnSerialize extends AbstractField
{
    // 这个属性设置为false则不会转义HTML代码
    public $escape = false;
    
    public function render($arg = '')
    {
        // 返回任意可被渲染的内容
        return unserialize($this->value);
    }
}
```
然后在`app/Admin/bootstrap.php`中注册扩展类

```php
use Dcat\Admin\Show\Field;
use App\Admin\Extensions\Show\UnSerialize;

Field::extend('unserialize', UnSerialize::class);
```
然后在控制器中使用这个扩展

```php
$show->column()->unserialize('xxx');
```
传入unserialize()方法的参数会按顺序传入UnSerialize::render()方法中。

在父类`Dcat\Admin\Show\AbstractField`中可以看到几个常用的属性

```php
/**
 * Field value.
 *
 * @var mixed
 */
protected $value;

/**
 * Current field model.
 *
 * @var Fluent
 */
protected $model;

/**
 * If this field show with a border.
 *
 * @var bool
 */
public $border = true;

/**
 * If this field show escaped contents.
 *
 * @var bool
 */
public $escape = true;
```
其中`$value`和`$model`分别是当前字段值和当前详情内容的数据，在`render()`方法中可以用来获取你想要的数据。

`$border`用来控制当前显示内容是否需要外边框，`$escape`分别用来设置当前显示内容要不要HTML转义。