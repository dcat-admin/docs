# 开发前必读

### 开发环境请打开debug模式
`Dcat Admin`提供一些开发工具（如代码生成器）需要在 `debug` 模式下才能使用。
建议开发者在开发环境打开`debug`模式，把`.env`配置文件中的参数`APP_DEBUG`值设置为`true`即可。


### 按需引入JS脚本

`Dcat Admin` 使用 `jquery-pjax` 构建无刷新页面（单页应用），并且支持<b>按需加载</b> `JS` 脚本，支持在任意的页面方法（模板文件除外）中引入`JS`脚本，每个页面只需加载当前页面所需要使用到的 `js` 脚本。


示例：

定义一个卡片组件，这个组件需要引入一些前端静态资源文件

```php
<?php

use Illuminate\Contracts\Support\Renderable;
use Admin;

class Card implements Renderable
{
	public static $js = [
		'xxx/js/card.min.js',
	];
	public static $css = [
		'xxx/css/card.min.css',
	];

	public function script()
	{
		return <<<JS
		console.log('所有JS脚本都加载完了');
		$('xxx').card();
JS;		
	}

	public function render()
	{
		// 在这里可以引入你的js或css文件
		Admin::js(static::$js);
		Admin::css(static::$css);
		
		// 需要在页面执行的JS代码
		// 通过 Admin::script 设置的JS代码会自动在所有JS脚本都加载完毕后执行
		Admin::script($this->script());
		
		return view('...')->render();
	}
}
```

在控制器中使用这个组件
```php
use Dcat\Admin\Layout\Content;
use Card;

class HomeController
{
	public function index(Content $content)
	{
		// 使用上面的Card组件
		// Card组件需要用到的静态文件只会在当前请求加载
		// 其他请求不会加载 
		return $content->body(new Card());
	}
}
```


### 在页面中添加JS代码

由于加入了页面按需加载`JS`脚本的功能，所以在本项目内添加的`JS`代码都必须使用`Dcat.ready`方法监听`JS`脚本加载完毕事件，在此事件内的`JS`代码会在所有`JS`脚本都加载完毕后才执行。


使用 `Dcat\Admin\Admin::script` 方法添加的代码会自动放在`Dcat.ready`方法内执行。

```php
<?php

use Dcat\Admin\Admin;
use Dcat\Admin\Layout\Content;

class UserController
{
   public function index(Content $content)
   {
       Admin::script(
           <<<JS
(function () {
    // 如果有定义局部变量的需求，最好放在匿名函数内，防止变量污染
    var name = 'test';

    console.log('所有JS脚本都加载完毕啦~~', name)
})()    
JS
       );
       
       return $content->header(...)->body(...);
   }
}
```


如果是在模板文件中添加`JS`代码，则需要把代码放在`Dcat.ready`内执行

```html
<script>
// 用 Dcat.ready() 代替 $()
// 此方法会在所有 js 脚本加载完成后执行
Dcat.ready(function () {
    // 写入你的 js 代码
    console.log('所有 js 脚本加载完毕啦~~');
});
</script>
```

<a name="bootstrap-styles"></a>
### Bootstrap4公共样式

`Dcat Admin`采用`bootstrap4`的栅格系统对页面进行布局，即简单又强大，开始开发前需要对此有所了解，另外`bootsrap4`提供了非常多非常有用的[公共样式](https://getbootstrap.net/docs/utilities/borders/)，对编写页面组件非常有帮助，能显著提高开发效率，建议编写组件前先查阅一遍文档，以下是推荐学习的样式列表：

- [栅格布局](https://getbootstrap.net/docs/layout/grid/)
- [Display显示属性](https://getbootstrap.net/docs/utilities/display/) 通过我们的显示实用程序，可以快速、有效地切换组件的显示值和更多，包括对一些更常见的值的支持，此样式列表对响应式布局非常有帮助。
- [flex弹性布局](https://getbootstrap.net/docs/utilities/flex/) 引入新的Flex弹性布局，可以实现通过一整套响应灵活的实用程序，快速管理栅格的列、导航、组件等的布局、对齐和大小。通过进一度的定义CSS，还可以实现更复杂的展示样式。
- [颜色(Color)](https://getbootstrap.net/docs/utilities/colors/) 通过颜色传达意义、表达不同的模块，这有一系列的定义方法，包括支持链接、悬停、选中等状态相关的的样式集。
- [Float浮动属性](https://getbootstrap.net/docs/utilities/float/) 使用我们的响应式float浮动通用样式，能在任何设备断点（浏览器尺寸）上切换浮动。                             
- [规格(sizi)](https://getbootstrap.net/docs/utilities/sizing/) 使用系统宽度和高度样式，轻松地定义任何元素的宽或高（相对于其父级）
- [间隔(spacing)](https://getbootstrap.net/docs/utilities/spacing/) 内置了各种的快速缩进、隔离、填充等间距处理工具，响应余量和填充实用程序类来修改元素的外观。
- [文本处理](https://getbootstrap.net/docs/utilities/text/) 用于控制文本的对齐、组合、字重等示例以及使用文档。
- [垂直对齐(vertical align)](https://getbootstrap.net/docs/utilities/vertical-align/) 轻松更改内联、内嵌块、内联表和表格单元格元素的垂直对齐方式。


### 内置样式
除了前面提到的[`bootstrap4`公共样式](notice.md#bootstrap-styles)，系统还内置了以下常用样式：


#### 颜色
请参考[颜色表样式](theme.md#颜色表与样式)

#### 阴影
<style>
	.shadow-item {
		width:120px;height:100px;line-height:100px;margin:5px 10px;display:inline-block;text-align:center;
	}
</style>

<div class="shadow-item" style="box-shadow:0 2px 4px 0 rgba(0,0,0,.08);">
	<code>.shadow</code>
</div>
<div class="shadow-item" style="box-shadow:0 3px 1px -2px rgba(0,0,0,.05), 0 2px 2px 0 rgba(0,0,0,.05), 0 1px 5px 1px rgba(0,0,0,.05);">
	<code>.shadow-100</code>
</div>
<div class="shadow-item" style="box-shadow:0 3px 1px -2px rgba(0,0,0,.1), 0 2px 2px 0 rgba(0,0,0,.1), 0 1px 5px 1px rgba(0,0,0,.1)">
	<code>.shadow-200</code>
</div>
