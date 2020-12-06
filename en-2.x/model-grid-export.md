# Data export

By default, the system uses <a href="https://github.com/jqhph/easy-excel" target="__blank">Easy Excel</a> as an export tool that supports exporting files in formats such as `csv`, `xlsx` and `ods`.


<a href="https://github.com/jqhph/easy-excel" target="__blank">Easy Excel</a> must be installed before you can use it.

```bash
composer require dcat/easy-excel
```

> {tip} The export function is not enabled by default.

### Enable export function
Enable or disable the export function
```php
$grid->export();
```

Disable the `Export All` option
```php
$grid->export()->disableExportAll();
```

Disable `Export Selected Rows` option
```php
$grid->export()->disableExportSelectedRow();
```

Disable the `Export current page` option
```php
$grid->export()->disableExportCurrentPage();
```

### Exporting file types

> {tip} The `xlsx` format file is exported by default.

```php
// csv
$grid->export()->csv();

// xlsx
$grid->export()->xlsx();

// ods
$grid->export()->ods();
```

### Set column TITLE

> {tip} If TITLE is set, the **number of columns** of the exported file is the same as the **number of columns** of TITLE, and the **ordering** of the columns is the same.

```php
// Export only the id, name and email columns.
$titles = ['id' => 'ID', 'name' => '名称', 'email' => '邮箱'];

$grid->export($titles);

// It can also be used like this
$grid->export()->titles($titles);
```

### Processing exported data

```php
$grid->export()->rows(function (array $rows) {
    foreach ($rows as $index => &$row) {
        $row['name'] = $row['first_name'].' '.$row['last_name'];
    }
    
    return $rows;
});
```

### Set export file name

```php
$grid->export()->filename('AdministratorData');
```

<a name="disable-export-extend"></a>
## Extended export function

If the system's built-in export function does not meet your needs, you can customize the export function by following these steps

This example uses [Laravel-Excel](https://github.com/Maatwebsite/Laravel-Excel) as the excel library, but you can also use any other excel library.

First install it:

```shell
composer require maatwebsite/excel:~2.1.0

php artisan vendor:publish --provider="Maatwebsite\Excel\ExcelServiceProvider"
```

Then create a new custom export class, such as `app/Admin/Extensions/ExcelExpoter.php`:
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
                
                 // Export up to 10W of data
                 // The maxSize must be set so that only the default 20 data items are exported when the Export All option is otherwise selected.
                $maxSize = 10000;

                // This logic takes the fields to be exported from the table data
                $rows = collect($this->buildData(1, $maxSize))->map(function ($item) {
                    return array_only($item, ['id', 'title', 'content', 'rate', 'keywords']);
                });

                $sheet->rows($rows);

            });

        })->export('xls');
    }
}
```

Then use this export class in `model-grid`.
```php

use App\Admin\Extensions\ExcelExpoter;

$grid->export(new ExcelExpoter());
```

For more information on using `Laravel-Excel`, see [laravel-excel/docs](http://www.maatwebsite.nl/laravel-excel/docs).