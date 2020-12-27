# 图片/文件上传

[数据表单](model-form.md)通过以下的调用来生成图片/文件上传表单，支持本地和云存储的文件上传。上传组件是基于[webuploader](https://fex.baidu.com/webuploader/)实现的，具体的使用配置可参考[webuploader官方文档](https://fex.baidu.com/webuploader/document.html)。

> {tip} 文件或图片上传表单字段请不要在模型中设置**访问器**和**修改器**拼接域名，如有相关需求可参考[文件/图片域名拼接](#withhost)。

```php
$form->file('file_column');
$form->image('image_column');
```

<a name="local"></a>
## 本地上传

先添加存储配置，`config/filesystems.php` 添加一项`disk`:

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

设置上传的路径为`public/uploads`(public_path('uploads'))。

然后选择上传的`disk`，打开`config/admin.php`找到：

```php
    
'upload'  => [

    'disk' => 'admin',

    'directory'  => [
        'image'  => 'images',
        'file'   => 'files',
    ]
],

```

将`disk`设置为上面添加的`admin`，`directory.image`和`directory.file`分别为用`$form->image($column)`和`$form->file($column)`上传的图片和文件的上传目录。

当然你也可以在代码中指定`disk`：
```php
$form->file('file')->disk('your disk name');
```

<a name="oss"></a>
## 云盘上传

如果需要上传到云存储，需要安装对应`laravel storage`的适配器，拿七牛云存储举例

首先安装 [zgldh/qiniu-laravel-storage](https://github.com/zgldh/qiniu-laravel-storage)

同样配置好disk，在`config/filesystems.php` 添加一项:

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

然后修改`dcat-admin`的上传配置，打开`config/admin.php`找到：

```php

'upload'  => [

    'disk' => 'qiniu',

    'directory'  => [
        'image'  => 'image',
        'file'   => 'file',
    ],
],

```

`disk`选择上面配置的`qiniu`，或：
```php
$form->file('file')->disk('qiniu');
```

<a name="public"></a>
## 公共方法

### 缩略图 (thumbnail)

上传图片的同时生成缩略图

```php
$form->image($column[, $label])->thumbnail('small', $width = 300, $height = 300);

// 生成多张缩略图
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
### 存储驱动 (disk)
修改文件上传源
```php
$form->file('file')->disk('your disk name');
```

<a name="move"></a>
### 上传路径 (move)
修改上传路径
```php
$form->file('file')->move('public/upload/image1/');
```

<a name="name"></a>
### 文件名称 (name)
修改上传文件名称
```php
$form->file('file')->name('test.text');

$form->image('picture')->name(function ($file) {
    return 'test.'.$file->guessExtension();
});
```

<a name="uniqueName"></a>
### 随机名称 (uniqueName)
使用随机生成文件名 (md5(uniqid()).extension)
```php
$form->image('picture')->uniqueName();
```

<a name="removable"></a>
### 禁止页面删除文件 (替换上传)

通过`removable`方法可以禁止用户从页面点击删除服务器上的文件，可以实现图片覆盖上传效果。


```php
$form->file($column[, $label])->removable(false);
```


<a name="autoUpload"></a>
### 自动上传 (autoUpload)

开启这个功能之后选择完文件之后会立即自动上传，页面将不再显示`上传`按钮，使用方法如下

```php
$form->file('file')->autoUpload();

$form->image('img')->autoUpload();
```

<a name="retainable"></a>
### 禁止删除 (retainable)

开启这个功能之后文件将不会从服务器删除

```php
$form->file('file')->retainable();

$form->image('img')->retainable();
```


<a name="storagePermission"></a>
### storagePermission
设置上传文件的权限
```php
$form->image('picture')->storagePermission(777);
```


<a name="accept"></a>
### 限制上传文件类型
限制上传文件的类型
```php
$form->file('file')->accept('jpg,png,gif,jpeg');

// 可以指定 mimeTypes， 多个用逗号分割
$form->file('file')->accept('jpg,png,gif,jpeg', 'image/*');
```

<a name="chunked"></a>
### 分块上传 (chunked)

启用分块上传

```php
$form->file('file')->chunked();
```

<a name="chunkSize"></a>
### 分块大小(chunkSize)

设置分块大小，单位为`KB`，默认`5MB`

> {tip} 调用这个方法会自动启用分块上传

```php
// 设置为 1MB
$form->file('file')->chunkSize(1024);
```

<a name="maxSize"></a>
### 文件大小(maxSize)
设置单个文件最大大小，单位为`Kb`，默认大小为`10M`。

> {tip} 同时应该保证`php.ini`配置文件的`upload_max_filesize`参数值必须大于这个方法设置的值。

```php
// 设置单个文件最大为1Mb
$form->file('file')->maxSize(1024);
```

<a name="threads"></a>
### 并发上传线程数 (threads)
设置并发上传线程数，默认`3`
```php
$form->file('file')->threads(5);
```

<a name="url"></a>
### 自定义上传接口 (url)
通过`url`可以设置自定义上传接口
```php
$form->file('file')->url('users/files');
```

系统提供了`Dcat\Admin\Traits\HasUploadedFile`这个`trait`来帮助开发者更轻松地处理上传文件，用法如下

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
        
        // 判断是否是删除文件请求
        if ($this->isDeleteRequest()) {
            // 删除文件并响应
            return $this->deleteFileAndResponse($disk);
        }
        
        // 获取上传的文件
        $file = $this->file();

        // 获取上传的字段名称
        $column = $this->uploader()->upload_column;

        $dir = 'my-images';
        $newName = $column.'-我的文件名称.'.$file->getClientOriginalExtension();

        $result = $disk->putFileAs($dir, $file, $newName);

        $path = "{$dir}/$newName";

        return $result
            ? $this->responseUploaded($path, $disk->url($path))
            : $this->responseErrorMessage('文件上传失败');
    }
}
```

在你的路由文件`app\Admin\routes.php`中加上

```php
$router->any('users/files', 'FileController@handle');
```


<a name="deleteUrl"></a>
### deleteUrl
修改删除已上传文件路径，此方法一般不需要修改
```php
$form->file('file')->deleteUrl('file/delete');
```

<a name="autoSave"></a>
### 自动保存字段值 (autoSave)

设置上传文件后是否自动保存文件路径到数据库，此方法默认启用，一般不需要修改

```php
$form->file('file')->autoSave(false);
```

<a name="options"></a>
### 配置 (options)
自定义[webuploader](https://fex.baidu.com/webuploader/document.html)配置
```php
$form->file('file')->options(['disableGlobalDnd' => true]);
```

### 可排序 (sortable)

此方法仅针对多图/文件上传表单有效

```php
$form->multipleImage('images')->sortable();
```


### 压缩图片 (compress)

默认不启用

```php
// 启用图片压缩功能

$form->multipleImage('images')->compress();

$form->image('avatar')->compress([
	'width' => 1600,
	'height' => 1600,

	// 图片质量，只有type为`image/jpeg`的时候才有效。
	'quality' => 90,

	// 是否允许放大，如果想要生成小图的时候不失真，此选项应该设置为false.
	'allowMagnify' => false,

	// 是否允许裁剪。
	'crop' => false,

	// 是否保留头部meta信息。
	'preserveHeaders' => true,

	// 如果发现压缩后文件大小比原来还大，则使用原来图片
	// 此属性可能会影响图片自动纠正功能
	'noCompressIfLarger' => false,

	// 单位字节，如果图片大小小于此值，不会采用压缩。
	'compressSize' => 0
]);
```

### 监听WebUploader文件上传事件 (on)

通过 `on` 方法可以监听 [WebUploader文件上传相关事件](http://fex.baidu.com/webuploader/doc/index.html#WebUploader_Uploader_events)

```php
$form->file('...')
	->on('startUpload', <<<JS
		function () {
			console.log('文件开始上传...', this);
			
			// 上传文件前附加自定义参数到文件上传接口
			this.uploader.options.formData['custom_field'] = '...';
		}
<<<JS
    )
	->on('uploadFinished', <<<JS
    	function () {
    		console.log('文件上传完毕');
    	}
<<<JS
    );
```


<a name="withhost"></a>
### 文件/图片域名拼接

文件或图片上传表单字段请不要在模型中设置**访问器**和**修改器**拼接域名，如果你需要在访问的时候拼接完整域名，可以在模型中定义一个`public`方法

```php
<?php

use Illuminate\Support\Str;
use Illuminate\Support\Facades\Storage;

class YourModel extends Model
{
    // 定义一个public方法访问图片或文件
	public function getImage()
	{
		if (Str::contains($this->image, '//')) {
		    return $this->image;
		}
		
		return Storage::disk('admin')->url($this->image);
	}
}
```


### 保存域名

如果你需要保存文件域名到数据表，可以使用`saveFullUrl`方法

```php
$form->image('avatar')->saveFullUrl();

$form->file('...')->saveFullUrl();
```

### 监听文件上传变动 (change)

通过以下方法可以监听文件**上传成功**或文件**被删除**时产生的变动

```php
$file = $form->file('...');

Admin::script(
	<<<JS
$('{$file->getElementClassSelector()} .file-input').on('change', function () {
	console.log('文件发生变动', this.value);
});
JS
);
```


<a name="imagefun"></a>
## 图片上传内置方法

<a name="intervention"></a>
### 压缩、裁切、添加水印等
可以使用压缩、裁切、添加水印等各种方法,需要先安装[intervention/image](http://image.intervention.io/getting_started/installation).

更多使用方法请参考[[Intervention](http://image.intervention.io/getting_started/introduction)]：
```php
$form->image($column[, $label]);

// 修改图片上传路径和文件名
$form->image($column[, $label])->move($dir, $name);

// 剪裁图片
$form->image($column[, $label])->crop(int $width, int $height, [int $x, int $y]);

// 加水印
$form->image($column[, $label])->insert($watermark, 'center');
```

<a name="dimensions"></a>
### 限制上传图片的尺寸
设置文件上传尺寸限制

参数： `array` 单位为像素
- `width` 指定宽度
- `height` 指定高度
- `min_width` 最小宽度
- `min_height` 最小高度
- `max_width` 最大宽度
- `max_height` 最大高度
- `ratio` 宽高比 (width/height)
  
```php
// 上传宽度为100-300像素之间的图片
$form->image('img')->dimensions(['min_width' = 100, 'max_width' => 300]);
```
<a name="referer"></a>


## 把图片/文件路径保存在其他数据表

通过下面的方法可以把图片或文件的路径保存在单独的附件表，而当前的图片/文件字段只保存ID

> {tip} 这里的示例用的是单图上传表单，如果是多图的话可以按这个思路自行调整。


图片/文件表结构
```sql
CREATE TABLE `images` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `path` varchar(191) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '',
  `created_at` timestamp NULL DEFAULT NULL,
  `updated_at` timestamp NULL DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=MyISAM AUTO_INCREMENT=4 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```
使用
```php
$form->image('image1')
    ->saving(function ($value) use ($form) {
        if ($form->isEditing() && ! $value) {
            // 编辑页面，删除图片逻辑
            Image::destroy($form->model()->image1);

            return;
        }

        // 新增或编辑页面上传图片
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


## 文件上传失败或无法访问？

如果你发现无法上传文件，那么通常有几下几点原因：

1. `Laravel`文件上传配置不正确，请检查`admin.upload.disk`参数。如果你不了解`laravel`文件上传功能，请阅读文档[Laravel - 文件存储](https://learnku.com/docs/laravel/7.x/filesystem/7485)
2. 文件过大，需要调整`php.ini`的`upload_max_filesize`参数
3. 文件上传目录没有写权限
4. `php`没有安装或没有开启`fileinfo`扩展

如果你的文件上传成功了，却无法正常访问，那么可能是`.env`配置文件中的`APP_URL`参数没有设置正确。



