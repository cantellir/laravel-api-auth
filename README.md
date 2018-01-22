# laravel-api-auth
Pratical step-by-step how to do a RESTful API in Laravel 5.5 with authentication by email and password using Laravel Passport (OAuth 2.0)

### Prerequisites
* Apache
* PHP
* Composers
* [Laravel new app created](https://github.com/cantellir/laravel-new-app)

### Initial notes
The project in this repo contains all the steps finalized

### Step 1 - Add Laravel Passport to composer.json
In the project dir run
```
composer require laravel/passport
```

### Step 2 - Run migrations
```
php artisan migrate
```

### Step 3 - Install Laravel Passport
```
php artisan passport:install
```

### Step 4 - Add HasApiTokens at app/User.php
```php
<?php

namespace App;

use Laravel\Passport\HasApiTokens;
use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use HasApiTokens, Notifiable;

    [...]    
}
```

### Step 5 - Add Passport Routes to auth provider
In the "app/Providers/AuthServiceProvider.php" add passport routes to boot method
```php
<?php

namespace App\Providers;

use Laravel\Passport\Passport;
use Illuminate\Support\Facades\Gate;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

class AuthServiceProvider extends ServiceProvider
{
    protected $policies = [
        'App\Model' => 'App\Policies\ModelPolicy',
    ];


    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();
    }
}
```

### Step 6 - Alter auth api driver to "passport"
In the "routes/api.php" adjust the driver for api auth
```
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],

    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],
],
```

### Step 7 - Add endpoints for auth control
In the "routes/api.php" add routes to login, register and logout
```php
<?php

use Illuminate\Http\Request;

Route::post('login', 'Auth\LoginController@login');
Route::post('register', 'Auth\RegisterController@register');

//protected routes
Route::group(['middleware' => 'auth:api'], function() {
    Route::get('logout', 'Auth\LoginController@logout');
});

```

### Step 8 - Create login and logout methods
In the Login Controller (Controllers/Auth/LoginController.php) add login and logout methods
```php
<?php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use Illuminate\Foundation\Auth\AuthenticatesUsers;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use App\User;

class LoginController extends Controller
{
    use AuthenticatesUsers;

    protected $redirectTo = '/home';
    public function __construct()
    {
        $this->middleware('guest')->except('logout');
    }

    public function login(Request $request)
    {
        $this->validateLogin($request);

        if ($this->attemptLogin($request)) {
            $user = Auth::user();
            $success['token'] = $user->createToken('MyApp')->accessToken;
            $success['user'] = $user;
            return response()->json($success, 200);
        }

        return $this->sendFailedLoginResponse($request);
    }

    public function logout()
    {
        $user = Auth::user();
        $user->token()->revoke();
        $user->token()->delete();
    }
}

```

### Step 9 - Create register method
In the Register Controller (Controllers/Auth/RegisterController.php) add register method
```php
<?php

namespace App\Http\Controllers\Auth;

use App\User;
use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\Validator;
use Illuminate\Foundation\Auth\RegistersUsers;
use Illuminate\Http\Request;

class RegisterController extends Controller
{
    use RegistersUsers;

    protected $redirectTo = '/home';
    public function __construct()
    {
        $this->middleware('guest');
    }

    protected function validator(array $data)
    {
        return Validator::make($data, [
            'name' => 'required|string|max:255',
            'email' => 'required|string|email|max:255|unique:users',
            'password' => 'required|string|min:6|confirmed',
        ]);
    }

    public function register(Request $request)
    {
        $this->validator($request->all())->validate();

        $user = User::create([
            'name' => $request['name'],
            'email' => $request['email'],
            'password' => bcrypt($request['password']),
        ]);

        $this->guard()->login($user);
        $success['token'] = $user->createToken('nfce_client')->accessToken;
        $success['user'] = $user;        
        return response()->json($success, 201);
    }
}

```

### Step 10 - Test endpoints
Register
```
curl -X POST -H 'Accept: application/json' -d 'name=user&email=user@test.com&password=passuser&password_confirmation=passuser' http://localhost/laravel-api-auth/api/register

```

Login
```
curl -X POST -H 'Accept: application/json' -d 'email=user@test.com&password=passuser' http://localhost/laravel-api-auth/api/login
```

Logout
```
curl -H 'Accept: application/json' -H 'Authorization: Bearer token_generated_on_register_or_login' http://localhost/laravel-api-auth/api/logout
```

## References
* [Laravel docs](https://laravel.com/docs/5.5) - Laravel Documentation
* [Laravel Passport Post](https://laravelcode.com/post/laravel-passport-create-rest-api-with-authentication) - Create REST API with authentication
* [Laravel API Tutorial](https://www.toptal.com/laravel/restful-laravel-api-tutorial) - How to Build and Test a RESTful API