# Customizable header navigation bar

To add an html element to the top navigation bar, open `app/Admin/bootstrap.php`.
```php
use Dcat\Admin\Layout\Navbar;
use Dcat\Admin\Admin;

Admin::navbar(function (Navbar $navbar) {

    $navbar->left('html...');

    $navbar->right('html...');

});
```

The `left` and `right` methods are used to add content to the left and right sides of the header, respectively, and the method arguments can be any renderable object (objects that implement the `Htmlable`, `Renderable` interface or include the `__toString()` method) or string.



## Adding drop-down panels

Add to the template file
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

Usage
```php
$navbar->right(view('...'));
```

## Adding drop-down menus

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
            <li class="dropdown-item" href="#" data-language="de">
                <a><i class="flag-icon flag-icon-de"></i>  German</a>
            </li>
        </ul>
    </li>
</ul>
```

Usage

```php
$navbar->right(view('...'));
```

More components can be found at [AdminLTE3](https://github.com/ColorlibHQ/AdminLTE)。


## Modifying the Default User Panel

Open `app/Admin/bootstrap.php`, write

```php
admin_inject_section(Admin::SECTION['NAVBAR_USER_PANEL'], function () {
	return view('admin.partials.navbar-user-panel', ['user' => Admin::user()]);
});
```

Content `navbar-user-panel.blade.php`
```php
@if($user)
<li class="dropdown dropdown-user nav-item">
    <a class="dropdown-toggle nav-link dropdown-user-link" href="#" data-toggle="dropdown">
        <div class="user-nav d-sm-flex d-none">
            <span class="user-name text-bold-600">{{ $user->name }}</span>
            <span class="user-status"><i class="fa fa-circle text-success"></i> {{ trans('admin.online') }}</span>
        </div>
        <span>
            <img class="round" src="{{ $user->getAvatar() }}" alt="avatar" height="40" width="40" />
        </span>
    </a>
    <div class="dropdown-menu dropdown-menu-right">
        <a href="{{ admin_url('auth/setting') }}" class="dropdown-item">
            <i class="feather icon-user"></i> {{ trans('admin.setting') }}
        </a>

        <div class="dropdown-divider"></div>

        <a class="dropdown-item" href="{{ admin_url('auth/logout') }}">
            <i class="feather icon-power"></i> {{ trans('admin.logout') }}
        </a>
    </div>
</li>
@endif
```
