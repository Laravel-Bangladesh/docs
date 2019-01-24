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
