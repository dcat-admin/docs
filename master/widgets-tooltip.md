# 提示窗

基本用法

```php
<?php

use Dcat\Admin\Widgets\Tooltip;
use Dcat\Admin\Widgets\Card;
use Dcat\Admin\Layout\Content;

class MyController
{
    public function index(Content $content)
    {
        Tooltip::make('.icon-help-circle')->title('我是提示信息');
        
        return $content->body(new Card(
            <<<HTML
<div class="p4">
    <i class="feather icon-help-circle"></i>
</div>
HTML
        ));
    }
}
```
效果
<a href="{{public}}/assets/img/screenshots/tooltip.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/tooltip.png" width="70%" style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" >
</a>