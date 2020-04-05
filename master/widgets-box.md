# 卡片

## Card

```php
<?php

use Dcat\Admin\Widgets\Card;

// 只填充内容，不需要标题
$card = Card::make(view('...'));

// 标题和内容
$card = Card::make('标题', '内容');

// 设置间距
$card->padding('0 15px 0 12px');

// 设置工具按钮
$card->tool('<button class="btn btn-sm btn-light shadow-none">按钮</button>');

// 设置底部内容
$card->footer(view('...'));
```

## Box

```php
<?php

use Dcat\Admin\Widgets\Box;

// 标题和内容
$box = Box::make('标题', '内容');

// 设置间距
$box->padding('0 15px 0 12px');

// 设置工具按钮
$box->tool('<button class="btn btn-sm btn-light shadow-none">按钮</button>');

// 设置收缩按钮
$box->collapsable();

// 设置移除按钮
$box->removable();
```