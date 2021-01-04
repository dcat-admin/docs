# Step-by-step form

## Installation

Go to [https://github.com/dcat-admin/form-step](https://github.com/dcat-admin/form-step) to download the step-by-step form extension, then install and enable it.

> {tip} To install the extension, please refer to the [Basic Use of Extensions] (extension-f.md) section of the documentation.

## Simple example

```php
protected function form()
{
    return Form::make(new Model(), function (Form $form) {
        $form->title('Step-by-Step Form');
        $form->action('step');
        $form->disableListButton();
    
        $form->multipleSteps()
            ->remember() // remember form steps, not enabled by default
            ->width('950px')
            ->add('General Information', function ($step) {
                $info = '<i class="fa fa-exclamation-circle"></i> The form fields support a mix of front-end and back-end validation, and front-end validation supports html form validation and custom validation.';
    
                $step->html(Alert::make($info)->info());
    
                $step->text('name', 'Fullname')->required()->maxLength(20);
                // h5 Form Validation
                $step->text('age', 'Age')
                    ->required()
                    ->type('number')
                    ->attribute('max', 150)
                    ->help('Front-end validation');
    
                $step->radio('sex', 'Sex')->options(['rather not say', 'male', 'woman'])->default(0);
    
                // Back-end validation
                $step->text('birthplace', 'place of ancestry')
                    ->rules('required')
                    ->help('Demonstrate back-end field validation');
    
                $step->url('homepage', 'personal homepage');
    
                $step->textarea('description', 'INTRODUCTION');
    
            })
            ->add('Hobbies', function ($step) {
                $step->tags('hobbies', 'preferences')
                    ->options(['sing', 'dance', 'rap', 'play football'])
                    ->required();
    
                $step->text('books', 'Book');
                $step->text('music', 'Music');
    
                // events
                $step->shown(function () {
                    return <<<JS
    Dcat.info('hobbies');
    console.log('hobbies', args);
    JS;
                });
    
            })
            ->add('Address', function ($step) {
                $step->text('address', 'street address');
                $step->text('post_code', 'zip code');
                $step->tel('tel', ' telephone number');
            })
            ->done(function () use ($form) {
                $resource = $form->getResource(0);
    
                $data = [
                    'title'       => 'Operating Success',
                    'description' => 'Congratulations on being the 10086th user!',
                    'createUrl'   => $resource,
                    'backUrl'     => $resource,
                ];
    
                return view('admin::form.done-step', $data);
            });
    });
}
```

result

<a href="{{public}}/assets/img/screenshots/step-form-1.png" target="_blank">
    <img style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" src="{{public}}/assets/img/screenshots/step-form-1.png">
</a>

## Running logic

Step-by-step form is very simple to use, and the running logic is not much different from the ordinary form, the following is a brief description of step-by-step form running logic.

> {tip} Step-by-step forms do not have the concept of `update`.

#### Parameter validation
When the user clicks `Next`, a request will be sent to the backend to verify if the parameter is correct or not, and an error message will be displayed if the parameter does not meet the requirement.

#### form submission
If the parameters do not meet the requirements, it will display an error message and save the form data if the validation is passed.

> {tip} The final method to save the form is `Form::store`.

#### complete page

This step cannot be ignored as the completed page will be displayed after the form is saved.


## Edit Form

Step-by-step form has no editing function by default, users do not need to edit step-by-step after entering a long step form, so if you need to edit step-by-step form, you can refer to the following way

```php
protected function form()
{
    return Form::make(new MyRepository(), function (Form $form) {
        // Determine if it is an edit page
        if ($form->isEditing()) {
            $form->text('age', 'Age')
                 ->required()
                 ->type('number')
                 ->attribute('max', 150)
                 ->help('Front-end validation')
        
            ...
            
            return;
        }
    
    
        $form->multipleSteps()
            ->remember()
            ->width('950px')
            ->add('General Information', function (Form\StepForm $step) {
                $info = '<i class="fa fa-exclamation-circle"></i> The form fields support a mix of front-end and back-end validation, and front-end validation supports H5 form validation and custom validation.';
    
                $step->html(Alert::make($info)->info());
    
                $step->text('name', 'Fullname')->required()->maxLength(20);
                // h5 Form Validation
                $step->text('age', 'Age')
                    ->required()
                    ->type('number')
                    ->attribute('max', 150)
                    ->help('Front-end validation');
    
                $step->radio('sex', 'Sex')->options(['rather not say', 'male', 'woman'])->default(0);
    
                ...
    
            })
            ->add('hobbies', function (Form\StepForm $step) {
                ...
            })
            ->done(function () use ($form) {
                ...
            });
    });
}
```

## Functional interface

### Set the maximum container width

默认 `1000px`。

> {tip} This method is only for large screens and the mobile page will automatically resize.

```php
$form->multipleSteps()->width('900px');
```

### Remembering form data

When this feature is enabled, when the user clicks the `Next` button and the parameters are validated, the form data will be saved in `session` and will not be destroyed until the entire step-by-step form is saved.

> {tip} This feature is not enabled by default.

```php
// turn on
$form->multipleSteps()->remember();

// turn off
$form->multipleSteps()->remember(false);
```

### Set container spacing

Default value `30px 18px 30px`.

```php
$form->multipleSteps()->padding('30px 18px 30px');
```

### Listening for page exit events

Listen for all step form page leave events, multiple can be added.

```php
$form->multipleSteps()->leaving(<<<JS
     // Get the step index of the current page
     var index = args.index; 
                 
     Dcat.info("You are leaving the " + (index + 1) + " page");
     
     // The args variable is a js object that contains the current event object, current step option, form object and form value fields.
     console.log("leaving", args);
     
     // Get the current event object
     var evt = args.event;
     // Get the step form TITLEtab object
     var tab = args.tab;
     // Whether to go to the previous or next page to get the movement: "forward", "backward"
     var direction = args.direction;
     // Get the form JQ object for the current step
     var $form = args.form;
     // Get the form values for the current step page
     var formArray = args.formArray;
     
     // Get the form JQ object for the specified step
     var $firstForm = args.getForm(0);
     // Get form values for a specified step
     var firstFormArray = args.getFormArray(0);
     
     // Stop leaving the current page
     return false;
JS    
)->leaving(...);
```

### Listening for page display events

Listen to all step form page display events, multiple can be added.

```php
$form->multipleSteps()->shown(<<<JS
     // Get the step index of the current page
     var index = args.index; 
                 
     Dcat.info("The current display is no. " + (index + 1) + " page");
     
     // The value of the args variable is the same as the value of the "leaving" event.
     console.log("shown", args);
JS    
)->shown(...);
```

### Adding step sheets

Step forms support all form fields.

```php
use Dcat\Admin\Form;

$form->multipleSteps()->add('TITLE', function (Form\StepForm $step) {

    $step->text('username')->rules('required');
    
    ...
});
```

#### Listening for step page leaving events

Listen to the current step page leave event, with multiple listeners.

```php
use Dcat\Admin\Form;

$form->multipleSteps()->add('General Information', function (Form\StepForm $step) {
    ...
    
    $step->leaving(<<<JS
    
    Dcat.info("You're Leaving General Information Page");
    
    // The args variable is a js object that contains the current event object, current step option, form object and form value fields.
    console.log("Leave General Information", args);
    
    // Get the current event object
    var evt = args.event;
    // Get the step index of the current page
    var index = args.index;
    // Get the step form TITLEtab object
    var tab = args.tab;
    // Whether to go to the previous or next page to get the movement: "forward", "backward"
    var direction = args.direction;
    // Get the form JQ object for the current step
    var $form = args.form;
    // Get the form values for the current step page
    var formArray = args.formArray;
    
    // Get the form JQ object for the specified step
    var $firstForm = args.getForm(0);
    // Get form values for a specified step
    var firstFormArray = args.getFormArray(0);
    
    // Stop leaving the current page
    return false;
JS    
    );
    
    // Multiple listeners
    $step->leaving(...);
});
```

#### Listening for step page display events

Listen to the current step page to display events, and listen multiple times.

```php
use Dcat\Admin\Form;

$form->multipleSteps()->add('General Information', function (Form\StepForm $step) {
    ...
    
    $step->shown(<<<JS
    
    Dcat.info("The current steps is General Information");
    
    // The value of the args variable is the same as the value of the "leaving" event.
    console.log("Show General Information", args);
JS    
    );
    
   // Multiple listeners
    $step->shown(...);
});
```

### Setting up the completion page

When all the steps are completed the completion page will be displayed, the system provides a default completion page, the developer can also customize the content to be displayed on the completion page by the following methods.

```php
use Dcat\Admin\Form;

$form->multipleSteps()->done(function (Form\DoneStep $done) {
    
    // Get the new ID
    // The value returned by Repository::store
    $newId = $done->getNewId();
    
    // Returns what you want to display, which can make a view also a string.
    return view(...);
});
```


