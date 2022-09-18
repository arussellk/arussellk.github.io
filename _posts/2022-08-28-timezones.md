---
layout: post
title: Time handling for purchase transactions
---

Advice commonly found on the internet for how to program with timezones is some
version of:
1. Only store UTC datetimes in your database. (e.g.,
   `2022-01-08T10:45:00.000Z`.)
2. Convert the UTC datetime to the user's timezone for display. (e.g.,
   `2022-01-08T10:45:00.000Z -> 2022-01-08T03:45:00-07:00`.)

This advice is good for keeping track of a single point in time (e.g., when
does the meeting start), but sometimes working with timezones is slightly more
complicated.
The purpose of this post is to explain a situation that I encountered where it
took some extra effort to convert the UTC datetime from the database to the
appropriate timezone for display.

This post uses
[JavaScript](https://en.wikipedia.org/wiki/ECMAScript)
and
[Moment.js](https://momentjs.com/)
for code examples.
You can go to
[Moment.js Timezone](https://momentjs.com/timezone/)
and open your developer console to execute and experiment with the code
yourself.
Dates shown in this post follow the
[ISO 8601 standard](https://en.wikipedia.org/wiki/ISO_8601).

# Background

I worked on a system that showed users their bank account transaction
history. Users could grant access to their transaction history via OAuth,
allowing our system to read their transactions and display them in a nice way.
One day I received a bug report that the transaction date shown in our system
did not match the date the user expected. I looked at the code responsible for
getting the date from the bank's API and found that the previous author did not
consider timezones in their implementation. The existing code looked like this:

```
// txn is loaded from the API
const txn = {
  utcDateTime: '2022-04-30T23:01:00.000Z',
  // ...
};

const txnDate = txn.utcDateTime.slice(0, 'YYYY-MM-DD'.length);
console.log(txnDate); // '2022-04-30'
```

The bug report said that this date should actually be `2022-05-01`.
I checked which bank API this transaction came from, and it was a bank located
in Great Britain (GB). The problem is that the existing code does not consider
timezones, and the solution is to interpret the transaction's `utcDateTime` in
the timezone that the transaction took place.

# What went wrong?

Before we fix the problem, let's look at how this code worked well enough that
that previous author did not immediately notice the problem.
Most of the users of this system were located in the United States (US) and GB.
This means that as long as a transaction occurs _late_ enough in the day in GB
such that representing it in UTC still appears in the same date, the simple
`.slice` works (i.e., after 1am local time while GB is using British Summer
Time).
On the other side of things, as long as a transaction occurs _early_ enough in
the day in the US such that representing it in UTC still appears in the same
date, the simple `.slice` works (i.e., before 8pm or so local time on the East
Coast).
When a person sees a list of transactions and corresponding dates, they expect
to see the date they were experiencing when the payment occurred, not the date
of that instant represented in UTC, so it is necessary to interpret a
transaction's `utcDateTime` in the right timezone prior to display for a user.

Examples when `.slice` is correct and not correct:

| Date                          | Date in UTC              | `.slice(utc)` | `.slice` correct? |
| ----------------------------- | ------------------------ | ------------- | ----------------- |
| 2022-04-30T12:34:56.789-04:00 | 2022-04-30T16:34:56.789Z | 2022-04-30    | yes               |
| 2022-04-30T20:34:56.789-04:00 | 2022-05-01T00:34:56.789Z | 2022-05-01    | no                |
| 2022-05-01T00:01:00.000+01:00 | 2022-04-30T23:01:00.000Z | 2022-04-30    | no                |
| 2022-05-01T01:01:00.000+01:00 | 2022-05-01T00:01:00.000Z | 2022-05-01    | yes               |

# What is the solution?

I have shown that the first ten characters of a UTC datetime string does not
always match the transaction date a user expects to see.
To always interpret a transaction's utcDateTime in the correct zone, I need to
know in what timezone the transaction happened.
Fortunately, the bank API I was reading from included a country code for each
transaction, allowing me to fix the reported bug for the user in GB:

```
// txn is loaded from the API
const txn = {
  utcDateTime: '2022-04-30T23:01:00.000Z',
  country: 'GB',
  // ...
};

const zones = moment.tz.zonesForCountry('GB'); // ['Europe/London']
const zone = zones[0];

const txnDate = moment(txn.utcDateTime).tz(zone).format('YYYY-MM-DD');
console.log(txnDate); // 2022-05-01
```

Knowing the country where a transaction took place is enough information to
unambiguously find the transaction date for countries which only have one
timezone.
Since the bank that sent this data response is based in GB,
including just the country in the
API response is sufficient for many of their users.
However, the code above is incomplete for countries which have more than one
timezone (e.g., the United States).

# What about transactions in the US?

Suppose someone makes a purchase late one evening in Denver, CO, USA.

```
// txn is loaded from the API
const txn = {
  utcDateTime: '2022-05-01T05:34:56.789Z', // same as 2022-04-30T23:34:56-06:00
  country: 'US',
  // ...
};

const zones = moment.tz.zonesForCountry('US'); // (29) ['America/Adak', ..., 'Pacific/Honolulu']
const zone = zones[0]; // This is not always correct. :(

moment.tz.zonesForCountry('US').map(z => {
    const m = moment('2022-05-01T05:34:56.789Z').tz(z);
    return `${m.format('YYYY-MM-DD')} in ${z} (${m.format('Z z')})`;
});

// [
//   '2022-04-30 in America/Adak (-09:00 HDT)',
//   '2022-04-30 in America/Anchorage (-08:00 AKDT)',
//   '2022-04-30 in America/Boise (-06:00 MDT)',
//   '2022-05-01 in America/Chicago (-05:00 CDT)',
//   '2022-04-30 in America/Denver (-06:00 MDT)',
//   '2022-05-01 in America/Detroit (-04:00 EDT)',
//   '2022-05-01 in America/Indiana/Indianapolis (-04:00 EDT)',
//   '2022-05-01 in America/Indiana/Knox (-05:00 CDT)',
//   '2022-05-01 in America/Indiana/Marengo (-04:00 EDT)',
//   '2022-05-01 in America/Indiana/Petersburg (-04:00 EDT)',
//   '2022-05-01 in America/Indiana/Tell_City (-05:00 CDT)',
//   '2022-05-01 in America/Indiana/Vevay (-04:00 EDT)',
//   '2022-05-01 in America/Indiana/Vincennes (-04:00 EDT)',
//   '2022-05-01 in America/Indiana/Winamac (-04:00 EDT)',
//   '2022-04-30 in America/Juneau (-08:00 AKDT)',
//   '2022-05-01 in America/Kentucky/Louisville (-04:00 EDT)',
//   '2022-05-01 in America/Kentucky/Monticello (-04:00 EDT)',
//   '2022-04-30 in America/Los_Angeles (-07:00 PDT)',
//   '2022-05-01 in America/Menominee (-05:00 CDT)',
//   '2022-04-30 in America/Metlakatla (-08:00 AKDT)',
//   '2022-05-01 in America/New_York (-04:00 EDT)',
//   '2022-04-30 in America/Nome (-08:00 AKDT)',
//   '2022-05-01 in America/North_Dakota/Beulah (-05:00 CDT)',
//   '2022-05-01 in America/North_Dakota/Center (-05:00 CDT)',
//   '2022-05-01 in America/North_Dakota/New_Salem (-05:00 CDT)',
//   '2022-04-30 in America/Phoenix (-07:00 MST)',
//   '2022-04-30 in America/Sitka (-08:00 AKDT)',
//   '2022-04-30 in America/Yakutat (-08:00 AKDT)',
//   '2022-04-30 in Pacific/Honolulu (-10:00 HST)',
// ]
```

The list above includes both 2022-05-01 and 2022-04-30, which demonstrates that
it is not possible to always know a transaction date given the `utcDateTime` and
country of a transaction that takes place in a country with more than one
timezone.

Continuing on my goal of fixing the bug for our users in both GB and US, I
checked the bank API documentation.
Unfortunately, the API only gives the _country_ in which the transaction took
place, not the _timezone_ in which the transaction took place.
Unfazed, I made an informed guess and got lucky:
I manually investigated the `utcDateTimes` of transactions given by this bank's
API that took place in the US and concluded that this bank used the `'US'`
country code to mean the `'America/New_York'` timezone. I adjusted the code to
account for this case, published the change, and have not received bug reports
of incorrect transaction dates since.

Concluding word of advice: if you ever design an API for transactions, please
include the timezone in which the transaction took place, not just the country.
