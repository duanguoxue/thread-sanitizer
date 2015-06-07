When `ThreadSanitizer` detects a bug it prints a report. Format of the reports differ depending on bug type (see DetectableBugs). We've tried to make the reports as self-explanatory as possible, but still there are some tips and tricks.

First, let's consider the simplest report:

```
WARNING: ThreadSanitizer: thread leak (pid=9509)
  Thread T1 (tid=0, finished) created at:
    #0 pthread_create tsan_interceptors.cc:683 (exe+0x00000001fb33)
    #1 main thread_leak3.c:10 (exe+0x000000003c7e)
```

As you can guess, it's for thread leak (non-joined thread).

Description of a thread always contains it's identifier, T1 in this case, or 'main thread'; it's OS tid (useful to match information with logs or a debugger); says whether it's currently running or finished and the stack trace where it was created.

Each stack frame contains function name, file name, file line and column (if it's available in the debug info). In the brackets you see dynamic module name ('exe' in this case stands for main executable) and offset in the module (can be useful to match with disassembly or to dedup reports).

Reports for other types of bugs are similar, with the only exception of data race reports -- they can contain a lot of blocks. Here is a simple data race report:

```
WARNING: ThreadSanitizer: data race (pid=9337)
  Write of size 4 at 0x7fe3c3075190 by thread T1:
    #0 foo1() simple_stack2.cc:9 (exe+0x000000003c9a)
    #1 bar1() simple_stack2.cc:16 (exe+0x000000003ce4)
    #2 Thread1(void*) simple_stack2.cc:34 (exe+0x000000003d99)

  Previous read of size 4 at 0x7fe3c3075190 by main thread:
    #0 foo2() simple_stack2.cc:20 (exe+0x000000003d0c)
    #1 bar2() simple_stack2.cc:29 (exe+0x000000003d74)
    #2 main simple_stack2.cc:41 (exe+0x000000003ddb)

  Thread T1 (tid=9338, running) created at:
    #0 pthread_create tsan_interceptors.cc:683 (exe+0x00000000de83)
    #1 main simple_stack2.cc:40 (exe+0x000000003dd6)
```

The report contains description of the conflicting memory accesses. For both accesses it says the address (that you can match with logs/debugger), size and whether it's read or write. Note that the first memory accesses is the "current" access, that is, the access during which the race is detected. While the second access is some previous memory access.

For both accesses the report says the thread id, and shows thread creation stack (except the main thread).

If the race happens on heap memory location, then the report also contains the allocation site and parameters of the heap block:
```
  Location is heap block of size 99 at 0x7d0600039030 allocated by thread T1:
    #0 malloc tsan_interceptors.cc:316 (exe+0x000000011717)
    #1 alloc() race_on_heap.cc:17 (exe+0x000000003dc4)
    #2 AllocThread(void*) race_on_heap.cc:21 (exe+0x000000003def)
```

Sometimes you can also see the following block:
```
  As if synchronized via sleep:
    #0 sleep tsan_interceptors.cc:214 (exe+0x000000004b9e)
    #1 Thread1(void*) simple_stack.c:27 (exe+0x000000003da4)
```
It is added in the following situation. Thread 1 had done the first conflicting memory access. Then thread 2 had executed some sleep function (sleep(), usleep(), nanosleep(), etc), and then had done the second conflicting access. The stack trace can be useful, because tests frequently "synchronize" this way.

If the threads hold some mutexes around the conflicting memory accesses, then the report is extended with the following information:
```
  Write of size 4 at 0x7f2f35fca190 by thread T1 (mutexes: write M1):
...
  Previous write of size 4 at 0x7f2f35fca190 by thread T2 (mutexes: write M2, read M3):
...
  Mutex M1 created at:
    #0 pthread_mutex_init tsan_interceptors.cc:737 (exe+0x00000000d9b1)
    #1 main mutexset6.cc:31 (exe+0x000000003e67)

  Mutex M2 created at:
    #0 pthread_spin_init tsan_interceptors.cc:796 (exe+0x00000000d091)
    #1 main mutexset6.cc:32 (exe+0x000000003e78)

  Mutex M3 created at:
    #0 pthread_rwlock_init tsan_interceptors.cc:839 (exe+0x00000000c8f1)
    #1 main mutexset6.cc:33 (exe+0x000000003e89)
```
For each memory access you can see the sets of locked mutexes and whether they were locked in write or read mode. And for each mutex the stack trace where the mutex was created.

Everything else is hopefully self-explanatory.