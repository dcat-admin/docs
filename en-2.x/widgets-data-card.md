# Statistical data cards

`Dcat Admin` has a variety of commonly used statistical cards built in, which can be easily interacted with the back-end API, the following is an introduction to the Usage.

## Basic cards

The base card (`Dcat\Admin\Widgets\Metrics\Card`) is a card that does not display charts by default and is the simplest of the data cards.

<a href="{{public}}/assets/img/screenshots/total-users.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/total-users.png"  style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" >
</a>


### Simple Example

The use of the basic cards can be found in the built-in `App\Admin\Metrics\Examples\TotalUsers` category.

```php
<?php

namespace App\Admin\Metrics\Examples;

use Dcat\Admin\Widgets\Metrics\Card;
use Illuminate\Contracts\Support\Renderable;
use Illuminate\Http\Request;

class TotalUsers extends Card
{
    /**
     * Card footer content.
     *
     * @var string|Renderable|\Closure
     */
    protected $footer;
    
    // Save custom parameters
    protected $data = [];
    
    // The method parameters must be set to default values
    public function __construct(array $data = []) 
    {
        $this->data = [];
        
        parent::__construct();
    }

    protected function init()
    {
        parent::init();

        // Setting the TITLE
        $this->title('Total Users');
        // Setting the drop-down menu
        $this->dropdown([
            '7' => 'Last 7 Days',
            '28' => 'Last 28 Days',
            '30' => 'Last Month',
            '365' => 'Last Year',
        ]);
    }

    /**
     * Processing of requests.
     *
     * @param Request $request
     *
     * @return void
     */
    public function handle(Request $request)
    {
        // Get custom parameters passed externally
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
    
    // Passing a custom parameter to the handle method
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
     * Set the content of the footer of the card
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
     * Render the card content.
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

### Basic card method

#### initialization (init)

The `init` method is called in the card constructor method and can be used for card initialization operations.

```php
class MyCard extend Card
{
    protected function init()
    {
        parent::init();
    
        // Your initialization operation
    }
}
```


#### set TITLE (title)

The `title` method allows you to set a TITLE in the top left corner of the card.

```php
class MyCard extend Card
{
    protected function init()
    {
        parent::init();
    
        $this->title('active user');
    }
}
```

#### Set dropdown menu (dropdown)

The `dropdown` method allows you to set a dropdown menu button in the upper right corner of the card, which needs to be used in combination with the `handle` method to have result.

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

#### set subTitle (subTitle)

The `subTitle` method allows you to set a subTitle in the upper corner of the card.

> {tip} This method will conflict with the `dropdown` method, so if the dropdown button is set, there is no need to set a secondary TITLE.

```php
class MyCard extend Card
{
    protected function init()
    {
        parent::init();
    
        $this->subTitle('Last 30 days');
    }
}
```

#### Set header content (header)

The `header` method is used to set the content of the card header. This method accepts one parameter, which can be `string`, `Closure`, or a template view (`Illuminate\Contracts\Support\Renderable`).

> {tip} The content set in this way is in the same `div` container as `title`.

```php
class MyCard extend Card
{
    protected function init()
    {
        parent::init();
    
        $this->header(
            <<<HTML
            <div>Header content</div>        
HTML            
        );
        
        // You can also pass a closure
        $this->header(function () {
            return ...;
        });
        
        // You can also pass views
        $this->header(view('...'));
    }
}
```

#### Set the main content (content)

The `content` method accepts one parameter, which can be `string`, `Closure`, or template view (`Illuminate\Contracts\Support\Renderable`).

```php
class MyCard extend Card
{
    protected function init()
    {
        parent::init();
        
        $this->withContent('Custom Content');
    }

    /**
     * Card Content
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

#### Setting height (height)

Through the `height` method, you can set the minimum height of the card, the default is `165`.

```php
protected function init()
{
    parent::init();
    
    $this->height(200);
}
```

#### Passing custom parameters (parameters)

Parameters can be passed to the `handle` method via the `parameters` method.
```php
// Passing a custom parameter to the handle method
public function parameters() : array
{
	return [
		'key1' => 'value1',
		
		...
	];
}
```

Get custom parameters

```php
public function handle(Request $request)
{
    // Get custom parameters
	$key1 = $request->get('key1');
}
```

#### Rendering content (renderContent)

To ensure flexibility and scalability of the content, the system does not have any preset styles for the card content (i.e., just display whatever content is set, with no preset layouts or other styles).
The `renderContent` method allows you to change the default rendering of a card.

The following example demonstrates the main functionality of the `renderContent` method.
```php
use Dcat\Admin\Support\Helper;

class MyCard extend Card
{
    protected $footer;
    
    protected function init()
    {
        parent::init();
        
        // Set the card content
        $this->content(...);
        // Set the content at the bottom of the card
        $this->footer(...);
    }
    
    /**
     * Add this method to set the content at the bottom of the card
     *
     * @return $this
     */
    public function footer($footer)
    {
        $this->footer = $footer;
        
        return $this;
    }
    
    /**
     * Rendering bottom content
     *
     * @return $this
     */
    public function renderFooter()
    {
        return Helper::render($this->footer);
    }
    
    /**
     * Render the card content
     * You can add the bottom of the card here
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

#### Enable and render charts.

Basic cards have charting enabled by default, you can enable charting by using the `useChart` method, which instantiates a chart class and saves it in the `chart` property.

When the chart is enabled, you need to render the chart in your card content, otherwise the chart will be initialized, but still can not be displayed.

```php
class MyCard extend Card
{
    protected function init()
    {
        parent::init();
    
        // Enable Charts
        $this->useChart();
    }
    
    /**
     * Render the card content
     * You need to add the code to render the chart here
     *
     * @return string
     */
    public function renderContent()
    {
        // Content set by the content method
        $content = parent::renderContent();
        
        // Rendering Charts
        $chart = $this->renderChart();
        
        return <<<HTML
    <div class="my-chart">{$chart}</div>
    {$content}    
HTML           
    }
}
```

#### Chart Default Configuration (defaultChartOptions)

The default chart configuration can be set by the `defaultChartOptions` method, which is available only when the chart is enabled.

> {tip} The chart configuration here also supports setting executable `JS` code, please refer to [chart configuration setting executable JS code](widgets-charts.md#js) for detailed usage.

```php
class MyCard extend Card
{
    protected function defaultChartOptions()
    {
        // Return the configuration of the chart
        return [
            ...
        ];
    }
}
```

#### Set up a chart (chart)

The `chart` method allows you to set the chart configuration.

> {tip} The chart configuration here also supports setting executable `JS` code, please refer to [chart configuration setting executable JS code](widgets-charts.md#js) for detailed usage.

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

#### Setting the chart configuration (chartOption)

Through the `chartOption` method, you can set the chart configuration, this method can only set one parameter at a time, supports setting multidimensional parameters.

> {tip} The chart configuration here also supports setting executable `JS` code, please refer to [chart configuration setting executable JS code](widgets-charts.md#js) for detailed usage.

```php
class MyCard extend Card
{
    protected function init()
    {
        parent::init();
        
        $this->chartOption('stroke.curve', 'smooth');
        $this->chartOption(
            'plotOptions.radialBar.dataLabels.total.formatter',
            // The return code of this value will be executed as JS code
            JavaScript::make("function () { return {$number}; }")
        );
    }
}
```

#### Set Chart Height (chartHeight)

Through the `chartHeight` method can set the height of the chart, this method is very important, often need to be used in conjunction with the `height` method to adjust the overall height of the card.

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

#### Set the spacing on the chart (chartMarginTop)

The distance between the chart and the parent element can be set by the `chartMarginTop` method, which accepts a parameter of type `int` and can pass a `negative` number.

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

#### Set the bottom spacing of the chart (chartMarginBottom)

The `chartMarginBottom` method is used to set the distance between the chart and the lower elements, the method accepts an `int` type parameter and can pass a `negative` number.

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

#### Set chart labels (chartLabels)

The `chartLabels` method allows you to set the label (`labels`) configuration of the chart.

```php
class MyCard extend Card
{
    protected function init()
    {
        parent::init();
        
        $this->chartLabels('Label 1');
        
        // You can also pass arrays
        $this->chartLabels(['Label 1']);
    }
}
```

#### Set Chart Color (chartColors)

The `chartColors` method allows you to set the color (`colors`) configuration of the chart.

```php
class MyCard extend Card
{
    protected function init()
    {
        parent::init();
        
        $this->chartColors('#4f41a1');
        
        // You can also pass arrays
        $this->chartColors(['#4f41a1']);
    }
}
```

#### Rendering Charts (renderChart)

The `renderChart` method is used to render the chart

```php
class MyCard extend Card
{
    protected function init()
    {
        parent::init();
    
        // Enable Charts
        $this->useChart();
    }
    
    /**
     * Render the card content
     * You need to add the code to render the chart here
     *
     * @return string
     */
    public function renderContent()
    {
        // Content set by the content method
        $content = parent::renderContent();
        
        // Rendering Charts
        $chart = $this->renderChart();
        
        return <<<HTML
    <div class="my-chart">{$chart}</div>
    {$content}    
HTML           
    }
}
```

## Linear trend chart cards (Line)

The Linear Trend (`Dcat\Admin/Widgets\Metrics\Line`) is a statistical card with a polyline/line graph, inherited from the base card `Dcat\Admin/Widgets\Metrics\Card`.

<a href="{{public}}/assets/img/screenshots/card-line.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/card-line.png"  style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" >
</a>

### Example

Reference may be made to the built-in `App\Admin\Metrics\Examples\NewUsers` category.

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
     * Initialize card content
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
     * Processing of requests
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
                // Card Content
                $this->withContent(mt_rand(1000, 5000).'k');
                // Chart data
                $this->withChart(collect($generator(30))->toArray());
                // 直线
                break;
            case '30':
                // Card Content
                $this->withContent(mt_rand(400, 1000).'k');
                // Chart data
                $this->withChart(collect($generator(30))->toArray());
                // 直线
                break;
            case '28':
                // Card Content
                $this->withContent(mt_rand(400, 1000).'k');
                // Chart data
                $this->withChart(collect($generator(28))->toArray());
                // 直线
                break;
            case '7':
            default:
                // Card Content
                $this->withContent('89.2k');
                // Chart data
                $this->withChart([28, 40, 36, 52, 38, 60, 55,]);
        }
    }

    /**
     * Setting up chart data.
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
     * Set the card content.
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

### Methods

#### Set lines to straight (chartStraight)

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

#### Setting Lines to Curves (chartSmooth)

The default display is the curve

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

## doughnut card (Donut)

The Doughnut Card (`Dcat\AdminWidgets\Metrics\Donut`) is a statistical card with a doughnut chart, inherited from the base card `Dcat\AdminWidgets\Metrics\Card`.

<a href="{{public}}/assets/img/screenshots/card-donut.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/card-donut.png"  style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" >
</a>

### Example

Reference may be made to the built-in `App\Admin\Metrics\Examples\NewDevices` category.

```php
<?php

namespace App\Admin\Metrics\Examples;

use Dcat\Admin\Admin;
use Dcat\Admin\Widgets\Metrics\Donut;

class NewDevices extends Donut
{
    protected $labels = ['Desktop', 'Mobile'];

    /**
     * Initialize card content
     */
    protected function init()
    {
        parent::init();

        $color = Admin::color();
        $colors = [$color->primary(), $color->alpha('blue2', 0.5)];

        $this->title('New Devices');
        $this->subTitle('Last 30 days');
        $this->chartLabels($this->labels);
        // Set Chart Color
        $this->chartColors($colors);
    }

    /**
     * Rendering template
     *
     * @return string
     */
    public function render()
    {
        $this->fill();

        return parent::render();
    }

    /**
     * Writing data
     *
     * @return void
     */
    public function fill()
    {
        $this->withContent(44.9, 28.6);

        // Chart data
        $this->withChart([44.9, 28.6]);
    }

    /**
     * Setting chart data
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
     * Set the card header content
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


### Methods

#### set content width (contentWidth)

Through the `contentWidth` method, you can set the width of the text content and the chart content, the default is `[6, 6]`.

> {tip} The width here is a value between `1-12`.

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


## Bar Chart Cards (Bar)


The Bar Chart Card (`Dcat\Admin\Widgets\MetricsBar`) is a statistical card with a bar chart, inherited from the base card `Dcat\Admin\Widgets\MetricsCard`.

<a href="{{public}}/assets/img/screenshots/card-bar.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/card-bar.png"  style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" >
</a>

### Example

Reference may be made to the built-in `App\Admin\Metrics\Examples\NewDevices` category.

```php
<?php

namespace App\Admin\Metrics\Examples;

use Dcat\Admin\Admin;
use Dcat\Admin\Widgets\Metrics\Bar;
use Illuminate\Http\Request;

class Sessions extends Bar
{
    /**
     * Initialize card content
     */
    public function init()
    {
        parent::init();

        $color = Admin::color();

        $dark35 = $color->dark35();

        // Card content width
        $this->contentWidth(5, 7);
        // TITLE
        $this->title('Avg Sessions');
        // Set drop-down options
        $this->dropdown([
            '7' => 'Last 7 Days',
            '28' => 'Last 28 Days',
            '30' => 'Last Month',
            '365' => 'Last Year',
        ]);
        // Set Chart Color
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
     * Processing of requests
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
                // Card Content
                $this->withContent('2.7k', '+5.2%');

                // Chart data
                $this->withChart([
                    [
                        'name' => 'Sessions',
                        'data' => [75, 125, 225, 175, 125, 75, 25],
                    ],
                ]);
        }
    }

    /**
     * Setting up chart data.
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
     * Set the card content.
     *
     * @param string $title
     * @param string $value
     * @param string $style
     *
     * @return $this
     */
    public function withContent($title, $value, $style = 'success')
    {
        // Display according to options
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


## Circle Charts (Round)

The Round Chart Card (`Dcat\Admin/Widgets\Metrics\Round`) is a statistical card with multiple rings, inherited from the base card `Dcat\Admin/Widgets\Metrics\Card`.

<a href="{{public}}/assets/img/screenshots/card-ra.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/card-ra.png"  style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" >
</a>

### Example

The specific Example is similar to that of the above card, which can be referred to in the built-in `App\Admin\Metrics\Examples\ProductOrders` category, which will not be posted here.

## More built-in cards

The system also has built-in cards such as `Dcat\Admin\Widgets\Metrics\RadialBar`, `Dcat\Admin\Widgets\Metrics\SingleRound`, etc. Since the Usages are similar to the above-mentioned cards, the code will not be repeated here.

[Click me to see an online demo of all the built-in cards] (http://103.39.211.179:8080/admin/components/metric-cards)

## Custom chart cards

For custom cards, refer to the code for the built-in chart above.


