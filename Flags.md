
# Runtime Flags #

Runtime flags are passed via TSAN\_OPTIONS environment variable, separate flags are separated with spaces, e.g.:
```
$ TSAN_OPTIONS="history_size=7 force_seq_cst_atomics=1" ./myprogram
```

| Flag name | Default value | Description |
|:----------|:--------------|:------------|
| report\_bugs | 1             | Turns off bug reporting entirely (useful for benchmarking). |
| report\_thread\_leaks | 1             | Report thread leaks at exit? |
| report\_destroy\_locked | 1             | Report destruction of a locked mutex? |
| report\_signal\_unsafe| 1             | Report violations of async signal-safety (e.g. malloc() call from a signal handler). |
| history\_size | 2             | Per-thread history size, controls how many previous memory accesses are remembered per thread.  Possible values are [0..7].  history\_size=0 amounts to 32K memory accesses.  Each next value doubles the amount of memory accesses, up to history\_size=7 that amounts to 4M memory accesses.  The default value is 2 (128K memory accesses).  If you consistently see only one stack trace in reports, you may try increasing the value of the flag but keep in mind that it increases memory consumption. |
| force\_seq\_cst\_atomics | 0             | This flag turns all atomic operations into sequentially consistent operations.  This is useful to check whether a race is caused by incorrect memory ordering or not. |
| strip\_path\_prefix |               | Strip that prefix from file paths in reports. |
| suppressions |               | Suppressions filename.  See [Suppressions](http://code.google.com/p/thread-sanitizer/wiki/Suppressions) page for details. |
| exitcode  | 66            | Override exit status of the process if something was reported. |
| log\_path | stderr        | Write logs to "log\_path.pid".  The special values are "stdout" and "stderr". |
| atexit\_sleep\_ms | 1000          | Sleep in main thread before exiting for that many milliseconds.  Useful to catch "at exit" races. |
| verbosity | 0             | Verbosity level (0 - silent, 1 - a bit of output, 2+ - more output). |
| flush\_memory\_ms | 0             | Flush information about previous memory accesses every X ms.  This can dramatically reduce memory consumption, but we observed false reports when the flag is set (seemingly to a bug in Linux kernel).  Also note that data races with flushed accesses can not be detected.  Reasonable values are 1000-20000 (1-20 seconds) depending on how quickly the program grows memory consumption. |
| memory\_limit\_mb | 0             | Resident memory set limit to aim at,  in MB. If the process consumes more memory, TSan will flush shadow memory. |
| external\_symbolizer\_path |               | Path to llvm-symbolizer.  By default `ThreadSanitizer` uses addr2line utility to symbolize reports.  llvm-symbolizer is faster, consumes less memory and produces much better reports.  llvm-symbolizer is built as part of the LLVM build (see [Development](http://code.google.com/p/thread-sanitizer/wiki/Development) page). |
| io\_sync  | 1             | Controls level of synchronization implied by IO operations. 0 - no synchronization; 1 - reasonable level of synchronization (write->read on the same fd); 2 - global synchronization of all IO operations. |
| halt\_on\_error | 0             | Exit after first reported error |
| stop\_on\_start | 0             | Pause the program during `ThreadSanitizer` initialization. Useful to attach GDB early. Run 'gdb -p PID' (where PID is the value printed by the program). Then in gdb execute 'call tsan\_resume()'. |

# Compiler Flags (LLVM-specific) #

The following flags can be passed to clang compiler:

| Flag name | Default value | Description |
|:----------|:--------------|:------------|
| -fsanitize-blacklist= |               | Pass a blacklist file (see below) |
| -mllvm -tsan-instrument-memory-accesses | true          | If set to false, turns off instrumentation of memory accesses. Data races won't be detected in such files. |
| -mllvm -tsan-instrument-atomics | true          | If set to false, turns off instrumentation of atomic operations. Synchronization via atomic operations won't be tracked in such files. |
| -mllvm -tsan-instrument-func-entry-exit | true          | If set to false, turns off instrumentation of function entry/exit. Stack frames won't be restored in such files. |
| -gline-tables-only |               | Emit debug line number tables only. Significantly reduces debug info size. |
| -gcolumn-info |               | Emit source column debug info. Increases debug info, but provides better reports. Works only with llvm-symbolizer (see external\_symbolizer\_path above) |

# Blacklist Format #

```
# Ignore exactly this function (the names are mangled)
fun:MyFooBar
# Ignore MyFooBar(void) if it is in C++:
fun:_Z8MyFooBarv
# Ignore all function containing MyFooBar
fun:*MyFooBar*
# Ignore the whole file
src:file_with_tricky_code.cc
```