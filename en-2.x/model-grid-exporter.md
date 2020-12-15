# Custom export

The dcat-admin data form supports exporting csv files by default.

```php
<?php

namespace App\Admin\Extensions;

use Dcat\Admin\Grid\Exporters\AbstractExporter;

class CustomExporter extends AbstractExporter
{
    public function export()
    {
        $filename = $this->getTable().'.csv';

        $data = $this->getData();

        $output = '';

        $headers = [
            'Content-Encoding'    => 'UTF-8',
            'Content-Type'        => 'text/csv;charset=UTF-8',
            'Content-Disposition' => "attachment; filename=\"$filename\"",
        ];

        response(rtrim($output, "\n"), 200, $headers)->send();

        exit;
    }
}
```