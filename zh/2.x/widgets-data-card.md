# 数据统计卡片

`Dcat Admin`中内置了多种常用数据统计卡片，可以非常方便的与后端API交互，下面逐一介绍用法。

## 基础卡片

基础卡片(`Dcat\Admin\Widgets\Metrics\Card`)是一种默认不显示图表的卡片，也是数据卡片中最简单的一种。

<a href="{{public}}/assets/img/screenshots/total-users.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/total-users.png"  style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" >
</a>


### 简单示例

基础卡片的使用可参考内置的`App\Admin\Metrics\Examples\TotalUsers`类。

```php
<?php

namespace App\Admin\Metrics\Examples;

use Dcat\Admin\Widgets\Metrics\Card;
use Illuminate\Contracts\Support\Renderable;
use Illuminate\Http\Request;

class TotalUsers extends Card
{
    /**
     * 卡片底部内容.
     *
     * @var string|Renderable|\Closure
     */
    protected $footer;
    
    // 保存自定义参数
    protected $data = [];
    
    // 构造方法参数必须设置默认值
    public function __construct(array $data = []) 
    {
        $this->data = [];
        
        parent::__construct();
    }

    protected function init()
    {
        parent::init();

        // 设置标题
        $this->title('Total Users');
        // 设置下拉菜单
        $this->dropdown([
            '7' => 'Last 7 Days',
            '28' => 'Last 28 Days',
            '30' => 'Last Month',
            '365' => 'Last Year',
        ]);
    }

    /**
     * 处理请求.
     *
     * @param Request $request
     *
     * @return void
     */
    public function handle(Request $request)
    {
        // 获取外部传递的自定义参数
        $key1 = $request->get('key1');
        
        switch ($request->get('option')) {
            case '365':
                $this->content(mt_rand(600, 1500));
                $this->down(mt_rand(1, 30));
                break;
            case '30':
                $this->content(mt_rand(170, 250));
                $this->up(mt_rand(12, 50));
                break;
            case '28':
                $this->content(mt_rand(155, 200));
                $this->up(mt_rand(5, 50));
                break;
            case '7':
            default:
                $this->content(143);
                $this->up(15);
        }
    }
    
    // 传递自定义参数到 handle 方法
    public function parameters() : array
    {
        return $this->data;
	}

    /**
     * @param int $percent
     *
     * @return $this
     */
    public function up($percent)
    {
        return $this->footer(
            "<i class=\"feather icon-trending-up text-success\"></i> {$percent}% Increase"
        );
    }

    /**
     * @param int $percent
     *
     * @return $this
     */
    public function down($percent)
    {
        return $this->footer(
            "<i class=\"feather icon-trending-down text-danger\"></i> {$percent}% Decrease"
        );
    }

    /**
     * 设置卡片底部内容
     *
     * @param string|Renderable|\Closure $footer
     *
     * @return $this
     */
    public function footer($footer)
    {
        $this->footer = $footer;

        return $this;
    }

    /**
     * 渲染卡片内容.
     *
     * @return string
     */
    public function renderContent()
    {
        $content = parent::renderContent();

        return <<<HTML
<div class="d-flex justify-content-between align-items-center mt-1" style="margin-bottom: 2px">
    <h2 class="ml-1 font-large-1">{$content}</h2>
</div>
<div class="ml-1 mt-1 font-weight-bold text-80">
    {$this->renderFooter()}
</div>
HTML;
    }

    /**
     * @return string
     */
    public function renderFooter()
    {
        return $this->toString($this->footer);
    }
}
```

### 基础卡片方法

#### 初始化 (init)

`init`方法会在卡片构造方法中被调用，可用于卡片初始化操作。

```php
class MyCard extend Card
{
    protected function init()
    {
        parent::init();
    
        // 你的初始化操作
    }
}
```


#### 设置标题 (title)

通过`title`方法可以在卡片的左上角设置一个标题

```php
class MyCard extend Card
{
    protected function init()
    {
        parent::init();
    
        $this->title('活跃用户');
    }
}
```

#### 设置下拉菜单 (dropdown)

通过`dropdown`方法可以在卡片右上角设置一个下拉菜单按钮，此功能需要结合`handle`方法一起使用才有效果。

```php
class MyCard extend Card
{
    protected function init()
    {
        $this->dropdown([
            '7' => 'Last 7 Days',
            '28' => 'Last 28 Days',
            '30' => 'Last Month',
            '365' => 'Last Year',
        ]);
    }
}
```

#### 设置副标题 (subTitle)

通过`subTitle`方法可以在卡片的又上角设置一个副标题。

> {tip} 此方法与`dropdown`方法会有冲突，如果设置过下拉菜单按钮，就不需要设置副标题。

```php
class MyCard extend Card
{
    protected function init()
    {
        parent::init();
    
        $this->subTitle('最近30天');
    }
}
```

#### 设置头部内容 (header)

通过`header`方法可以设置卡片头部内容，此方法接受一个参数，可以是`string`、`Closure`，也可以是模板视图(`Illuminate\Contracts\Support\Renderable`)。

> {tip} 通过此方法设置的内容与`title`在同一个`div`容器内。

```php
class MyCard extend Card
{
    protected function init()
    {
        parent::init();
    
        $this->header(
            <<<HTML
            <div>头部内容</div>        
HTML            
        );
        
        // 也可以传闭包
        $this->header(function () {
            return ...;
        });
        
        // 也可以传视图
        $this->header(view('...'));
    }
}
```

#### 设置主体内容 (content)

通过`content`方法可以设置卡片的内容主体，此方法接受一个参数，可以是`string`、`Closure`，也可以是模板视图(`Illuminate\Contracts\Support\Renderable`)。

```php
class MyCard extend Card
{
    protected function init()
    {
        parent::init();
        
        $this->withContent('自定义内容');
    }

    /**
     * 卡片内容
     *
     * @param string $content
     *
     * @return $this
     */
    public function withContent($content)
    {
        return $this->content(
            <<<HTML
<div class="d-flex flex-column flex-wrap text-center">
    <h1 class="font-large-2 mt-2 mb-0">{$content}</h1>
    <small>Tickets</small>
</div>
HTML
        );
    }
}
```

#### 设置高度 (height)

通过`height`方法可以设置卡片的最小高度，默认`165`。

```php
protected function init()
{
    parent::init();
    
    $this->height(200);
}
```

#### 传递自定义参数 (parameters)

通过 `parameters` 方法可以把参数传递到 `handle` 方法

```php
// 传递自定义参数到 handle 方法
public function parameters() : array
{
	return [
		'key1' => 'value1',
		
		...
	];
}
```

获取自定义参数

```php
public function handle(Request $request)
{
    // 获取自定义参数
	$key1 = $request->get('key1');
}
```

#### 渲染内容 (renderContent)

为了保证内容的灵活和可扩展性，系统没有对卡片内容预设任何样式(即设置什么内容就只显示什么内容，没有预设布局或其他样式)，
通过`renderContent`方法即可以更改卡片默认的渲染方式。

以下的例子演示了`renderContent`方法的主要功能
```php
use Dcat\Admin\Support\Helper;

class MyCard extend Card
{
    protected $footer;
    
    protected function init()
    {
        parent::init();
        
        // 设置卡片内容
        $this->content(...);
        // 设置卡片底部内容
        $this->footer(...);
    }
    
    /**
     * 增加此方法设置卡片底部内容
     *
     * @return $this
     */
    public function footer($footer)
    {
        $this->footer = $footer;
        
        return $this;
    }
    
    /**
     * 渲染底部内容
     *
     * @return $this
     */
    public function renderFooter()
    {
        return Helper::render($this->footer);
    }
    
    /**
     * 渲染卡片内容
     * 在这里即可加上卡片底部内容
     *
     * @return string
     */
    public function renderContent()
    {
        $content = parent::renderContent();
        $footer = $this->renderFooter();
    
        return <<<HTML
<div class="card-content">
    <div class="row">
        {$content}
    </div>
    <div class="metric-footer">
        {$footer}
    </div>
</div>
HTML;      
    }
}
```

#### 启用以及渲染图表

基本卡片默认是启用图表功能的，通过`useChart`方法可以启用图表功能，调用此方法之后会实例化一个图表类，然后保存在`chart`属性当中。

当图表启用之后，还需要在你的卡片内容中渲染图表，否则图表虽然被初始化了，但是仍无法显示。

```php
class MyCard extend Card
{
    protected function init()
    {
        parent::init();
    
        // 启用图表
        $this->useChart();
    }
    
    /**
     * 渲染卡片内容
     * 需要在这里加上渲染图表的代码
     *
     * @return string
     */
    public function renderContent()
    {
        // 通过 content 方法设置的内容
        $content = parent::renderContent();
        
        // 渲染图表
        $chart = $this->renderChart();
        
        return <<<HTML
    <div class="my-chart">{$chart}</div>
    {$content}    
HTML           
    }
}
```

#### 图表默认配置 (defaultChartOptions)

通过`defaultChartOptions`方法可以设置图表默认配置，此方法只有启用图表之后才有效。

> {tip} 这里的图表配置同样支持设置可执行`JS`代码，详细用法请参考[图表配置设置可执行JS代码](widgets-charts.md#js)。

```php
class MyCard extend Card
{
    protected function defaultChartOptions()
    {
        // 返回图表的配置
        return [
            ...
        ];
    }
}
```

#### 设置图表 (chart)

通过`chart`方法可以设置图表配置。

> {tip} 这里的图表配置同样支持设置可执行`JS`代码，详细用法请参考[图表配置设置可执行JS代码](widgets-charts.md#js)。

```php
class MyCard extend Card
{
    protected function init()
    {
        parent::init();
        
        $this->chart([...]);
    }
}
```

#### 设置图表配置 (chartOption)

通过`chartOption`方法可以设置图表配置，此方法一次只能设置一个参数，支持设置多维参数。

> {tip} 这里的图表配置同样支持设置可执行`JS`代码，详细用法请参考[图表配置设置可执行JS代码](widgets-charts.md#js)。

```php
class MyCard extend Card
{
    protected function init()
    {
        parent::init();
        
        $this->chartOption('stroke.curve', 'smooth');
        $this->chartOption(
            'plotOptions.radialBar.dataLabels.total.formatter',
            // 这个值最后段代码会作为JS代码执行
            JavaScript::make("function () { return {$number}; }")
        );
    }
}
```

#### 设置图表高度 (chartHeight)

通过`chartHeight`方法可以设置图表的高度，这个方法非常重要，经常需要结合`height`方法一起使用，调整卡片的整体高度。

```php
class MyCard extend Card
{
    protected function init()
    {
        parent::init();
        
        $this->chartHeight(150);
    }
}
```

#### 设置图表上间距 (chartMarginTop)

通过`chartMarginTop`方法可以设置图表与上级元素的间距，此方法接受一个`int`类型参数，可以传`负数`。

```php
class MyCard extend Card
{
    protected function init()
    {
        parent::init();
        
        $this->chartMarginTop(-10);
    }
}
```

#### 设置图表下间距 (chartMarginBottom)

通过`chartMarginBottom`方法可以设置图表与下级元素的间距，此方法接受一个`int`类型参数，可以传`负数`。

```php
class MyCard extend Card
{
    protected function init()
    {
        parent::init();
        
        $this->chartMarginBottom(10);
    }
}
```

#### 设置图表标签 (chartLabels)

通过`chartLabels`方法可以设置图表的标签(`labels`)配置。

```php
class MyCard extend Card
{
    protected function init()
    {
        parent::init();
        
        $this->chartLabels('标签1');
        
        // 也可以传递数组
        $this->chartLabels(['标签1']);
    }
}
```

#### 设置图表颜色 (chartColors)

通过`chartColors`方法可以设置图表的颜色(`colors`)配置。

```php
class MyCard extend Card
{
    protected function init()
    {
        parent::init();
        
        $this->chartColors('#4f41a1');
        
        // 也可以传递数组
        $this->chartColors(['#4f41a1']);
    }
}
```

#### 渲染图表 (renderChart)

通过`renderChart`方法可以渲染图表。

```php
class MyCard extend Card
{
    protected function init()
    {
        parent::init();
    
        // 启用图表
        $this->useChart();
    }
    
    /**
     * 渲染卡片内容
     * 需要在这里加上渲染图表的代码
     *
     * @return string
     */
    public function renderContent()
    {
        // 通过 content 方法设置的内容
        $content = parent::renderContent();
        
        // 渲染图表
        $chart = $this->renderChart();
        
        return <<<HTML
    <div class="my-chart">{$chart}</div>
    {$content}    
HTML           
    }
}
```

## 线性趋势图卡片 (Line)

线性趋势图卡片(`Dcat\Admin\Widgets\Metrics\Line`)是一个附带了折线\曲线图的数据统计卡片，继承自基础卡片`Dcat\Admin\Widgets\Metrics\Card`。

<a href="{{public}}/assets/img/screenshots/card-line.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/card-line.png"  style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" >
</a>

### 示例

可参考内置的`App\Admin\Metrics\Examples\NewUsers`类。

```php
<?php

namespace App\Admin\Metrics\Examples;

use Dcat\Admin\Widgets\Metrics\Line;
use Illuminate\Http\Request;

class NewUsers extends Line
{
    /**
     * @var string
     */
    protected $label = 'New Users';

    /**
     * 初始化卡片内容
     *
     * @return void
     */
    protected function init()
    {
        parent::init();

        $this->title($this->label);
        $this->dropdown([
            '7' => 'Last 7 Days',
            '28' => 'Last 28 Days',
            '30' => 'Last Month',
            '365' => 'Last Year',
        ]);
    }

    /**
     * 处理请求
     *
     * @param Request $request
     *
     * @return mixed|void
     */
    public function handle(Request $request)
    {
        $generator = function ($len, $min = 10, $max = 300) {
            for ($i = 0; $i <= $len; $i++) {
                yield mt_rand($min, $max);
            }
        };

        switch ($request->get('option')) {
            case '365':
                // 卡片内容
                $this->withContent(mt_rand(1000, 5000).'k');
                // 图表数据
                $this->withChart(collect($generator(30))->toArray());
                // 直线
                break;
            case '30':
                // 卡片内容
                $this->withContent(mt_rand(400, 1000).'k');
                // 图表数据
                $this->withChart(collect($generator(30))->toArray());
                // 直线
                break;
            case '28':
                // 卡片内容
                $this->withContent(mt_rand(400, 1000).'k');
                // 图表数据
                $this->withChart(collect($generator(28))->toArray());
                // 直线
                break;
            case '7':
            default:
                // 卡片内容
                $this->withContent('89.2k');
                // 图表数据
                $this->withChart([28, 40, 36, 52, 38, 60, 55,]);
        }
    }

    /**
     * 设置图表数据.
     *
     * @param array $data
     *
     * @return $this
     */
    public function withChart(array $data)
    {
        return $this->chart([
            'series' => [
                [
                    'name' => $this->label,
                    'data' => $data,
                ],
            ],
        ]);
    }

    /**
     * 设置卡片内容.
     *
     * @param string $content
     *
     * @return $this
     */
    public function withContent($content)
    {
        return $this->content(
            <<<HTML
<div class="d-flex justify-content-between align-items-center mt-1" style="margin-bottom: 2px">
    <h2 class="ml-1 font-large-1">{$content}</h2>
    <span class="mb-0 mr-1 text-80">{$this->label}</span>
</div>
HTML
        );
    }
}
```

### 方法

#### 设置线条为直线 (chartStraight)

```php
use Dcat\Admin\Widgets\Metrics\Line;

class MyCard extend Line
{
    protected function init()
    {
        parent::init();
        
        $this->chartStraight();
    }
}
```

#### 设置线条为曲线 (chartSmooth)

默认显示的是曲线。

```php
use Dcat\Admin\Widgets\Metrics\Line;

class MyCard extend Line
{
    protected function init()
    {
        parent::init();
        
        $this->chartSmooth();
    }
}
```

## 圆环图卡片 (Donut)

圆环图卡片(`Dcat\Admin\Widgets\Metrics\Donut`)是一个附带了圆环图的数据统计卡片，继承自基础卡片`Dcat\Admin\Widgets\Metrics\Card`。

<a href="{{public}}/assets/img/screenshots/card-donut.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/card-donut.png"  style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" >
</a>

### 示例

可参考内置的`App\Admin\Metrics\Examples\NewDevices`类。

```php
<?php

namespace App\Admin\Metrics\Examples;

use Dcat\Admin\Admin;
use Dcat\Admin\Widgets\Metrics\Donut;

class NewDevices extends Donut
{
    protected $labels = ['Desktop', 'Mobile'];

    /**
     * 初始化卡片内容
     */
    protected function init()
    {
        parent::init();

        $color = Admin::color();
        $colors = [$color->primary(), $color->alpha('blue2', 0.5)];

        $this->title('New Devices');
        $this->subTitle('Last 30 days');
        $this->chartLabels($this->labels);
        // 设置图表颜色
        $this->chartColors($colors);
    }

    /**
     * 渲染模板
     *
     * @return string
     */
    public function render()
    {
        $this->fill();

        return parent::render();
    }

    /**
     * 写入数据.
     *
     * @return void
     */
    public function fill()
    {
        $this->withContent(44.9, 28.6);

        // 图表数据
        $this->withChart([44.9, 28.6]);
    }

    /**
     * 设置图表数据.
     *
     * @param array $data
     *
     * @return $this
     */
    public function withChart(array $data)
    {
        return $this->chart([
            'series' => $data
        ]);
    }

    /**
     * 设置卡片头部内容.
     *
     * @param mixed $desktop
     * @param mixed $mobile
     *
     * @return $this
     */
    protected function withContent($desktop, $mobile)
    {
        $blue = Admin::color()->alpha('blue2', 0.5);

        $style = 'margin-bottom: 8px';
        $labelWidth = 120;

        return $this->content(
            <<<HTML
<div class="d-flex pl-1 pr-1 pt-1" style="{$style}">
    <div style="width: {$labelWidth}px">
        <i class="fa fa-circle text-primary"></i> {$this->labels[0]}
    </div>
    <div>{$desktop}</div>
</div>
<div class="d-flex pl-1 pr-1" style="{$style}">
    <div style="width: {$labelWidth}px">
        <i class="fa fa-circle" style="color: $blue"></i> {$this->labels[1]}
    </div>
    <div>{$mobile}</div>
</div>
HTML
        );
    }
}
```


### 方法

#### 设置内容宽度 (contentWidth)

通过`contentWidth`方法可以设置文本内容以及图表内容的宽度，默认为`[6, 6]`。

> {tip} 这里的宽度是一个`1-12`之间的一个值。

```php
use Dcat\Admin\Widgets\Metrics\Line;

class MyCard extend Line
{
    protected function init()
    {
        parent::init();
        
        $this->contentWidth(4, 8);
    }
}
```


## 柱状图卡片 (Bar)


柱状图卡片(`Dcat\Admin\Widgets\Metrics\Bar`)是一个附带了柱状图的数据统计卡片，继承自基础卡片`Dcat\Admin\Widgets\Metrics\Card`。

<a href="{{public}}/assets/img/screenshots/card-bar.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/card-bar.png"  style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" >
</a>

### 示例

可参考内置的`App\Admin\Metrics\Examples\NewDevices`类。

```php
<?php

namespace App\Admin\Metrics\Examples;

use Dcat\Admin\Admin;
use Dcat\Admin\Widgets\Metrics\Bar;
use Illuminate\Http\Request;

class Sessions extends Bar
{
    /**
     * 初始化卡片内容
     */
    public function init()
    {
        parent::init();

        $color = Admin::color();

        $dark35 = $color->dark35();

        // 卡片内容宽度
        $this->contentWidth(5, 7);
        // 标题
        $this->title('Avg Sessions');
        // 设置下拉选项
        $this->dropdown([
            '7' => 'Last 7 Days',
            '28' => 'Last 28 Days',
            '30' => 'Last Month',
            '365' => 'Last Year',
        ]);
        // 设置图表颜色
        $this->chartColors([
            $dark35,
            $dark35,
            $color->primary(),
            $dark35,
            $dark35,
            $dark35
        ]);
    }

    /**
     * 处理请求
     *
     * @param Request $request
     *
     * @return mixed|void
     */
    public function handle(Request $request)
    {
        switch ($request->get('option')) {
            case '7':
            default:
                // 卡片内容
                $this->withContent('2.7k', '+5.2%');

                // 图表数据
                $this->withChart([
                    [
                        'name' => 'Sessions',
                        'data' => [75, 125, 225, 175, 125, 75, 25],
                    ],
                ]);
        }
    }

    /**
     * 设置图表数据.
     *
     * @param array $data
     *
     * @return $this
     */
    public function withChart(array $data)
    {
        return $this->chart([
            'series' => $data,
        ]);
    }

    /**
     * 设置卡片内容.
     *
     * @param string $title
     * @param string $value
     * @param string $style
     *
     * @return $this
     */
    public function withContent($title, $value, $style = 'success')
    {
        // 根据选项显示
        $label = strtolower(
            $this->dropdown[request()->option] ?? 'last 7 days'
        );

        $minHeight = '183px';

        return $this->content(
            <<<HTML
<div class="d-flex p-1 flex-column justify-content-between" style="padding-top: 0;width: 100%;height: 100%;min-height: {$minHeight}">
    <div class="text-left">
        <h1 class="font-large-2 mt-2 mb-0">{$title}</h1>
        <h5 class="font-medium-2" style="margin-top: 10px;">
            <span class="text-{$style}">{$value} </span>
            <span>vs {$label}</span>
        </h5>
    </div>

    <a href="#" class="btn btn-primary shadow waves-effect waves-light">View Details <i class="feather icon-chevrons-right"></i></a>
</div>
HTML
        );
    }
}
```


## 多环形图卡片 (Round)

柱状图卡片(`Dcat\Admin\Widgets\Metrics\Round`)是一个附带了多环形图的数据统计卡片，继承自基础卡片`Dcat\Admin\Widgets\Metrics\Card`。

<a href="{{public}}/assets/img/screenshots/card-ra.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/card-ra.png"  style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" >
</a>

### 示例

具体示例与上述卡片的示例大同小异，具体可参考内置的`App\Admin\Metrics\Examples\ProductOrders`类，这里不再贴出。

## 更多内置卡片

系统还内置了`Dcat\Admin\Widgets\Metrics\RadialBar`、`Dcat\Admin\Widgets\Metrics\SingleRound`等卡片，由于用法与上述卡片雷同，这里不再重复贴出代码。

[点击我查看所有内置卡片在线演示](http://103.39.211.179:8080/admin/components/metric-cards)

## 自定义图表卡片

如需自定义卡片，可参考上述内置图表的代码。


