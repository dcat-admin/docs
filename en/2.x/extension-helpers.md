# Development tools

In the latest version of the new developer-oriented help tools have been added to provide help in development to improve efficiency, currently provides `scaffolding`, `database command line` and `artisan command line` three tools, if there are better ideas for other utilities, welcome to provide suggestions.

Installation:
```php
composer require dcat-admin-ext/helpers

php artisan admin:import helpers
```

> {tip} Some of the functions of the tool will create or delete files in the project, and there may be issues with file or directory permissions, which will need to be resolved on your own.
Also some of the database and artisan commands are not available in a web environment.


## Scaffolding Tools

The scaffolding tool helps you to generate controller, model, migration file in one click, and run the migration file, visit `http://localhost/admin/helpers/scaffold` to open it.

In which the primary key field is automatically generated without filling when setting the migration table structure.

![qq20170220-2](https://cloud.githubusercontent.com/assets/1479100/23147949/cbf03e84-f81d-11e6-82b7-d7929c3033a0.png)

## Database command line

Database command-line tool for web integration, currently supports `mysql`, `mongodb` and `redis`, visit `http://localhost/admin/helpers/terminal/database` to open.

In the upper right corner of the `select` selection box to switch the database connection, and then at the bottom of the input box enter the corresponding database query statement and then enter, you can get the results of the query: `mysql`, `mongodb` and `redis`.

![qq20170220-3](https://cloud.githubusercontent.com/assets/1479100/23147951/ce08e5d6-f81d-11e6-8b20-605e8cd06167.png)

The utility is identical to the database manipulation on the terminal, allowing you to run the supported queries for the selected database.

## artisan command line tool

A web implementation of the `artisan` command for `Laravel` can be opened by running the artisan command on it and visiting `http://localhost/admin/helpers/terminal/artisan`.

![qq20170220-1](https://cloud.githubusercontent.com/assets/1479100/23147963/da8a5d30-f81d-11e6-97b9-239eea900ad3.png)


## Routing list

This tool can be used to visually display all the routes in the system, including the route uri, methods and middleware, as well as querying routes. Visit `http://localhost/admin/helpers/routes` to open.

![helpers_routes](https://user-images.githubusercontent.com/1479100/30899066-e8bdd5ca-a390-11e7-809d-4ceccd0da27f.png)
