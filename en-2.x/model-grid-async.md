# Asynchronous rendering of tables

When the table on the page displays a particularly large amount of data (many columns and rows) and loads more components, there may be a lagging phenomenon, which can be effectively mitigated by using the table asynchronous rendering feature.


```php
// Enable asynchronous rendering of tables
$grid->async();

// Disable
$grid->async(false);

// Determine if the request is an asynchronous rendering request
if ($grid->isAsyncRequest()) {
    ...
}
```

> Note that there is no need to enable this feature if the page is not significantly stuttering. And if there are multiple data tables on the page, then this feature is also not available.

When this feature is enabled, the table **toolbar** (`toolbar`) and below will be rendered asynchronously, in other words, the **toolbar** (`toolbar`) and above will only be refreshed once! You need to pay attention to any special effects during the actual coding process.
