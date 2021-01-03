# 静态资源加载

`Dcat Admin`支持了`js`脚本按需加载的功能，开发者只需在控制器中或其他任意位置（包括`view`）中引入需要用到的`js`组件即可，而无需在项目初始化时引入所有的`js`组件。


### 更改静态资源域名

打开配置文件`config/admin.php`，找到`assets_server`参数进行更改即可；也可以在`.env`中加上

```dotenv
ADMIN_ASSETS_SERVER=http://xxx.com
```

### 注册路径别名

打开`app/Admin/bootstrap.php`，然后加入以下代码

```php
Admin::asset()->alias('@my-name1', 'assets/admin1');
Admin::asset()->alias('@my-name2', 'assets/admin2');

// 也可以批量注册
Admin::asset()->alias([
	'@my-name1' => 'assets/admin1',
	'@my-name2' => 'assets/admin2',
]);
```

使用别名

```php
Admin::js('@my-name1/index.js');
Admin::css('@my-name1/index.css');
```

### 注册组件

当某个组件的`js`和`css`文件比较多的话，我们可以把这些静态资源文件统一注册成一个组件，这样使用的时候会更方便。打开`app/Admin/bootstrap.php`，然后加入以下代码

```php
Admin::asset()->alias('@editor-md', [
	'js' => [
		// 支持使用路径别名
		'@admin/dcat/plugins/editor-md/lib/raphael.min.js',
		'@admin/dcat/plugins/editor-md/lib/marked.min.js',
		'@admin/dcat/plugins/editor-md/lib/prettify.min.js',
		'@admin/dcat/plugins/editor-md/lib/jquery.flowchart.min.js',
		'@admin/dcat/plugins/editor-md/editormd.min.js',
	],
	'css' => [
		'@admin/dcat/plugins/editor-md/css/editormd.preview.min.css',
		'@admin/dcat/extra/markdown.css',
	],
]);
```

使用

```php
Admin::requireAssets(['@editor-md']);
```

如果你只需要加载这个组件的`js`或`css`，并不想加载所有文件，那么可以用以下方法

```php
// 只加载js文件
Admin::js('@editor-md');

// 只加载css文件
Admin::css('@editor-md');
```

使用动态参数

```php
use Dcat\Admin\Admin;

// 注册前端组件别名
// {lang} 为动态参数
Admin::asset()->alias('@test', [
    'js' => ['/vendor/test/js/{lang}.min.js'],
]);

// {lang} 会被替换为 zh_CN
Admin::requireAssets('@test', ['lang' => 'zh_CN']);
// 也可以这样使用
Admin::requireAssets('@test?lang=zh_CN');
```



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

