# 组合表头

通过 `Grid::combine` 方法可以很方便的把任意两个以上的字段组合成一级表头

<a href="http://103.39.211.179:8080/admin/reports" target="_blank">
    <img style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" src="{{public}}/assets/img/screenshots/grid-combination.png">
</a>

示例

```php
protected function grid()
{
    return Grid::make(new Report(), function (Grid $grid) {
        // 第一个参数为一级表头字段名称，第二个字段为二级表头字段名称，二级表头字段最少设置两个
        $grid->combine('avgCost', ['avgMonthCost', 'avgQuarterCost', 'avgYearCost']);
        
        $grid->combine('avgVist', ['avgMonthVist', 'avgQuarterVist', 'avgYearVist']);
        
        // 设置样式
        $grid->combine('top', ['topCost', 'topVist', 'topIncr'])->style('color:#1867c0');
        
        $grid->column('content')->limit(50);
        $grid->column('cost')->sortable();
        $grid->column('avgMonthCost');
        $grid->column('avgQuarterCost')->setHeaderAttributes(['style' => 'color:#5b69bc']);
        $grid->column('avgYearCost');
        $grid->column('avgMonthVist');
        $grid->column('avgQuarterVist');
        $grid->column('avgYearVist');
        $grid->column('incrs');
        $grid->column('avgVists');
        $grid->column('topCost');
        $grid->column('topVist');
        $grid->column('topIncr');
        $grid->column('date')->sortable();
    });
}
```

