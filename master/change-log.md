# 更新日志


### v1.2.0

**发布时间** `2020-04-24`

#### 功能新增以及优化部分

**1.集成`editor-md`作为内置markdown编辑器，并支持图片上传功能**

使用
```php
$form->markdown('content')->disk('oss');
```

<a href="https://cdn.learnku.com/uploads/images/202004/24/38389/CvmM1N0yre.png!large" target="_blank">
    <img class="img img-full" src="https://cdn.learnku.com/uploads/images/202004/24/38389/CvmM1N0yre.png!large">
</a>

**2.表格过滤搜索增加`panel`布局方式**

目前系统内置两种过滤器的布局方式，默认的是`rightSide`(右侧滑动面板)布局，在这个版本中通过以下方式可以切换过滤器的布局方式

```php
use Dcat\Admin\Grid;

$grid->filter(function (Grid\Filter $filter) {
    // 更改为 panel 布局
    $filter->panel();
    
    // 注意切换为panel布局方式时需要重新调整表单字段的宽度
    $filter->equal('id')->width(3);
});
```

**3.优化数据表格边框模式**

这个版本中优化了表格的边框模式，即使是非组合表头也可以使用边框模式

```php
$grid->withBorder();
```

<a href="https://cdn.learnku.com/uploads/images/202004/24/38389/zQXwonRNhV.png!large" target="_blank">
    <img class="img img-full" src="https://cdn.learnku.com/uploads/images/202004/24/38389/zQXwonRNhV.png!large">
</a>


**4.工具表单增加`buildSuccessScript`方法**

工具表单自定义类中可以通过`buildSuccessScript`和`buildErrorScript`方法控制表单保存之后的行为，比如你可以在表单保存成功之后进行打印小票等操作。

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
 	 * 设置表单保存成功后执行的JS
 	 * 
	 * @return string|void
	 */
	protected function buildSuccessScript()
	{
	    return <<<JS
		// data 为接口返回数据
		if (! data.status) {
			Dcat.error(data.message);

			return false;
		}

		Dcat.success(data.message);

		if (data.redirect) {
			Dcat.reload(data.redirect)
		}

		// 中止后续逻辑（默认逻辑）
		return false;
JS;
	}

	/**
	 * 设置表单保存失败后执行的JS
	 * 
	 * @return string|void
	 */
	protected function buildErrorScript()
	{
		return <<<JS
		var errorData = JSON.parse(response.responseText);
		
		if (errorData) {
			Dcat.error(errorData.message);
		} else {
			console.log('提交出错', response.responseText);
		}
		
		// 终止后续逻辑执行（默认逻辑）
		return false;
JS;
	}
}
```


**5.数据表格表头过滤器重置按钮优化**

**6.通过`Form::action`方法设置`url`时自动拼接路由前缀**

**7.按钮样式优化**

**8.菜单配色及导航栏样式优化**

**9.快速创建功能样式优化**

**10.登陆页面优化**

**11.进度条样式优化**

**12.默认禁用滚动条插件**

**13.增加`action`以及`form`生成命令对非`app`目录的支持**

#### Bug修复部分

- 修复数据表格`checkbox`显示类型不兼容非数组字段值问题
- 修复登陆页面不兼容`Laravel5.5`问题



### v1.1.0

**发布时间** `2020-04-20`

功能新增以及优化部分

- 集成`TinyMCE`作为内置编辑器，并支持图片上传功能
- 增加绿色主题配色
- 数据表格快捷搜索增加`auto`方法，允许禁用自动提交功能
- 优化背景色以及表格边框颜色，加深对比度
- 登陆页面优化
- 模型树字段名称设置功能简化
- 优化菜单栏样式
- 增加切换或刷新页面时移除模态窗遮罩层功能
- 表单页面禁用`perfectScrollbar`滚动条美化插件
- 表单`Field`增加`resolving`和`composing`事件支持

`Bug`修复部分

- 修复`action`确认弹窗设置空字符串或去除弹窗方法报错问题
- 修复`Show\Field::view`方法会转义`HTML`实体问题
- 修复编辑页面聚焦输入框时按回车键会触发删除确认弹窗问题
- 修复 `Laravel5.5`兼容问题
- 修复`switchGroup`没有颜色默认值问题





### v1.0.1

- 修复数据表格筛选器`selectOne`方法筛选报错问题
- 文件/图片上传表单自定义上传接口时关联文件删除相关功能
- 优化图片上传表单样式
- 优化菜单栏样式

