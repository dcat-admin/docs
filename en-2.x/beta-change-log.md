# BETA version update log


## v2.0.14-beta

Release Date 2020/12/24

To upgrade the method, step by step execute the following command
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.14-beta"
php artisan admin:publish --assets --migrations --force
php artisan migrate
```

### Function improvement

**1. Optimize the error message prompt for file upload failure**

In the old version, the error message for file upload failure was not very clear, which made it difficult to define the reason for the error, so in this version, the error message is optimized, and the specific reason will be displayed once the file upload fails.

### Bug fixes

1. fix the problem that the form field conflicts with the model `casts` attribute and the abnormal display problem using string splicing in the `display` closure [#876](https://github.com/jqhph/dcat-admin/issues/876)
2. fix the problem that the dynamic display function of the form cannot be used [#879](https://github.com/jqhph/dcat-admin/issues/879)
3. fix the problem of not being able to display editing data when using `Block` layout [#877](https://github.com/jqhph/dcat-admin/issues/877)

## v2.0.13-beta

Release Date 2020/12/23

To upgrade the method, step by step execute the following command
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.13-beta"
php artisan admin:publish --assets --migrations --force
php artisan migrate
```

### Bug fixes

1. fix the problem that the table may report an error when displaying related fields when the related data does not exist [#867](https://github.com/jqhph/dcat-admin/issues/867)
2. fix the problem that `display` method is invalid when the table uses data repository to return arrays or non-model `collection` [#869](https://github.com/jqhph/dcat-admin/issues/869)


## v2.0.12-beta

Release Date 2020/12/22

To upgrade the method, step by step execute the following command
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.12-beta"
php artisan admin:publish --assets --migrations --force
php artisan migrate
```

### Disruptive changes

**1. image/file upload form `removeable` renamed to `removable`**

```php
$form->file('...')->removable();
```

### Feature improvement

**1.Support PHP8.0**

**2. Image/file upload form supports listening to WebUploader events**

The `on` method can listen to [WebUploader file upload related events](http://fex.baidu.com/webuploader/doc/index.html#WebUploader_Uploader_events)

```php
$form->file('...')
	->on('startUpload', <<<JS
		function () {
			console.log('File upload started...', this);
			
			// Attach custom parameters to the file upload interface before uploading files
			this.uploader.options.formData['custom_field'] = '...';
		}
JS
    )
	->on('uploadFinished', <<<JS
    	function () {
    		console.log('File upload is complete');
    	}
JS
    );
    
//       
```

**3. Listen for changes when a file is successfully uploaded or deleted**

You can listen for changes when a file **is uploaded successfully** or when a file **is deleted** by the following methods

```php
$file = $form->file('...');

Admin::script(
	<<<JS
$('{$file->getElementClassSelector()} .file-input').on('change', function () {
	console.log('Document changed', this.value);
});
JS
);
```

**4. Allow intercepting and responding to error messages in the uploading event**

Starting from this version, we support intercepting file uploads in the `uploading` event of the form and responding to error messages on the front-end.

```php
$form->uploading(function (Form $form) {
	return $form->response()->error('File upload failed, please try again!');
});
```



**5. Listen to selectTable selected value change**

```php
$selectTable = $form->selectTable('...')->from(...);

Admin::script(
	<<<JS
$('{$selectTable->getElementClassSelector()} input[type="hidden"]').on('change', function () {
	console.log('Change of selected values', this.value);
});
JS
);
```

**6. adjust the tree form no data return, cancel the return of 404 status code **

**7. Adjust the value type of row attribute of table displayer class to model**

**8. Dark mode details optimization**

```php
$grid->column(...)->modal(function () {
    // $this points to the model object
    dd($this);
});

$grid->actions(function () {
    // $this points to the model object
    dd($this);
});
```


**9. Optimize the problem of overflowing chart display in cards**

[#822](https://github.com/jqhph/dcat-admin/pull/822)


**10.widget component add when method**

```php
$modal = Dcat\Admin\Widgets\Modal::make()
	->when($condition, function ($modal) {
		// When the value of $condition is true, the logic inside the closure will be executed
	    $modal->xl();
	})
	->body(...)
	->render();
```



### Bug fixes

1. fix `Grid\Filter::group` can't keep selected state [#739](https://github.com/jqhph/dcat-admin/pull/793)
2. fix `Form::hasMany` problem of validating `required` even after form entry is deleted [#795](https://github.com/jqhph/dcat-admin/pull/795)
3. fix the problem that map form cannot be used
4. fix the problem that `guessClassFileName` will report error when `composer` class mapping file is generated and the class file is deleted
5. fix the problem of error when using `Fetched` event for data export [#815](https://github.com/jqhph/dcat-admin/issues/815)
6. fix the problem that `filter` cannot be reset after setting `Grid name`.
7. fix the problem that `select2` can't use Chinese language pack automatically [#839](https://github.com/jqhph/dcat-admin/issues/839)
8. fix the problem that `continue to create` and `continue to edit` jump route error [#814](https://github.com/jqhph/dcat-admin/issues/814)
9. fix `range` form setting `rules` invalid when one-to-one association is enabled
10. fix the problem that time filtering dropdown will be blocked when `fixColumns` is enabled [#833](https://github.com/jqhph/dcat-admin/issues/833)
11. fix the problem that menu `fa` icon can't be aligned automatically [#758](https://github.com/jqhph/dcat-admin/pull/758)
12. fix the problem of submitting errors with `hasMany` in `row` layout [#801](https://github.com/jqhph/dcat-admin/issues/801)
13. fix form `hasMany` can't use `select` linkage [#769](https://github.com/jqhph/dcat-admin/issues/769)



## v2.0.11-beta

Release date 2020/12/06

To upgrade, step-by-step execute the following command
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin: "2.0.11-beta"
php artisan admin:publish --assets --force
php artisan admin:publish --migrations --force # table structure changes
php artisan migrate
```


### Bug fixes

1. fix `Dcat.init` listener failure due to repeated page refreshes using `pjax`
2. fix `admin:export-seed --users` which generates redundant code
3. repair the problem of abnormal jumping after saving the form editing page
4. repair the problem that `hasMany` will add data repeatedly if you choose to continue editing on the form page
5. fix the problem that the original `select2 config` is lost after linking `select` forms [#779](https://github.com/jqhph/dcat-admin/issues/779)
6. fix `map` form loading exception problem [#764](https://github.com/jqhph/dcat-admin/issues/764)
7. fix the problem that the page cannot be refreshed automatically after the data is deleted by the batch delete function
8. fix the problem that `hasMany` form editing page cannot display `row` and `column` layouts properly
9. fix the problem that `Dcat.init` listener will be unbound by asynchronous pop-up window
10. fix a bug that the table toolbar dropdown menu is blocked by the fixed list pane [#728](https://github.com/jqhph/dcat-admin/issues/728)
11. fix a problem where cached content is still read when `showColumnSelector` is disabled

### Functional improvements

**1. Add title parameter to Form::divider**.

Add `title` parameter to display the title in the middle of the divider, usage

``` php
$form->divider('title');
```


**2. Grid::footer and Grid::header adjusted to support multiple callbacks**

``` php
$grid->header(...) ;

$grid->header(...) ;

$grid->header(...) ;
```

**3. Optimize form specification filters and select form styles**


## v2.0.10-beta


Posted on 2020/11/29

To upgrade the method, execute the following step-by-step commands
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.10-beta"
php artisan admin:publish --assets --force
php artisan admin:publish --migrations --force # Table structure changed.
php artisan migrate
```
### Functional improvements

**1. Add a prompt window in the upper right corner of the form to display field validation error messages**.

This feature is enabled by default and can be disabled by the `validationErrorToastr` method.

```php
$form->validationErrorToastr(false);
```

**2.Add Tree::maxDepth method to limit the maximum level of the model tree **.

```php
$tree->maxDepth(5);
```

**3. Optimize the export function, support title settings associated with the relational fields and automatically read the title of the grid column **.

In the current version, exported columns are the same as `column` columns by default, so you no longer need to set the exported column name and translation manually, and the associated relational fields are supported.

```php
$grid->column('id');
$grid->column('name');
...

// Default is the same as the column above
$grid->export();
```

**4. Add a resetButton and submitButton method to the tool form**.

```php
// Disable the Reset and Submit buttons
$form->resetButton(false);
$form->submitButton(false);
```

**5. Add parameters to the `disable` and `readOnly` methods of the form fields to control whether they are enabled or not**.

```php
// pass false to disable
$form->text(...)->disable(false);
```

**6. Adding `withDeleteData` to file uploads allows users to set request parameters and add primary key fields to the upload and delete interfaces**.

The `withDeleteData` method allows you to pass custom parameters to the file delete interface.

```php
$form->file(...)->withDeleteData(['key' => 'value]);
```

**7. Add `embeds` form to disable display of titles**.

The second parameter, passed as `false`, does not display the title.

```php
$form->embeds('field', false, function ($form) {
    ...
});
```

**8. Rewrite some of the unit test cases to support 2.x usage**.

### Bug fixes

1. fix the problem that existing permissions cannot be selected on admin detail page.
2. fix the problem of `admin:export-seed` command exporting `seeder` class name exception.
3. fix the form deletion jump exception
4. fix the problem of form jumping exception when continuing to edit.
5. fix the problem that parent table fields cannot be saved when parent table and `hasMany` have the same field name.
6. fix the problem that the style of selected submenu is abnormal in dark mode [#712](https://github.com/jqhph/dcat-admin/issues/712)
7. fix the problem that form dynamic display function is invalid under form `block` layout [#723](https://github.com/jqhph/dcat-admin/issues/723)
8. optimize the display of `selectOptions` hierarchy and solve the problem of prefix rendering increasing with the hierarchy depth index [#618](https://github.com/jqhph/dcat-admin/issues/618)
9. fix the problem that `admin_view` does not return data.
10. fix the problem that links set by `select` form, `ajax` and `load` cannot take parameters [#745](https://github.com/jqhph/dcat-admin/issues/745)
11. fix the problem that the `handle` method of the table row operation `action` can only get `id` of the last row of data.
12. fix the problem that `list` form edit page cannot delete existing data [#759](https://github.com/jqhph/dcat-admin/issues/759)
13. Fix the error in the `embeds` scope form's `name` attribute.
    
## v2.0.9-beta

Posted on 2020/11/18

To upgrade the method, execute the following step-by-step commands
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.9-beta"
php artisan admin:publish --assets --force
php artisan admin:publish --migrations --force # 表结构有变动
php artisan migrate
```


**Bug Fix**.

1. fix the function failure of form `filter::select` form remote load `/load/ajax` etc.
2. fix the front-end `moment-timezone` component path loading error [#701](https://github.com/jqhph/dcat-admin/issues/701)
3. fix the problem of not being able to set permissions due to `Form::tree` not being able to save data.
4. fix the problem of filling default value exception when the `hasMany` form has the same field name as the parent table.
5. repair the problem of adding new page report when form `tab` layout is nested with `row` layout [#648](https://github.com/jqhph/dcat-admin/issues/648)
6. fix the problem of not being able to get all the values under `range` after submission when the form has `range` type field.
7. fix the problem that select2 component is invalid when `Form::select` uses form linkage.


## v2.0.8-beta

Posted on 2020/11/16

To upgrade the method, execute the following command step by step
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.8-beta"
php artisan admin:publish --assets --force
php artisan admin:publish --migrations --force # Table structure changed
php artisan migrate
```

As a supplement to `2.0.7-beta`, the following issues are fixed in this release

1. fix the problem of not being able to view permissions on the admin page
2. fix the problem of form block layout failure
3. fix the problem of abnormal initialization of file upload form.
4. fix the problem that some form fields don't support rendering multiple fields on a single page at the same time.


## v2.0.7-beta

Posted on 2020/11/15

To upgrade the method, execute the following command step by step
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.7-beta"
php artisan admin:publish --assets --force
php artisan admin:publish --migrations --force # Changes in table structure
php artisan migrate
```

**Functional improvements**

1. Introduce the [jquery.initialize](https://github.com/pie6k/jquery.initialize) component to listen to dynamically generated page elements and set a callback, the following is a simple example to demonstrate usage.

In older versions, if an element is dynamically generated by `JS`, and we need to bind a click event to that element, then we usually do this

```html
<div class="selector">test</div>

<script>
Dcat.ready(function () {
    // You need to be off first and then on, otherwise page refresh will cause double bind problem.
    $(document).off('click', '.selector').on('click', '.selector', function () {
        ...
    })
});
</script>
```

The above approach is troublesome, you need to `off` and then `on`; secondly, you can not do some special treatment for dynamically generated elements, for example, if you want to change the background color after `.selector` is generated, there is no way to do this.

In this version we can use the `Dcat.init` method to listen to the dynamically generated elements.

```html
<div class="selector">test</div>

<script>
Dcat.ready(function () {
    // $this is the jquery dom object of the current element.
    // id is the id attribute of the current element, if the current element has no id, a random id will be generated automatically.
    Dcat.init('.selector', function ($this, id) {
        // Change the background color of the element.
        $this.css({background: "#fff"});
        
        // There's no need for off and then on again, because the anonymous function will only be executed once!
        $this.on('click', function () {
            ...
        });
    });
});
</script>
```

Thanks to the introduction of the [jquery.initialize](https://github.com/pie6k/jquery.initialize) component, in the current version we have optimized the front-end code of the form component to support the dynamically generated form type `HasMany` more easily. Significantly reduces the complexity of the code.


2.Form::hasMany and Form::array forms support column and row layout

If there are more fields, you can use `column` and `row` layout to save space.

```php
$form->array('field', function (NestedForm $form) {
    $form->column(6, function (NestedForm $form) {
        $form->text('...');
        
        ...
    });
    
    $form->column(6, function (NestedForm $form) {
        ...
    });
});
```

3.Routes configured with the admin.auth.except parameter do not require authentication privileges [#673](https://github.com/jqhph/dcat-admin/issues/673)


4. Add when method to Form, Grid and Show field classes

Usage examples, similar to `Laravel QueryBuilder`s `when` method

in the table
```php
// Closing code will be executed when the first parameter is true.
$grid->column('title')->when(true, function (Grid\Column $column, $value) {
    $column->label();
});
```

form (document)
```php
// Closing code will be executed when the first parameter is true.
$form->text('email')->when(true, function (Form\Field\Text $text, $value) {
    $text->type('email');
});
```

5. Administrator model add canSeeMenu method to control whether the menu is visible or not.

```php
<?php

namespace App\Models;

use Dcat\Admin\Models\Administrator as Model;

class Administrator extends Model
{
    /**
     * :: Control whether the menu is visible or not, return true by default
     * 
     * @param array|\Illuminate\Database\Eloquent\Model $menu menu node
     * @return bool
     */
    public function canSeeMenu($menu)
    {
        return true;
    }
}
```

6.Add admin_script、admin_style、admin_js、admin_css and admin_require_assets functions

```php
// Equivalent to Admin::script
admin_script('console.log(xxx)');

// Equivalent to Admin::style
admin_style('.my-class {color: red}');

// Equivalent to Admin::js() 
admin_js(['@admin/xxx.js']);

// Equivalent to Admin::css() 
admin_css(['@admin/xxx.css']);

// Equivalent to Admin::requireAssets() 
admin_require_assets(['@select2']);
```

7. simplify the action (Action) of the `JS` code logic, to remove the memory of the `selector` function

**BUG FIX**

1. Fix anomaly in the orderable form [#674](https://github.com/jqhph/dcat-admin/issues/674)
2. fix the JsonResponse methodIf error.
3. fix table, form, and data detail specifying `label` [#684](https://github.com/jqhph/dcat-admin/issues/684)
4. fix the problem that the table `Grid::rows` callback doesn't work properly.
5. fix the problem of some types of statistical cards failing to load asynchronously due to exceptions in adding `JS` code to widgets.
6. fix the getKey method exception for table row operations [#691](https://github.com/jqhph/dcat-admin/issues/691)
7. fix the problem of not being able to use the linkage function when there are multiple select forms in the page.
8. Fix the problem of the table not being able to refresh automatically after deleting data.


## v2.0.6-beta

Posted on 2020/11/7

To upgrade the method, execute the following command step by step
```bash
composer remove dcat/laravel-admin
composer require dcat/laravel-admin:"2.0.6-beta"
php artisan admin:publish --assets --force
php artisan admin:publish --migrations --force # 表结构有变动
php artisan migrate
```

**Breaking Changes**

1.`Form::tags` form is saved as `array` type by default
```php
// You need to convert the format you save to the database yourself
$form->tags('tag')->saveAsJson();
```

2.The session middleware is disabled by default

3.`Form\Tree::disableFilterParents` renamed to `Form\Tree::exceptParentNode`
```php
$form->tree('cate')->exceptParentNode(false);
```

4. Adjust the method name of the file upload form part
```php
// Enable chunked uploads, disableChunked changed to chunked
$form->image('avatar')->chunked(true);

// Enable auto-save field values, disableAutoSave changed to autoSave
$form->image('avatar')->autoSave(false);

// Enable the file deletion function, disableRemove changed to removeable
$form->image('avatar')->removeable(false);
```


**Functional improvements**

1.Code generator add field dragging sorting function, this method is contributed by partner [@codingyu](https://github.com/codingyu).

2. menu table to add `show` and `extension` field, `show` field is used to control whether to display the menu; `extension` field is used to mark whether to expand the menu

3.`Form::table`、`Form::array`、`Form::embeds` supports relational fields
```php
$form->table('profile.options', function ($form) {
    ...
});
```

4. Add vertical display of `Form::checkbox` and `Form::radio` form options.
```php
$form->checkbox('xxx')->inline(false)->options([...]);
```

5. Configuration file skip login and permission authentication allows configuration of routing aliases.
```php
'auth' => [
    'except' => [
        ...
        'user.login',
    ],
],
```

6.`Form\Row` adds `getKey` and `model` methods

7. Optimize the form filter select form selection effect, the default is not selected

8. Form tab layout optimization


**BUG FIX**
1. fix `Form::checkbox` problem when check/uncheck all options.
2. fix the problem that the default menu TITLE can not be translated in Taiwan Traditional Chinese. 
3. fix the problem that the `number` field in `NestedForm` is filtered when the input value is 0 [#634](https://github.com/jqhph/dcat-admin/issues/634)
4. fix the problem of getting primary key error when the model tree `RowAction` asynchronously processes the interface.
5. Fix the problem of table filters failing to reset the search value of the associated table fields [#650] (https://github.com/jqhph/dcat-admin/issues/650)
6. fix form filter multipleSelect form exception problem


## v2.0.5-beta 

Posted on 2020/10/29

BUG FIXES
1. fix the problem of table search multiple related table fields sql error [I232T7](https://gitee.com/jqhph/dcat-admin/issues/I232T7)
2. fix the problem of `Form::datetimeRange` form not being able to select logs.
3. Fix the problem of not being able to add multiple `Form::table` form fields [#627](https://github.com/jqhph/dcat-admin/issues/627)
4. fix the error in the form filter MultipleSelectTable.
5. Fix the problem of abnormal style of table specification filter.


## v2.0.4-beta 

Posted on 2020/10/27

BUG FIXES
1. fix the problem that the admin_javascript_json function will automatically empty-filter an array of null values.
2. fix the error of using tab layout for data form [#620](https://github.com/jqhph/dcat-admin/issues/620)
3. fix the problem of abnormal permissions for temporary directories generated by extended local installation [#625](https://github.com/jqhph/dcat-admin/issues/625)
4. fix the error when using html method to set view with script tag [#624](https://github.com/jqhph/dcat-admin/issues/624).
5. Fix the error of displaying data details using correlations (one-to-many) [#623](https://github.com/jqhph/dcat-admin/issues/623)
6. fix the dropdown menu calculation display position exception [#I22S2N](https://gitee.com/jqhph/dcat-admin/issues/I22S2N)


## v2.0.3-beta 

Posted on 2020/10/27

BUG FIXES
1. fix the problem of abnormal display of return information in form row editing
2. Fix invalid setting of `admin.auth.member` [#613](https://github.com/jqhph/dcat-admin/issues/613)
3. fix the abnormal Chinese translation problem of `editor` form [#611](https://github.com/jqhph/dcat-admin/issues/611)
4. fix the problem that `Filter::scope` can't filter pagination parameters when selecting filter item.
5. fix issues related to form event interception
6. Fix the problem of abnormal use of tree form [#619](https://github.com/jqhph/dcat-admin/issues/619)

Functional improvements
1. add the configuration of skip privileges and login authentication
2. Extending the service provider to include middleware and route-checking registrations
3. batch operation change event monitoring optimization
4. Increase `RowSelector` robustness to avoid errors due to unprocessed `json` array type fields [#609](https://github.com/jqhph/dcat-admin/pull/609)

## v2.0.2-beta 

Posted on 2020/10/21

BUG FIXES
1. Fix naming space exception in controller file generated by code generator #600 
2. fix the problem of configuration file logo path error
3. Fix the problem of invalid search for associated fields in tables
4. fix the problem of duplicate selectors generated by model tree operation
5. fix the problem of accessing permissionless page report
6. Fix the problem that table filter multipleSelect cannot select the value of the associated table field #603 
7. Fix invalid form tab layout #605 

Functional improvements
1. Auth\Permission to move to Http directory
2. Replace the json field in the data table with text
3. add login password error translation
4. add admin_javascript_json function, make most of the component configuration support passing JS code
5. Admin::color Add dark mode color.




## v2.0.1-beta 

Posted on 2020/10/20

BUG FIXES
- Fix data table filter search bug #599 
- Fix code generator error in generating controller base class namespace #600 

Functional improvements

- Code Generator adds page TITLE and breadcrumb translation functionality.
- Exception handling optimization
- Add admin_setting_array function.