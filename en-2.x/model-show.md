# Basic use of data details

The `Dcat\Admin\Show` is used to display data details, starting with an example where there is a posts table in the database:

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
The corresponding data model is `App\Admin\Repositories\Post` and the data repository is `App\Admin\Repositories\Post` and the following code displays the data details of the posts table:


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
            ->description('Details')
            ->body(Show::make($id, new Post(), function (Show $show) {
                $show->id('ID');
                $show->title('TITLE');
                $show->content('Content');
                $show->rate();
                $show->created_at();
                $show->updated_at();
                $show->release_at();
            }));
    }
}
```

## Basic usage

### HTML content escaping
To prevent XSS attacks, the default output will be escaped using HTML:

```php
$show->avatar()->unescape()->as(function ($avatar) {

    return "<img src='{$avatar}' />";

});
```

### Set the field width
The default value of the field width is "3", and you can set the number between 1 and 12.

```php
$show->created_at->width(4);
```

### Modify the panel style and TITLE.
```php
$show->panel()
    ->style('danger')
    ->title('post Basic Information...');
```
The values of style can be primary, info, danger, warning, default.

### Panel tool settings
There are three buttons to edit, delete, and list by default in the upper right corner of the panel, and they can each be turned off in the following manner:

```php
$show->panel()
    ->tools(function ($tools) {
        $tools->disableEdit();
        $tools->disableList();
        $tools->disableDelete();
        // Show quick edit buttons
        $tools->showQuickEdit();
    });
```

#### Custom Complex Tool Button

Please refer to the document [Data Detail Action](action-show.md).


### Multicolumn layout

use

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

result
<a href="{{public}}/assets/img/screenshots/show-rows.png" target="_blank">
    <img class="img img-full" src="{{public}}/assets/img/screenshots/show-rows.png">
</a>