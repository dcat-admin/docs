# 数据详情基本使用

`Dcat\Admin\Show`用来显示数据详情，先来个例子，数据库中有posts表：

```sql
CREATE TABLE `posts` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `author_id` int(10) unsigned NOT NULL ,
  `title` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
  `content` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
  `rate` int(255) COLLATE utf8_unicode_ci NOT NULL,
  `release_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  `created_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  `updated_at` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
```
对应的数据模型为`App\Models\Post`，数据仓库为`App\Admin\Repositories\Post`，下面的代码可以显示posts表的数据详情：


```php
<?php

namespace App\Admin\Controllers;

use App\Http\Controllers\Controller;
use App\Admin\Repositories\Post;
use Dcat\Admin\Layout\Content;
use Dcat\Admin\Show;
use Dcat\Admin\Admin;

class PostController extends Controller
{
    public function show($id, Content $content)
    {
        return $content->header('Post')
            ->description('详情')
            ->body(Show::make($id, new Post(), function (Show $show) {
                $show->id('ID');
                $show->title('标题');
                $show->content('内容');
                $show->rate();
                $show->created_at();
                $show->updated_at();
                $show->release_at();
            }));
    }
}
```

## 基本使用方法

### HTML内容转义
为了防止XSS攻击, 默认输出的内容都会使用HTML转义，如果你不想转义输出`HTML`，可以调用`unescape`方法：

```php
$show->avatar()->unescape()->as(function ($avatar) {

    return "<img src='{$avatar}' />";

});
```

### 设置字段宽度
字段宽度默认值为“3”，可以设置1-12之间的数字。

```php
$show->created_at->width(4);
```

### 修改面板的样式和标题
```php
$show->panel()
    ->style('danger')
    ->title('post基本信息...');
```
style的取值可以是primary、info、danger、warning、default

### 面板工具设置
面板右上角默认有三个按钮编辑、删除、列表，可以分别用下面的方式关掉它们：

```php
$show->panel()
    ->tools(function ($tools) {
        $tools->disableEdit();
        $tools->disableList();
        $tools->disableDelete();
        // 显示快捷编辑按钮
        $tools->showQuickEdit();
    });
```

#### 自定义复杂工具按钮

请参考文档[数据详情动作](action-show.md)


### 多列布局

使用

```php
$show->row(function (Show\Row $show) {
    $show->width(3)->id;
    $show->width(3)->name;
    $show->width(5)->email;
});

$show->row(function (Show\Row $show) {
    $show->width(5)->email_verified_at;
    $show->created_at;
    $show->updated_at;
});

$show->row(function (Show\Row $show) {
    $show->width(3)->field('profile.first_name');
    $show->field('profile.last_name');
    $show->width(3)->field('profile.postcode');
});
```

效果
<a href="{{public}}/assets/img/screenshots/show-rows.png" target="_blank">
    <img class="img img-full" src="{{public}}/assets/img/screenshots/show-rows.png">
</a>