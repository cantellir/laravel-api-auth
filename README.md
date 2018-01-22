# laravel-api-auth
Pratical step-by-step how to do a REST API in Laravel 5.5 with authentication by email and password using Laravel Passport (OAuth 2.0)

### Prerequisites
* Apache
* PHP
* Composers
* [Laravel new app created](https://github.com/cantellir/laravel-new-app)
* Configured database

### Step 1 - Add Laravel Passport to composer.json
In the project dir, run on terminal
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

### Step 4 - Add HasApiTokens at App\User.php
```
<?php

namespace App;

**use Laravel\Passport\HasApiTokens;**
use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use <b>HasApiTokens,</b> Notifiable;
}
```