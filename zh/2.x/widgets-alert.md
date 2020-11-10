# 警告框

### Alert
基本用法

```php
<?php

use Dcat\Admin\Widgets\Alert;

$alert = Alert::make('内容', '标题');

// 类型
$alert->success();
$alert->danger();
$alert->info();
$alert->warning();

// 图标
$alert->icon('feather icon-x');

// 可移除按钮
$alert->removable();
```
效果
<a href="{{public}}/assets/img/screenshots/alert.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/alert.png" width="100%" style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" >
</a>

### Callout

```php
<?php

use Dcat\Admin\Widgets\Callout;

$callout = Callout::make('内容', '标题');

// 颜色
$callout->light();
$callout->primary();

// 可移除按钮
$callout->removable();
```

效果
<a href="{{public}}/assets/img/screenshots/callout.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/callout.png" width="100%" style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" >
</a>
