The cost of a context switch
============================

*Note: all of the results below were generated on my laptop, an early 2015 MacBook Pro with a 3.1 GHz Intel Core i7 CPU running El Capitan.*

To measure the cost of a context switch with `gettimeofday`, we'll create a program that spawns two child processes, connects them via two pipes and has them alternately read from one pipe and write to the other. This will result in the processor rapidly context switching between the two processes (and potentially other processes as well--for example, the user might move the cursor using the mouse).

On a computer with multiple processors, the two child processes might be scheduled onto different processors, which will give us the wrong result. On macOS there is no way to guarantee that two processes get executed on the same processor; the best we can do is to give both processes the same 'affinity tag' and hope that the OS schedules the processes to the same processor.

There is another implementation detail worth mentioning: making the child that will issue the first write wait until the other child is ready to receive it (and thus avoid including the time taken to finish forking the other the child) can significantly lower measurements, especially when only a small number of context switches are measured.

Given the constraints on the precision of `gettimeofday`, we'll measure how long it takes to execute `n` context switches in a row. I have implemented this strategy in `context_switch.c`. Compiling this code and running it on my machine I get the *minimum* times outlined below.

Without modifying any affinity tags (i.e. letting the OS schedule the processes as it sees fit):

* 20 context switches in a row: 49 microseconds total, ~2.5 microseconds/switch
* 200 context switches in a row: 463 microseconds total, ~2.3 microseconds/switch
* 20000 context switches in a row: 39691 microseconds total, ~2 microseconds/switch

Modifying the affinity tags of all processes to all be nonzero and the same (i.e. indicating to the OS that the processes should be scheduled onto the same processor if possible):

* 20 context switches in a row: 49 microseconds total, ~2.5 microseconds/switch
* 200 context switches in a row: 505 microseconds total, ~2.5 microseconds/switch
* 20000 context switches in a row: 48239 microseconds total, ~2.4 microseconds/switch

Running this multiple times produces outputs that vary by as much as 20%, which suggests that either (1) regardless of the proccesses' affinity tags, both child processes are being scheduled to different processors or (2) the same processor, or (3) the processes are scheduled to the same processor for one setting of the affinity tag and different processors for the other setting, in which case a write then read on a pipe across processors takes about as long as a context switch (possibly because it involves a context switch to wake up the process that reads from the pipe).

These results should be compared against the results given by the lmbench tool.
