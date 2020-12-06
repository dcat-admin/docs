# Custom charts

`dcat-admin 1.5` has removed all chart components, if you want to add a chart component to your page, you can refer to the following procedure

To use `chartjs` as an example, first download [chartjs](http://chartjs.org/) and put it in the public directory, for example, in `public/vendor/chartjs`.

Then include the component in `app/Admin/bootstrap.php'.
```php
use Dcat\Admin\Facades\Admin;

Admin::js('/vendor/chartjs/dist/Chart.min.js');

```

New view file `resources/views/admin/charts/bar.blade.php`

```php
<canvas id="myChart" width="400" height="400"></canvas>
<script>

$(function () {
   var ctx = document.getElementById("myChart").getContext('2d');
   var myChart = new Chart(ctx, {
       type: 'bar',
       data: {
           labels: ["Red", "Blue", "Yellow", "Green", "Purple", "Orange"],
           datasets: [{
               label: '# of Votes',
               data: [12, 19, 3, 5, 2, 3],
               backgroundColor: [
                   'rgba(255, 99, 132, 0.2)',
                   'rgba(54, 162, 235, 0.2)',
                   'rgba(255, 206, 86, 0.2)',
                   'rgba(75, 192, 192, 0.2)',
                   'rgba(153, 102, 255, 0.2)',
                   'rgba(255, 159, 64, 0.2)'
               ],
               borderColor: [
                   'rgba(255,99,132,1)',
                   'rgba(54, 162, 235, 1)',
                   'rgba(255, 206, 86, 1)',
                   'rgba(75, 192, 192, 1)',
                   'rgba(153, 102, 255, 1)',
                   'rgba(255, 159, 64, 1)'
               ],
               borderWidth: 1
           }]
       },
       options: {
           scales: {
               yAxes: [{
                   ticks: {
                       beginAtZero:true
                   }
               }]
           }
       }
   }); 
});
</script>
```

You can then include this chart view anywhere on the page:

```php
public function index()
{
    return Admin::content(function (Content $content) {

        $content->header('chart');
        $content->description('.....');
        
        $content->body(view('admin.charts.bar'));
    });
}

```

Any chart library can be introduced in the above way, see [view layout](layout.md) for multi-chart page layout