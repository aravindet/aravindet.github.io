---
title: Boring Time
---

# Background

The way humans keep time is unnecessarily complicated. Time should be more boring.

# Boring Time

This is a method of measuring time and date that makes calculations dead simple. That means no timezones, daylight saving time, leap years, leap seconds and other weirdness.

## Playground

<div class="clock-converter">
  <label>Gregorian Date & Time (Local)<input type='datetime-local' step='1' id='greg' /></label>
  <label>Boring Date & Time<input id='boring' size='18' /></label>
  <label>Boring Timestamp<input id='stamp' size='14' /></label>
</div>

Change any value to stop the clock. Clear any value to resume.

## The Coincidence

The ratio of the lengths of the year and day<sup>1</sup>, 365.2422, can be very closely approximated by the fraction 46,751 / 128. We'll make use of this fact.

> For comparison, the Gregorian Calendar with its “leap year every 4 years, except every 100, except every 400” rule results in an average of 365.2425, which is a less accurate approximation.

<sup>1</sup> mean solar year to mean solar day. Due [precession](https://en.wikipedia.org/wiki/Precession#Astronomy) this ratio varies more than the error in the boring approximation.

## Durations

- The base unit is the **bor**, which is exactly 675 SI seconds long.
- Bors may be subdivided into millibors (0.675 seconds), microbors (675 microseconds), etc.
- A day is 128 bors (86,400 seconds)
- A week is 7 days, or 896 bors
- A year is 46,751 bors (31,556,925 seconds)
- There is no concept of months or quarters

Converting between these units uses normal multiplication and division. While it is well-defined to specify a duration as a combination of multiple units (like 2 years and 26 weeks), this is discouraged: just use decimals (2.5 years). 

## Date and Time

The duration between a moment in time and the Boring Epoch (see below) is the Boring Timestamp. In contexts where decimals are permissible, it should be expressed in years; otherwise bors, millibors or microbors may be used based on the required precision. This sentence was written on 2022 Nov 5 at 18:25 SGT, which has the Boring Timestamp **2022.845705**.

Most date math involving durations is done using Timestamps, but for use in daily life they may be represented using friendlier Boring Datetimes. It's made up of the following components, separated by dots:

- **Year**, integer; the year 2022 CE is Boring Year 2022
- **Week of year**, an integer between 0 and 53
- **Day of week**, integer between 0 and 6; Monday is 0.
- **Bor of day**, real number between 0 (inclusive) and 128 (exclusive)

The timestamp above can also be written as the Boring Datetime **2022.44.6.75.56**.

## Quirks

Compared to our current Gregorian time system, there are a few key differences in when the boring day and year advances.

- At any moment, it is exactly the same Boring Time for everyone. There are no timezones or daylight saving.
  - As a consequence, different locations on earth will develop different conventions on what's the appropriate Bor of day to wake up, go to work, etc.
- The Day increments when the Bor rolls over from 127.99… to 0.
  - As a consequence, for some parts of the world the Day will increment during typical waking hours.
- The Year increments at the moment when the Boring Timestamp is a multiple of 46,751. This "New Years Moment" will not, in general, coincide with the moment the Day increments.
- The Week that contains the New Years Moment, called the New Years Week, is considered Week 52 (or 53) of the previous year until that moment _and_ Week 0 of the following year after it.

## Calculations

### Convert Unix timestamp to Boring Timestamp
Given a Unix timestamp ***t*** in seconds, the Boring Timestamp ***T*** in bors is ***t / 675 + 92,099,476***.

### Convert Boring Timestamp to Unix timestamp
Given a Boring Timestamp ***T*** in bors, the Unix timestamp ***t*** in seconds is ***(T - 92,099,476) * 675***.

### Convert Boring Timestamp to Boring Datetime

Given a timestamp ***T*** in bors,
- Year (Y) is ***⌊ T / 46,751 ⌋***
- Week (W) is ***⌊ T / 896 ⌋ - ⌊ 46,751 Y / 896 ⌋***
- Day (D) is ***⌊ T / 128 ⌋ % 7***
- Bor (B) is ***T % 128***

where ***⌊ … ⌋*** represents the floor function and ***%*** the modulo operation.

### Convert Boring Datetime to Boring Timestamp

Given a Year, Week, Day and Bor, the Boring Timestamp ***T*** is ***896 × (⌊ 46,751 Y / 896 ⌋ + W) + 128 D + B***.

## The Boring Epoch

The Boring Epoch, i.e. the moment at which the Boring Timestamp was 0, was chosen to meet the following compatibility considerations:

- The Boring Year should match the common calendar year. (Except ±1 day around New Years Day).
- Day increment should happen when the side of the earth where a majority of humans live is in darkness.

Accorrdingly, the Boring Epoch is 1 January 0001 BCE, at 00:00 in UTC+03:45.

# FAQ

- **Q** What about birthdays?
  - **A** Annually recurring events like birthdays should be defined as a *moment* rather than a day. The birthday moment repeats every year, i.e. 46,751 bors. This eliminates the problem of February 29th birthdays.

- **Q** Without months, when do I get my salary? When should I pay rent?
  - **A** How about every four weeks? i.e. whenever the Boring Timestamp in days can be expressed as ***28 N*** where ***N*** is an integer.

- **Q** When are my quarterly reports due?
  - **A** Every ¼ of a year, i.e. whenever the Boring Timestamp in bors can be expressed as ***46751 N / 4*** where ***N*** is an integer.

- **Q** Is this a serious proposal?
  - **A** No.

<style>
  .clock-converter {
    display: flex;
    flex-flow: row wrap;
    align-items: flex-start;
    justify-content: stretch;
  }
</style>

<script>
    document.querySelector('#greg').addEventListener('input', onGreg);
    document.querySelector('#stamp').addEventListener('input', onStamp);
    document.querySelector('#boring').addEventListener('input', onBoring);
    
    const epochT = 92099476;
    const yearBor = 46751;
    const borMs = 675000;
    const mBorMs = borMs / 1000;
    
    let gregTimer = null;
    let stampTimer = null;
    let boringTimer = null;
    function setNowGreg() {
      const d = new Date();
      const greg = d.toISOString().substr(0, 19);
      document.querySelector('#greg').value = greg;
      gregTimer = setTimeout(setNowGreg, 1000 - d.getTime() % 1000);
    }

    function setNowStamp() {
      const t = Date.now();
      const T = t / borMs + epochT;

      putStamp(T);
      const scale = 1e8 / (yearBor * borMs);
      const delay = Math.round(t * scale + 0.5) / scale - t;
      stampTimer = setTimeout(setNowStamp, delay);
    }

    function setNowBoring() {
      const t = Date.now();
      const T = t / borMs + epochT;
      putBoring(T);
      boringTimer = setTimeout(setNowBoring, mBorMs - t % mBorMs);
    }

    function startClock() {
      setNowBoring();
      setNowStamp();
      setNowGreg();
    }

    function stopClock() {
      clearTimeout(gregTimer);
      clearTimeout(stampTimer);
      clearTimeout(boringTimer);
    }

    function onGreg(event) {
      if (!event.target.value) {
        startClock();
        return;
      }
      stopClock();
      const t = Date.parse(event.target.value);
      const T = t / borMs + epochT;
      putStamp(T);
      putBoring(T);
    }

    function onBoring(event) {
      if (!event.target.value) {
        startClock();
        return;
      }
      stopClock();
      let [Y, W, D, B, frac] = event.target.value.split('.')
          .map(function (c) { return parseInt(c); });
      if (frac) B = parseFloat(B + '.' + frac);
      const T = (Math.floor(Y * yearBor / 896) + W) * 896 + D * 128 + B;
      putGreg(T);
      putBoring(T);
      putStamp(T);
    }

    function onStamp(event) {
      if (!event.target.value) {
        startClock();
        return;
      }
      stopClock();
      const T = parseFloat(event.target.value) * yearBor;
      putGreg(T);
      putBoring(T);
    }
    
    function putStamp(T) {
      document.querySelector('#stamp').value = (T / yearBor).toFixed(8);
    }

    function putBoring(T) {
      const Y = Math.floor(T / yearBor);
      const W = (Math.floor(T / 896) - Math.floor(Y * yearBor / 896))
          .toFixed(0); 
      const D = Math.floor(T / 128) % 7;
      const B = (T % 128).toFixed(3);
      const boring = `${Y}.${W}.${D}.${B}`;
      document.querySelector('#boring').value = boring;
    }

    function putGreg(T) {
      const t = (T - epochT) * borMs;
      const z = new Date().getTimezoneOffset() * 60 * 1000;
      const formatted = new Date(t - z).toISOString().substr(0, 16);
      document.querySelector('#greg').value = formatted;
    }

    startClock();
</script>
