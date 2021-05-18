# Data table actions

## GridAction Base Classes

All datagrid-related action classes, including [toolbar buttons](model-grid-custom-tools.md) (`AbstractTool`), the
[Row actions](model-grid-actions.md)(`RowAction`), [Batch actions](model-grid-custom-tools.md#batch)(`BatchAction`)
The base class of action buttons such as `Dcat\Admin\Grid\GridAction` class is inherited from `Dcat\Admin\GridAction` class, while `GridAction` inherits from [action class base class](action.md)(`Action`).

The methods or attributes added to the `GridAction` class are described below.

### Table Example (parent)

The table instance (`Dcat\AdminGrid`) can be retrieved through the `parent` attribute.


Here's a simple demonstration of how it's used:
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

### Table page address (resource)

The `resource` method is used to get the address of the form page.

The following is a simple demonstration of how this works.
```php
use Dcat\Admin\Grid\GridAction

class MyAction extend GridAction
{
    public function html()
    {
        // If the path to your listings page is /admin/users, the value here is http://domain/admin/users    
        $path = $this->resource();
        
        return ...;
    }
    
    ...
}
```

## Toolbar Action Button Base Class (AbstractTool)

The table toolbar button base class (`Dcat\Admin\GridTools\AbstractTool`) inherits from the `GridAction` class.

The following will introduce the methods or properties added in the `AbstractTool` class.

### Button style (style)

The `style` property allows you to set the class of the toolbar button (`class`), which defaults to `btn btn-white-white-wave-effect`.


Here's a simple demonstration
```php
use Dcat\Admin\Grid\AbstractTool

class MyTool extend AbstractTool
{
    protected $style = 'btn btn-outline-primary waves-effect';
    
    ...
}
```


## BatchAction

The table toolbar button base class (`Dcat\Admin\Tools\BatchAction`) inherits from the `GridAction` class.

The following describes the methods or attributes added in the `BatchAction` class.

### Get the primary key array for the selected row (getSelectedKeysScript)

The `getSelectedKeysScript` method can be used to generate `JS` code that gets the primary key array of the selected row.


Here's a simple demonstration

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


## RowAction base class (RowAction)

The table toolbar button base class (`Dcat\Admin\Tools\RowAction`) inherits from the `GridAction` class.

The following will introduce the methods or attributes added in the `RowAction` class.

### row data (row)

The `row` property is used to retrieve the contents of the current row.

Here's a simple demonstration of how it's used.
```php
use Dcat\Admin\Grid\RowAction

class MyRowAction extend RowAction
{
    public function html()
    {
        // Get the field value of the current row
        $username = $this->row->username;
        
        // to an array
        $rowArray = $this->row->toArray();
        
        return ...;
    }
    
    ...
}
```

### Primary key value (getKey)

The `getKey` property method is used to get the primary key (`ID`) of the current row data.

The following is a simple demonstration of this usage.
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
