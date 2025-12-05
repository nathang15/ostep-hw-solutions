# The Abstraction: The Process

## Homework (Simulation)

This program, `process-run.py`, allows you to see how process states change as programs run and either use the CPU (e.g., perform an addin struction) or do I/O (e.g., send a request to a disk and wait for it to complete). See the README for details.

### Questions

<b>1. Run `process-run.py` with the following flags: `-l 5:100,5:100`. What should the CPU utilization be (e.g., the percent of time the CPU is in use?) Why do you know this? Use the `-c` and `-p` flags to see if you were right.</b>

    100%, there is no IO process.

<b>2. Now run with these flags: `./process-run.py -l 4:100,1:0`.These flags specify one process with 4 instructions (all to use the CPU), and one that simply issues an I/O and waits for it to be done.How long does it take to complete both processes? Use `-c` and `-p` to find out if you were right.</b>
    ```
    $ ./process-run.py -l 4:100,1:0 -c -p
    Time        PID: 0        PID: 1           CPU           IOs
      1        RUN:cpu         READY             1
      2        RUN:cpu         READY             1
      3        RUN:cpu         READY             1
      4        RUN:cpu         READY             1
      5           DONE        RUN:io             1
      6           DONE       BLOCKED                           1
      7           DONE       BLOCKED                           1
      8           DONE       BLOCKED                           1
      9           DONE       BLOCKED                           1
     10           DONE       BLOCKED                           1
     11*          DONE   RUN:io_done             1
    
    Stats: Total Time 11
    Stats: CPU Busy 6 (54.55%)
    Stats: IO Busy  5 (45.45%)
    ```

<b>3. Switch the order of the processes: `-l 1:0,4:100`. What happens now? Does switching the order matter? Why? (As always, use `-c` and `-p` to see if you were right)</b>

    Now process 1 runs when process 0 is waiting for IO completes. Yes switching the order matter because it dictates which process finishes first and that will affect the total runtime.

    ```
    $ ./process-run.py -l 1:0,4:100  -c -p
    Time        PID: 0        PID: 1           CPU           IOs
      1         RUN:io         READY             1
      2        BLOCKED       RUN:cpu             1             1
      3        BLOCKED       RUN:cpu             1             1
      4        BLOCKED       RUN:cpu             1             1
      5        BLOCKED       RUN:cpu             1             1
      6        BLOCKED          DONE                           1
      7*   RUN:io_done          DONE             1
    
    Stats: Total Time 7
    Stats: CPU Busy 6 (85.71%)
    Stats: IO Busy  5 (71.43%)
    ```

<b>4. We’ll now explore some of the other flags. One important flag is `-S`, which determines how the system reacts when a process issues an I/O. With the flag set to `SWITCH_ON_END`, the systemwill NOT switch to another process while one is doing I/O, instead waiting until the process is completely finished. What happens when you run the following two processes (`-l 1:0,4:100 -c -S SWITCH_ON_END`), one doing I/O and the other doing CPU work?</b>

    Proecss 1 will not run when process 0 is waiting for IO.

<b>5. Now, run the same processes, but with the switching behavior set to switch to another process whenever one is WAITING for I/O (`-l 1:0,4:100 -c -S SWITCH_ON_IO`). What happens now? Use `-c` and `-p` to confirm that you are right.</b>
```
    $ ./process-run.py -l 1:0,4:100 -c -p -S SWITCH_ON_IO
    Time        PID: 0        PID: 1           CPU           IOs
      1         RUN:io         READY             1
      2        BLOCKED       RUN:cpu             1             1
      3        BLOCKED       RUN:cpu             1             1
      4        BLOCKED       RUN:cpu             1             1
      5        BLOCKED       RUN:cpu             1             1
      6        BLOCKED          DONE                           1
      7*   RUN:io_done          DONE             1
    
    Stats: Total Time 7
    Stats: CPU Busy 6 (85.71%)
    Stats: IO Busy  5 (71.43%)
```
With SWITCH_ON_IO, when process 0 issues its I/O, the system immediately switches to process 1. Process 1 runs all 4 of its CPU instructions while process 0 is blocked waiting for I/O. Then process 0's I/O completes and it runs its io_done handler. This is way more efficient because the CPU isn't sitting idle during I/O; process 1 does useful work during that time.
       
<b>6. One other important behavior is what to do when an I/O completes. With `-I IO_RUN_LATER`, when an I/O completes, the process that issued it is not necessarily run right away; rather, whatever was running at the time keeps running. What happens when you run this combination of processes? (Run `./process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_LATER -c -p`) Are system resources being effectively utilized?</b>
```
    $ ./process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_LATER -c -p
    Time        PID: 0        PID: 1        PID: 2        PID: 3           CPU           IOs
      1         RUN:io         READY         READY         READY             1
      2        BLOCKED       RUN:cpu         READY         READY             1             1
      3        BLOCKED       RUN:cpu         READY         READY             1             1
      4        BLOCKED       RUN:cpu         READY         READY             1             1
      5        BLOCKED       RUN:cpu         READY         READY             1             1
      6        BLOCKED       RUN:cpu         READY         READY             1             1
      7*         READY          DONE       RUN:cpu         READY             1
      8          READY          DONE       RUN:cpu         READY             1
      9          READY          DONE       RUN:cpu         READY             1
     10          READY          DONE       RUN:cpu         READY             1
     11          READY          DONE       RUN:cpu         READY             1
     12          READY          DONE          DONE       RUN:cpu             1
     13          READY          DONE          DONE       RUN:cpu             1
     14          READY          DONE          DONE       RUN:cpu             1
     15          READY          DONE          DONE       RUN:cpu             1
     16          READY          DONE          DONE       RUN:cpu             1
     17    RUN:io_done          DONE          DONE          DONE             1
     18         RUN:io          DONE          DONE          DONE             1
     19        BLOCKED          DONE          DONE          DONE                           1
     20        BLOCKED          DONE          DONE          DONE                           1
     21        BLOCKED          DONE          DONE          DONE                           1
     22        BLOCKED          DONE          DONE          DONE                           1
     23        BLOCKED          DONE          DONE          DONE                           1
     24*   RUN:io_done          DONE          DONE          DONE             1
     25         RUN:io          DONE          DONE          DONE             1
     26        BLOCKED          DONE          DONE          DONE                           1
     27        BLOCKED          DONE          DONE          DONE                           1
     28        BLOCKED          DONE          DONE          DONE                           1
     29        BLOCKED          DONE          DONE          DONE                           1
     30        BLOCKED          DONE          DONE          DONE                           1
     31*   RUN:io_done          DONE          DONE          DONE             1
    
    Stats: Total Time 31
    Stats: CPU Busy 21 (67.74%)
    Stats: IO Busy  15 (48.39%)
```
With IO_RUN_LATER, when process 0's I/O completes, it doesn't immediately run. Instead, whatever CPU-bound process is currently running continues:
- Process 0 issues 3 I/Os quickly (switching on each)
- Processes 1, 2, 3 run their CPU instructions while process 0 is blocked
- When process 0's I/Os complete, they sit in READY state until their turn comes around
<b>7. Now run the same processes, but with `-I IO_RUN_IMMEDIATE` set, which immediately runs the process that issued the I/O. How does this behavior differ? Why might running a process that just completed an I/O again be a good idea?</b>
```
    $ ./process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_IMMEDIATE -c -p
    Time        PID: 0        PID: 1        PID: 2        PID: 3           CPU           IOs
      1         RUN:io         READY         READY         READY             1
      2        BLOCKED       RUN:cpu         READY         READY             1             1
      3        BLOCKED       RUN:cpu         READY         READY             1             1
      4        BLOCKED       RUN:cpu         READY         READY             1             1
      5        BLOCKED       RUN:cpu         READY         READY             1             1
      6        BLOCKED       RUN:cpu         READY         READY             1             1
      7*   RUN:io_done          DONE         READY         READY             1
      8         RUN:io          DONE         READY         READY             1
      9        BLOCKED          DONE       RUN:cpu         READY             1             1
     10        BLOCKED          DONE       RUN:cpu         READY             1             1
     11        BLOCKED          DONE       RUN:cpu         READY             1             1
     12        BLOCKED          DONE       RUN:cpu         READY             1             1
     13        BLOCKED          DONE       RUN:cpu         READY             1             1
     14*   RUN:io_done          DONE          DONE         READY             1
     15         RUN:io          DONE          DONE         READY             1
     16        BLOCKED          DONE          DONE       RUN:cpu             1             1
     17        BLOCKED          DONE          DONE       RUN:cpu             1             1
     18        BLOCKED          DONE          DONE       RUN:cpu             1             1
     19        BLOCKED          DONE          DONE       RUN:cpu             1             1
     20        BLOCKED          DONE          DONE       RUN:cpu             1             1
     21*   RUN:io_done          DONE          DONE          DONE             1
    
    Stats: Total Time 21
    Stats: CPU Busy 21 (100.00%)
    Stats: IO Busy  15 (71.43%)
```
With IO_RUN_IMMEDIATE, when an I/O completes, that process immediately preempts whatever is running to handle the completion.
This is better because process 0 can quickly issue its next I/O operation, allowing multiple I/Os to be in flight simultaneously. While process 0 waits for one I/O, it can have others completing and immediately issue new ones. This maximizes I/O device utilization. I/O-bound processes should handle completions immediately so they can issue new I/Os ASAP, keeping the I/O devices busy while CPU-bound processes use the CPU during the wait times.

<b>8. Now run with some randomly generated processes: `-s 1 -l 3:50,3:50` or `-s 2 -l 3:50,3:50` or `-s 3 -l 3:50,3:50`. See if you can predict how the trace will turn out. What happens when you use the flag `-I IO_RUN_IMMEDIATE` vs. `-I IO_RUN_LATER`? What happens when you use `-S SWITCH_ON_IO` vs. `-S SWITCH_ON_END`?</b>

Seed 1 Results
P0 does I/O twice, P1 does 3 CPU operations.
SWITCH_ON_IO: 15 time units
SWITCH_ON_END: 18 time units
With SWITCH_ON_IO, when P0 blocks on I/O at time 2, P1 immediately takes over and burns through its 3 CPU instructions while P0 waits. Done by time 15.
With SWITCH_ON_END, P1 just sits there in READY doing absolutely nothing while P0 waits for I/O. The CPU is idle for no reason. P1 doesn't run until time 16 after P0 finishes everything. Wasted 3 time units.
IO_RUN_IMMEDIATE vs IO_RUN_LATER didn't matter here because by the time P0's I/O finishes, P1 is already done.

------
Seed 2 Results
Both processes do I/O.
SWITCH_ON_IO: 16 time units (87.5% I/O utilization)
SWITCH_ON_END: 30 time units (66.7% I/O utilization)
This one's brutal. With SWITCH_ON_END, the scheduler just waits for P0 to completely finish before P1 even starts. Look at the trace - P0 runs from time 1-15, then P1 runs from time 16-30. The I/O operations run one after another with zero overlap. Almost double the time.
With SWITCH_ON_IO, both processes get their I/Os going early. Check times 4-6 and 11-13 in the trace - the IOs column shows "2", meaning both I/O devices are working simultaneously. That's why I/O utilization hits 87.5%.
IO_RUN_IMMEDIATE vs IO_RUN_LATER still didn't make a difference. The I/O completions happened to line up with natural switching points anyway.

------
Seed 3 Results
P0: cpu, io, io_done, cpu
P1: io, io_done, io, io_done, cpu
SWITCH_ON_IO + IO_RUN_IMMEDIATE: 17 time units
SWITCH_ON_IO + IO_RUN_LATER: 18 time units
SWITCH_ON_END: 24 time units
This is where IO_RUN_IMMEDIATE finally matters.
Look at what happens at time 8-9:
With IO_RUN_IMMEDIATE:
Both I/Os finish at time 8. P0 handles its completion, but then P1 immediately preempts and runs its io_done handler. P1 issues its next I/O at time 10. Meanwhile, P0 gets to run its last CPU instruction at time 11 while P1's I/O is in flight. Better overlap, done at time 17.
With IO_RUN_LATER:
Both I/Os finish at time 8. P0 handles completion and runs its last CPU instruction at time 9. P0 is done. Only then does P1 get to run at time 10. P1 issues its I/O, but now P0 is already finished so there's no CPU work to do during the wait. Finished at time 18.
With SWITCH_ON_END:
Same disaster as before. Sequential execution, no overlap, 24 time units.
