# Development tools

`Dcat Admin` provides a number of development tools to help developers improve development efficiency.
If you need to disable the development tools, just set `app.debug` or `admin.helpers.enable` to `false` in the configuration file.


### Code generators

The code generator can generate add, delete, redact, and check codes from the interface with one click. It supports generating add, delete, redact, and check codes from existing data tables.

> If your development environment is not `windows`, please give `777` permission to the whole project, otherwise you may not be able to generate the file.


### Extension package management
`Dcat Admin` supports visual management of extensions. As long as the extensions installed through `composer` can be seen in the management interface, the extensions can be enabled and imported through the interface, open the browser and visit `http://localhost:8000/admin/helpers/extensions` to use it.

### IDE autocomplete
The `php artisan admin:ide-helper` command can generate IDE autocomplete files, which can generate hints for `Grid`, `Form`, `Show` and other functions.

### Icon
Visit `http://localhost:8000/admin/helpers/icons` for a list of supported icons.

