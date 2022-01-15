---
title: Boring Time
---

# Background

The way humans keep time is unnecessarily complicated. Time should be more boring.

# Boring Time

This is a method of measuring time and date with no timezones, daylight saving time, leap years, leap seconds and other weirdness.

## The Coincidence

The ratio of the length of the year to day<sup>1</sup>, 365.2422, can be very closely approximated by the fraction 46,751 / 128. It's freakishly close, with an error under 0.000002%. We'll make use of this fact.

<sup>1</sup> mean solar year to mean solar day.

## Base unit

The base unit is the **bor**, which is exactly 675 SI seconds long. A boring day is 128 bors (86,400 seconds), and a boring year is 46,751 bors (31,556,925 seconds). 

Note that these boring versions of the day and year are close enough to the real things that the difference won't matter for normal human affairs. For orbital mechanics and other reasons, the lengths of the actual day and year vary quite a bit more than the error here.

## Date and time

The number of bors since the Boring Epoch<sup>2</sup> is the Absolute Boring Time (ABT), written as a simple (albeit long) number. Boring Time can also be broken down into human friendly components separated by dots.

- Year, integer; the year 2022 CE is Boring Year 12022
- Day, integer between 0 and 365 or 366
- Bor, real number between 0 (inclusive) and 128 (exclusive)

This line was written on 15 January 2022 at 00:34 Singapore Time, or Boring Time 12022.014.088.

<input type='datetime-local' id='gregarious' /> -> <input id='boring' />
<script>
    document.querySelector('#gregarious').addEventListener('input', toBoring);
    document.querySelector('#boring').addEventListener('input', toGregarious);
    //
    function toBoring(event) {
        const t = Date.parse(event.target.value);
        const T = t / 675000 + 559609472;
        const Y = Math.floor(T / 46751);
        const D = (Math.floor(T / 128) - Math.floor(Y * 46751 / 128)).toFixed(0).padStart(3, '0');
        const B = (T % 128).toFixed(0).padStart(3, '0');
        const boring = `${Y}.${D}.${B}`;
        document.querySelector('#boring').value = boring;
    }
    //
    function toGregarious(event) {
        const [Y, D, B] = event.target.value.split('.').map(function (c) { return parseInt(c); });
        const T = (Math.floor(Y * 46751 / 128) + D) * 128 + B;
        const t = (T - 559609472) * 67500;
        const z = new Date().getTimezoneOffset() * 60 * 1000;
        const formatted = new Date(t - z).toISOString().substr(0, 16);
        document.querySelector('#gregorious').value = formatted;
    }
</script>

Compared to our current Gregorian (Gregarious?) time system, there are a few key differences in when the boring day and year advances.

- At any moment, it is exactly the same Boring Time for everyone. There are no timezones or daylight saving. As a consequence, different locations on earth will develop different conventions on what's the appropriate Bor of day to wake up, go to work, etc.
- The Day increments when the Bor hits 128. This means that for some parts of the world, the Day will increment during typical waking hours; people might go to work on Tuesday and return on Wednesday.
- The Year increments at the moment when ABT is a multiple of 46,751. This "new years moment" will not, in general, coincide with the moment the Day increments.
- The Day in which the Year increments is Day 365 (or 366) of the previous year until the new years moment, and Day 0 of the next year afterwards.

<sup>2<sup> The Boring Epoch is set to January 1, 10,001 BCE at midnight UTC, so the Boring Year equals 10,000 + the CE year. See the [Holocene Calendar](https://en.wikipedia.org/wiki/Holocene_calendar).

## Calculations

> This is where we collect our reward for the uncomfortable changes above.

Given a Unix timestamp t in seconds, the Absolute Boring Time T is `t / 675 + 559,656,221`.

Given an ABT T,
- Year is `floor(T / 46751)`
- Day is `floor(T / 128) - floor(Year * 46751 / 128)`
- Bor is `T % 128`
- Day of the week is `(floor(T / 128) + 5) * 7`,
 where 0 is Monday and 6 is Sunday. The 5 represents the fact that Jan 1 10,001 BCE was a Saturday.

Given a Year, Day and Bor, ABT is `(floor(Year * 46751 / 128) + Day) * 128 + Bor`. Time durations can be converted into an equivalent number of Bors with the formula `Years * 46751 + Days * 128 + Bors` and manipulated with simple arithmetic.

# FAQ

- **Q** Is this a serious proposal?
  **A** No.

