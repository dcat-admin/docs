# Tabs

The `tab` tab can be quickly constructed using the `Dcat\Admin\Widgets\Tab` method.

### Basic Usage

```php
<?php

use Dcat\Admin\Widgets\Tab;

$tab = Tab::make();

// The first parameter is the tab TITLE, the second parameter is the content, and the third parameter is whether or not the tab is selected.
$tab->add('Tab 1', view('...'), true);
$tab->add('Option 2', 'html');
// Adding tab links
$tab->addLink('skip link', 'http://xxx');

return $content->body($tab->withCard());
```

### Switching display mode

```php
// theme color
$tab = Tab::make()->theme();
```

### vertically (vertical)

The `vertical` method allows the tab TITLE column to be aligned vertically.

```php
<?php

use Dcat\Admin\Widgets\Tab;

$tab = Tab::make();

$tab->add('Tab 1', view('...'));
$tab->add('Option 2', 'html');

return $content->body($tab->withCard()->vertical());
```






