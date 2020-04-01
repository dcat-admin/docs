# JS组件

`Dcat Admin`内置了一些常用的JS功能组件，通过全局变量`Dcat`可以访问到这些功能方法。

### 监听JS脚本加载完毕事件

通过`Dcat.ready`方法设置的回调函数会在所有的`JS`脚本都加载完毕后执行。

> {tip} 只有在模板文件中写`JS`代码才需要使用`Dcat.ready`，当在`php`代码中使用`Dcat\Admin\Admin::script`方法添加`JS`代码时是不需要使用`Dcat.ready`方法的。因为在构建页面的时候系统会自动把代码放在`Dcat.ready`事件内执行。

```html
<div>...</div>
<script>
Dcat.ready(function () {
    // 写你的逻辑
    
    console.log('所有JS脚本都加载完了');
});
</script>
```

### 手动触发JS脚本加载完毕事件

通过`Dcat.triggerReady`方法可以手动触发`JS`脚本加载完毕事件，这就意味着会自动执行在此之前所有通过`Dcat.ready`方法设置的回调函数。

> {tip} 这个功能普通开发很少会用到，只有一些比较深度的组件定制会用到，比如[表单弹窗](model-form-modal.md)功能就用到了此方法。

```js
Dcat.triggerReady();
```


### 标