# 数据导出

系统默认使用<a href="https://github.com/jqhph/easy-excel" target="__blank">Easy Excel</a>作为导出工具，支持导出 `csv`、 `xlsx` 和 `ods` 等格式文件。


使用前必须先安装<a href="https://github.com/jqhph/easy-excel" target="__blank">Easy Excel</a>：

```bash
composer require dcat/easy-excel
```

> {tip} 默认不开启导出功能。

### 启用导出功能
启用或禁用导出功能
```php
$grid->export();
```

禁用 `导出所有` 选项
```php
$grid->export()->disableExportAll();
```

禁用 `导出选中行` 选项
```php
$grid->export()->disableExportSelectedRow();
```

禁用 `导出当前页` 选项
```php
$grid->export()->disableExportCurrentPage();
```

### 导出文件类型

> {tip} 默认导出 `xlsx` 格式文件。

```php
// csv
$grid->export()->csv();

// xlsx
$grid->export()->xlsx();

// ods
$grid->export()->ods();
```

### 设置列标题

> {tip} 如果设置了标题，那么导出的文件的**列数**与标题的**列数**相同，且列的**排序**也相同。

```php
// 只导出 id, name和email 三列数据
$titles = ['id' => 'ID', 'name' => '名称', 'email' => '邮箱'];

$grid->export($titles);

// 也可以这么使用
$grid->export()->titles($titles);
```

### 处理导出数据

```php
$grid->export()->rows(function (array $rows) {
    foreach ($rows as $index => &$row) {
        $row['name'] = $row['first_name'].' '.$row['last_name'];
    }
    
    return $rows;
});
```

### 设置导出文件名

```php
$grid->export()->filename('管理员数据');
```

<a name="disable-export-extend"></a>
## 扩展导出功能

如果系统内置的导出功能不了自己的需求，可以按照下面的步骤来自定义导出功能

本示例用[Laravel-Excel](https://github.com/Maatwebsite/Laravel-Excel)作为excel操作库，当然也可以使用任何其他excel库

首先安装好它：

```shell
composer require maatwebsite/excel:~2.1.0

php artisan vendor:publish --provider="Maatwebsite\Excel\ExcelServiceProvider"
```

然后新建自定义导出类，比如`app/Admin/Extensions/ExcelExpoter.php`:
```php
<?php

namespace App\Admin\Extensions;

use Dcat\Admin\Grid\Exporters\AbstractExporter;
use Maatwebsite\Excel\Facades\Excel;

class ExcelExpoter extends AbstractExporter
{
    public function export()
    {
        Excel::create('Filename', function($excel) {

            $excel->sheet('Sheetname', function($sheet) {
                
                 // 最多导出10W条数据
                 // 必须设置maxSize，当否则选择导出所有选项时只能导出默认的20条数据。
                $maxSize = 10000;

                // 这段逻辑是从表格数据中取出需要导出的字段
                $rows = collect($this->buildData(1, $maxSize))->map(function ($item) {
                    return array_only($item, ['id', 'title', 'content', 'rate', 'keywords']);
                });

                $sheet->rows($rows);

            });

        })->export('xls');
    }
}
```

然后在`model-grid`中使用这个导出类：
```php

use App\Admin\Extensions\ExcelExpoter;

$grid->export(new ExcelExpoter());
```

有关更多`Laravel-Excel`的使用方法，参考[laravel-excel/docs](http://www.maatwebsite.nl/laravel-excel/docs)