## <p dir="rtl" style="position:right;">فرانت GraphQL با استفاده از React + Apollo </p>  



<p dir="rtl" style="position:right;">
در این قسمت می‌خواهیم برنامه‌ای برای سمت کلاینت graphQL پیاده‌سازی کنیم که در این قسمت کاربر می‌تواند query مورد نظر خود را به سمت سرور بفرستد و جواب مورد انتظار خودش را دریافت کند.
</p>
 <p dir="rtl" style="position:right;">
این پروژه در واقع پیاده‌سازی ‌ای برای سایت hackernews است که شامل تعدادی لینک است که کاربران این سایت می‌توانند لینک جدید اضافه کنند و یا اینکه به لینک‌های موجود رای دهند و یا رای خود را بازگردانند. 
</p>

<p dir="rtl" style="position:right;">
در کل، عمل‌هایی که در این پروژه انجام می‌شوند عبارتند از : <br />
- نمایش لیستی از لینک‌ها <br />
- امکان جست و جو در لیست لینک‌ها <br />
- احراز هویت کاربر <br />
- به کاربرانی که احراز هویت شده‌اند اجازه می‌دهد که لینک‌های جدید ایجاد کنند. <br />
- به کاربرانی که احراز هویت شده اند، اجازه‌ی رای دادن به هر کدام از لینک‌ها یا برداشتن رای خود از آن‌ها را می‌دهد. <br />
- اگر کاربران تغییری در لیست لینک‌ها دهند، سریع به روز رسانی می‌شود. 
</p>

<p  dir="rtl" style="position:right;">
برای پیاده‌سازی این پروژه برای  frontend و backend از تکنولوژی‌های زیر استفاده شده‌است. <br />
- فرانت : React و Apollo Client 3.2<br />
- بک : Apollo Server 2.18 و Prisma
</p>

## <p  dir="rtl" style="position:right;"> کلاینت GraphQL</p>

<p dir="rtl" style="position:right;">
در سمت کلاینت برای انجام دادن کار‌های تکراری مانند فرستادن query و غیره از GraphQL client استفاده می‌کنیم که از کار‌های تکراری جلوگیری می‌کند و تنها با یک بار صدا زدن API می‌تواند اطلاعات موردنیاز خود را از سمت سرور دریافت کند. <br />
برای پیاده‌سازی کلاینت GraphQL کتابخانه‌های متفاوتی همانند Apollo، Relay و urql وجود دارد که هر کدام از این کتاب‌خانه‌ها خوبی‌ها و بدی‌هایی دارند و میزان کنترل هر یک از آن‌ها روی روند عملیا‌ت‌های در حال انجام GraphQL متفاوت است. <br />
در پیاده‌سازی پروژه‌ي ذکر شده از Apollo Client استفاده می‌کنیم. <br />
</p>

### <p  dir="rtl" style="position:right;"> توضیح قسمت frontend پروژه</p>

<p dir="rtl" style="position:right;">
کل فایل‌های مربوط به بخش front سایت در پوشه‌ی front قرار دارد.<br />
در پوشه‌ی src پوشه‌ی دیگری به نام components وجود دارد که بخش‌های مختلف برای ساختن front در آن پیاده‌سازی شده‌اند.<br/>
حال به توضیح هر کدام از آن‌ها می‌پردازیم.
</P>

<p dir="rtl" style="position:right;">
در پوشه‌ی components فایلی به نام Link.js وجود دارد که مربوط به هر کدام از لینک‌های موجود در برنامه است. این component به صورت زیر پیاده‌سازی می‌شود.
</p>

```graphql
import React from 'react';

const Link = (props) => {
  const { link } = props;
  return (
    <div>
      <div>
        {link.description} ({link.url})
      </div>
    </div>
  );
};

export default Link;
```
<p dir="rtl" style="position:right;">
فایل دیگری به نام LinkList.js وجود دارد که در آن لینک‌هایی که در صفحه‌ نمایش داده‌ می‌شوند را پیاده‌سازی می‌کنیم. پیاده‌سازی این component به صورت زیر است .
</p>

```graphql
import React from 'react';
import Link from './Link';

const LinkList = () => {
  const linksToRender = [
    {
      id: '1',
      description:
        'Prisma gives you a powerful database toolkit 😎',
      url: 'https://prisma.io'
    },
    {
      id: '2',
      description: 'The best GraphQL client',
      url: 'https://www.apollographql.com/docs/react/'
    }
  ];

  return (
    <div>
      {linksToRender.map((link) => (
        <Link key={link.id} link={link} />
      ))}
    </div>
  );
};

export default LinkList;
```


<p dir="rtl" style="position:right;">
از linked list پیاده‌سازی شده در App.js استفاده می‌کنیم که component اصلی ما است تا لینک‌ها در صفحه نمایش داده شوند.
</p>

<p dir="rtl" style="position:right;">
حال به توضیح نحوه‌ی استفاده از query‌های GraphQL می‌پردازیم و اینکه چگونه از آن‌ها در برنامه استفاده کنیم.
برای استفاده از query مورد نظر خود ابتدا باید آن را تعریف کنیم. برای مثال اگر بخواهیم یک query به نام feed تعریف کنیم داریم :
</p>

```graphql
{
  feed {
    id
    links {
      id
      createdAt
      description
      url
    }
  }
}
```
<p dir="rtl" style="position:right;">
اکنون باید ببینیم که چگونه query‌های خود را در برنامه و با استفاده از javascript پیاده‌سازی کنیم. برای اینکار از کتاب‌خانه‌ی apollo/client استفاده می‌کنیم و از توابع آن در پیاده‌سازی query‌ها استفاده می‌کنیم. برای تعریف query از تابع useQuery و کتاب‌خانه‌ی gql در apollo/client استفاده می‌کنیم. <br />
برای مثال می‌خواهیم در فایل linkedlist.js یک query به نام FEED_QUERY تعریف کنیم. با استفاده از gql داریم :
</p>

```
import { useQuery, gql } from '@apollo/client';

const FEED_QUERY = gql`
  {
    feed {
      id
      links {
        id
        createdAt
        url
        description
      }
    }
  }
`;
```
