# Form callbacks

`Form` currently provides the following methods for receiving callback functions:

### creating

Calling on a new page (non-committed action)

```php
$form->creating(function (Form $form) {
    if (...) { // verification logic
		$form->responseValidationMessages('title', 'Wrong title format');
		
		// If there are multiple error messages, the second parameter can be passed as an array.
		$form->responseValidationMessages('content', ['content format error', 'content cannot be empty']);
	}
});
```

### editing

Invoked on the edit page (non-commit operation)

```php
$form->editing(function (Form $form) {
    if (...) { // verification logic
		$form->responseValidationMessages('title', 'Wrong title format');
		
		// If there are multiple error messages, the second parameter can be passed as an array.
		$form->responseValidationMessages('content', ['content format error', 'content cannot be empty']);
	}
});
```

### submitted

Called before the form is submitted, this event allows you to modify or delete the data submitted by the user or abort the submitted operation.

```php
$form->submitted(function (Form $form) {
    // Get user submission parameters
    $title = $form->title;
    
    // The above is equivalent to
    $title = $form->input('title');
    
    // Delete user-submitted data
    $form->deleteInput('title');
    
    // Interruption follow-up logic
    return $form->response()->error('The server is down.~');
});
```

### saving
A callback before saving, in which you can modify or delete the data submitted by the user or interrupt the commit operation.

```php
$form->saving(function (Form $form) {
    // Determine if it is a new operation
    if ($form->isCreating()) {
    
    }
    
    // Delete user-submitted data
    $form->deleteInput('title');
    
    // Interruption follow-up logic
    return $form->response()->error('The server is down.~');
});
```

### saved

This event is shared by both the add and modify operations. The second parameter `$result` is used to determine if the data is stored in success.

> {tip} The value of `$result` under the Add page is the self-added ID of the new record.

```php
$form->saved(function (Form $form, $result) {
    // Determine if it is a new operation
    if ($form->isCreating()) {
    	// Self-added ID
    	$newId = $result;
    	// You can also get a self-added ID like this
    	$newId = $form->getKey();
    	
    	if (! $newId) {
    		return $form->error('Data save failed');
    	}
    	
    	return;
    }
    
    // Modification operations
});
```
> {tip} `$form->repository()->eloquent()` to get the currently added or edited eloquent model.

```php
$form->saved(function (Form $form, $result) {
    // Get eloquent after the form is saved
    $form->repository()->eloquent()->update(['data' => 'new']);
});
```

### deleting

Callback before deletion

```php
$form->deleting(function (Form $form) {
    // Gets the data for the deleted rows, in this case a two-dimensional array.
	$data = $form->model()->toArray();
});
```

### deleted

The second parameter `$result` is used to determine if the data is deleted successively.

```php
$form->deleted(function (Form $form, $result) {
	// Gets the data for the deleted rows, in this case a two-dimensional array.
	$data = $form->model()->toArray();
	
	// With $result, you can determine if the data is deleted from the successor.
	if (! $result) {
		return $form->response()->error('Data deletion failure');
	}

    // Return to delete success reminder, where the jump parameter is invalid.
    return $form->response()->success('Delete success');
});
```

### uploading

Image, file upload events

> {tip} The file upload is a separate api request and the `redirect` method is invalid in this event.

```php
use Dcat\Admin\Form;
use Dcat\Admin\Form\Field;
use Dcat\Admin\Contracts\UploadField as UploadFieldInterface;
use Symfony\Component\HttpFoundation\File\UploadedFile;

$form->uploading(function (Form $form, UploadFieldInterface $field, UploadedFile $file) {
	// $file is the currently uploaded file.
	
	/* @var Field $field */
	// Get the name of the file upload field
	$column = $field->column();
});
```

### uploaded

Image/file upload completion event

> {tip} The file upload is a separate api request and the `redirect` method is invalid in this event.

```php
use Dcat\Admin\Form;
use Dcat\Admin\Form\Field;
use Dcat\Admin\Contracts\UploadField as UploadFieldInterface;
use Symfony\Component\HttpFoundation\File\UploadedFile;

$form->uploaded(function (Form $form, UploadFieldInterface $field, UploadedFile $file, $response) {
	// $file is the currently uploaded file.
	
	/* @var Field $field */
	// Get the name of the file upload field
	$column = $field->column();
	
	$response = (array) $response->getData();
	
	// File Uploadsuccess
	if ($response['status']) {
		// file access address
		$url = $response['url'];
	}
});
```

### Access to data in the model
```php
$form->saved(function (Form $form) {

    $id = $form->getKey();

    $username = $form->model()->username;
    
    // Get the final saved array
    $updates = $form->updates();
});
```

### Modify or delete user-submitted data

This feature is valid for the `saving` and `submitted` events.

```php
$form->select('author_id');

$form->saving(function (Form $form) {
    // Modifying user-submitted data
    $form->author_id = 1;
    
    // Delete, ignore data submitted by users
    $form->deleteInput('author_id');  
});
```

### Modify the data in the model
Modifying the data in the model needs to be used in conjunction with the hidden form. For example:
```php
$form->hidden('author_id');

$form->saving(function (Form $form) {

    $form->author_id = 1;
});
```

### Form response

> {tip} This method is not available in `creating` and `editing` events.

Refer to the [Actions and Form Responses](response.md) section of the documentation for detailed usage.


redirect (partial refresh / single page refresh)
```php
// Jump and prompt for success information
$form->saved(function (Form $form) {
    return $form->response()->success('Save success')->redirect('auth/user');
});

// Jump and error messages
$form->saved(function (Form $form) {
    return $form->response()->error('system error')->redirect('auth/user');
});
```

Returns only error messages without skipping

```php
$form->saving(function (Form $form) {
    return $form->response()->error('system anomaly');
});
```

### Return field validation error message

The `responseValidationMessages` method makes it easy to return field validation error messages without using the `Laravel validation` feature.

General Use
```php
protected function form()
{
	return Form::make(new Model(), function (Form $form) {
		if (...) { // verification logic
			$form->responseValidationMessages('title', 'Wrong title format');
			
			// If there are multiple error messages, the second parameter can be passed as an array.
			$form->responseValidationMessages('content', ['content format error', 'content cannot be empty']);
		}
	});
}
```

Use in events
> {tip} This method is only available for the `submitted` event.

```php
$form->submitted(function (Form $form) {
	// Receive form parameters
	$title = $form->title;

    if (...) { // verification logic
        $form->responseValidationMessages('title', 'Wrong title format');
        
        // If there are multiple error messages, the second parameter can be passed as an array.
        $form->responseValidationMessages('content', ['content format error', 'content cannot be empty']);
    }
});
```
