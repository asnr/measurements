The cost of a system call
=========================

*Note: all of the results below were generated on my laptop, an early 2015 MacBook Pro with a 3.1 GHz Intel Core i7 CPU running El Capitan.*

We'd like to measure the overhead of a system call using `gettimeofday` (see `../timers/README.md`). To do this, we'll measure the *shortest* amount of time taken to `read` zero bytes. Given the constraints on the precision of `gettimeofday`, we'll measure how long it takes to execute `n` `read` operations in a row. I have implemented this strategy in `read_system_call.c`. Compiling this code and running it on my machine I get the following *minimum* times:

* 100 read operations in a row: 330 nanosecs/read
* 1000 read operations in a row: 302 nanosecs/read
* 10000 read operations in a row: 302 nanosecs/read

(One potential weakness with using the minimum is that the system clock can be adjusted backwards by [NTP](https://en.wikipedia.org/wiki/Network_Time_Protocol), causing the cost to be underestimated. However, I ran each measurement multiple times and found that all the runs took the minimum amount of time or very close to it, which suggests that clock adjustments did not affect this result.)

Thus our best estimate of the overhead of a system call is ~300 nanoseconds.
