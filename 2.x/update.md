# 版本升级须知


### 说明

`Dcat Admin`的版本发行将会参考主流`web框架`的发行策略，尽量降低版本升级带来的影响，小版本和补丁**决不**包含非兼容性更改；同时我们也将会提供更新日志，详细说明新版本的改动以及可能造成的影响。




### 升级命令
升级命令
```bash
composer update dcat/laravel-admin
```

升级成功之后需要运行 `admin:update` 命令进行重新发布语言包、配置文件、前端静态资源等文件，然后**清理浏览器缓存**

```bash
# 发布 语言包、配置文件、前端静态资源、数据迁移文件等
php artisan admin:update
```

运行 `admin:update`，相当于运行

```
php artisan admin:publish --assets --migrations --lang --force
php artisan migrate
``` 

#### 发布文件命令

> 运行 `admin:update` 后一般不需要运行 `admin:publish` 命令

```bash
php artisan admin:publish --force
```

只更新语言包
```bash
php artisan admin:publish --force --lang
```

只更新配置文件
```bash
php artisan admin:publish --force --config
```


只更新前端静态资源
```bash
php artisan admin:publish --force --assets
```

只更新数据库迁徙文件(这个一般不需要更新)
```bash
php artisan admin:publish --force --migrations
```