# 图片/文件上传

[model-form](model-form.md)通过以下的调用来生成form元素，支持本地和云存储的文件上传。

> {tip} 上传组件是基于[webuploader](https://fex.baidu.com/webuploader/)实现的，具体的使用配置可参考[webuploader官方文档](https://fex.baidu.com/webuploader/document.html)。

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

<a name="disk"></a>
### disk
修改文件上传源
```php
$form->file('file')->disk('your disk name');
```

<a name="move"></a>
### move
修改上传路径
```php
$form->file('file')->move('public/upload/image1/');
```

<a name="name"></a>
### name
修改上传文件名称
```php
$form->file('file')->name('test.text');

$form->image('picture')->name(function ($file) {
    return 'test.'.$file->guessExtension();
});
```

<a name="uniqueName"></a>
### uniqueName
使用随机生成文件名 (md5(uniqid()).extension)
```php
$form->image('picture')->uniqueName();
```

<a name="storagePermission"></a>
### storagePermission
设置上传文件的权限
```php
$form->image('picture')->storagePermission(777);
```

<a name="disableRemove"></a>
### disableRemove
禁止删除按钮
```php
// 禁止删除按钮
$form->file($column[, $label])->disableRemove();
```

<a name="accept"></a>
### 限制上传文件类型
限制上传文件的类型
```php
$form->file('file')->accept('jpg,png,gif,jpeg');

// 可以指定 mimeTypes， 多个用逗号分割
$form->file('file')->accept('jpg,png,gif,jpeg', 'image/*');
```

<a name="disableChunked"></a>
### disableChunked
禁用分块上传
```php
$form->file('file')->disableChunked();
```

<a name="chunkSize"></a>
### chunkSize
分块大小，单位为“KB”，默认“5MB”
```php
// 设置为 1MB
$form->file('file')->chunkSize(1024);
```

<a name="maxSize"></a>
### maxSize
设置单个文件最大大小，单位为“KB”
```php
$form->file('file')->maxSize(1024);
```

<a name="threads"></a>
### threads
设置并发上传线程数，默认3
```php
$form->file('file')->threads(5);
```

<a name="url"></a>
### url
修改文件上传路径，此方法一般不需要修改
```php
$form->file('file')->url('users/upload');
```

<a name="deleteUrl"></a>
### deleteUrl
修改删除已上传文件路径，此方法一般不需要修改
```php
$form->file('file')->deleteUrl('file/delete');
```

<a name="disableAutoSave"></a>
### disableAutoSave
禁止上传文件后自动保存文件路径到数据库，此方法一般不需要修改
```php
$form->file('file')->disableAutoSave();
```

<a name="options"></a>
### options
自定义[webuploader](https://fex.baidu.com/webuploader/document.html)配置
```php
$form->file('file')->options(['disableGlobalDnd' => true]);
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

