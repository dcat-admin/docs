# 表单初始化

通过`Form::resolving`方法设置的回调函数会在`Dcat\Admin\Form`类被实例化时触发；

通过`Form::composing`方法设置的回调函数会在`render()`方法被调用时（渲染页面时）触发；

开发者可以在这两个事件中改变`Form`的一些设置或行为，比如需要禁用掉某些操作，可以在`app/Admin/bootstrap.php`加入下面的代码：

```php
use Dcat\Admin\Form;

Form::resolving(function (Form $form) {

     $form->disableEditingCheck();
    
     $form->disableCreatingCheck();
    
     $form->disableViewCheck();
    
     $form->tools(function (Form\Tools $tools) {
          $tools->disableDelete();
          $tools->disableView();
          $tools->disableList();
     });

});
```
这样就不用在每一个控制器的代码中来设置了。

如果全局设置后，要在其中某一个表单中开启设置，比如开启显示`继续编辑`的checkbox，在对应的实例上调用`$form->disableEditingCheck(false);`就可以了