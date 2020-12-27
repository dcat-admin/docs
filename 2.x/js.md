# JS组件

`Dcat Admin`内置了一些常用的JS功能组件，通过全局变量`Dcat`可以访问到这些功能方法。

## 监听JS脚本加载完毕事件 (ready)

通过`Dcat.ready`方法设置的回调函数会在所有的`JS`脚本都加载完毕后执行。

> {tip} 只有在模板文件中写`JS`代码才需要使用`Dcat.ready`，当在`php`代码中使用`Dcat\Admin\Admin::script`方法添加`JS`代码时是不需要使用`Dcat.ready`方法的。因为在构建页面的时候系统会自动把代码放在`Dcat.ready`事件内执行。

```html
<div>...</div>
<script>
Dcat.ready(function () {
    // 写你的逻辑
    
    console.log('所有JS脚本都加载完了');
});
</script>
```

<a name="init"></a>
## 动态监听元素生成 (init)

通过`Dcat.init`可以监听动态生成的页面元素并设置一个回调，下面来举一个简单的例子来演示用法：

假如一个元素是`JS`动态生成的，如果我们需要对这个元素绑定一个点击事件的话，那么我们通常需要这么做

```html
<div class="selector">test</div>

<script>
Dcat.ready(function () {
    // 需要先 off 再 on 否则页面刷新后会造成重复绑定问题
    $(document).off('click', '.selector').on('click', '.selector', function () {
        ...
    })
});
</script>
```

上面这种做法一来比较麻烦，需要先`off`再`on`；二来无法对动态生成的元素做一些特殊处理，例如你想在`.selector`生成后改变背景颜色，这个操作就没办法做到。

在`Dcat Admin`中我们可以使用`Dcat.init`方法来监听元素动态生成，可以很方便的解决上面两个问题

```html
<div class="selector">test</div>

<script>
Dcat.ready(function () {
    // $this 是当前元素的jquery dom对象
    // id 是当前元素的id属性，如果当前元素没有id则会自动生成一个随机id
    Dcat.init('.selector', function ($this, id) {
        // 修改元素的背景色
        $this.css({background: "#fff"});
        
        // 这里不需要 off 再重新 on，因为这个匿名函数只会执行一次
        $this.on('click', function () {
            ...
        });
    });
});
</script>
```

`Dcat.init` 接受两个参数

1. `selector` 需要监听的元素的`css选择器`
2. `callback` 事件回调，当元素生成时触发，且只触发一次

其中`callback`回调接收两个参数如下

- `$this` 是当前元素的jquery dom对象
- `id` 是当前元素的id属性，如果当前元素没有id则会自动生成一个随机id




## 手动触发JS脚本加载完毕事件

通过`Dcat.triggerReady`方法可以手动触发`JS`脚本加载完毕事件，这就意味着会自动执行在此之前所有通过`Dcat.ready`方法设置的回调函数。

> {tip} 这个功能普通开发很少会用到，只有一些比较深度的组件定制会用到，比如[表单弹窗](model-form-modal.md)功能就用到了此方法。

```js
Dcat.triggerReady();
```


## Pjax刷新页面

通过`Dcat.reload`方法可以调用`Pjax.reload`方法实现页面刷新和跳转功能。

刷新当前页面
```php
Admin::script(
<<<JS
    // 3秒后刷新当前页面
    setTimeout(function () {
        Dcat.reload();
    }, 3000);
JS
);
```

跳转页面
```php
$url = admin_url('auth/users');

Admin::script(
<<<JS
    // 3秒后跳转到 admin/auth/users 页面
    setTimeout(function () {
        Dcat.reload('{$url}');
    }, 3000);
JS
);
```

## Toastr提示框


`Dcat Admin`集成了[Toastr](https://github.com/CodeSeven/toastr)插件，下面是使用方法

### success
```js
Dcat.success('更新成功');

// 使用标题
Dcat.success('更新成功', '我是标题');

// 传递参数
Dcat.success('更新成功', null, {
    timeOut: 5000, // 5秒后自动消失
});
```

![](https://cdn.learnku.com/uploads/images/202004/26/38389/zmB4EPhS3u.png!large)


### error
```js
Dcat.error('服务器出现未知错误');

// 使用标题
Dcat.error('服务器出现未知错误', '我是标题');

// 传递参数
Dcat.error('服务器出现未知错误', null, {
    timeOut: 5000, // 5秒后自动消失
});
```

### warning
```js
Dcat.warning('警告');

// 使用标题
Dcat.warning('警告', '我是标题');

// 传递参数
Dcat.warning('警告', null, {
    timeOut: 5000, // 5秒后自动消失
});
```


### info
```js
Dcat.info('提示信息');

// 使用标题
Dcat.info('提示信息', '我是标题');

// 传递参数
Dcat.info('提示信息', null, {
    timeOut: 5000, // 5秒后自动消失
});
```

更多用法请参考[toastr官方文档](https://github.com/CodeSeven/toastr)

## sweetalert2弹窗

`Dcat Admin`集成了[sweetalert2](https://github.com/sweetalert2/sweetalert2)插件，下面是使用方法

### confirm

通过`Dcat.confirm`方法可以弹出确认弹窗，此方法接收5个参数

- `title` `string` 确认信息标题
- `message` `string` 确认信息内容，此参数可以不传
- `success` `function` 点击确认按钮触发的回调函数
- `fail` `function` 点击取消按钮触发的回调函数，此参数可以不传
- `options` `object` 配置参数，可参考[sweetalert2官方文档](https://github.com/sweetalert2/sweetalert2)

```js
Dcat.confirm('确认要删除这行数据吗？', null, function () {
    console.log('确认删除');
    
    $.post(...);
});
```
![](https://cdn.learnku.com/uploads/images/202004/26/38389/lp40C74OtV.png!large)



### success

```js
Dcat.swal.success('标题');

Dcat.swal.success('标题', '内容');

Dcat.swal.success('标题', '内容', {
    ...
});
```
![](https://cdn.learnku.com/uploads/images/202004/26/38389/OrhZ4dvA5R.png!large)


### error

```js
Dcat.swal.error('标题');

Dcat.swal.error('标题', '内容');

Dcat.swal.error('标题', '内容', {
    ...
});
```
![](https://cdn.learnku.com/uploads/images/202004/26/38389/lnp47PDecK.png!large)


### warning

```js
Dcat.swal.warning('标题');

Dcat.swal.warning('标题', '内容');

Dcat.swal.warning('标题', '内容', {
    ...
});
```

### info

```js
Dcat.swal.info('标题');

Dcat.swal.info('标题', '内容');

Dcat.swal.info('标题', '内容', {
    ...
});
```

更多用法请参考[sweetalert2官方文档](https://github.com/sweetalert2/sweetalert2)


## layer弹出层

`Dcat Admin`集成了[layer弹出层](http://layer.layui.com/)，用法请参考官方文档

```js
layer.open({
    ...
})
```

## Loading效果

`Dcat Admin`集成了三种常见的loading效果，[在线体验点我](http://103.39.211.179:8080/admin/components/loading)

### 全屏

通过`Dcat.loading`方法可以设置一个占满全屏幕的遮罩层，并在中间位置显示`loading`图标。

此方法接收一个`object`类型的参数：

| 参数     | 类型   | 默认值      |  描述  |
| ---------- | ----- |-------- | ------- |
|  zIndex  | `int` | 999991014 |   设置css的z-index(层重叠顺序)样式 |
|  width |   `string`   | 58px | 设置loading图标宽度 |
|  color |   `string` | #bacad6  | 设置loading图标的颜色 |
|  background |   `string`   | transparent  | 设置遮罩层背景颜色 |
|  style |  `string`    |  | 设置loading图标的css样式 |


```js
// 开启loading效果
Dcat.loading();

// 3秒后自动移除loading效果
setTimeout(function () {
    Dcat.loading(false);
})
```

效果

![](https://cdn.learnku.com/uploads/images/202004/26/38389/FIWAUFg1qn.png!large)


更改loading图标的颜色

```js
// 更改颜色
Dcat.loading({
    color: Dcat.color.primary,
});
```
![](https://cdn.learnku.com/uploads/images/202004/26/38389/WPIC4wwq5Q.png!large)


### 附着于指定元素

通过`$.fn.loading`方法可以把loading效果附着于当前元素，此方法同样接收一个`object`类型参数：

| 参数     | 类型   | 默认值      |  描述  |
| ---------- | ----- |-------- | ------- |
|  zIndex  | `int` | 100 |   设置css的z-index(层重叠顺序)样式 |
|  width |   `string`   | 52px | 设置loading图标宽度 |
|  color |   `string` | #bacad6  | 设置loading图标的颜色 |
|  background |   `string`   | #fff  | 设置遮罩层背景颜色 |
|  style |  `string`    |  | 设置loading图标的css样式 |


```js
// 开启loading效果
$('#card').loading();

// 关闭loading效果
$('#card').loading(false);

// 更改loading图标颜色
$('#card').loading({
    color: Dcat.color.primary,
});

// 更改遮罩层颜色
$('#card').loading({
    background: '#f3f3f3',
});
```

效果

![](https://cdn.learnku.com/uploads/images/202004/26/38389/ziHL5feEAV.png!large)



### 按钮

```js
// 开启loading效果
$('#submit-button').buttonLoading();

// 关闭loading效果
$('#submit-button').buttonLoading(false);
```

效果
![](https://cdn.learnku.com/uploads/images/202004/26/38389/rNMFWAHPqJ.png!large)


### a标签

点击`a`标签同样支持loading效果


```js
// 开启loading效果
$('a').buttonLoading();

// 关闭loading效果
$('a').buttonLoading(false);
```
效果
![](https://cdn.learnku.com/uploads/images/202004/26/38389/IE8kGdupKW.png!large)




## Ajax表单提交

`Dcat Admin`集成了[jquery-form](https://github.com/jquery-form/form)组件用于实现`ajax`提交表单功能。

通过`Dcat.Form`方法可以当即发起`ajax`提交表单请求，此方法接受一个`object`类型参数：

| 参数     | 类型   | 默认值      |  描述  |
| ---------- | ----- |-------- | ------- |
|  form  | `string` | `object`  |  表单的 jquery 对象或者css选择器  |
|  validate  | `bool` | `false` | 是否开启表单验证，可参考[表单验证](#validate)  |
|  errorClass  | `string` | has-error | 字段验证出错时添加的样式，一般使用默认值即可  |
|  errorContainerSelector  | `string` | .with-errors | 表单错误信息容器css选择器，一般使用默认值即可  |
|  groupSelector  |  `string` | .form-group,.form-label-group,.form-field | 表单组css选择器，一般使用默认值即可 |
|  errorTemplate  | `string` | |  错误信息模板，使用默认值即可 |
|  autoRedirect  | `bool` | `false` |  保存成功后自动跳转 |
|  autoRemoveError  |`bool`  | true | 当表单值发生变化时自动移除表单错误信息  |
|  before  | `function` |  |  表单提交之前事件，返回`false`可以阻止表单继续提交 |
|  after  | `function` |  |  单提交之后事件(不论成功还是失败都会触发)，返回`false`可以中止后续逻辑 |
|  success  | `function` |  | 成功事件（接口返回状态码为`200`则判断为成功），返回`false`可以中止后续逻辑  |
|   error | `function` |  | 失败事件（接口返回状态码非`200`则判断为失败），返回`false`可以中止后续逻辑  |


### 基本用法

```html
<script>
Dcat.ready(function () {
    // ajax表单提交
    $('#login-form').form({
        validate: true, //开启表单验证
        before: function (fields, form, opt) {
            // fields 为表单内容
            console.log('所有表单字段的值', fields);
            
            if (...) {
                // 返回 false 可以阻止表单继续提交
                return false;
            }
        },
        success: function (response) {
            // data 为接口返回数据
            if (! response.status) {
                Dcat.error(response.data.message);

                return false;
            }

            Dcat.success(response.data.message);

			if (data.redirect) {
			    Dcat.reload(response.data.value)
			}

            // 中止后续逻辑（默认逻辑）
            return false;
        },
        error: function (response) {
            // 当提交表单失败的时候会有默认的处理方法，通常使用默认的方式处理即可
            var errorData = JSON.parse(response.responseText);
            
            if (errorData) {
                Dcat.error(errorData.message);
            } else {
                console.log('提交出错', response.responseText);
            }
            
            // 终止后续逻辑执行
            return false;
        },
    });
});
</script>
```


### 高阶用法

如果你想要实现更细粒度的控制，可以通过类似下面这种方式自己绑定提交按钮，然后提交表单

```js
var $form = $('#login-form');

// 禁用默认提交
$form.on('submit', function () {
    return false;
});

// ajax表单提交
function submit() {
    Dcat.Form({
        form: $form,
        success: function (response) {
            if (! response.status) {
                Dcat.error(response.data.message);

                return false;
            }

            Dcat.success(response.data.message);

            location.href = response.data.value;

            return false;
        },
        error: function () {
            // 非200状态码响应错误
        }
    });
}

// h5表单验证
function validateForm() {
    $form.validator('validate');

    // 如果出现错误，则返回false
    if ($form.find('.has-error').length > 0) {
        return false;
    }

    return true;
}

// 绑定登陆按钮点击事件
$form.find('[type="submit"],.submit').click(function (e) {
    // 表单验证
    if (validateForm() === false) {
        return false;
    }

    // 提交表单
    submit();

    return false;
});
```


<a name="validate"></a>
### 表单验证
    
`Dcat Admin`集成了[bootstrap-validator](https://github.com/1000hz/bootstrap-validator)组件用于表单前端验证的功能，
[bootstrap-validator](https://github.com/1000hz/bootstrap-validator)是一款支持H5表单验证的验证器，只需把验证规则写在表单元素的属性上即可自动开启验证，非常方便。



#### 添加验证规则
```html
<fieldset class="form-label-group form-group position-relative has-icon-left">
    <input
    
        minlength="5" <!-- 加上验证规则 -->
        maxlength="20" <!-- 加上验证规则 -->
        required  <!-- 加上验证规则 -->
        type="password"
        class="form-control"
        name="password"
    >

    <div class="form-control-position">
        <i class="feather icon-lock"></i>
    </div>
    <label for="password">{{ trans('admin.password') }}</label>

    <!-- 这个加了 .with-errors 样式的 div 即是表单错误信息显示的位置，非常重要 -->
    <div class="help-block with-errors"></div>
</fieldset>
```
#### 开启表单验证
```js
$('#xx-form').form({
    validate: true
});
```

效果

![](https://cdn.learnku.com/uploads/images/202004/26/38389/wJxcYaC9GP.png!large)
![](https://cdn.learnku.com/uploads/images/202004/26/38389/qdXUaNEMSQ.png!large)

<a name="extend-validator"></a>
#### 扩展验证规则

通过`Dcat.validator.extend`方法可以扩展表单验证规则

```js
Dcat.validator.extend('maxlength', function ($el) {
    return $el.val().length > $el.attr('data-maxlength');
});
```

使用自定义规则验证表单

```html
<input 
    type="input"
    class="form-control"
    name="username"
    data-maxlength="20" <!-- 使用刚刚自定义的验证规则 -->
    data-maxlength-error="已超出输入字符长度限制，请输入20个或以下的字符" <!-- 定义错误信息 -->
 />
```

#### 内置验证规则
更多内置验证规则请参考[bootstrap-validator官方文档](http://1000hz.github.io/bootstrap-validator/)
