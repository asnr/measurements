Timer precisions
================

*Note: all of the results below were generated on my laptop, an early 2015 MacBook Pro with a 3.1 GHz Intel Core i7 CPU running El Capitan.*

We will use the `gettimeofday` function to measure various things. To do this, we will first need to get an idea of how precise `gettimeofday` is (we are not interested in accuracy, or how closely the time matches UTC, as we will be measuring differences in time).


Precision of `gettimeofday`
---------------------------

The function `gettimeofday` returns the number of seconds and microseconds since the Unix epoch (midnight January 1st 1970).

Without using a reference clock, we can get a rough idea of how precise `gettimeofday` is by calling it repeatedly and seeing (1) how many calls in a row return the same second, microsecond pair and (2) by how many microseconds the time advances when it does advance. (Note this approach will not detect if the clock that backs `gettimeofday` is consistently slow or consistently fast.)

The code `gettimeofday_runtimes.c` does exactly this. The number of calls to `gettimeofday` that it makes back-to-back are controlled by the static constant `NUM_SAMPLES`. Setting `NUM_SAMPLES` to 100,000 and inspecting the output (on my machine) shows that for most calls, `gettimeofday` returns the same value for 25 consecutive calls, indicating that most calls roughly 40 nanoseconds to complete. Very rarely however (on one indicative run, 215 times, or ~0.2%), between two consecutive calls there can be difference of many microseconds, with the maximum difference usually around 50 microseconds (although on the very occasional run, this will jump to almost 2000 microseconds).

In summary, the distribution of time between two calls to `gettimeofday` (and hence, a guide to the time taken to complete a single call and the precision of that call) appears to be a long-tailed distribution with the approximate shape of an exponential distribution and has a minimum of 40 nanoseconds.