# JSON format field handling

The `dcat-admin` form provides the following components to handle `JSON` formatted fields, which are convenient for handling `JOSN` formatted objects, 1D arrays, 2D arrays, etc.


## KeyValue object (keyValue)

![]({{public}}/assets/img/screenshots/key-value.png)

If your field stores a variable `key` in the `{"field": "value"}` format, you can use the `keyValue` component:

```php
$form->keyValue('column_name');

// Set the validation rule
$form->keyValue('column_name')->rules('required|min:5');
```

Custom key names and translations of key titles

```php
$form->keyValue(...) ->setKeyLabel('KeyName')->setValueLabel('KeyValue');
```

## Fixed key-value objects (embeds)

![]({{public}}/assets/img/screenshots/embeds.png)

Used to process `mysql`'s `JSON` type field data or `mongodb`'s `object` type data, or store multiple `field` data values as `JSON` strings in `mysql`'s string type fields.

Applies to fields of type `JSON` with a fixed key value

```php
$form->embeds('column_name', function ($form) {

    $form->text('key1')->required();
    $form->email('key2')->required();
    $form->datetime('key3');

    $form->dateRange('key4', 'key5', 'scope')->rules('required');
})->saving(funtion ($v) {
    // Converted to json format for storage
    return json_encode($v);
});

// Custom TITLE
$form->embeds('column_name', 'Field TITLE', function ($form) {
    ...
});
```

The method calls for building form elements inside the callback function are the same as outside.

## one-dimensional array (list)

![]({{public}}/assets/img/screenshots/form-list.png)

If your field is used to store a one-dimensional array of `["foo", "Bar"]` format, you can use the `list` component:

```php
$form->list('column_name');

// Setting the validation rule
$form->list('column_name')->rules('required|min:5');

// Set the maximum and minimum number of elements
$form->list('column_name')->max(10)->min(5);
```

## Two-dimensional array (table)

![]({{public}}/assets/img/screenshots/form-table.png)

If a field stores a two-dimensional array in `json` format, the `table` form component can be used for fast editing:

```php
$form->table('column_name', function ($table) {
    $table->text('key');
    $table->text('value');
    $table->textarea('desc');
})->saving(function ($v) {
    return json_encode($v);
});
```

This component is similar to the `hasMany` component, but is used to handle the case of a single field, for simple two-dimensional data.


## Two-dimensional arrays (array)

![]({{public}}/assets/img/screenshots/has-many.png)

If a field stores a two-dimensional array in `json` format, and there are more fields, you can use the `array` form component for fast editing:

```php
$form->array('column_name', function ($table) {
    $table->text('key');
    $table->text('value');
    $table->textarea('desc');
})->saveAsJson();
```
