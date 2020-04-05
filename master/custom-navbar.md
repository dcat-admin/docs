# 自定义头部导航条

如果要在顶部导航条上添加html元素,  打开`app/Admin/bootstrap.php`：
```php
use Dcat\Admin\Layout\Navbar;
use Dcat\Admin\Admin;

Admin::navbar(function (Navbar $navbar) {

    $navbar->left('html...');

    $navbar->right('html...');

});
```

`left`和`right`方法分别用来在头部的左右两边添加内容，方法参数可以是任何可以渲染的对象(实现了`Htmlable`、`Renderable`接口或者包含`__toString()`方法的对象)或字符串

## 添加搜索框示例

举个例子，比如在左边添加一个搜索条，先创建一个blade视图`resources/views/search-bar.blade.php`：
```php
use Dcat\Admin\Layout\Navbar;
use Dcat\Admin\Admin;

Admin::navbar(function (Navbar $navbar) {
    $navbar->right(
        <<<HTML
<ul class="nav navbar-nav float-right">
    <li class="nav-item nav-search"><a class="nav-link nav-link-search"><i class="ficon feather icon-search"></i></a>
      <div class="search-input">
            <div class="search-input-icon"><i class="feather icon-search primary"></i></div>
            <input class="input" type="text" placeholder="Explore Vuexy..." tabindex="-1" data-search="laravel-search-list">
            <div class="search-input-close"><i class="feather icon-x"></i></div>
            <ul class="search-list search-list-main ps"><div class="ps__rail-x" style="left: 0px; bottom: 0px;"><div class="ps__thumb-x" tabindex="0" style="left: 0px; width: 0px;"></div></div><div class="ps__rail-y" style="top: 0px; right: 0px;"><div class="ps__thumb-y" tabindex="0" style="top: 0px; height: 0px;"></div></div></ul>
      </div>
    </li>
</ul>
HTML
    );
});
```


## 添加下拉面板

在模板文件中加上
```html
<ul class="nav navbar-nav">
    <li class="dropdown dropdown-notification nav-item">
        <a class="nav-link nav-link-label" href="#" data-toggle="dropdown" aria-expanded="true"><i class="ficon feather icon-bell"></i><span class="badge badge-pill badge-primary badge-up">5</span></a>
        <ul class="dropdown-menu dropdown-menu-media dropdown-menu-right ">
            <li class="dropdown-menu-header">
                <div class="dropdown-header m-0 p-2">
                    <h3 class="white">5 New</h3><span class="grey darken-2">App Notifications</span>
                </div>
            </li>
            <li class="scrollable-container media-list ps ps--active-y">
                <a class="d-flex justify-content-between" href="javascript:void(0)">
                    <div class="media d-flex align-items-start">
                        <div class="media-left"><i class="feather icon-plus-square font-medium-5 primary"></i></div>
                        <div class="media-body">
                            <h6 class="primary media-heading">You have new order!</h6><small class="notification-text"> Are
                                your going to meet me
                                tonight?</small>
                        </div><small>
                            <time class="media-meta" datetime="2015-06-11T18:29:20+08:00">9 hours
                                ago</time></small>
                    </div>
                </a><a class="d-flex justify-content-between" href="javascript:void(0)">
                    <div class="media d-flex align-items-start">
                        <div class="media-left"><i class="feather icon-download-cloud font-medium-5 success"></i></div>
                        <div class="media-body">
                            <h6 class="success media-heading red darken-1">99% Server load</h6>
                            <small class="notification-text">You got new order of goods.</small>
                        </div><small>
                            <time class="media-meta" datetime="2015-06-11T18:29:20+08:00">5 hour
                                ago</time></small>
                    </div>
                </a><a class="d-flex justify-content-between" href="javascript:void(0)">
                    <div class="media d-flex align-items-start">
                        <div class="media-left"><i class="feather icon-alert-triangle font-medium-5 danger"></i></div>
                        <div class="media-body">
                            <h6 class="danger media-heading yellow darken-3">Warning notifixation
                            </h6><small class="notification-text">Server have 99% CPU usage.</small>
                        </div><small>
                            <time class="media-meta" datetime="2015-06-11T18:29:20+08:00">Today</time></small>
                    </div>
                </a>
                <div class="ps__rail-x" style="left: 0px; bottom: 0px;"><div class="ps__thumb-x" tabindex="0" style="left: 0px; width: 0px;"></div></div><div class="ps__rail-y" style="top: 0px; right: 0px; height: 254px;"><div class="ps__thumb-y" tabindex="0" style="top: 0px; height: 184px;"></div></div></li>
            <li class="dropdown-menu-footer"><a class="dropdown-item p-1 text-center" href="javascript:void(0)">Read
                    all notifications</a></li>
        </ul>
    </li>
</ul>
```

使用

```php
$navbar->right(view('...'));
```

## 添加下拉菜单

```html
<ul class="nav navbar-nav">
    <li class="dropdown dropdown-language nav-item">
        <a class="dropdown-toggle nav-link" href="#" id="dropdown-flag" data-toggle="dropdown">
            <i class="flag-icon flag-icon-us"></i>
            <span class="selected-language">English</span>
        </a>
        <ul class="dropdown-menu" aria-labelledby="dropdown-flag">
            <li class="dropdown-item" href="#" data-language="en">
                <a><i class="flag-icon flag-icon-us"></i> English</a>
            </li>
            <li class="dropdown-item" href="#" data-language="fr">
                <a><i class="flag-icon flag-icon-fr"></i> French</a>
            </li>
            <lia class="dropdown-item" href="#" data-language="de">
                <a><i class="flag-icon flag-icon-de"></i>  German</a>
            </lia>
        </ul>
    </li>
</ul>
```
使用

```php
$navbar->right(view('...'));
```

更多的组件可以参考[Vuexy](https://pixinvent.com/demo/vuexy-vuejs-admin-dashboard-template/demo-1/)