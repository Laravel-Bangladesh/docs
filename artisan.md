# Artisan Console

- [Introduction](#introduction)
- [Writing Commands](#writing-commands)
    - [Generating Commands](#generating-commands)
    - [Command Structure](#command-structure)
    - [Closure Commands](#closure-commands)
- [Defining Input Expectations](#defining-input-expectations)
    - [Arguments](#arguments)
    - [Options](#options)
    - [Input Arrays](#input-arrays)
    - [Input Descriptions](#input-descriptions)
- [Command I/O](#command-io)
    - [Retrieving Input](#retrieving-input)
    - [Prompting For Input](#prompting-for-input)
    - [Writing Output](#writing-output)
- [Registering Commands](#registering-commands)
- [Programmatically Executing Commands](#programmatically-executing-commands)
    - [Calling Commands From Other Commands](#calling-commands-from-other-commands)

<a name="introduction"></a>
## Introduction

Artisan কমান্ড লাইন ইন্টারফেস যা Laravel এর সঙ্গে অন্তর্ভুক্ত করা হয়। এটি বেশ কয়েকটি সহায়ক কমান্ড সরবরাহ করে যা আপনার অ্যাপ্লিকেশন তৈরি করার সময় আপনাকে সহায়তা করতে পারে। Artisan এর সমস্ত কমান্ডের একটি তালিকা দেখতে, আপনি `list` কমান্ডটি ব্যবহার করতে পারেন:

    php artisan list

প্রতিটি কমান্ডটিতে একটি "help" পর্দা রয়েছে যা কমান্ডের প্রাপ্য আর্গুমেন্ট এবং বিকল্পগুলি প্রদর্শন করে এবং বর্ণনা করে। একটি হেল্প স্ক্রীন দেখতে, এর সাথে কমান্ডের নাম আগে `help`:

    php artisan help migrate

#### Laravel REPL

সমস্ত লারভেল অ্যাপ্লিকেশনগুলিতে Tinker, [PsySH](https://github.com/bobthecow/psysh) প্যাকেজ দ্বারা চালিত একটি REPL অন্তর্ভুক্ত। Tinker আপনাকে এলকোভেন্ট ORM, জব, ইভেন্ট এবং আরও অনেক কিছু সহ কমান্ড লাইনে আপনার সমগ্র Laravel অ্যাপ্লিকেশনটির সাথে ইন্টারঅ্যাক্ট করার অনুমতি দেয়। Tinker পরিবেশ প্রবেশ করতে, চালান `tinker` Artisan কমান্ড:

    php artisan tinker

<a name="writing-commands"></a>
## Writing Commands

আর্টিসান দ্বারা সরবরাহিত কমান্ড ছাড়াও, আপনি নিজের নিজস্ব কাস্টম কমান্ডগুলিও তৈরি করতে পারেন। কমান্ড সাধারণত সংরক্ষণ করা হয় `app/Console/Commands` ডিরেক্টরিতে; যাইহোক, আপনি যতক্ষণ আপনার কমান্ডগুলি কম্পোজার দ্বারা লোড করা যেতে পারে ততক্ষণ আপনি নিজের সঞ্চয়স্থান নির্বাচন করতে পারবেন।

<a name="generating-commands"></a>
### Generating Commands

নতুন কমান্ড তৈরি করতে, ব্যবহার করুন `make:command` Artisan কমান্ড। এই কমান্ডটি একটি নতুন কমান্ড ক্লাস তৈরি করবে `app/Console/Commands` ডিরেক্টরিতে। এই ডিরেক্টরিটি আপনার অ্যাপ্লিকেশানে উপস্থিত না থাকলে চিন্তা করবেন না, কারণ এটি প্রথমবারে চালানো হবে `make:command` Artisan command. উত্পন্ন কমান্ডটি সমস্ত কমান্ডের উপস্থিত থাকা বৈশিষ্ট্য এবং পদ্ধতিগুলির ডিফল্ট সেট অন্তর্ভুক্ত করবে:

    php artisan make:command SendEmails

<a name="command-structure"></a>
### Command Structure

আপনার কমান্ড তৈরি করার পরে, আপনি পূরণ করা উচিত `signature` এবং `description`  ক্লাস বৈশিষ্ট্যগুলো, যা আপনার কমান্ড প্রদর্শন করার সময় ব্যবহার করা হবে `list` ্ক্রীন এ. `handle` আপনার কমান্ড কার্যকর করার সময় কল করা হবে। আপনি এই পদ্ধতিতে আপনার কমান্ড লজিক স্থাপন করতে পারেন।

> {tip} বৃহত্তর কোড পুনঃব্যবহারের জন্য, আপনার কনসোল কমান্ড হালকা রাখা ভাল অনুশীলন এবং তাদের কর্ম সম্পাদন করার জন্য অ্যাপ্লিকেশন সেবা আলাদা করা । নিচের উদাহরনে, উল্লেখ্য যে আমরা একটি সার্ভিস ক্লাস ইনজেকশন করেছি "heavy lifting" ইমেইল পাঠানোর।

চলুন একটি উদাহরণ কমান্ড দেখুন. উল্লেখ্য যে আমরা কমান্ডের কন্সট্রকটারে আমাদের যেকোনো নির্ভরতাগুলি ইনজেক্ট করতে সক্ষম। Laravel এর [service container](/docs/{{version}}/container) স্বয়ংক্রিয়ভাবে নির্মাতার টাইপ-ইঙ্গিত সমস্ত নির্ভরতা ইনজেকশন করবে:

    <?php

    namespace App\Console\Commands;

    use App\User;
    use App\DripEmailer;
    use Illuminate\Console\Command;

    class SendEmails extends Command
    {
        /**
         * The name and signature of the console command.
         *
         * @var string
         */
        protected $signature = 'email:send {user}';

        /**
         * The console command description.
         *
         * @var string
         */
        protected $description = 'Send drip e-mails to a user';

        /**
         * The drip e-mail service.
         *
         * @var DripEmailer
         */
        protected $drip;

        /**
         * Create a new command instance.
         *
         * @param  DripEmailer  $drip
         * @return void
         */
        public function __construct(DripEmailer $drip)
        {
            parent::__construct();

            $this->drip = $drip;
        }

        /**
         * Execute the console command.
         *
         * @return mixed
         */
        public function handle()
        {
            $this->drip->send(User::find($this->argument('user')));
        }
    }

<a name="closure-commands"></a>
### Closure Commands

Closure ভিত্তিক কমান্ড শ্রেণী হিসাবে কনসোল কমান্ড সংজ্ঞায়িত করার জন্য একটি বিকল্প প্রদান করে। একই পথে যে রুট Closures কন্ট্রোলার একটি বিকল্প, কমান্ড ক্লাস বিকল্প হিসাবে কমান্ড Closure. আনার `commands` মেথড `app/Console/Kernel.php` ফাইল, Laravel `routes/console.php` file লোড করে:

    /**
     * Register the Closure based commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        require base_path('routes/console.php');
    }

যদিও এই ফাইল HTTP রুট সংজ্ঞায়িত করে না, এটি আপনার অ্যাপ্লিকেশনে কনসোল ভিত্তিক এন্ট্রি পয়েন্ট (রুট) সংজ্ঞায়িত করে। এই ফাইলের মধ্যে, আপনি  আপনার সমস্ত ক্লোজার ভিত্তিক ব্যবহার করে রুট সংজ্ঞায়িত করতে পারেন `Artisan::command` মেথড গুলো. এই `command` মেথড দুটি আর্গুমেন্ট গ্রহণ করে:  [command signature](#defining-input-expectations) এবং একটি ক্লোজার যা কমান্ড আর্গুমেন্ট এবং বিকল্পগুলি গ্রহণ করে:

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    });

ক্লোজার অন্তর্নিহিত কমান্ড ইনস্ট্যান্সের সাথে আবদ্ধ, তাই আপনি সম্পূর্ণ সহায়তার ক্লাসে আপনি অ্যাক্সেস করতে সক্ষম এবং হেলপারের সমস্ত পদ্ধতিতে সম্পূর্ণ অ্যাক্সেস পাবেন।

#### Type-Hinting Dependencies
আপনার কমান্ডের আর্গুমেন্ট এবং বিকল্পগুলি পাওয়ার পাশাপাশি কমান্ড ক্লোজারগুলি অতিরিক্ত নির্ভরতাগুলি টাইপ-ইন্টিন্ট করতে পারে যা আপনি সমাধান করতে চান
[service container](/docs/{{version}}/container):

    use App\User;
    use App\DripEmailer;

    Artisan::command('email:send {user}', function (DripEmailer $drip, $user) {
        $drip->send(User::find($user));
    });

#### Closure Command Descriptions

Closure ভিত্তিক কমান্ড সংজ্ঞায়িত করার সময়, আপনি ব্যবহার করতে পারেন `describe` মেথড, command এর বিবরন লিখতে. `php artisan list` or `php artisan help` command গুলো চালানোর সময় বিবরন গুলো দেখতে পারবেন। 

Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    })->describe('Build the project');

<a name="defining-input-expectations"></a>
## Defining Input Expectations

console commands লিখার সময়, arguments অথবা options এর মাধ্যমে ব্যবহারকারী থেকে মাধ্যমে ইনপুট সংগ্রহ করা সাধারণ পদ্ধতি. Laravel এটা খুব সুবিধাজনক করে তোলে ব্যাবহার কারী থেকে আশানরুপ ইনপুট খুজতে পারেন `signature` প্রপার্টি আনার command এ ্যাবহার করে . এই `signature` প্রপার্টি আপনাকে নাম সনাক্ত করনে বা সংজ্ঞায় সাহায্য করতে পারে আর্গুমেন্ট,অশন, একক, প্রকাশক, রুট মত সিনট্যাক্স কমান্ডের জন্য।

<a name="arguments"></a>
### Arguments

All user supplied arguments and options are wrapped in curly braces. In the following example, the command defines one **required** argument: `user`:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user}';

You may also make arguments optional and define default values for arguments:

    // Optional argument...
    email:send {user?}

    // Optional argument with default value...
    email:send {user=foo}

<a name="options"></a>
### Options

Options, like arguments, are another form of user input. Options are prefixed by two hyphens (`--`) when they are specified on the command line. There are two types of options: those that receive a value and those that don't. Options that don't receive a value serve as a boolean "switch". Let's take a look at an example of this type of option:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue}';

In this example, the `--queue` switch may be specified when calling the Artisan command. If the `--queue` switch is passed, the value of the option will be `true`. Otherwise, the value will be `false`:

    php artisan email:send 1 --queue

<a name="options-with-values"></a>
#### Options With Values

Next, let's take a look at an option that expects a value. If the user must specify a value for an option, suffix the option name with a `=` sign:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue=}';

In this example, the user may pass a value for the option like so:

    php artisan email:send 1 --queue=default

You may assign default values to options by specifying the default value after the option name. If no option value is passed by the user, the default value will be used:

    email:send {user} {--queue=default}

<a name="option-shortcuts"></a>
#### Option Shortcuts

To assign a shortcut when defining an option, you may specify it before the option name and use a | delimiter to separate the shortcut from the full option name:

    email:send {user} {--Q|queue}

<a name="input-arrays"></a>
### Input Arrays

If you would like to define arguments or options to expect array inputs, you may use the `*` character. First, let's take a look at an example that specifies an array argument:

    email:send {user*}

When calling this method, the `user` arguments may be passed in order to the command line. For example, the following command will set the value of `user` to `['foo', 'bar']`:

    php artisan email:send foo bar

When defining an option that expects an array input, each option value passed to the command should be prefixed with the option name:

    email:send {user} {--id=*}

    php artisan email:send --id=1 --id=2

<a name="input-descriptions"></a>
### Input Descriptions

You may assign descriptions to input arguments and options by separating the parameter from the description using a colon. If you need a little extra room to define your command, feel free to spread the definition across multiple lines:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send
                            {user : The ID of the user}
                            {--queue= : Whether the job should be queued}';

<a name="command-io"></a>
## Command I/O

<a name="retrieving-input"></a>
### Retrieving Input

While your command is executing, you will obviously need to access the values for the arguments and options accepted by your command. To do so, you may use the `argument` and `option` methods:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $userId = $this->argument('user');

        //
    }

If you need to retrieve all of the arguments as an `array`, call the `arguments` method:

    $arguments = $this->arguments();

Options may be retrieved just as easily as arguments using the `option` method. To retrieve all of the options as an array, call the `options` method:

    // Retrieve a specific option...
    $queueName = $this->option('queue');

    // Retrieve all options...
    $options = $this->options();

If the argument or option does not exist, `null` will be returned.

<a name="prompting-for-input"></a>
### Prompting For Input

In addition to displaying output, you may also ask the user to provide input during the execution of your command. The `ask` method will prompt the user with the given question, accept their input, and then return the user's input back to your command:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $name = $this->ask('What is your name?');
    }

The `secret` method is similar to `ask`, but the user's input will not be visible to them as they type in the console. This method is useful when asking for sensitive information such as a password:

    $password = $this->secret('What is the password?');

#### Asking For Confirmation

If you need to ask the user for a simple confirmation, you may use the `confirm` method. By default, this method will return `false`. However, if the user enters `y` or `yes` in response to the prompt, the method will return `true`.

    if ($this->confirm('Do you wish to continue?')) {
        //
    }

#### Auto-Completion

The `anticipate` method can be used to provide auto-completion for possible choices. The user can still choose any answer, regardless of the auto-completion hints:

    $name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);

#### Multiple Choice Questions

If you need to give the user a predefined set of choices, you may use the `choice` method. You may set the array index of the default value to be returned if no option is chosen:

    $name = $this->choice('What is your name?', ['Taylor', 'Dayle'], $defaultIndex);

<a name="writing-output"></a>
### Writing Output

To send output to the console, use the `line`, `info`, `comment`, `question` and `error` methods. Each of these methods will use appropriate ANSI colors for their purpose. For example, let's display some general information to the user. Typically, the `info` method will display in the console as green text:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->info('Display this on the screen');
    }

To display an error message, use the `error` method. Error message text is typically displayed in red:

    $this->error('Something went wrong!');

If you would like to display plain, uncolored console output, use the `line` method:

    $this->line('Display this on the screen');

#### Table Layouts

The `table` method makes it easy to correctly format multiple rows / columns of data. Just pass in the headers and rows to the method. The width and height will be dynamically calculated based on the given data:

    $headers = ['Name', 'Email'];

    $users = App\User::all(['name', 'email'])->toArray();

    $this->table($headers, $users);

#### Progress Bars

For long running tasks, it could be helpful to show a progress indicator. Using the output object, we can start, advance and stop the Progress Bar. First, define the total number of steps the process will iterate through. Then, advance the Progress Bar after processing each item:

    $users = App\User::all();

    $bar = $this->output->createProgressBar(count($users));

    $bar->start(); 

    foreach ($users as $user) {
        $this->performTask($user);

        $bar->advance();
    }

    $bar->finish();

For more advanced options, check out the [Symfony Progress Bar component documentation](https://symfony.com/doc/2.7/components/console/helpers/progressbar.html).

<a name="registering-commands"></a>
## Registering Commands

Because of the `load` method call in your console kernel's `commands` method, all commands within the `app/Console/Commands` directory will automatically be registered with Artisan. In fact, you are free to make additional calls to the `load` method to scan other directories for Artisan commands:

    /**
     * Register the commands for the application.
     *
     * @return void
     */
    protected function commands()
    {
        $this->load(__DIR__.'/Commands');
        $this->load(__DIR__.'/MoreCommands');

        // ...
    }

You may also manually register commands by adding its class name to the `$commands` property of your `app/Console/Kernel.php` file. When Artisan boots, all the commands listed in this property will be resolved by the [service container](/docs/{{version}}/container) and registered with Artisan:

    protected $commands = [
        Commands\SendEmails::class
    ];

<a name="programmatically-executing-commands"></a>
## Programmatically Executing Commands

Sometimes you may wish to execute an Artisan command outside of the CLI. For example, you may wish to fire an Artisan command from a route or controller. You may use the `call` method on the `Artisan` facade to accomplish this. The `call` method accepts the name of the command as the first argument, and an array of command parameters as the second argument. The exit code will be returned:

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

Using the `queue` method on the `Artisan` facade, you may even queue Artisan commands so they are processed in the background by your [queue workers](/docs/{{version}}/queues). Before using this method, make sure you have configured your queue and are running a queue listener:

    Route::get('/foo', function () {
        Artisan::queue('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

You may also specify the connection or queue the Artisan command should be dispatched to:

    Artisan::queue('email:send', [
        'user' => 1, '--queue' => 'default'
    ])->onConnection('redis')->onQueue('commands');

#### Passing Array Values

If your command defines an option that accepts an array, you may pass an array of values to that option:

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--id' => [5, 13]
        ]);
    });

#### Passing Boolean Values

If you need to specify the value of an option that does not accept string values, such as the `--force` flag on the `migrate:refresh` command, you should pass `true` or `false`:

    $exitCode = Artisan::call('migrate:refresh', [
        '--force' => true,
    ]);

<a name="calling-commands-from-other-commands"></a>
### Calling Commands From Other Commands

Sometimes you may wish to call other commands from an existing Artisan command. You may do so using the `call` method. This `call` method accepts the command name and an array of command parameters:

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    }

If you would like to call another console command and suppress all of its output, you may use the `callSilent` method. The `callSilent` method has the same signature as the `call` method:

    $this->callSilent('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);
