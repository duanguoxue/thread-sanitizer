# Introduction #

`ThreadSanitizer` understands various flavors of compiler built-in atomic operations:
  1. [\_\_sync\_fetch\_and\_add](http://gcc.gnu.org/onlinedocs/gcc-4.3.5/gcc/Atomic-Builtins.html). This does not support atomic load operation and precise memory ordering.
  1. [\_\_c11\_atomic\_fetch\_add](http://clang.llvm.org/docs/LanguageExtensions.html#langext-c11-atomic). This is supported only by clang.
  1. [\_\_atomic\_load\_n](http://gcc.gnu.org/onlinedocs/gcc/_005f_005fatomic-Builtins.html). This requires relatively fresh compiler (at least gcc 4.7).

If std::atomic<> type is implemented using some sort of compiler built-ins (e.g. libc++), then `ThreadSanitizer` will understand it as well.

Also `ThreadSanitizer` runtime provides own set of atomic operations of the form:
```
__tsan_atomic8 __tsan_atomic8_load(const volatile __tsan_atomic8 *a, __tsan_memory_order mo);
```
The full list is available [here](http://llvm.org/viewvc/llvm-project/compiler-rt/trunk/lib/tsan/rtl/tsan_interface_atomic.h?view=markup).

So there are 3 options:
  1. Use only compiler intrinsics (or std::atomic<>).
  1. Use home-grown atomic operations for old compilers, for newer compilers use atomic\_load\_n atomic operations.
  1. Use home-grown atomic operations for normal build, under tsan use tsan\_atomic8\_load atomic operations.

You can see an example of option 2 [here](https://codereview.appspot.com/9586043/diff/31001/util/atomicops.h).