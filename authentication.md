# Authentication

- [ভূমিকা](#introduction)
    - [ডাটাবেস সম্পর্কে কথা](#introduction-database-considerations)
- [অথিন্তিকেশনের শুরুর কথা](#authentication-quickstart)
    - [রাউটিং](#included-routing)
    - [ভিউ গুলো](#included-views)
    - [বিশুদ্ধতা প্রমাণ করা](#included-authenticating)
    - [লগইন কৃত ব্যবহারকারীর তথ্য বের করা](#retrieving-the-authenticated-user)
    - [রাউটগুলো কে সুরক্ষা দেওয়া](#protecting-routes)
    - [লগইন থ্রটলিং সম্পর্কে](#login-throttling)
- [ম্যানুয়ালি ইউজারকে লগইন করানো](#authenticating-users)
    - [ইউজারকে মনে রাখার সুবিধা](#remembering-users)
    - [অন্যন্য ভাবে অথিন্টিকেশোনের উপায়](#other-authentication-methods)
- [HTTP বেসিক অথিক্টিকেশন](#http-basic-authentication)
    - [Stateless HTTP বেসিক অথিক্টিকেশন](#stateless-http-basic-authentication)
- [লগাউট সম্পর্কে](#logging-out)
    - [অন্যন্য ডিভাইসে সেশন বাতিল করণ](#invalidating-sessions-on-other-devices)
- [সামাজিক অথিন্টিকেশন](https://github.com/laravel/socialite)
- [নিজেদের গার্ড তৈরি করা](#adding-custom-guards)
    - [ক্লোজার রিকোয়েস্ট গার্ড](#closure-request-guards)
- [নিজেদের ব্যহারকারির প্রভাইডার](#adding-custom-user-providers)
    - [ব্যহারকারির প্রভাইডার কন্ট্রাক](#the-user-provider-contract)
    - [অথিক্টিকেটেবল কন্ট্রাক](#the-authenticatable-contract)
- [ইভেন্ট গুলো](#events)

<a name="introduction"></a>
## ভূমিকা

> {tip} **অতি দ্রুত শুরু করতে চান?** শুধু রান করুন `php artisan make:auth` এবং `php artisan migrate` নতুন তৈরি কৃত লারাভেল আপ্লিকেশনে. তারপর, ব্রাউজারে গিয়ে খুলুন `http://your-app.test/register` অথবা যেকোনো ইউ আর এল এ যেটা আপনার এপ্লিকেশনে আসাইন করা আছে। এই দুটো কমান্ডই যথেষ্ট আপনার এপ্লিকেশনের অথিকন্টিকেশন তৈরি করার জন্য! 

লারাভেল অথিন্টিকেশন খুবই সহজ করেছে। আসলে, প্রায় সব কিছুই কনফিগার করা থাকে আগে থেকেই। অথিক্টিকেশন এর কনফিগারেশন ফাইল এই লোকেশন এ থাকে `config/auth.php`, যেটাতে সুন্দর করে অথিকন্টিকেশন এর তথ্য দেওয়া থাকে ।

এর মুল অংশে, লারাভেল অথিন্টিকেশন সুবিধা তৈরি করে "গার্ড" এবং "প্রভাইডার" এর উপর ভিত্তি করে। গার্ড নির্ধারণ করে কিভাবে ব্যবহারকারী অথিন্টিকেইট হবে প্রতিটি রিকুয়েস্টে। উধাহরনস্বরূপঃ লারাভেল এসেছে `session` গার্ডের সাথে যেটা, স্টেইটকে মেইটেইন করে সেশন এবং কুকির সাহায্যে। 

প্রভাইডার নির্ধারণ করে কিভাবে আপনার নির্ধারিত স্টোরেজ থেকে ডাটা আহরন করবে। লারাভেল এলকয়েন্ট এবং কোয়েরি বিল্ডারের সাথে এসেছে ইউজারের ডাটা আহরন করার জন্য। যাইহোক আপনি চাইলে আপনার তৈরি কৃত প্রভাইডার ও ব্যবহার করতে পারবেন যদি আপনার সেটি দরকার হয় । 

চিন্তার কোন কারন নেই যদি এই কথা গুলো আপনার মাথার উপর দিয়ে যায় । অনেক আপ্লিকেশন আছে যেখানে কখনই পূর্বনির্ধারিত আপ্লিকেশন কনফিগারেশন পরিবর্তন করার প্রয়োজন ই পরে না। 

<a name="introduction-database-considerations"></a>
### ডাটাবেস সম্পর্কে কথা

পুর নির্ধারিত ভাবে, লারাভেল দিয়ে থেকে একটি `App\User` [ইলুকয়েন্ট মডেল](/docs/{{version}}/eloquent) আপনার এপ্লিকেশনের `app` ফোল্ডারে। এই মডেল ব্যবহ্রত হতে পারে পূর্বনির্ধারিত ইলুকয়েন্ট অথিন্টিকেশন ড্রাইভার দ্বারা। যদি আপনার এপ্লিকেশন ইলুকয়েন্ট ব্যবহার না করে তবে আপনি ড`database` ড্রাইভার ব্যবহার করতে পারেন যেটা লারাভেলের কুয়েরই বিল্ডার ব্যবহার করে।

যখন আপনি ডাটাবেইস এর স্কিমা তৈরি করছেন  `App\User` মডেল এর জন্য, এই বিষয় টি নিশ্চিত করবেন যেন সেখান কার password কলাম যেন সর্বনিম্ন ৬০ অক্ষর নিতে পারে। পূর্বনির্ধারিত স্ট্রিং এর কলাম ২৫৫ টি অক্ষর নিতে পারে এই ব্যবহার করাই ভালো ।

এছাড়াও, আপনি যাচাই করবেন আপনার `users` (অথবা এর মত) টেবিলে যেন নালবল স্ট্রিং  `remember_token` কলাম থাকে যেটি ১০০ অক্ষর পর্যন্ত ধারণ করতে পারে। এই কলাম ব্যবহ্রত হবে টকেন জমা রাখার জন্য যখন ইউজার "remember me" অপশন সিলেক্ট করবে তার আপ্লিকেশনে লগইন করার জন্য ।

<a name="authentication-quickstart"></a>
## অথিন্তিকেশনের শুরুর কথা

লারাভেল এসেছে অনেক গুলো আগে থেকেই তৈরি করা অথেনটিকেশন কন্ট্রোলার নিয়ে, যেগুলো রাখা আছে `App\Http\Controllers\Auth` নেইম স্প্রেইস এর আন্ডারে। `RegisterController` পরিচালনা করে নতুন ইউজার রেজিস্ট্রেশন করতে,  `LoginController` পরিচালনা করে অথেনটিকেশন করতে, `ForgotPasswordController` পরিচালনা করে ইমেইল এ পাসওয়ার্ড রিসেট লিঙ্ক পাঠাতে, এবং `ResetPasswordController` এ থাকে পাসওয়ার্ড রিসেট করার লিজিক। এখানকার প্রতিটি কন্ট্রোলার ব্যবহার করে একটি ট্রেইট যেখানে তাদের প্রয়োজনীয় মেথড গুলো আছে । অনেক আপ্লিকেশনের জন্য আপনাকে এইসব কন্ট্রোলার কে পরিবর্তন করার প্রয়োজনই পরবে না।

<a name="included-routing"></a>
### রাউটিং

লারাভেল আপনাকে খুবই দ্রত এবং সহজ পদ্ধতি তে অথিন্টিকেশোনের জন্য দরকারি সকল রাউট এবং ভিউ তৈরি করার কমান্ড দিচ্ছেঃ

    php artisan make:auth

এই কমান্ড ব্যবহার করতে হবে একেবারে নতুন লারাভেল আপ্লিকেশনে এবং এরপরে একটি লেয়াউট ভিউ, রেজিস্ট্রেশন ভিউ, এবং লগইন ভিউ ইন্সটল হয়ে যাবে, পাশাপাশি সকল অথেনটিকেশন রাউটও। একটি `HomeController` ও তৈরি হবে লগইন করার পরে রিকোয়েস্ট হ্যন্ড্যাল করে আপ্লিকেশনের ড্যাসবরড এ নিয়ে যাওয়ার জন্য।

<a name="included-views"></a>
### ভিউ গুলো

পূর্ববর্তী সেকশনে বলা, `php artisan make:auth` কমান্ড আপনাকে তৈরি করে দিবে সকল ভিউ যেগুলো আপনার অথিন্টিকেশনের জন্য লাগবে এবং এগুলো সব রেখে দিবে `resources/views/auth` ফোল্ডারে।

`make:auth` এই কমান্ডটি এছাড়াও তৈরি করে `resources/views/layouts` ফোল্ডারে আপনার এপ্লিকেশনের জন্য বেইজ লেয়াউট ফাইল। সব ভিউ গুলো ব্যবহার করে Bootstrap CSS framework, কিন্তু আপনি এগুলোকে আপনার ইচ্ছামত পরিবর্তন করে নিতে পারবেন আপনি যেভাবে চান।

<a name="included-authenticating"></a>
### অথিন্টিকেটিং 

Now that you have routes and views setup for the included authentication controllers, you are ready to register and authenticate new users for your application! You may access your application in a browser since the authentication controllers already contain the logic (via their traits) to authenticate existing users and store new users in the database.

#### Path Customization

When a user is successfully authenticated, they will be redirected to the `/home` URI. You can customize the post-authentication redirect location by defining a `redirectTo` property on the `LoginController`, `RegisterController`, `ResetPasswordController`, and `VerificationController`:

    protected $redirectTo = '/';

Next, you should modify the `RedirectIfAuthenticated` middleware's `handle` method to use your new URI when redirecting the user.

If the redirect path needs custom generation logic you may define a `redirectTo` method instead of a `redirectTo` property:

    protected function redirectTo()
    {
        return '/path';
    }

> {tip} The `redirectTo` method will take precedence over the `redirectTo` attribute.

#### Username Customization

By default, Laravel uses the `email` field for authentication. If you would like to customize this, you may define a `username` method on your `LoginController`:

    public function username()
    {
        return 'username';
    }

#### Guard Customization

You may also customize the "guard" that is used to authenticate and register users. To get started, define a `guard` method on your `LoginController`, `RegisterController`, and `ResetPasswordController`. The method should return a guard instance:

    use Illuminate\Support\Facades\Auth;

    protected function guard()
    {
        return Auth::guard('guard-name');
    }

#### Validation / Storage Customization

To modify the form fields that are required when a new user registers with your application, or to customize how new users are stored into your database, you may modify the `RegisterController` class. This class is responsible for validating and creating new users of your application.

The `validator` method of the `RegisterController` contains the validation rules for new users of the application. You are free to modify this method as you wish.

The `create` method of the `RegisterController` is responsible for creating new `App\User` records in your database using the [Eloquent ORM](/docs/{{version}}/eloquent). You are free to modify this method according to the needs of your database.

<a name="retrieving-the-authenticated-user"></a>
### Retrieving The Authenticated User

You may access the authenticated user via the `Auth` facade:

    use Illuminate\Support\Facades\Auth;

    // Get the currently authenticated user...
    $user = Auth::user();

    // Get the currently authenticated user's ID...
    $id = Auth::id();

Alternatively, once a user is authenticated, you may access the authenticated user via an `Illuminate\Http\Request` instance. Remember, type-hinted classes will automatically be injected into your controller methods:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class ProfileController extends Controller
    {
        /**
         * Update the user's profile.
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            // $request->user() returns an instance of the authenticated user...
        }
    }

#### Determining If The Current User Is Authenticated

To determine if the user is already logged into your application, you may use the `check` method on the `Auth` facade, which will return `true` if the user is authenticated:

    use Illuminate\Support\Facades\Auth;

    if (Auth::check()) {
        // The user is logged in...
    }

> {tip} Even though it is possible to determine if a user is authenticated using the `check` method, you will typically use a middleware to verify that the user is authenticated before allowing the user access to certain routes / controllers. To learn more about this, check out the documentation on [protecting routes](/docs/{{version}}/authentication#protecting-routes).

<a name="protecting-routes"></a>
### Protecting Routes

[Route middleware](/docs/{{version}}/middleware) can be used to only allow authenticated users to access a given route. Laravel ships with an `auth` middleware, which is defined at `Illuminate\Auth\Middleware\Authenticate`. Since this middleware is already registered in your HTTP kernel, all you need to do is attach the middleware to a route definition:

    Route::get('profile', function () {
        // Only authenticated users may enter...
    })->middleware('auth');

Of course, if you are using [controllers](/docs/{{version}}/controllers), you may call the `middleware` method from the controller's constructor instead of attaching it in the route definition directly:

    public function __construct()
    {
        $this->middleware('auth');
    }

#### Redirecting Unauthenticated Users

When the `auth` middleware detects an unauthorized user, it will redirect the user to the `login` [named route](/docs/{{version}}/routing#named-routes). You may modify this behavior by updating the `redirectTo` function in your `app/Http/Middleware/Authenticate.php` file:

    /**
     * Get the path the user should be redirected to.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return string
     */
    protected function redirectTo($request)
    {
        return route('login');
    }

#### Specifying A Guard

When attaching the `auth` middleware to a route, you may also specify which guard should be used to authenticate the user. The guard specified should correspond to one of the keys in the `guards` array of your `auth.php` configuration file:

    public function __construct()
    {
        $this->middleware('auth:api');
    }

<a name="login-throttling"></a>
### Login Throttling

If you are using Laravel's built-in `LoginController` class, the `Illuminate\Foundation\Auth\ThrottlesLogins` trait will already be included in your controller. By default, the user will not be able to login for one minute if they fail to provide the correct credentials after several attempts. The throttling is unique to the user's username / e-mail address and their IP address.

<a name="authenticating-users"></a>
## Manually Authenticating Users

Of course, you are not required to use the authentication controllers included with Laravel. If you choose to remove these controllers, you will need to manage user authentication using the Laravel authentication classes directly. Don't worry, it's a cinch!

We will access Laravel's authentication services via the `Auth` [facade](/docs/{{version}}/facades), so we'll need to make sure to import the `Auth` facade at the top of the class. Next, let's check out the `attempt` method:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Auth;

    class LoginController extends Controller
    {
        /**
         * Handle an authentication attempt.
         *
         * @param  \Illuminate\Http\Request $request
         *
         * @return Response
         */
        public function authenticate(Request $request)
        {
            $credentials = $request->only('email', 'password');

            if (Auth::attempt($credentials)) {
                // Authentication passed...
                return redirect()->intended('dashboard');
            }
        }
    }

The `attempt` method accepts an array of key / value pairs as its first argument. The values in the array will be used to find the user in your database table. So, in the example above, the user will be retrieved by the value of the `email` column. If the user is found, the hashed password stored in the database will be compared with the `password` value passed to the method via the array. You should not hash the password specified as the `password` value, since the framework will automatically hash the value before comparing it to the hashed password in the database. If the two hashed passwords match an authenticated session will be started for the user.

The `attempt` method will return `true` if authentication was successful. Otherwise, `false` will be returned.

The `intended` method on the redirector will redirect the user to the URL they were attempting to access before being intercepted by the authentication middleware. A fallback URI may be given to this method in case the intended destination is not available.

#### Specifying Additional Conditions

If you wish, you may also add extra conditions to the authentication query in addition to the user's e-mail and password. For example, we may verify that user is marked as "active":

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
        // The user is active, not suspended, and exists.
    }

> {note} In these examples, `email` is not a required option, it is merely used as an example. You should use whatever column name corresponds to a "username" in your database.

#### Accessing Specific Guard Instances

You may specify which guard instance you would like to utilize using the `guard` method on the `Auth` facade. This allows you to manage authentication for separate parts of your application using entirely separate authenticatable models or user tables.

The guard name passed to the `guard` method should correspond to one of the guards configured in your `auth.php` configuration file:

    if (Auth::guard('admin')->attempt($credentials)) {
        //
    }

#### Logging Out

To log users out of your application, you may use the `logout` method on the `Auth` facade. This will clear the authentication information in the user's session:

    Auth::logout();

<a name="remembering-users"></a>
### Remembering Users

If you would like to provide "remember me" functionality in your application, you may pass a boolean value as the second argument to the `attempt` method, which will keep the user authenticated indefinitely, or until they manually logout. Of course, your `users` table must include the string `remember_token` column, which will be used to store the "remember me" token.

    if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
        // The user is being remembered...
    }

> {tip} If you are using the built-in `LoginController` that is shipped with Laravel, the proper logic to "remember" users is already implemented by the traits used by the controller.

If you are "remembering" users, you may use the `viaRemember` method to determine if the user was authenticated using the "remember me" cookie:

    if (Auth::viaRemember()) {
        //
    }

<a name="other-authentication-methods"></a>
### Other Authentication Methods

#### Authenticate A User Instance

If you need to log an existing user instance into your application, you may call the `login` method with the user instance. The given object must be an implementation of the `Illuminate\Contracts\Auth\Authenticatable` [contract](/docs/{{version}}/contracts). Of course, the `App\User` model included with Laravel already implements this interface:

    Auth::login($user);

    // Login and "remember" the given user...
    Auth::login($user, true);

Of course, you may specify the guard instance you would like to use:

    Auth::guard('admin')->login($user);

#### Authenticate A User By ID

To log a user into the application by their ID, you may use the `loginUsingId` method. This method accepts the primary key of the user you wish to authenticate:

    Auth::loginUsingId(1);

    // Login and "remember" the given user...
    Auth::loginUsingId(1, true);

#### Authenticate A User Once

You may use the `once` method to log a user into the application for a single request. No sessions or cookies will be utilized, which means this method may be helpful when building a stateless API:

    if (Auth::once($credentials)) {
        //
    }

<a name="http-basic-authentication"></a>
## HTTP Basic Authentication

[HTTP Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication) provides a quick way to authenticate users of your application without setting up a dedicated "login" page. To get started, attach the `auth.basic` [middleware](/docs/{{version}}/middleware) to your route. The `auth.basic` middleware is included with the Laravel framework, so you do not need to define it:

    Route::get('profile', function () {
        // Only authenticated users may enter...
    })->middleware('auth.basic');

Once the middleware has been attached to the route, you will automatically be prompted for credentials when accessing the route in your browser. By default, the `auth.basic` middleware will use the `email` column on the user record as the "username".

#### A Note On FastCGI

If you are using PHP FastCGI, HTTP Basic authentication may not work correctly out of the box. The following lines should be added to your `.htaccess` file:

    RewriteCond %{HTTP:Authorization} ^(.+)$
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="stateless-http-basic-authentication"></a>
### Stateless HTTP Basic Authentication

You may also use HTTP Basic Authentication without setting a user identifier cookie in the session, which is particularly useful for API authentication. To do so, [define a middleware](/docs/{{version}}/middleware) that calls the `onceBasic` method. If no response is returned by the `onceBasic` method, the request may be passed further into the application:

    <?php

    namespace App\Http\Middleware;

    use Illuminate\Support\Facades\Auth;

    class AuthenticateOnceWithBasicAuth
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, $next)
        {
            return Auth::onceBasic() ?: $next($request);
        }

    }

Next, [register the route middleware](/docs/{{version}}/middleware#registering-middleware) and attach it to a route:

    Route::get('api/user', function () {
        // Only authenticated users may enter...
    })->middleware('auth.basic.once');

<a name="logging-out"></a>
## Logging Out

To manually log users out of your application, you may use the `logout` method on the `Auth` facade. This will clear the authentication information in the user's session:

    use Illuminate\Support\Facades\Auth;

    Auth::logout();

<a name="invalidating-sessions-on-other-devices"></a>
### Invalidating Sessions On Other Devices

Laravel also provides a mechanism for invalidating and "logging out" a user's sessions that are active on other devices without invalidating the session on their current device. Before getting started, you should make sure that the `Illuminate\Session\Middleware\AuthenticateSession` middleware is present and un-commented in your `app/Http/Kernel.php` class' `web` middleware group:

    'web' => [
        // ...
        \Illuminate\Session\Middleware\AuthenticateSession::class,
        // ...
    ],

Then, you may use the `logoutOtherDevices` method on the `Auth` facade. This method requires the user to provide their current password, which your application should accept through an input form:

    use Illuminate\Support\Facades\Auth;

    Auth::logoutOtherDevices($password);

> {note} When the `logoutOtherDevices` method is invoked, the user's other sessions will be invalidated entirely, meaning they will be "logged out" of all guards they were previously authenticated by.

<a name="adding-custom-guards"></a>
## Adding Custom Guards

You may define your own authentication guards using the `extend` method on the `Auth` facade. You should place this call to `extend` within a [service provider](/docs/{{version}}/providers). Since Laravel already ships with an `AuthServiceProvider`, we can place the code in that provider:

    <?php

    namespace App\Providers;

    use App\Services\Auth\JwtGuard;
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

            Auth::extend('jwt', function ($app, $name, array $config) {
                // Return an instance of Illuminate\Contracts\Auth\Guard...

                return new JwtGuard(Auth::createUserProvider($config['provider']));
            });
        }
    }

As you can see in the example above, the callback passed to the `extend` method should return an implementation of `Illuminate\Contracts\Auth\Guard`. This interface contains a few methods you will need to implement to define a custom guard. Once your custom guard has been defined, you may use this guard in the `guards` configuration of your `auth.php` configuration file:

    'guards' => [
        'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
        ],
    ],

<a name="closure-request-guards"></a>
### Closure Request Guards

The simplest way to implement a custom, HTTP request based authentication system is by using the `Auth::viaRequest` method. This method allows you to quickly define your authentication process using a single Closure.

To get started, call the `Auth::viaRequest` method within the `boot` method of your `AuthServiceProvider`. The `viaRequest` method accepts a guard name as its first argument. This name can be any string that describes your custom guard. The second argument passed to the method should be a Closure that receives the incoming HTTP request and returns a user instance or, if authentication fails, `null`:

    use App\User;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Auth;

    /**
     * Register any application authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Auth::viaRequest('custom-token', function ($request) {
            return User::where('token', $request->token)->first();
        });
    }

Once your custom guard has been defined, you may use this guard in the `guards` configuration of your `auth.php` configuration file:

    'guards' => [
        'api' => [
            'driver' => 'custom-token',
        ],
    ],

<a name="adding-custom-user-providers"></a>
## Adding Custom User Providers

If you are not using a traditional relational database to store your users, you will need to extend Laravel with your own authentication user provider. We will use the `provider` method on the `Auth` facade to define a custom user provider:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Auth;
    use App\Extensions\RiakUserProvider;
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

            Auth::provider('riak', function ($app, array $config) {
                // Return an instance of Illuminate\Contracts\Auth\UserProvider...

                return new RiakUserProvider($app->make('riak.connection'));
            });
        }
    }

After you have registered the provider using the `provider` method, you may switch to the new user provider in your `auth.php` configuration file. First, define a `provider` that uses your new driver:

    'providers' => [
        'users' => [
            'driver' => 'riak',
        ],
    ],

Finally, you may use this provider in your `guards` configuration:

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
    ],

<a name="the-user-provider-contract"></a>
### The User Provider Contract

The `Illuminate\Contracts\Auth\UserProvider` implementations are only responsible for fetching a `Illuminate\Contracts\Auth\Authenticatable` implementation out of a persistent storage system, such as MySQL, Riak, etc. These two interfaces allow the Laravel authentication mechanisms to continue functioning regardless of how the user data is stored or what type of class is used to represent it.

Let's take a look at the `Illuminate\Contracts\Auth\UserProvider` contract:

    <?php

    namespace Illuminate\Contracts\Auth;

    interface UserProvider {

        public function retrieveById($identifier);
        public function retrieveByToken($identifier, $token);
        public function updateRememberToken(Authenticatable $user, $token);
        public function retrieveByCredentials(array $credentials);
        public function validateCredentials(Authenticatable $user, array $credentials);

    }

The `retrieveById` function typically receives a key representing the user, such as an auto-incrementing ID from a MySQL database. The `Authenticatable` implementation matching the ID should be retrieved and returned by the method.

The `retrieveByToken` function retrieves a user by their unique `$identifier` and "remember me" `$token`, stored in a field `remember_token`. As with the previous method, the `Authenticatable` implementation should be returned.

The `updateRememberToken` method updates the `$user` field `remember_token` with the new `$token`. A fresh token is assigned on a successful "remember me" login attempt or when the user is logging out.

The `retrieveByCredentials` method receives the array of credentials passed to the `Auth::attempt` method when attempting to sign into an application. The method should then "query" the underlying persistent storage for the user matching those credentials. Typically, this method will run a query with a "where" condition on `$credentials['username']`. The method should then return an implementation of `Authenticatable`. **This method should not attempt to do any password validation or authentication.**

The `validateCredentials` method should compare the given `$user` with the `$credentials` to authenticate the user. For example, this method should probably use `Hash::check` to compare the value of `$user->getAuthPassword()` to the value of `$credentials['password']`. This method should return `true` or `false` indicating on whether the password is valid.

<a name="the-authenticatable-contract"></a>
### The Authenticatable Contract

Now that we have explored each of the methods on the `UserProvider`, let's take a look at the `Authenticatable` contract. Remember, the provider should return implementations of this interface from the `retrieveById`, `retrieveByToken`, and `retrieveByCredentials` methods:

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

This interface is simple. The `getAuthIdentifierName` method should return the name of the "primary key" field of the user and the `getAuthIdentifier` method should return the "primary key" of the user. In a MySQL back-end, again, this would be the auto-incrementing primary key. The `getAuthPassword` should return the user's hashed password. This interface allows the authentication system to work with any User class, regardless of what ORM or storage abstraction layer you are using. By default, Laravel includes a `User` class in the `app` directory which implements this interface, so you may consult this class for an implementation example.

<a name="events"></a>
## Events

Laravel raises a variety of [events](/docs/{{version}}/events) during the authentication process. You may attach listeners to these events in your `EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Auth\Events\Registered' => [
            'App\Listeners\LogRegisteredUser',
        ],

        'Illuminate\Auth\Events\Attempting' => [
            'App\Listeners\LogAuthenticationAttempt',
        ],

        'Illuminate\Auth\Events\Authenticated' => [
            'App\Listeners\LogAuthenticated',
        ],

        'Illuminate\Auth\Events\Login' => [
            'App\Listeners\LogSuccessfulLogin',
        ],

        'Illuminate\Auth\Events\Failed' => [
            'App\Listeners\LogFailedLogin',
        ],

        'Illuminate\Auth\Events\Logout' => [
            'App\Listeners\LogSuccessfulLogout',
        ],

        'Illuminate\Auth\Events\Lockout' => [
            'App\Listeners\LogLockout',
        ],

        'Illuminate\Auth\Events\PasswordReset' => [
            'App\Listeners\LogPasswordReset',
        ],
    ];
