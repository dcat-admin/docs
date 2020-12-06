# Custom login

### Rewrite the login page and login logic

Way 1, rewrite the login controller method:

The default login controller uses the class `App\Admin\AuthController`, which can be modified by configuring the parameter `admin.auth.controller`.

```php
<?php

namespace App\Admin\Controllers;

use Dcat\Admin\Controllers\AuthController as BaseAuthController;

class AuthController extends BaseAuthController
{
    // Custom login view template
    protected $view = 'admin.login';
	
	// Rewrite the logic of your landing page
	public function getLogin(Content $content)
    {
        ...
    }

    ...
}

```


Way 2, override routing:

In the routing file `app/Admin/routes.php`, override the routing of the landing page and login logic to implement custom features

```php
Route::group([
    'prefix'        => config('admin.prefix'),
    'namespace'     => Admin::controllerNamespace(),
    'middleware'    => ['web', 'admin'],
], function (Router $router) {

    $router->get('auth/login', 'AuthController@getLogin');
    $router->post('auth/login', 'AuthController@postLogin');
    
});
```

Implement your own login page and login logic in the `getLogin` and `postLogin` methods of your custom router AuthController.


### Rewrite lavel authentication

If you don't use the built-in authentication login logic of `Dcat Admin`, you can customize the login authentication logic in the following way.

The first thing to do is to define a `user provider`, which is used to get the user's identity, e.g. `app/Providers/CustomUserProvider.php`:

```php
<?php

namespace App\Providers;

use Illuminate\Contracts\Auth\Authenticatable;
use Illuminate\Contracts\Auth\UserProvider;

class CustomUserProvider implements UserProvider
{
    public function retrieveById($identifier)
    {}

    public function retrieveByToken($identifier, $token)
    {}

    public function updateRememberToken(Authenticatable $user, $token)
    {}

    public function retrieveByCredentials(array $credentials)
    {
        // Use the username and password in $credentials to get user information, and then return the Illuminate\Contracts\Auth\Authenticatable object.
    }

    public function validateCredentials(Authenticatable $user, array $credentials)
    {
        // Verifies the user with the username and password in $credentials.
    }
}

```

In the methods `retrieveByCredentials` and `validateCredentials`, the incoming `$credentials` is the array of usernames and passwords submitted by the login page, and you can use `$credentials` to implement your own login logic

Interface `Illuminate\Contracts\Auth\Authenticatable` definitions are as follows：
```php
<?php

namespace Illuminate\Contracts\Auth;

interface Authenticatable {

    public function getAuthIdentifierName();
    public function getAuthIdentifier();
    public function getAuthPassword();
    public function getRememberToken();
    public function setRememberToken($value);
    public function getRememberTokenName();

}
```

Explanation of each method of the above interface reference [adding-custom-user-providers](https://laravel.com/docs/5.5/authentication#adding-custom-user-providers)

Once you have defined `User provider`, open `app/Providers/AuthServiceProvider.php` and register it:

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Auth;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

class AuthServiceProvider extends ServiceProvider
{
    /**
     * Register any application authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Auth::provider('custom', function ($app, array $config) {
            
            // Return an instance of Illuminate\Contracts\Auth\UserProvider...
            return new CustomUserProvider();
        });
    }
}
```

Finally, change the configuration, open `config/admin.php`, find the `auth` section to change:

```php
    'auth' => [
        'guards' => [
            'admin' => [
                'driver' => 'session',
                'provider' => 'admin',
            ]
        ],

        // modify the following
        'providers' => [
            'admin' => [
                'driver' => 'custom',
            ]
        ],
    ],
```
This completes the logic of custom login authentication. Custom login is a complex part of laravel and requires patience to debug step by step.

