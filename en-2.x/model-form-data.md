# Form data source

## Models and data warehousing


Repository (`Repository`) is a class that can provide a specific interface to read and write operations on the data, through the introduction of repositories, you can make the construction of the page no longer care about the specific implementation of data read and write functions, developers only need to implement a specific interface to easily switch data sources. For detailed usage of the data warehouse, please refer to the document [Repository](model-repository.md).

## Data from the model

> {tip} If your data comes from `Model`, then you can also use the `Model` instance directly, and the underlying layer will automatically convert `Model` into a repository instance.



When the data source supports the model, it is sufficient to create a very simple `Repository' class.


```php
<?php

namespace App\Admin\Repositories;

use Dcat\Admin\Repositories\EloquentRepository;
use App\Models\Movie as MovieModel;

class Movie extends EloquentRepository
{
    // Define your model class name here
    protected $eloquentClass = MovieModel::class;
    
    // With this method, you can specify the fields of the form page query, default "*"
    public function getFormColumns()
    {
        return [$this->getKeyName(), 'name', 'title', 'created_at'];
    }
}
```
usage:
```php
use App\Admin\Repositories\Movie;

$form = new Form(new Movie);

...
```

## Data from external API

The following is an example of the `Douban movie` API to demonstrate the usage of the `Repository` interface related to read and write operations on form data.

```php
<?php
namespace App\Admin\Repositories;

use Dcat\Admin\Repositories\Repository;
use Dcat\Admin\Form;

class ComingSoon extends Repository
{
    protected $api = 'https://api.douban.com/v2/movie/coming_soon';
    
    // Returns the name of your id field, default "id"
    protected $keyName = '_id';

    // Query edit page data
    // This method needs to return an array.
    public function edit(Form $form): array
    {
        // Get id
        $id = $form->builder()->getResourceId();

        $data = file_get_contents("http://api.douban.com/v2/movie/subject/$id");

        return json_decode($data, true);
    }
    
    // This method is used to query the original record before modifying the data.
    // If a file upload form is used, the old file will be automatically deleted based on this original record when the file is changed.
     // If this data is not needed, just return an empty array.
    public function getDataWhenUpdating(Form $form): array
    {
        // Get id
        $id = $form->builder()->getResourceId();
        
        return [];
    }

    // Modify operation
    // Returns a bool type data.
    public function update(Form $form)
    {
        // Get id
        $id = $form->builder()->getResourceId();

        // Get the data to be modified
        $attributes = $form->updates();

        // TODO
        // Write your modification logic here.
        
        return true;
    }

    // This method is used to query the original data before modifying it.
    // If a file upload form is used, files are automatically deleted based on this data.
    // If this data is not needed, just return an empty array.
    public function getDataWhenDeleting(Form $form): array
    {
        $id = $form->builder()->getResourceId();
        
        $id = explode(',', $id);

//        $data = file_get_contents("http://api.douban.com/v2/movie/subject/$id");
//
//        return json_decode($data, true);

        return [];
    }

    // Delete data
    // $deletingData is the data returned by the getDataWhenDeleting method.
    public function destroy(Form $form, array $deletingData)
    {
        // Note that there may be multiple ids here.
        $id = $form->builder()->getResourceId();
        
        // When using the batch delete function, the id here is a string separated by ",".
        $id = explode(',', $id);

        // TODO
//        var_dump($id, $deletingData);

        return true;
    }

}
```


