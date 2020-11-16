# Markdown

基本用法

```php
<?php

use Dcat\Admin\Widgets\Markdown;
use Dcat\Admin\Widgets\Card;
use Dcat\Admin\Layout\Content;

class MyController
{
    public function index(Content $content)
    {
        return $content->body(Card::make(
            Markdown::make('你的markdown本文内容')
        ));
    }
}
```

