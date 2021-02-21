# 主题与颜色

### 切换主题

`Dcat Admin`支持主题切换功能，目前内置了四种主题色：`indigo`、`blue`、`blue-light`、`green`，可通过配置参数`admin.layout.color`进行切换。

打开配置文件`config/admin.php`
```php
     'layout' => [
         'color' => 'blue',
         
         ...
     ],
     
     ...
```

部分主题色预览

<a href="{{public}}/assets/img/screenshots/users-blue-dark.png" target="_blank">
    <img  src="{{public}}/assets/img/screenshots/users-blue-dark.png" style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%">
</a>
<a href="{{public}}/assets/img/screenshots/users-green.png" target="_blank">
    <img  src="{{public}}/assets/img/screenshots/users-green.png" style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%">
</a>


<a name="custom"></a>
### 自定义主题配色

> {tip} 需要注意的是，如果自定义了主题之后，每次更新新版本，都需要重新编译一次你的自定义主题！！！

开发者可以通过这个功能随意添加自己想要的主题配色，在使用这个功能之前需要先安装[NodeJs](http://nodejs.cn/)，没安装的同学前往[http://nodejs.cn/](http://nodejs.cn/)下载安装即可。

安装完`NodeJs`之后可打开命令行运行`npm -v`测试一下是否安装成功。

```bash
npm -v
```

如果正常返回版本号，则说明已安装成功，同时建议使用淘宝镜像
```bash
npm config set registry https://registry.npm.taobao.org
```

然后运行以下命令编译自定义主题的文件，只需输入主题的名称和主题颜色代码(`十六进制`)即可。
这里我们以生成一个`orange`主题为例

> {tip} 这个命令第一次运行时需要较长时间，请耐心等待。如果运行失败，请尝试给`vendor`目录写权限。

```bash
php artisan admin:minify orange --color fbbd08 --publish
```

上面的命令的意思是生成一个`orange`主题，颜色代码为`#fbbd08`，并且生成之后自动发布静态资源。如果编译成功，命令行会输出以下内容
```bash
...

 DONE  Compiled successfully in 48001ms8:24:28 PM


                                              Asset      Size  Chunks
               Chunk Names
               /resources/dist/adminlte/adminlte.js  29.7 KiB       0  [emitted]
               /resources/dist/adminlte/adminlte
           /resources/dist/adminlte/adminlte.js.map  87.8 KiB       0  [emitted]
 [dev]         /resources/dist/adminlte/adminlte
               /resources/dist/dcat/extra/action.js   3.7 KiB       1  [emitted]
               /resources/dist/dcat/extra/action
           /resources/dist/dcat/extra/action.js.map  12.9 KiB       1  [emitted]
 [dev]         /resources/dist/dcat/extra/action
          /resources/dist/dcat/extra/grid-extend.js  4.87 KiB       2  [emitted]
               /resources/dist/dcat/extra/grid-extend
      /resources/dist/dcat/extra/grid-extend.js.map  21.7 KiB       2  [emitted]
 [dev]         /resources/dist/dcat/extra/grid-extend
    /resources/dist/dcat/extra/resource-selector.js   5.8 KiB       3  [emitted]
               /resources/dist/dcat/extra/resource-selector
/resources/dist/dcat/extra/resource-selector.js.map    24 KiB       3  [emitted]
 [dev]         /resources/dist/dcat/extra/resource-selector
               /resources/dist/dcat/extra/upload.js  17.2 KiB       4  [emitted]
               /resources/dist/dcat/extra/upload
           /resources/dist/dcat/extra/upload.js.map  66.8 KiB       4  [emitted]
 [dev]         /resources/dist/dcat/extra/upload
                /resources/dist/dcat/js/dcat-app.js  88.8 KiB       5  [emitted]
               /resources/dist/dcat/js/dcat-app
            /resources/dist/dcat/js/dcat-app.js.map   164 KiB       5  [emitted]
 [dev]         /resources/dist/dcat/js/dcat-app
        resources/dist/adminlte/adminlte-orange.css   656 KiB       0  [emitted]
        [big]  /resources/dist/adminlte/adminlte
        resources/dist/dcat/css/dcat-app-orange.css    43 KiB       0  [emitted]
               /resources/dist/adminlte/adminlte
      resources/dist/dcat/extra/markdown-orange.css  1.72 KiB       0  [emitted]
               /resources/dist/adminlte/adminlte
          resources/dist/dcat/extra/step-orange.css  8.56 KiB       0  [emitted]
               /resources/dist/adminlte/adminlte
        resources/dist/dcat/extra/upload-orange.css  6.42 KiB       0  [emitted]
               /resources/dist/adminlte/adminlte
               

Copied Directory [\dcat-admin\resources\dist] To [\public\vendors\dcat-admin]
Publishing complete.
Compiled views cleared!
```

主题文件编译成功之后，还需要在`app/Admin/bootstrap.php`中加入以下代码

```php
Dcat\Admin\Color::extend('orange', [
    'primary'        => '#fbbd08',
    'primary-darker' => '#fbbd08',
    'link'           => '#fbbd08',
]);
```

最后把你的配置参数`admin.layout.color`的值设置为`orange`就行了。


<a name="darkmode"></a>
### 深色模式

![]({{public}}/assets/img/screenshots/users-dark.png)


#### 启用切换按钮

通过配置参数 `admin.layout.dark_mode_switch` 可以启用或禁用深色模式切换开关。开启后会在页面顶部导航栏中增加一个开关按钮，点击可以切换深色和明亮模式，并且会把状态保存在`localStorage`中。

```php
     'layout' => [
         'dark_mode_switch' => true,
         
         ...
     ],
     
     ...
```

效果如下
![]({{public}}/assets/img/screenshots/dark-switch.gif)

#### 默认深色

打开配置文件`config/admin.php`，写入
```php
     'layout' => [
         'body_class' => 'dark-mode',
         
         ...
     ],
     
     ...
```

<a name="sidebar"></a>
### 菜单样式

通过配置参数 `admin.layout.sidebar_style` 可以配置菜单样式（如果配置文件中不存在这个参数则可以手动添加），支持三个值 `light`、`primary`、`dark`，默认为 `light`

> {tip} `sidebar_dark`参数即将被废弃！`sidebar_style`参数会覆盖`sidebar_dark`参数，只有当`sidebar_style`不存在时`sidebar_dark`才会生效！！！

```php
     'layout' => [
     	 // 支持 light、primary、dark
         'sidebar_style' => 'light',
         
         ...
     ],
     
     ...
```

`light` 效果

![]({{public}}/assets/img/users.jpg)


`primary` 效果

![]({{public}}/assets/img/users-menu-primary.jpg)

![]({{public}}/assets/img/users-green-menu-primary.jpg)


### 菜单布局

#### 顶部横向 (Horizontal)

设置配置参数 `admin.layout.horizontal_menu` 的值为 `true` 开启此功能，效果如下

![](https://cdn.learnku.com/uploads/images/202102/20/38389/SpmXMujJ3D.png!large)



#### sidebar-separate

添加 `sidebar-separate` 到 `admin.layout.body_class` 参数中

```php
     'layout' => [
         'body_class' => ['sidebar-separate'],
         
         ...
     ],
     
     ...

```

效果

![]({{public}}/assets/img/users-2.jpg)
![]({{public}}/assets/img/users-dark-2.jpg)


<a name="color"></a>
### PHP颜色管理

`Dcat Admin`内置了颜色管理模块，此功能可以很方便的配合主题切换功能，让页面颜色与主题色相适应！

通过 `Dcat\Admin\Admin::color()` 这个服务可以很轻松的获取常用颜色（可参考[颜色表与样式](theme.md#颜色表与样式)）。

#### 获取内置颜色

通过`Color::get`或魔术方法可以获取颜色代码，当通过`Color::get`获取的颜色不存在时，会返回参数的原始值。

```php
<?php
use Dcat\Admin\Admin;

// get 方法获取颜色
echo Admin::color()->get('primary'); // 输出 #5c6bc6


// 通过魔术方法获取颜色
echo Admin::color()->primary(); // 输出 #5c6bc6
``` 

#### 颜色淡化

通过`Color::lighten`方法或魔术方法可以获取淡化后的颜色的16进制颜色代码。

`Color::lighten`方法接收两个参数：

- `$name` `string` 颜色别名
- `$amt` `int` 颜色偏差值，值越大颜色越`淡`

```php
echo Admin::color()->lighten('primary', 10); // 输出 #6675d0

// 也可以这样使用，注意这里的参数要传负数
echo Admin::color()->primary(-10); // 输出 #6675d0
```

也支持直接传颜色代码

```php
echo Admin::color()->lighten('#5c6bc6', 10); // 输出 #6675d0
```

#### 颜色深化

通过`Color::darken`方法或魔术方法可以获取深化后的颜色的16进制颜色代码。

`Color::darken`方法接收两个参数：

- `$name` `string` 颜色别名
- `$amt` `int` 颜色偏差值，值越大颜色越`深`

```php
echo Admin::color()->darken('primary', 10); // 输出 #5261bc

// 也可以这样使用
echo Admin::color()->primary(10); // 输出 #5261bc
```

也支持直接传颜色代码

```php
echo Admin::color()->darken('#5c6bc6', 10); // 输出 #5261bc
```

#### 颜色透明化
通过`Color::alpha`方法可以设置颜色的透明度。

`Color::alpha`方法接收两个参数：

- `$name` `string` 颜色别名
- `$alpha` `float` 透明度，`0 ~ 1`之间的值，值越小透明度越高

```php
echo Admin::color()->alpha('primary', 0.1); // 输出 rgba(92, 107, 198, 0.1)
```

也支持直接传颜色代码

```php
echo Admin::color()->alpha('5c6bc6', 0.1); // 输出 rgba(92, 107, 198, 0.1)
```


#### 获取所有内置颜色

通过`Color::all`方法可以获取所有内置颜色的16进制代码，此方法返回一个数组

```php
$allColors = Admin::color()->all();
```


### JS颜色管理

`JS`模块中同样也包含颜色管理功能，通过`Dcat.color`对象可以像在PHP代码中一样管理颜色。

#### 获取内置颜色

在`JS`代码中可以通过以下三种方式获取颜色代码
```php
Admin::script(
<<<JS
	// 方式1
	var primary = Dcat.color.primary;
	
	// 方式2
	var primary = Dcat.color['primary'];
	
	// 方式3
	var primary = Dcat.color.get('primary');
	
	console.log(primary);  // 打印 #5c6bc6
JS
);
```

#### 颜色淡化

通过`Dcat.color.lighten`方法或魔术方法可以获取淡化后的颜色的16进制颜色代码。

`color.lighten`方法接收两个参数：

- `name` `string` 颜色别名
- `amt` `int` 颜色偏差值，值越大颜色越`淡`

```php
Admin::script(
    <<<JS
    var primary = Dcat.color.lighten('primary', 10)
    
    console.log(primary); // 输出 #6675d0
JS    
);
```

也支持直接传颜色代码

```js
var primary = Dcat.color.lighten('5c6bc6', 10);
    
console.log(primary); // 输出 #6675d0
```

#### 颜色深化

通过`Dcat.color.darken`方法或魔术方法可以获取深化后的颜色的16进制颜色代码。

`color.darken`方法接收两个参数：

- `name` `string` 颜色别名
- `amt` `int` 颜色偏差值，值越大颜色越`深`

```php
Admin::script(
    <<<JS
    var primary = Dcat.color.darken('primary', 10)
    
    console.log(primary); // 输出 #5261bc
JS    
);
```

也支持直接传颜色代码

```php
var primary = Dcat.color.darken('5c6bc6', 10)

console.log(primary); // 输出 #5261bc
```

#### 颜色透明化
通过`Dcat.color.alpha`方法可以设置颜色的透明度。

`color.alpha`方法接收两个参数：

- `$name` `string` 颜色别名
- `$alpha` `float` 透明度，`0 ~ 1`之间的值，值越小透明度越高

```php
Admin::script(
    <<<JS
    var primary = Dcat.color.alpha('primary', 0.1)
    
    console.log(primary); // 输出 rgba(92, 107, 198, 0.1)
JS    
);
```

也支持直接传颜色代码

```php
var primary = Dcat.color.alpha('#5c6bc6', 0.1)
    
console.log(primary); // 输出 rgba(92, 107, 198, 0.1)
```


#### 获取所有内置颜色

通过`Dcat.color.all`方法可以获取所有内置颜色的16进制代码，此方法返回一个键值对对象。

```js
var allColors = Dcat.color.all();
```


<a name="颜色表与样式"></a>
### 颜色表与样式

`Dcat Admin`前端是采用`bootstrap4`编写的，因此首先要学习[Bootstrap4 颜色(Color)样式](https://getbootstrap.net/docs/utilities/colors/)的使用，这里不再赘述相关内容。

以下是`Dcat Admin`中常用颜色样式表，其中以`.bg-*` 开头的样式是背景色，以`.text-` 开头的样式是文本颜色


<style>
	.color-sections {

	}
	.color-section {
		width: 600px;
		height: 60px;
		line-height: 60px;
		text-align: center;
		vertical-align: middle;
		/*display: inline-block;*/
		margin-bottom: 5px;
	}

	.white {
		color: #fff;
	}
</style>
 <section class="container color-sections" style="min-height: 500px">
        <div class="color-section white" style="background: #5c6bc6">
            <code>.text-primary</code> <code>.bg-primary</code> primary/indigo
        </div>
        <div class="color-section white" style="background: #495abf">
            <code>.text-primary-darker</code> indigo-darker
        </div>

        <div class="color-section white" style="background: #5b69bc">
            purple
        </div>

        <div class="color-section white" style="background: #7367f0">
            cyan
        </div>
        <div class="color-section white" style="background: #6355ee">
            cyan-darker
        </div>

        <div class="color-section white" style="background: #3085d6">
            <code>.text-info</code> <code>.bg-info</code> blue/info
        </div>
        <div class="color-section white" style="background: #236bb0">
            <code>.text-blue-darker</code> blue-darker
        </div>
        <div class="color-section white" style="background: #007ee5">
            <code>.text-blue-1</code> <code>.bg-blue-1</code> blue1
        </div>
        <div class="color-section white" style="background: #4199de">
            <code>.text-blue-2</code> <code>.bg-blue-2</code> blue2
        </div>

        <div class="color-section white" style="background: #59a9f8">
            <code>.text-custom</code> <code>.bg-custom</code> custom
        </div>

        <div class="color-section white" style="background: #21b978">
            <code>.text-success</code> <code>.bg-success</code> green/success
        </div>
        <div class="color-section white" style="background: #ea5455">
            <code>.text-danger</code> <code>.bg-danger</code> danger/red
        </div>
        <div class="color-section white" style="background: #bd4147">
            <code>.text-danger-darker</code> red-darker
        </div>


        <div class="color-section white" style="background: #dda451">
            <code>.text-warning</code> <code>.bg-warning</code> warning/orange
        </div>


        <div class="color-section white" style="background: #ffcc80">
            <code>.text-orange-1</code> <code>.bg-orange-1</code> orange1
        </div>
        <div class="color-section white" style="background: #F99037">
            <code>.text-orange-2</code> <code>.bg-orange-2</code> orange2
        </div>
        <div class="color-section white" style="background: #edc30e">
            <code>.text-yellow</code> <code>.bg-yellow</code> yellow
        </div>

        <div class="color-section white" style="background: #ff8acc">
            <code>.text-pink</code> <code>.bg-pink</code> pink
        </div>


        <div class="color-section white" style="background: #01847f">
            <code>.text-tear</code> <code>.bg-tear</code> tear
        </div>
        <div class="color-section white" style="background: #00b5b5">
            <code>.text-tear-1</code> <code>.bg-tear-1</code> tear1
        </div>


        <div class="color-section white" style="background: #22292f">
            dark
        </div>

        <div class="color-section white" style="background: #b9c3cd">
            <code>.text-gray</code> <code>.bg-gray</code> gray
        </div>
        <div class="color-section white" style="background: #f7f7f9;color: #666">
            <code>.text-light</code> <code>.bg-light</code> light
        </div>
        <div class="color-section white" style="background: #f6fbff;color: #666">
            <code>.text-dark20</code> <code>.bg-dark20</code> dark20
        </div>
        <div class="color-section white" style="background: #f4f7fa;color: #666">
            <code>.text-dark30</code> <code>.bg-dark30</code> dark30
        </div>
        <div class="color-section white" style="background: #e7eef7;color: #666">
            <code>.text-dark35</code> <code>.bg-dark35</code> dark35
        </div>

        <div class="color-section white" style="background: #ebf0f3;color: #666">
            <code>.text-dark40</code> <code>.bg-dark40</code> dark40
        </div>
        <div class="color-section white" style="background: #d3dde5;color: #666">
            <code>.text-dark50</code> <code>.bg-dark50</code> dark50
        </div>
        <div class="color-section white" style="background: #bacad6">
            <code>.text-dark60</code> <code>.bg-dark60</code> dark60
        </div>
        <div class="color-section white" style="background: #b3b9bf">
            <code>.text-dark70</code> <code>.bg-dark70</code> dark70
        </div>
        <div class="color-section white" style="background: #7c858e">
            <code>.text-dark80</code> <code>.bg-dark80</code> dark80
        </div>
        <div class="color-section white" style="background: #5c7089">
            <code>.text-dark85</code> <code>.bg-dark85</code> dark85
        </div>
        <div class="color-section white" style="background: #252d37">
            dark90
        </div>
        <div class="color-section white" style="background: #414750">
            font字体颜色
        </div>
        <div class="color-section white" style="background: #f1f1f1;color: #666">
            gray-bg
        </div>
        <div class="color-section white" style="background: #ebeff2;color: #666">
            border
        </div>
        <div class="color-section white" style="background: #d9d9d9;color: #666">
            input-border
        </div>
    </section>
