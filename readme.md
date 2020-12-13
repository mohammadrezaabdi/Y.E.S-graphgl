# GraphQL

## Members
    ArshiA Akhavan          97110422
    Mohamadreza Abdi        97110285
    bahar khodabakhshian    97105906


# GraphQl
<p dir="rtl" style="position:right;">
GraphQl یک استاندارد تازه برای توصیف (Application Programing Interface) API می‌باشد که توسط facebook  توسعه و متن باز شده است.
</p>

<p dir="rtl" style="position:right;">
GraphQl به کلاینت این اجازه را می‌دهد که request خود را به صورت توصیفی بیان کند و دقیقا همان دیتایی را که نیاز دارد از سرور دریافت کند. این به این معنی است که دیگر لازم نیست GraphQl Server تعداد زیادی endpoint برای انواع مختلف request داشته باشد و می‌تواند تمام دیتا ها رو در یک enpoint سرو کند
</p>

# GraphQl vs REST
<p dir="rtl" style="position:right;">
۴ دلیل برای این که از   GraphQl استفاده کنیم:
<ol dir="rtl" style="position:right;">
<li>شمایی با ساختار قوی</li>
<li>رفع مشکل overfetching و underfetching</li>
<li>مجتمع کردن تمام endpoint ها در یک endpoint</li>
<li>دسترس بالای front-end developer ها و سرعت بالا development</li>
</ol>
<p dir="rtl" style="position:right;">
به مثال زیر توجه کنید:

<p dir="rtl" style="position:right;">
فرض کنید می‌خواهیم یک صفحه بلاگ دولوپ کنیم.
در این صفحه در بالای آن نام کاربر، سپس در پایین آن ۳ پست اخیر و در آخر هم ۳ تن از دوستان آن کاربر را نشان می‌دهیم:

    Marry

    posts:    
    * love GraphQl
    * YES   is   BEST
    * death with REST
    
    friends:
    ArshiA,Bahar,Mohamad Reza

<p dir="rtl" style="position:right;">
حال می‌خواهیم back-end این API را یک بار با REST و یک بار با GraphQl پیاده سازی کنیم.

### REST
<p dir="rtl" style="position:right;">
برای این API نیاز به سه endpoint داریم
<ol dir="rtl" style="position:right;">
<li>/users/#id/ برای گرفتن نام کاربر</li>
<li>/users/#id/posts برای گرفتن ۳ پست آخر</li>
<li>/users/#id/followers برای گرفتن ۳ دوست اخیر کاربر</li>
</ol>

<p dir="rtl" style="position:right;">
حال می‌خواهیم تک‌تک API ها را با بررسی کنیم:

+ /user/#id/
<p dir="rtl" style="position:right;">
این endpoint  با گرفتن id کاربر مورد نظر، اطلاعات آن را باز می‌گرداند. این اطلاعات می‌تواند شامل name, username, email, password hash, birthday, bio , ... باشد در حالی که ما تنها به نام کاربر نیاز داریم.

+ /user/#id/posts
<p dir="rtl" style="position:right;">
این endpoint با گرفتن id کاربر، تمامی پست‌های آن کاربر را به صورت کامل برمی‌گرداند.
این درحالیست که ما تنها نیاز به سه پست اخیر و از بین آن ها فقط نیاز به متن آنها داریم(برای مثال می‌دانیم که نویسنده‌ی آن پست ها کیست!)

+ /user/#id/followers
<p dir="rtl" style="position:right;">
این endpoint با گرفتن id کاربر، لیستی از تمامی کاربرانی که با او دوست هستند را برمی‌گرداند و این درحالی است که ما فقط ۳ کاربر اخیر و از آن‌ها فقط نام آن‌ها را نیاز داریم.

<p dir="rtl" style="position:right;">
همان طور که متوجه شدید REST API در هر endpoint مقدار زیادی اطلاعات را برمی‌گرداند که شاید ما به همه آن ها نیاز نداشته باشیم (overfetching) و در دنیاي مدرن امروزی که کاربران با اینترنت همراه به دنیای مجازی متصل می‌شوند این مسئله‌ی مهمی به شمار می‌آید زیرا باعث هدر رفتن ترافیک کاربران شده و از طرفی باعث می‌شود تا صفحه‌ی ما دیرتر load شود و از محبوبیت ‌آن کاسته شود

<p dir="rtl" style="position:right;">
ولی به هر روی صفحه‌ی ما load شد!

### GraphQl
<p dir="rtl" style="position:right;">
برای پیاده سازی API بالا در GraphQl تنها به یک enpoint نیاز دارید و می‌توانید دقیقا data‌ی مورد نیاز خود را (نه بیشتر نه کمتر) با query ای شبیه به query زیر دریافت کنید!

```graphql
{
    user(id: "1111111") {
       name
       Posts (last: "3"){
           text
       }
       followers(last: "3"){
           name
       }
    }
}
``` 
<p dir="rtl" style="position:right;">
همان طور که مشاهده می‌کنید برای پیاده سازی صفحه با استفاده از GraphQl تنها نیاز به باز کردن **یک** connection به سرور است و هیچ دیتای اضافه‌تری دریافت نمی‌شود!
