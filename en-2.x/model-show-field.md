# Field display

### HTML


The `html` method allows you to insert a piece of `HTML` code without displaying `label` on the details page

```php
// pass in string
$show->html('<br/>');

// pass in view
$show->html(view(...));

// incoming closures
$show->html(function () {
	// Get field information
	$id = $this->id;
	$username = $this->username;
	
	return view(..., ['id' => $id]);
});
```

### Dividers
If a separator line is to be added between fields.

```php
$show->divider();
```

### Line feed
If you want to use line feeds between fields.

```php
$show->newline();
```

### Modify display content
Modify the display using the following methods

```php
$show->title()->as(function ($title) {
    // Get other fields of the current row
    $username = $this->username;

    return "<{$title}> {$username}";
});

$show->contents()->as(function ($content) {
    return "<pre>{$content}</pre>";
});
```

### Ways to help
The help method is used in the same way as the data form field help method, see [Help Method] (model-grid-column.md#help).


### Built-in Display Extension Methods
Here are a few common display styles that are built in as methods:

#### view
The `view` method can introduce a view file.
```php
// The following three variables are received in the template:
// name field name
// value field value
// model Current row data
$show->content->view('admin.fields.content');
```

#### explode
The `explode` method splits the string into arrays.
```php
$show->tag->explode()->label();

// You can specify delimiter, default ",".
$show->tag->explode('|')->label();
```

#### prepend

The `prepend` method is used to insert content in front of a value of type `string` or `array`.

```php
// When the value of a field is a string
$show->email->prepend('mailto:');

// When the value of a field is an array
$show->arr->prepend('first item');
```

The `prepend` method allows passing parameters to closures
```php
$show->email->prepend(function ($value, $original) {
    // $value is the current field value
    // $original is the original value of the current field as queried from the database.
    
    // Get other field values
    $username = $this->username;
    
    return "[{$username}]";
});
```


#### append


The `append` method is used to insert content after a value of type `string` or `array`.

```php
// When the value of a field is a string
$show->email->append('@gmail.com');

// When the value of a field is an array
$show->arr->append('last item');
```

The `append` method allows passing parameters to closures
```php
$show->email->append(function ($value, $original) {
    // $value is the current field value
    // $original is the original value of the current field as queried from the database.
    
    // Get other field values
    $username = $this->username;
    
    return "[{$username}]";
});
```

#### image
The content of the field avatar is the path or url of the image, which can be displayed as the image.

```php
$show->avatar()->image();
```
The parameters of the image() method refer to Field::image().

#### file
The content of the field document is the path or url of a file, which can be displayed as a file.

```php
$show->avatar()->file();
```
The arguments of the file() method refer to Field::file().

#### link
The content of the field homepage is the url link, which can be displayed as an HTML link.

```php
$show->homepage()->link();
```
The arguments of the link() method refer to Field::link()

#### label
Display the contents of the field tag as label.

```php
$show->tag()->label();
```
The arguments of the label() method refer to Field::label().

#### badge
Display the contents of the field rate as badge.

```php
$show->rate()->badge();
```
The arguments of the badge() method are referenced in Field::badge().

#### using
If the field gender takes the values `f` and `m`, it needs to be displayed as female and male respectively.

```php
$show->gender()->using(['f' => '女', 'm' => '男']);
```

#### dot

Column text can be preceded by a colored dot using the `dot` method.


```php
use Dcat\Admin\Admin;

$show->state
	->using([1 => 'Unprocessed', 2 => 'Processed', ...])
	->dot(
		[
			1 => 'primary',
			2 => 'danger',
			3 => 'success',
			4 => Admin::color()->info(),
		], 
	    'primary' // The second parameter is the default value.
	);
```

#### Display file size
If the field data is a number of bytes representing the size of the file, you can display more readable text by calling the filezise method

```php
$show->field('file_size')->filesize();
```
This value 199812019 will show 190.56 MB.
