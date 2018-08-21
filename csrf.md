# CSRF সুরক্ষা

- [সূচনা](#csrf-introduction)
- [Excluding URIs](#csrf-excluding-uris)
- [X-CSRF-Token](#csrf-x-csrf-token)
- [X-XSRF-Token](#csrf-x-xsrf-token)

<a name="csrf-introduction"></a>
## সূচনা

লারাভেল আপনার অ্যাপ্লিকেশনটিকে ধোঁকাবাজদের  [ক্রস সাইট অনুরোধ জালিয়াতি](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF) আক্রমণ থেকে সহজে সুরক্ষা প্রধান করে। ক্রস সাইট অনুরোধ জালিয়াতি একটি ওয়েব সাইট জন্য একধরনের দূষিত, একটি অথেন্টিকেটেড ব্যবহারকারী কমেন্ড এর মতোই।

প্রত্যেক সক্রিয় ব্যবহারকারীর সেশনে লারাভেল অ্যাপ্লিকেশান দ্বারা স্বয়ংক্রিয়ভাবে CSRF "token" তৈরি করে। এই টোকেনটি দ্বারা যাচাই করার হয় সত্যিকার অর্থে অথেন্টিকেটেড ব্যবহারকারী অ্যাপ্লিকেশনের জন্য অনুরোধ(requests) করেছে কিনা।

যে কোন সময় আপনার অ্যাপ্লিকেশানটিতে HTML ফর্ম ডিফাইন করতে পারেন, আপনাকে অবশ্যই একটি hidden CSRF token field অন্তরবুক্ত করতে হবে। যাতে CSRF protection middleware অনুরোধটি যাচাই করতে পারে। আপনে সম্ভাবত `@csrf` ব্যাবহার করবেন Blade একটি  token field তৈরি করতে নির্দেশিত করবে।

    <form method="POST" action="/profile">
        @csrf
        ...
    </form>

`VerifyCsrfToken` [middleware](/docs/{{version}}/middleware),
যা `web` middleware group এ অন্তর্ভুক্ত করা রয়েছে, স্বয়ংক্রিয়ভাবে টকেন যাচাই করা হয় যে অনুরোধ(request) করা ইনপুটে টোকেন সেশনে সঞ্চিত টোকেন এর সাথে মিলে কিনা।


#### CSRF Tokens & JavaScript

যখন জাভাস্ক্রিপ্ট চালিত অ্যাপ্লিকেশন তৈরি করবেন, আপনার জাভাস্ক্রিপ্ট `HTTP` লাইব্রেরি স্বয়ংক্রিয়ভাবে প্রতিটি আউটগোয়িং অনুরোধে ( outgoing request ) CSRF টোকেন সংযুক্ত করে। ডিফল্ট ভাবে,  `resources/assets/js/bootstrap.js` ফাইলটি
`Axios HTTP` লাইব্রেরি সঙ্গে মেটা ট্যাগের `csrf- টোকেন` এর value অন্তর্ভুক্ত করে। আপনি যদি এই লাইব্রেরি ব্যবহার না করেন, আপনাকে অ্যাপ্লিকেশনের জন্য এই আচরণটি ম্যানুয়ালয় কনফিগার করতে হবে।

<a name="csrf-excluding-uris"></a>
## Excluding URIs From CSRF সুরক্ষা

কখনও কখনও আপনে আগ্রহী হবেন উল্লিখিত একটি সেট URI গুলিতে CSRF সুরক্ষা প্রক্রিয়া চালাতে। উদাহরণস্বরূপ, যদি আপনি পেমেন্ট প্রক্রিয়া করার জন্য [Stripe](https://stripe.com) ব্যবহার করেন এবং তাদের webhook সিস্টেম কাজে লাগান,

আপনার প্রয়োজন হবে CSRF সুরক্ষা webhook handler route এ exclude করতে হবে, যেহেতু Stripe জানতে পারবে যে CSRF টোকেন  আপনার route গুলিতে পাঠানো হয়েছে কিনা।

সাধারণত, আপনাকে `RouteServiceProvider` এর `routes/web.php` ফাইলের  `web` মিডিলওয়্যার গ্রুপের বাইরে এই ধরনের রাউটগুলি স্থাপন করা উচিত।

যাইহোক, আপনি `VerifyCsrfToken` মিডিলারের URI গুলিকে `$except` property কে exclude করতে চাইবেন।


    <?php

    namespace App\Http\Middleware;

    use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as Middleware;

    class VerifyCsrfToken extends Middleware
    {
        /**
         * The URIs that should be excluded from CSRF verification.
         *
         * @var array
         */
        protected $except = [
            'stripe/*',
            'http://example.com/foo/bar',
            'http://example.com/foo/*',
        ];
    }

> যখন [tests প্রক্রিয়া ](/docs/{{version}}/testing) চলমান, তখন CSRF মিডডওয়াল স্বয়ংক্রিয়ভাবে বন্ধ (disabled) থাকে।

<a name="csrf-x-csrf-token"></a>
## X-CSRF-TOKEN

POST প্যারামিটারের হিসাবে CSRF টোকেন যাচাই করার পাশাপাশি `VerifyCsrfToken` মিডিলওয়্যারে, `X-CSRF-TOKEN` অনুরোধ(request) হেডারের জন্যও যাচাই করবে


উদাহরণস্বরূপ, আপনি একটি এইচটিএমএল `meta` ট্যাগে টোকেন সংরক্ষণ করতে পারেন:

    <meta name="csrf-token" content="{{ csrf_token() }}">

তারপর, একবার আপনি মেটা ট্যাগ তৈরি করেছেন, আপনে জকুয়ার‍্য লাইব্রেরি এর মাধ্যমে নির্দেশনা দিয়ে, সমস্ত অনুরোধ (request) হেডারগুলিতে টোকেন যোগ করতে পারেন। এটি AJAX এর ভিত্তিক অ্যাপ্লিকেশন জন্য সহজ, সুবিধাজনক CSRF সুরক্ষা প্রদান করে:

    $.ajaxSetup({
        headers: {
            'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
        }
    });

> {টিপ} ডিফল্টরূপে, `resources/assets/js/bootstrap.js` ফাইল `Axios HTTP` লাইব্রেরি  `csrf-token` মেটা ট্যাগের সাথে যুক্ত(register) করে । আপনি যদি এই লাইব্রেরিটি ব্যবহার না করেন, তবে আপনাকে অ্যাপ্লিকেশনে এই  আচরণের(behavior) জন্য ম্যানুয়ালয় কনফিগার করতে হবে।

<a name="csrf-x-xsrf-token"></a>
## X-XSRF-TOKEN

লারাভেল বর্তমান CSRF টোকেন  `XSRF-TOKEN` কুকিতে সঞ্চয় করে যা প্রতিটি প্রতিক্রিয়াতে ফ্রেমওয়ার্ক দ্বারা উৎপন্ন হয়।  আপনি কুকি ভ্যালু `X-XSRF-TOKEN` request header এ ব্যবহার করতে পারেন।

এই কুকি মূলত কিছু জাভাস্ক্রিপ্ট ফ্রেমওয়ার্ক এবং লাইব্রেরি সাথে একটি সুবিধা জনক ভাবে [primarily] পাঠানো হয়। যেমন  Angular এবং Axios, স্বয়ংক্রিয়ভাবে  `X-XSRF-TOKEN` header তার মানটি রাখে।

এই কুকি মূলত কিছু জাভাস্ক্রিপ্ট ফ্রেমওয়ার্ক এবং লাইব্রেরি সাথে একটি সুবিধা জনক ভাবে [primarily] পাঠানো হয়,

like Angular and Axios, automatically place its value in the `X-XSRF-TOKEN` header.

যেমন  Angular এবং Axios, স্বয়ংক্রিয়ভাবে  `X-XSRF-TOKEN` header তার মানটি রাখে।
