# Routing

- [Basic Routing](#basic-routing)
    - [Redirect Routes](#redirect-routes)
    - [View Routes](#view-routes)
- [Route Parameters](#route-parameters)
    - [Required Parameters](#required-parameters)
    - [Optional Parameters](#parameters-optional-parameters)
    - [Regular Expression Constraints](#parameters-regular-expression-constraints)
- [Named Routes](#named-routes)
- [Route Groups](#route-groups)
    - [Middleware](#route-group-middleware)
    - [Namespaces](#route-group-namespaces)
    - [Sub-Domain Routing](#route-group-sub-domain-routing)
    - [Route Prefixes](#route-group-prefixes)
    - [Route Name Prefixes](#route-group-name-prefixes)
- [Route Model Binding](#route-model-binding)
    - [Implicit Binding](#implicit-binding)
    - [Explicit Binding](#explicit-binding)
- [Fallback Routes](#fallback-routes)
- [Rate Limiting](#rate-limiting)
- [Form Method Spoofing](#form-method-spoofing)
- [Accessing The Current Route](#accessing-the-current-route)


<a name="basic-routing"></a>
## Basic Routing

সবচেয়ে সাধারণ লারাভেল রাউট URI হিসাবে ব্যবহার কর হয় এবং একটি Closure, প্রধান করে রাউট ডিফাইন করার জন্য খুব সাধারণ এবং অর্থপূর্ণ মেথড নাম ব্যবহার করা হয়:

    Route::get('foo', function () {
        return 'Hello World';
    });

#### ডিফল্ট রাউট ফাইলগুলি

সমস্ত লারাভেল রাউটস সংজ্ঞায়িত রাউট ফাইল হিসাবে, যা `route` ডিরেক্টরির মধ্যে অবস্থিত। এই ফাইলগুলি স্বয়ংক্রিয়ভাবে ফ্রেমওয়ার্ক দ্বারা লোড হয়। `routes/web.php` ফাইল আপনার ওয়েব ইন্টারফেস জন্য যে রাউটস সংজ্ঞায়িত। এই রাউটস `web` middleware group আরোপিত করা হয়, যা session stat এবং CSRF সুরক্ষা মত বৈশিষ্ট্য প্রদান করে। `routes/api.php` রাউট `api` middleware group এ অন্তর্ভুক্ত।

অধিকাংশ অ্যাপ্লিকেশনের জন্য, আপনি  'route/ web.php'  ফাইলের রুটগুলি সংজ্ঞায়িত করে শুরু করতে পারবেন। `routes/web.php` এ সংজ্ঞায়িত রুটগুলি আপনার ব্রাউজারে নির্ধারিত রাউটের URL টি প্রবেশ করে অ্যাক্সেস করা যেতে পারে। উদাহরণস্বরূপ, আপনি  ব্রাউজারে  `http://your-app.test/user` এ নেভিগেট করার মাধ্যমে নিম্নোক্ত রুটটি ব্যবহার করা যায়:

    Route::get('/user', 'UserController@index');

 `routes/api.php`  ফাইলের রেফারেন্সগুলি রাউট গ্রুপ মধ্যে `RouteServiceProvider` দ্বারা নিথর করা হয়। এই গোষ্ঠীর মধ্যে, `/ api` URI প্রিফিক্স স্বয়ংক্রিয়ভাবে প্রয়োগ করা হয় তাই আপনাকে এটি ফাইলটির প্রতিটি রাউট ম্যানুয়ালি প্রয়োগ করতে হবে না। আপনি  `RouteServiceProvider` class এ  প্রিফিক্স  এবং অন্যান্য route group  বিকল্পগুলি পরিবর্তন করতে পারেন।

#### সচারাচর রাউট মেথড গুলো
রাউটার অনুমতিদেয় আপনার   register রাউটস এ যা যেকোনো HTTP verb এ respond করে:

    Route::get($uri, $callback);
    Route::post($uri, $callback);
    Route::put($uri, $callback);
    Route::patch($uri, $callback);
    Route::delete($uri, $callback);
    Route::options($uri, $callback);

কিছু কিছু সময় রেজিস্টার করা রাউট multiple HTTP verbs এ responds করানোর প্রয়োজন হয়। আপনি `match` মেথড  ব্যবহার করতে পারেন অথবা, আপনি এমন কোনো রাউট রেজিস্টার করতে পারেন যা সমস্ত HTTP verbs  responds   'any' মেথড ব্যবহার করে:

    Route::match(['get', 'post'], '/', function () {
        //
    });

    Route::any('foo', function () {
        //
    });

#### CSRF Protection


যেকোনো HTML  ফর্ম `POST`, `PUT`, or `DELETE` `web`  রাউট এর মধ্যে  ব্যাবহার করা হলে অবশ্যই CSRF token field অন্তর ভুক্ত করতে হবে। অন্যথায়, request বাতিল করা হবে। CSRF protection সম্পর্কে বিস্তারিত জানার জন্য: [CSRF ডকুমেন্টেশন](/docs/{{version}}/csrf):

    <form method="POST" action="/profile">
        @csrf
        ...
    </form>

<a name="redirect-routes"></a>
### Redirect Routes

অন্য URI তে  redirects করার জন্য `Route::redirect` মেথড ব্যবহার করতে পারেন। এটি  সুবিধাজনক shortcut  মেথড প্রধান করে, তাই আপনি পুরো রাউট define করতে পারেনা অথবা controller সহজে redirect করতে পারে:


    Route::redirect('/here', '/there', 301);

<a name="view-routes"></a>
### ভিউ রুটস

যদি view তে return  রাউট এর প্রয়োজন হয় , আপনি  `Route::view` মেথড ব্যবহার করতে পারেন। `redirect` মেথড এর মতো, এই মেথড একটি সহজ শর্টকাট প্রদান করে ,যাতে আপনি একটি পূর্ণ রাউট বা controller  ডিফাইন  না করতে হয়।`view` মেথড একটি URI হিসাবে তার প্রথম argument এবং name দ্বিতীয় argument  হিসাবে  গ্রহণ করে। এছাড়াও, আপনি data  array  তৃতীয় অপশনাল argument হিসাবে view  পাঠাতে পারেন।

    Route::view('/welcome', 'welcome');

    Route::view('/welcome', 'welcome', ['name' => 'Taylor']);

<a name="route-parameters"></a>
## Route Parameters

<a name="required-parameters"></a>
### Required Parameters

অবশ্যই, কখনও কখনও আপনাকে আপনার রাউট মধ্যে URI এর অংশগুলিকে ক্যাপচার করতে হবে। উদাহরণস্বরূপ, আপনাকে URL থেকে একটি ব্যবহারকারীর আইডি ক্যাপচার করতে হতে পারে। আপনি রাউট প্যারামিটার ডিফাইন করে তা করতে পারেন:

    Route::get('user/{id}', function ($id) {
        return 'User '.$id;
    });

আপনি আপনার রাউট দ্বারা প্রয়োজনীয় অনেক রাউট প্যারামিটার সংজ্ঞায়িত করতে পারেন:

    Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
        //
    });

রাউট প্যারামিটার সর্বদা `{}` বন্ধনীগুলির মধ্যে আবদ্ধ থাকে এবং এতে alphabetic অক্ষর থাকা উচিত এবং এতে একটি `-` অক্ষর থাকতে পারে না। `-` অক্ষর ব্যবহার করার পরিবর্তে, একটি আন্ডারস্কোর  (`_`) ব্যবহার করুন। রাউট প্যারামিটারগুলি তাদের অর্ডারের ভিত্তিতে রাউট callbacks / controllers ইনজেক্ট করা হয় - callback / controller আর্গুমেন্টগুলির নামগুলি কোন ব্যাপার না।

<a name="parameters-optional-parameters"></a>
### ঐচ্ছিক প্যারামিটার

কখনত্ত কখনত্ত আপনি একটি রুট প্যারামিটার নির্দিষ্ট করার প্রয়োজন হতে পারে, তবে ঐ রাউট প্যারামিটারের ঐচ্ছিক বিকল্পটি হিসাবে তৈরি করুন। আপনি প্যারামিটার নামের পরে `?` চিহ্নটি স্থাপন করুন। রাউট এর সংশ্লিষ্ট ভেরিয়েবল একটি ডিফল্ট মান দিতে নিশ্চিত করুন:

    Route::get('user/{name?}', function ($name = null) {
        return $name;
    });

    Route::get('user/{name?}', function ($name = 'John') {
        return $name;
    });

<a name="parameters-regular-expression-constraints"></a>
### Regular Expression Constraints

আপনি একটি রাউট  এ `where` মেথড  ব্যবহার করে আপনার রাউট প্যারামিটারগুলির বিন্যাস সীমাবদ্ধ হতে পারে।`where` মেথড প্যারামিটারের name এবং একটি regular expression  ডিফাইন করে যা প্যারামিটারকে  সীমাবদ্ধ করা উচিত:


    Route::get('user/{name}', function ($name) {
        //
    })->where('name', '[A-Za-z]+');

    Route::get('user/{id}', function ($id) {
        //
    })->where('id', '[0-9]+');

    Route::get('user/{id}/{name}', function ($id, $name) {
        //
    })->where(['id' => '[0-9]+', 'name' => '[a-z]+']);

<a name="parameters-global-constraints"></a>
#### Global Constraints


আপনি একটি  regular expression দ্বারা সবসময় সীমাবদ্ধ একটি রাউট প্যারামিটার চান, আপনি `pattern` মেথড ব্যবহার করতে পারেন। আপনার `RouteServiceProvider` এর `boot` মেথড এ   এই patterns ডিফাইন করতে হবে:

    /**
     * Define your route model bindings, pattern filters, etc.
     *
     * @return void
     */
    public function boot()
    {
        Route::pattern('id', '[0-9]+');

        parent::boot();
    }


একবার pattern ডিফাইন করা হয়ে গেলে, এটি যে parameter name মাধ্যমে স্বয়ংক্রিয়ভাবে সব রাউট এ প্রয়োগ হয়:

    Route::get('user/{id}', function ($id) {
        // Only executed if {id} is numeric...
    });

<a name="parameters-encoded-forward-slashes"></a>
#### Encoded Forward Slashes

The Laravel routing component allows all characters except `/`. You must explicitly allow `/` to be part of your placeholder using a `where` condition regular expression:

    Route::get('search/{search}', function ($search) {
        return $search;
    })->where('search', '.*');

> {note} Encoded forward slashes are only supported within the last route segment.

<a name="named-routes"></a>
## Named Routes

Named routes নির্দিষ্ট রাউট জন্য ইউআরএল বা redirects সুবিধাজনক প্রজন্মের অনুমতি দেয়। রাউট ডিফাইন এর সময় `name` মেথড এর মাধ্যমে রাউটকে সহজে নির্দিষ্ট করতে পারেন:

    Route::get('user/profile', function () {
        //
    })->name('profile');

এছাড়াও আপনি controller  জন্য route name উল্লেখ করতে পারেন:

    Route::get('user/profile', 'UserController@showProfile')->name('profile');

#### Generating URLs To Named Routes

একবার একটি রাউট একটি নাম নির্ধারিত হলে, আপনি global `route` ফাংশনের মাধ্যমে URL বা redirects তৈরি করার সময় রাউটটির নাম ব্যবহার করতে পারেন:

    // Generating URLs...
    $url = route('profile');

    // Generating Redirects...
    return redirect()->route('profile');

যদি named route প্যারামিটার ডিফাইন করা হয়, আপনি  `route` ফাংশনে দ্বিতীয় argument হিসাবে পরামিতিগুলি পাস করতে পারেন। প্রদত্ত প্যারামিটারগুলি স্বয়ংক্রিয়ভাবে তাদের সঠিক অবস্থানে URL- এ সন্নিবেশ করা হবে:

    Route::get('user/{id}/profile', function ($id) {
        //
    })->name('profile');

    $url = route('profile', ['id' => 1]);

#### Inspecting The Current Route

আপনি যদি নির্ধারণ করতে চান if the current request was routed to a given named route, you may use the `named` method on a Route instance. For example, you may check the current route name from a route middleware:

    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        if ($request->route()->named('profile')) {
            //
        }

        return $next($request);
    }

<a name="route-groups"></a>
## গ্রুপ রাউট

রাউটে গ্রুপ আপনার রাউটের attributes ভাগাভাগী করতে সহায়তা করে , উদাহরণ সরূপ middleware অথবা namespaces, across a large number of routes without needing to define those attributes on each inividual route. Shared attributes are specified in an array format as the first parameter to the `Route::group` method.

<a name="route-group-middleware"></a>
### Middleware

To assign middleware to all routes within a group, you may use the `middleware` method before defining the group. Middleware are executed in the order they are listed in the array:

    Route::middleware(['first', 'second'])->group(function () {
        Route::get('/', function () {
            // Uses first & second Middleware
        });

        Route::get('user/profile', function () {
            // Uses first & second Middleware
        });
    });

<a name="route-group-namespaces"></a>
### Namespaces

Another common use-case for route groups is assigning the same PHP namespace to a group of controllers using the `namespace` method:

    Route::namespace('Admin')->group(function () {
        // Controllers Within The "App\Http\Controllers\Admin" Namespace
    });

Remember, by default, the `RouteServiceProvider` includes your route files within a namespace group, allowing you to register controller routes without specifying the full `App\Http\Controllers` namespace prefix. So, you only need to specify the portion of the namespace that comes after the base `App\Http\Controllers` namespace.

<a name="route-group-sub-domain-routing"></a>
### Sub-Domain Routing

Route groups may also be used to handle sub-domain routing. Sub-domains may be assigned route parameters just like route URIs, allowing you to capture a portion of the sub-domain for usage in your route or controller. The sub-domain may be specified by calling the `domain` method before defining the group:

    Route::domain('{account}.myapp.com')->group(function () {
        Route::get('user/{id}', function ($account, $id) {
            //
        });
    });

<a name="route-group-prefixes"></a>
### Route Prefixes

The `prefix` method may be used to prefix each route in the group with a given URI. For example, you may want to prefix all route URIs within the group with `admin`:

    Route::prefix('admin')->group(function () {
        Route::get('users', function () {
            // Matches The "/admin/users" URL
        });
    });

<a name="route-group-name-prefixes"></a>
### Route Name Prefixes

The `name` method may be used to prefix each route name in the group with a given string. For example, you may want to prefix all of the grouped route's names with `admin`. The given string is prefixed to the route name exactly as it is specified, so we will be sure to provide the trailing `.` character in the prefix:

    Route::name('admin.')->group(function () {
        Route::get('users', function () {
            // Route assigned name "admin.users"...
        })->name('users');
    });

<a name="route-model-binding"></a>
## Route Model Binding

When injecting a model ID to a route or controller action, you will often query to retrieve the model that corresponds to that ID. Laravel route model binding provides a convenient way to automatically inject the model instances directly into your routes. For example, instead of injecting a user's ID, you can inject the entire `User` model instance that matches the given ID.

<a name="implicit-binding"></a>
### Implicit Binding

Laravel automatically resolves Eloquent models defined in routes or controller actions whose type-hinted variable names match a route segment name. For example:

    Route::get('api/users/{user}', function (App\User $user) {
        return $user->email;
    });

Since the `$user` variable is type-hinted as the `App\User` Eloquent model and the variable name matches the `{user}` URI segment, Laravel will automatically inject the model instance that has an ID matching the corresponding value from the request URI. If a matching model instance is not found in the database, a 404 HTTP response will automatically be generated.

#### Customizing The Key Name

If you would like model binding to use a database column other than `id` when retrieving a given model class, you may override the `getRouteKeyName` method on the Eloquent model:

    /**
     * Get the route key for the model.
     *
     * @return string
     */
    public function getRouteKeyName()
    {
        return 'slug';
    }

<a name="explicit-binding"></a>
### Explicit Binding

To register an explicit binding, use the router's `model` method to specify the class for a given parameter. You should define your explicit model bindings in the `boot` method of the `RouteServiceProvider` class:

    public function boot()
    {
        parent::boot();

        Route::model('user', App\User::class);
    }

Next, define a route that contains a `{user}` parameter:

    Route::get('profile/{user}', function (App\User $user) {
        //
    });

Since we have bound all `{user}` parameters to the `App\User` model, a `User` instance will be injected into the route. So, for example, a request to `profile/1` will inject the `User` instance from the database which has an ID of `1`.

If a matching model instance is not found in the database, a 404 HTTP response will be automatically generated.

#### Customizing The Resolution Logic

If you wish to use your own resolution logic, you may use the `Route::bind` method. The `Closure` you pass to the `bind` method will receive the value of the URI segment and should return the instance of the class that should be injected into the route:

    public function boot()
    {
        parent::boot();

        Route::bind('user', function ($value) {
            return App\User::where('name', $value)->first() ?? abort(404);
        });
    }

<a name="rate-limiting"></a>
## Rate Limiting

Laravel includes a [middleware](/docs/{{version}}/middleware) to rate limit access to routes within your application. To get started, assign the `throttle` middleware to a route or a group of routes. The `throttle` middleware accepts two parameters that determine the maximum number of requests that can be made in a given number of minutes. For example, let's specify that an authenticated user may access the following group of routes 60 times per minute:

    Route::middleware('auth:api', 'throttle:60,1')->group(function () {
        Route::get('/user', function () {
            //
        });
    });

#### Dynamic Rate Limiting

You may specify a dynamic request maximum based on an attribute of the authenticated `User` model. For example, if your `User` model contains a `rate_limit` attribute, you may pass the name of the attribute to the `throttle` middleware so that it is used to calculate the maximum request count:

    Route::middleware('auth:api', 'throttle:rate_limit,1')->group(function () {
        Route::get('/user', function () {
            //
        });
    });

<a name="form-method-spoofing"></a>
## Form Method Spoofing

HTML forms do not support `PUT`, `PATCH` or `DELETE` actions. So, when defining `PUT`, `PATCH` or `DELETE` routes that are called from an HTML form, you will need to add a hidden `_method` field to the form. The value sent with the `_method` field will be used as the HTTP request method:

    <form action="/foo/bar" method="POST">
        <input type="hidden" name="_method" value="PUT">
        <input type="hidden" name="_token" value="{{ csrf_token() }}">
    </form>

You may use the `@method` Blade directive to generate the `_method` input:

    <form action="/foo/bar" method="POST">
        @method('PUT')
        @csrf
    </form>

<a name="accessing-the-current-route"></a>
## Accessing The Current Route

You may use the `current`, `currentRouteName`, and `currentRouteAction` methods on the `Route` facade to access information about the route handling the incoming request:

    $route = Route::current();

    $name = Route::currentRouteName();

    $action = Route::currentRouteAction();

Refer to the API documentation for both the [underlying class of the Route facade](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html) and [Route instance](https://laravel.com/api/{{version}}/Illuminate/Routing/Route.html) to review all accessible methods.
