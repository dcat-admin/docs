# 选项卡

通过`Dcat\Admin\Widgets\Tab`方法可以快速构建`tab`选项卡。

### 基本用法

```php
<?php

use Dcat\Admin\Widgets\Tab;

$tab = Tab::make();

// 第一个参数是选项卡标题，第二个参数是内容，第三个参数是是否选中
$tab->add('选项卡1', view('...'), true);
$tab->add('选项2', 'html');
// 添加选项卡链接
$tab->addLink('跳转链接', 'http://xxx');

return $content->body($tab->withCard());
```

### 切换显示模式

```php
// 主题色
$tab = Tab::make()->theme();
```

### 垂直 (vertical)

通过`vertical`方法可以让选项卡标题栏呈垂直排列。

```php
<?php

use Dcat\Admin\Widgets\Tab;

$tab = Tab::make();

$tab->add('选项卡1', view('...'));
$tab->add('选项2', 'html');

return $content->body($tab->withCard()->vertical());
```






