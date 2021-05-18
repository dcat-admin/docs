# Form field dynamic display

> {tip} This works just as well in [tools-form](widgets-form.md).

Dynamic display of form fields means that when the specified option of a form item is selected, other form items are displayed in association.

<a href="{{public}}/assets/img/screenshots/form-when.gif" target="_blank">
    <img class="img" src="{{public}}/assets/img/screenshots/form-when.gif" />
</a>    


The currently supported components for form linkage are.

- `select`
- `multipleSelect`
- `radio`
- `checkbox`

## Methods of use

The above components can be divided into two types of single-selected and multi-selected, where `select` and `radio` are single-selected components, and the others are multi-selected components.

### Single-selected components

In the following example, selecting a different nationality type will toggle the selection of a different linkage table item.

```php
$form->radio('radio')
    ->when([1, 4], function (Form $form) {
        // The text box is displayed when the values are 1 and 4
        $form->text('text1');
        $form->text('text2');
        $form->text('text3');
    })
    ->when(2, function (Form $form) {
        $form->editor('editor');
    })
    ->when(3, function (Form $form) {
        $form->image('image');
    })
    ->options([
        1 => 'Show text box',
        2 => 'Show Editor',
        3 => 'Show file upload',
        4 => 'Still showing the text box',
    ])
    ->default(1);
```

In the above example, the method `when(1, $callback)` is equivalent to `when('=', 1, $callback)`, which can be omitted if the operator `=` is used.

The also supported operators `=`, `>`, `>=`, `<`, `<=`, `!=` are used as follows:

```php
$form->radio('check')
    ->when('>', 1, function () {

    })->when('>=', 2, function () {

    });
```

The `select` component is used in the same way as `radio`.

Also note that if the form cannot use the `required` method after using the dynamic display feature, you should use `required_if` instead, as in

```php
$form->radio('type')
    ->when([1, 4], function (Form $form) {
        $form->text('text1')
            ->rules('required_if:type,1,4') // use required_if
            ->setLabelClass(['asterisk']); // show * number
    });
```

### Multiple choice components

Multiple choice components support two operators: `in`、`notIn`

```php
$form->checkbox('nationality', 'citizenship')
    ->options([
        1 => 'China',
        2 => 'Abroad',
    ])->when([1, 2], function (Form $form) { 

        $form->text('name', 'name and surname');
        $form->text('idcard', 'identity card');

    })->when('notIn', 2, function (Form $form) { 

        $form->text('name', 'name and surname');
        $form->text('passport', 'passport number');

    });
```

The `multipleSelect` component is used in the same way as `checkbox`.


### Layout

The form dynamic display function can be used in conjunction with `column` and `tab` layout functions.


`tab` layout
```php
$form->tab('Radio', function (Form $form) {
    $form->display('title')->value('单选框动态展示');

    $form->radio('radio')
        ->when([1, 4], function (Form $form) {
            $form->text('text1');
            $form->text('text2');
        })
        ->when(2, function (Form $form) {
            $form->editor('editor');
        })
        ->options($this->options)
        ->default(1);
});
```

`column` layout
```php
$form->column(6, function (Form $form) {
    $form->radio('radio')
        ->when([1, 4], function (Form $form) {
            $form->text('text1');
            $form->text('text2');
        })
        ->when(2, function (Form $form) {
            $form->editor('editor');
        })
        ->options($this->options)
        ->default(1);
});
```