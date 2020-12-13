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

```js
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

```js
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

```js
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

<p dir="rtl" style="position:right;">
پس تا اینجا query را تعریف کرده‌ایم. برای استفاده از آن از تابع useQuery استفاده می‌کنیم و query تعریف شده را به آن پاس می‌دهیم. این تابع یک داده به ما باز می‌گرداند که اطلاعاتی را که در query تعریف کرده بودیم و میخواستیم از سرور دریافت کنیم شامل می‌شود. کد linkedlist.js به صورت زیر در می‌آید :
</p>

```js
const LinkList = () => {
  const { data } = useQuery(FEED_QUERY);

  return (
    <div>
      {data && (
        <>
          {data.feed.links.map((link) => (
            <Link key={link.id} link={link} />
          ))}
        </>
      )}
    </div>
  );
};
```

## <p dir="rtl" style="position:right;">توضیح gql و useQuery</p>
<p dir="rtl" style="position:right;">
Gql کتاب‌خانه‌ای است که برای تجزیه‌ی query تعریف شده برای GraphQL استفاده می‌شود. سپس query تعریف شده توسط gql را به تابع useQuery پاس می‌دهیم. چیزی که این تابع باز می‌گرداند از ۳ بخش زیر تشکیل شده‌است :<br />
- Loading : یک متغیر bool است که اگر مقدار آن true باشد، به این معناست که هنوز پاسخ از سمت سرور دریافت نشده و عملیات request دادن همچنان در حال انجام است. <br />
- Error : اگر در حین request دادن و یا دریافت داده اروری رخ دهد، توضیحات مربوط به این ارور و علت آن در این قسمت قرار می‌گیرد. <br />
- Data : این قسمت همان داده‌ی خواسته شده از سمت client است که از سرور دریافت شده است. در مثال بالا در فایل linkedlist.js این مقدار برابر با لیستی از لینک‌های موجود می‌باشد. این داده‌ها به همان شکل query تعریف شده دریافت می‌شود به این صورت که لینک‌ها در data.feed.links قرار دارند.
</p>

<p dir="rtl" style="position:right;">
همان‌طور که توضیح داده شد، در GraphQL دو عملیات مهم داریم : Query و Mutation <br />
در قسمت قبل نشان دادیم که چگونه به سرور query میفرستیم و  چطور می‌توانیم داده‌ها را از سرور بگیریم و آن را نمایش دهیم. حال می‌خواهیم ببینیم چگونه می‌توانیم با استفاده از Apollo client داده‌ها را تغییر دهیم و یا داده‌ی جدیدی اضافه کنیم. برای اینکار همانند قسمت قبل از gql استفاده می‌کنیم، اما به جای تابع useQuery از useMutation استفاده می‌کنیم. <br />
<br />
در پوشه‌ی src/components فایلی به نام CreateLink.js وجود دارد که مربوط به ساختن لینک جدید توسط کاربر در برنامه است . <br />
در ابتدا این فایل به شکل زیر است : <br />
</p>

```js
import React, { useState } from 'react';

const CreateLink = () => {
  const [formState, setFormState] = useState({
    description: '',
    url: ''
  });

  return (
    <div>
      <form
        onSubmit={(e) => {
          e.preventDefault();
        }}
      >
        <div className="flex flex-column mt3">
          <input
            className="mb2"
            value={formState.description}
            onChange={(e) =>
              setFormState({
                ...formState,
                description: e.target.value
              })
            }
            type="text"
            placeholder="A description for the link"
          />
          <input
            className="mb2"
            value={formState.url}
            onChange={(e) =>
              setFormState({
                ...formState,
                url: e.target.value
              })
            }
            type="text"
            placeholder="The URL for the link"
          />
        </div>
        <button type="submit">Submit</button>
      </form>
    </div>
  );
};

export default CreateLink;
```
<p dir="rtl" style="position:right;">
حال با استفاده از gql یک mutation به نام CREATE_LINK_MUTATION به صورت زیر تعریف می‌کنیم.
</p>

```js
import { useMutation, gql } from '@apollo/client';

const CREATE_LINK_MUTATION = gql`
  mutation PostMutation(
    $description: String!
    $url: String!
  ) {
    post(description: $description, url: $url) {
      id
      createdAt
      url
      description
    }
  }
`;
```
<p dir="rtl" style="position:right;">
با استفاده از تابع useMutation می‌خواهیم عوض کردن لینک‌ها را پیاده‌سازی کنیم. به این تابع mutation تعریف شده در قسمت قبل و متغیر‌هایی‌ که می‌خواهیم تغییر دهیم را پاس دهیم. این تابع یک function به ما برمی‌گرداند که هر گاه بخواهیم می‌توانیم برای عوض کردن لینک‌ها از آن استفاده کنیم. پس تکه کد زیر به CreateLink اضافه می‌شود.
</p>

```js
const CreateLink = () => {
  // ...
  const [createLink] = useMutation(CREATE_LINK_MUTATION, {
    variables: {
      description: formState.description,
      url: formState.url
    }
  });
  // ...
};
```
<p dir="rtl" style="position:right;">
در کد بالا، createLink همان تابعی است که می‌توانیم هر گاه بخواهیم آن را صدا کنیم. پس آن را به هنگام submitکردن فرم صدا می‌زنیم. 
</p>
```js
return (
  <div>
    <form
      onSubmit={(e) => {
        e.preventDefault();
        createLink();
      }}
    >
      ...
    </form>
  </div>
);
```
<p dir="rtl" style="position:right;">
حال می‌خواهیم به توضیح قسمت‌های مربوط به authentication همانند login بپردازیم. <br />
در پوشه‌ی src/components فایلی به نام login.js داریم که کد اولیه آن به صورت زیر است :
</p>

```js
import React, { useState } from 'react';
import { useHistory } from 'react-router';

const Login = () => {
  const history = useHistory();
  const [formState, setFormState] = useState({
    login: true,
    email: '',
    password: '',
    name: ''
  });

  return (
    <div>
      <h4 className="mv3">
        {formState.login ? 'Login' : 'Sign Up'}
      </h4>
      <div className="flex flex-column">
        {!formState.login && (
          <input
            value={formState.name}
            onChange={(e) =>
              setFormState({
                ...formState,
                name: e.target.value
              })
            }
            type="text"
            placeholder="Your name"
          />
        )}
        <input
          value={formState.email}
          onChange={(e) =>
            setFormState({
              ...formState,
              email: e.target.value
            })
          }
          type="text"
          placeholder="Your email address"
        />
        <input
          value={formState.password}
          onChange={(e) =>
            setFormState({
              ...formState,
              password: e.target.value
            })
          }
          type="password"
          placeholder="Choose a safe password"
        />
      </div>
      <div className="flex mt3">
        <button
          className="pointer mr2 button"
          onClick={() => console.log('onClick')}
        >
          {formState.login ? 'login' : 'create account'}
        </button>
        <button
          className="pointer button"
          onClick={(e) =>
            setFormState({
              ...formState,
              login: !formState.login
            })
          }
        >
          {formState.login
            ? 'need to create an account?'
            : 'already have an account?'}
        </button>
      </div>
    </div>
  );
};

export default Login;
```

<p dir="rtl" style="position:right;">
در کد بالا دو قسمت اصلی وجود دارد:<br />
-  یک قسمت برای کاربرانی که ثبت‌نام کرده اند و حساب کاربری دارند که برای این کاربران دو ورودی مربوط به ایمیل کاربر و رمز او نمایش داده‌ می‌شود. <br />
- یک قسمت برای کاربرانی که هنوز ثبت نام نکرده اند و اول باید در سایت ثبت نام کنند. برای این کاربران دو ورودی قبلی به همراه یک ورودی دیگر برای اسم کاربر نمایش داده می‌شود. <br />
<br />
وضعیت کاربر با توجه به متغیر formState.login مشخص می‌شود و با توجه به آن، یکی از دو قسمت توضیح داده‌شده در بالا نمایش داده می‌شود. <br />
<br />
برای هر کدام از login یا signup ، نیاز به تعریف و استفاده mutation‌های مربوط به هر یک داریم. این mutation‌ها را به صورت زیر تعریف می‌کنیم : <br />
</p>

```js
const SIGNUP_MUTATION = gql`
  mutation SignupMutation(
    $email: String!
    $password: String!
    $name: String!
  ) {
    signup(
      email: $email
      password: $password
      name: $name
    ) {
      token
    }
  }
`;

const LOGIN_MUTATION = gql`
  mutation LoginMutation(
    $email: String!
    $password: String!
  ) {
    login(email: $email, password: $password) {
      token
    }
  }
`;
```

<p dir="rtl" style="position:right;">
هر کدام از این mutation ها یک سری پارامتر‌هایی می‌گیرد و یک token باز می‌گردانند که از آن برای احراز هویت کاربر استفاده می‌کنیم. اکنون باید دو تابع login و signup با استفاده از تابع useMutation تعریف کنیم که آن‌ها را به هنگام کلیک کردن بر روی دکمه‌های login و signup صدا کنیم. پس داریم :
</p>

```js
const [login] = useMutation(LOGIN_MUTATION, {
  variables: {
    email: formState.email,
    password: formState.password
  },
  onCompleted: ({ login }) => {
    localStorage.setItem(AUTH_TOKEN, login.token);
    history.push('/');
  }
});

const [signup] = useMutation(SIGNUP_MUTATION, {
  variables: {
    name: formState.name,
    email: formState.email,
    password: formState.password
  },
  onCompleted: ({ signup }) => {
    localStorage.setItem(AUTH_TOKEN, signup.token);
    history.push('/');
  }
});
```


