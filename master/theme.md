# 主题与颜色

### 切换主题

后续版本将会提供更多主题和配色，敬请期待



### PHP颜色管理

作为日常开发我们离不开颜色的使用，`Dcat Admin`内置了颜色管理模块，此功能可以很方便的配合主题切换功能，让页面颜色与主题色相适应！

通过 `Dcat\Admin\Admin::color()` 这个服务可以很轻松的获取常用颜色（可参考[颜色表与样式](theme.md#颜色表与样式)）。

#### 获取内置颜色

通过`get`或魔术方法可以获取颜色代码，当获取的颜色不存在时，会返回参数的原始值。

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
通过`Color::alpha`方法可以获取设置颜色的透明度。

`Color::darken`方法接收两个参数：

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
