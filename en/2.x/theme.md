# Themes and colors

### Switching Themes

`Dcat Admin` supports theme switching function. There are four built-in theme colors: `indigo`, `blue`, `blue-light`, `green`, which can be switched by configuring the parameter `admin.layout.color`.

> {tip} New Color `blue-dark` added in Version `v1.3.0` 

Open configuration file `config/admin.php`
```php
     'layout' => [
         'color' => 'blue',
         
         ...
     ],
     
     ...
```

Preview of some theme colors

<a href="{{public}}/assets/img/screenshots/users-blue-dark.png" target="_blank">
    <img  src="{{public}}/assets/img/screenshots/users-blue-dark.png" style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%">
</a>
<a href="{{public}}/assets/img/screenshots/users-green.png" target="_blank">
    <img  src="{{public}}/assets/img/screenshots/users-green.png" style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%">
</a>


<a name="custom"></a>
### Custom theme coloring

> {tip} Note that if you customize your theme, you'll need to recompile your custom theme every time you update it with a new version!!!!

Developers can use this feature to add their own theme color scheme, before using this feature you need to install [NodeJs](http://nodejs.cn/), if you do not have it installed, go to [http://nodejs.cn/](http://nodejs.cn/) to download and install it.

After installing `NodeJs`, you can run `npm -v` on the command line to test if success is installed.

```bash
npm -v
```

If the version number is returned normally, it means that it was success successfully installed. It is recommended to use Taobao mirror.
```bash
npm config set registry https://registry.npm.taobao.org
```

Then run the following command to compile the custom theme file, just enter the name of the theme and the theme color code (`hexadecimal`).
Here we will generate an `orange` theme as an example.

> {tip} This command takes a long time to run the first time, so please be patient. If it fails, try writing permissions to the `vendor` directory.

```bash
php artisan admin:minify orange --color fbbd08 --publish
```

The above command is meant to generate an `orange` theme with the color code `#fbbd08`, and then automatically publish the static resource after generation. If you compile success, the command line will output the following
```bash
...

 DONE  Compiled successfully in 48001ms8:24:28 PM


                                              Asset      Size  Chunks
               Chunk Names
               /resources/dist/adminlte/adminlte.js  29.7 KiB       0  [emitted]
               /resources/dist/adminlte/adminlte
           /resources/dist/adminlte/adminlte.js.map  87.8 KiB       0  [emitted]
 [dev]         /resources/dist/adminlte/adminlte
               /resources/dist/dcat/extra/action.js   3.7 KiB       1  [emitted]
               /resources/dist/dcat/extra/action
           /resources/dist/dcat/extra/action.js.map  12.9 KiB       1  [emitted]
 [dev]         /resources/dist/dcat/extra/action
          /resources/dist/dcat/extra/grid-extend.js  4.87 KiB       2  [emitted]
               /resources/dist/dcat/extra/grid-extend
      /resources/dist/dcat/extra/grid-extend.js.map  21.7 KiB       2  [emitted]
 [dev]         /resources/dist/dcat/extra/grid-extend
    /resources/dist/dcat/extra/resource-selector.js   5.8 KiB       3  [emitted]
               /resources/dist/dcat/extra/resource-selector
/resources/dist/dcat/extra/resource-selector.js.map    24 KiB       3  [emitted]
 [dev]         /resources/dist/dcat/extra/resource-selector
               /resources/dist/dcat/extra/upload.js  17.2 KiB       4  [emitted]
               /resources/dist/dcat/extra/upload
           /resources/dist/dcat/extra/upload.js.map  66.8 KiB       4  [emitted]
 [dev]         /resources/dist/dcat/extra/upload
                /resources/dist/dcat/js/dcat-app.js  88.8 KiB       5  [emitted]
               /resources/dist/dcat/js/dcat-app
            /resources/dist/dcat/js/dcat-app.js.map   164 KiB       5  [emitted]
 [dev]         /resources/dist/dcat/js/dcat-app
        resources/dist/adminlte/adminlte-orange.css   656 KiB       0  [emitted]
        [big]  /resources/dist/adminlte/adminlte
        resources/dist/dcat/css/dcat-app-orange.css    43 KiB       0  [emitted]
               /resources/dist/adminlte/adminlte
      resources/dist/dcat/extra/markdown-orange.css  1.72 KiB       0  [emitted]
               /resources/dist/adminlte/adminlte
          resources/dist/dcat/extra/step-orange.css  8.56 KiB       0  [emitted]
               /resources/dist/adminlte/adminlte
        resources/dist/dcat/extra/upload-orange.css  6.42 KiB       0  [emitted]
               /resources/dist/adminlte/adminlte
               

Copied Directory [\dcat-admin\resources\dist] To [\public\vendors\dcat-admin]
Publishing complete.
Compiled views cleared!
```

After the theme file compiles success, the following code needs to be added to `app/Admin/bootstrap.php`

```php
Dcat\Admin\Color::extend('orange', [
    'primary'        => '#fbbd08',
    'primary-darker' => '#fbbd08',
    'link'           => '#fbbd08',
]);
```

Finally, set the value of your configuration parameter `admin.layout.color` to `orange`.


<a name="darkmode"></a>
### Dark Mode

![]({{public}}/assets/img/screenshots/users-dark.png)


#### Enable toggle button

The dark mode switch can be enabled or disabled by configuring the parameter `admin.layout.dark_mode_switch`. When enabled, a switch button will be added in the top navigation bar of the page, click on it to switch between dark and bright mode, and the state will be saved in `localStorage`.

```php
     'layout' => [
         'dark_mode_switch' => true,
         
         ...
     ],
     
     ...
```

The result is as follows
![]({{public}}/assets/img/screenshots/dark-switch.gif)

####  Dark Mode by Default

Open the configuration file `config/admin.php`, write
```php
     'layout' => [
         'body_class' => 'dark-mode',
         
         ...
     ],
     
     ...
```

<a name="sidebar"></a>
### Menu style

You can configure the menu style by configuring the parameter `admin.layout.sidebar_style` (if this parameter does not exist in the configuration file, you can add it manually), which supports three values `light`, `primary`, `dark`, and the default is `light`.

> {tip} The `sidebar_dark` parameter is about to be deprecated! The `sidebar_style` parameter overrides the `sidebar_dark` parameter and `sidebar_dark` will only take effect if `sidebar_style` does not exist!!!

```php
     'layout' => [
     	 // Supports light, primary, dark
         'sidebar_style' => 'light',
         
         ...
     ],
     
     ...
```

`light` result

![]({{public}}/assets/img/users.jpg)


`primary` result

![]({{public}}/assets/img/users-menu-primary.jpg)

![]({{public}}/assets/img/users-green-menu-primary.jpg)


### Menu layout

Just add `sidebar-separate` to the `admin.layout.body_class` parameter.

```php
     'layout' => [
         'body_class' => 'sidebar-separate',
         
         ...
     ],
     
     ...

```

result

![]({{public}}/assets/img/users-2.jpg)
![]({{public}}/assets/img/users-dark-2.jpg)



### PHP Color Management

As a daily development we can not do without the use of color, `Dcat Admin` has a built-in color management module, this feature can easily switch with the theme, let the page color and theme color to adapt!

Commonly used colors can be easily obtained through the `Dcat\Admin\Admin::color()` service (see [Color Tables and Styles](theme.md#Color Tables and Styles)).

#### Getting the built-in colors

You can get the color code by using the `Color::get` or magic method. When the color obtained by `Color::get` does not exist, the original value of the parameter is returned.

```php
<?php
use Dcat\Admin\Admin;

// get method to get the color
echo Admin::color()->get('primary'); // output #5c6bc6


// Getting Colors by Magic Method
echo Admin::color()->primary(); // output #5c6bc6
``` 

#### Color fading

The `Color::lighten` method or the magic method can be used to get the hex color code of the faded color.

The `Color::lighten` method takes two arguments.

- `$name` `string` Color Alias
- `$amt` `int` color deviation value, the larger the value, the more `light` color

```php
echo Admin::color()->lighten('primary', 10); // output #6675d0

// You can also use it like this, but be sure to pass negative numbers for the arguments.
echo Admin::color()->primary(-10); // output #6675d0
```

Direct color code transmission is also supported.

```php
echo Admin::color()->lighten('#5c6bc6', 10); // output #6675d0
```

#### color darkening

The `Color::darken` method or the magic method can be used to get the hexadecimal color code of the darkened color.

The `Color::darken` method takes two parameters:

- `$name` `string` Color Alias
- `$amt` `int` color deviation value, the higher the value the darker the color

```php
echo Admin::color()->darken('primary', 10); // output #5261bc

// It can also be used like this
echo Admin::color()->primary(10); // output #5261bc
```

Direct color code transmission is also supported

```php
echo Admin::color()->darken('#5c6bc6', 10); // output #5261bc
```

#### Color Transparency
The `Color::alpha` method allows you to set the transparency of a color.

The `Color::alpha` method takes two arguments:

- `$name` `string` Color Alias
- `$alpha` `float` Transparency, value between `0 ~ 1`, the smaller the value, the higher the transparency

```php
echo Admin::color()->alpha('primary', 0.1); // output rgba(92, 107, 198, 0.1)
```

Direct color code transmission is also supported

```php
echo Admin::color()->alpha('5c6bc6', 0.1); // output rgba(92, 107, 198, 0.1)
```


#### Get all built-in colors

The `Color::all` method gets the hexadecimal code of all built-in colors.

```php
$allColors = Admin::color()->all();
```


### JS Color Management

The `JS` module also includes a color management function, and the `Dcat.color` object allows you to manage colors just like in PHP code.

#### to get the built-in colors

There are three ways to get the color code in the `JS` code
```php
Admin::script(
<<<JS
	// Mode 1
	var primary = Dcat.color.primary;
	
	// Mode 2
	var primary = Dcat.color['primary'];
	
	// Mode 3
	var primary = Dcat.color.get('primary');
	
	console.log(primary);  // output #5c6bc6
JS
);
```

#### Color fading

The `Dcat.color.lighten` method or magic method can be used to get the hex color code of the faded color.

The `color.lighten` method takes two arguments:

- `name` `string` Color Alias
- `amt` `int` color deviation value, the larger the value, the more `light` color

```php
Admin::script(
    <<<JS
    var primary = Dcat.color.lighten('primary', 10)
    
    console.log(primary); // output #6675d0
JS    
);
```

Direct color code transmission is also supported

```js
var primary = Dcat.color.lighten('5c6bc6', 10);
    
console.log(primary); // output #6675d0
```

#### color darkening

The `Dcat.color.darken` method or the magic method can be used to get the hex color code for the darkened color.

The `color.darken` method takes two arguments:

- `name` `string` Color Alias
- `amt` `int` color deviation value, the larger the value, the darker the `darker'

```php
Admin::script(
    <<<JS
    var primary = Dcat.color.darken('primary', 10)
    
    console.log(primary); // output #5261bc
JS    
);
```

Direct color code transmission is also supported

```php
var primary = Dcat.color.darken('5c6bc6', 10)

console.log(primary); // output #5261bc
```

#### Color Transparency
The `Dcat.color.alpha` method allows you to set the transparency of the color.

The `color.alpha` method takes two arguments:

- `$name` `string` Color Alias
- `$alpha` `float` Transparency, value between `0 ~ 1`, the smaller the value, the higher the transparency

```php
Admin::script(
    <<<JS
    var primary = Dcat.color.alpha('primary', 0.1)
    
    console.log(primary); // output rgba(92, 107, 198, 0.1)
JS    
);
```

Direct color code transmission is also supported

```php
var primary = Dcat.color.alpha('#5c6bc6', 0.1)
    
console.log(primary); // output rgba(92, 107, 198, 0.1)
```


#### gets all the built-in colors

The `Dcat.color.all` method, which returns a key-value pair object, retrieves the hexadecimal code for all built-in colors.

```js
var allColors = Dcat.color.all();
```


<a name="Color charts and styles"></a>
### Color charts and styles

The front end of `Dcat Admin` is written using `bootstrap4`, so first we need to learn how to use [Bootstrap4 Color(Color) Style](https://getbootstrap.net/docs/utilities/colors/), and we won't repeat it here.

The following is the `Dcat Admin` common color style sheet, where `.bg-*` is the style of background color, `.text-` is the style of text color.


<style>
	.color-sections {

	}
	.color-section {
		width: 600px;
		height: 60px;
		line-height: 60px;
		text-align: center;
		vertical-align: middle;
		/*display: inline-block;*/
		margin-bottom: 5px;
	}

	.white {
		color: #fff;
	}
</style>
 <section class="container color-sections" style="min-height: 500px">
        <div class="color-section white" style="background: #5c6bc6">
            <code>.text-primary</code> <code>.bg-primary</code> primary/indigo
        </div>
        <div class="color-section white" style="background: #495abf">
            <code>.text-primary-darker</code> indigo-darker
        </div>

        <div class="color-section white" style="background: #5b69bc">
            purple
        </div>

        <div class="color-section white" style="background: #7367f0">
            cyan
        </div>
        <div class="color-section white" style="background: #6355ee">
            cyan-darker
        </div>

        <div class="color-section white" style="background: #3085d6">
            <code>.text-info</code> <code>.bg-info</code> blue/info
        </div>
        <div class="color-section white" style="background: #236bb0">
            <code>.text-blue-darker</code> blue-darker
        </div>
        <div class="color-section white" style="background: #007ee5">
            <code>.text-blue-1</code> <code>.bg-blue-1</code> blue1
        </div>
        <div class="color-section white" style="background: #4199de">
            <code>.text-blue-2</code> <code>.bg-blue-2</code> blue2
        </div>

        <div class="color-section white" style="background: #59a9f8">
            <code>.text-custom</code> <code>.bg-custom</code> custom
        </div>

        <div class="color-section white" style="background: #21b978">
            <code>.text-success</code> <code>.bg-success</code> green/success
        </div>
        <div class="color-section white" style="background: #ea5455">
            <code>.text-danger</code> <code>.bg-danger</code> danger/red
        </div>
        <div class="color-section white" style="background: #bd4147">
            <code>.text-danger-darker</code> red-darker
        </div>


        <div class="color-section white" style="background: #dda451">
            <code>.text-warning</code> <code>.bg-warning</code> warning/orange
        </div>


        <div class="color-section white" style="background: #ffcc80">
            <code>.text-orange-1</code> <code>.bg-orange-1</code> orange1
        </div>
        <div class="color-section white" style="background: #F99037">
            <code>.text-orange-2</code> <code>.bg-orange-2</code> orange2
        </div>
        <div class="color-section white" style="background: #edc30e">
            <code>.text-yellow</code> <code>.bg-yellow</code> yellow
        </div>

        <div class="color-section white" style="background: #ff8acc">
            <code>.text-pink</code> <code>.bg-pink</code> pink
        </div>


        <div class="color-section white" style="background: #01847f">
            <code>.text-tear</code> <code>.bg-tear</code> tear
        </div>
        <div class="color-section white" style="background: #00b5b5">
            <code>.text-tear-1</code> <code>.bg-tear-1</code> tear1
        </div>


        <div class="color-section white" style="background: #22292f">
            dark
        </div>

        <div class="color-section white" style="background: #b9c3cd">
            <code>.text-gray</code> <code>.bg-gray</code> gray
        </div>
        <div class="color-section white" style="background: #f7f7f9;color: #666">
            <code>.text-light</code> <code>.bg-light</code> light
        </div>
        <div class="color-section white" style="background: #f6fbff;color: #666">
            <code>.text-dark20</code> <code>.bg-dark20</code> dark20
        </div>
        <div class="color-section white" style="background: #f4f7fa;color: #666">
            <code>.text-dark30</code> <code>.bg-dark30</code> dark30
        </div>
        <div class="color-section white" style="background: #e7eef7;color: #666">
            <code>.text-dark35</code> <code>.bg-dark35</code> dark35
        </div>

        <div class="color-section white" style="background: #ebf0f3;color: #666">
            <code>.text-dark40</code> <code>.bg-dark40</code> dark40
        </div>
        <div class="color-section white" style="background: #d3dde5;color: #666">
            <code>.text-dark50</code> <code>.bg-dark50</code> dark50
        </div>
        <div class="color-section white" style="background: #bacad6">
            <code>.text-dark60</code> <code>.bg-dark60</code> dark60
        </div>
        <div class="color-section white" style="background: #b3b9bf">
            <code>.text-dark70</code> <code>.bg-dark70</code> dark70
        </div>
        <div class="color-section white" style="background: #7c858e">
            <code>.text-dark80</code> <code>.bg-dark80</code> dark80
        </div>
        <div class="color-section white" style="background: #5c7089">
            <code>.text-dark85</code> <code>.bg-dark85</code> dark85
        </div>
        <div class="color-section white" style="background: #252d37">
            dark90
        </div>
        <div class="color-section white" style="background: #414750">
            font字体颜色
        </div>
        <div class="color-section white" style="background: #f1f1f1;color: #666">
            gray-bg
        </div>
        <div class="color-section white" style="background: #ebeff2;color: #666">
            border
        </div>
        <div class="color-section white" style="background: #d9d9d9;color: #666">
            input-border
        </div>
    </section>
