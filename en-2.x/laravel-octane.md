# Laravel Octane

[Laravel Octane](https://github.com/laravel/octane) is a project based on `Swoole/RoadRunner` driver that can improve the performance of `Laravel` framework, after installation it can significantly improve the performance of `Laravel` projects.

`Dcat Admin` is compatible with the `Laravel Octane` environment, just add the following configuration to the configuration file `config/octane.php`.

```php

    'listeners' => [
        ... ,

        RequestReceived::class => [
            ... .Octane::prepareApplicationForNextOperation(),
            ... .Octane::prepareApplicationForNextRequest(),
            
            // Enable support for Dcat Admin
            Dcat\Admin\Octane\Listeners\FlushAdminState::class,
        class],
        
        ...
    ],    
```

> [Laravel Octane](https://github.com/laravel/octane) is still in `beta` phase, for installation and more information about [Laravel Octane](https://github.com/laravel/octane) Please go to the documentation at https://github.com/laravel/octane for more information.

