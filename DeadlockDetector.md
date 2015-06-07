# Introduction #

The current Clang trunk version of ThreadSanitizer has an **experimental**
(i.e. "untested", "no warranty", etc) detector of lock order inversions (potential deadlocks).

Current status:
  * Only `pthread_mutex_*`/`pthread_rwlock_*`/`pthread_spin_*` are supported.
  * Tested only on tiny test cases.
  * The bug reports are not as informative as they could be (yet).

Contribution is welcome, first of all in form of tests.

# Algorithm #
The deadlock detector maintains a directed graph of lock acquisitions.
If a lock B is acquired while a lock A is being held by the same thread,
a directed edge `A=>B` is added to the lock acquisition graph.
A potential deadlock is reported when there is a cycle in the graph.

# Usage #
Just use `TSAN_OPTIONS=detect_deadlocks=1` when running your tsan-instrumented program.
```
% cat mutex_cycle2.c 
#include <pthread.h>

int main() {
  pthread_mutex_t mu1, mu2;
  pthread_mutex_init(&mu1, NULL);
  pthread_mutex_init(&mu2, NULL);

  // mu1 => mu2
  pthread_mutex_lock(&mu1);
  pthread_mutex_lock(&mu2);
  pthread_mutex_unlock(&mu2);
  pthread_mutex_unlock(&mu1);

  // mu2 => mu1
  pthread_mutex_lock(&mu2);
  pthread_mutex_lock(&mu1);  // <<<<<<< OOPS
  pthread_mutex_unlock(&mu1);
  pthread_mutex_unlock(&mu2);

  pthread_mutex_destroy(&mu1);
  pthread_mutex_destroy(&mu2);
}
% clang -g  -fsanitize=thread mutex_cycle2.c
% TSAN_OPTIONS=detect_deadlocks=1:second_deadlock_stack=1 ./a.out 
WARNING: ThreadSanitizer: lock-order-inversion (potential deadlock) (pid=24257)
  Cycle in lock order graph: M0 (0x7fff6bc83f68) => M1 (0x7fff6bc83f40) => M0
  
  Mutex M1 acquired here while holding mutex M0:
    #0 pthread_mutex_lock
    #1 main /tmp/mutex_cycle2.c:10
    
  Mutex M0 previously acquired by the same thread here:
    #0 pthread_mutex_lock
    #1 main /tmp/mutex_cycle2.c:9
    
  Mutex M0 acquired here while holding mutex M1:
    #0 pthread_mutex_lock
    #1 main /tmp/mutex_cycle2.c:16
    
  Mutex M1 previously acquired by the same thread here:
    #0 pthread_mutex_lock
    #1 main /tmp/mutex_cycle2.c:15
    
SUMMARY: ThreadSanitizer: lock-order-inversion (potential deadlock) /tmp/mutex_cycle2.c:10 main

% 
```

# Flags #
The flags are passed in TSAN\_OPTIONS env var for tsan, and DSAN\_OPTIONS for stand-alone deadlock detector.

| Flag name | Default value | Description |
|:----------|:--------------|:------------|
| detect\_deadlocks | false         | Enable deadlock detector (for tsan) |
| second\_deadlock\_stack | false         | Report two stacks per edge instead of one (where edge destination mutex is locked and where edge source mutex is locked) |