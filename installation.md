# Installation

- [Installation](#installation)
    - [Server Requirements](#server-requirements)
    - [Installing Laravel](#installing-laravel)
    - [Configuration](#configuration)
- [Web Server Configuration](#web-server-configuration)
    - [Pretty URLs](#pretty-urls)

<a name="installation"></a>
## Installation

> {video} লারাভেল সম্পর্কে নতুনদের জন্য  Laracasts [বিনামূল্যে প্রাথমিক ধারণা](http://laravelfromscratch.com) প্রধান করে। এটি যাত্রা শুরু করার জন্য একটি ভালো জায়গা।

<a name="server-requirements"></a>
### Server Requirements

লারাভেল ফ্রেমওয়ার্ক কিছু সিস্টেম আবশ্যকতা রয়েছে। অবশ্যই, সমস্ত প্রয়োজনীয়তা পূরণ করে লারাভেল ইনস্টল করা উচিত। 
এই সব আবশ্যকতা  [Laravel Homestead](/docs/{{version}}/homestead) দ্বারা ভার্চুয়াল মেশিন দ্বারা সন্তুষ্ট হয়, তাই এটি আপনার লোকাল  লারাভেল ডেভেলপমেন্ট পরিবেশের হিসাবে হোমসাইট্ড ব্যবহার করার জন্য অত্যন্ত উপযোগী।

যাইহোক, যদি আপনি হোমস্টেড ব্যবহার না করেন, তবে আপনার সার্ভার নিম্নলিখিত প্রয়োজনীয়তাগুলি নিশ্চিত করতে হবে:

<div class="content-list" markdown="1">
- PHP >= 7.1.3
- OpenSSL PHP Extension
- PDO PHP Extension
- Mbstring PHP Extension
- Tokenizer PHP Extension
- XML PHP Extension
- Ctype PHP Extension
- JSON PHP Extension
- BCMath PHP Extension
</div>

<a name="installing-laravel"></a>
### Installing Laravel
লারাভেল প্যাকেজ dependencie  ম্যানেজার হিসাবে [কম্পোজার](https://getcomposer.org) ব্যবহার করে। সুতরাং, লারাভেল ব্যবহার করার আগে নিশ্চিত করুন যে আপনি আপনার মেশিনে কম্পোজার ইনস্টল করা আছে কিনা ।

#### Via Laravel Installer

প্রথমে, কম্পোজার ব্যবহার করে Laravel ইন্সটলার ডাউনলোড করুন:

    composer global require laravel/installer

আপনার `$PATH`তে সুরকারের system-wide vendor bin ডিরেক্টরি স্থাপন করা নিশ্চিত করুন যাতে আপনার সিস্টেমের দ্বারা লরেভেল এক্সিকিউটেবলটি সনাক্ত করা যায়। আপনার অপারেটিং সিস্টেমের উপর ভিত্তি করে এটি বিভিন্ন স্থানে বিদ্যমান; তবে, কিছু সাধারণ অবস্থানে রয়েছে:

<div class="content-list" markdown="1">
- macOS: `$HOME/.composer/vendor/bin`
- GNU / Linux Distributions: `$HOME/.config/composer/vendor/bin`
</div>

একবার ইনস্টল করা হলে, `laravel new` কমান্ডটি আপনার নির্দিষ্ট করা ডিরেক্টরীতে একটি নতুন লারাভেল ইন্সটলেশন প্রজেক্ট  তৈরি করবে। উদাহরণস্বরূপ, `laravel new blog` command `blog` নামের একটি ডিরেক্টরি তৈরি করে,  যা লারাভেল সমস্ত নির্ভরযোগ্যতাগুলি ইতিমধ্যে ইনস্টল করা হয়েছে তা দিয়ে একটি নতুন লারাভেল ইনস্টল করা হয়েছে:

    laravel new blog

#### Via Composer Create-Project

অন্যথা, আপনি কম্পোজারের `create-project` আপনার টার্মিনালে কমান্ড মাধ্যমে লারাভেল ইনস্টল করতে পারেন:

   composer create-project --prefer-dist laravel/laravel blog

#### Local Development Server

যদি আপনার PHP লোকাল মিশন এ ইনস্টল থাকেন এবং আপনি PHP এর বিল্ট ইন ডেভেলপমেন্ট সার্ভার ব্যবহার করতে চান আপনার অ্যাপ্লিকেশন এ, আপনি Artisan এর `serve` কমান্ড ব্যবহার করুন। এই কমান্ডটি একটি ডেভেলপমেন্ট সার্ভার শুরু করবে যা `http://localhost:8000`:

    php artisan serve


অবশ্যই, আরও শক্তিশালী স্থানীয় উন্নয়ন বিকল্পগুলি পাওয়া যাবে  [Homestead](/docs/{{version}}/homestead) এবং [Valet](/docs/{{version}}/valet) তে। 

<a name="configuration"></a>
### Configuration

#### Public Directory

লারাভেল ইনস্টল করার পর, আপনার ওয়েব সার্ভারের document / web root হিসাবে `public`  ডিরেক্টরি কে কনফিগার করা উচিত। এই ডিরেক্টরিতে `index.php` আপনার অ্যাপ্লিকেশনে প্রবেশের জন্য সমস্ত HTTP requests এর জন্য ফ্রন্ট কন্ট্রোলার হিসাবে কাজ করে।

#### Configuration Files

লারাভেল ফ্রেমওয়ার্কের জন্য সমস্ত কনফিগারেশন ফাইল `config` ডিরেক্টরির মধ্যে সংরক্ষণ করা হয়। প্রতিটি বিকল্প নথিভুক্ত করা হয়, তাই ফাইলগুলি দেখুন আপনার কাছে উপলব্ধ বিকল্পগুলির সাথে পরিচিত হন।

#### Directory Permissions

লারাভেল ইনস্টল করার পর,আপনি কিছু permissions কনফিগার করতে হতে পারে। storage এবং bootstrap/cache ডিরেক্টরিগুলির মধ্যে থাকা ডিরেক্টরিগুলি আপনার ওয়েব সার্ভারের মাধ্যমে writable হওয়া উচিত অন্যথায় লারাভেল চলবে না। If আপনি ভার্চুয়াল মেশিন [Homestead](/docs/{{version}}/homestead)  ব্যবহার  করেন, এই অনুমতিগুলি ইতিমধ্যে দেয়া আছে ।


#### Application Key

লারাভেল ইনস্টল করার পর আপনি যা করতে হবে তা আপনার অ্যাপ্লিকেশন কী হিসাবে একটি random string এ সেট করা হয়। যদি আপনি কম্পোজার বা লারাভেল ইন্সটলার মাধ্যমে লারাভেল  ইনস্টল করেন, এই কীটি ইতিমধ্যে আপনার জন্য সেট করা হয়েছে  `php artisan key:generate` কমান্ড দ্বারা।

সাধারণত, এই স্ট্রিংটি 32 অক্ষর দীর্ঘ হয় । কী   `.env`  environment ফাইলের মধ্যে সেট করা হয়। যদি  `.env.example` ফাইলটি `.env` তে নামকরণ না করে হয় তবে আপনাকে এখন এটি করতে হবে। ** যদি অ্যাপ্লিকেশন কী সেট না করা হয়, তাহলে আপনার ব্যবহারকারীর সেশনগুলি এবং অন্যান্য এনক্রিপ্ট হওয়া ডেটা সুরক্ষিত হবে না! **

#### Additional Configuration

লারাভেল প্রায় প্রয়োজন বাক্সের বাইরে অন্য কোন কনফিগারেশন। আপনি ডেভেলপিং শুরু বিনামূল্যে করতে পারবেন! যাহোক, আপনি `config/app.php` ফাইল এবং এর ডকুমেন্টেশন পর্যালোচনা করতে পারেন। এটা যেমন `timezone` এবং` locale` যে আপনি আপনার প্রয়োজন অনুযায়ী পরিবর্তন করতে চান তাহলে,  বেশ কয়েকটি বিকল্প রয়েছে।

আপনি লারাভেল এর কয়েকটি অতিরিক্ত উপাদান কনফিগার করতে পারেন, যেমন:

<div class="content-list" markdown="1">
- [Cache](/docs/{{version}}/cache#configuration)
- [Database](/docs/{{version}}/database#configuration)
- [Session](/docs/{{version}}/session#configuration)
</div>


<a name="web-server-configuration"></a>
## Web Server Configuration

<a name="pretty-urls"></a>
### Pretty URLs

#### Apache


লারাভেল - এ একটি  `public/.htaccess`  ফাইল অন্তর্ভুক্ত রয়েছে যা `index.php` path সামনে কন্ট্রোলার ছাড়া URL সরবরাহ করতে ব্যবহৃত হয়। অ্যাপাচি- র সাথে লারাভেল পরিসেবা ব্যবহার করার জন্য, `mod_rewrite` মডিউলটি সক্ষম করতে ভুলবেন না, তাই` .htaccess` ফাইলটি সার্ভার দ্বারা প্রস্থাপিত হবে।

`.htaccess` যে ফাইলটি লারাভেল সঙ্গে প্রকল্প আপনার এ্যাপাচি ইন্সটলেশন এ কাজ করে না, এই বিকল্প চেষ্টা করুন:

    Options +FollowSymLinks
    RewriteEngine On
    
    RewriteCond %{HTTP:Authorization} .
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}] 
   
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]

#### Nginx

আপনি যদি Nginx ব্যবহার করেন ,আপনার সাইট কনফিগারেশনে নিম্নোক্ত নির্দেশিকা `index.php` ফ্রন্ট কন্ট্রোলারের সমস্ত আনুরুধ কে নির্দেশ করে:

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

অবশ্যই, [Homestead](/docs/{{version}}/homestead) বা [Valet](/docs/{{version}}/valet) ব্যবহার করার সময়, সুন্দর URL স্বয়ংক্রিয়ভাবে কনফিগার করা যায়।
