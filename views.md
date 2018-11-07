# ভিউস

- [ভিউস তৈরি](#creating-views)
- [যদি একটি ভিউ থাকে তা নির্ধারণ](#passing-data-to-views)
    - [Sharing Data With All Views](#sharing-data-with-all-views)
- [View Composers](#view-composers)

<a name="creating-views"></a>
## ভিউস তৈরি

> {tip} ব্লাড টেমপ্লেট লিখতে আরো তথ্যের জন্য খুঁজছেন? শুরু করতে সম্পূর্ণ [ব্লাডে ডকুমেন্টেশন](/docs/{{version}}/blade) দেখুন ।

ভিউস(Views) এর মধ্যে আপনার অ্যাপ্লিকেশন এর HTML ফাইল থাকেে এবং  আপনার উপস্থাপনা যুক্তি থেকে আপনার কন্ট্রোলার / অ্যাপ্লিকেশন যুক্তিটি আলাদা করে। ভিউস ফাইল গুলো `resources/views` ডিরেক্টরিতে থাকে।  নিচে একটা খুব সাধারণ উধাহরন দেয়া হল:


    <!-- View stored in resources/views/greeting.blade.php -->

    <html>
        <body>
            <h1>Hello, {{ $name }}</h1>
        </body>
    </html>

যেহেতু এই ভিউ `resources/views/greeting.blade.php` এ সংরক্ষিত আছে, গ্লোবাল `ভিউ` হেলপার(helper) ব্যবহার করে আমরা এটি রিটার্ন(return) করতে পারি:

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

আপনি দেখতে পারেন, `ভিউ` হেলপারা প্রেরিত প্রথম আর্গুমেন্ট টি `resources/views` ডিরেক্টরিতে  ভিউ  ফাইলটির নামের সাথে সম্পর্কিত। দ্বিতীয় আর্গুমেন্ট তথ্য একটি অ্যারের যা ভিউ উপলব্ধ করা উচিত। এই ক্ষেত্রে, আমরা `name` ভ্যারিয়েবল পাস করছি, যা ভিউ ব্যবহার করে প্রদর্শিত হয় [ব্লাডে সিনটেক্স](/docs/{{version}}/blade)। অবশ্যই, ভিউস  উপ-ডিরেক্টরি(sub-directories) `resources/views` এর মধ্যে নেস্টেড ভাবে থাকতে পারে। "ডট" নোটেশন নেস্টেড ভিউস উল্লেখ করতে ব্যবহার করা যেতে পারে।
উদাহরণ স্বরূপ, যদি আপনার ভিউ `resources/views/admin/profile.blade.php` এ সংরক্ষিত থাকে, তবে আপনি এটির মত উল্লেখ করতে পারেন:

    return view('admin.profile', $data);

#### যদি একটি ভিউ থাকে তা নির্ধারণ 

যদি আপনাকে যাচাই করার প্রয়োজন হলে যদি একটি ভিউ বিদ্যমান, আপনি ভিউ ফ্যাকাড ব্যবহার করতে পারেন। যদি ভিউ থাকে তাহলে `exists` মেথড `true` রিটার্ন করবে। 


    use Illuminate\Support\Facades\View;

    if (View::exists('emails.customer')) {
        //
    }

#### তৈরি করুন প্রথম উপলব্ধ(available) ভিউ

`first` মেথড টি ব্যবহার করে, আপনি আপনার প্রথম একটি ভিউ  তৈরি করতে পারেন যা একটি প্রদত্ত ভিউস অ্যারে উপস্থিত রয়েছে। আপনার অ্যাপ্লিকেশন বা প্যাকেজ যদি ভিউএস কাস্টমাইজ বা ওভাররাইট করার অনুমতি দেয় তবে এটি উপযোগী হবে। 

    return view()->first(['custom.admin', 'admin'], $data);

অবশ্যই, আপনি এই মেথড গুলিকে `ভিউ` এর মাধ্যমে কল করতে পারেন  [facade](/docs/{{version}}/facades):

    use Illuminate\Support\Facades\View;

    return View::first(['custom.admin', 'admin'], $data);

<a name="passing-data-to-views"></a>
## ভিউস তে ডাটা পাস

যেমন আপনি পূর্ববর্তী উদাহরনে দেখেছেন, আপনি ভিউস তে একটি ডাটা অ্যারে পাঠাতে পারেন:

    return view('greetings', ['name' => 'Victoria']);

When passing information in this manner, the data should be an array with key / value pairs. Inside your view, you can then access each value using its corresponding key, such as `<?php echo $key; ?>`. As an alternative to passing a complete array of data to the `view` helper function, you may use the `with` method to add individual pieces of data to the view:

    return view('greeting')->with('name', 'Victoria');

<a name="sharing-data-with-all-views"></a>
#### Sharing Data With All Views

Occasionally, you may need to share a piece of data with all views that are rendered by your application. You may do so using the view facade's `share` method. Typically, you should place calls to `share` within a service provider's `boot` method. You are free to add them to the `AppServiceProvider` or generate a separate service provider to house them:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            View::share('key', 'value');
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

<a name="view-composers"></a>
## View Composers

View composers are callbacks or class methods that are called when a view is rendered. If you have data that you want to be bound to a view each time that view is rendered, a view composer can help you organize that logic into a single location.

For this example, let's register the view composers within a [service provider](/docs/{{version}}/providers). We'll use the `View` facade to access the underlying `Illuminate\Contracts\View\Factory` contract implementation. Remember, Laravel does not include a default directory for view composers. You are free to organize them however you wish. For example, you could create an `app/Http/ViewComposers` directory:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;
    use Illuminate\Support\ServiceProvider;

    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function boot()
        {
            // Using class based composers...
            View::composer(
                'profile', 'App\Http\ViewComposers\ProfileComposer'
            );

            // Using Closure based composers...
            View::composer('dashboard', function ($view) {
                //
            });
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

> {note} Remember, if you create a new service provider to contain your view composer registrations, you will need to add the service provider to the `providers` array in the `config/app.php` configuration file.

Now that we have registered the composer, the `ProfileComposer@compose` method will be executed each time the `profile` view is being rendered. So, let's define the composer class:

    <?php

    namespace App\Http\ViewComposers;

    use Illuminate\View\View;
    use App\Repositories\UserRepository;

    class ProfileComposer
    {
        /**
         * The user repository implementation.
         *
         * @var UserRepository
         */
        protected $users;

        /**
         * Create a new profile composer.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            // Dependencies automatically resolved by service container...
            $this->users = $users;
        }

        /**
         * Bind data to the view.
         *
         * @param  View  $view
         * @return void
         */
        public function compose(View $view)
        {
            $view->with('count', $this->users->count());
        }
    }

Just before the view is rendered, the composer's `compose` method is called with the `Illuminate\View\View` instance. You may use the `with` method to bind data to the view.

> {tip} All view composers are resolved via the [service container](/docs/{{version}}/container), so you may type-hint any dependencies you need within a composer's constructor.

#### Attaching A Composer To Multiple Views

You may attach a view composer to multiple views at once by passing an array of views as the first argument to the `composer` method:

    View::composer(
        ['profile', 'dashboard'],
        'App\Http\ViewComposers\MyViewComposer'
    );

The `composer` method also accepts the `*` character as a wildcard, allowing you to attach a composer to all views:

    View::composer('*', function ($view) {
        //
    });

#### View Creators

View **creators** are very similar to view composers; however, they are executed immediately after the view is instantiated instead of waiting until the view is about to render. To register a view creator, use the `creator` method:

    View::creator('profile', 'App\Http\ViewCreators\ProfileCreator');
