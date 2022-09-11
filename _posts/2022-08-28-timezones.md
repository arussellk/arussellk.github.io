---
layout: post
title: Time handling for purchase transactions
---

Advice commonly found on the internet for how to program with timezones is some
version of:
1. Only store UTC datetimes in your database. (e.g.,
   `2022-01-08T10:45:00.000Z`.)
2. Convert the UTC datetime to the user's current timezone for display. (e.g.,
   `2022-01-08T10:45:00.000Z -> 2022-01-08T03:45:00-07:00`.)

This advice is sufficient for keeping track of a a single point in time (e.g.,
when does the meeting start), but it is not an exhaustive list of advice. The
purpose of this post is to demonstrate an example of when storing just a UTC
datetime is not sufficient and give a useful analogy for how to consider dates
in your application in the future.

I worked on a system that showed users their bank account's transaction
history. Users could grant access to their transaction history via OAuth,
allowing our system to read their transactions and display them in a nice way.
One day I received a bug report that the transaction date shown in our system
did not match the date the user expected. I looked at the code responsible for
getting the date from the bank's API and found that the previous author did not
consider timezones in their implementation. The existing code looked like this:

```
const txnDate = txn.utcDatetime.slice(0, 'YYYY-MM-DD'.length);
```

This code worked for some cases:
```
moment('2022-01-02T22:30:00.000-01')
  .toISOString() // '2022-01-02T23:30:00.000Z'
  .slice(0, 'YYYY-MM-DD'.length) // '2022-01-02'
```

... but not for other cases:
```
moment('2022-01-02T02:30:00.000+06').toISOString()
'2022-01-01T20:30:00.000Z'
moment('2022-01-02T22:30:00.000+06').toISOString()
'2022-01-02T16:30:00.000Z'
```

Clearly, the first ten characters of an ISO datetime string is not always the
same as the transaction date.
Following the common advice I listed above, the bank's API already did the
first part by giving me a UTC datetime. Now I just need to convert the UTC
datetime to the user's timezone. But what does "the user's current timezone"
mean for a transaction?

  When a user views a list of purchases,
  they expect to see the date they were experiencing in the location they
  were when they made the transaction.
  e.g., If I am in the United States and purchase something from an Australian
  company at `2022-09-06T22:30:00-06`, I say that the txn date is
  `2022-09-06` even though it is `2022-09-07` in Sydney
  (`2022-09-06T22:30:00-06 == 2022-09-07T14:30:00+10:00`).
  Further, even if I travel to Sydney next week, I still say that I
  purchased the thing on `2022-09-06` because it was `2022-09-06` for me when I
  purchased it. (i.e., While looking at my purchase history from a new
  location, I do not convert purchase timestamps to the timezone I am currently
  standing in.)
  So, we actually need _per-transaction_ timezone information instead of
  _per-user_ timezone information. Fortunately, 

The transaction model looked something like this:

```
{
  id: uint,
  description: string,
  time: string, // ISO 8601
  // country: string, // ISO 3166 alpha-2 // don't include yet
}
```

example
```
{
  id: 1,
  description: "Some Description",
  amount: { currency: "USD", minorUnits: 1067, },
  time: "2022-08-28T22:58:30Z",
  // country: "DE",
}
```

Now let's show this transaction to our user:
- $10.67 _Some Description_ on 29 Aug 2022

Oh no! What happened?

The front end code probably looks like this:
```
${txn.amount} ${txn.description} on ${moment(tx.time)}
```

Users expect a transaction date to be the date they experienced when the
payment occurred.

links
https://en.wikipedia.org/wiki/ISO_8601
https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2
https://developer.starlingbank.com/docs/aisp

https://momentjs.com/timezone/

---

Suppose you want to show something like this to a user:
- $10.67 _Some Description_ on 29 Aug 2022

And suppose you are getting the user's transactions from their bank's API:
```
{
  "time": "2022-08-31T23:01:00.000Z",
  ...
}
```

Is this enough information? No.

```
moment("2022-08-31T23:01:00.000Z").utcOffset('-01:00')
```

The same number can be written different ways:
```
4+0
3+1
6-2
```

And the same instant in time can be written different ways:
```
examples
moment('2010-10-20').isSame('2009-12-31', 'year');
```

Transaction Date and Transaction Instant.
Birthdate and Birthinstant.
Birthdates stay the same no matter what timezone you are in.

Imagine two babies born at the same time in two different locations:
Nandor in New York City, USA
Tom in Tokyo, Japan

On DATE New York City is in x (GMT-4)
Tokyo (GMT+9?)

Suppose both are born at the exact same time of DATE
Nandor's doctor will write down that they were born at DATE-04 and say their birthday is 03
Tom's doctor will write down that they were born at DATE+09 and say their birthday is 04

If these two birthinstants are reported to a central database that stores everything
in Z, we will have something like this:

```
Name   | Birthinstant (Z)
-------------------------
Nandor |
Tom    |
```

```
{
  "name": "Nandor",
  "birthinstant": "..."
}
```

From this API response, can we tell when we should celebrate Nandor's birthday?
What about Tom's birthday?
  No.

Oh no! Quick - add a column to store the birth country!
(This is what the neobank does.)

```
Name   | Birthinstant (Z) | Country
-----------------------------------
Nandor |                  | US
Tom    |                  | JP
```

```
{
  "name": "Nandor",
  "birthinstant": "...",
  "country": "US"
}
```

```
moment.tz.zonesForCountry('JP') // just one
moment.tz.zonesForCountry('US') // too many!

moment('...').tz('Asia/Tokyo').format('...')
```
