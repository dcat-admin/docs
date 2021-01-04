# Picture/file upload

The [data form](model-form.md) generates an image/file upload form with the following calls, supporting both local and cloud storage file uploads. The uploading component is based on [webuploader](https://fex.baidu.com/webuploader/), please refer to the [webuploader official documentation](https://fex.baidu.com/webuploader/document) for specific configuration. .html)

> {tip} Please do not set the **accessor** and **mutator** splicing domain name in the file or image upload form fields in the model, please refer to [File/Image Domain Splicing](#withhost) for related requirements.

```php
$form->file('file_column');
$form->image('image_column');
```

<a name="local"></a>
## Local Upload

First add the storage configuration, `config/filesystems.php` and add a `disk`:

```php

'disks' => [
    ... ,

    'admin' => [
        'driver' => 'local',
        'root' => public_path('uploads'),
        'visibility' => 'public',
        'url' => env('APP_URL').'/uploads',
    ],
],

```

Set the upload path to `public/uploads`(public_path('uploads')).

Then select the uploaded `disk`, open `config/admin.php` and find

```php
    
'upload'  => [

    'disk' => 'admin',

    'directory'  => [
        'image'  => 'images',
        'file'   => 'files',
    ]
],

```

Set `disk` to `admin` added above, and `directory.image` and `directory.file` to the upload directories for images and files uploaded with `$form->image($column)` and `$form->file($column)`, respectively.

Of course, you can also specify `disk` in the code:
```php
$form->file('file')->disk('your disk name');
```

<a name="oss"></a>
## Cloud uploads

If you need to upload to cloud storage, you need to install an adapter for `laravel storage`.

First install [zgldh/qiniu-laravel-storage](https://github.com/zgldh/qiniu-laravel-storage)

With the disk configured as well, add an entry to `config/filesystems.php`:

```php
'disks' => [
    ... ,
    'qiniu' => [
        'driver'  => 'qiniu',
        'domains' => [
            'default'   => 'xxxxx.com1.z0.glb.clouddn.com', //你的七牛域名
            'https'     => 'dn-yourdomain.qbox.me',         //你的HTTPS域名
            'custom'    => 'static.abc.com',                //你的自定义域名
         ],
        'access_key'=> '',  //AccessKey
        'secret_key'=> '',  //SecretKey
        'bucket'    => '',  //Bucket名字
        'notify_url'=> '',  //持久化处理回调地址
        'url'       => 'http://of8kfibjo.bkt.clouddn.com/',  // 填写文件访问根url
    ],
],

```

Then modify the `dcat-admin` upload configuration by opening `config/admin.php` and find:

```php

'upload'  => [

    'disk' => 'qiniu',

    'directory'  => [
        'image'  => 'image',
        'file'   => 'file',
    ],
],

```

`disk` select `qiniu` as configured above, or:
```php
$form->file('file')->disk('qiniu');
```

<a name="public"></a>
## Public methods

### Thumbnail

Generate thumbnails while uploading images

```php
$form->image($column[, $label])->thumbnail('small', $width = 300, $height = 300);

// Generate multiple thumbnails
$form->image($column[, $label])->thumbnail([
    'small1' => [100, 100],
    'small2' => [200, 200],
    'small3' => [300, 300],
]);
```

```php
use Dcat\Admin\Traits\Resizable;

class Photo extends Model
{
    use Resizable;
}

// To access thumbnail
$photo->thumbnail('small', 'photo_column');
```

<a name="disk"></a>
### Storage drive (disk)
Modify file upload source
```php
$form->file('file')->disk('your disk name');
```

<a name="move"></a>
### Upload path (move)
Modify the upload path
```php
$form->file('file')->move('public/upload/image1/');
```

<a name="name"></a>
### File name (name)
Modify the upload file name
```php
$form->file('file')->name('test.text');

$form->image('picture')->name(function ($file) {
    return 'test.'.$file->guessExtension();
});
```

<a name="uniqueName"></a>
### randomName (uniqueName)
Using randomly generated filenames (md5(uniqid()).extension)
```php
$form->image('picture')->uniqueName();
```

<a name="removable"></a>
### Disable page delete files (replace upload)

The `removable` method allows you to prevent users from deleting files on the server by clicking on them on the page, and allows you to achieve an image overlay upload effect.


```php
$form->file($column[, $label])->removable(false);
```


<a name="autoUpload"></a>
### AutoUpload

When this function is enabled, the file will be uploaded automatically immediately after selection, and the page will no longer show the `Upload` button.

```php
$form->file('file')->autoUpload();

$form->image('img')->autoUpload();
```

<a name="retainable"></a>
### Disable removal (recallable)

Files will not be deleted from the server after this feature is enabled.

```php
$form->file('file')->retainable();

$form->image('img')->retainable();
```


<a name="storagePermission"></a>
### storagePermission
Setting Permissions for Uploading Files
```php
$form->image('picture')->storagePermission(777);
```


<a name="accept"></a>
### Restrict file upload types
Restrict the types of files to be uploaded
```php
$form->file('file')->accept('jpg,png,gif,jpeg');

// can specify mimeTypes, multiple comma-separated
$form->file('file')->accept('jpg,png,gif,jpeg', 'image/*');
```

<a name="chunked"></a>
### disableChunked

Enable chunked Upload

```php
$form->file('file')->chunked();
```

<a name="chunkSize"></a>
### chunkSize

Set the block size in `KB`, default `5MB`.

> {tip} Calling this method will automatically enable block uploads.

```php
// Set to 1MB
$form->file('file')->chunkSize(1024);
```

<a name="maxSize"></a>
### File size(maxSize)
Set the maximum size of a single file in `Kb`, the default size is `10M`.

> {tip} Also make sure that the `upload_max_filesize` parameter in the `php.ini` configuration file must be greater than the value set by this method.

```php
// Set the maximum size of a single file to 1Mb
$form->file('file')->maxSize(1024);
```

<a name="threads"></a>
### Number of concurrent upload threads (threads)
Set the number of concurrent upload threads, default `3`
```php
$form->file('file')->threads(5);
```

<a name="url"></a>
### Custom upload interface (url)
Custom upload interfaces can be set via `url`.
```php
$form->file('file')->url('users/files');
```

A `trait`, `Dcat\Admin\Traits\HasUploadedFile`, is provided to help developers handle uploaded files more easily, as follows

```php
<?php

namespace App\Admin\Controllers;

use Dcat\Admin\Traits\HasUploadedFile;

class FileController
{
    use HasUploadedFile;

    public function handle()
    {
        $disk = $this->disk('local');
        
        // Determine if it is a file deletion request
        if ($this->isDeleteRequest()) {
            // Delete files and respond
            return $this->deleteFileAndResponse($disk);
        }
        
        // Get uploaded files
        $file = $this->file();

        // Get the name of the uploaded field
        $column = $this->uploader()->upload_column;

        $dir = 'my-images';
        $newName = $column.'-MyFileName.'.$file->getClientOriginalExtension();

        $result = $disk->putFileAs($dir, $file, $newName);

        $path = "{$dir}/$newName";

        return $result
            ? $this->responseUploaded($path, $disk->url($path))
            : $this->responseErrorMessage('File upload failed');
    }
}
```

To your routing file `app\Admin\routes.php` add the following

```php
$router->any('users/files', 'FileController@handle');
```


<a name="deleteUrl"></a>
### deleteUrl
Modify and delete the path of uploaded files, this method generally does not need to be modified
```php
$form->file('file')->deleteUrl('file/delete');
```

<a name="autoSave"></a>
### AutoSave field values (autoSave)

Set whether to automatically save the file path to the database after uploading a file, this method is enabled by default.

```php
$form->file('file')->autoSave(false);
```

<a name="options"></a>
### Configure (options)
customization configuration [webuploader](https://fex.baidu.com/webuploader/document.html)
```php
$form->file('file')->options(['disableGlobalDnd' => true]);
```

### Sortable (sortable)

This method is only available for multiple image/file upload forms.

```php
$form->multipleImage('images')->sortable();
```


### Compress pictures (compress)

Not enabled by default

```php
// Enable image compression.

$form->multipleImage('images')->compress();

$form->image('avatar')->compress([
	'width' => 1600,
	'height' => 1600,

	// Image quality, only valid if type is `image/jpeg`.
	'quality' => 90,

	// This option should be set to false if you want to generate small images without distortion.
	'allowMagnify' => false,

	// Whether cropping is allowed.
	'crop' => false,

	// Whether to keep the header meta information.
	'preserveHeaders' => true,

	// If you find that the compressed file size is even larger than the original, use the original image.
	// This property may affect the image auto-correct function.
	'noCompressIfLarger' => false,

	// Unit byte, if the image size is smaller than this value, compression will not be applied.
	'compressSize' => 0
]);
```

### Listening to WebUploader file upload events (on)

The `on` method can listen for [WebUploader file upload-related events](http://fex.baidu.com/webuploader/doc/index.html#WebUploader_Uploader_events)

```php
$form->file('...')
	->on('startUpload', <<<JS
		function () {
			console.log('File upload started...', this);
			
			// Attach custom parameters to the file upload interface before uploading files
			this.uploader.options.formData['custom_field'] = '...';
		}
<<<JS
    )
	->on('uploadFinished', <<<JS
    	function () {
    		console.log('File upload is complete');
    	}
<<<JS
    );
```


<a name="withhost"></a>
### File/image domain splicing

Please do not set the **accessor** and **mutator** to splice the domain name in the model. If you need to splice the full domain name at the time of access, you can define a `public` method in the model.

```php
<?php

use Illuminate\Support\Str;
use Illuminate\Support\Facades\Storage;

class YourModel extends Model
{
    // Define a public method to access an image or file
	public function getImage()
	{
		if (Str::contains($this->image, '//')) {
		    return $this->image;
		}
		
		return Storage::disk('admin')->url($this->image);
	}
}
```


### Save the domain name

If you need to save a file domain name to a datasheet, you can use the `saveFullUrl` method.

```php
$form->image('avatar')->saveFullUrl();

$form->file('...')->saveFullUrl();
```

### Listening for file upload changes (change)

You can listen for changes when a file **is uploaded successfully** or when a file **is deleted** by using the following methods

```php
$file = $form->file('...');

Admin::script(
	<<<JS
$('{$file->getElementClassSelector()} .file-input').on('change', function () {
	console.log('File changed', this.value);
});
JS
);
```


<a name="imagefun"></a>
## Picture uploading built-in method

<a name="intervention"></a>
### Compressing, cropping, adding watermarks, etc.
Compression, cropping, watermarking and other methods can be used, which requires installation [intervention/image](http://image.intervention.io/getting_started/installation).

For more information on how to use it, please refer to [[Intervention](http://image.intervention.io/getting_started/introduction)]：
```php
$form->image($column[, $label]);

// Modify image upload path and file name
$form->image($column[, $label])->move($dir, $name);

// Cropping pictures
$form->image($column[, $label])->crop(int $width, int $height, [int $x, int $y]);

// add a watermark
$form->image($column[, $label])->insert($watermark, 'center');
```

<a name="dimensions"></a>
### Limit the size of uploaded images
Set file upload size limit

Parameters: `array` in pixels
- `width` Specify the width
- `height` Specify the height
- `min_width` Minimum width
- `min_height` Minimum height
- `max_width` Maximum width
- `max_height` Maximum height
- `ratio` Aspect ratio (width/height)
  
```php
// Upload an image between 100-300 pixels wide.
$form->image('img')->dimensions(['min_width' = 100, 'max_width' => 300]);
```
<a name="referer"></a>


## Save image/file paths to other data tables

The following method allows you to save the path of an image or file in a separate attachment table, while the current image/file field saves only the ID.

> {tip} The example here uses a single image upload form, so you can adjust it yourself if it's multiple images.


Image/File Table Structure
```sql
CREATE TABLE `images` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `path` varchar(191) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '',
  `created_at` timestamp NULL DEFAULT NULL,
  `updated_at` timestamp NULL DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=MyISAM AUTO_INCREMENT=4 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```
use
```php
$form->image('image1')
    ->saving(function ($value) use ($form) {
        if ($form->isEditing() && ! $value) {
            // Editing Pages, Deleting Image Logic
            Image::destroy($form->model()->image1);

            return;
        }

        // Add or edit images to a page
        if ($value) {
            $model = Image::where('path', $value)->first();
        }

        if (empty($model)) {
            $model = new Image();
        }

        $model->path = $value;

        $model->save();

        return $model->getKey();
    })
    ->customFormat(function ($v) {
        if (! $v) {
            return;
        }

        return Image::find((array) $v)->pluck('path')->toArray();
    });
```


## File upload failed or inaccessible?

If you find that you are unable to upload a file, then there are usually several reasons for this.

1. `Laravel` file upload is not configured correctly, check the `admin.upload.disk` parameter. If you are not familiar with the `laravel` file upload feature, please read the documentation [Laravel - File Storage](https://learnku.com/docs/laravel/7.x/filesystem/7485).
2. Files are too large, need to adjust `upload_max_filesize` parameter of `php.ini`.
3. File upload directory without write permissions
4. `php` is not installed or the `fileinfo` extension is not enabled

If your file uploads success but can't be accessed properly, then the `APP_URL` parameter in the `.env` configuration file may not be set correctly.



