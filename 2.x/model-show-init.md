# 数据详情初始化

通过`Show::resolving`方法设置的回调函数会在`Dcat\Admin\Show`类被实例化时触发；

通过`Show::composing`方法设置的回调函数会在`render()`方法被调用时触发；

开发者可以在这两个事件中改变`Show`的一些设置或行为，比如需要禁用掉某些操作，可以在`app/Admin/bootstrap.php`加入下面的代码：

```php
use Dcat\Admin\Show;

Show::resolving(function (Show $show) {

     $show->showQuickEdit();
   
});
```
