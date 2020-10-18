# 开发扩展

## 新版本预告
`Dcat Admin`计划在`2.0`版本上线插件市场功能，将对整个扩展功能进行重构，以提升用户体验。
新的扩展系统将可以让用户只需在管理页面点点鼠标即可完成插件的`安装`、`更新`和`卸载`等操作。
并且会上线插件付费功能，以激励开发者开发高质量的插件。

如果有任何建议，欢迎提`issue`或者私信我，`Dcat Admin`团队将会致力于构建一个于开发者和用户都有利的生态，感谢大家的支持！


<a name="pre"></a>
## 开始
`Dcat Admin`支持安装扩展工具来帮助丰富你的后台功能。
>需要注意的是，`Laravel Admin`原有的扩展无法直接在`Dcat Admin`中使用，但大部分扩展只需要做一些微小的调整就可以正常使用了，有兴趣的同学可以自行移植。

如果大家在使用的过程中有在`Dcat Admin`的基础上添加一些自己的功能或者组件，不妨做成一个`Dcat Admin`扩展，这样可以给其它`Dcat Admin`使用者提供帮助，并且在其它人的使用反馈中的提升扩展的质量。

这篇文档将会以开发一个`干货集中营`扩展为例，一步一步的开发一个扩展，并且发布给他人使用，最终的效果参考[Demo]()。

<a name="composer"></a>
## 创建composer包

`Dcat Admin`的包将会用`composer`安装，所以先要创建一个`composer`包，可以用内置的`admin:extend`命令来生成一个扩展的骨架。


运行命令的时候，可能会提示输入一个目录来存放你的扩展文件，你可以在`config/admin.php`里面增加一个配置`'extension_dir' => admin_path('Extensions')`，这样扩展文件将会存放在`app/Admin/Extensions目录下`,当然你也可以放在任何其它目录。


```php
php artisan admin:extend dcat-admin-extensions/gank --namespace="Dcat\Admin\Extension\Gank"
```

其中`dcat-admin-extensions/gank`是包名，`namespace`选项是这个包使用的顶级命名空间，运行这个命令之后, 将会在在`config/admin.php`中设置的扩展目录中生成目录`dcat-admin-extensions/gank`和下面的文件结构：

    ├── LICENSE
    ├── README.md
    ├── composer.json
    ├── bootstrap.php
    ├── database
    │   ├── migrations
    │   └── seeds
    ├── resources
    │   ├── assets
    │   └── views
    │       └── index.blade.php
    ├── routes
    │   └── web.php
    └── src
        ├── Gank.php
        ├── GankServiceProvider.php
        └── Http
            └── Controllers
                └── GankController.php

`bootstrap.php`用于注册扩展，此文件的代码不需要做任何修改；`resources`用来放置视图文件和静态资源文件；`src`主要用来放置逻辑代码； `routes/web.php`用来存放这个扩展的路由设置，`database`用来放置数据库迁移文件和数据`seeders`。

<a name="dev"></a>
## 功能开发

这个扩展的功能主要用来展示[gank.io](http://gank.io)接口的内容，集成进`Dcat Admin`中，它将会有一个路由和一个控制器，没有数据库文件、模板和静态资源文件，我们可以将没有用到的文件或目录清理掉，清理之后的目录文件为：

    ├── LICENSE
    ├── README.md
    ├── composer.json
    ├── bootstrap.php
    ├── routes
    │   └── web.php
    └── src
        ├── Gank.php
        ├── GankServiceProvider.php
        └── Http
            └── Controllers
                └── GankController.php
                
生成完扩展框架之后，你可能需要一边调试一边开发，所以可以参考下面的本地安装，先把扩展安装进系统中，继续开发

<a name="route"></a>
### 添加路由
首先添加一个路由，在`routes/web.php`中已经自动生成好了一个路由配置

```php
<?php

use Dcat\Admin\Extension\Gank\Http\Controllers;

Route::get('gank', Controllers\GankController::class.'@index');
```

访问路径`http://localhost:8000/admin/gank`，将会由`Dcat\Admin\Extension\Gank\Http\Controllers\GankController`控制器的`index`方法来处理这个请求。

<a name="attrs"></a>
### 设置扩展属性
`src/Gank.php`作为扩展类，用来设置扩展的属性

```php
<?php

namespace Dcat\Admin\Extension\Gank;

use Dcat\Admin\Extension;

class Gank extends Extension
{
    // 扩展别名，用于在配置文件中获取配置信息
    const NAME = 'gank';

    // service provider类名，这个用自动生成的就行，不需要改
    public $serviceProvider = GankServiceProvider::class;

    // 静态资源目录，如果不需要静态资源可用删除掉  
//    public $assets = __DIR__.'/../resources/assets';

    // composer配置文件路径，用默认的即可
    public $composerJson = __DIR__.'/../composer.json';

    // 自定义属性
    public static $categoryColorsMap = [
        'App'      => 'var(--purple)',
        '前端'     => 'var(--primary)',
        '拓展资源' => 'var(--primary-dark)',
        '瞎推荐'   => 'var(--blue)',
        '福利'     => 'var(--danger)',
        'Android'  => 'var(--purple-dark)',
        'iOS'      => 'var(--info)',
        '休息视频' => 'var(--warning)',
    ];
}

```
这个文件用来设置这个扩展的一些属性，`$name`是这个扩展的别名，如果这个扩展有视图文件需要渲染，则必须指定这个扩展的`$views`属性，同样如果有静态资源文件需要发布，则必须设置`$assets`属性， 如果需要在左侧边栏增加一项菜单按钮，设置`$menu`属性，可以根据需要去掉不必要的属性。

然后打开`src/GankServiceProvider.php`，这个`ServiceProvider`将会在启用扩展的时候运行（不需要开发者自己添加），用来将这个扩展的一些服务注册进系统中。

<a name="loadview"></a>
### 加载视图
如果这个扩展需要加载视图文件，在`src/GankServiceProvider.php`的`boot`方法中加入以下的代码：

```php
$extension = Gank::make();

if ($extension->views) {
    $this->loadViewsFrom($extension->views, 'gank');
}
```
`loadViewsFrom`方法的的第一个参数为在扩展类`src/Gank.php`中设置的视图属性，第二个参数是视图文件目录的命名空间，设置为`gank`之后，在控制器中用`view('gank::index')`来加载`resources/views`目录下的视图文件。

<a name="loadassets"></a>
### 引入静态资源
如果你的项目中有静态资源文件需要引入，先把文件放在`resources/assets`目录中，比如放入`resources/assets/foo.js`和`resources/assets/bar.css`这两个文件。

接着在扩展类`src/Gank.php`中设置`$assets`属性即可：
```php
public $assets = __DIR__.'/../resources/assets';
```
    
安装完成之后，文件将会复制到`public/vendor/dcat-admin-extensions/gank`目录中。

然后我们可以在需要使用这些静态资源的地方加上下面的代码即可

> {tip} 因为`Dcat Admin`支持静态资源按需加载，所以不推荐直接在`ServiceProvider`或者是`bootstrap.php`中引入静态资源，这样会拖慢所有页面的加载时间。

```php
use Dcat\Admin\Admin;

Admin::js('vendor/dcat-admin-extensions/gank/phpinfo/foo.js');
Admin::css('vendor/dcat-admin-extensions/gank/phpinfo/bar.css');
```

这样就完成了静态资源的引入，在`gank`这个扩展中，由于没有静态资源需要引入，所以可以忽略掉这一步。

<a name="addmenu"></a>
### 添加菜单
添加菜单的方式有两种：
- 在`src/Gank.php`设置`$menu`属性，这种方式是把菜单写入到菜单表中，用户更易于控制
- 使用`Admin::menu`接口通过数组的方式添加菜单，这种方式更加方便，并且支持添加多级菜单

这里使用的是第二种方式添加菜单，在`src/GankServiceProvider.php`的`boot`方法加入下面的代码：
```php
Admin::menu()->add([
    [
        'id'            => 1,
        'title'         => '干货集中营',
        'icon'          => ' fa-newspaper-o',
        'uri'           => 'gank',
        'parent_id'     => 0,
        'permission_id' => 'gank', // 绑定权限
        'roles'         => [['slug' => 'gank']], // 绑定角色
    ],
    [
        'id'            => 2,
        'title'         => '所有干货',
        'icon'          => 'fa-smile-o',
        'uri'           => 'gank',
        'parent_id'     => 1,
        'permission_id' => 'gank', // 绑定权限
        'roles'         => [['slug' => 'gank']], // 绑定角色
    ],
]);
```

最终的`src/GankServiceProvider.php`代码如下：
```php
<?php

namespace Dcat\Admin\Extension\Gank;

use Dcat\Admin\Admin;
use Illuminate\Support\ServiceProvider;

class GankServiceProvider extends ServiceProvider
{
    /**
     * {@inheritdoc}
     */
    public function boot()
    {
        $extension = Gank::make();

        if ($extension->views) {
            $this->loadViewsFrom($extension->views, 'gank');
        }

        if ($extension->lang) {
            $this->loadTranslationsFrom($extension->lang, 'gank');
        }

        if ($extension->migrations) {
            $this->loadMigrationsFrom($extension->migrations);
        }

        $this->app->booted(function () use ($extension) {
            $extension->routes(__DIR__.'/../routes/web.php');
        });

        // 添加菜单
        $this->registerMenus();
    }

    protected function registerMenus()
    {
        Admin::menu()->add([
            [
                'id'            => 1,
                'title'         => '干货集中营',
                'icon'          => ' fa-newspaper-o',
                'uri'           => 'gank',
                'parent_id'     => 0,
                'permission_id' => 'gank', // 绑定权限
                'roles'         => [['slug' => 'gank']], // 绑定角色
            ],
            [
                'id'            => 2,
                'title'         => '所有干货',
                'icon'          => 'fa-smile-o',
                'uri'           => 'gank',
                'parent_id'     => 1,
                'permission_id' => 'gank', // 绑定权限
                'roles'         => [['slug' => 'gank']], // 绑定角色
            ],
        ]);
    }
}
```

<a name="importcode"></a>
## 添加自定义安装逻辑
如果你想在导入扩展时执行一些自定义逻辑，可以在`src/Gank.php`中添加以下代码：

>注意请保证重复执行导入命令不会报错

```php
...

    public function import(Command $command)
    {
        parent::import($command); // TODO: Change the autogenerated stub
        
        // 在这里写入你的导入逻辑
        
        $command->info('导入成功');
        
    }

```

<a name="code"></a>
## 代码逻辑开发
这个扩展主要是调用[gank.io]()提供的接口，再把数据展示出来。

我们需要新增一个数据仓库文件用于获取[gank.io]()的数据，创建文件`src/Repositories/Ganks.php`：
```php
<?php

namespace Dcat\Admin\Extension\Gank\Repositories;

use Dcat\Admin\Grid;
use Dcat\Admin\Repositories\Repository;
use Illuminate\Pagination\LengthAwarePaginator;
use Illuminate\Support\Collection;

class Ganks extends Repository
{
    protected $api = 'http://gank.io/api/data/{category}/{limit}/{page}';

    protected $searchApi = 'http://gank.io/api/search/query/{key}/category/{category}/count/{limit}/page/{page}';

    /**
     * 查询表格数据
     *
     * @param Grid\Model $model
     * @return Collection
     */
    public function get(Grid\Model $model)
    {
        $currentPage = $model->getCurrentPage();
        $perPage = $model->getPerPage();

        // 获取筛选参数
        $category = $model->filter()->input(Grid\Filter\Scope::QUERY_NAME, 'all');
        $keyword = trim($model->filter()->input('keyword'));

        $api = $keyword ? $this->searchApi : $this->api;

        $client = new \GuzzleHttp\Client();

        $response = $client->get(str_replace(
            ['{category}', '{limit}', '{page}', '{key}'],
            [$category, $perPage, $currentPage, $keyword],
            $api
        ));
        $data = collect(
            json_decode((string)$response->getBody(), true)['results'] ?? []
        );

        $total = $keyword ? 400 : ($category == 'all' ? 1000 : 500);

        $paginator = new LengthAwarePaginator(
            $data,
            $category == '福利' ? 680 : $total,
            $perPage, // 传入每页显示行数
            $currentPage // 传入当前页码
        );

        $paginator->setPath(\url()->current());

        return $paginator;
    }

    public function getKeyName()
    {
        return '_id';
    }

}
```

修改控制器代码如下：

```php
<?php

namespace Dcat\Admin\Extension\Gank\Http\Controllers;

use Dcat\Admin\Extension\Gank\Gank;
use Dcat\Admin\Extension\Gank\Repositories\Ganks;
use Dcat\Admin\Grid;
use Dcat\Admin\Layout\Content;
use Dcat\Admin\Layout\Row;
use Illuminate\Routing\Controller;
use Dcat\Admin\Widgets\Navbar;

class GankController extends Controller
{
    public function index(Content $content)
    {
        $grid = $this->grid();

        $grid->disableFilter();
        $grid->filter()
            ->withoutInputBorder()
            ->expand()
            ->resetPosition()
            ->hiddenResetButtonText();

        return $content
            ->header('所有干货')
            ->description('每日分享妹子图 和 技术干货')
            ->body($grid->filter())
            ->body(function (Row $row) {
                $items = array_keys(Gank::$categoryColorsMap);
                
                array_unshift($items, '全部');
                
                $navbar = Navbar::make('#', array_combine($items, $items))
                    ->checked(request(Grid\Filter\Scope::QUERY_NAME, '全部'))
                    ->click()
                    ->map(function ($v) {
                        if ($v == '全部') {
                            $url = '?';
                        } else {
                            $url = '?'.Grid\Filter\Scope::QUERY_NAME.'='.$v;
                        }
        
                        return "<a href='$url'>$v</a>";
                    })
                    ->style('max-width:705px');
                
                $row->column(7, $navbar);

            })
            ->body($grid);
    }

    protected function grid()
    {
        $this->define();

        $grid = new Grid(new Ganks);

        $grid->number();
        $grid->desc('描述')->width('300px');
        $grid->images('图片')->image(150);
        $grid->type('类别');
        $grid->who('作者')->label();
        $grid->publishedAt('发布于');

        $grid->disableActions();
        $grid->disableBatchDelete();
        $grid->disableExport();
        $grid->disableCreateButton();
        $grid->disableFilterButton();
        $grid->disableQuickCreateButton();
        $grid->perPages([]);

        return $grid->filter(function (Grid\Filter $filter) {
            $category = $filter->input(Grid\Filter\Scope::QUERY_NAME, '全部');

            if ($category != '福利') {
                $filter->like('keyword', ucfirst($category))->width('300px')->placeholder('请输入');
            }

        });
    }

    protected function define()
    {
        Grid\Column::define('desc', function ($v) {
            if ($this->type == '福利') {
                $width = '150';
                $height = '200';
                return "<img data-init='preview' src='{$this->url}' style='max-width:{$width}px;max-height:{$height}px;cursor:pointer' class='img img-thumbnail' />";
            }

            return sprintf('<a href="%s" target="_blank">%s</a>', $this->url, $v);
        });

        Grid\Column::define('publishedAt', function ($v) {
            return date('Y-m-d', strtotime($v));
        });
        Grid\Column::define('datetime', function ($v) {
            return date('Y-m-d H:i:s', strtotime($v));
        });

        Grid\Column::define('type', function ($v) {
            $map = Gank::$categoryColorsMap;

            return "<span class='label' style='background:{$map[$v]}'>$v</span>";
        });
    }
}
```
这样一个完整的扩展就开发完成了。

<a name="updatejson"></a>
## 修改 composer.json & README.md
代码部分完成之后，需要修改`composer.json`里面的内容，将`description`、`keywords`、`license`、`authors`等内容替换为你的信息，然后不要忘记完善`README.md`，补充使用文档等相关信息。

<a name="install"></a>
## 安装
完成了扩展开发之后，根据情况可以用下面的的方式安装你的扩展

<a name="local"></a>
### 本地安装
在开发的过程中，一般需要一边调试一边开发，所以先按照下面的方式进行本地安装

打开你的项目中`composer.json`文件，在加入下面的配置

```
"repositories": [
    {
        "type": "path",
        "url": "app/Admin/Extensions/dcat-admin-extensions/gank"
    }
]
```
然后运行`composer require dcat-admin-extensions/gank @dev`完成安装，如果有静态文件需要发布，运行下面的命令:

```
php artisan vendor:publish --provider=Dcat\Admin\Extension\GankServiceProvider
```

<a name="remote"></a>
### 远程安装
如果开发完成之后，希望开源出来给大家使用，按照下面的步骤进行

<a name="github"></a>
#### 上传到Github
先登录你的Github，创建一个仓库，然后按照页面上的提示把你的代码push上去

```
git init
git remote add origin https://github.com/<your-name>/<your-repository>.git
git add .
git commit -am "Initial commit."
git push origin master
```

<a name="release"></a>
#### 发布release
可以用下面的方式在本地发布版本

```
git tag 0.0.1 && git push --tags
```
也可以在Github的仓库页面的Releases页面手动设置

<a name="packagist"></a>
#### 发布到Packagist.org
接下来就是发布你的项目到`Packagist.org`，如果没有账号的话，先注册一个，然后打开顶部导航的`Submit`, 填入仓库地址提交

默认情况下，当您推送新代码时，`Packagist.org`不会自动更新，所以，您需要创建一个GitHub服务钩子， 你也可以使用点击页面上的`Update`按钮手动更新它，但我建议自动执行这个过程

提交之后，由于各地的镜像同步时间的延迟，可能在用composer安装的时候，会暂时找不到你的项目，这个时候可能需要等待同步完成

发布完成之后就可以通过composer安装你的扩展了

加入到https://github.com/dcat-admin-extensions
如果想把你开发的扩展加入到`dcat-admin-extensions`，欢迎用各种方式联系我，这样可以让更多人看到并使用你开发的工具。

<a name="import"></a>
## 导入和启用扩展

<a name="importext"></a>
### 导入扩展
安装完成后，运行以下命令即可以导入扩展，导入的文件主要有：静态资源文件、菜单、权限等。

>视图、数据库迁移文件、语言包一般不需要导入。如果要导入其他文件，需要扩展开发者自行定义。
>另外此命令允许重复执行。

```php
php artisan admin:import Dcat\Admin\Extension\Gank\Gank
```

<a name="enable"></a>
### 启用扩展
扩展的启用与否是通过配置文件控制的，打开`/config/admin-extension.php`，加入以下代码：
```php
return [
    'gank' => [
        'enable' => true,
    ],
];
```
<a name="view"></a>
### 可视化管理扩展
访问`http://localhost:8000/admin/helpers/extensions`，可以通过页面操作启用或关闭扩展、导入扩展。
>如果`/config/admin-extension.php`不存在，系统会自动创建


这样就完成了安装，打开`http://localhost/admin/gank`访问这个扩展。

