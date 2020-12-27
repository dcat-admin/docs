# 字段显示

### HTML


通过`html`方法可以在详情页插入一段不显示`label`的`HTML`代码

```php
// 传入字符串
$show->html('<br/>');

// 传入视图
$show->html(view(...));

// 传入闭包
$show->html(function () {
	// 获取字段信息
	$id = $this->id;
	$username = $this->username;
	
	return view(..., ['id' => $id]);
});
```

### 分隔线
如果要在字段之间添加一条分隔线：

```php
$show->divider();
```

### 换行
如果要在字段之间使用换行：

```php
$show->newline();
```

### 修改显示内容
用下面的方法修改显示内容

```php
$show->title()->as(function ($title) {
    // 获取当前行的其他字段
    $username = $this->username;

    return "<{$title}> {$username}";
});

$show->contents()->as(function ($content) {
    return "<pre>{$content}</pre>";
});
```

### 帮助方法
帮助方法与数据表格字段帮助方法使用一致，可参考[帮助方法](model-grid-column.md#help)。


### 内置显示扩展方法
下面是通过as方法内置实现的几个常用的显示样式：

#### view
`view`方法可以引入一个视图文件。
```php
// 模板中接收以下三个变量：
// name 字段名称
// value 字段值
// model 当前行数据
$show->content->view('admin.fields.content');
```

#### explode
`explode`方法可以把字符串分割为数组。
```php
$show->tag->explode()->label();

// 可以指定分隔符，默认","
$show->tag->explode('|')->label();
```

#### prepend

`prepend` 方法用于给 `string` 或 `array` 类型的值前面插入内容。

```php
// 当字段值是一个字符串
$show->email->prepend('mailto:');

// 当字段值是一个数组
$show->arr->prepend('first item');
```

`prepend`方法允许传入闭包参数
```php
$show->email->prepend(function ($value, $original) {
    // $value 是当前字段值
    // $original 是当前字段从数据库中查询出来的原始值
    
    // 获取其他字段值
    $username = $this->username;
    
    return "[{$username}]";
});
```


#### append


`append` 方法用于给 `string` 或 `array` 类型的值后面插入内容。

```php
// 当字段值是一个字符串
$show->email->append('@gmail.com');

// 当字段值是一个数组
$show->arr->append('last item');
```

`append`方法允许传入闭包参数
```php
$show->email->prepend(function ($value, $original) {
    // $value 是当前字段值
    // $original 是当前字段从数据库中查询出来的原始值
    
    // 获取其他字段值
    $username = $this->username;
    
    return "[{$username}]";
});
```

#### image
字段avatar的内容是图片的路径或者url，可以将它显示为图片：

```php
$show->avatar()->image();
```
image()方法的参数参考Field::image()

#### file
字段document的内容是文件的路径或者url，可以将它显示为文件：

```php
$show->avatar()->file();
```
file()方法的参数参考Field::file()

#### link
字段homepage的内容是url链接，可以将它显示为HTML链接：

```php
$show->homepage()->link();
```
link()方法的参数参考Field::link()

#### label
将字段tag的内容显示为label：

```php
$show->tag()->label();
```
label()方法的参数参考Field::label()

#### badge
将字段rate的内容显示为badge：

```php
$show->rate()->badge();
```
badge()方法的参数参考Field::badge()

#### using
如果字段gender的取值为`f`、`m`，分别需要用女、男来显示

```php
$show->gender()->using(['f' => '女', 'm' => '男']);
```

#### dot

通过`dot`方法可以在列文字前面加上一个带颜色的圆点


```php
use Dcat\Admin\Admin;

$show->state
	->using([1 => '未处理', 2 => '已处理', ...])
	->dot(
		[
			1 => 'primary',
			2 => 'danger',
			3 => 'success',
			4 => Admin::color()->info(),
		], 
	    'primary' // 第二个参数为默认值
	);
```

#### 显示文件尺寸
如果字段数据是表示文件大小的字节数，可以通过调用filezise方法来显示更有可读性的文字

```php
$show->field('file_size')->filesize();
```
这样数值199812019将会显示为190.56 MB
