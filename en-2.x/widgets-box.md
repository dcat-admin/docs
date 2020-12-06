# Cards

## Card

```php
<?php

use Dcat\Admin\Widgets\Card;

// Fill only the content, no TITLE
$card = Card::make(view('...'));

// TITLE and CONTENT
$card = Card::make('TITLE', 'content');

// Set the spacing
$card->padding('0 15px 0 12px');

// Set tool buttons
$card->tool('<button class="btn btn-sm btn-light shadow-none">button text</button>');

// Set footer content
$card->footer(view('...'));
```

## Box

```php
<?php

use Dcat\Admin\Widgets\Box;

// TITLE and CONTENT
$box = Box::make('TITLE', 'content');

// Set the spacing
$box->padding('0 15px 0 12px');

// Set tool buttons
$box->tool('<button class="btn btn-sm btn-light shadow-none">button text</button>');

// Setting the shrink button
$box->collapsable();

// Set the remove button
$box->removable();
```