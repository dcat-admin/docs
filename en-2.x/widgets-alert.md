# Warning boxes

### Alert
Basic Usage

```php
<?php

use Dcat\Admin\Widgets\Alert;

$alert = Alert::make('content', 'TITLE');

// types
$alert->success();
$alert->danger();
$alert->info();
$alert->warning();

// icon
$alert->icon('feather icon-x');

// Removable buttons
$alert->removable();
```
result
<a href="{{public}}/assets/img/screenshots/alert.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/alert.png" width="100%" style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" >
</a>

### Callout

```php
<?php

use Dcat\Admin\Widgets\Callout;

$callout = Callout::make('content', 'TITLE');

// color
$callout->light();
$callout->primary();

// Removable buttons
$callout->removable();
```

result
<a href="{{public}}/assets/img/screenshots/callout.png" target="_blank">
    <img src="{{public}}/assets/img/screenshots/callout.png" width="100%" style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" >
</a>
