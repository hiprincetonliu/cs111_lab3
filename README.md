# Hash Hash Hash
This lab tests a hash table which can be accessed by multiple threads at the same time. We are implementing pthread_mutex_t locks which help us ensure that no race conditions (nondeterministic behavior) are created when multiple threads are trying to access the hash table at the same time. Each entry of the hash table is a linked list of key/value pairs.

## Building
Run "make" in a terminal directory with all the files
```shell
make
```

## Running
To run, run command "./hash-table-tester". The "-t" flag is followed by the number of threads you want to run. The "-s" flag is followed by the number of hash table entries per thread.
```shell
./hash-table-tester -t NUMBER_OF_THREADS -s NUMBER_OF_ENTRIES
```
The output of the command will be the speed (in microseconds) of the generation of the hash table entries, the speeds of the hash table base, v1, v2, and the number of missing entries (caused by race conditions) each hash table version has ended with.

## First Implementation
In the `hash_table_v1_add_entry` function, I added a global pthread_mutex_t lock to control when threads are allowed to add to the hash table. If a thread is holding the lock, no other thread can modify the hash table.

The global lock is declared statically in the global scope. The lock is initialized in hash_table_v1_create(), and the lock is destroyed in hash_table_v1_destroy().

### Performance
```shell
./hash-table-tester -t 8 -s 100000
```
Version 1 is a little slower than the base version. This is because we have a single global lock for the hash table, meaning that multiple threads trying to access the hash table is completely redundant (since only one thread can modify the hash table at a time) and the program will spend time doing context switches (extra overhead) between these threads to check if the lock is available.

For example, ./hash-table-tester -t 8 -s 100000 on a 4 core machine yields a speed of 23,000,000 microseconds for the base hash table and a speed of 53,000,000 microseconds for the version 1 hash table.

## Second Implementation
In the `hash_table_v2_add_entry` function, I used a pthread_mutex_t lock for each one of the 4096 hash table entries. If a thread is holding the lock for a specific hash table entry (Ex: index 1234), no other thread can modify that specific hash table entry (index 1234). However, to increase speed, other threads can modify other hash table entries if they are unlocked (if index 2345 is unlocked we can change it).

In hash_table_entry, I added a pthread_mutex_t field because each hash_table_entry has its own lock. In hash_table_v2_create(), each lock is initialized when its corresponding entry is initialized. In hash_table_v2_destroy(), each lock is destroyed when its corresponding entry list is freed.

### Performance
```shell
./hash-table-tester -t 8 -s 100000
```

Version 2 is faster than the base version roughly by a factor of -t or the number of cores your CPU has, depending on which one is smaller. This is because the program will have -t threads accessing the hash table, but if there are more threads than cores, the number of concurrent threads is limited by the number of cores. Version 2 is faster than version 1 because there will be less collisions between threads since each entry has its own lock rather than the entire hash table. This means threads can run in parallel so long as they are not trying to modify the same hash table entry.

For example, ./hash-table-tester -t 8 -s 100000 on a 4 core machine yields a speed of 23,000,000 microseconds for the base hash table and a speed of 6,400,000 microseconds for the version 2 hash table. The reason why version 2 is not exactly 4 times faster than the base hash table is because of threads spending CPU time waiting to access a lock for a specific entry, which increases overhead slightly and prevents the threads from running in perfect parallel.

## Cleaning up
To clean the build, run "make clean" in a terminal directory with all the files
```shell
make clean
```