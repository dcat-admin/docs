# Laravel Octane

[Laravel Octane](https://github.com/laravel/octane) 是一个基于 `Swoole/RoadRunner` 驱动的可以提升 `Laravel` 框架性能的项目，安装后可以大幅提升`Laravel`项目的性能。

`Dcat Admin`兼容了`Laravel Octane`环境，只需在配置文件`config/octane.php`中加入如下配置即可：

```php

    'listeners' => [
        ...,

        RequestReceived::class => [
            ...Octane::prepareApplicationForNextOperation(),
            ...Octane::prepareApplicationForNextRequest(),
            
            // 开启对 Dcat Admin 的支持
            Dcat\Admin\Octane\Listeners\FlushAdminState::class,
        ],
        
        ...
    ],    
```

> [Laravel Octane](https://github.com/laravel/octane)目前仍处于`beta`版本阶段，关于[Laravel Octane](https://github.com/laravel/octane)的安装与更多介绍请前往文档 https://github.com/laravel/octane 查看。

