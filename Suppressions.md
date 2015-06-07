If you have a bug report that you are already aware of but can't fix right away, it may be useful to temporary suppress it.  Create an empty text file for suppressions and specify it in the suppressions runtime flag:
```
$ TSAN_OPTIONS="suppressions=/my/suppressions/file" ./myprogram
```

Each non-empty line of the suppressions file represents one suppression, each suppression is of the following form:
```
suppression_type:suppression_string
```
The suppression types are:
| race | suppresses data races and use-after-free reports |
|:-----|:-------------------------------------------------|
| thread | suppresses reports related to threads (leaks)    |
| mutex | suppresses reports related to mutexes (destruction of a locked mutex) |
| signal | suppresses reports related to signal handlers (handler calls malloc()) |
| deadlock | suppresses lock inversion reports                |
| called\_from\_lib | suppresses all interceptors in a particular library |

Suppression string is matched with functions names and file names in the main (first) stack of the report (for data races it's matched with the previous memory access stack as well).  Suppressions also matched with global variable names if present.

The suppression string can contain `*` symbol that matches with any substring.  `*` is automatically prepended and appended to each suppression string.  `^` and `$` are matched with string begin and end respectively.

Comment lines start with `#`.

An example of the suppressions file is:
```
# ThreadSanitizer suppressions file for project X.

# Library foobar is full of races.
# Filed bug 123, but do not want to deal with it now.
race:foobar

# The function turns to be racy. Bug 345.
race:NuclearRocket::Launch

# The race is introduced in patch 456. Bug 567.
race:src/surgery/laser_scalpel.cc

# Global var global_var is racy. Bug 568.
race:global_var

# short() function is racy, but not match any other functions containing "short". Bug 569.
race:^short$

# The following thread leaks. Bug 678.
thread:MonitoringThread

# Uninstrumented library.
called_from_lib:libzmq.so
```