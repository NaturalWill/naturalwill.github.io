---
layout: post
title: How To Create Facade On Laravel 5.2
tags: 
  - PHP
  - Laravel
date: 2016/08/24
---


Reference [How To Create Facade On Laravel 5.1](http://www.n0impossible.com/article/how-to-create-facade-on-laravel-51)


Here is step by step to create facade on laravel 5.2

1. Create PHP Class File.
2. Bind that class to Service Provider
3. Register that ServiceProvider to config\app.php as providers
4. Create Class which is this class extends to Illuminate\Support\Facades\Facade
5. Register point 4 to config\app.php as aliases


## Step 1 - Create PHP Class File, for example in App\Classes\UauthHelper.php

	<?php

	namespace App\Classes;

	class UauthHelper {
		public function foo()
		{
			echo "foo";
		}
	}
	
## Step 2 - Bind that class to Service Provider

In case i create a new serviceprovider by execute

	php artisan make:provider 'UauthServiceProvider'

then add

			$this->app->bind('uauth', function () {
				return new \App\Classes\UauthHelper;
			});
			
			
<!-- more -->
			
Like so

	<?php

	namespace App\Providers;

	use Illuminate\Support\ServiceProvider;
	use Illuminate\Support\Facades\App;

	class UauthServiceProvider extends ServiceProvider
	{
		/*
		 * Bootstrap the application services.
		 *
		 * @return void
		 */
		public function boot()
		{
			//
		}

		/*
		 * Register the application services.
		 *
		 * @return void
		 */
		public function register()
		{
			//
			$this->app->bind('uauth', function () {
				return new \App\Classes\UauthHelper;
			});
		}
	}


## Step 3 - Register that ServiceProvider to config\app.php as providers

         /*
         * Application Service Providers...
         */
    	App\Providers\UauthServiceProvider::class,

## Step 4 - Create Class which is this class extends to Illuminate\Support\Facades\Facade

For Example I create this class in App\Facades\Uauth.php

	<?php

	namespace App\Facades;
	use Illuminate\Support\Facades\Facade;


	class Uauth extends Facade{
		
		protected static function getFacadeAccessor()
		{
			return 'uauth';
		}
	}
	
## Step 5 - Register point 4 to config\app.php as aliases

		'Uauth' => App\Facades\Uauth::class
		
## Testing

On App\Http\routes.php create single route

	Route::get('/', function(){
		Uauth::foo();
	});
	
Then check on your browser