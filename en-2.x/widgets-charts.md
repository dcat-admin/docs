# Chart

`Dcat Admin` introduces the [apexchartsChart](https://apexcharts.com/) feature, and the `Dcat\Admin\Widgets\ApexCharts\Chart` class can help developers to render charts quickly.


### Simple Usage

If you need to build a chart, you can refer to it as follows

> {tip} For more types of charts, please refer to [official apexcharts documentation](https://apexcharts.com/).

```php
<?php

namespace App\Admin\Widgets\Charts;

use Dcat\Admin\Admin;
use Dcat\Admin\Widgets\ApexCharts\Chart;

class MyBar extends Chart
{
    public function __construct($containerSelector = null, $options = [])
    {
        parent::__construct($containerSelector, $options);

        $this->setUpOptions();
    }

    /**
     * 初始化图表配置
     */
    protected function setUpOptions()
    {
        $color = Admin::color();

        $colors = [$color->primary(), $color->primaryDarker()];

        $this->options([
            'colors' => $colors,
            'chart' => [
                'type' => 'bar',
                'height' => 430
            ],
            'plotOptions' => [
                'bar' => [
                    'horizontal' => true,
                    'dataLabels' => [
                        'position' => 'top',
                    ],
                ]
            ],
            'dataLabels' => [
                'enabled' => true,
                'offsetX' => -6,
                'style' => [
                    'fontSize' => '12px',
                    'colors' => ['#fff']
                ]
            ],
            'stroke' => [
                'show' => true,
                'width' => 1,
                'colors' => ['#fff']
            ],
            'xaxis' => [
                'categories' => [],
            ],
        ]);
    }

    /**
     * Working with graphical data
     */
    protected function buildData()
    {
        // Execute your data query logic
        $data = [
            [
                'data' => [44, 55, 41, 64, 22, 43, 21]
            ],
            [
                'data' => [53, 32, 33, 52, 13, 44, 32]
            ]
        ];
        $categories = [2001, 2002, 2003, 2004, 2005, 2006, 2007];

        $this->withData($data);
        $this->withCategories($categories);
    }

    /**
     * Setting up chart data
     *
     * @param array $data
     *
     * @return $this
     */
    public function withData(array $data)
    {
        return $this->option('series', $data);
    }

    /**
     * Set chart categories
     *
     * @param array $data
     *
     * @return $this
     */
    public function withCategories(array $data)
    {
        return $this->option('xaxis.categories', $data);
    }

    /**
     * Rendering Charts
     *
     * @return string
     */
    public function render()
    {
        $this->buildData();

        return parent::render();
    }
}
```

使用

```php
<?php

use App\Admin\Widgets\Charts\MyBar;
use Dcat\Admin\Widgets\Card;
use Dcat\Admin\Layout\Content;

class MyController
{
    public function index(Content $content)
    {
        return $content->body(
            Card::make('我的图表', MyBar::make())
        );
    }
}
```

result

<a href="{{public}}/assets/img/screenshots/widget-bar.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/widget-bar.png"  style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" >
</a>

### 图表与后端API交互

如果你的图表需要与后端API交互，可以参考以下方式

> {tip} 为了方便阅读，这里的Example代码直接继承前面定义的`MyBar`类。

```php
<?php

namespace App\Admin\Widgets\Charts;

use Illuminate\Http\Request;

class MyAjaxBar extends MyBar
{
    protected $id;
    protected $username;
    
    // 这里的参数一定要设置默认值
    public function __construct($id = null, $username = null) 
    {
        parent::__construct();
        
        $this->id = $id;
        $this->username = $username;
    }
    
    /**
     * 处理请求
     * 如果你的图表类中包含此方法，则可以通过此方法处理前端通过ajax提交的获取图表数据的请求
     *
     * @param Request $request
     * @return mixed|void
     */
    public function handle(Request $request)
    {
        // 获取 parameters 方法设置的自定义参数
        $id = $request->get('id');
        $username = $request->get('username');
        
        switch ((int) $request->get('option')) {
            case 30:
                // 你的数据查询逻辑
                $data = [
                    [
                        'data' => [44, 55, 41, 64, 22, 43, 21]
                    ],
                    [
                        'data' => [53, 32, 33, 52, 13, 44, 32]
                    ]
                ];
                $categories = [2001, 2002, 2003, 2004, 2005, 2006, 2007];

                break;
            case 28:
                // 你的数据查询逻辑
                $data = [
                    [
                        'data' => [44, 55, 41, 64, 22, 43, 21]
                    ],
                    [
                        'data' => [53, 32, 33, 52, 13, 44, 32]
                    ]
                ];
                $categories = [2001, 2002, 2003, 2004, 2005, 2006, 2007];

                break;
            case 7:
            default:
                // 你的数据查询逻辑
                $data = [
                    [
                        'data' => [44, 55, 41, 64, 22, 43, 21]
                    ],
                    [
                        'data' => [53, 32, 33, 52, 13, 44, 32]
                    ]
                ];
                $categories = [2001, 2002, 2003, 2004, 2005, 2006, 2007];
                break;
        }

        $this->withData($data);
        $this->withCategories($categories);
    }

	/**
 	 * 这里返回需要异步传递到 handler 方法的参数 
 	 * 
	 * @return array
	 */
	public function parameters(): array
	{
	    return [
	        'id' 	   => $this->id,
	        'username' => $this->username,
		];
	}

    /**
     * 这里覆写父类的方法，不再查询数据
     */
    protected function buildData()
    {
    }
}
```

用户点击构建下拉菜单加载不同的图表数据

```php
<?php

use App\Admin\Widgets\Charts\MyAjaxBar;
use Dcat\Admin\Widgets\Dropdown;
use Dcat\Admin\Widgets\Box;
use Dcat\Admin\Layout\Row;
use Dcat\Admin\Layout\Content;

class MyController
{
    public function index(Content $content)
    {
        return $content->body(function (Row $row) {
            // 构建下拉菜单，当点击菜单时发起请求获取数据重新渲染图表
            $menu = [
                '7'  => '最近7天',
                '28' => '最近28天',
                '30' => '最近30天',
            ];
            $dropdown = Dropdown::make($menu)
                ->button(current($menu))
                ->click()
                ->map(function ($v, $k) {
                    // 此处设置的 data-xxx 属性会作为post数据发送到后端api
                    return "<a class='switch-bar' data-option='{$k}'>{$v}</a>";
                });

			// 传递自定义参数
            $id = ...;
            $username = ...;

            $bar = MyAjaxBar::make($id, $username)
                ->fetching('$("#my-box").loading()') // 设置loadingresult
                ->fetched('$("#my-box").loading(false)') // 移除loadingresult
                ->click('.switch-bar'); // 设置图表点击菜单则重新发起请求，且被点击的目标元素上的 data-xxx 属性会被作为post数据发送到后端API

            $box = Box::make('我的图表2', $bar)
                ->id('my-box') // 设置盒子的ID
                ->tool($dropdown); // 设置下拉菜单按钮

            $row->column(12, $box);
        });
    }
}
```

result

<a href="{{public}}/assets/img/screenshots/widget-bar2.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/widget-bar2.png"  style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" >
</a>

<a href="js"></a>
### 设置图表配置为可执行JS代码

如果你需要在图表配置加入可执行的JS代码，可参考以下方式

```php
use use Dcat\Admin\Support\JavaScript;
use Dcat\Admin\Admin;
use Dcat\Admin\Widgets\ApexCharts\Chart;

class MyBar extends Chart
{
    public function __construct($containerSelector = null, $options = [])
    {
        parent::__construct($containerSelector, $options);

        $this->setUpOptions();
    }

    /**
     * 初始化图表配置
     */
    protected function setUpOptions()
    {
        $number = 20;
    
        $this->option(
            'plotOptions.radialBar.dataLabels.total.formatter',
            // 这个值最后段代码会作为JS代码执行
            JavaScript::make("function () { return {$number}; }")
        );
        
        ...
    }
    
    ...   
}    
```



