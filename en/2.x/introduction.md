# Dcat Admin


`Dcat Admin` is a backend system builder based on [laravel-admin](https://www.laravel-admin.org/) that allows you to quickly build a full-featured, high-value backend system with very little code. Supports button click CRUD code generation, built-in rich backend components, out of the box, allowing developers to say goodbye to redundant HTML code, very friendly to back-end developers.

<p align="center">
    <a href="https://github.com/jqhph/dcat-admin/blob/master/LICENSE"><img style="display:inline" src="https://img.shields.io/badge/license-MIT-7389D8.svg?style=flat" ></a>
    <a href="https://travis-ci.org/jqhph/dcat-admin">
        <img style="display:inline"  src="https://travis-ci.org/jqhph/dcat-admin.svg?branch=master" alt="Build Status">
    </a>
    <a href="https://packagist.org/packages/dcat/laravel-admin">
    	<img style="display:inline"  src="https://img.shields.io/packagist/dt/dcat/laravel-admin.svg?color=" /></a> 
    <a><img style="display:inline"  src="https://img.shields.io/badge/php-7.1+-59a9f8.svg?style=flat" /></a> 
    <a><img style="display:inline"  src="https://img.shields.io/badge/laravel-5.5+-59a9f8.svg?style=flat" ></a>
</p>

### Technology stack

- [Laravel](https://laravel.com/)
- [AdminLTE3](https://github.com/ColorlibHQ/AdminLTE)
- [Bootstrap4](https://getbootstrap.net/)
- jQuery3


### Features

- Simple, elegant, flexible and scalable API
- user management
- RBAC permission management, supports unlimited permission nodes.
- Menu management
- build without refreshing the page using pjax to support **load on-demand** static resources , you can infinitely expand the component without affecting the overall performance
- Loosely coupled page building and data manipulation design, with easy switching of data sources
- Custom pages
- Custom theme coloring
- Multi-theme switch function, built-in multiple theme colors
- Easily build standalone pages without a menu bar (e.g. for building pop-up selectors, etc.)
- Plug-in Features
- Visual code generator to generate add/delete pages based on data tables with one click.
- Data table builder with a rich set of built-in table features (e.g. combobox headers, data export, search, quick create, batch operations, etc.)
- Tree table feature builder with pagination and click-to-load support
- Data form builder, rich built-in form types, support for asynchronous submission of forms
- Step-by-step form builder
- Popup form builder
- Data detail page builder
- Infinite hierarchical tree page builder with drag and drop support for data hierarchy, sorting and other operations
- Rich built-in common page components (such as charts, statistics cards, drop-down menus, tab cards, hint tools, etc.)
- `Section` function (similar to `Filter` of `Wordpress` and `section` tab of `blade` template).
- Asynchronous file upload form, support block multi-threaded uploading
- Multi-application (multi-backend)
- Plugin Marketplace, which allows you to install, update and uninstall plugins with a single click of the mouse on the administration page (`not implemented yet`).



### Similarities and Differences with Laravel Admin

``Dcat Admin`` is a backend builder based on ``Laravel Admin``. The overall style is the same as ``Laravel Admin``, but with a lot of tweaks in the details.


Adjustments:
- Building front-end pages using [AdminLTE3](https://github.com/ColorlibHQ/AdminLTE)(`bootstrap4` + `jQuery3`)
- Use `PJAX` to build refresh-free pages, and support front-end resources on-demand loading , developers no longer need to worry about installing too many components will affect the page load speed
- Loosely coupled page construction and data manipulation design, building pages no longer requires concern for the specific implementation of the data manipulation interface.
- Adjusted the form submission method to `ajax` submission
- Adjusted the code generator to support the generation of add/delete pages from existing data tables with one click.
- Adjusted the multi-language translation function for easier use.
- Adjusted permissions function, support for hierarchy and sorting
- Adjusted extension system to support page management
- ...

Added:
- New multi-theme switching function
- New form popups allow you to build a non-Iframe form popup with just a few lines of code.
- Quickly build pages without menu bar
- New popup selector form, you can select form data in a popup window.
- New AJAX form submission and form front-end validation features.
- Add asynchronous file upload component, support block upload, batch upload, upload progress bar, etc.
- New slide panel layout to the right of the table filter
- New form field value filtering feature
- New step-by-step form
- New `section` feature (similar to `wordpress` `add_filter` feature)
- New tree table function to display large quantities of hierarchical data by page.
- New dual header table feature allows you to build dual header tables with just a few lines of code.
- Added a variety of useful page components, such as charts, drop-down menus, markdowns, checkboxes, etc.
- New `Tree` form
- Add the ability to add menus by array, support for binding permissions and roles.
- Add menu function by array
- New menu cache function
- ...

### Exchange

**QQQQQun** 704661955

**Extension Developer QQ group** 679738409

### Join us

If you are interested in this project, you are very welcome to join the project development team and participate in the maintenance and development of the features of this project. Contributions of any kind are welcome (including, but not limited to, the following).

* :: Contribution codes
* :: Improving documentation
* :: Writing a curriculum
* :: Refinement of notes
* ...
