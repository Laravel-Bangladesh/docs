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

এখন আপনার কাছে রাউট এবং ভিউ গুলো আছে সাথে দেওয়া অথেনটিকেশন করট্রলার এর জন্য, আপনি এখন তৈরি, আপনার এপ্লিকেশনে নতুন ব্যবহারকারি তৈরি এবং অথেনটিকেশন করার জন্য।
আপনি এখন ব্রাউজার থেকে আপনার অ্যাপ্লিকেশনটি অ্যাক্সেস করতে পারেন কারণ অথিন্টিকেশনের লজিক গুলি ইতিমধ্যে বিদ্যমান (তাদের traits এর মাধ্যমে) আগে থেকে থাকা ব্যবহারকারীদের লগিন এবং নতুন ব্যবহারকারীদের ডাটাবেস এ সংরক্ষণ করতে।

#### পাথ নিজের মত করে নেওয়া 

যখন একটি ব্যবহারকারী সঠিক ভাবে লগিন করবে, সে রিডাইরেক্ট হয়ে `/home` URI তে যাবে। আপনি লগিন করার পরের রিডাইরেক্ট লোকেশন পরিবর্তন করতে পারবেন  `redirectTo` প্রোপার্টি নির্ধারণ করে `LoginController`, `RegisterController`, `ResetPasswordController`, এবং `VerificationController` ক্লাসে :

    protected $redirectTo = '/';

পরবর্তীতে, ব্যবহারকারীকে রিডাইরেক্ট করার সময় আপনার নতুন URI ব্যবহার করার জন্য আপনাকে RedirectIfAuthorified মিডলওয়্যারের `handle` মেথড পরিবর্তন করতে হবে।

যতি আপনার রিডাইরেক্ট পাথ এর মধ্যে লিজিক ইমপ্লিমেন্ট করতে হয়ে তবে আপনি `redirectTo` প্রোপার্টি এর বদলে `redirectTo` মেথড ডিক্লেয়ার করতে পারেন:

    protected function redirectTo()
    {
        return '/path';
    }

> {tip} এখানে `redirectTo` মেথডের গুরুত্ব  `redirectTo` প্রোপার্টি এর থেকে বেশি দেওয়া হয়। 

#### ইউজারনেম নিজের মত করে নেওয়া

সাধারণ ভাবে, লারাভেল অথেন্টিকেইট করার জন্য `email` ফিল্ড ব্যবহার করে। আপনি যদি এটি পরিবর্তন করতে চান তবে আপনি `username` মেথড আপনার `LoginController` কন্ট্রোলারে ব্যবহার করতে পারেন :

    public function username()
    {
        return 'username';
    }

#### গার্ড নিজের মত করে নেওয়া

আপনি "guard" কাস্টমাইজ করতে পারেন যা ব্যবহারকারীদের অথেনটিকেশন এবং তৈরি করতে ব্যবহার করা হয়। শুরু করতে, আপনার `LoginController`, `RegisterController`, এবং `ResetPasswordController` এ একটি `guard` মেথড তৈরি করুন। এই মেথড অবশ্যই guard ইন্সটেন্স রিটার্ন করবেঃ 

    use Illuminate\Support\Facades\Auth;

    protected function guard()
    {
        return Auth::guard('guard-name');
    }

#### শুদ্ধতা পরীক্ষা / স্টোরেজ নিজের মত করে নেওয়া 

আপনার আপ্লিকেশনের মাধ্যমে নতুন ব্যবহারকারী তৈরির ফর্ম এর যে সকল আবশ্যক ফিল্ড আছে সেগুলো পরিবর্তন, অথবা নতুন ব্যবহারকারী কিভাবে আপনার ডাটাবেস এ স্টোর করবেন তা পরিবর্তন করতে, আপনি ইচ্ছে করলে `RegisterController` ক্লাস টি পরিবর্তন করতে পারেন । এই ক্লাস টি নতুন ব্যবহারকারীর তথ্য শুদ্ধতা পরীক্ষা এবং তৈরির কাজ করে আপনার আপ্লিকেশনে ।

`RegisterController` এর `validator` মেথড টিতে আপনার আপ্লিকেশনে নতুন ব্যবহারকারীর শুদ্ধতা পরীক্ষার নিয়ম গুলো দেওয়া আছে । আপনি ইচ্ছা করলে এই মেথড টি আপনার মত করে পরিবর্তন করে নিতে পারেন।  

`RegisterController` এর `create` মেথড টির দায়িত্ব হলো আপনার ডেটাবেস এ `App\User` এর নতুন রেকর্ড তৈরি করা [Eloquent ORM](/docs/{{version}}/eloquent) ব্যবহার করে । আপনি ইচ্ছা করলে এই মেথড টি আপনার মত করে আপনার ডাটাবেস অনুযায়ী পরিবর্তন করে নিতে পারেন।

<a name="retrieving-the-authenticated-user"></a>
### অথেন্টিকেইট ব্যবহারকারীর তথ্য নেওয়া  

আপনি ইচ্ছে করলে অথেন্টিকেইট ব্যবহারকারী এক্সেস করতে পারবেন `Auth` ফেছাদ এর মাধ্যমে 

    use Illuminate\Support\Facades\Auth;

    // Get the currently authenticated user...
    $user = Auth::user();

    // Get the currently authenticated user's ID...
    $id = Auth::id();

বিকল্পভাবে, একবার একটি ব্যবহারকারী যদি অথেন্টিকেইট হয়, তবে আপনি `Illuminate\Http\Request` ক্লাসের মাধ্যমে অথেন্টিকেইট ব্যবহারকারীকে এক্সেস করতে পারবেন। মনে রাখবেন, টাইপ হিন্টেড ক্লাস স্বয়ংক্রিয়ভাবে আপনার কন্ট্রোলার এর মেথডে ইনজেক হবে:

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

#### বর্তমান ব্যবহারকারী অথেন্টিকেট কিনা তা নির্ধারণ

ব্যবহারকারী আপনার এপ্লিকেশনে লগিন কি না তা নির্ধারণ করতে, আপনি ইচ্ছা করলে `Auth` ফেসাদের `check` মেথড ব্যবহার করতে পারেন, যেটা `true` রিটার্ন করবে যদি ব্যবহারকারী অথেন্টিকেট হয় ।

    use Illuminate\Support\Facades\Auth;

    if (Auth::check()) {
        // The user is logged in...
    }

> {tip} যদিও এটা সম্ভব যে আপনি নির্ধারণ করতে পারবেন ব্যবহারকারী অথেন্টিকেট কি না `check` মেথড এর মাধ্যমে, কিন্তু সাধরনত আপনি মিডিলওয়ার ব্যবহার করবেন যাচাই করতে ব্যবহারকারী অথেন্টিকেট কিনা কিছু কন্ট্রোলার / রাউট এক্সেস করার পূর্বে। এই বিষয়ে আরও জানতে, [protecting routes](/docs/{{version}}/authentication#protecting-routes) এই ডকুমেন্টেশন দেখে আসতে পারেন।

<a name="protecting-routes"></a>
### রাউটগুলোকে সুরক্ষা দেওয়া 

[Route middleware](/docs/{{version}}/middleware) ব্যবহার হয় কিছু রাউট গুলো কে শুধুমাত্র অথেন্টিকেট ব্যবহারকারী দের এক্সেস দিতে। লারাভেল এসেছে একটি `auth` মিডিলওয়ার সাথে নিয়ে, যেটা ডিফাইন করা আছে `Illuminate\Auth\Middleware\Authenticate` এখানে। যদিও এই মিডলওয়্যার আপনার HTTP কার্নেল এ আগে থেকেই রেজিস্টার্ড করা আছে, আপনাকে যা করতে হবে তা হল রাউট ডিফাইন করার সময় এই মিডলওয়্যার সংযুক্ত করতে হবে: 

    Route::get('profile', function () {
        // Only authenticated users may enter...
    })->middleware('auth');

অবশ্য, যদি আপনি [controllers](/docs/{{version}}/controllers) ব্যবহার করে থাকেন, আপনি ইচ্ছে করলে `middleware` মেথড কল করতে পারবেন ঐ কন্ট্রোলার এর constructor মেথড এর মধ্যে, সরাসরি রাঊটের মধ্যে ডিফাইনের পরিবর্তে ।

    public function __construct()
    {
        $this->middleware('auth');
    }

#### Redirecting অননুমোদিত ব্যবহারকারীদের

যখন `auth` মিডিলওয়ার ডিটেক্ট করে অননুমোদিত ব্যবহারকারী, এটা ঐ ব্যবহারকারী কে `login` [named route](/docs/{{version}}/routing#named-routes) রুটে রিডাইরেক্ট করবে। আপনি ইচ্ছা করলে পরিবর্তন করতে পারবেন এর কাজের ধারা `app/Http/Middleware/Authenticate.php` ফাইলের `redirectTo` মেথড পরিবর্তন করার মাধ্যমে। 

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

#### গার্ড নির্ধারিত করা 

যখন একটি রাউটে `auth` মিডিলওয়ার আট্যাচ করা হয়, আপনি চাইলে এইটিও নির্ধারিত করে দিতে পারবেন, কোন গার্ড ব্যবহার করে ব্যবহারকারীকে অথেন্টিকেট করবে। গার্ড নির্ধারণ করতে `auth.php` কনফিগারেশন ফাইলে `guards` এরে তে কি আকারে দিতে হবে।

    public function __construct()
    {
        $this->middleware('auth:api');
    }

<a name="login-throttling"></a>
### লগইন থ্রোটলিং

আপনি যদি লারাভেলের তৈরিকৃত `LoginController` ক্লাস ব্যবহার করে থাকেন, তাহলে `Illuminate\Foundation\Auth\ThrottlesLogins` ট্রেইট আগে থেকেই আপনার কন্ট্রোলার এ ইনক্লুড করা আছে। ডিফল্টরূপে, ব্যবহারকারী যদি কয়েকবার একটানা তার লগিন ক্রেডেনশিয়াল ভুল করে, তবে সে ১ মিনিট লগিন করতে পারবে না। এই থ্রোটলিং ব্যবহারকারীকে তার ইউজারনেম/ইমেইল এবং আইপি এড্রেস দ্বারা চিনহিত করে। 


<a name="authenticating-users"></a>
## ম্যানুয়ালি ইউজারকে লগইন করানো

অবশ্য, আপনার লারাভেলের সাথে দেওয়া অথেন্টিকেশন কন্ট্রোলার ব্যবহার করার কোন বাধ্যবাধকতা নাই । আপনি ইচ্ছে করলে এই কন্ট্রোলার গুলো মুছে ফেলতে পারেন, আপনি সরাসরি লারাভেলের অথেনটিকেশন ক্লাস ব্যবহার করেও ব্যবহারকারীর অথেনটিকেশন করতে পারেন। চিন্তার করবেন না, এটি খুবই সহজ। 

আমরা লারাভেলের এর অথেনটিকেশন সার্ভিস `Auth` [facade](/docs/{{version}}/facades) ফেছাদ এর মাধ্যমে এক্সেস করতে পারবো, তো আমাদের ক্লাসে সবার উপরে `Auth` ফেছাদ টিকে ইম্পরট করতে হবে। এরপরে, চলুন `attempt` মেথড টি চেক করে দেখিঃ

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

`attempt` মেথড টি তার প্রথম আর্গুমেন্ট হিসেবে কি/ভ্যলু আকারের array এক্সেপ্ট করে। এই ভ্যলুর দ্বারাই ডাটাবেস টেবিলে ব্যবহারকারী কে খুঁজা হবে। তো, উপরের উধাহরনে, ব্যবহারকারীকে `email` কলামের ভ্যলুর দ্বারা খুঁজে আনা হয়েছে। যদি ব্যবহারকারীকে খুঁজে পাওয়া যায়, তবে ডাটাবেসে সংরক্ষিত থাকা হ্যাস পাসওয়ার্ডের সাথে মেথডে পাঠানো array এর `password` এর ভ্যালুর সাথে মিলানো হয়। আপনার নিদিষ্ট করে `password` এর ভ্যালু কে হ্যাস করার দরকার নাই, ফ্রেমওয়ার্ক ডেটাবেসের মধ্যে থাকা হ্যাস্কৃত পাসওয়ার্ড এর সাথে মিলানোর পূর্বে সয়ংক্ক্রিত ভাবে হ্যাস করে নিবে। যদি দুটি হ্যাস কৃত পাসওয়ার্ড মিলে যায় তবে ওই ব্যবহারকারীর জন্য একটি অথেন্টিকেটেড সেশন চলতে শুরু করে । 

`attempt` মেথড টি `true` রিটার্ন করে যদি এথেন্টিকেশন ঠিক ভাবে হয়। নাহলে এইটি `false` রিটার্ন করে। 

রিডাইরেক্টরের `intended` মেথড ব্যবহারকারী ওই URL এ নিয়ে যাবে যেখানে যাওয়ার সময় অথেন্টিকেট মিদিলয়ার তাকে আটকে দিয়েছ। আপনি একটি বিকল্প URL ওই মেথডে দিতে পারেন যদি উদ্দেশ্যে গন্তব্য না থাকে। 

#### অতিরিক্ত শর্ত উল্লেখ করা

আপনি যদি চান তবে ব্যবহারকারীর ইমেইল এবং পাসওয়ার্ডের পাশাপাশি আপনি অতিরিক্ত শর্তাদি যোগ করতে পারেন। উদাহরণস্বরূপ, আমরা যাচাই করতে চাই ব্যবহারকারীটি "active" হিসাবে ও চিহ্নিত আছে:

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
        // The user is active, not suspended, and exists.
    }

> {note} এই উদাহরণগুলিতে, 'ইমেল' কিন্তু অবশ্যক অপশন নয়, এটি কেবল উদাহরণ হিসাবে ব্যবহৃত হয়েছে। আপনি আপনার ডাটাবেসের যেকোনো কলামের নাম ব্যবহার করতে পারেন, যেটি আপনার ব্যবাহারকারির "username" হিসেবে ব্যবহার করা হয়েছে।

#### নির্দিষ্ট গার্ড এর ইন্সটেন্স অ্যাক্সেস করা

`Auth` ফেসাদের `guard` মেথড ব্যবহার করে আপনি কোন গার্ড ইনস্ট্যান্সটি ব্যবহার করতে চান তা নির্দিষ্ট করতে পারেন। এটি আপনাকে আপনার আপ্লিকেশনের আলাদা আলাদা পার্টের আলাদা অথেন্টিকেবল মডেল অথবা ব্যবহারকারী টেবিলের সাথে অথেনটিকেশনের সুযোগ করে দিবে। 

`guard` মেথডে প্রেরিত গার্ড এর নামটি আপনার `auth.php` কনফিগারেশন ফাইলে কনফিগার করা গার্ড লিস্টের মধ্যে একটি হইতে হবে:

    if (Auth::guard('admin')->attempt($credentials)) {
        //
    }

#### লগ আউট করা

আপনার অ্যাপ্লিকেশন থেকে ব্যবহারকারীদের লগ আউট করতে, আপনি `Auth` ফেসাদের `logout` মেথড টি ব্যবহার করতে পারেন। এটি ব্যবহারকারীর সেশনের অথেনটিকেশনের তথ্য গুলো মুছে দিবে:


    Auth::logout();

<a name="remembering-users"></a>
### ব্যবহারকারীদের মনে রাখা

আপনি যদি আপনার অ্যাপ্লিকেশনটিতে "remember me" ফাংশনালিটি তৈরি করতে চান, তবে আপনি `attempt` মেথডে দ্বিতীয় আর্গুমেন্ট হিসাবে একটি বুলিয়ান মান পাস করতে পারেন, যা ব্যবহারকারীকে অনির্দিষ্টকালের জন্য মনে রাখবে, কিংবা তারা নিজে লগআউট না করা পর্যন্ত। অবশ্যই, আপনার ব্যবহারকারীদের টেবিলে অবশ্যই `remember_token` কলামটি থাকতে হবে, যা "remember me" টোকেন সংরক্ষণ করতে ব্যবহার করা হবে।

    if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
        // The user is being remembered...
    }

> {tip} আপনি যদি বিল্টইন `LoginController` ব্যবহার করেন যা লারভেলের সাথে দেওয়া হয়েছিল, তবে ব্যবহারকারীদের "মনে রাখার" সঠিক যুক্তিটি ইতিমধ্যে ট্রেইটের দ্বারা কন্ট্রোলারে ইমপ্লিমেন্ট করা আছে।

আপনি যদি ব্যবহারকারীদের "মনে রাখেন", তবে ব্যবহারকারীটি "remember me" cookie ব্যবহার করে অথেন্টিকেইট হয়েছে কিনা তা নির্ধারণের জন্য আপনি 'via Remember` মেথড ব্যবহার করতে পারেন:

    if (Auth::viaRemember()) {
        //
    }

<a name="other-authentication-methods"></a>
### অন্যান্য অথেনটিকেশন মেথড

#### একটি ইউজার ইন্সটেন্সকে অথেন্টিকেট করা

আপনার অ্যাপ্লিকেশনে বিদ্যমান ইউজার ইন্সটেন্স কে লগ ইন করতে হলে, আপনি ইউজার ইন্সটেন্স কে `login` মেথডের মধ্যে কল করতে পারেন। প্রদত্ত অব্জেক্টি তে `Illuminate\Contracts\Auth\Authenticatable` [contract](/docs/{{version}}/contracts) ইন্টারফেইস টিতে ইমপ্লিমেন্টেট থাকতে হবে। অবশ্যই, লারাভেল এর সাথে অন্তর্ভুক্ত করা `App\User` মডেলটি ইতিমধ্যেই এই ইন্টারফেসটিতে ইমপ্লিমেন্ট করা আছে:

    Auth::login($user);

    // Login and "remember" the given user...
    Auth::login($user, true);

অবশ্যই, আপনি যে গার্ড ইনস্ট্যান্সটি ব্যবহার করতে চান তার উল্লেখ করতে পারেন

    Auth::guard('admin')->login($user);

#### ইউজার এর আইডি ব্যবহার করে অথেন্তিকেট করা।

ব্যবহারকারী কে আপনার আপ্লিকেশনে তাদের আইডি দিয়ে লগইন করানোর জন্য, `loginUsingId` মেথড টি ব্যবহার করতে পারেন। এই মেথড টি প্যরামিটার হিসেবে ইউজারের প্রাইমারি কি একসেপ্ট করবে যাকে আপনি অথেন্টিকেইট করতে চানঃ

    Auth::loginUsingId(1);

    // Login and "remember" the given user...
    Auth::loginUsingId(1, true);

#### ব্যবহারকারী কে একবারের জন্য অথেন্টিকেইট করা

আপনি ইচ্ছে করলে `once` মেথড ব্যবহার করে আপনার আপ্লিকেশনে ব্যবহারকারী কে শুধু মাত্র একবারের জন্য লগইন করাতে পারেন। এজন্য কোন রকম সেশন কিংবা কুকি ব্যবহার হবে না, তার মানে এই মেথড টি অনেক প্রয়োজনীয় হতে পারে যখন আপনি stateless API বানাবেনঃ 

    if (Auth::once($credentials)) {
        //
    }

<a name="http-basic-authentication"></a>
## HTTP Basic Authentication

[HTTP Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication) আপনাকে খুবই দ্রুত আপনার আপ্লিকেশনে ব্যাবহারকারী দের লগইন করার সিস্টেম দিবে কোন প্রকার লগইন পেইজ তৈরি করা ছাড়া। এটি শুরু করতে হলে, আপনাকে রাউটে `auth.basic` [middleware](/docs/{{version}}/middleware) যুক্ত করতে হবে। `auth.basic` লারাভেল ফ্রেমওয়ার্ক এর সাথে দেওয়াই থাকে, তাই আপনার এটি আলাদা ভাবে ডিফাইন করার দরকার নাইঃ 

    Route::get('profile', function () {
        // Only authenticated users may enter...
    })->middleware('auth.basic');

মিডিলয়ার রাউটে যুক্ত করা হয়ে গেলে, আপনার কাছে স্বয়ংক্রিয় ভাবে লগইন এর তথ্য চাওয়া হবে যখন আপনি ব্রাউজারের মাধ্যমে এক্সেস করবেন। সাধারণত `auth.basic` মিডিলয়ার ব্যবহারকারীর তথ্যের `email` কলাম "ইউজারনেম" হিসেবে ব্যবহার করে।

#### A Note On FastCGI

আপনি যদি PHP FastCGI ব্যবহার করে থাকেন, HTTP Basic authentication তাহলে ঠিক ভাবে শুরুতে কাজ নাও করতে পারে। এজন্য নিচের লাইন গুলো আপনার `.htaccess` ফাইলে এড করতে হবেঃ 

    RewriteCond %{HTTP:Authorization} ^(.+)$
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="stateless-http-basic-authentication"></a>
### Stateless HTTP Basic Authentication

আপনি আরও HTTP Basic Authentication ব্যবহার করতে পারবেন একটি ব্যবহারকারীর আইডেন্টিফায়ার কুকি কোন রকম সেশনে সেট করা ছাড়াই, যেটা API authentication এর জন্য বেশ দরকারি। তো এটি করতে [একটি মিডিলয়ার ডিফাইন করতে হবে](/docs/{{version}}/middleware) যেটা `onceBasic` মেথডটি কল করবে। যদি কোন রেসপন্স পাওয়া না যায় `onceBasic` মেথড থেকে, তাহলে সেই রিকোয়েস্ট টি পরে আপ্লিকেশনে পাস হয়ে যাবেঃ

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

এর পরে, [রাউট মিডিলয়ারে রেজিস্টার করতে হবে](/docs/{{version}}/middleware#registering-middleware) এবং এটিকে রাউটে যুক্ত করে দিতে হবেঃ

    Route::get('api/user', function () {
        // Only authenticated users may enter...
    })->middleware('auth.basic.once');

<a name="logging-out"></a>
## লগআউট করা

আপনার আপ্লিকেশন থেকে ব্যবহারকারীকে ম্যনুয়ালি লগ আউট করতে চাইলে `Auth` ফেসাদের `logout` মেথড ব্যবহার করতে পারেন। এটি অথেনটিকেশন তথ্য ব্যবহারকারীর সেশন থেকে মুছে দিবেঃ 

    use Illuminate\Support\Facades\Auth;

    Auth::logout();

<a name="invalidating-sessions-on-other-devices"></a>
### অন্য ডিভাইসের সেশন নষ্ট করা

লারাভেল এছাড়াও ব্যবহারকারীদের বর্তমান ডিভাইসে সেশন ঠিক রেখে অন্য ডিভাইসে সক্রিয় ব্যবহারকারীর সেশন নষ্ট এবং "লগ আউট" করার জন্য একটি মেথড  দিয়ে থেকে। শুরু করার আগে, আপনাকে অবশ্যই নিশ্চিত করতে হবে `Illuminate\Session\Middleware\AuthenticateSession` মিডলওয়্যার আছে এবং আপনার `app/Http/Kernel.php` ক্লাসে `web` মিডলওয়্যার গ্রুপে আন কমেন্ট করা আছেঃ 

    'web' => [
        // ...
        \Illuminate\Session\Middleware\AuthenticateSession::class,
        // ...
    ],

তারপর, আপনি ইচ্ছে করলে `Auth` ফেসাদের `logoutOtherDevices` মেথডটি ব্যবহার করতে পারেন। এই মেথড ব্যবহার করতে হলে ব্যবহারকারীর বর্তমান পাসওয়ার্ড টি দিতে হবে, যেটা আপনার এপ্লিকেশন এর ইনপুট ফর্ম থেকে নিতে পারেনঃ 

    use Illuminate\Support\Facades\Auth;

    Auth::logoutOtherDevices($password);

> {note} যখন `logoutOtherDevices` মেথডটি কল করা হয়, তখন ব্যবহারকারীদের অন্যান্য সেশন গুলো নষ্ট হয়ে যায়, তার মানে তাদের সকল গার্ড থেকে লগ আউট হয়ে যাবে যা তারা পূর্বে অথেন্টিকেইট করে রেখেছিল। 

<a name="adding-custom-guards"></a>
## কাস্টম গার্ড যোগ করা

আপনি ইচ্ছা করলে নিজস্ব গার্ড ডিফাইন করতে পারেন `Auth` ফেসাদের `extend` মেথড ব্যবহার করে। আপনার `extend` মেথডের কলিং এর কোড গুলো কোন এক [service provider](/docs/{{version}}/providers) এর মধ্যে রাখতে হবে। আর লারাভেল আগে থেকেই `AuthServiceProvider` সহ এসেছে, আমরা এই কোড গুলো এই প্রভাইডারের মধ্যে রাখতে পারিঃ

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

আপনি উপরের উধাহরন থেকে দেখতে পারেন, যে কলব্যাক টি `extend` মেথডে পাস করা হয়েছে সেটি `Illuminate\Contracts\Auth\Guard` এর ইমপ্লিমেটেসন রিটার্ন করছে। এই ইন্টারফেসটি তে অনেকগুলো মেথড রয়েছে যেগুলো আপনাকে ডিফাইন করতে হবে কাস্টম গার্ড তৈরি করতে। আপনার কাস্টম গার্ড ডিফাইন করা হয়ে গেলে, আপনি ইচ্ছে করলে এই গার্ডটি `auth.php` কনফিগারেশন ফাইলে `guards` এ ব্যবহার করতে পারেনঃ 

    'guards' => [
        'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
        ],
    ],

<a name="closure-request-guards"></a>
### ক্লোজার রিকুয়েস্ট গার্ড



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
