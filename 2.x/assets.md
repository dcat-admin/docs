# 静态资源加载

`Dcat Admin`修改了`pjax`源码，增加了`js`脚本按需加载的功能，开发者只需在控制器`action`中引入需要用到的`js`组件即可，而无需在项目初始化时引入所有的`js`组件。

### 加载js脚本

`Admin::js`方法可以引入`js`脚本，使用如下：
```php
class UserController extend Controller
{
    public function index()
    {
        Admin::js('/assets/js/index.js');
        
        Admin::js([
            '/assets/js/index2.js'
        ]);
    }
}
```

### 加载css脚本

`Admin::css`方法可以引入`css`脚本，使用如下：
```php
class UserController extend Controller
{
    public function index()
    {
        Admin::css('/assets/css/index.css');
        
        Admin::css([
            '/assets/css/index2.css'
        ]);
    }
}
```

### 动态添加js代码

`Admin::script`方法可以动态添加`js`代码，使用如下：
```php
    public function index()
    {
        Admin::script(
            <<<JS
    console.log('Hello world!');
JS            
        );
    }
```

### 动态添加css代码

`Admin::style`方法可以动态添加`css`代码，使用如下：
```php
    public function index()
    {
        Admin::style(
            <<<CSS
    body {
        color: #333;
    }
CSS            
        );
    }
```

### 在模板中引入静态资源
在模板中手动引入静态资源需要使用`admin_asset`函数：

```html
// 引入css
<link rel="stylesheet" href="{{ admin_asset("vendor/dcat-admin/dcat-admin/main.min.css") }}">

// 引入js
<script src="{{ admin_asset('vendor/dcat-admin/dcat-admin/main.min.js')}}"></script>
```

### 在模板中添加js代码

要在模板中添加的`js`代码需要放在`Dcat.ready`方法内执行，这样才能保证你的`js`代码在所有`js`脚本加载完成之后执行。

```html
<script>
Dcat.ready(function () {
   console.log('所有js都加载完成了'); 
});
</script>
```

