# Hash Hash Hash
In this lab I implemented two versions of a hash table, starting from a given base implementation, and adding locks to ensure the program can be run concurrently by multiple threads.

## Building
In order to build the program (and get all necessary executables), run:
```shell
$ make clean
$ make
```

## Running
The program is to be run as follows:
```shell
$ ./hash-table-tester -t <#ofthreads> -s <#ofentriesperthread>
```
The integer after t sets the number of threads, and the integer after s sets the number of entries to be added per thread.

## First Implementation
In the `hash_table_v1_add_entry` function, I added a mutex lock, which I defined in the hash_table_v1 struct and created in the create() function of the table. In add_entry, I lock the lock before any other code. Then kept the code there to be ran once at a time by whoever owns the lock, and unlocked the lock at the end of the function. When we check if the value already exists and we just want to replace it, we also unlock here, before returning, thus avoiding a deadlock. In destroy() I destroyed the mutex lock before freeing the hash table. For every pthread_mutex function I used, I checked for errors and exited with an error code if it was the case.

This implementation is correct and I ran a lot of different test cases to test; it never missed any elements.

### Performance
```shell
$ ./hash-table-tester -t 3 -s 50000
```

Generation: 27,154 usec
Hash table base: 111,807 usec
  - 0 missing
Hash table v1: 275,958 usec
  - 0 missing
Hash table v2: 45,165 usec
  - 0 missing

Version 1 is slower than the base version. This version doesn't improve performance as compared to base, since only one thread at a time can add an entry. There is overhead associated with locks and locking a critical section.

## Second Implementation
In the `hash_table_v2_add_entry` function, I defined the mutex in the hash_table_entry, unlike before. This time we will have a lock per each table entry (or bucket), so that threads can work in parallel, as long as they each access a different table entry. So in the create() function, we initialize a lock in on every loop of the for, so one for every hash table entry. In add_entry(), we compute the hash_table_entry, and after this first line we lock the lock corresponding to this specific entry, so only one thread at a time can access this bucket. We unlock the lock in the if statement that just updates the value if it already exists, and unlock again at the end of the function, in the case this if statement was false. In the destroy() function we destroy each lock in the for loop that accesses all the table entries. I added proper error exists for different errors the program might encounter.

I tested this for correctness, it never missses any element. 
 
### Performance
```shell
$ ./hash-table-tester -t 4 -s 300000
```

Generation: 213,213 usec
Hash table base: 18,880,654 usec
  - 0 missing
Hash table v1: 23,848,719 usec
  - 0 missing
Hash table v2: 5,429,454 usec
  - 0 missing

After a lot of testing with different values for t and s, v2 always shows a significant performance improvement compared to base. In this case, ran with 4 threads (on a 4 core lnxsrv15 machine), and 300.000 elements per thread, v2 is 3.5x faster than base. This performance is achieved through a correct use of parallelism. The multi-threaded approach works amazingly here, as opposed to the previous v1, because threads can run the same code in parallel as long as they access different table entries. 

## Cleaning up
All binary files can be cleaned up by executing:
```shell
$ make clean
```