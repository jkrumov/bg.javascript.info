# Date and time

Let's meet a new built-in object: [Date](mdn:js/Date). It stores the date and provides methods for date/time management.

For instance, we can use it to store creation/modification times, or to measure time, or just to print out the current date.

[cut]

## Creation

To create a new `Date` object call `new Date()` with one of the following arguments:

`new Date()`
: Without arguments -- create a `Date` object for the current date and time:

    ```js run
    let now = new Date();
    alert( now ); // shows current date/time
    ```

`new Date(milliseconds)`
: Create a `Date` obeject with the time equal to number of milliseconds (1/1000 of a second) passed after the Jan 1st of 1970 UTC+0.

    ```js run
    // 0 means 01.01.1970 UTC+0
    let Jan01_1970 = new Date(0);
    alert( Jan01_1970 );

    // now add 24 hours, get 02.01.1970 UTC+0
    let Jan02_1970 = new Date(24 * 3600 * 1000);
    alert( Jan02_1970 );
    ```

    The number of milliseconds that has passed since the beginning of 1970 is called a *timestamp*.

    It is a lightweight numeric representation of a date. We can always create a date from a timestamp using `new Date(timestamp)` and convert the existing `Date` object to a timestamp, there's a `date.getTime()` method (see below).

`new Date(datestring)`
: If there is a single argument and it's a string, then it is parsed with the `Date.parse` algorithm (see below).


    ```js run
    let date = new Date("2017-01-26");
    alert( date ); // Thu Jan 26 2017 ...
    ```

`new Date(year, month, date, hours, minutes, seconds, ms)`
: Create the date with the given components in the local time zone. Only two first arguments are obligatory.

    Note:

    - The `year` must have 4 digits: `2013` is okay, `98` is not.
    - The `month` count starts with `0` (Jan), up to `11` (Dec).
    - The `date` parameter is actually the day of month, if absent then `1` is assumed.
    - If `hours/minutes/seconds/ms` is absent, then it is equal `0`.

    For instance:

    ```js
    new Date(2011, 0, 1, 0, 0, 0, 0); // // 1 Jan 2011, 00:00:00
    new Date(2011, 0, 1); // the same, hours etc are 0 by default
    ```

    The precision is 1 ms (1/1000 sec):

    ```js run
    let date = new Date(2011, 0, 1, 2, 3, 4, 567);
    alert( date ); // 1.01.2011, 02:03:04.567
    ```

## Access date components

The are many methods to access the year, month etc from the `Date` object. But they can be easily remembered when categorized.

`getFullYear()`
: Get the year (4 digits)

`getMonth()`
: Get the month, **from 0 to 11**.

`getDate()`
: Get the day of month, from 1 to 31, the name of the method does look a little bit strange.

`getHours(), getMinutes(), getSeconds(), getMilliseconds()`
: Get the corresponding time components.

```warn header="Not `getYear()`, but `getFullYear()`"
Many JavaScript engines implement a non-standard method `getYear()`. This method is non-standard. It returns 2-digit year sometimes. Please never use it. There is `getFullYear()` for that.
```

Additionally, we can get a day of week:

`getDay()`
: Get the day of week, from `0` (Sunday) to `6` (Saturday). The first day is always Sunday, in some countries that's not so, but can't be changed.

**All the methods above return the components relative to the local time zone.**

There are also their UTC-counterparts, that return day, month, year etc for the time zone UTC+0: `getUTCFullYear()`, `getUTCMonth()`, `getUTCDay()`. Just insert the `"UTC"` right after `"get"`.

If your local time zone is shifted relative to UTC, then the code below shows different hours:

```js run
// currend date
let date = new Date();

// the hour in your current time zone
alert( date.getHours() );

// what time is it now in London winter time (UTC+0)?
// the hour in UTC+0 time zone
alert( date.getUTCHours() );
```

Besides the given methods, there are two special ones, that do not have a UTC-variant:

`getTime()`
: Returns the timestamp for the date -- a number of milliseconds passed from the January 1st of 1970 UTC+0.

`getTimezoneOffset()`
: Returns the difference between the local time zene and UTC, in minutes:

    ```js run
    alert( new Date().getTimezoneOffset() ); // For UTC-1 outputs 60
    ```

## Setting date components

The following methods allow to set date/time components:

- `setFullYear(year [, month, date])`
- `setMonth(month [, date])`
- `setDate(date)`
- `setHours(hour [, min, sec, ms])`
- `setMinutes(min [, sec, ms])`
- `setSeconds(sec [, ms])`
- `setMilliseconds(ms)`
- `setTime(milliseconds)` (sets the whole date by milliseconds since 01.01.1970 UTC)

Every one of them except `setTime()` has a UTC-variant, for instance: `setUTCHours()`.

As we can see, some methods can set multiple components at once, for example `setHours`. Those components that are not mentioned -- are not modified.

For instance:

```js run
let today = new Date();

today.setHours(0);
alert( today ); // today, but the hour is changed to 0

today.setHours(0, 0, 0, 0);
alert( today ); // today, 00:00:00 sharp.
```

## Autocorrection

The *autocorrection* is a very handy feature of `Date` objects. We can set out-of-range values, and it will auto-adjust itself.

For instance:

```js run
let date = new Date(2013, 0, *!*32*/!*); // 32 Jan 2013 ?!?
alert(date); // ...is 1st Feb 2013!
```

**Out-of-range date components are distributed automatically.**

Let's say we need to increase the date "28 Feb 2016" by 2 days. It may be "2 Mar" or "1 Mar" in case of a leap-year. We don't need to think about it. Just add 2 days. The `Date` object will do the rest:

```js run
let date = new Date(2016, 1, 28);
*!*
date.setDate(date.getDate() + 2);
*/!*

alert( date ); // 1 Mar 2016
```

That feature is often used to get the date after the given period of time. For instance, let's get the date for "70 seconds after now":

```js run
let date = new Date();
date.setSeconds(date.getSeconds() + 70);

alert( date ); // shows the correct date
```

We can also set zero or even negative componens. For example:

```js run
let date = new Date(2016, 0, 2); // 2 Jan 2016

date.setDate(1); // set day 1 of month
alert( date );

date.setDate(0); // min day is 1, so the last day of the previous month is assumed
alert( date ); // 31 Dec 2015
```

## Date to number, date diff

When a `Date` object is converted to number, it becomes its number of milliseconds:

```js run
let date = new Date();
alert( +date ); // will the milliseconds, same as date.getTime()
```

**The important side effect: dates can be substracted, the result is their difference in ms.**

That's some times used for time measurements:

```js run
let start = new Date(); // start counting

// do the job
for (let i = 0; i < 100000; i++) {
  let doSomething = i * i * i;
}

let end = new Date(); // done

alert( `The loop took ${end - start} ms` );
```

## Date.now()

If we only want to measure the difference, we don't need `Date` object.

There's a special method `Date.now()` that returns the current timestamp.

It is semantically equivalent to `new Date().getTime()`, but it does not create an intermediate `Date` object. So it's faster and does not put pressure on garbage collection.

It is used mostly for convenience or when performance matters, like in games in JavaScript or other specialized applications.

So this is probably better:

```js run
*!*
let start = Date.now(); // milliseconds count from 1 Jan 1970
*/!*

// do the job
for (let i = 0; i < 100000; i++) {
  let doSomething = i * i * i;
}

*!*
let end = Date.now(); // done
*/!*

alert( `The loop took ${end - start} ms` ); // substract numbers, not dates
```

## Benchmarking

If we want a reliable benchmark of CPU-hungry function, we should be careful.

For instance, let's measure two function that get a delta between two dates, which one is faster?

```js
function diffSubstract(date1, date2) {
  return date2 - date1;
}

function diffGetTime(date1, date2) {
  return date2.getTime() - date1.getTime();
}
```

These two do exactly the same, because when dates are substracted, they are coerced into numbers of milliseconds, exactly the same thing as `date.getTime()` returns.

The first idea is to run them many times in a row and measure the difference. For our case, functions are very simple, so we have to do it around 100000 times.

Let's measure:

```js run
function diffSubstract(date1, date2) {
  return date2 - date1;
}

function diffGetTime(date1, date2) {
  return date2.getTime() - date1.getTime();
}

function bench(f) {
  let date1 = new Date(0);
  let date2 = new Date();

  let start = Date.now();
  for (let i = 0; i < 100000; i++) f(date1, date2);
  return Date.now() - start;
}

alert( 'Time of diffSubstract: ' + bench(diffSubstract) + 'ms' );
alert( 'Time of diffGetTime: ' + bench(diffGetTime) + 'ms' );
```

Wow! Using `getTime()` is so much faster! That's because there's no type conversion, it is much easier for engines to optimize.

Okay, we have something. But that's not a good benchmark yet.

Imagine that in the time of running `bench(diffSubstract)` CPU was doing something in parallel, and it was taking resources. And at the time of the second benchmark the work was finished.

Is that real? Of course it is, especially for the modern multi-process OS.

As a result, the first benchmark will have less CPU resources than the second. That's a reason for wrong results.

**For more reliable benchmarking, the whole pack of benchmarks should be rerun multiple times.**

Here's the code example:

```js run
function diffSubstract(date1, date2) {
  return date2 - date1;
}

function diffGetTime(date1, date2) {
  return date2.getTime() - date1.getTime();
}

function bench(f) {
  let date1 = new Date(0);
  let date2 = new Date();

  let start = Date.now();
  for (let i = 0; i < 100000; i++) f(date1, date2);
  return Date.now() - start;
}

let time1 = 0;
let time2 = 0;

*!*
// run bench(upperSlice) and bench(upperLoop) each 10 times alternating
for (let i = 0; i < 10; i++) {
  time1 += bench(diffSubstract);
  time2 += bench(diffGetTime);
}
*/!*

alert( 'Total time for diffSubstract: ' + time1 );
alert( 'Total time for diffGetTime: ' + time2 );
```

Modern JavaScript engines start applying advanced optimizations only to "hot code" that executes many times (no need to optimize rarely executed things). So, in the example above, first executions are not well-optimized. We may want to add a heat-up run:

```js
// heat up
bench(diffSubstract);
bench(diffGetTime);

// now benchmark
for (let i = 0; i < 10; i++) {
  time1 += bench(diffSubstract);
  time2 += bench(diffGetTime);
}
```

```warn header="Be careful doing micro-benchmarking"
Modern JavaScript engines perform many optimizations. Some of them may tweak artificial tests results compared to normal usage.

So if you seriously want to understand performance, then please first study how the JavaScript engine works. And then you probably won't need microbenchmarks at all, or apply them rarely.

The great pack of articles about V8 can be found at <http://mrale.ph>.
```

## Date.parse from a string

The method [Date.parse](mdn:js/Date/parse) can read the date from a string.

The format is: `YYYY-MM-DDTHH:mm:ss.sssZ`, where:

- `YYYY-MM-DD` -- is the date: year-month-day.
- The ordinary character `T` is used as the delimiter.
- `HH:mm:ss.sss` -- is the time: hours, minutes, seconds and milliseconds.
- The optional `'Z'` part denotes the time zone in the format `+-hh:mm` or a single `Z` that would mean UTC+0.

Shorter variants are also possible, like `YYYY-MM-DD` or `YYYY-MM` or even `YYYY`.

The call to `Date.parse(str)` parses the string in the given format and returns the timestamp (number of milliseconds from 1 Jan 1970 UTC+0). If the format is invalid, returns `NaN`.

For instance:

```js run
let ms = Date.parse('2012-01-26T13:51:50.417-07:00');

alert( ms ); // 1327611110417  (timestam)
```

We can instantly create a `new Date` object from the timestamp:

```js run
let date = new Date( Date.parse('2012-01-26T13:51:50.417-07:00') );

alert( date );  
```


## Summary

- Date and time in JavaScript are represented with the [Date](mdn:js/Date) object. We can't create "only date" or "only time".
- Months are counted from the zero (yes, January is a zero month).
- Days of week (for `getDay()`) are also counted from the zero (Sunday).
- `Date` auto-corrects itself when out-of-range components are set. Good for adding/substracting days/months/hours.
- Dates can be substructed, giving their difference in milliseconds. That's because a `Date` becomes the timestamp if converted to a number.
- Use `Date.now()` to get the current timestamp fast.

Note that unlike many other systems, timestamps in JavaScript are in milliseconds, not in seconds.

Also, sometimes we need more precise time measurements. JavaScript itself does not have a way to measure time in microseconds (1 millionth of a second), but most environments provide it.

For instance, browser has [performance.now()](mdn:api/Performance/now) that gives the number of milliseconds from the start of page loading, but adds 3 digits after the point to it. So totally it becomes microsecond percision:

```js run
alert(`Loading started ${performance.now()}ms ago`);
// Something like: "Loading started 34731.26000000001ms ago"
// it may show more than 3 digits after the decimal point, but only 3 first are correct
```

Node.JS has `microtime` module and other ways. Technically, any device and environment allows to get that, it's just not in `Date`.