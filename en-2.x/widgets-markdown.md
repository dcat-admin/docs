# Markdown

Basic Usage

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
            Markdown::make('The content of your markdown article')
        ));
    }
}
```

