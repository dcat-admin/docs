# 工具表单

工具表单(`Dcat\Admin\Widgets\Form`)用来构建表单和处理提交数据，可以很方便的独立处理数据，而不需要而外注册路由。

> {tip} 工具表单中如果要使用图片或文件上传表单，需要自定义上传接口，用法请参考[图片/文件上传自定义上传接口](model-form-upload.md#url)。

### 基本使用
用命令`admin:form`来生成表单类文件：

```bash
php artisan admin:form Setting
```
将会生成表单文件`app/Admin/Forms/Setting.php`

```php
<?php

namespace App\Admin\Forms;

use Dcat\Admin\Widgets\Form;
use Symfony\Component\HttpFoundation\Response;

class Setting extends Form
{
    /**
     * Handle the form request.
     *
     * @param array $input
     *
     * @return Response
     */
    public function handle(array $input)
    {
        // dump($input);

        // return $this->error('Your error message.');

        return $this->success('Processed successfully.', '/');
    }

    /**
     * Build a form here.
     */
    public function form()
    {
        $this->text('name')->required();
        $this->email('email')->rules('email');
    }

    /**
     * The data of the form.
     *
     * @return array
     */
    public function default()
    {
        return [
            'name'  => 'John Doe',
            'email' => 'John.Doe@gmail.com',
        ];
    }
}
```
在上面的表单类里面，在`form`方法中构建表单项，使用方法和[数据表单](model-form.md)一致，`data`方法用来给这个表单项设置默认数据。

在页面中填入数据提交表单之后，请求会进入到`handle`方法中，在这里可以加入你的数据处理逻辑，处理完成之后可以通过`success`或`error`方法响应数据到前端：
```php
    public function handle(array $input)
    {
        // $input是你接收到的表单数据
        // 在这里可以写你的处理逻辑


        // 第一个参数是响应的成功信息，第二个参数是要跳转的路由
        return $this->success('Processed successfully.', '/');
    }
```

然后按照下面的方法将上面的表单放到你的页面中：

```php
<?php

use App\Admin\Forms\Setting;
use App\Http\Controllers\Controller;
use Dcat\Admin\Widgets\Card;
use Dcat\Admin\Layout\Content;

class UserController extends Controller
{
    public function setting(Content $content)
    {
        return $content
            ->title('网站设置')
            ->body(new Card(new Setting()));
    }
}
```

