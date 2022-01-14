# Prior art

Swatch Internet Time.

# The Coincidence

The ratio of the length of the year to day<sup>1</sup>, 365.2422, can be very closely approximated by the fraction 46751 / 128. 128 being a power of two allows for a convenient system for telling time, which we will call binary time.

<sup>1</sup> mean solar year to mean solar day.

### Base unit

The base unit is the mean solar day, subdivided using binary orders of magnitude analogous to Kibi, Mebi etc. As the IEC standard does not define negative exponents, the following are proposed:

- 2<sup>-10</sup>: mibi (mi; not to be confused with mebi, Mi)
- 2<sup>-20</sup>: mubi (ui; named after the letter mu μ, symbol for micro)
- nabi (ni), pibi (pi), febi (fi) etc.

A mibiday is exactly 84.375 SI seconds. A binary day is 1024 mibidays (86,400 seconds), and a binary year is 374,008 mibidays (31,556,925 seconds). 

Note that these are approximations of the astronomical definitions of day and year, and are only intended to hold over periods of the order of magnitude as several human lifetimes. Therefore no attempt is made to account for shifts in the lengths of the day and year.

### Date and time

The number of mibidays since the Unix epoch (1 January 1970) is considered the Absolute Time of an event. This means that 

It can be written as a simple (albeit long) number, or broken down into components, separated by dots:
- A year, with 1970 as year 0
- A day of year, between 0 and 366
- A time of day, between 0 and 1023
- Subdivisions of a mibiday, each between 0 and 1023

Given an absolute time T
- the time of day is `T % 1024`
- the year is `floor(T / 374008)`
- the day of year is `floor(T / 1024) - floor((year * 374008) / 1024)`

### No time zones

At any moment, it is exactly the same binary time for everyone.

There is no concept of local time, time zones, daylight saving time, etc. Different geographic locations will have local customs regarding the binary time at which they wake up, go to sleep, etc. The "day of year" component of the time may increment at an arbitrary time during the day, not necessarily in the night.

### No leap seconds

Binary time does not aim to fix the times of any astronomical events as seen from any fixed points on earth, and therefore does not need leap seconds to account for variations in the Earth's rotational speed. This means that over long timeframes, the time of day at which local midnight occurs will shift slowly (by perhaps a mibiday over a couple of centuries).

### No leap year cycle

The "new year" occurs at the specific moment where the year component increments. The day on which the moment falls (new years' day) is considered day 365 or 366 of the previous year until the new years' moment, *and* day 0 of the new year after that moment. This removes the need for a leap year.
