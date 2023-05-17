# Hash Hash Hash
### Zinnia Kwan, UID: 205777626
This lab we carry out the implementation of two distinct approaches in employing multiple threads to handle hash tables, while dealing with possiblity of race conditions. The initial approach in version 1 prioritizes accuracy by utilizing a single lock. The second approach in version 2 combines accuracy and performance by incorporating multiple locks. We use the pthread library to create and utilize mutexes.

## Building
we build the hash table executable by running
```shell
make
```

## Running
to run, in our command line we type 
```shell
./hash-table-tester -t [number of threads to use] -s [number of hash table entries per thread]
```
an example with 8 threads and 50000 entries per thread is:
```shell
./hash-table-tester -t 8 -s 50000
# Generation: 56,279 usec
# Hash table base: 1,748,427 usec
#   - 0 missing
# Hash table v1: 2,350,377 usec
#   - 0 missing
# Hash table v2: 231,345 usec
#   - 0 missing
```

## First Implementation
In the `hash_table_v1_add_entry` function, we statically declared a single mutex lock at at the beginning of the file. Static varialbes are shared by all threads, and we will take advantage of this to prevent race conditions. After initializing the lock in hash_table_v1_create(), we call lock and unlock in the hash_table_v1_add_entry() function. The call to lock is at the beginning of the function and the call to unlock occurs before both return statements. The add_entry function is a crititcal section because it changes the hash table. For example, in the case that multiple threads are attempting to access the same linked list at a time, both threads could have the same pointer to the head of the linked list, thread 1 could insert a new node which becomes the new head, and then following thread 2 attempts to insert a new node but no longer has the currect pointer to the head of the linked list, leading to that node become "missing". By locking that whole function, all other threads must "wait" until the current thread accessing the function completes and unlocks. Because we always unlock before returning from the function, no locks will have inifinite wait time. We also prevent memory leaks by destroying the lock (calling pthread_mutex_destroy()) in the destroy function. This implementation provides atomicity, but is extremely slow as it creates a bottleneck of threads attempting to access this data structure, and behaves as slowly as a single-threaded program.

### Performance
```shell

```

This time version 1 is xxx

## Second Implementation
In the `hash_table_v2_add_entry` function, xxx

### Performance
```shell

```

This time the speed up is xxx

## Cleaning up
```shell
make clean
```