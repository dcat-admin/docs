# 数据表格动作

## 表格动作基类 (GridAction)

所有数据表格相关的动作类，包括[工具栏按钮](model-grid-custom-tools.md)(`AbstractTool`)、
[行操作](model-grid-actions.md)(`RowAction`)、[批量操作](model-grid-custom-tools.md#batch)(`BatchAction`)
等等操作按钮的基类都继承自`Dcat\Admin\Grid\GridAction`类，而`GridAction`则继承自[动作类基类](action.md)(`Action`)。

下面将介绍`GridAction`类中增加的方法或属性

### 表格实例 (parent)

通过 `parent` 属性可以获取到表格实例 (`Dcat\Admin\Grid`)。


下面简单的演示下用法，这段代码没有任何实际意义
```php
use Dcat\Admin\Grid\GridAction

class MyAction extend GridAction
{
    public function html()
    {
        $gridName = $this->parent->getName();
        
        return ...;
    }
    
    ...
}
```

### 表格页面地址 (resource)

通过 `resource` 方法可以获取到表格页面的地址。

下面简单的演示下用法，这段代码没有任何实际意义
```php
use Dcat\Admin\Grid\GridAction

class MyAction extend GridAction
{
    public function html()
    {
        // 假如你的列表页路径为 /admin/users，则这里的值为 http://域名/admin/users    
        $path = $this->resource();
        
        return ...;
    }
    
    ...
}
```

## 工具栏操作按钮基类 (AbstractTool)

表格工具栏按钮基类(`Dcat\Admin\Grid\Tools\AbstractTool`)继承自`GridAction`类。

下面将介绍`AbstractTool`类中增加的方法或属性

### 按钮样式 (style)

通过 `style` 属性可以设置工具栏按钮的类(`class`)，默认值为 `btn btn-white waves-effect`。


下面简单的演示下用法
```php
use Dcat\Admin\Grid\AbstractTool

class MyTool extend AbstractTool
{
    protected $style = 'btn btn-outline-primary waves-effect';
    
    ...
}
```


## 批量操作基类 (BatchAction)

表格工具栏按钮基类(`Dcat\Admin\Grid\Tools\BatchAction`)继承自`GridAction`类。

下面将介绍`BatchAction`类中增加的方法或属性

### 获取选中行的主键数组 (getSelectedKeysScript)

通过 `getSelectedKeysScript` 方法可以生成获取选中的行的主键数组的`JS`代码。


下面简单的演示下用法
```php
use Dcat\Admin\Grid\BatchAction

class MyBatchAction extend BatchAction
{
    /**
     * {@inheritdoc}
     */
    public function actionScript()
    {
        $warning = __('No data selected!');

        return <<<JS
    var key = {$this->getSelectedKeysScript()}
    
    if (key.length === 0) {
        Dcat.warning('{$warning}');
        return ;
    }
    Object.assign(data, {_key:key});
JS;
    }
    
    ...
}
```


## 行操作基类 (RowAction)

表格工具栏按钮基类(`Dcat\Admin\Grid\Tools\RowAction`)继承自`GridAction`类。

下面将介绍`RowAction`类中增加的方法或属性

### 行数据 (row)

通过 `row` 属性可以获取到当前行数据内容。

下面简单的演示下用法，这段代码没有任何实际意义
```php
use Dcat\Admin\Grid\RowAction

class MyRowAction extend RowAction
{
    public function html()
    {
        // 获取当前行的字段值
        $username = $this->row->username;
        
        // 转化为数组
        $rowArray = $this->row->toArray();
        
        return ...;
    }
    
    ...
}
```

### 主键值 (getKey)

通过 `getKey` 属性方法可以获取到当前行数据的主键值(`ID`)。

下面简单的演示下用法，这段代码没有任何实际意义
```php
use Dcat\Admin\Grid\RowAction

class MyRowAction extend RowAction
{
    public function html()
    {
        $id = $this->getKey();
        
        return ...;
    }
    
    ...
}
```
