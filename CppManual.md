
# Introduction #

`ThreadSanitizer` (aka TSan) is a data race detector for C/C++. Data races are one of the most common and hardest to debug types of bugs in concurrent systems. A data race occurs when two threads access the same variable concurrently and at least one of the accesses is write. [C++11](http://en.wikipedia.org/wiki/C%2B%2B11) standard officially bans data races as _undefined behavior_.

Here is an example of a data race that can lead to crashes and memory corruptions:

```
#include <pthread.h>
#include <stdio.h>
#include <string>
#include <map>

typedef std::map<std::string, std::string> map_t;

void *threadfunc(void *p) {
  map_t& m = *(map_t*)p;
  m["foo"] = "bar";
  return 0;
}

int main() {
  map_t m;
  pthread_t t;
  pthread_create(&t, 0, threadfunc, &m);
  printf("foo=%s\n", m["foo"].c_str());
  pthread_join(t, 0);
}
```

There are a lot of various ways to trigger a data race in C++, see PopularDataRaces, TSan detects all of them and more -- DetectableBugs.

# Obtaining `ThreadSanitizer` #

`ThreadSanitizer` is part of clang 3.2 and gcc 4.8. To build the freshest version see [Development](http://code.google.com/p/thread-sanitizer/wiki/Development) page.

# Supported Platforms #

TSan is supported on Linux x86\_64, tested on Ubuntu 12.04.

# Usage #

Simply compile your program with -fsanitize=thread and link it with -fsanitize=thread. To get a reasonable performance add -O2. Use -g to get file names and line numbers in the warning messages.

When you run the program, TSan will print a report if it finds a data race. Here is an example:

```
$ cat simple_race.cc
#include <pthread.h>
#include <stdio.h>

int Global;

void *Thread1(void *x) {
  Global++;
  return NULL;
}

void *Thread2(void *x) {
  Global--;
  return NULL;
}

int main() {
  pthread_t t[2];
  pthread_create(&t[0], NULL, Thread1, NULL);
  pthread_create(&t[1], NULL, Thread2, NULL);
  pthread_join(t[0], NULL);
  pthread_join(t[1], NULL);
}
```

```
$ clang++ simple_race.cc -fsanitize=thread -fPIE -pie -g
$ ./a.out 
==================
WARNING: ThreadSanitizer: data race (pid=26327)
  Write of size 4 at 0x7f89554701d0 by thread T1:
    #0 Thread1(void*) simple_race.cc:8 (exe+0x000000006e66)

  Previous write of size 4 at 0x7f89554701d0 by thread T2:
    #0 Thread2(void*) simple_race.cc:13 (exe+0x000000006ed6)

  Thread T1 (tid=26328, running) created at:
    #0 pthread_create tsan_interceptors.cc:683 (exe+0x00000001108b)
    #1 main simple_race.cc:19 (exe+0x000000006f39)

  Thread T2 (tid=26329, running) created at:
    #0 pthread_create tsan_interceptors.cc:683 (exe+0x00000001108b)
    #1 main simple_race.cc:20 (exe+0x000000006f63)
==================
ThreadSanitizer: reported 1 warnings
```

Refer to ReportFormat for explanation of reports format.

There is a bunch of runtime and compiler flags to tune behavior of TSan -- see [Flags](http://code.google.com/p/thread-sanitizer/wiki/Flags).

# Suppressing Reports #

Sometimes you can't fix the race (e.g. in third-party code) or don't want to do it straight away. There are several options how you can suppress known reports:

  * [Suppressions](http://code.google.com/p/thread-sanitizer/wiki/Suppressions) files (runtime mechanism).
  * [Blacklist](http://code.google.com/p/thread-sanitizer/wiki/Flags) files (compile-time mechanism).
  * Exclude problematic code/test under TSan with `#if defined(__has_feature) && __has_feature(thread_sanitizer)`.

# How To Test #

To start, run your tests using `ThreadSanitizer`. The race detector only finds races that happen at runtime, so it can't find races in code paths that are not executed. If your tests have incomplete coverage, you may find more races by running a binary built with -fsanitize=thread under a realistic workload.

# Runtime Overhead #

The cost of race detection varies by program, but for a typical program, memory usage may increase by 5-10x and execution time by 2-20x.

# FAQ #

  * Q: When I run the program, it says: `FATAL: ThreadSanitizer can not mmap the shadow memory (something is mapped at 0x555555554000 < 0x7cf000000000)`. What to do?
You need to enable ASLR:

```
$ echo 2 >/proc/sys/kernel/randomize_va_space
```

This may be fixed in future kernels, see https://bugzilla.kernel.org/show_bug.cgi?id=66721

  * Q: When I run the program under gdb, it says: `FATAL: ThreadSanitizer can not mmap the shadow memory (something is mapped at 0x555555554000 < 0x7cf000000000)`. What to do?
Run as:

```
$ gdb -ex 'set disable-randomization off' --args ./a.out
```

  * Q: Does it actually use hundreds of GBs of memory (as I see in top)?
No, it does not. TSan maps (but does not reserve) a lot of virtual address space. This means that tools like ulimit may not work as expected.

  * Q: I want to link libc/libstdc++ statically into my program. With `ThreadSanitizer` it produces either link-time or run-time errors.
Libc/libstdc++ static linking is not supported.

  * Q: What synchronization primitives are supported?
TSan supports pthread synchronization primitives, built-in compiler atomic operations (sync/atomic), C++ `<atomic>` operations are supported with llvm libc++ (not very throughly tested, though).

  * Q: I see what looks like a false report inside the libstdc++ (libc++) code.
This may happen in c++11 mode. One such case is discussed [here](http://lists.cs.uiuc.edu/pipermail/cfe-dev/2014-February/035408.html).
Note that `ThreadSanitizer` is not [yet](https://code.google.com/p/thread-sanitizer/issues/detail?id=48) heavily tested on code that uses c++11 threading.
The solution is be to rebuild libstdc++ (libc++) with `ThreadSanitizer` (this might be tricky, and the process changes periodically; contact us for details).
It is also possible that libstdc++ (libc++) has a bug, we've seen at least [one such](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=59215) before.

  * Q: My code with C++ exceptions does not work with tsan.
Tsan does not support C++ exceptions.

# Comments/Questions? #

Send comments/questions to thread-sanitizer@googlegroups.com

