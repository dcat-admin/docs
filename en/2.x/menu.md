# Menu

The menus of `Dcat Admin` are stored on the datasheet `admin_menu` and can be managed in the background menu management page.

### Menu Permissions
Each menu can be tied to a permission or role, or if not set it will be a public menu, visible to all accounts.


The `admin.menu.bind_permission` configuration parameter allows you to set whether or not to allow bind permissions.
> {tip} By default, a menu can bind up to one permission and one role.

### Menu translation
Append the menu TITLE to the menu_titles index of your language file. For example, the "Workplace" TITLE:

In `resources/lang/{current language}/admin.php`
```php
...
'menu_titles' => [
    'work_units' => 'Unidades de trabajo'
],
```

### Menu cache
The menu cache can be turned on or off through the `admin.menu.cache.enable` configuration parameter, it is recommended to turn it on.

### Add menu dynamically through Menu::add interface
`Dcat Admin` also provides the ability to instantly add menus to the code by means of arrays.

Add the following code to `app\Admin\bootstrap.php`:
```php
<?php
use Dcat\Admin\Admin;
use Dcat\Admin\Layout\Menu;

Admin::menu(function (Menu $menu) {
    $menu->add([
        [
            'id'            => '1', // This id just needs to be unique in the current array.
            'title'         => 'Test Menu',
            'icon'          => 'fa-file-text-o',
            'uri'           => '',
            'parent_id'     => 0, 
            'permission_id' => 'test', // Binding to permissions
            'roles'         => 'test-roles', // Binding to roles
        ],  
        [
            'id'            => '2', // This id just needs to be unique in the current array.
            'title'         => 'Test Menu 2',
            'icon'          => 'fa-file-text-o',
            'uri'           => 'test-menu2',
            'parent_id'     => '1', 
        ],  
    ]);
});

```

### Why don't menus with no permissions automatically hide?

This problem is because you don't bind permissions or roles to menus, just bind permissions or roles to menus that you want to show without permissions.


