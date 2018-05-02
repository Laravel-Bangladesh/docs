# ইনস্টলেশন

- [ইনস্টলেশন](#installation)
    - [সার্ভার আবশ্যকতা](#server-requirements)
    - [লারাভেল ইন্সটল](#installing-laravel)
    - [কনফিগারেশন](#configuration)
- [ওয়েব সার্ভার কনফিগারেশন](#web-server-configuration)
    - [Pretty URLs](#pretty-urls)

<a name="installation"></a>
## ইনস্টলেশন

> {video}  লারাভেল  শিক্ষার্থী এর জন্য? Laracasts একটি প্রদান করে [বিনামূল্যে, লারাভেল এর সূচনা ](http://laravelfromscratch.com) ফ্রেমওয়ার্ক নতুনদের জন্য । এটি আপনার যাত্রা শুরু করার জন্য একটি ভালো জায়গা।

<a name="server-requirements"></a>
### সার্ভার আবশ্যকতা
সমস্ত প্রয়োজনীয়তা পূরণ করা হয় যদি Laravel ইনস্টল করা উচিত। আপনার জ্যাকল শুরু করার আগে নিশ্চিত করুন যে আপনার সিস্টেমে নিম্নোক্ত সফটওয়্যার/প্যাকেজ/লাইব্রেরি রয়েছে কিনা 

লারাভেল ফ্রেমওয়ার্ক কিছু সিস্টেম অবশকতা রয়েছে। অবশ্যই,  all of these requirements are satisfied by the [Laravel Homestead](/docs/{{version}}/homestead) virtual machine, so it's highly recommended that you use Homestead as your local Laravel development environment.

However, if you are not using Homestead, you will need to make sure your server meets the following requirements:
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
</div>

<a name="installing-laravel"></a>
### লারাভেল ইন্সটল
লারাভেল প্যাকেজ dependencie  ম্যানেজার হিসাবে [কম্পোজার](https://getcomposer.org) ব্যবহার করে। 
Laravel utilizes [Composer](https://getcomposer.org) to manage its dependencies. সুতরাং, Laravel ব্যবহার করার আগে নিশ্চিত করুন যে আপনি আপনার মেশিনে কম্পোজার ইনস্টল করা আছে কিনা ।

#### লারেজ ইনস্টলারের মাধ্যমে

প্রথমে, কম্পোজার ব্যবহার করে Laravel ইনস্টলার ডাউনলোড করুন:

    composer global require "laravel/installer"

Make sure to place composer's system-wide vendor bin directory in your `$PATH` so the laravel executable can be located by your system. This directory exists in different locations based on your operating system; however, some common locations include:

আপনার অপারেটিং সিস্টেমের উপর ভিত্তি করে এটি বিভিন্ন স্থানে বিদ্যমান; তবে, কিছু সাধারণ অবস্থানে রয়েছে:

<div class="content-list" markdown="1">
- macOS: `$HOME/.composer/vendor/bin`
- GNU / Linux Distributions: `$HOME/.config/composer/vendor/bin`
</div>

Once installed, the `laravel new` command will create a fresh Laravel installation in the directory you specify. For instance, `laravel new blog` will create a directory named `blog` containing a fresh Laravel installation with all of Laravel's dependencies already installed:

    laravel new blog

#### কম্পোজারের মাধ্যমে Create-Project

অন্যথা, আপনি কম্পোজারের `create-project` আপনার টার্মিনালে কমান্ড মাধ্যমে ল্যারেজ ইনস্টল করতে পারেন:

    composer create-project --prefer-dist laravel/laravel blog

#### Local Development Server

যদি আপনার পিএইচপি লোকাল মিশন এ ইনস্টল থাকেন এবং আপনি পিএইচপি এর বিল্ট ইন ডেভেলপমেন্ট সার্ভার ব্যবহার করতে চান আপনার অ্যাপ্লিকেশন এ, আপনি Artisan এর `serve` কমান্ট ব্যবহার করুন। এই কমান্ডটি একটি ডেভেলপমেন্ট সার্ভার শুরু করবে যা `http://localhost:8000`:

    php artisan serve


অবশ্যই, আরও শক্তিশালী স্থানীয় উন্নয়ন বিকল্পগুলি পাওয়া যাবে  [Homestead](/docs/{{version}}/homestead) এবং [Valet](/docs/{{version}}/valet) তে। 

<a name="configuration"></a>
### কনফিগারেশন

#### পাবলিক ডিরেক্টরি

লারাভেল ইনস্টল করার পর, আপনার ওয়েব সার্ভারের document / web root হিসাবে `public`  ডিরেক্টরি কে কনফিগার করা উচিত। এই ডিরেক্টরিতে `index.php` আপনার অ্যাপ্লিকেশনে প্রবেশের জন্য সমস্ত HTTP requests এর জন্য ফ্রন্ট কন্ট্রোলার হিসাবে কাজ করে।

#### কনফিগারেশন ফাইলগুলি

লারাভেল ফ্রেমওয়ার্কের জন্য সমস্ত কনফিগারেশন ফাইল `config` ডিরেক্টরির মধ্যে সংরক্ষণ করা হয়। প্রতিটি বিকল্প নথিভুক্ত করা হয়, তাই ফাইলগুলি দেখুন আপনার কাছে উপলব্ধ বিকল্পগুলির সাথে পরিচিত হন।

#### ডিরেক্টরি permissions

লারাভেল ইনস্টল করার পর,আপনি কিছু permissions কনফিগার করতে হতে পারে।  Directories within the `storage` and the `bootstrap/cache` directories should be writable by your web server or Laravel will not run. If you are using the [Homestead](/docs/{{version}}/homestead) virtual machine, these permissions should already be set.

#### Application Key

The next thing you should do after installing Laravel is set your application key to a random string. If you installed Laravel via Composer or the Laravel installer, this key has already been set for you by the `php artisan key:generate` command.

Typically, this string should be 32 characters long. The key can be set in the `.env` environment file. If you have not renamed the `.env.example` file to `.env`, you should do that now. **If the application key is not set, your user sessions and other encrypted data will not be secure!**

#### Additional Configuration

Laravel needs almost no other configuration out of the box. You are free to get started developing! However, you may wish to review the `config/app.php` file and its documentation. It contains several options such as `timezone` and `locale` that you may wish to change according to your application.

You may also want to configure a few additional components of Laravel, such as:

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

Laravel includes a `public/.htaccess` file that is used to provide URLs without the `index.php` front controller in the path. Before serving Laravel with Apache, be sure to enable the `mod_rewrite` module so the `.htaccess` file will be honored by the server.

If the `.htaccess` file that ships with Laravel does not work with your Apache installation, try this alternative:

    Options +FollowSymLinks
    RewriteEngine On

    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]

#### Nginx

If you are using Nginx, the following directive in your site configuration will direct all requests to the `index.php` front controller:

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

অবশ্যই, [Homestead](/docs/{{version}}/homestead) বা [Valet](/docs/{{version}}/valet) ব্যবহার করার সময়, সুন্দর ইউআরএল স্বয়ংক্রিয়ভাবে কনফিগার করা যায়।
