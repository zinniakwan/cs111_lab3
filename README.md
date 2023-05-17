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
In the `hash_table_v1_add_entry` function, we statically declared a single mutex lock at at the beginning of the file. Static varialbes are shared by all threads, and we will take advantage of this to prevent race conditions. After initializing the lock in hash_table_v1_create(), we call lock and unlock in the hash_table_v1_add_entry() function. The call to lock is at the beginning of the function and the call to unlock occurs before both return statements. The add_entry function is a crititcal section because it changes the hash table. For example, in the case that multiple threads are attempting to access the same linked list at a time, both threads could have the same pointer to the head of the linked list, thread 1 could insert a new node which becomes the new head, and then following thread 2 attempts to insert a new node but no longer has the currect pointer to the head of the linked list, leading to that node become "missing". By locking that whole function, all other threads must "wait" until the current thread accessing the function completes and unlocks. Because we always unlock before returning from the function, no locks will have inifinite wait time. We also prevent memory leaks by destroying the lock (calling pthread_mutex_destroy()) in the destroy function.

### Performance
Example 1:
```shell
./hash-table-tester -t 4 -s 25000
# Generation: 21,877 usec
# Hash table base: 41,559 usec
#   - 0 missing
# Hash table v1: 151,725 usec
#   - 0 missing
```

This time version 1 is 3.65 times slower than the base version. This is because our implementation provides atomicity through mutual eclusion, but is extremely slow because each thread must wait for another thread to finish accessing the data structure. Therefore, it creates a bottleneck of threads attempting to access this data structure, and behaves as slowly as a single-threaded program. Although we have multiple threads, it does not allow for much speed up through parallelism. Because the base implementation runs all of our code in serial, our version 1 is slower accounting for the overhead of creating the lock, time to lock/unlock, and waiting times of other threads.

Example 2:
```shell
./hash-table-tester -t 2 -s 50000
# Generation: 18,799 usec
# Hash table base: 40,819 usec
#   - 0 missing
# Hash table v1: 81,500 usec
#   - 0 missing
```
In this example, we have halfed the number of threads (but doubled number of requests per thread so total requests remains the same). In this case, the version 1 is only 1.99 times slower. Because we have fewer threads, there is less overhead in the process of locking/unlocking so the version 1 begins to behave more and more similar to the base case.

## Second Implementation
In the `hash_table_v2_add_entry` function, we create a seperate mutex for every hash table entry by declaring an array of mutex locks of the same size as the amount of entries. Because we only have possible race conditions when two threads try to insert into the same linked list, we only need to lock the specific hash_table_entry. The location of the locks remains the same as in v1 since the critical sections remain the same, however we access each lock by first calculating the index of the list_entry, and then grabbing lock from locks[index]. This implementation should be successful, because it prevents multiple theads from accessing the same list concurrently, but provides us extra speed because threads are still able to access other entries at the same time.

### Performance
Example 1:
```shell
./hash-table-tester -t 4 -s 40000
# Generation: 24,575 usec
# Hash table base: 173,758 usec
#   - 0 missing
# Hash table v2: 54,703 usec
#   - 0 missing
```
This time the speed up is 3.176 times faster, since we have more locks. Having a lock for each seperate list_entry, allows us to have more fine granularity and thus more chance for parallelism/speed-up. Ultimately, though we still have overhead of locking/unlocking, the speed we get from being able to run multiple threads on multiple cores is more advantageous.

Example 2:
```shell
# ./hash-table-tester -t 2 -s 80000
# Generation: 24,408 usec
# Hash table base: 162,249 usec
#   - 0 missing
# Hash table v1: 266,271 usec
#   - 0 missing
# Hash table v2: 94,822 usec
#   - 0 missing
```
When we decrease the number of threads, we ssee there is still a speedup of 1.71, but it has decreased from the previous example. This demonstrates how the advantage of the second implementation is the possiblity of running machine commands in parallel. Having more threads (as long as less then number of cores), provides more opportunity for parallelism and more speed-up.

## Cleaning up
```shell
make clean
```