# 表格快捷搜索

快捷搜索是除了`filter`之外的另一个表格数据搜索方式，用来快速过滤你想要的数据，开启方式如下：

```php
$grid->quickSearch();

// 设置表单提示值
$grid->quickSearch()->placeholder('搜索...');
```
这样表头会出现一个搜索框:
<a href="{{public}}/assets/img/screenshots/grid-quick-search.png" target="_blank">
    <img style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" src="{{public}}/assets/img/screenshots/grid-quick-search.png">
</a>


通过给`quickSearch`方法传入不同的参数，来设置不同的搜索方式，有下面几种使用方法

## Like搜索

第一种方式，通过设置字段名称来进行简单的like查询
```php
$grid->quickSearch('title');

// 提交后模型会执行下面的查询
$model->where('title', 'like', "%{$input}%");
```

或者对多个字段做like查询:
```php
$grid->quickSearch('title', 'desc', 'content');
// 或
$grid->quickSearch(['title', 'desc', 'content']);

// 提交后模型会执行下面的查询
$model->where('title', 'like', "%{$input}%")
    ->orWhere('desc', 'like', "%{$input}%")
    ->orWhere('content', 'like', "%{$input}%");
```

### 关联关系


如果安装了 [dcat/laravel-wherehasin](https://github.com/jqhph/laravel-wherehasin)，则会优先使用`whereHasIn`方法进行查询操作

```php
$grid->quickSearch('user.name', 'user.username', 'content');
```


## 自定义搜索
第二种方式可以让你更灵活的控制搜索条件

```php
$grid->quickSearch(function ($model, $query) {
    $model->where('title', $query)->orWhere('desc', 'like', "%{$query}%");
});
```
其中闭包的参数`$query`为你填入搜索框中的内容，提交之后进行闭包中的查询。

## 快捷语法搜索
第三种方式参考了`Github`的搜索语法，来进行快捷搜索，调用方式：

```php
// 不传参数
$grid->quickSearch();
```
填入搜索框的内容按照以下的语法，在提交之后会进行相应的查询 :

### 比较查询
`title:foo` 、`title:!foo`

```php
$model->where('title', 'foo');

$model->where('title', '!=', 'foo');
```

`rate:>10`、`rate:<10`、`rate:>=10`、`rate:<=10`
```php
$model->where('rate', '>', 10);

$model->where('rate', '<', 10);

$model->where('rate', '>=', 10);

$model->where('rate', '<=', 10);
```

### In、NotIn查询
`status:(1,2,3,4)`、`status:!(1,2,3,4)`

```php
$model->whereIn('status', [1,2,3,4]);

$model->whereNotIn('status', [1,2,3,4]);
```

### Between查询
`score:[1,10]`

```php
$model->whereBetween('score', [1, 10]);
```

### 时间日期函数查询
`created_at:date,2019-06-08`

```php
$model->whereDate('created_at', '2019-06-08');
```

`created_at:time,09:57:45`
```php
$model->whereTime('created_at', '09:57:45');
```

`created_at:day,08`
```php
$model->whereDay('created_at', '08');
```

`created_at:month,06`
```php
$model->whereMonth('created_at', '06');
```

`created_at:year,2019`
```php
$model->whereYear('created_at', '2019');
```

### Like查询
`content:%Laudantium%`、`content:Laud%`

```php
$model->where('content', 'like', '%Laudantium%');

$model->where('content', 'like', 'Laud%');
```

### 正则查询
`username:/song/`

> {tip} 这里请使用MYSQL正则语法

```php
$model->where('username', 'REGEXP', 'song');
```

### 多条件组合搜索
用空格隔开多个搜索语句就可以实现多个字段的AND查询，比如`username:%song% status:(1,2,3)`, 提交之后会运行下面的搜索
```php
$model->where('username', 'like', '%song%')->whereIn('status', [1, 2, 3]);
```

如果某一个条件是`OR`查询, 只需要在语句单元前增加一个|符号即可： `username:%song% |status:(1,2,3)`
```php
$model->where('username', 'like', '%song%')->orWhereIn('status', [1, 2, 3]);
```

> {tip} 如果填入的查询文字中包含空格，需要放在双引号里面：`updated_at:"2019-06-08 09:57:45"`

### Label作为查询字段名称
不方便得到字段名的情况下，可以直接使用label名称作为查询字段

```php
 // 比如设置了`user_status`的表头列名为`用户状态`
$grid->column('user_status', '用户状态');
```

那么可以填入`用户状态:(1,2,3)`来执行下面的查询

```php
$model->whereIn('user_status', [1, 2, 3]);
```


## 禁止自动提交

快捷搜索默认是开启自动提交功能的，如果你不需要自动提交，可以通过以下方式禁用这个功能

> {tip} 禁用了自动提交功能之后需要通过按回车(`Enter`)键进行搜索。

```php
$grid->quickSearch()->auto(false);
```
