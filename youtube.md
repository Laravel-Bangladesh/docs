<a name="defining-middleware"></a>
##  মিডলওয়্যার তৈরিঃ 

একটি নতুন মিডলওয়্যার তৈরি করতে, `make: middleware` আর্টিজান কমান্ডটি ব্যবহার করুনঃ 

    php artisan make:middleware CheckAge

এই কমান্ডটি আপনার `app/Http/Middleware` ডিরেক্টরিতে একটি নতুন `CheckAge` ক্লাস তরী করবে। এই মিডলওয়্যার মধ্যে, যদি সরবরাহকৃত আগে `age` ২০০  এর চেয়ে বেশি হয় তা হলে আমরা শুধুমাত্র রাউট অ্যাক্সেস অনুমতি দেবে। অন্যথা, আমরা ব্যবহারকারীদের `home` URI এ পুনঃনির্দেশিত বা পাঠিয়ে দিব করব। 