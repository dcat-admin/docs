# Combination headers

The `Grid::combine` method makes it easy to combine any two or more fields into a first-level table header.

<a href="http://103.39.211.179:8080/admin/reports" target="_blank">
    <img style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" src="{{public}}/assets/img/screenshots/grid-combination.png">
</a>

Example

```php
protected function grid()
{
    return Grid::make(new Report(), function (Grid $grid) {
        // The first parameter is the primary table header field name, the second field is the secondary table header field name, the secondary table header field is set to at least two
        $grid->combine('avgCost', ['avgMonthCost', 'avgQuarterCost', 'avgYearCost']);
        
        $grid->combine('avgVist', ['avgMonthVist', 'avgQuarterVist', 'avgYearVist']);
        
        // Set the style
        $grid->combine('top', ['topCost', 'topVist', 'topIncr'])->style('color:#1867c0');
        
        $grid->content->limit(50);
        $grid->cost->sortable();
        $grid->avgMonthCost;
        $grid->avgQuarterCost->setHeaderAttributes(['style' => 'color:#5b69bc']);
        $grid->avgYearCost;
        $grid->avgMonthVist;
        $grid->avgQuarterVist;
        $grid->avgYearVist;
        $grid->incrs->hide();
        $grid->avgVists->hide();
        $grid->topCost;
        $grid->topVist;
        $grid->topIncr;
        $grid->date->sortable();
    });
}
```

