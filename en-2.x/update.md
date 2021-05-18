# Version upgrade instructions


### Description

The release of `Dcat Admin` will take into account the release strategy of the mainstream `web framework` to minimize the impact of version upgrades, minor versions and patches **never** contain non-compatibility changes; we will also provide an update log detailing the changes and possible impact of the new version.




### Upgrade command
upgrade command
```bash
composer update dcat/laravel-admin
```

After successful upgrade, you need to run ``admin:update`` to republish the language pack, configuration files, front-end static resources and other files, and then **clean the browser cache**

```bash
# Publish language packs, configuration files, front-end static resources, data migration files, etc.
php artisan admin:update
```

Running ``admin:update`` is equivalent to running

```bash
php artisan admin:publish --assets --migrations --lang --force
php artisan migrate
```

#### publish file command

> After running ``admin:update`'' you don't usually need to run the ``admin:publish`'' command

```bash
php artisan admin:publish --force
```

Only language pack updates
```bash
php artisan admin:publish --force --lang
```

Update Configuration Files Only
```bash
php artisan admin:publish --force --config
```


Only front-end static resources are updated
```bash
php artisan admin:publish --force --assets
```

Update only the database migration file (this generally does not need to be updated)
```bash
php artisan admin:publish --force --migrations
```