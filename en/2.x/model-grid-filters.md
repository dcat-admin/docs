# Query filter


`model-grid` provides a number of methods to implement query filtering of table data:

```php
// prohibit the use of sth.
$grid->disableFilter();
// show
$grid->showFilter();

// Disable filter button
$grid->disableFilterButton();
// Display filter button
$grid->showFilterButton();

$grid->filter(function($filter){
    // Expand Filter
    $filter->expand();
    
    // Add a field filter here
    $filter->equal('id', '产品序列号');
    $filter->like('name', 'name');
    ...

});
```

## Filter layout

The default layout is `rightSide`.

### rightSide
```php
use Dcat\Admin\Grid;

$grid->filter(function (Grid\Filter $filter) {
    // Change to rightSide Layout
    $grid->rightSide();
    
    ...
});
```
result

<a href="{{public}}/assets/img/screenshots/filter-right-side.png" target="_blank">
    <img class="img img-full" src="{{public}}/assets/img/screenshots/filter-right-side.png">
</a>


### panel

```php
use Dcat\Admin\Grid;

$grid->filter(function (Grid\Filter $filter) {
    // Change to panel layout
    $filter->panel();
    
    // Note that you need to readjust the width of the form fields when switching to panel layout.
    $filter->equal('id')->width(3);
});
```
result

<a href="{{public}}/assets/img/screenshots/filter-panel.png" target="_blank">
    <img class="img img-full" src="{{public}}/assets/img/screenshots/filter-panel.png">
</a>

### Custom layout (view)

If the above layout does not meet your needs, you can customize the filter template using the `view` method

```php
$grid->filter(function ($filter) {
    $filter->view('xxx');
    
    ...
});
```


<a name="type"></a>
## Query type

The following filter types are currently supported:

<a name="equal"></a>
### equal
`sql: ... WHERE `column` = "$input"`：
```php
$filter->equal('column', $label);
```

<a name="nequal"></a>
### notEqual
`sql: ... WHERE `column` != "$input"`：
```php
$filter->notEqual('column', $label);
```
<a name="like"></a>
### like
`sql: ... WHERE `column` LIKE "%$input%"`：
```php
$filter->like('column', $label);
```

### ilike
`sql: ... WHERE `column` ILIKE "%$input%"`：
```php
$filter->ilike('column', $label);
```
<a name="startwith"></a>
### startWith
`sql: ... WHERE `column` LIKE "$input%"`：
```php
$filter->startWith('column', $label);

// If you need to use “ilike”
$filter->startWith('column', $label)->ilike();
```

<a name="endwith"></a>
### endWith
`sql: ... WHERE `column` LIKE "%$input"`：
```php
$filter->endWith('column', $label);

// If you need to use “ilike”
$filter->endWith('column', $label)->ilike();
```

<a name="gt"></a>
### gt
`sql: ... WHERE `column` > "$input"`：
```php
$filter->gt('column', $label);
```

<a name="lt"></a>
### lt
`sql: ... WHERE `column` < "$input"`：
```php
$filter->lt('column', $label);
```

<a name="ngt"></a>
### ngt
`sql: ... WHERE `column` <= "$input"`：
```php
$filter->ngt('column', $label);
```

<a name="nlt"></a>
### nlt
`sql: ... WHERE `column` >= "$input"`：
```php
$filter->nlt('column', $label);
```

<a name="between"></a>
### between
`sql: ... WHERE `column` BETWEEN "$start" AND "$end"`：
```php
$filter->between('column', $label);

// Set datetime type
$filter->between('column', $label)->datetime();

// Set the time type
$filter->between('column', $label)->time();
```

<a name="in"></a>
### in
`sql: ... WHERE `column` in (...$inputs)`：
```php
$filter->in('column', $label)->multipleSelect(['key' => 'value']);
```

<a name="notIn"></a>
### notIn
`sql: ... WHERE `column` not in (...$inputs)`：
```php
$filter->notIn('column', $label)->multipleSelect(['key' => 'value']);
```

<a name="date"></a>
### date
`sql: ... WHERE DATE(`column`) = "$input"`：
```php
$filter->date('column', $label);
```

<a name="day"></a>
### day
`sql: ... WHERE DAY(`column`) = "$input"`：
```php
$filter->day('column', $label);
```

<a name="month"></a>
### month
`sql: ... WHERE MONTH(`column`) = "$input"`：
```php
$filter->month('column', $label);
```

<a name="year"></a>
### year
`sql: ... WHERE YEAR(`column`) = "$input"`：
```php
$filter->year('column', $label);
```

<a name="where"></a>
### Complex query where

You can use WHERE to build more complex query filters.

`sql: ... WHERE `title` LIKE "%$input" OR `content` LIKE "%$input"`：
```php
$filter->where('search', function ($query) {

    $query->where('title', 'like', "%{$this->input}%")
        ->orWhere('content', 'like', "%{$this->input}%");

});
```

`sql: ... WHERE `rate` >= 6 AND `created_at` = {$input}`:
```php
$filter->where('Text', function ($query) {

    $query->whereRaw("`rate` >= 6 AND `created_at` = {$this->input}");

});
```

Relational query for the field corresponding to the relationship `profile`.
```php
$filter->where('mobile', function ($query) {

    $query->whereHas('profile', function ($query) {
        $query->where('address', 'like', "%{$this->input}%")->orWhere('email', 'like', "%{$this->input}%");
    });

}, 'Address or mobile number');
```

<a name="whereBetween"></a>
### Complex range queries whereBetween

Customizable range queries via `whereBetween`

```php
$filter->whereBetween('created_at', function ($q) {
	$start = $this->input['start'] ?? null;
	$end = $this->input['end'] ?? null;

	$q->whereHas('goods', function ($q) use ($start) {
		if ($start !== null) {
			$q->where('price', '>=', $start);
		}
		 
		if ($end !== null) {
			$q->where('price', '<=', $end);
		}
	});
});       
```

This method also supports time and date range queries

```php
$filter->whereBetween('created_at', function ($q) {
	...
})->datetime(); 
```

<a name="group"></a>
### Filter group
Sometimes you need to set multiple filters for the same field, which can be done in the following ways

```php
$filter->group('rate', function ($group) {
    $group->gt('大于');
    $group->lt('小于');
    $group->nlt('不小于');
    $group->ngt('不大于');
    $group->equal('等于');
});
```
There are several methods that can be invoked
```php
// equal
$group->equal();

// not equal to
$group->notEqual();

// greater than
$group->gt();

// less than
$group->lt();

// greater than or equal to
$group->nlt();

// less than equal to
$group->ngt();

// match
$group->match();

// complex condition
$group->where();

// like query
$group->like();

// like query
$group->contains();

// ilike search
$group->ilike();

// Begin with what you have entered
$group->startWith();

// End with what you have entered
$group->endWith();
```

<a name="scope"></a>
### Range query scope
It is possible to define the queries you use most often as a range of queries that will appear in the drop-down menu of the Filter button, here are a few examples:
```php
$filter->scope('male', '男性')->where('gender', 'm');

// multi-conditional query
$filter->scope('new', 'Recently changed')
    ->whereDate('created_at', date('Y-m-d'))
    ->orWhere('updated_at', date('Y-m-d'));

// Affiliation Search
$filter->scope('address')->whereHas('profile', function ($query) {
    $query->whereNotNull('address');
});

$filter->scope('trashed', 'Soft-deleted data')->onlyTrashed();
```
The first argument of the scope method is the key of the query, which will appear in the url, the second argument is the label of the drop-down menu item.

The scope method can be chained to any eloquent query condition, result reference Demo

<a name="form"></a>
## Type of form

<a name="text"></a>
### text

The default form type is `text input`, you can set `placeholder`:

```php
$filter->equal('column')->placeholder('Please enter...');
```

It is also possible to restrict the user input format in some of the following ways:

```php
$filter->equal('column')->url();

$filter->equal('column')->email();

$filter->equal('column')->integer();

$filter->equal('column')->ip();

$filter->equal('column')->mac();

$filter->equal('column')->mobile();

// $options see https://github.com/RobinHerbots/Inputmask/blob/4.x/README_numeric.md
$filter->equal('column')->decimal($options = []);

// $options see https://github.com/RobinHerbots/Inputmask/blob/4.x/README_numeric.md
$filter->equal('column')->currency($options = []);

// $options see https://github.com/RobinHerbots/Inputmask/blob/4.x/README_numeric.md
$filter->equal('column')->percentage($options = []);

// $options see https://github.com/RobinHerbots/Inputmask, $icon为input前面的图标
$filter->equal('column')->inputmask($options = [], $icon = 'pencil');
```


<a name="select-table"></a>
### Table selector (selectTable)


```php
use App\Admin\Renderable\UserTable;
use Dcat\Admin\Models\Administrator;

$filter->equal('user_id')
	->selectTable(UserTable::make(['id' => ...])) // Set the rendering class instance and pass custom parameters
	->title('Pop-up window TITLE')
	->dialogWidth('50%') // Popup width, default 800px
	->model(Administrator::class, 'id', 'name'); // Set the display of editing data
	
// The code above is equivalent to
$filter->equal('user_id')
	->selectTable(UserTable::make(['id' => ...])) // Set the rendering class instance and pass custom parameters
	->options(function ($v) { // Set the display of editing data
		if (! $v) {
			return [];
		}
		
		return Administrator::find($v)->pluck('name', 'id');
	});
```

Define the rendering class as follows, which needs to inherit `Dcat\Admin\Grid\LazyRenderable`.

> {tip} The data table is loaded asynchronously here, please refer to [load asynchronously](lazy.md) for more details.

```php
<?php

namespace App\Admin\Renderable;

use Dcat\Admin\Grid;
use Dcat\Admin\Grid\LazyRenderable;
use Dcat\Admin\Models\Administrator;

class UserTable extends LazyRenderable
{
    public function grid(): Grid
    {
        // Getting externally passed parameters
        $id = $this->id;
        
        return Grid::make(new Administrator(), function (Grid $grid) {
            $grid->column('id');
            $grid->column('username');
            $grid->column('name');
            $grid->column('created_at');
            $grid->column('updated_at');
            
            // Specify the field name of the value to be displayed when the line selector is selected.
			// Specify the field name of the value to be displayed when the line selector is selected.
			// Specify the field name of the value to be displayed when the line selector is selected.
			// If the form data has a "name", "title", or "username" field, you do not have to set this.
			$grid->rowSelector()->titleColumn('username');

            $grid->quickSearch(['id', 'username', 'name']);

            $grid->paginate(10);
            $grid->disableActions();

            $grid->filter(function (Grid\Filter $filter) {
                $filter->like('username')->width(4);
                $filter->like('name')->width(4);
            });
        });
    }
}
```


#### multipleSelectTable

Multi-selected Uses are consistent with the `selectTable` method above

```php
$filter->in('user_id')
	->multipleSelectTable(UserTable::make(['id' => $form->getKey()])) // Set the rendering class instance and pass custom parameters
	->max(10) // Select up to 10 options. If you don't pass this, there's no limit.
	->model(Administrator::class, 'id', 'name'); // Set the display of editing data
```



<a name="select"></a>
### select
```php
$filter->equal('column')->select(['key' => 'value'...]);

// Or get data from an api, the format of the api refers to the select component of the model-form
$filter->equal('column')->select('api/users');
```

<a name="multipleSelect"></a>
### multipleSelect
Generally used with `in` and `notIn` query types that require an array of queries, but can also be used with `where` type queries:
```php
$filter->in('column')->multipleSelect(['key' => 'value'...]);

// Or get data from an api, the format of which is referenced in the model-form's multipleSelect component
$filter->in('column')->multipleSelect('api/users');
```

<a name="datetime"></a>
### datetime

Search by date time component. Reference for parameters and values of `$options` [bootstrap-datetimepicker](http://eonasdan.github.io/bootstrap-datetimepicker/Options/)

```php
$filter->equal('column')->datetime($options);

// `date()` equal to `datetime(['format' => 'YYYY-MM-DD'])`
$filter->equal('column')->date();

// `time()` equal to `datetime(['format' => 'HH:mm:ss'])`
$filter->equal('column')->time();

// `day()` equal to `datetime(['format' => 'DD'])`
$filter->equal('column')->day();

// `month()` equal to `datetime(['format' => 'MM'])`
$filter->equal('column')->month();

// `year()` equal to `datetime(['format' => 'YYYY'])`
$filter->equal('column')->year();

```

If your time-date field is stored in the database as a timestamp type, you can convert the form value to a timestamp by using the `toTimestamp` method
```php
$filter->equal('column')->datetime($options)->toTimestamp();
```


<a name="method"></a>
## Common methods

<a name="width"></a>
### Filter width
```php
// Set width to a value between "1-12", default value is "3".
$filter->equal('column')->width(3);

// you can also write out the absolute width
$filter->equal('column')->width('250px');
```

<a name="default"></a>
### Set default values (default)
```php
$filter->equal('column')->default('text');

$filter->equal('column')->select([0 => 'PHP', 1 => 'Java'])->default(1);
```

<a name="expand"></a>
### Expanding the filter (expand)
```php
$filter->expand();

$filter->equal('column');
...
```

<a name="withoutInputBorder"></a>
### Do not display the border of the filter input box
```php
$filter->withoutInputBorder();

$filter->equal('column');
...
```

<a name="style"></a>
### Set filter container style
```php
$filter->style('padding:0');

$filter->equal('column');
...
```

<a name="padding"></a>
### Set filter container padding
```php
$filter->padding('10px', '10px', '10px', '10px');

$filter->equal('column');
...
```

### ignore filter (ignore)

The `ignore` method allows you to ignore the current filter when submitting the form.

```php
$filter->equal('column')->ignore();
```


<a name="relation"></a>
## Relational field lookup

Suppose your model is as follows

```php
class User extends Model
{
    public function profile()
    {
        return $this->hasOne(...);
    }
    
    public function myPosts()
    {
        return $this->hasMany(...);
    }
}
```

The `first_name` field of the `profiles` table and the `title` field of the `posts` table can be queried by the following methods

```php
$grid->filter(function ($filter) {
    $filter->like('profile.first_name');
    
    $filter->like('myPosts.title');
});
```

If [dcat/laravel-wherehasin](https://github.com/jqhph/laravel-wherehasin) is installed, the `whereHasIn` method will be used in preference to queries


<a name="extend"></a>
## Custom Filters
The following is an implementation of `between` to explain how to customize the filter.

First create a new filter class to inherit `Dcat\Admin\Filter\AbstractFilter`:
```php
<?php

namespace Dcat\Admin\Grid\Filter;

use Dcat\Admin\Admin;
use Dcat\Admin\Grid\Filter\Presenter\DateTime;
use Illuminate\Support\Arr;

class Between extends AbstractFilter
{
    // Customize your filter display templates
    protected $view = 'admin::filter.between';

    // This method is used to generate a unique id for the filter field
    // This unique id can be manipulated with js code
    public function formatId($column)
    {
       $id   = str_replace('.', '_', $column);
       $name = $this->parent->getGrid()->getName();

       return ['start' => "{$name}{$id}_start", 'end' => "{$name}{$id}_end"];
    }

    // Form form name attribute formatting
    protected function formatName($column)
    {
        $columns = explode('.', $column);

        if (count($columns) == 1) {
            $name = $columns[0];
        } else {
            $name = array_shift($columns);

            foreach ($columns as $column) {
                $name .= "[$column]";
            }
        }

        return ['start' => "{$name}[start]", 'end' => "{$name}[end]"];
    }

    // Create conditions
    // The build condition here supports the `Laravel query builder`.
    public function condition($inputs)
    {
        if (!Arr::has($inputs, $this->column)) {
            return;
        }

        $this->value = Arr::get($inputs, $this->column);

        $value = array_filter($this->value, function ($val) {
            return $val !== '';
        });

        if (empty($value)) {
            return;
        }

        if (!isset($value['start']) && isset($value['end'])) {
            // The array returned here is equivalent to
            // $query->where($this->column, '<=', $value['end']);
            return $this->buildCondition($this->column, '<=', $value['end']);
        }

        if (!isset($value['end']) && isset($value['start'])) {
            // The array returned here is equivalent to
            // $query->where($this->column, '>=', $value['end']);
            return $this->buildCondition($this->column, '>=', $value['start']);
        }

        $this->query = 'whereBetween';

        // The array returned here is equivalent to
        // $query->whereBetween($this->column, $value['end']);
        return $this->buildCondition($this->column, $this->value);
    }

    // Customizing the filter form display
    public function datetime($options = [])
    {
        $this->view = 'admin::filter.betweenDatetime';

        DateTime::collectAssets();

        $this->setupDatetime($options);

        return $this;
    }

    protected function setupDatetime($options = [])
    {
        $options['format'] = Arr::get($options, 'format', 'YYYY-MM-DD HH:mm:ss');
        $options['locale'] = Arr::get($options, 'locale', config('app.locale'));

        $startOptions = json_encode($options);
        $endOptions = json_encode($options + ['useCurrent' => false]);

        // Do what you want with the form using the formatted id above.
        $script = <<<JS
            $('#{$this->id['start']}').datetimepicker($startOptions);
            $('#{$this->id['end']}').datetimepicker($endOptions);
            $("#{$this->id['start']}").on("dp.change", function (e) {
                $('#{$this->id['end']}').data("DateTimePicker").minDate(e.date);
            });
            $("#{$this->id['end']}").on("dp.change", function (e) {
                $('#{$this->id['start']}').data("DateTimePicker").maxDate(e.date);
            });
JS;

        Admin::script($script);
    }
}
```
`admin::filter.between` looks like：
```php
<div class="filter-input col-sm-{{ $width }} "  style="{!! $style !!}">
    <div class="form-group" >
        <div class="input-group input-group-sm">
            <span class="input-group-addon"><b>{!! $label !!}</b></span>
            <input type="text" class="form-control" placeholder="{{$label}}" name="{{$name['start']}}" value="{{ request($name['start'], \Illuminate\Support\Arr::get($value, 'start')) }}">
            <span class="input-group-addon" style="border-left: 0; border-right: 0;">To</span>
            <input type="text" class="form-control" placeholder="{{$label}}" name="{{$name['end']}}" value="{{ request($name['end'], \Illuminate\Support\Arr::get($value, 'end')) }}">
        </div>
    </div>
</div>
```
`admin::filter.betweenDatetime`looks like：
```php
<div class="filter-input col-sm-{{ $width }}"  style="{!! $style !!}">
    <div class="form-group">
        <div class="input-group input-group-sm">
            <span class="input-group-addon"><b>{{$label}}</b>  &nbsp;<i class="fa fa-calendar"></i></span>
            <input type="text" class="form-control" id="{{$id['start']}}" placeholder="{{$label}}" name="{{$name['start']}}" value="{{ request($name['start'], \Illuminate\Support\Arr::get($value, 'start')) }}">
            <span class="input-group-addon" style="border-left: 0; border-right: 0;">To</span>
            <input type="text" class="form-control" id="{{$id['end']}}" placeholder="{{$label}}" name="{{$name['end']}}" value="{{ request($name['end'], \Illuminate\Support\Arr::get($value, 'end')) }}">
        </div>
    </div>
</div>
```

Now just call the `extend` method to use it, open `app/Admin/bootstrap.php` and add the following code:
```php
Filter::extend('customBetween', Filter\Between::class);
```

use：
>After extending the filter method, the IDE will not be completed automatically by default, so you can generate an IDE hint file with `php artisan admin:ide-helper`.

```php
$filter->customBetween('created_at')->datetime();
```

